# Time Trial App — Product and System Requirements

Version: 1.17 — reusable riders and protected setup through Beta 0.10  
Original baseline: 14 October 2026  
Last updated: 31 October 2026  
Repository: debagged68/Time-Trial-App  
Status: baseline requirements approved by the organiser. Mobile beta feedback below is approved for implementation; no production timing accuracy is claimed.

Later approved sections supersede conflicting earlier descriptions. Sections 22–25 define the current course/race separation, start-list controls, corrections, speech batches and Beta 0.6.1 speech handling. Requirements describe the intended product; implementation limitations remain explicitly identified below.

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

A race stores name, date, timezone, scheduled start time, gap between rider starts (default 01:00), start location, route name, total distance and optional total elevation gain (interpretation for review). The organiser configures timing stations and registers/imports riders before racing.

Riders are reusable people; rider entries are their participation in a particular race. Each entry stores race number, mandatory age on race day, mandatory category (OPEN or FEMALE), predicted completion duration, planned start order and handicap duration.

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

A dedicated Checkpoint (Lap) screen has a large LAP — CAPTURE TIME button. A race has zero or more configured required checkpoints. The screen is hidden for Start-and-Finish-only races. Identify the checkpoint being timed prominently; station configuration is outside the rapid capture path. Repeated visits to the same physical location are separate configured checkpoints.

Each press saves a separate timestamp BEFORE rider selection. Multiple unassigned captures may accumulate. Show unassigned recording rows newest first, each with captured clock time and recognised text. For manual batch assignment, retain oldest-first selection and automatic advance.

With Audio ON, capture listens for up to five seconds. A recognised eligible number creates a suggestion; the organiser must press Confirm to assign it. If nothing is spoken, recognition is unclear, or voice is unavailable, select the rider from the next/active-rider list. For queued manual assignment, select the relevant capture then the eligible rider using large rider-number/name controls. Assignment does not change the timestamp. Capture remains available while assigning an earlier capture.

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

Show mandatory age on race day and category (OPEN or FEMALE) on the rider-list screen and in relevant dashboard details. Support sorting by planned/actual start order, finish clock time/crossing sequence, actual duration/position and adjusted duration/position. Keep waiting, DNS, DNF and withdrawn riders visible in appropriate operational views; do not rank them as finishers.

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

Confirmed course options: Start and Finish only, or an ordered list of required checkpoints between them. Enter repeated circuit visits as separate checkpoints with cumulative distances from the start. Match captures to checkpoint IDs and captured timestamps, never to the order of rider assignment.

## 6. Logical data model

All records have stable unique IDs. Event records carry creation time and responsible organiser/session where applicable.

| Entity | Principal fields and relationships |
| --- | --- |
| Race | id, name, race date, timezone, scheduled start time, start gap duration (default 60000 ms), start location, route name, total_distance_m, optional total_elevation_gain_m, required_checkpoint_count (derived from configured checkpoint rows), lifecycle status; optional source provider/event/list identifiers |
| Rider | id, name, email, optional date of birth or recorded age with reference date; optional stable provider person ID; no sex/gender field |
| RiderEntry | id, race_id, rider_id, rider number, mandatory age on race day, mandatory category (OPEN or FEMALE), predicted duration, planned start order, scheduled start timestamp, handicap duration, source entry ID/status, last imported time |
| Organiser | id, display name, email |
| RaceOrganiser | race_id, organiser_id, role/permissions; limits access to the assigned race |
| RaceCheckpoint | id, race_id, sequence, name, distance_from_start_m (cumulative course distance); zero or more required checkpoint rows, including separate rows for repeated visits to one physical location |
| TimingStation | id, race_id, name, type Start/Lap/Finish, checkpoint_id or checkpoint references; physical station/device is distinct from a required course crossing |
| UnitPreference | organiser and/or device reference, metric/imperial selection; storage scope proposed for review |
| CaptureAudio | id, timing_capture_id, race/station/session reference, recording start/end, MIME type, audio reference/local device reference, availability and retention status |
| AssignmentEvidence | capture_id, source manual/voice, optional recognised text/confidence, recognition outcome, assigned entry, organiser and timestamp |
| DeviceSession | id, organiser_id, station_id, device/session identifier, readiness and clock synchronisation records |
| TimingCapture | id generated on device, race_id, station_id, session_id, event type, original timestamp/timebase, synchronised race timestamp, offset/uncertainty reference, capture sequence, nullable rider_entry_id, assignment/void state, server receipt time, checkpoint_id for checkpoint captures |
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
| Category | Required; exactly OPEN or FEMALE; replaces sex/gender |
| Age or date of birth | Required input sufficient to establish age on race day; validate and preserve mandatory race-day age |
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

CSV includes race name/date/start location/route, rider number/name, category and race-day age, rider status, planned and actual start order, actual start timestamp, each lap/checkpoint timestamp and split, finish timestamp, finish crossing sequence, actual duration/position, predicted duration/difference, handicap duration, adjusted duration/position. Also include total distance, optional elevation, each checkpoint ID/sequence/name/cumulative distance, section distances/speeds and overall average speed. Use consistent headers, explicit timezone/units and a documented representation for a variable number of laps. Audio and transcripts are excluded from rider-facing results.

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

### 13.1 Course data and required checkpoint list

- Race data includes total distance, optional total elevation gain/ascent, and **Required checkpoints**.
- Set Required checkpoints to zero for a Start-and-Finish-only course.
- Setting N required checkpoints provides N ordered rows, each containing checkpoint name and cumulative distance from the start. Add/remove controls may update the number; the saved count must always match the rows.
- Start (0 distance) and Finish (total distance) are separate endpoints and do not count as checkpoints.
- Multiple checkpoints may be at different locations, or represent successive visits to the same physical circuit location.
- For repeated laps, explicitly enter each crossing as a checkpoint. No separate required-pass count, circuit length or automatic lap-distance inference is needed.
- Example for a 16 km course: checkpoint 1 “Bridge — lap 1” at 4 km; checkpoint 2 “Bridge — lap 2” at 9 km; checkpoint 3 “Bridge — lap 3” at 14 km; Finish at 16 km.
- Distances are cumulative along the route, not straight-line/map distances. Validate nonblank names, ordered unique sequences, and strictly increasing positive distances less than total distance. Repeated names are permitted; IDs and sequence distinguish visits.
- Store authoritative distance/elevation values in metres. Blank elevation means unknown; zero means known zero ascent.
- Audit edits and removals. Preserve historical captures and published snapshots; never silently discard checkpoint evidence when changing a course.

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
- First section: checkpoint 1 distance from start / duration from start to its captured crossing.
- Subsequent section: current checkpoint cumulative distance minus previous checkpoint cumulative distance, divided by duration between their valid captured crossings.
- Final section: remaining distance to finish / duration from last lap crossing to finish.
- Total average speed: total race distance / actual finish duration.
- If there are no lap captures, start-to-finish is the only measured section.
- Optional cumulative speed: cumulative distance from start / cumulative elapsed duration.
- With stored milliseconds: km/h = distance_metres × 3,600 / duration_milliseconds; mph = km/h / 1.609344.
- Example: 5 km in 10 minutes = 30.0 km/h, approximately 18.6 mph.
- Calculate at full stored precision; proposed display is one decimal place.
- Identify each crossing by its configured checkpoint ID. Use checkpoint sequence and captured timestamps, never rider-assignment order. A missing checkpoint must remain missing; do not shift a later crossing into its place.
- Missing distance, nonpositive duration or impossible crossing distance displays — with an appropriate review warning, never infinity, NaN or a guessed speed.
- Full-course total speed remains blank for unfinished riders; use measured split/cumulative speeds where available.
- Label sections using configured checkpoint names/sequences and include the final section to Finish. If an intermediate capture is missing, any calculated wider split must state its actual endpoints and distance.
- Show overall speed in the results table and lap distance/duration/speed in rider details or lap-comparison view. Portrait remains readable, with expanded detail if necessary; landscape exposes more columns.

