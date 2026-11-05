# CaptainPBX guides

These pages follow the admin menu from top to bottom. They are for the person configuring the system.

The menu changes when you select an organization. **System view** is the whole appliance. **Inside an organization** is one phone system on that appliance.

## Home

See whether the appliance and the phones are healthy before you change anything.

- [Captain Watch](watch.md) — A live view of this CaptainPBX. In the system view it is the whole appliance. After you select an organization, the same menu item is named Watch and shows that organization.
- [EndpointWatch](endpoint-watch.md) — Which phones are registered, and alerts when a phone drops offline.
## Voice

The numbers people dial, and what happens on the call.

- [Extensions](extensions.md) — The phone numbers for this organization. Each number can have a person, a SIP phone, and voicemail.
- [Business Hours](business-hours.md) — The weekly open and closed times. Inbound routes and menus use this to send calls a different way after hours.
- [Ring groups](ring-groups.md) — One number that rings several extensions together, or one after another.
- [IVR](ivr.md) — A recorded menu. The caller presses a key and the call goes to an extension, a group, a queue, or another menu.
- [Parking](parking.md) — Put a live call in a shared slot, then pick it up from any phone by dialing that slot.
- [Paging](paging.md) — A one-way announcement to a group of phones. The called phones answer on speaker.
- [Conferences](conferences.md) — A room number. Everyone who dials it joins the same call.
- [Calling Policy](calling-policy.md) — Rules for which destinations an extension is allowed to dial.
- [Feature Codes](feature-codes.md) — Short codes dialed from a phone, such as pickup, park, or voicemail.
## Audio

The recordings callers hear, and the music they hear while they wait.

- [Sounds](sounds.md) — Recorded prompts and announcements. IVR menus and inbound routes play these.
- [Music on Hold](music-on-hold.md) — What a caller hears while they wait, on hold, or in a queue.
## Media

Voicemail messages and recordings of calls.

- [Voicemail](voicemail.md) — Greetings and messages for the extensions in this organization.
- [Call recordings](call-recordings.md) — Listen to calls that this organization recorded.
## Queue

Callers wait here until a person is free to answer.

- [Queue](queues.md) — Callers wait in line until an agent is free. Agents are extensions that join the queue.
## Routes

How calls come in from outside, and how people dial out.

- [Trunks](trunks.md) — The connection to your SIP provider. Outside calls use a trunk. Inside calls do not.
- [Inbound](inbound.md) — What happens when someone calls your outside number.
- [Outbound](outbound.md) — How extensions dial the outside world, and which trunk carries the call.
## People

Who can sign in, which team they belong to, and the directory.

- [People](people.md) — The logins. A person can be an administrator, a phone user, or both. A person is not an extension, though an extension often has one.
- [Teams](teams.md) — A named set of people, such as a department. A team is not a permission.
- [Contacts](contacts.md) — The directory of names and numbers people can look up from a phone or the portal.
## Captain AI

Transcription activity for the organization you have open.

- [Activity](captain-ai-activity.md) — Transcription and other Captain AI work for this organization.
## Insights

Reports you can read, filter, and export.

- [Call Analytics](call-analytics.md) — Reports of calls: who called, who answered, and how long it lasted.
- [IVR Analytics](ivr-analytics.md) — Which menu keys callers press.
- [Queue Analytics](queue-analytics.md) — How long callers waited, and how the queue's agents answered.
- [Conference Analytics](conference-analytics.md) — Who joined conference rooms, and for how long.
- [Calling Policy Analytics](calling-policy-analytics.md) — Calls that a calling policy allowed or blocked.
- [Logs](logs.md) — Messages the appliance recorded. Use this when something failed and you need the time and the text.
## Platform

Settings for the whole appliance: organizations, sign-in, and SIP.

- [Organizations](organizations.md) — Each organization is its own phone system on this appliance: its own people, numbers, and SIP domain.
- [Roles & Permissions](roles.md) — What each role is allowed to open and change.
- [Global SIP Settings](global-sip.md) — SIP settings for the whole appliance, shared by every organization.
- [AMI / ARI users](ami-ari.md) — Usernames for other software that connects to the call engine.
- [Browser phone](browser-phone.md) — Place and answer calls from a browser, without a desk phone.
- [Captain Auth](captain-auth.md) — How people sign in on the web, including a second factor.
- [Settings](settings.md) — Appliance-wide preferences, such as the name and time defaults.
- [API tokens](api-tokens.md) — A key a script can use to talk to CaptainPBX for this organization. Treat it like a password.
## System

Security, backups, certificates, and updates for this machine.

- [Captain Shield](captain-shield.md) — Who may reach this appliance: the admin website, SSH, and SIP phones.
- [Captain Audit](captain-audit.md) — A history of who changed what. Passwords are not written in the history.
- [Captain Vault](captain-vault.md) — Encrypted backups of this appliance.
- [Captain Bulk](captain-bulk.md) — Add many extensions or people from one list, instead of one at a time.
- [Captain AI](captain-ai.md) — Turn transcription on for the appliance, and choose how it is allowed to run.
- [Captain Bridge](captain-bridge.md) — Tasks that belong to the machine itself, such as its network address.
- [Captain Trust](captain-trust.md) — The HTTPS certificate the browser checks when people open CaptainPBX.
- [Captain Update Center](update-center.md) — See what is installed, and apply updates.
- [Maintenance](maintenance.md) — Restart a stuck service, or run a maintenance task, when the phone system needs a nudge.

Phones and SIP domains: [Register a SIP phone](../sip-registration.md).

