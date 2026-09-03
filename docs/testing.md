# Testing evidence

The recorded evidence includes a 649-test middleware suite and a separate 46-test
deployment-generator suite. The full delivery and operational validation set
contains more than 700 tests; the two named suites are important subsets, not an
exhaustive sum. The primary application has a 97% coverage baseline.

This document describes the test layers and file-name patterns. Test bodies,
fixtures, product code, schemas, manifests, and deployment scripts are not
published here.

## Suite organization

| Layer | File-name pattern | What it proves |
|---|---|---|
| Middleware unit tests | `tests/unit/test_*.py` | Individual rules behave predictably without live source systems or a live signage server. |
| Middleware integration tests | `tests/integration/test_*.py` | Web routes, authentication boundaries, application state, and mocked upstream interactions work together. |
| Renderer and scheduling tests | `tests/unit/test_due.py`, `tests/unit/test_tick.py`, `tests/unit/test_browser.py`, `tests/unit/test_watermark.py` | Due-item selection, cycle isolation, render readiness, output handling, failure behavior, and stale-content treatment. |
| State and concurrency tests | `tests/unit/test_state*.py`, `tests/unit/test_lock*.py` | State survives interruption, overlapping cycles are prevented, and compatible state can survive rollback. |
| Source and signage integration tests | `tests/unit/test_powerbi_*.py`, `tests/unit/test_magicinfo_*.py` | Authentication, content resolution, upload, approval, retry, and upstream-error boundaries. |
| Service and management tests | `tests/unit/test_server_*.py`, `tests/unit/test_routes_*.py`, `tests/integration/test_flask_routes.py` | Health and management endpoints enforce their contracts and authorization boundaries. |
| Observability and security tests | `tests/unit/test_logging_*.py`, `tests/unit/test_eventlog.py`, `tests/unit/test_heartbeat.py`, `tests/unit/test_secrets_guard.py`, `tests/unit/test_audit.py` | Logs remain structured, health signals are raised at the intended boundary, recovery channels stay independent, and protected settings are not exposed. |
| Offline-delivery contract tests | `tests/unit/test_deploy_*.py`, `tests/unit/test_usb_payload.py`, `tests/deploy/*.Tests.ps1` | Release contents, checksums, Windows compatibility, installation wiring, active-version switching, rollback selection, and recovery-task registration remain intact. |
| Documentation contract tests | `tests/unit/test_docs_*.py`, `tests/unit/test_schema_*.py` | Published operating claims and generated contracts do not drift from the application. |

## What the counts mean

The 649-test figure is the recorded middleware-suite baseline. It combines the
unit and integration layers rather than counting only test files or function
names. Parameterized cases can produce more executed tests than the number of
named functions.

The 46-test deployment-generator suite is recorded separately because it validates
release construction and Windows delivery behavior rather than the running
middleware alone. Other operational checks bring the complete evidence set above
700 tests.

Coverage is reported only for the primary application. The 97% baseline does not
claim that deployment scripts, external platforms, or Windows host behavior are
97% covered. Host-only behavior is closed through installation checks, recovery
validation, forced rendering, rollback exercises, and the post-73-hour continuity
gate.

## Evidence boundaries

Most unit and integration tests use controlled substitutes for external systems.
That makes failure paths reproducible, but it does not prove a particular Windows
host, account permission, network route, or signage assignment. Those facts are
verified during deployment and recorded by the operator.

The production counter of 1,511 successful renderer completions is operating
evidence, not a substitute for the test suites. Conversely, passing tests do not
by themselves prove that a display is showing the assigned content. Application,
delivery, and live-display evidence answer different questions and are kept
separate.
