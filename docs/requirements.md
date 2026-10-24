# Time Trial App — Product and System Requirements

Version: 1.1 — review draft  
Original baseline: 14 October 2026  
Last updated: 24 October 2026  
Repository: debagged68/Time-Trial-App  
Status: awaiting organiser review of the additions below. Further implementation and deployment are paused until that review. No production timing accuracy is claimed.

## Version 1.1 changes for review

The new requirements are consolidated in section 13, with data model changes in section 6. They cover total distance, optional elevation, named lap-point distances, metric/imperial settings, lap/total average speed, and optional recorded audio/spoken rider-number assignment with manual fallback. Proposed implementation choices are explicitly distinguished from user requirements.

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

A race stores name, date, timezone, start location, route name, total distance and optional total elevation gain (interpretation for review). The organiser configures timing stations and registers/imports riders before racing.

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

After capture, optionally speak the rider number for automatic assignment. If nothing is spoken, recognition is unclear, or voice is unavailable, select the rider from the next/active-rider list. For queued manual assignment, select the relevant capture then the eligible rider using large rider-number/name controls and quick number/name search. Assignment does not change the timestamp. Capture remains available while assigning an earlier capture.

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
- Results: actual position, rider, actual duration, finish clock time/crossing sequence, handicap, adjusted duration, adjusted position, prediction, prediction difference and total average speed.
- Lap comparison: rider/status, each lap or split distance, duration and average speed, plus total duration and total average speed, with access to raw crossing timestamps.

Expose rider details including gender and race-day age where needed. Support sorting by planned/actual start order, finish clock time/crossing sequence, actual duration/position and adjusted duration/position. Keep waiting, DNS, DNF and withdrawn riders visible in appropriate operational views; do not rank them as finishers.

Portrait: compact table with rider number/name, status and elapsed time; tapping a row expands start, lap distances/durations/speeds, finish, overall speed, handicap and prediction details.

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
| Race | id, name, race date, timezone, start location, route name, total_distance_m, optional total_elevation_gain_m, circuit_distance_m, lifecycle status; optional source provider/event/list identifiers |
| Rider | id, name, gender, email, optional date of birth or recorded age with reference date; optional stable provider person ID |
| RiderEntry | id, race_id, rider_id, rider number, age on race day, predicted duration, planned start order, handicap duration, source entry ID/status, last imported time |
| Organiser | id, display name, email |
| RaceOrganiser | race_id, organiser_id, role/permissions; limits access to the assigned race |
| RaceLapPoint | id, race_id, name, distance_from_start_m for first crossing; one repeated physical lap checkpoint in current scope |
| TimingStation | id, race_id, name, type Start/Lap/Finish, optional lap_point_id; devices reference the same physical point |
| UnitPreference | organiser and/or device reference, metric/imperial selection; storage scope proposed for review |
| CaptureAudio | id, timing_capture_id, race/station/session reference, recording start/end, MIME type, audio reference/local device reference, availability and retention status |
| AssignmentEvidence | capture_id, source manual/voice, optional recognised text/confidence, recognition outcome, assigned entry, organiser and timestamp |
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

Derived status, rankings, lap/split distances and durations, lap and total average speeds, actual start order and finish sequence can be materialised for performance but must be reproducible from authoritative records and versioned rules.

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

CSV includes race name/date/start location/route, rider number/name, gender and race-day age, rider status, planned and actual start order, actual start timestamp, each lap/checkpoint timestamp and split, finish timestamp, finish crossing sequence, actual duration/position, predicted duration/difference, handicap duration, adjusted duration/position. Also include total distance, optional elevation, checkpoint name/distance, circuit distance, lap distances/speeds and overall average speed. Use consistent headers, explicit timezone/units and a documented representation for a variable number of laps. Audio and transcripts are excluded from rider-facing results.

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

Open decisions: course/voice implementation choices in section 13; RiderHQ access contract; supported device/browser matrix; email sender/provider; Android delivery approach; production scale limits; retention policy; final tie convention under applicable race rules. A successful wireframe interaction is not proof of timing accuracy or multi-device reliability.

## 12. Research references