### 13.4 Sound recording and voice assignment

**User requirement: capture first, speak second**
1. Organiser presses LAP or FINISH; the app immediately records and saves that crossing timestamp.
2. The app records sound/listens for a rider number for that SPECIFIC capture.
3. The organiser can say the number, for example “42”, “forty two” or “rider number four two”.
4. A clear, unique, eligible number is displayed beside that recording with a Confirm button. Only pressing Confirm assigns it and removes it from the pending list; retain it in history.
5. If nothing is said or the number cannot be interpreted, show the next/active-rider list for manual rider selection.
6. Whether assigned by voice or manually, the original captured time is unchanged.

**Required fallback and rapid-capture behaviour**
- Voice must never delay timestamp capture or block another LAP/FINISH press.
- Allow multiple pending captures while recognising or assigning earlier ones.
- Bind recording and recognition request to immutable capture ID, race and station.
- Never associate a late recognition response with the newest capture just because it arrived last.
- If overlapping speech cannot be associated reliably, retain the affected capture(s) for manual assignment.
- Do not overwrite a manual assignment or act on a voided capture, another race or a stale station session.
- Only suggest a number matching a started, unfinished eligible rider in that race; revalidate when Confirm is pressed.
- Silence, ambiguity, low confidence, unknown/ineligible number, duplicate result, microphone denial or speech-service failure leaves the capture available for manual selection.
- Do not guess from arrival order or list position.
- The next/active-rider list shows large numbers and names in observed live race order, stays stable during selection. Separate voice/audio search controls are removed.
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
| Course validation | Required checkpoint count matches list; cumulative distances increase between Start and Finish; zero checkpoints is valid; optional elevation distinguishes blank from zero |
| Unit conversions | Known metric/imperial cases match; repeated switches preserve authoritative values and timing/queue state |
| Speeds | Tests cover first partial section, repeated laps, final section, total and correction-driven recalculation; handicap has no influence |
| Unavailable speeds | No fabricated values, infinity or NaN from missing distances, nonpositive durations or beyond-course crossings |
| Voice success | One timing press records the time; the spoken number proposes the correct rider; Confirm assigns the original capture and moves it to history |
| Voice fallback | Silence, ambiguity, unknown/ineligible number, unsupported browser, denied microphone and speech/network failure expose manual selection without losing the timestamp |
| Voice concurrency | Rapid presses and late callbacks do not attach speech to the wrong capture or override manual corrections |
| Audio association | Replay identifies its original capture and clearly states local/shared availability |
| Outdoor usability | Test wind, traffic, accents, gloves and closely spaced arrivals on supported devices before race use |
| Timing independence | Enabling audio or voice does not change the timing acceptance targets in section 10 |

### 13.6 Review decisions and implementation status

Please review these proposed details: elevation means total ascent; per-organiser versus per-device units; one-decimal speed display; five-second voice window; audio storage/retention and recognition provider.

The selected visual direction remains Cycling Computer: dark surfaces, high-contrast timing numerals and large minimal controls. Delivery remains web first, Android later.

This document is the review draft. Further app implementation and deployment are paused until the organiser completes the requested review.

## 14. Batch assign — agreed default manual workflow

Agreed by the organiser on 24 October 2026. This changes the default manual assignment experience; individual capture selection remains available for corrections. Implementation remains paused for the requested requirements review.

### 14.1 Capture first

- On the separate Lap and Finish screens, press the timing button once per passing rider, including riders arriving close together.
- Each press immediately saves a separate timestamp and appends it to the unassigned queue.
- No rider selection or confirmation interrupts capture.
- The timing button stays visible and usable during all assignment actions.

### 14.2 One-tap batch assignment

- Automatically select the oldest unassigned capture and prominently display its timestamp.
- Show large rider-number tiles with names underneath.
- Tapping a rider assigns the selected timestamp immediately, then automatically selects the next unassigned capture.
- No separate timestamp-selection, Save or confirmation tap is required in the normal sequential workflow.
- For a group of riders, the organiser taps their numbers in the order they passed.
- Minimum normal interaction: one timing press plus one rider-number tap per rider.
- Order eligible riders by observed live on-course position, initially actual start order and then checkpoint passing order as overtakes are recorded. Update between assignments; keep tap targets stable during an interaction. See section 18.
- Never infer passing order from planned/actual start order or assign a rider automatically from list position.

### 14.3 Exceptions and corrections

- Undo last assignment immediately restores the affected capture to the unassigned queue and restores rider eligibility where appropriate; retain audit history.
- Assign later leaves an unidentified capture unresolved and advances to another pending capture. Show deferred captures and their count explicitly; they must not disappear or cause a loop back to the skipped capture while advancing through the batch.
- Permit direct selection of any queued capture when the observed rider order differs from the current assignment order.
- A later capture must not steal selection from the capture currently being assigned.
- Lap-assigned riders remain eligible for later crossings; finish-assigned riders leave normal active selection. A deliberate correction can recover a wrong assignment.
- Switching between batch/manual correction and optional voice must preserve capture IDs, timestamps, pending queue and existing assignments.
- Once no captures remain to assign, show queue complete and do not interpret rider taps as new timed events.

### 14.4 Acceptance criteria

| Scenario | Expected result |
| --- | --- |
| Three riders close together | Three timing presses save three captures; three rider-number taps assign them in sequence without extra taps |
| Capture during assignment | New time is retained; current selected capture/rider target remains stable |
| Unknown rider | Assign later preserves that timestamp and allows the rest of the batch to continue |
| Wrong rider tapped | Undo restores the original time and correct eligibility, with no timestamp change |
| Out-of-order identification | Organiser can select the intended pending capture directly |
| Repeated laps | Same rider can receive later lap captures; numbering follows captured timestamps |
| Finish assignment | Rider is removed from normal eligible finish choices; correction remains possible |

