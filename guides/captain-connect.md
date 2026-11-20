# Captain Connect

**Integrations** menu. Inside an organization.

Connect your own software to live calls: call events, click-to-call, caller lookup, screen pop, and live call audio for transcription or a voice assistant. To set up users, extensions or routes from another system, use the [System API](api-tokens.md) instead.

## Do this

1. Select the organization.
2. Open **Integrations → Captain Connect**.
3. On **Apps**, choose **Connect an app**. Pick what it is for, check what it may do, and enter its webhook address if it should receive events.
4. Copy the key and the signing secret for the app's developer. They are shown once.
5. To let the app take part in calls, open **Call flows** and add a step: ask an app, look up the caller, stream call audio, or let an app talk with the caller. Then choose that step as a destination on an inbound number, menu or queue.
6. For screen pop, open **Settings** and enter the address of your CRM or helpdesk.
7. Watch **Activity** to see every event sent and what the app answered. Use **Send again** after you fix a problem.

Every field has help text, and each call-flow step has a **How it works** section. The **API reference** tab has a quick start, examples, how to check a webhook signature, the live-audio frames, and the full OpenAPI file.

## Good to know

- Webhooks are signed (`webhook-id`, `webhook-timestamp`, `webhook-signature`). The app answers 2xx within 5 seconds. Failed deliveries are retried for about two days. An app that fails for three days is turned off with an alert.
- Every call-flow step has a next destination, so callers never hit silence when the app is slow or down.
- Live audio goes through a media bridge on the appliance to the app's `wss://` address as JSON frames: base64 μ-law, 8 kHz, 20 ms per frame.

## You are done when

**Send a test event** reaches the app, **Activity** shows events as delivered, and a call routed to a Connect step does what the app said.
