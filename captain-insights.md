# Captain Insights

Captain Insights is the reporting surface for a tenant. It reads stored call facts, IVR path events, and policy events. It does not sit on the live media path, so running a report does not take a channel off a call.

Reports use the signed-in tenant. A tenant id from the browser is not accepted as the scope.

## Queue reports

Queue reports open on categories, then a report. Filters (range, queue, service-level target) apply when you run the report.

| Category | What it covers |
| --- | --- |
| Operations | Performance, abandoned calls, ring and offer, busy and overflow, period peaks |
| Call detail | Answered, unanswered, and all visits, plus call length |
| Service level | Service level, average speed of answer, breaches, trend |
| Distribution | By hour, by weekday, and hour by weekday |
| Agents | Activity, scorecard, sessions, pauses, missed rings, agent by queue |
| Management | Executive, queue compare, period compare, productivity |
| Quality | Handle-time outliers |

Short hangups (abandoned under 5 seconds by default) are counted on their own so they do not distort service level. Service level is answered within the target, divided by offered calls minus those short hangups.

Export to CSV, JSON, or PDF uses the same window and filters as the screen. Permission: `queuereports.read`.

## IVR

IVR reports use path events written when a caller moves through a menu, not the channel event log alone. The page covers performance, menu digits, funnel drop-off, timeouts, paths, timing, exits, and sessions. A call in Call analytics can open the matching IVR path. Permission: `ivr.reports`.

## Calls, conferences, and policy

- **Call analytics** summarizes CDR for the tenant: volume, disposition, and top called numbers, including media-quality fields when the call reported them.
- **Conferences** report room sessions.
- **Calling policy usage** lists blocked outside and inside calls, pattern blocks, PIN checks, and account-code events, with the same kind of date presets as IVR reports, and a CSV export.

## API

Queue, IVR, and CDR report endpoints live under `/api/v1/`. The appliance publishes the contract at `/api/docs`.
