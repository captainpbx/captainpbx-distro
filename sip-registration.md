# Register a SIP phone

A phone needs two names, and they answer different questions.

The **SIP server** is where the phone sends packets. That is the appliance IP, or the appliance's real FQDN when DNS points that name at the IP. The **organization SIP domain** is which organization the extension belongs to. CaptainPBX matches the registration `extension@sip-domain`. Two organizations can both have extension 1001 because the domain is different.

Copy the SIP domain from the organization in CaptainPBX. The phone does not invent one. After the extension and its SIP device exist, run Sync & Apply so the appliance will accept the registration.

The field names in the tables below are the ones MicroSIP shows. Other phones use the same values under their own labels. That mapping is at the end.

## Three names, for example

| Name | Example | Where it goes | What it does |
| --- | --- | --- | --- |
| Appliance IP | `64.177.48.xx` | **SIP Server** and **SIP Proxy** | Always reaches this CaptainPBX. |
| Real FQDN | `pbx.yourcompany.com` | **SIP Server** and **SIP Proxy**, in place of the IP | The public hostname of this SIP server. DNS must point it at the appliance IP. |
| Organization SIP domain | `sip.captainpbx` | **Domain** | Selects the organization. CaptainPBX stores this on the organization. It does not have to be a public DNS name. |

The real FQDN and the organization SIP domain can be the same string. They can also be different. The phone still has to be told both jobs: where to send packets, and which organization it is joining.

### The organization SIP domain is the real FQDN

One organization can use the server's public hostname as its SIP domain. Set that organization's SIP domain to `pbx.yourcompany.com`, and give that name a DNS record that points at `64.177.48.xx`.

| Field | Value |
| --- | --- |
| SIP Server | `pbx.yourcompany.com` |
| SIP Proxy | `pbx.yourcompany.com` |
| Domain | `pbx.yourcompany.com` |

The registration is `1001@pbx.yourcompany.com`, and the phone can find the server by that same name. SIP Server may still be the IP. Domain stays `pbx.yourcompany.com` either way.

Only one organization can use that FQDN. A second organization needs its own SIP domain.

### The organization SIP domain is its own name

This is the usual multi-organization layout. The real FQDN (or the IP) is how every phone finds the server. Each organization has a SIP domain of its own, and that name does not need a DNS record.

| Field | Value |
| --- | --- |
| SIP Server | `64.177.48.xx` or `pbx.yourcompany.com` |
| SIP Proxy | The same value as SIP Server |
| Domain | `sip.captainpbx` |

The registration is `1001@sip.captainpbx` sent to `64.177.48.xx` (or to `pbx.yourcompany.com` if that name resolves there), port 5060. UDP is the usual transport. Use TCP or TLS only when the device in CaptainPBX is set that way.

`sip.captainpbx` is not the server's hostname here. Putting it in **SIP Server** makes the phone look that name up in DNS. With no public record, the registration never leaves the phone. Keep it in **Domain** only.

## Account fields

| Field | What to enter | Used for registration |
| --- | --- | --- |
| Account Name | Any label, for example `vultr-captainpbx` | No. Only the name in the phone's account list. |
| SIP Server | Appliance IP, or its real FQDN. For example `64.177.48.xx` or `pbx.yourcompany.com` | Yes. Where packets go. |
| SIP Proxy | The same value as SIP Server | Yes. Sends the registration to that host. |
| Username | Extension number, for example `1001` | Yes. The user half of `1001@domain`. |
| Domain | Organization SIP domain. This may be the real FQDN, or a separate name such as `sip.captainpbx` | Yes. The domain half. This selects the organization. |
| Login | Same as Username for the first device on that extension | Yes. Digest username. |
| Password | SIP password of that device | Yes. This is the device secret, not the web login. |
| Display Name | Optional | No. |
| Voicemail Number | Optional | No. |
| Dialing Prefix | Optional | No. |

A second SIP device on the same extension uses Username and Login `1001-2`. The domain stays the organization's domain. Further devices use `1001-3`, and so on.

## Two organizations, one appliance

The server's real FQDN is `pbx.yourcompany.com`, and DNS points it at `64.177.48.xx`. Both phones may use that FQDN or the IP in SIP Server and SIP Proxy. Both use extension 1001. The organization SIP domain is what keeps them apart. Neither organization is using the real FQDN as its SIP domain.

### North — domain `sip.captainpbx`

| Field | Value |
| --- | --- |
| SIP Server | `64.177.48.xx` |
| SIP Proxy | `64.177.48.xx` |
| Username | `1001` |
| Domain | `sip.captainpbx` |
| Login | `1001` |
| Password | SIP password of North's 1001 |

CaptainPBX registers this phone as `1001@sip.captainpbx`. Calls from it stay in the North organization.

### South — domain `south.sip.captainpbx`

| Field | Value |
| --- | --- |
| SIP Server | `pbx.yourcompany.com` |
| SIP Proxy | `pbx.yourcompany.com` |
| Username | `1001` |
| Domain | `south.sip.captainpbx` |
| Login | `1001` |
| Password | SIP password of South's 1001 |

CaptainPBX registers this phone as `1001@south.sip.captainpbx`. South's phone finds the server by the real FQDN. North's phone finds it by the IP. Both arrive at the same appliance. The organization is chosen by Domain, so South's 1001 is a different extension from North's 1001. Dialing 1001 on the South phone rings South. Dialing 1001 on the North phone rings North.

Replace `64.177.48.xx` and `pbx.yourcompany.com` with your appliance IP and its real FQDN. Replace the domains with the SIP domain shown on each organization. One server, one SIP domain per organization. That domain may be the real FQDN, or a separate name.

## Before the phone will register

1. The organization has a SIP domain. It is unique on this appliance. It can be the appliance's real FQDN, or a separate name. If you leave it blank at create time and a SIP base is configured, CaptainPBX assigns `{organization slug}.{sip base}`.
2. The extension exists in that organization, with a SIP device. Copy that device's SIP password into the phone.
3. Sync & Apply has run since the device was created.
4. Captain Shield allows this phone's address to reach SIP. A phone that is not allowed never completes registration, even when the account fields are right. See [Captain Shield](shield.md).

UDP SIP is port 5060. TLS is 5061 when the device transport is TLS. Media uses RTP ports 10000–20000.

## Field names on other phones

The labels differ. The split does not. Server, proxy, outbound proxy, or registrar address is the appliance IP or its real FQDN. Domain, realm, or the host part of the SIP URI is the organization's SIP domain, which may be that same FQDN or a separate name. Auth user and auth password are the extension (or `1001-2` for a second device) and that device's SIP password.
