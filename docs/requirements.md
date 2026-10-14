# Time Trial App — Product and System Requirements

Version: 1.0  
Agreed requirements recorded: 14 October 2026  
Repository: debagged68/Time-Trial-App  
Status: requirements baseline for review; no production timing accuracy claimed.

## 1. Purpose and delivery

Build a shared race-timing application that replaces organisers' manual stopwatch workflow. Riders start individually from a pre-registered entry list. Different organisers record starts, lap/checkpoint crossings and finishes on separate devices. All authorised organisers share the same race and results.

Deliver a responsive web application first, designed for phones and outdoor use. Android is the target deployment platform for a subsequent release. Preserve the same timing, data and interaction rules across web and Android; the Android packaging approach remains to be chosen.

Primary product measures: simple operation, sub-second timing, a single button press at each timing instant, and easy use in cold conditions with fast riders.

## 2. Users and race lifecycle

- Race administrator: creates races, imports entries, assigns planned start order, predictions and handicaps, grants organiser access, reviews corrections and publishes results.
- Starter: starts riders or marks them DNS.
- Lap organiser: captures lap/checkpoint times and subsequently assigns riders.
- Finish organiser: captures finish times and subsequently assigns riders.
- Results organiser: reviews the dashboard and resolves timing issues.
- Riders: receive published results by email; a rider login is not required for the agreed scope.

Lifecycle: setup -> ready -> live -> results review -> completed/published. Publications are versioned independently so corrections can be republished.

Each device is assigned a race and station/role. Start, Lap and Finish MUST be separate screens. Switching roles must not discard captures or rider assignments.

## 3. Race setup and registration

A race stores name, date, timezone, start location and route name. The organiser configures timing stations and registers/imports riders before racing.

Riders are reusable people; rider entries are their participation in a particular race. Each entry stores race number, age on race day, predicted completion duration, planned start order and handicap duration.

The organiser can enter/edit planned start order, predicted duration and handicap before racing. Handicap input is minutes and seconds, with seconds restricted to 00–59. No handicap means 00:00. Predicted duration and handicap are distinct fields.

Rider numbers and planned start positions are unique within each race. A rider has at most one entry per race in the current scope. Multiple entries for heats/categories in one race would require a later scope decision.

## 4. Race-day wireframe and interaction requirements

The following screen specifications capture the reviewed wireframe behaviour. Example people and times are illustrative.

### 4.1 Start screen

Layout, from top to bottom:
1. Race name, role and connection/timing readiness.
2. Current rider: very large number, name, planned start position and predicted duration.
3. Large primary button: START #42.
4. Separate large secondary button: DNS — did not start.
5. Feedback for the last action.
6. Upcoming riders in planned order and a compact recorded-action history.

START captures the actual input time, assigns it to the current rider immediately, and automatically moves/scrolls the next waiting rider into the current position. No further selection or confirmation is required. Controls remain in consistent positions.

DNS records a did-not-start status with organiser/time history, creates no start capture, and advances to the next rider in the same way. A correction path can reverse an erroneous DNS or start without silently destroying history.

At the end of the list, show that all riders have been processed and disable start/DNS actions until an eligible entry is selected or restored. Actual start sequence is stored separately from planned order.

### 4.2 Lap screen

A dedicated Lap screen has a large LAP — CAPTURE TIME button. There is one lap checkpoint, crossed repeatedly on successive laps. Identify this device's station prominently; station configuration is outside the rapid capture path.

Each press saves a separate timestamp BEFORE rider selection. Multiple unassigned captures may accumulate. Show a numbered queue, oldest first, with captured clock time and assignment state.

Select a capture, then assign an eligible rider using large rider-number/name controls and quick number/name search. Assignment does not change the timestamp. Capture remains available while assigning an earlier capture.

### 4.3 Finish screen

A dedicated Finish screen follows the same capture-first queue workflow with a large FINISH — CAPTURE TIME button. There is no combined Lap/Finish mode selector on the timing screen.

An unassigned finish capture does not mark a rider finished. Assignment to the correct rider creates that rider's finish result. Exclude finished riders from ordinary new-assignment choices; explicit correction/reassignment can recover mistakes.

### 4.4 Shared capture and correction behaviour

- One press per passing rider; no rider lookup or confirmation at the timing instant for Lap or Finish.
- Preserve every distinct intentional press, including closely spaced captures and captures displaying the same tenth.
- Prevent duplicate captures caused by handling one physical interaction twice.
- Give immediate visible acknowledgement and optionally sound/vibration.
- Clearly show unassigned counts, recent captures and assignment feedback.
- Eligible assignment choices normally show started, unfinished riders. Unexpected or offline cases must be retained for organiser review rather than silently discarded.
- Permit reassignment and voiding accidental captures with an audit trail.
- Keep lists stable during selection; background updates must not move the intended tap target.
- Timing buttons must remain reachable throughout the capture/assignment workflow.