## 15. Close checkpoint and mark Did Not Finish (DNF)

Agreed organiser requirement: the organiser at the lap point or finish point can press a Close button to identify riders who have not passed that point and mark them DNF.

### 15.1 Controls and workflow

- Provide a clearly labelled **Close lap point** button on the Lap screen and **Close finish point** on the Finish screen.
- Closing is a station-management action, separate from the large timing capture button.
- Show a preview of affected rider numbers/names and a count. One explicit confirmation applies DNF to the listed riders and closes that station.
- Recheck eligibility when confirming so a crossing recorded by another device since the preview is not ignored.
- Do not close while that station has unassigned captures or known unsynchronised/conflicted timing records. Explain what needs resolution first. A disconnected station cannot claim a final shared closure.
- Once closed, show station closed clearly and disable normal timing at that station until an authorised organiser reopens it.
- Closing one station does not automatically close the other stations or publish results.

### 15.2 Who receives DNF

- Only started, unfinished riders can be newly marked DNF. Exclude Waiting, DNS, withdrawn, existing DNF and valid finishers.
- Closing Finish marks eligible riders without a valid finish capture DNF.
- Closing a checkpoint marks eligible riders without a valid assigned capture for THAT checkpoint ID DNF, even if they completed earlier checkpoints.
- Earlier crossings at the same physical location do not satisfy a later checkpoint: each configured row is a separate required crossing.
- The closure preview identifies checkpoint name, sequence, cumulative distance and affected riders; it checks checkpoint identities rather than just a crossing count.
- For a circuit station handling several listed checkpoints, show which configured checkpoint(s) the closure covers. Closing the final/all-checkpoints session checks every required checkpoint, not an inferred lap total.
- Riders who completed all required checkpoints but lack a finish remain Racing until finish closure.
- Missing checkpoint evidence is an informational warning. Preserve and rank a valid finish, permit race closure, and leave missing checkpoint cells unrecorded.
- On a zero-checkpoint course, there is no checkpoint closure or missing-lap conflict; only finish closure applies.

### 15.3 Results and audit behaviour

- Record DNF as a rider-entry status event, with race, closing station, organiser, closure time and reason such as “Finish closed — no finish recorded” or “Lap point closed — no crossing recorded”.
- Retain actual start and any existing lap timestamps, split durations, distances and speeds.
- Do not manufacture a finish timestamp or total completed-race duration.
- Display DNF prominently in operational tables and published CSV/results; exclude DNF riders from actual and handicap-adjusted finisher rankings.
- Keep individual DNF available as a deliberate manual status action, for example when a rider reports retirement before station closure.
- Reopening a station does not silently erase DNF decisions. An authorised organiser can reverse/correct an erroneous DNF with retained history, then resolve any newly recorded crossing/finish explicitly.
- Any finish/status conflict, including a late synchronised finish, must be reviewed rather than silently overwriting either record.
- Bulk closure is idempotent: retries do not create duplicate closure or DNF events.

### 15.4 Data model addition

| Record | Fields |
| --- | --- |
| StationClosure | id, race_id, station_id, covered checkpoint IDs/sequences, closed_by, closed_at, affected rider-entry IDs, state closed/reopened, reopening organiser/time/reason |
| RiderStatusEvent extension | optional closure_id and station_id; DNF reason and reversal/correction history |

### 15.5 Acceptance criteria

- Closing Finish marks only started, unfinished riders lacking a finish as DNF.
- Closing checkpoint 3 marks an eligible rider missing checkpoint 3 DNF, even with valid earlier checkpoint captures. Duplicate captures for checkpoint 1 never substitute for checkpoint 3.
- Riders with all required checkpoints and no finish remain Racing until finish closure. A finish is not a checkpoint capture.
- DNS, waiting, withdrawn, already-DNF and valid finished riders are not incorrectly reclassified.
- Outstanding captures/conflicts prevent premature closure and remain available for assignment.
- Counts and eligible riders are rechecked immediately before applying closure.
- DNF riders retain timing history and appear as DNF without finisher positions.
- Closure retries and reopening/correction preserve a complete audit trail.

## 16. Zero-checkpoint courses and updated setup experience

- Course setup asks for **Required checkpoints**, followed by that many editable rows: sequence, name and distance from start.
- Zero means Start and Finish only. Hide checkpoint entry rows, Lap/Checkpoint screen, checkpoint closure and checkpoint result columns.
- With one or more rows, show the ordered checkpoint workflow. Each repeated circuit crossing is its own row; no extra “required checkpoint passes” control is shown.
- Overall average speed always uses total course distance and actual start-to-finish duration. Sections use differences between configured cumulative distances.
- Retain metric/imperial conversion, batch assignment, optional voice, DNS, Finish DNF closure, handicap rankings and publication.
- During capture/assignment, make checkpoint context clear. If one physical station serves multiple configured crossings, distinguish which checkpoint a captured rider time belongs to; do not silently assume that a missing earlier capture means the current crossing is the earlier checkpoint.
- Verify both Start/Finish-only and multi-checkpoint/circuit courses in UX review before app deployment.

## 17. Mandatory rider age and race category

Confirmed organiser requirement: the rider-list screen includes **Age** and **Category**, both mandatory. Category replaces sex/gender throughout the app.

- Display rider number, name, age on race day and category in the rider list. Keep these readable in portrait and landscape; use a compact second line where needed rather than hiding mandatory fields.
- Category is a controlled selection with exactly **OPEN** and **FEMALE**. It is a competition classification, not a stored assertion of sex or gender.
- Do not retain separate sex/gender fields in the target model, entry forms, imports, results or exports.
- Category belongs to RiderEntry as a race-specific classification. Rider remains a reusable identity; age on race day is also preserved on RiderEntry.
- Require an explicit category selection; do not silently default to OPEN or infer FEMALE from a name or legacy sex/gender value.
- Require a valid whole-number age on race day. The organiser may supply that age directly, or a date of birth from which race-day age is calculated. Date of birth remains optional and private.
- CSV import requires category and either valid race-day age or a usable date of birth. Missing/invalid values are shown as row-level errors before import.
- Validate RiderHQ mappings against the same rules. A source sex/gender field is not a substitute for a confirmed competition category.
- Replace sex/gender with category in results CSVs, dashboard details and publication. Keep age on race day.
- Existing incomplete records and synthetic fixtures must be explicitly completed/remapped; preserve timings and identity links, flag invalid entries and do not invent personal data.
- In rapid timing assignment, retain prominent rider-number tiles and names; age/category can appear as compact secondary information without adding assignment taps.

