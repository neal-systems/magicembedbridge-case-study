# Operating model

MagicEmbedBridge is operated as a scheduled Windows service with a separate
health-and-recovery path. Normal operation is observable from structured logs,
health summaries, the Windows event log, scheduler results, and the freshness of
the last successful render. The operating documents define both routine checks
and the point at which an automated recovery must be escalated to a person.

## Monitoring

Each rendering cycle records a correlation value so events from one run can be
followed across authentication, rendering, upload, and state changes. The service
keeps application-wide health and a separate result history for each content item.
Useful signals include:

- whether the service health probe responds;
- the latest scheduler result and whether scheduled work is still repeating;
- the time and outcome of each item's last render and upload;
- consecutive failures and the oldest successful result;
- authentication, invalid-content, credential-expiry, and unhandled-error events;
- actions and results from the recovery task.

The event log is the primary host-level alerting surface. Structured, rotating log
files provide the detail needed for diagnosis. Health summaries distinguish
healthy, stale, failing, disabled, and degraded items so one failed source is not
confused with a service-wide outage.

## Content failure behavior

The renderer waits for an explicit ready or error signal before taking an image.
If the source cannot load, it does not publish a blank or incomplete replacement.
The last successful image is retained. When that image exceeds the configured
freshness limit, the service can publish it with a visible stale warning so a
viewer is not led to believe old data is current.

A failed item is isolated from the rest of the cycle. State is saved atomically,
and scheduler cycles cannot overlap. If bounded parallel rendering is enabled,
completed results are persisted as workers finish so one slow or failed item does
not erase the progress of others.

## Automated recovery

The health task runs independently of the long-running service. It probes the
local health endpoint, records a warning while a failure is below the restart
threshold, and restarts the service after repeated failed probes. It probes again
after the restart. A service that remains unhealthy is reported as a failed
recovery and requires human attention.

An operator then checks the scheduler state, the health response, and the service
log for a startup problem such as invalid settings, an unavailable port, or a
damaged runtime. Authentication failures are handled separately because restarting
cannot repair expired or incorrect credentials. Invalid source content is also
kept separate from host failure so it cannot trigger unnecessary service-wide
restarts.

The production acceptance record includes continuity beyond the inherited
72-hour scheduler limit. The service process must remain alive after 73 hours,
scheduled rendering must still be repeating, and fresh results must still be
reaching the signage system.

## Versioned offline installation

Each release is built as a complete offline package. It contains the application
runtime, fixed dependencies, browser runtime, operating documents, and integrity
hashes. The package and its transport copy are verified before extraction, and
the extracted files are verified again before installation. The host does not
resolve new package versions or download a browser during the install.

Releases are installed side by side in versioned directories. Configuration,
state, rendered images, and logs are shared outside those directories. A new
release is prepared and checked without changing the currently active version.
The scheduler follows an active-version pointer, so cutover occurs only after the
new runtime is ready. Production tasks remain disabled until the operator's
go/no-go checks pass.

## Upgrade and rollback

For an upgrade, the operator verifies the package, prepares the new isolated
runtime, checks permissions and state compatibility, switches the active-version
pointer, and runs health plus forced-render validation. Old versions are retained
for a defined window.

Rollback switches the pointer to the recorded prior version and repeats the
verification. Shared state is designed to tolerate additive fields written by a
newer release. A change that renames or removes required state is treated as a
compatibility decision before cutover because it may prevent an older version
from starting.

For a first-generation replacement, the previous installation is renamed and
preserved before the clean install. Restoring it is therefore a rename-back
operation rather than reconstruction from memory. Material is retired by disabling
its scheduled work and retaining the renamed directory until the recovery window
has passed; deletion is not part of the immediate procedure.

## Backup scope

Version directories are reproducible from their verified release packages. The
durable backup set is the protected configuration, the content definition, and
the state that maps content items to their latest outcomes and signage references.
Secrets are backed up only to an approved secret store. A restore places those
durable files back beside a verified application version and then repeats the
normal health and render checks.