### 4.5 Dashboard

Use rows and columns, not rider cards as the main results presentation.

Provide these complementary table views:
- Race control (default): planned order, rider number/name, status, actual start time, lap count, finish clock time and elapsed duration.
- Results: actual position, rider, actual duration, finish clock time/crossing sequence, handicap, adjusted duration, adjusted position, prediction and prediction difference.
- Lap comparison: rider/status, each lap or split duration, total duration, with access to raw crossing timestamps.

Expose rider details including gender and race-day age where needed. Support sorting by planned/actual start order, finish clock time/crossing sequence, actual duration/position and adjusted duration/position. Keep waiting, DNS, DNF and withdrawn riders visible in appropriate operational views; do not rank them as finishers.

Portrait: compact table with rider number/name, status and elapsed time; tapping a row expands start, lap, finish, handicap and prediction details.

Landscape: wider table with additional detail columns. Adapt automatically to available width and orientation, including narrow landscape devices. Preserve selected view, sort, selected rider and expanded detail on rotation. Keep text and controls readable rather than shrinking the full table. Horizontal scrolling is acceptable for genuinely wide optional detail tables.

## 5. Timing and calculations

### 5.1 Required precision and accuracy

Display timestamps and durations to 0.1 seconds. Retain finer precision internally and calculate before display rounding.

Capture input time locally, before UI rendering, storage/network work or rider assignment. Server receipt time must never replace the event timestamp.

Use a monotonic high-resolution timebase for measurement and map it to a shared race clock. Device wall-clock changes, browser reloads, device sleep and clock drift require detection/revalidation.

Synchronise devices against a common clock, estimate offset and uncertainty, and show timing readiness. Preserve original timestamps and the synchronisation evidence associated with each capture. Do not silently rewrite historical captures when clock estimates change.

Target technical cross-device duration error: no more than 0.1 seconds under a defined, validated supported operating setup. Displaying tenths alone is not proof of this. Human reaction and touch hardware effects must be reported separately from software/clock measurement. If the target cannot be demonstrated, flag the limitation and consider external timing input rather than claiming compliance.

### 5.2 Results rules

Actual duration = finish timestamp - start timestamp.

Adjusted duration = actual duration - handicap duration.

Actual position ranks valid finishers by actual elapsed duration, fastest first.
Adjusted position ranks valid finishers by adjusted duration, fastest first.

Finish crossing sequence is chronological order across finish captures and is distinct from both elapsed-time rankings. It must remain associated with the original capture when a rider is assigned later.

Prediction difference = actual duration - predicted duration.

For each crossing:
- Split duration = crossing timestamp - previous valid crossing (or start).
- Cumulative duration = crossing timestamp - start.

All comparable results display to tenths. Equal displayed durations share a position; use competition ranking (1, 1, 3) as the baseline convention. Whether official race rules require a different tie policy must be confirmed before competition use. Do not break a displayed tie using hidden precision.

Negative durations, finishes before starts, duplicate valid starts/finishes and impossible crossing order are review errors. A handicap producing a negative adjusted duration is flagged for review; never silently clamp it.

Confirmed course structure: one lap checkpoint, crossed repeatedly on successive laps, with separate Start and Finish stations. Assign lap numbers by captured crossing timestamp, not assignment order.

## 6. Logical data model

All records have stable unique IDs. Event records carry creation time and responsible organiser/session where applicable.

| Entity | Principal fields and relationships |
| --- | --- |
| Race | id, name, race date, timezone, start location, route name, lifecycle status; optional source provider/event/list identifiers |
| Rider | id, name, gender, email, optional date of birth or recorded age with reference date; optional stable provider person ID |
| RiderEntry | id, race_id, rider_id, rider number, age on race day, predicted duration, planned start order, handicap duration, source entry ID/status, last imported time |
| Organiser | id, display name, email |
| RaceOrganiser | race_id, organiser_id, role/permissions; limits access to the assigned race |
| TimingStation | id, race_id, name, type Start/Lap/Finish, optional checkpoint sequence |
| DeviceSession | id, organiser_id, station_id, device/session identifier, readiness and clock synchronisation records |
| TimingCapture | id generated on device, race_id, station_id, session_id, event type, original timestamp/timebase, synchronised race timestamp, offset/uncertainty reference, capture sequence, nullable rider_entry_id, assignment/void state, server receipt time |
| RiderStatusEvent | id, rider_entry_id, DNS/DNS reversed/DNF/withdrawn event, reason, organiser/session, timestamp |
| CorrectionHistory | id, affected record, original value, revised value, reason, organiser, timestamp; retains original evidence |
| IntegrationConnection | id, provider, organiser/account scope, secure authorisation reference, connection status |
| ImportRun | id, race_id, source, organiser_id, timestamp, source reference, counts added/updated/rejected, validation issues |
| ResultsPublication | id, race_id, version, publisher, publication time, immutable results snapshot, CSV reference, publication status |
| EmailDelivery | id, publication_id, recipient reference/address snapshot, state, attempt information, sent time/provider reference, failure reason |