Checked 14 October 2026:
- RiderHQ API reference (linked by RiderHQ; returned 404): https://www.riderhq.com/api/reference
- RiderHQ CSV export guide: https://www.riderhq.com/help/p/ubx1p7856hntis2/5-minute-guide-to-accepting-online-entries-for-your-event-with-RiderHQ
- RiderHQ shared custom reports: https://www.riderhq.com/help/p/uhbevbkojz2ksry/Share-entrant-information-with-ride-leaders
- RiderHQ help/integrations: https://www.riderhq.com/help
- High Resolution Time specification: https://www.w3.org/TR/hr-time-3/

Research establishes available documentation, not a confirmed API integration agreement.

## 13. Course, units, speed and voice requirements — version 1.1 review

These additions extend the earlier requirements. They describe intended behaviour, not features verified in the deployed test app. The organiser has requested review BEFORE further app work.

### 13.1 Course data

**User requirements**
- Race data includes total race distance and optionally total elevation.
- Lap points have names and distances from the start.
- Current race format remains ONE physical lap checkpoint, crossed repeatedly on successive laps, with separate Start and Finish stations.
- Course information must be editable within race setup, retained with the race and usable for automatic dashboard speed calculations.

**Proposed modelling details for review**
- Interpret total elevation as total elevation gain/ascent, rather than altitude above sea level. Blank means unknown/not supplied; zero means known zero ascent.
- Store authoritative distances and elevation in metres, independently of display units.
- Store first-pass distance from the start on the named lap point.
- Also store circuit distance between repeated passes. This is needed to calculate later crossing distances without assuming every lap begins at the start line.
- Cumulative distance at crossing n = first-pass distance + (n - 1) × circuit distance.
- Remaining distance to finish = total race distance - cumulative distance at the final recorded lap.
- Example: first pass 4 km, circuit 5 km, total 16 km. Passes are at 4, 9 and 14 km; final section to finish is 2 km.
- Validate positive total/circuit distances, nonblank point name, first-pass distance between start and finish, and optional nonnegative elevation gain.
- Flag crossings beyond the configured total distance. Preserve their timing captures; show unavailable/invalid distances and speeds instead of inventing values.
- Audit distance changes and recalculate provisional speeds; never alter original timestamps or silently change a published snapshot.

### 13.2 Metric and imperial settings

**User requirements**
- Settings allow Metric or Imperial selection.
- Dashboard and setup display the selected distance and speed units consistently.

**Proposed presentation and storage rules**
- Metric: kilometres, km/h and metres of elevation gain.
- Imperial: miles, mph and feet of elevation gain.
- Input forms accept the selected units; values and exports identify their units.
- Switching units converts presentation only, without changing measured times or authoritative SI distances.
- Use exact conversion constants: 1 mile = 1,609.344 metres; 1 foot = 0.3048 metres.
- Retain the preference across reloads. Whether it belongs to the organiser account or individual device remains for review; device-local fallback is acceptable.
- Unit changes and portrait/landscape rotation preserve queue, selected rider and dashboard sort state.

### 13.3 Automatic lap and total speed

**User requirements**
- Dashboard calculates average speed for each lap and overall, using duration and distance.
- Show units and update calculated values automatically when relevant race data or valid timing assignments/corrections change.

**Calculation rules**
- Use ACTUAL elapsed durations, never handicap-adjusted durations, for physical speed.
- First section: distance from start to first checkpoint / duration from start to first crossing.
- Subsequent lap: distance between consecutive passes / duration between their captured timestamps.
- Final section: remaining distance to finish / duration from last lap crossing to finish.
- Total average speed: total race distance / actual finish duration.
- If there are no lap captures, start-to-finish is the only measured section.
- Optional cumulative speed: cumulative distance from start / cumulative elapsed duration.
- With stored milliseconds: km/h = distance_metres × 3,600 / duration_milliseconds; mph = km/h / 1.609344.
- Example: 5 km in 10 minutes = 30.0 km/h, approximately 18.6 mph.
- Calculate at full stored precision; proposed display is one decimal place.
- Sequence crossings by CAPTURED timestamp, not the order in which riders were assigned.
- Missing distance, nonpositive duration or impossible crossing distance displays — with an appropriate review warning, never infinity, NaN or a guessed speed.
- Full-course total speed remains blank for unfinished riders; use measured split/cumulative speeds where available.
- Label first partial section, repeated circuit laps and final finish section distinctly.
- Show overall speed in the results table and lap distance/duration/speed in rider details or lap-comparison view. Portrait remains readable, with expanded detail if necessary; landscape exposes more columns.