Acceptance: blank age, fractional/negative age, blank category and any category outside OPEN/FEMALE are rejected; correct age/category appear in rider lists and exports; no automatic demographic inference occurs.

## 18. Batch assignment ordered by live on-course position

Confirmed organiser requirement: the riders available for batch assignment are listed in their LIVE POSITION ON THE COURSE, rather than permanently in start order. Slower riders often start first; later, faster riders may overtake them.

### 18.1 Meaning of live order

- Live position means observed physical/passing order in the race. It is distinct from fastest elapsed-time position, handicap-adjusted position, category position and planned start position.
- Before checkpoint observations exist, initialise the list from actual start order; use planned order only for riders awaiting a start. Only eligible started, unfinished riders appear in the normal assignment list.
- Update the live order when checkpoint captures are assigned: use the riders' captured crossing times at the configured checkpoint, not assignment time.
- Example: #42 starts before #17. Initial order is #42, #17. At checkpoint 1, #17 crosses at 14:10:00.0 and #42 at 14:10:02.0. Once assigned, show #17 ahead of #42 for subsequent checkpoint/finish assignment.
- A later assignment of an earlier crossing must cause the correct order to be recalculated. Corrections, reassignments and voided captures must also recalculate it.

### 18.2 Course progress and observations

- Use configured checkpoint sequence/cumulative distance to distinguish progress. An observed later course checkpoint represents greater confirmed progress than an earlier one.
- Among riders at the same latest checkpoint, earlier crossing time is ahead. When a rider's next crossing confirms they have caught/passed another rider, update their observed relative order.
- Unobserved riders keep their last known order until newer evidence supports changing it; a missing/pending capture must not be treated as proof of an overtake.
- Never compare raw times from different checkpoint locations as if they were the same crossing, or rank road position by elapsed start-to-checkpoint duration.
- With no checkpoints, start order is the only known order before finish observations. Do not imply continuous tracking or knowledge of an unseen overtake.
- Equal displayed crossing times and unresolved observations use the prior known order as a stable tie-break, flagged as uncertain where relevant.
- The list is a useful last-observed ordering, not a guarantee of current GPS position. Manual rider selection remains authoritative.

### 18.3 Interaction

- Present the main batch-assignment rider list/tiles in this live order, with prominent rider numbers and names; age/category remain available as secondary information.
- Keep one-tap assignment and automatic advance to the next queued time.
- Apply ordering updates BETWEEN completed assignments or selection actions, never moving a target during pointer/touch contact.
- A new capture arriving while the organiser is choosing must not replace the selected capture or abruptly move its rider target.
- Riders may still be selected out of the suggested order. Never auto-assign merely because a rider is first in the list.
- Finish-assigned, DNF and withdrawn riders leave normal eligible choices; lap/checkpoint-assigned riders remain eligible for later required checkpoints.
- Optional voice matching checks eligibility and rider number independently of the displayed list order.

### 18.4 Acceptance examples

| Scenario | Expected order/behaviour |
| --- | --- |
| Slow rider starts first | Initial list follows actual start order |
| Later starter crosses checkpoint first | Once passing times are assigned, later starter moves ahead in live order |
| Times assigned out of sequence | Rebuild order from captured times, not assignment sequence |
| Missing/unassigned observation | Retain last confirmed order without declaring a speculative overtake |
| Same physical circuit point, different configured visits | Compare course progress using checkpoint IDs/sequence and cumulative distance |
| No checkpoints | Retain last known start order until finish evidence; no invented intermediate positions |
| Timing correction or Undo | Recalculate observed order from retained valid evidence |
| Active tap/new capture | Selected capture and rider tap target remain stable |

## 19. Revision history

| Version | Date | Change |
| --- | --- | --- |
| 1.0 | 14 October 2026 | Initial timing, screens, model, ingestion, handicap and publication requirements; repeated lap checkpoint subsequently clarified |
| 1.1 review draft | 24 October 2026 | Adds course distances/elevation, named point distance, metric/imperial units, lap/total speed and optional audio/voice assignment with manual fallback |

| 1.2 review draft | 24 October 2026 | Agreed Batch assign as default manual workflow: oldest capture selected automatically, one-tap rider assignment and advance, Undo, Assign later and individual correction. |

| 1.3 review draft | 24 October 2026 | Adds lap/finish Close controls, bulk DNF eligibility and preview, unresolved-capture checks, status/closure audit records and correction behaviour. |

| 1.4 review draft | 24 October 2026 | Confirms DNF at lap closure for incomplete required checkpoint passes; makes required_lap_crossings mandatory in race setup and distinguishes it from the separate finish. |

| 1.5 review draft | 24 October 2026 | Confirms zero lap checkpoints are supported: Start/Finish-only setup and navigation, optional lap fields, overall speed and finish-only DNF closure. |

| 1.6 review draft | 24 October 2026 | Replaces required checkpoint-pass/circuit configuration with an ordered required-checkpoint list; names and cumulative distances are entered for every crossing, including repeated circuit visits. Updates model, speeds and DNF identity checks. |

| 1.7 review draft | 24 October 2026 | Makes age and category mandatory, adds both to rider lists, restricts category to OPEN/FEMALE and replaces sex/gender across model, imports and results. |

| 1.8 review draft | 24 October 2026 | Orders batch-assignment riders by observed live on-course position, updating start order using checkpoint passing times and preserving safe tap targets. |

## 20. Mobile UX feedback — consolidated changes

Confirmed after the organiser completed the feedback list and said “Done”.

### 20.1 Mobile timing layout

- Optimise for a phone in portrait, with responsive landscape support.
- Lap/Finish capture button at the top of its screen; selected captured time and riders awaiting assignment directly below.
- Use large, readable touch targets. Keep demo guidance, voice settings, history and station closure in secondary/collapsible controls.
- Results is a separate live-results tab. Do not place the full dashboard in the timing/assignment screen.
- Keep the capture action available without returning from an assignment screen; retain the live-order and batch-advance rules.
- Review the proposed mobile screens before changing the main race application.

### 20.2 Drag-and-drop rider start order

- Rider list supports dragging a rider by a visible handle to change planned start order.
- Renumber positions consistently after a drop, preserving unique sequential order.
- Support touch and pointer use plus accessible keyboard/move controls.
- Only waiting riders can be freely reordered; preserve recorded starts and actual start order for riders already processed.
- Start screen automatically reflects revised waiting-rider order.

### 20.3 Live race and lap timers

