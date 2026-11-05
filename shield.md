# Captain Shield firewall — production design

Canonical design for CaptainPBX host firewall (module **`captainshield`**). Privilege model and layers: [architecture](architecture.md).

PHP never becomes root, never reads `/etc`, never `shell_exec`. Desired policy under `/var/lib/captainpbx/captainshield/desired/` → Unix socket → `captain-system-agent` → `captain-fw` / nftables.

---

## Operator story (keep the UI this simple)

Firewall is critical. The UI answers three questions:

1. **Who may manage this box?** (SSH + HTTPS)
2. **Who may talk SIP?** (phones / trunks)
3. **Who is blocked, and what extra shields are on?** (blocks, countries, IDS, community list, rates)

Primary copy never says zones, nft, chains, or jails.

### Page layout

One **Captain Shield → Firewall** page, four cards + collapsed Advanced:

| Card | Title | Contents |
| --- | --- | --- |
| Status | Firewall status | Healthy badge, last Apply, **Your IP**, **Allow my IP**, **Apply changes** |
| Access | Who can access | **Admin** whitelist + **Phone/SIP** whitelist — IP, CIDR, or hostname (FQDN) |
| Lists | Whitelist / blocklist | Open ports, Admin/SIP whitelist with resolved FQDN IPs, blocklist; move between lists |
| Blocklist | Permanent + temporary | Manual blocklist + auto bans; move back to whitelist |
| Countries | Countries | Allow/deny/off; country multi-select for Admin and for Phone/SIP |

**IDS** tab: editable failure thresholds (web / SIP / SSH), ban scope (single IP vs /24), and connection rate meters (not IDS — nft flood caps). Temporary bans are managed on **Blocklist**.

**Advanced** (removed): content lives under **IDS**.

Lockout guard: refuse Apply that locks out the browser IP unless “I have console access” is checked. IDS never auto-bans the active platform-admin session IP.

Actions on Overview:

| Action | Effect |
| --- | --- |
| **Restart setup** | Re-open wizard; draft seeded from current lists (no wipe) |
| **Emergency lockdown** | Force Admin trusted + SIP lock; seed client IP if missing; keep lists |
| **Clear & start fresh** | Wipe Admin/SIP/block/geo; leave Admin/SIP open; empty wizard (needs console confirm) |

Allow / block entries: **IP**, **subnet (CIDR)**, or **FQDN** (agent resolves all A/AAAA into nft sets).

### First login reminder

When the Admin allow list is empty (`setup_needed`), platform admins see:

1. A top-of-app **Firewall setup required** banner (Set up Firewall / Later this session).
2. On the Firewall page, a **step-by-step setup wizard**. Choices stay in a local draft until **Apply Captain Shield** on the last step, which calls `POST /api/v1/captainshield/setup` once (Admin + SIP + geo + agent apply).

The banner returns on the next login until at least one Admin allow entry exists.

---

## Default posture (production)

| Control | Default |
| --- | --- |
| Admin (SSH / HTTP / HTTPS) | **Open until setup** — first UI login seeds the browser IP and private LAN `/24`. Empty trusted mode is treated as open so the wizard cannot lock the operator out. Apply setup → trusted. |
| SIP (5060/5061) | **Lock** — SIP allow list + trusted |
| AMI / ARI / MariaDB / Redis | **Always blocked** on WAN — no toggle |
| RTP | **Always open** (not IP-whitelisted) — range from Global SIP Settings (`desired/rtp.json`) |
| IDS | On |
| Countries | Allow-list chosen at setup |
| Rates | On (SSH + SIP) |
| Community threat list | On when APIBAN key present + enabled in UI |

Evaluation order: block/community/IDS ban → allow-list hit → country rule → rate limit → accept/drop.

`captain-fw reload` installs SIP **allow-then-drop** first, then loads country packs and inserts those rules in front of the lock drop. A hung US/IN geo load must not leave `sip_access` as drop-only (REGISTER visible in a packet capture but never reaching Asterisk). The nft stub in `captainpbx-iso/overlay/etc/nftables.conf` (and the ISO bake copy `dev/firewall/nftables.conf`) is also allow-then-drop so `nft -f` itself is safe. Reload must not abort on a missing `sip_ports` chain — older appliances had a drop-only stub and `set -e` stopped before allow-then-drop (`sip_chain=drop_only`).