### 13.4 Sound recording and voice assignment

**User requirement: capture first, speak second**
1. Organiser presses LAP or FINISH; the app immediately records and saves that crossing timestamp.
2. The app records sound/listens for a rider number for that SPECIFIC capture.
3. The organiser can say the number, for example “42”, “forty two” or “rider number four two”.
4. A clear, unique, eligible number is assigned automatically without another confirmation press.
5. If nothing is said or the number cannot be interpreted, show the next/active-rider list for manual rider selection.
6. Whether assigned by voice or manually, the original captured time is unchanged.

**Required fallback and rapid-capture behaviour**
- Voice must never delay timestamp capture or block another LAP/FINISH press.
- Allow multiple pending captures while recognising or assigning earlier ones.
- Bind recording and recognition request to immutable capture ID, race and station.
- Never associate a late recognition response with the newest capture just because it arrived last.
- If overlapping speech cannot be associated reliably, retain the affected capture(s) for manual assignment.
- Do not overwrite a manual assignment or act on a voided capture, another race or a stale station session.
- Only auto-assign a reliably recognised number that uniquely matches a started, unfinished rider in that race.
- Silence, ambiguity, low confidence, unknown/ineligible number, duplicate result, microphone denial or speech-service failure leaves the capture available for manual selection.
- Do not guess from arrival order or list position.
- The next/active-rider list shows large numbers and names, stays stable during selection and supports number search.
- Display the recognised rider and capture time clearly; allow correction through the normal audited reassignment controls.
- Assignment evidence records manual/voice source and any recognised text/confidence supplied by the engine.

**Proposed audio behaviour for review**
- Voice/audio is optional and explicitly enabled before use, with microphone permission.
- Record a short clip after each capture; proposed maximum is five seconds. No undisclosed continuous ambient recording.
- Show recording/listening status and a clear off/stop control.
- Permit replay of a clip to resolve an unclear rider number.
- Proposed first release stores clips only on the recording device, linked to capture ID. Shared audio storage, backup, retention and deletion policy remain to be decided.
- Disclose whether recognition occurs on-device or uses an external browser/vendor service before enabling it.
- Browser speech support and offline recognition are not guaranteed. Unsupported/offline speech falls back to manual selection; recording/replay can work independently when supported.
- Raw audio and transcripts must not be included in rider results CSVs or emails.

### 13.5 Additional acceptance criteria

| Measure | Required evidence |
| --- | --- |
| Course validation | Invalid total/first-pass/circuit distances rejected; optional elevation distinguishes blank from zero; original timing evidence retained |
| Unit conversions | Known metric/imperial cases match; repeated switches preserve authoritative values and timing/queue state |
| Speeds | Tests cover first partial section, repeated laps, final section, total and correction-driven recalculation; handicap has no influence |
| Unavailable speeds | No fabricated values, infinity or NaN from missing distances, nonpositive durations or beyond-course crossings |
| Voice success | One timing press followed by a clear eligible number assigns the correct capture without a confirmation tap |
| Voice fallback | Silence, ambiguity, unknown/ineligible number, unsupported browser, denied microphone and speech/network failure expose manual selection without losing the timestamp |
| Voice concurrency | Rapid presses and late callbacks do not attach speech to the wrong capture or override manual corrections |
| Audio association | Replay identifies its original capture and clearly states local/shared availability |
| Outdoor usability | Test wind, traffic, accents, gloves and closely spaced arrivals on supported devices before race use |
| Timing independence | Enabling audio or voice does not change the timing acceptance targets in section 10 |

### 13.6 Review decisions and implementation status

Please review these proposed details: elevation means total ascent; explicit circuit length; per-organiser versus per-device units; one-decimal speed display; five-second voice window; audio storage/retention and recognition provider.

The selected visual direction remains Cycling Computer: dark surfaces, high-contrast timing numerals and large minimal controls. Delivery remains web first, Android later.

This document is the review draft. Further app implementation and deployment are paused until the organiser completes the requested review.

## 14. Revision history

| Version | Date | Change |
| --- | --- | --- |
| 1.0 | 14 October 2026 | Initial timing, screens, model, ingestion, handicap and publication requirements; repeated lap checkpoint subsequently clarified |
| 1.1 review draft | 24 October 2026 | Adds course distances/elevation, named point distance, metric/imperial units, lap/total speed and optional audio/voice assignment with manual fallback |
