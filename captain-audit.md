# Captain Audit

Captain Audit is the appliance trail of who changed what. It records admin and portal actions, sign-in and sign-out, and database writes. A controller or a background job cannot skip it: the hook sits on Captain Core records, before the row is saved.

## What is recorded

- Create, update, and delete from the admin workspace and the user portal
- Authentication events
- Database writes that pass through the record layer

Each event keeps enough context to see the actor, the action, and the before and after state. Tenant-owned changes carry the tenant they belong to (`subject_tenant_id`). Platform changes stay on the appliance trail and are not mixed into a tenant purge.

## Redaction

Passwords, SIP secrets, and API tokens are replaced with `[redacted]` before the event is stored. The trail is for operators, not a second copy of credentials.

## Who can read it

| Who | What they see |
| --- | --- |
| Tenant admin | That tenant’s audit only |
| Platform admin | The appliance trail, including platform events that are not tied to one tenant |

The browser does not choose the tenant by sending a tenant id in the body. The signed-in context does. Interactive API docs for the audit endpoints are on the appliance at `/api/docs/audit`.
