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

See also [Core architecture](core-architecture.md) for how User, Identity, Extension, and Device stay separate.
