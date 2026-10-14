# Time Trial — Cycling Computer web app

Private test edition: responsive dark cycling-computer interface, shared append-only event storage in R2, local IndexedDB upload queue, separate Start/Lap/Finish screens, repeated crossings at one lap checkpoint, DNS, capture assignment/correction, handicap rankings, portrait/landscape tables, entry editing, add-new CSV imports, exports and backups.

20 synthetic riders, 3 races, 2 test organisers, 60 entries. Default is the live sample race; select the upcoming Woodland race to try the full start queue. Organiser selection records test attribution only, not separate authenticated volunteer permissions. The Site remains owner-private.

## Build

Requires Python 3 and Node.js. Run `python build-worker.py`, `bash scripts/build.sh`, then `node scripts/validate-artifact.mjs`. Worker source serves embedded HTML, JavaScript and seed fixture. Source files are in worker/; generated worker/index.js is deployable. Configure the R2 binding BUCKET in .openai/hosting.json. Only private Sites access currently protects the shared API.

Run `node tests.mjs` for domain and worker route checks. Browser QA was unavailable in this environment; field/device validation remains required.

## Test workflows

- Start/DNS automatically advance the next waiting rider.
- Lap or Finish: capture repeatedly, then Assign rider to the matching recorded time. Reassign with Correct. Void accidental presses with a reason.
- One checkpoint supports successive lap crossings ordered by capture time.
- Results: select sorting, tap a rider for splits, export CSV.
- Setup: edit handicap/prediction/start order; import additional riders using the downloadable CSV template.
- Download full backup exports the original seed and append-only recorded events.

## Explicit implementation limits

This is a working test release, not validated race equipment. Clock offset is estimated with five HTTP round trips and periodically checked; 0.1-second cross-device accuracy is not established. Clock uncertainty is displayed. Background/sleep and human reaction have not been certified.

Device queue survives reload but first page load requires connectivity. Concurrent conflicting assignments/starts are not yet guarded by database transactions. Events are immutable, but authoritative conflict review, role invitations, repeat-import mapping/update mode, independent checkpoint operator access and full production validation remain to implement.

RiderHQ access is pending its supported API contract. Email provider/sender is unconfigured; Publish explains this and offers CSV review. No email is sent. All synthetic addresses use example.invalid. Actual/adjusted ties use competition ranking at the displayed tenth.

Do not use with a real race until timing, multi-device concurrency, security, offline recovery and publication delivery requirements in docs/requirements.md are satisfied.
