# System API

**Integrations** menu. Inside an organization.

Use it when another system should set up or read this phone system for you: a provisioning tool, a billing system or a script. For live call events, click-to-call or live call audio, use [Captain Connect](captain-connect.md) instead.

Create a token only for a system you trust. If it leaks, revoke it. Revoking it does not affect phone calls.

## Do this

1. Select the organization.
2. Open **Integrations → System API**.
3. On **Tokens**, choose **Generate new token**. Give it a name and pick a preset, such as **Provisioning**. Turn on only what the system needs.
4. Copy the refresh token and the access token now. They are shown once.
5. Open **How to connect** for the steps and copy-ready examples, and **API reference** for every request, by area.
6. Revoke a token you no longer use.

## Good to know

- Send `Authorization: Bearer <access token>` on every request. The access token lasts one hour.
- To get a new one, POST the refresh token to `/api/v1/auth/token/refresh`. The answer has a new refresh token too. Save it, because the old one stops working.
- Many changes reach phones only after they are applied. Answers say `needs_apply: true`. Apply with `POST /api/v1/asterisk/apply` (the token needs Admin on **Apply dialplan / PJSIP**) or with the header button.
- A token only sees the organization it was made in. Make one token per connected system.

## You are done when

The system can call the API with its token, and a revoked token cannot.