Timing captures and status events link to RiderEntry, not directly to Rider, so races remain separate.

Durations use integer units finer than a tenth, not formatted strings or floating-point minute values. Full event timestamps include date and UTC reference; display in race timezone. Age on race day is preserved as a race-specific snapshot.

Derived status, rankings, lap/split durations, actual start order and finish sequence can be materialised for performance but must be reproducible from authoritative records and versioned rules.

Constraints: unique (race, rider); unique (race, rider number); unique planned start position when assigned; at most one valid start and finish per entry; captures must belong to the same race as their assigned entry. Capture IDs and upload handling must be idempotent.

## 7. Entry ingestion

### 7.1 RiderHQ integration

The organiser initiates import within the app:
Connect RiderHQ -> authorise supported access -> select event and entry list/category -> preview and validate -> import.

Required capabilities: authorised event selection, entrant retrieval, stable source identifiers where available, repeat import, and handling of changes, transfers and withdrawals.

Implementation dependency: RiderHQ's published API link returned 404 during research on 14 October 2026. Authentication, endpoints, scopes, fields, rate limits, pagination, webhook support, fees and integration permission remain unverified. Do not assume OAuth, an API key or any endpoint until RiderHQ supplies its supported contract.

Use a backend connector for provider credentials and API access. Import only fields needed by this application; credentials must not enter public source or client bundles. Revoke/expire connections appropriately.

RiderHQ entry IDs identify entries, not necessarily reusable people. Do not merge riders by name alone; unresolved matches need review.

### 7.2 CSV import

Support organiser upload in the app with column mapping, preview, validation and explicit import.

| CSV field | Rule |
| --- | --- |
| Rider name | Required |
| Rider number | Required; unique in race |
| Gender | Optional |
| Age or date of birth | Optional; validate and calculate/snapshot race-day age where possible |
| Rider email | Supported; validate; missing addresses shown before publishing |
| Predicted duration | Optional; editable later |
| Planned start order | Optional; assign/edit later |
| Handicap | Optional minutes:seconds; defaults to zero |
| External rider/entry ID | Optional; supports repeat matching |

Provide a downloadable template and clear expected time formats. Validate headers, encoding, empty rows, required values, duplicate identifiers/numbers/orders, invalid ages/emails and invalid durations. Show row-level errors and counts before changes are applied. Do not silently drop bad rows or merge ambiguous riders.

### 7.3 Repeat import and source ownership

Allow add-new or update-matching modes with a preview of changes. Match by reliable scoped source entry ID or explicit reviewed matching rules; rider number alone is not a permanent cross-race identity.

Preserve timing captures, assignments, DNS/status history, and organiser-entered start order, handicap and prediction unless a specific permitted field replacement is explicitly selected. Source withdrawals/transfers require review; do not delete a timed entry automatically.

Store import provenance and counts. Network or parsing failure must not leave an unexplained partially imported roster. Preload the registered list onto timing devices before racing.

## 8. Results export, publication and email

Provide a simple Publish results & email action from results review. This is explicit organiser authorisation to publish that version and send to the displayed race recipients; no separate manual per-recipient send is required.

Before enabling final publication, show recipient counts, missing/invalid addresses, unassigned captures and unresolved timing issues. Unresolved timing errors require resolution or an explicit audited exclusion decision. Warn about still-racing riders.

On publication:
1. Create an immutable, versioned results snapshot.
2. Generate CSV from that snapshot.
3. Queue individual emails with CSV attachment for the race's riders and authorised organiser recipients.
4. Show queued/sent/failed delivery state and allow failed-only retry.

Keep recipient addresses private. Deduplicate identical addresses within a publication. Exclude emails and unrelated personal/contact details from the results CSV.

CSV includes race name/date/start location/route, rider number/name, gender and race-day age, rider status, planned and actual start order, actual start timestamp, each lap/checkpoint timestamp and split, finish timestamp, finish crossing sequence, actual duration/position, predicted duration/difference, handicap duration, adjusted duration/position. Use consistent headers, explicit timezone/units and a documented representation for a variable number of laps.

Missing-email riders still remain in the results. Publication must clearly report recipients skipped due to missing/invalid addresses. Email queued/sent status must not be described as proof of inbox delivery.

