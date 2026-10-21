# Testly MoSCoW Prioritization

This classification defines the scope of the first Testly release. Priorities apply to the MVP and should be reviewed after MVP validation.

## Must have

Without these features, Testly cannot deliver its core no-code testing workflow.

| Feature | MVP scope | Rationale |
|---|---|---|
| Browser recording | Record clicks and text input in Chrome using a clean browser profile | Provides the core no-code test creation experience |
| Readable steps and `.flow.yaml` | Convert recorded actions into readable steps and save them as plain-text `.flow.yaml` files | Makes tests understandable, portable, and suitable for version control |
| Steps and YAML editor | Provide synchronized Steps and YAML views while preserving valid test content | Allows users to review and correct recorded tests |
| Test runner and output panel | Run a saved test in Chrome and display the status of every step | Completes the create-edit-run workflow required for a usable MVP |
| Secure secret handling | Replace sensitive values with environment variables, store values in a local `.env` file, and mask them in output | Prevents credentials from being committed to Git or exposed in logs |

## Should have

These features provide significant value and should be delivered after the Must scope is stable.

| Feature | Target scope | Rationale |
|---|---|---|
| Failure triage | Show the failed step, screenshot, browser console, and network activity | Reduces the time required to identify test failures |
| Git versioning | Initialize or connect a repository and support saving test history | Enables collaboration, traceability, and code review |
| Reports and CLI | Generate HTML and JUnit reports and provide a `testly test` command | Enables CI/CD execution and machine-readable results |
| API steps | Send requests, extract response values, perform assertions, and share variables with UI steps | Supports broader end-to-end scenarios without requiring another tool |

## Could have

These features are valuable enhancements but are not required for the first usable release.

| Feature | Future scope | Rationale |
|---|---|---|
| Export to Playwright | Generate maintainable Playwright `*.spec.ts` files from Testly scenarios | Reduces vendor lock-in and supports advanced customization |
| SQL steps | Run masked Postgres, MySQL, and SQLite checks using shared scenario variables | Extends validation to the data layer |
| AI agent | Generate tests and repair fragile selectors through a configurable AI provider | Accelerates test creation and maintenance but adds cost and technical risk |
| Step re-recording | Re-record only a failed or outdated step without recreating the complete scenario | Improves maintenance efficiency after the core recorder is reliable |

## Won't have in this release

These features are outside the MVP and first-release scope. They may be reconsidered after web testing is validated.

| Feature | Deferred scope | Rationale |
|---|---|---|
| Mobile recording | Record Android and iOS interactions through device mirroring | Requires a separate capture and execution architecture beyond the Chrome-focused MVP |
| Multi-browser recording | Record directly in Firefox, Safari, or Edge | The first release focuses on a reliable Chrome recording workflow |
| Hosted test management platform | Provide cloud storage, dashboards, user management, and team administration | Testly initially relies on local files and Git rather than operating a cloud service |
| Proprietary test format without export | Lock tests into a closed storage or execution format | Conflicts with the product goal of readable files and low vendor lock-in |

## Priority summary

| Priority | Feature count |
|---|---:|
| Must have | 5 |
| Should have | 4 |
| Could have | 4 |
| Won't have in this release | 4 |
| Total | 17 |