- Start screen shows live elapsed race time for each started rider, measured from the original start timestamp.
- Lap/checkpoint screen shows each eligible rider's live current-section/lap elapsed time, measured from start until their first crossing, then from their latest valid checkpoint crossing.
- Assignment uses the captured crossing timestamp as the timer boundary, never the later assignment time.
- Timers update without requiring page refresh or rebuilding/reordering active tap targets.
- Finish freezes elapsed race time at the captured finish; DNS has no live clock. DNF timers stop at the recorded DNF decision, with earlier timing evidence retained.
- These are elapsed-time displays; their updates must not determine event timestamps.

### 20.4 Clear accidental Lap/Finish captures

- Every pending capture can be cleared as an accidental press without assigning a rider.
- Remove a cleared capture from the assignment queue and advance appropriately; clearing another capture must not steal the current selection.
- Mark the record void with original ID/timestamp, prior assignment and audit history retained; no destructive deletion.
- Provide Undo clear/restore, checking for conflicting valid assignments before restoration.
- Permit clearing an already assigned incorrect capture through deliberate correction, with clear impact/confirmation. Recalculate rider status, live order, progress, times and results from remaining valid records.
- Cleared captures do not block station closure or count as checkpoint evidence.

### 20.5 Live results — one row per rider

- Results update live as starts, checkpoint assignments, finishes, voids and corrections occur.
- Exactly one main table row per rider; checkpoint/lap points are separate ordered columns rather than additional rider rows.
- Each checkpoint column is labelled with its configured name and distance. Proposed cell presentation: section/split duration with cumulative elapsed-from-start as secondary text; keep definitions explicit.
- Include a Last crossing / Finish point column showing the latest valid configured point reached.
- State **FINISHED** when a valid finish capture is assigned; preserve a separate review warning if required checkpoint evidence is missing.
- Retain actual and adjusted times/positions, speeds, handicap, age/category, DNS/DNF and active race elapsed time.
- Do not advance a rider's crossing point from an unassigned or voided capture.
- On mobile, allow horizontal table scrolling with rider identity held visible; preserve all checkpoint columns instead of creating multiple rows per rider.
- Run live display updates without losing the user's horizontal scroll position, sorting or selected time.

### 20.6 Additional acceptance checks

- Drag/drop changes the next waiting rider while recorded starts stay intact.
- Race and current-lap timers advance continuously; finish/DNF stops them correctly.
- Accidental capture can be cleared, skipped by the queue and restored with its original timestamp.
- Clearing an assigned finish removes FINISHED until a valid finish is restored/reassigned.
- Checkpoint columns populate in the rider's existing row; the Last crossing / Finish column changes to FINISHED on finish assignment.
- Live timer ticks do not recreate/reorder buttons during a touch interaction.
- Test at narrow phone widths and landscape; retain readable controls and checkpoint access.

| 1.9 mobile UX updates | 24 October 2026 | Mobile capture-first layout, separate live Results, draggable start order, live race/lap clocks, Clear/restore accidental captures and checkpoint columns in one rider row. |

## 21. Mobile beta testing feedback — approved 24 October 2026

This section supersedes earlier conflicting layout, voice and missing-checkpoint rules.

### 21.1 Scheduled starts

- Race setup includes a race start time in the race timezone and an editable gap between starts, default **01:00** (minutes:seconds).
- Rider list displays handicap and scheduled start time. **Highest handicap first** sorts waiting riders by descending handicap, preserving ties in existing order.
- Initial schedules and schedules rebuilt after drag/drop or handicap ordering place the first rider at race start and each following rider one configured gap later.
- Editing a waiting rider’s scheduled time inserts them in chronological start order and resets every rider after them using the configured gap. Earlier riders retain their times. Scheduled time cannot precede race start or riders already processed.
- Rebuild planned start positions consistently. Recorded actual start timestamps and actual sequence remain independent and unchanged.
- Start screen shows the next rider’s live countdown. Beep at 30 seconds, 10 seconds, then 5, 4, 3, 2, 1 and the start instant. Provide a countdown sound toggle and enable browser audio through a user gesture.
- Below the main controls show **Start list** with the next five waiting riders, and **Started riders** with the five most recent actual starters, newest first and with elapsed clocks.

### 21.2 Results and closure

- Default live results place finished riders first by fastest actual elapsed duration, followed by active live road order and other statuses. Adjusted ordering remains selectable.
- Exact column order: **Rider/status, Place, Race time, Adjusted/place, Last crossing/finish, Handicap, then checkpoint columns**.
- Finish screen has **Close race**. Confirmation states how many started, unfinished riders are still out (including zero), names them, and explains DNF consequences. Confirm must work with zero or more affected riders.
- Missing checkpoint times are warnings, not closure blockers or grounds to withhold an otherwise valid finish ranking. Do not invent missed checkpoint times.
- Pending unassigned non-void captures must still be assigned or explicitly cleared before closing. Preserve race closure, DNF, reopening and correction history. Publishing remains separate.

### 21.3 Audio assignment

- Prominent **Audio ON/OFF** button on Lap and Finish. Audio ON enables listening after every capture for up to five seconds; timestamp capture always comes first.
- Show each pending recording in newest-first order with its original captured time and recognised text, or a clear no-speech/unsupported message.
- If text matches an eligible rider number, show the rider and a **Confirm** button. Never auto-assign from speech alone. Confirm removes it from pending and retains the transcript, time, assignment and available local audio in history.
- When a new capture interrupts listening, retain the earlier capture and any audio/transcript. Late speech callbacks remain attached to their original capture. Ambiguous, silent or failed recognition uses the existing manual rider list.
- Keep individual Clear, manual batch assignment, Undo and history playback. Remove the old voice/audio search controls.

Acceptance checks: configurable gap; stable descending handicap ties; drag resets; edited-time insertion/cascade; actual capture immutability; countdown thresholds; latest-five lists; fastest finisher above earlier slower finisher; closure with missing laps at both zero and nonzero outstanding riders; three close captures confirmed out of order; no automatic voice assignment; clear/unknown/late speech safety; existing saved races migrate without losing timings.

## 22. Further approved mobile test feedback — Beta 0.4

This section supersedes earlier combined course/race setup and closure eligibility descriptions.

### Course and race separation

- Add a reusable **Course** entity with stable ID, course identifier, course name, start location, route name, total distance, optional ascent and ordered named checkpoint distances. Provide a saved course list and separate Course setup screen.
- Race stores its own name, date, timezone, scheduled start, standard rider gap and selected course ID. Include organiser name, start and finish timekeeper names, checkpoint timekeeper names and optional marshall names. Rider entries remain race-specific.
- Retain the selected course snapshot with the race so editing a reusable route cannot rewrite past timings or distances. Changing a timed race's course requires a new race; legacy combined records migrate without losing captures, riders, schedules or results.
- New race start defaults to **current time + 5 minutes, rounded to the nearest 5-minute boundary**, including date rollover in race timezone.