**RTP** is jumped early in the input chain (`rtp_media`) and is never gated by Admin/SIP allow lists. Changing RTP start/end under Global SIP writes `desired/rtp.json`; Sync & Apply / save triggers `firewall.apply` so `captain-fw` rebuilds the UDP range.

### Listen ports (Captain Shield → Ports)

Operators change SSH, HTTP, HTTPS, SIP UDP/TCP/TLS, and add extra ports. Each extra uses the **UI & SSH** or **SIP** allow list. **Attach** is Captain HTTPS (nginx extra listen), PJSIP, or **Firewall only** (open the hole; you run the service). Desired file `ports.json`. `captain-fw` rebuilds `admin_ports` / `sip_ports`. The system agent rewrites sshd `Port` and nginx `captainpbx-listen.d` only for Captain HTTPS extras. Extra PJSIP binds compile as extra transports (Asterisk Sync & Apply). AMI/ARI/MariaDB/Redis ports stay forbidden.

Allow-list **beats** country rules. Explicit block **beats** allow.

---

## FQDN allow / block

Operators may enter a hostname on Admin allow, SIP allow, or Block.

- Desired stores the **name**; the **agent** resolves A/AAAA on Apply and on a timer.
- One FQDN → many IPs is normal; all go into nft sets.
- DNS failure: keep last good IPs and warn on Status.
- Boot uses last resolved snapshot; live DNS is not required at boot.

---

## Community threat list (APIBAN)

Ship **APIBAN** (SIP + HTTP honeypot IPs). UI (Captain Shield → APIBAN):

- Paste API key → **Save & enable** (agent verb `threat.key.set`, then `threat.refresh`)
- Status shows key configured (never the secret), last sync, listed IP count, last error
- Manual **Refresh now** / Enable / Disable

Agent stores the key at `/var/lib/captainpbx/captainshield/secrets/apiban.key` (root-only). Desired meta in `threat.json` includes `key_configured` and `last_error`. Allow-list overrides community block for the same IP.

CrowdSec / foreign iptables clients are out of scope; we own apply via the system agent.

---

## IDS and rates

| Jail | Source | How it fires |
| --- | --- | --- |
| `web_login` | UI / API login failures | AuthService → `IntrusionBan::reportFailure` on every fail |
| `sip_auth` | Asterisk auth / REGISTER fails | Job `captainshield-ids-poll` → agent `ids.poll` (journal + Asterisk logs) → reportFailure |
| `ssh` | sshd Failed password / Invalid user | Same poll job / agent |

`sip_scan` remains in `ids.json` for a future flood heuristic; it is not shown in the UI and is not wired.

**Ban scope** (`ids.json` → `ban_scope`): `ip` (default) bans only the offending host; `subnet24` bans `x.x.x.0/24` (aggressive — for scanners). Whitelisted Admin/SIP sources are never banned. Temporary bans appear under **Blocklist** (unban manually there). Shared NAT/office egress IPs should be on the SIP whitelist so one bad phone does not ban everyone behind that public IP.

Large allow/block lists: status returns **counts** only for enriched rows; `GET /api/v1/captainshield/lists?kind=admin|sip|block|bans&q=&page=&per_page=` serves search + pagination (default 25/page, max 100). UI renders one page at a time.

---

## AMI policy

Never expose AMI (5038) or ARI HTTP on the WAN. Tenants use CaptainPBX HTTPS API or localhost AMI via tunnel. No UI toggle.

---

## Lockout recovery (minimal)

- UI: Allow my IP + refuse suicidal Apply.
- Console (user `captain`): `captain shield status|blocked|ban|unban|trust|untrust|lookup|panic-open`. These never dump the community threat set.
- Root SSH last resort: `captain-fw panic`.
- Hypervisor serial/VNC if SSH is dead.

**Deferred:** WireGuard, NetBird, PortKnocker.

---

## Desired tree

`/var/lib/captainpbx/captainshield/desired/`

