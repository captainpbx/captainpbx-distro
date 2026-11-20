# Multi-tenant

One CaptainPBX appliance serves many tenants. There is one MariaDB, one Asterisk, and one web UI. From a tenant’s point of view they have their own PBX. They do not see another tenant’s extensions, SIP secrets, recordings, or admins.

A tenant is the isolation boundary. Platform admin works in **SYSTEM**. Tenant admin works inside one tenant.

## Data

The browser cannot choose a tenant by putting `tenant_id` in a request body. The server accepts a tenant only after it has checked that this login may enter it.

| Kind | Examples | Rule |
| --- | --- | --- |
| Global | Tenants, users, identities, roles | No tenant column. Membership is an explicit link from a user to a tenant. Caps live on the tenant row. |
| Tenant | Extensions, devices, groups, CDR, IVR, queues | Every query requires the current tenant. A child row keeps the parent’s tenant. |
| System | Audit, jobs, appliance health | Not a tenant-owned table. If a row refers to a tenant, that column is named as a subject, not as `tenant_id`. |

Tenant APIs refuse to run when no tenant is selected.

## SIP and calls

Every tenant has its own SIP domain from the day it is created. Two tenants never share that domain. A phone registers the local extension at that domain (`100@pbx.example.com`). The same extension number may exist in another tenant.

Calls from that tenant’s phones enter a dialplan context named for the tenant, not one shared internal context. The channel is tagged with the tenant so CDR and recordings follow the call, not whoever is logged into the website.

PJSIP object names include the SIP domain, so two tenants cannot compile the same endpoint. How a phone fills in the server address versus that domain is in [Register a SIP phone](sip-registration.md).

Media files (recordings, voicemail, music on hold) are stored under that tenant’s directory. Download and delete go through the tenant record, not a raw path from the browser.

## People

A login is an identity. The person is a user. Membership says which tenants that person may enter, and which catalog role they have in each one. The same person can be an admin in one tenant and a portal user in another. Switching tenant reloads what they are allowed to do.

Groups (teams) are organizational. They are not permissions.

Platform admin can open SYSTEM and any tenant. Everyone else only sees tenants they belong to. Tenant admin cannot raise caps or create tenants.

## Capacity

Platform admin sets seats, extensions, devices, and concurrent calls on the tenant. Tenant admin can see used against the cap and cannot raise it. Creating a user, extension, or device past the cap is refused. The dialplan also enforces the concurrent-call cap, including internal calls.

See also [Core architecture](core-architecture.md) for how User, Identity, Extension, and Device stay separate, [Register a SIP phone](sip-registration.md) for the same extension number in two SIP domains, and [Captain Shield](shield.md) for how the firewall is applied.

## Isolation

Two tenants can use the same extension number. The SIP domain, the dialplan context, and the tenant on the session keep each surface inside the tenant that owns it.

| Surface | Behavior |
| --- | --- |
| SIP registration | A phone registers `extension@sip-domain`. The password belongs to that endpoint. The same number on another domain is a different endpoint with its own password, and the registration shows only on the tenant that owns the domain. |
| Transfers | Blind and attended transfer use `tenant-<slug>-transfer`. That context includes the tenant’s extensions, features, parking, paging, and ring groups. It does not include outbound routes or another tenant, so extension 1001 in one tenant does not ring extension 1001 in another. |
| Voicemail | Mailboxes are reached through `tenant-<slug>-vm`. Each mailbox has its own PIN. The portal plays the signed-in user’s own mailbox. The admin voicemail page lists only the tenant that is open. |
| Recordings and CDR | The channel is tagged with the tenant. CDR and the recording follow that tag. Files live under the tenant directory. Admin pages list only the open tenant. The portal lists that user’s own recordings. Download and delete use the tenant record. A path or another tenant’s id from the browser does not open the file. |
| API roles | Membership sets the role for the tenant that is open. Switching tenant reloads that role. An API token is bound to the tenant it was created for. A `tenant_id` in the request does not change the tenant. Platform admin enters one tenant at a time. |

## Firewall, SIP ports, and RTP

One Asterisk serves every tenant. SIP listen ports and the RTP range are appliance settings, not a block of ports per tenant. Asterisk lends each live stream the next free port pair from that one RTP range and returns the ports when the stream ends. Two tenants can both use extension 1001 because the SIP domain and the dialplan context differ.

Captain Shield owns the firewall. The web app writes a desired file under `/var/lib/captainpbx/captainshield/desired/`. `captain-system-agent` applies it through `captain-fw` into nftables. PHP does not edit nftables.

| Exposure | Who sets it | Where it is enforced | What an installer checks |
| --- | --- | --- | --- |
| SIP signaling ports | **Captain Shield → Ports**. Defaults are UDP and TCP 5060, TLS 5061. Saved as `ports.json`. Global SIP shows these same listen ports. | nftables set `sip_ports`. The SIP allow list decides which addresses may use them. SIP stays locked to that list. | Captain Shield **Ports** and **Status**. `captain-fw sip-check` should report `sip_chain=ok`. |
| Who may send SIP | **Captain Shield** SIP allow list, plus country and rate rules when those are on. | The SIP chain is allow, then drop. Country rules sit in front of that drop. | Captain Shield **Access**. A phone off the allow list must not register. |
| RTP port range | **Platform → Global SIP Settings**. Default UDP 10000–20000 (`rtp_start` / `rtp_end`). | Sync & Apply writes `rtpstart` and `rtpend` in `/var/lib/captainpbx/asterisk/rtp_captain.conf`. Saving the range also writes `desired/rtp.json` and asks Captain Shield to open that UDP range (`rtp_media`). The range stays open so media can flow after SIP is allowed. It is not limited to the SIP allow list and it is not split by tenant. | The start and end in Global SIP, the same numbers in `rtp_captain.conf`, and the same UDP range on Captain Shield status. |
| Admin (SSH, HTTP, HTTPS) | **Captain Shield** admin allow list and **Ports**. | nftables set `admin_ports`. | Captain Shield **Access**. AMI, ARI, MariaDB, and Redis stay closed from the WAN. |