### Start list and entry controls

- Provide **TIME GAP** as an extra row between riders, default duration equal to the standard race start gap. It adds that extra pause to the normal interval; recalculate subsequent schedules. Permit gap removal.
- Rider list includes Remove with explicit confirmation. Archive the removed entry and preserve evidence; a rider with actual start evidence must be retained and marked DNF rather than deleted.
- On Start, provide a control to override the next scheduled start to current time rounded **up** to the nearest minute. Before any recorded starts, update the race schedule too; once riders have started, preserve their actual timestamps and adjust waiting riders only.

### Closure

- Confirmation states both the number out on course and the number waiting to start, explicitly warning how many will become DNF and how many will become DNS. Include zero counts.
- Closing changes Racing to DNF and Waiting to DNS, leaving finishers and existing DNS/DNF unchanged. Missing checkpoint times still do not block closure; pending captures must still be assigned or cleared.
- Previously closed race records with Waiting riders are upgraded to DNS. Preserve audit; explicit reopening may reverse statuses generated by that closure.

### Speech diagnosis and fallback

- Include browser speech recognition, separate from local audio recording. Do not imply that successful recording proves transcription is supported or functioning.
- Add a five-second speech test with browser/microphone API availability, connection state and explicit recognition error codes/messages (permission, service unavailable, network, no speech, audio capture or language failure).
- Provide Enter / dictate number as a fallback. Device keyboard dictation can fill that field where available; it is distinct from automatic browser recognition. Rider suggestion still requires Confirm.
- Keep recording playback accessible for pending captures and history, preserve immutable time/capture identity, and never discard a time when transcription fails.
- Device-specific microphone/speech testing remains required. Do not claim the reported phone issue fixed without observing successful speech on that phone.

Acceptance: old race migration retains timestamps; course reuse leaves prior snapshots unchanged; default rounding handles midnight; TIME GAP inserts extra standard duration; removal confirms and archives; override rounds up without altering actual captures; closure counts and applies both DNF/DNS; speech failures show actionable diagnostics and manual fallback.

## 23. Approved next-iteration feedback — Beta 0.5

### Simpler names, list and edit behaviour

- Course setup keeps one **Course name**, removing the duplicate Route name field. Course identifier remains.
- Whenever Edit or a correction dialog opens, automatically scroll to the top/dialog so it is visible from a long roster.
- Rider-row Edit and Remove controls appear side by side for compact rows.
- Provide one **+ Time Gap** control at the top of Riders. Each tap appends a TIME GAP row, defaulting to the standard race gap. Every gap has a drag handle and Remove. Repeated taps add independent gaps.
- Riders and gaps form a single ordered start list. Moving either recalculates subsequent scheduled starts. Gaps may be leading, intermediate or trailing. Preserve the processed prefix and actual timing evidence; never move a processed rider or a gap into that locked prefix.
- Lap and Finish retain active riders and add a separate **Yet to start** list below the closure control, sorted by scheduled start order.

### Start modes and autostart

- The Start-screen mode button cycles **Countdown sound ON → Autostart on / sound ON → Countdown sound OFF**.
- Autostart is armed only from a positive countdown. Disable/grey the manual Start button while armed; fill it green left to right over the final ten seconds.
- At zero, create one start for the scheduled rider and advance. Store the scheduled automatic timestamp plus actual trigger time/lateness and clock evidence. Preserve one-button manual input capture unchanged in manual modes.
- Overdue riders use manual Start. Do not invent catch-up starts after a backgrounded app, missed deadline, changed race/rider/schedule or cancelled arm. The beta accepts at most 100 ms timer lateness; otherwise it requests manual start. This does not establish cross-device accuracy.
- Cancel or revalidate the arm on station/navigation changes, backgrounding, closure, changes of mode and schedule. Autostart requires an explicit mode choice for the browser session rather than silently rearming on reload.

### Corrections and results

- After race closure, permit correction of existing assigned Start, checkpoint/lap and Finish timestamps without reopening the race.
- Retain original timestamps, clock evidence and per-capture before/after values, reason and correction time. Require a reason and valid Start → recorded checkpoints → Finish chronological order.
- Start/finish corrections recalculate race duration, adjusted duration, speed, actual/adjusted positions and crossing sequence. Checkpoint corrections recalculate affected splits/cumulative durations/speeds; they do not change start-to-finish duration.
- Add **Start time** immediately before Race time in Results. Other agreed column ordering remains intact.
- Export results CSV in **actual place order**, fastest actual finishers first regardless of current dashboard sort; preserve other statuses afterwards. Include actual start timestamp and use corrected values.

### Speech correction and current beta limitation

- The phone test works with Audio OFF but fails with Audio ON. Code inspection found an app-held microphone stream plus a separate browser recognition microphone. Remove that competing ownership.
- In Beta 0.5, Audio ON gives recognition exclusive microphone access. Do not open a parallel getUserMedia stream or MediaRecorder. Serialise recognisers across quick captures, retain late text against its original capture and keep explicit rider confirmation.
- Preserve transcript, errors, manual/keyboard-dictation fallback and previous stored clips. The beta speech-only path does **not** create new replayable raw-audio clips; disclose this in the UI. A reliable single audio-capture/transcription pipeline remains needed to restore simultaneous raw-clip archiving across phones.
- Recognition and device audibility still require a real-phone retest. Automated microphone lifecycle tests are not proof of phone recognition quality.

Acceptance checks: mixed draggable gap positions; compact controls; auto-scroll; both timing screens' waiting order; real browser autostart at zero; missed/hidden/overdue cancellation; one automatic event; post-close start/finish/lap corrections; retained originals and chronological rejection; start column; exported actual-place order; Audio ON with no competing stream/recorder and safely serialised speech sessions.

### 23.1 Excel-compatible CSV encoding — Beta 0.5.1

Results and template CSV downloads include one UTF-8 BOM so Excel can identify Unicode text. Export durations use the plain ASCII minus (-), missing durations are empty fields, and names/checkpoint labels retain Unicode accents. Preserve actual-place ordering, original calculation values, quoted-field escaping and CRLF rows. Regression checks inspect downloaded bytes and cover negative adjusted durations, DNS/missing times and accented rider names.

### 23.2 Nonnegative result durations — confirmed

A handicap larger than the actual duration is invalid. Preserve actual duration and actual place; show Check handicap and leave adjusted duration/position blank until corrected. Exclude the rider only from adjusted ranking. Export an explicit validation warning and no negative duration. Do not clamp to zero or change actual timestamps to conceal an invalid handicap. Equal handicap and actual duration may yield an adjusted duration of exactly zero. Provide a post-race Edit handicap action and recalculate on correction. Invalid chronological timing must be corrected rather than shown as a negative lap/race duration.

## 24. Shared speech batches for close riders — Beta 0.6