| File | Role |
| --- | --- |
| `mode` | `open` \| `trusted` |
| `sip_mode` | `open` \| `lock` |
| `admin.allow` | IP/CIDR/FQDN lines (Admin: UI + SSH) |
| `sip.allow` | IP/CIDR/FQDN lines (SIP) |
| `block.permanent` | IP/CIDR/FQDN lines |
| `block.meta.json` | Who added each permanent block (`source`, `by` login, `at`) |
| `bans.active.json` | Temporary bans with `source` (`user`\|`ids`), `by`, `reason`, `until` |
| `geo.json` | mode + admin/sip country codes |
| `ids.json` | thresholds |
| `rates.json` | SSH/SIP limits |
| `threat.json` | `{ "enabled": true, "key_configured": true, "last_sync": "…", "count": 1234, "last_error": null }` |
| `rtp.json` | `{ "start": 10000, "end": 20000, "always_open": true, "source": "asterisk.global_sip" }` — media UDP range (always accepted) |
| `resolved/*.json` | agent-owned last-good DNS expansions (root) |

Live nft inputs (root only): `/etc/captainpbx/firewall/`.

---

## Agent verbs

| Verb | Effect |
| --- | --- |
| `firewall.apply` | Resolve FQDNs, sync desired → live, reload nft, then `sip-check`. Success when `sip_chain=ok` (even if geo is still loading). SSH/nginx listen errors are `listen_error`, not a failed SIP apply. |
| `firewall.inspect` | Read-only `captain-fw sip-check` (no desired sync) |
| `firewall.allow` / `revoke` | Admin allow mutate + apply |
| `firewall.ban` / `unban` | Temporary blocklist |
| `firewall.panic-open` | Timed admin open, then restore |
| `geo.refresh` | Refresh country CIDR packs |
| `threat.refresh` | Pull APIBAN into community set |
| `fqdn.refresh` | Re-resolve hostnames only |

---

## Permissions

- `captainshield.read` / `captainshield.write` — platform admin; host firewall is not tenant-scoped.
- Never trust `tenant_id` from the browser for host FW.

## Audit

Every Captain Shield write (setup, lists, block/unban, geo, rates, APIBAN, lockdown/reset) records an `audit_events` row via `AuditHook` (`insert`/`update`/`delete` on `captainshield_*` entity types). IDS auto-bans record as Created Temporary ban with Who = `IDS · …`. Actor login is resolved for user actions. API keys are never stored in after_json.

---

## Explicit non-goals

- PHP editing nft or reading `/etc`
- Public AMI/ARI/DB
- Per-tenant host firewall
- PortKnocker / WireGuard / NetBird (later)
- Crowdsourced feed as the only defense

## Success criteria

- Unlisted WAN IP cannot reach SSH, HTTPS, or SIP when posture is locked.
- Non-network admins can manage allow/block/geo from one page.
- FQDN expands to many IPs safely; APIBAN blocks scanners without breaking allow-listed trunks.
- Every Apply returns ok/error in one request via the system agent.
- Apply / Status fail closed if live `sip_access` is drop-only (`sip_chain` ≠ `ok`).

---

## Debug SIP that never reaches Asterisk

If a phone REGISTER is on the wire but Asterisk shows the endpoint Unavailable, check the **live** chain — not only the UI allow list.

| Command | What it tells you |
| --- | --- |
| `journalctl -t captain-fw -n 80 --no-pager` | Reload steps, geo start/finish, drop-only warnings |
| `journalctl -t captain-system-agent -n 80 --no-pager` | `sip-check sip_chain=… sip_drop_packets=…` after Apply |
| `captain-fw sip-check` | `sip_chain=ok` / `drop_only` / `drop_before_allow` / `missing` |
| `captain shield status` | Same `sip_chain=` line from `firewall.inspect` |
| `nft list chain inet captain sip_access` | Must list `@sip_allow4` accept **before** the lock drop |

`sip_drop_packets` climbing while `sip_chain=drop_only` means new SIP is discarded before Asterisk. Fix: `captain-fw reload` (or Re-apply in Captain Shield). Do not hand-edit `/etc/asterisk`.

Platform Watch (SYSTEM) and the header Alerts bell raise a critical notification when `sip_chain` is `missing` (nftables table not loaded) or `drop_only` / `drop_before_allow`. The alert links to Captain Shield. It clears after a healthy sip-check (Re-apply succeeded). Probes run from SystemWatch collect, Shield IDS poll, and Shield status.