Retries must not resend successful deliveries. Corrections after publishing create a new version using Publish revised results and notify the selected recipients explicitly. An automated email service is sufficient; an AI agent is not required for delivery.

## 9. Shared operation, offline resilience and security

- Propagate starts, assignments, finishes and corrections to authorised devices in the same race.
- Save captures durably on the device before relying on network success; queue offline uploads.
- After reconnection, reconcile idempotently without losing or duplicating captures.
- Distinguish local pending, synchronised and conflicted records visibly.
- Offline devices cannot see new remote starts immediately; expose this limitation and retain unexpected captures for later assignment/review.
- Resolve concurrent assignment/correction conflicts explicitly; never silently overwrite a different organiser's work.
- Detect another starter attempting to start the same entry.
- Protect race access and organiser administration; only authorised roles import, correct or publish.
- Treat rider emails and any birth dates as private. Do not publish them in dashboards intended for wider viewing or exports.
- Keep secrets server-side, including email provider credentials.
- Preserve timing/audit history and define backup, retention and recovery policies before production use.

## 10. Acceptance measures

These are proposed measurable release gates for the agreed experience; validate on actual supported phones and browsers.

| Measure | Acceptance condition |
| --- | --- |
| Single-button timing | Start, Lap and Finish each capture with exactly one press; Lap/Finish require no rider selection first and no confirmation dialog |
| Start progression | START or DNS automatically makes the next waiting rider current, with no extra tap; DNS creates no start timestamp |
| Visible acknowledgement | Target acknowledgement within 100 ms at the 95th percentile under the documented supported workload; timestamp capture occurs before acknowledgement |
| Sub-second output | All relevant times display to 0.1 s; precise inputs retained and calculations performed before rounding |
| Cross-device accuracy | Controlled common-reference tests demonstrate <=0.1 s technical duration error under the supported setup; record clock uncertainty and separately assess manual input effects |
| Rapid captures | Test distinct captures 0.1 s apart on supported devices with no dropped or duplicate events, including while assigning earlier captures |
| Capture durability | Once acknowledged as durably saved, a capture survives reload/restart and eventual reconnection; no duplicate event on retry |
| Shared visibility | Target updates visible on other connected devices within 1 s at the 95th percentile under documented network/load conditions |
| Large controls | Primary timing area at least 88 CSS px tall; other race-day tap targets at least 48 by 48 CSS px with separation |
| Cold-condition usability | Field-test with intended volunteers, gloves compatible with the selected device and fast rider arrivals; no compulsory typing at the timing instant |
| Simplicity | A volunteer can complete start/DNS, repeated captures and assignments after a short briefing; assess with representative users before release |
| Orientation | Portrait/landscape work without clipped primary controls, unreadable text or lost view/selection state |
| Import | Preview catches duplicates/invalid data; repeat import does not duplicate entries or overwrite timing/history |
| Ranking | Verified examples show actual and handicap-adjusted rankings independently, including DNS, ties and corrections |
| Publish | One final publish action creates versioned CSV and queues private individual emails; retries avoid successful recipients |

Timing validation must include different devices, clock drift, delayed/lost network, backgrounding/sleep, reloads and high input activity. If readiness/uncertainty is outside the validated range, show a clear degraded state and retain the evidence; do not silently advertise compliant timing.

## 11. Delivery sequence and open decisions

1. Approve requirements and revised screen flows.
2. Build responsive web app with sample races, separate screens and CSV ingestion.
3. Add durable shared backend, organiser access, clock synchronisation and offline reconciliation; validate against timing gates.
4. Add result review, actual/adjusted tables, CSV publication and automated email.
5. Implement RiderHQ connector when its supported API contract/access is available.
6. Field-test a full race, then package/adapt for Android and repeat platform-specific timing tests.

Open decisions: RiderHQ access contract; supported device/browser matrix; email sender/provider; Android delivery approach; production scale limits; retention policy; final tie convention under applicable race rules. A successful wireframe interaction is not proof of timing accuracy or multi-device reliability.

## 12. Research references

Checked 14 October 2026:
- RiderHQ API reference (linked by RiderHQ; returned 404): https://www.riderhq.com/api/reference
- RiderHQ CSV export guide: https://www.riderhq.com/help/p/ubx1p7856hntis2/5-minute-guide-to-accepting-online-entries-for-your-event-with-RiderHQ
- RiderHQ shared custom reports: https://www.riderhq.com/help/p/uhbevbkojz2ksry/Share-entrant-information-with-ride-leaders
- RiderHQ help/integrations: https://www.riderhq.com/help
- High Resolution Time specification: https://www.w3.org/TR/hr-time-3/

Research establishes available documentation, not a confirmed API integration agreement.