- Every Lap/Finish press still saves its own timestamp immediately. The first press starts one continuous speech session; further presses at the same checkpoint/station join that batch without stopping/restarting recognition and extend the window to five seconds after the latest press. Limit one window to 30 seconds or 20 captures; subsequent presses start a new batch.
- Speak numbers in passing order, preferably separated by “then”. Parse multiple digit or spoken numbers conservatively. Pair the first recognised number with the first press, and so on. Keep capture IDs and timestamps immutable.
- After listening finishes, freeze the batch for review. Show transcript and ordered time/rider suggestions. Require Confirm batch; recognition never assigns automatically. New captures form a separate batch rather than moving frozen suggestions.
- Count mismatches, ambiguous speech, duplicate numbers, unknown/ineligible riders and captures changed by manual assignment or clear block bulk confirmation. Review riders permits explicit selection for every remaining capture; do not shift surviving numbers over a missing rider.
- Revalidate the full proposal before an atomic all-or-none confirmation. Preserve transcript, batch identity, capture assignment source and audit history. Cleared/assigned times stay retained. Normal manual selection, Clear and Undo remain available.
- Recognition keeps exclusive microphone ownership. A later batch waits for release of a closing recogniser; no competing MediaRecorder/getUserMedia stream is added. Existing raw-clip limitations remain.
- Late callbacks cannot change a frozen/manually reviewed/confirmed batch. Recover interrupted batches as review-required on reload. Browser/network errors preserve all timestamps and expose manual review. Real phone recognition still needs testing.

## 25. Final beep, speech revisions and pauses — Beta 0.6.1

### Countdown sound consistency

- The zero/start beep must use the same volume and envelope as the preceding countdown beeps.
- Advancing automatically to the next rider must not truncate a playing final beep or cancel the imminent zero beep. Explicitly turning sound off may stop playback.
- Verify scheduling and cancellation in automated tests; audible consistency still needs a device test.

### Final speech results and pauses

- Request final speech results only for race capture and the speech help test. Intermediate recognition hypotheses must not accumulate in the saved transcript.
- Store results by browser result index. Repeated delivery or revision at the same index replaces that result, rather than appending another copy.
- Preserve separate final speech segments as explicit number boundaries. “46 [pause] 55” must produce suggestions for riders 46 and 55 when the browser exposes separate segments, equivalent to “46 then 55”.
- A pause is not guaranteed to produce a segment on every device. If no reliable boundary exists, do not guess that 4655 means 46 and 55. Count mismatch or ambiguity requires manual review.
- Do not remove genuine duplicate final numbers merely to make the count match: retain the evidence and block confirmation through duplicate-rider validation.
- Interpret the completed transcript when listening ends, retaining the shared-window rules in section 24. Each button press remains a separate immutable timestamp; a batch transcript may appear against multiple related captures in history.
- Preserve existing historical transcripts and timing evidence. This fix does not retrospectively repair old recognition text.
- Acceptance: repeated interim “46” / “46 then” / “46 then 55” does not duplicate text; repeated callbacks at the same final index are idempotent; separate final 46 and 55 segments remain separate; unsegmented 4655 for two captures requires review; genuine duplicate numbers remain review errors; frozen batches ignore late callbacks.
- Automated tests cover these cases. Actual Android/browser speech service recognition and pause detection remain subject to the organiser’s phone retest.

### Current authoritative setup and results summary

- Course has course identifier, one Course name, start location, total distance, optional elevation and ordered named cumulative-distance checkpoints. There is no separate Route name field.
- Race has name, date, timezone, scheduled start, standard gap, selected course and its retained snapshot, organiser, start/finish/checkpoint timekeepers and optional marshalls. Rider entries belong to the race.
- Current main results column order is **Rider/status, Place, Start time, Race time, Adjusted/place, Last crossing/finish, Handicap, then checkpoint columns**.
- Display the running app version in the header status bar on every screen. Beta 0.6.1 identifies the speech-revision/pause update.

This specification is current through the approved Beta 0.6.1 changes. It does not imply all original product requirements are implemented: RiderHQ access, results email delivery, simultaneous raw-audio archiving, robust multi-device event merging and demonstrated cross-device timing accuracy retain their stated dependencies or beta limitations.

## 26. Approved race dashboard and timing feedback — Beta 0.7, 25 October 2026

This section supersedes conflicting earlier countdown-tone and speech punctuation rules.

- Final zero/start beep is longer than preceding countdown tones. Beta uses one second, retaining the same pitch and volume; automatic next-rider advance must not cut it off.
- Lap and Finish show a **Recently assigned** list after **Yet to start** and before **Recording history**, with rider number/name and original recorded crossing time. Sort latest assignment first, irrespective of capture order. Track assignment order separately. Legacy records without that evidence use capture order rather than inventing assignment timestamps.
- Show a completion tick on the selected checkpoint and Finish when all applicable riders have a valid assigned crossing there and no unassigned recordings remain. Waiting riders prevent completion; DNS/DNF are excluded. Do not show a completion tick for an empty applicable list.
- Beside **Live results**, show elapsed time from the first actual start. Freeze when every entry is Finished/DNS/DNF, using the latest valid finish/DNF time. On closure replace the timer with **Race closed**.
- Saved races display **Closed** if closed; otherwise **Live** after any valid rider start; otherwise **Pending** before scheduled race start or **Overdue** at/after it. A completed but unclosed race stays Live.
- Add **Dashboard** navigation and access from the race chooser. One row per race contains full race ID, name, course name, race date/time, entered count, status, finished count, on-course count, fastest valid actual duration and fastest valid adjusted duration.
- Combine course, inclusive date range and rider filters. With a selected rider, show only their entered races and replace fastest duration columns with that rider’s actual and adjusted durations. Pending/DNS/DNF or invalid values are blank/dash, never fabricated; invalid handicap leaves adjusted blank.
- Shared summaries refresh every 15 seconds and on demand. Include unsynced/device-only records, flag unavailable shared data, retain filters and table scroll, and provide Open race. Use the race’s saved course snapshot for historical course names.
- Identify a rider across races by stable reusable rider identity where available, otherwise email. Beta legacy fallback uses bib/name/age/category and may create separate choices if those details differ; do not silently merge solely by reused bib.
- Speech always yields whole rider numbers, never a time/duration. “12 [pause] 25”, “12 then 25”, and browser text “12:25” represent two numbers, 12 and 25. Non-number words/punctuation delimit numbers and remain in the audit text. Do not guess that undelimited 1225 is 12 and 25.
- Display **Recognised text** and **Interpreted rider numbers** separately in capture/batch review and retained history, including unsuccessful interpretations. Preserve complete final segment wording; a visual segment separator may be used. Keep explicit Confirm and existing count, ambiguity, duplicate and eligibility protection.
- Acceptance covers status precedence and boundary time; checkpoint-specific completion and clear/restore; reverse-order assignment confirmations; running/frozen/closed race timer; aggregate and rider-specific durations and combined filters; long final beep preservation; colon/word-separated speech, immutable capture times and retained raw text.

Physical Android speech recognition and audible countdown checks remain device-validation tasks; automated tests do not establish microphone quality or cross-device timing accuracy.

## 27. Approved compact management iteration — Beta 0.8, 25 October 2026

- Dashboard hides the internal race ID. Business uniqueness is normalised race name + race date + HH:mm scheduled time; enforce both in setup and shared writes, including concurrent creation. Preserve stable internal IDs for references. Do not silently rename existing duplicate historical records.
- Dashboard race/course names use compact two-line cells and wrapping headers. Fit the table to landscape mobile width; portrait may scroll. Add ascending/descending header arrows, semantic sort state and a combined Status filter.
- Results column headers support ascending/descending rider, actual place, start time, actual duration, adjusted duration, finish crossing, handicap and checkpoint time sorting. Keep export in actual-place order regardless of display sort.
- Use fixed mobile tab rows: **Course, Race, Riders, Dashboard** then **Start, Lap, Finish, Results**. Keep Lap disabled when there are no checkpoints so the layout stays consistent.
- Start-list batch removal selects only entries with no historical start. Review the count/names and confirm. Validate the entire batch before any removal; archive entries and recalculate schedules without altering actual timing evidence.
- Race scheduled start input is HH:mm, without seconds. Preserve unchanged legacy timestamp evidence when syncing old races; edited setup uses minute resolution.
- Match Course/Race list controls: saved-list selection, New, Duplicate and confirmed Delete. New race starts with an empty roster. Duplicate copies setup, entries and start gaps while resetting timings, statuses, closure and speech history. Choose a noncolliding default name.
- Race Delete is allowed only when no start has ever been recorded, including subsequently voided starts; revalidate on the shared service with version checking. Implement as archive, retaining records and preventing stale-device resurrection. Course archival preserves snapshots already attached to races.
- Extend capture listening to **10 seconds after the latest press**, retaining the existing 30-second/20-capture cap. If the browser closes recognition naturally within the window, reopen serially without competing microphone ownership and retain previous final segments.
- **Requested but not guaranteed in Beta 0.8:** every acoustic pause longer than 0.5 seconds must delimit rider numbers, including within a single browser transcription. The browser path does not reliably expose that within-result timing. Separate recognised segments, punctuation and explicit words remain supported; do not guess a split of a concatenated integer. UI must disclose the limitation and “then” fallback. A suitable single audio-capture/VAD/transcription pipeline or native speech capability remains required for exact threshold enforcement.
- Acceptance: uniqueness including concurrent saves; minute-only edits; protected starts and atomic batch removal; confirmed archive and stale-save rejection; duplication resets; header sort directions and missing-values-last; status filter; correct tab placement; landscape fit; ten-second deadline and natural speech restart without segment loss.

Physical-phone speech/pause detection, beep audibility and narrow landscape usability remain device validation tasks. This section supersedes prior five-second voice and dashboard race-ID display rules.

## 28. Approved list-first Race/Course design — Beta 0.9

- Remove the header **Races** button. Race/Course tabs open lists rather than saved-item dropdowns with an always-open editor.
- Highlight the currently selected entry. A race row shows name, date/time, course, entry count and status; a course row shows name, identifier, distance and checkpoint count.
- Place **New, Duplicate, Batch delete** above each list and **Edit, Remove** beside each entry. Selection and editing are distinct actions; timing always uses the selected race.
- Edit opens the complete existing form above the list and scrolls to it. Include Save and Cancel editing. Preserve every existing race field (name, date, HH:mm start, standard gap, selected course, organiser, start/finish/checkpoint timekeepers, optional marshalls and unit selection) and course field (identifier, name, start location, distance, optional elevation, checkpoint names/distances).
- Reuse existing validation, uniqueness and timed-course snapshot protections. New/duplicate race drafts are committed on Save; cancellation restores prior selection. Duplication keeps the already-agreed timing/status reset rules.
- Batch mode provides checkboxes and Select all eligible items. Removal confirmation lists count and item names; include race date/time. Started races cannot be selected or removed, including historical starts later voided.
- Recheck local/shared state and version at execution. Archive each record using conditional writes; preserve snapshots and reject stale resurrection. If some per-record operations fail, report successes and each failure rather than claiming an atomic bulk transaction.
- If the selected race is removed, select a remaining saved race. If none remain, show no selected race and require selection/new saved setup before timing; do not silently create a replacement saved race.
- Prominently show **Current race: name, date, scheduled time, course** on Start, Lap, Finish and Results. The banner follows the selected race and uses its course snapshot.
- Acceptance: list-first render, selected highlighting, full-field edit coverage, Save/Cancel flows, New/Duplicate, confirmed bulk race/course archival, started-race protection and rejection reporting, selected-race switching/empty list, removed redundant button, all four banners and retained timing/voice/export behaviour.

The reviewed UI preview simplified fields solely to demonstrate interactions; the implemented forms must retain the full required fields.

## 29. Approved reusable riders and setup protection — Beta 0.10

- Show the current selected race banner on Riders as well as Start, Lap, Finish and Results.
- Race setup becomes read-only after any rider has started or the race has completed/closed. A subsequently voided start or reopened race does not unlock setup. Course setup is read-only when any race using it has started or completed. Enforce these rules in the UI and shared API; retain explicit timing corrections after closure.
- Lap and Finish order is active riders, recently assigned riders (latest assignment first), then riders yet to start. This supersedes section 26 placement.
- Race edit provides Import riders from another race, with source selection, preview and confirmation. Import entry details, handicaps and predictions; skip existing people, show reassigned colliding bib numbers, and never copy starts, crossings, finishes, statuses or speech evidence.
- Add rider offers unique saved riders across races, ordered by surname then age, with best valid actual finish duration on the same course. Reuse identity across entries. Prefer stored identity, then name/DOB, email/name, or exact legacy name/age/category; legacy incomplete records can remain separate choices. Do not merge by bib alone.
- Optional date of birth is available on rider add/edit and CSV import. Calculate mandatory age on the selected race date, validate actual calendar dates and prohibit future births. Without DOB require age explicitly. Recalculate when race date changes; DOB is not included in results CSV.
- Acceptance covers birthdays/leap dates, age refresh, directory order and matching, same-course best-time exclusions, duplicate/repeated imports and bib collisions, entry-only import, locked race/course API writes including historical starts, preserved timing corrections, selected-race banner and recent-assignment order.

This release does not change the existing speech, external integration or field timing validation limitations.
