# Viedoc.Data quality KPIs

Oct 28, 2026 · Phuong Chu

What to monitor for the quality of Viedoc.Data (the Nucleus data platform and the `viedoc-data` MCP): the test pipelines, bugs and requirements, the health of the running app on dev1, and the gaps that keep some of it unmeasured. Started from the shared Claude Doc "Viedoc.Data quality KPIs" (rev 53).

## Test pipelines to monitor

Tests run at four steps, from pull request to the running app. Each row is something to monitor; a step that is not built yet is a gap, not a green. Actual state checked 2026-11-03 against `origin/main` of the Viedoc.Data repo and the ADO pipeline list.

| Step | Pipeline | Tests | Expected | Actual | Watch |
| --- | --- | --- | --- | --- | --- |
| 1. Pull request, before merge to main | 861 "[PR Validation] Build & Test Apps" (Viedoc.Data, `build-validation.yml`) | .NET: unit, integration, system, acceptance, convention. Cockpit web: vitest. Code format check (`dotnet format`) | Every PR into main runs all dev tests | As expected. A PR that changes only `docs/*` or `*.md` files skips it | Pass rate, failed tests, test count per suite, run duration |
| 2. Before deploy | 887 "1. Deploy App" (Viedoc.Data, `deploy.yml`), Test stage | Same as step 1: every `*Tests.csproj` and vitest | Every deploy runs all dev tests first and deploys only when they pass | As expected. Queued by hand (`trigger: none`), not on every merge to main; branch and environment (`dev1` or `pgd`) chosen at queue time. "Dry run" runs the tests and skips the deploy | Pass rate, failed tests, test count per suite, run duration |
| 3. Smoke test after deploy | 887 "1. Deploy App", after "Deploy via deploy.ps1" | A check against the running app | Each deploy ends with a check against the running app, and the run fails when the app is broken | **Not built**: 887 ends at "Deploy via deploy.ps1". In progress: Task 108676 "Add smoke test to the deployment pipeline", under TBI 107801 "Shift Left Testing" | Pass or fail per deploy, once built |
| 4. End-to-end | None yet; end-to-end suite `Viedoc.Data.Testing` (Viedoc POC, this repo) | Questions to the chatbot as named test users on dev1, answers checked against MySQL | A pipeline runs the end-to-end suite: the short set (about 20–30 questions) after each dev1 deploy, the full set (paraphrases, answer wording, red-team) weekly and before each release; its pass rate shows on the dashboard | **No pipeline**: run by hand per ticket. Not planned yet | Pass rate per run, test count, run duration |

- In steps 1 and 2 a skipped test counts as failed (`xUnit.FailSkips`), so a green run has no skipped tests. Checked 2026-11-03: the latest 861 and 887 runs have 0 skipped.
- Steps 1 and 2 run the same tests; step 2 is the last gate before the deploy, and the only one that runs on the code that is deployed.
- Test count: watch the trend, not the absolute number; a drop means tests were deleted while the build stays green. Baseline on main, 2026-10-29, in test methods: about 2,950 unit, 66 integration, 136 system, 116 acceptance, 17 convention and 63 vitest. ADO counts test cases, so its totals are higher (a theory runs several cases): the 887 run of 2026-10-30 published 4,696 .NET results and 63 vitest.
- Run duration matters most for the end-to-end suite: it is slow, costs LLM tokens and decides how often it can run.
- The chatbot is not deterministic: a scheduled end-to-end fail counts only when the same question fails 3 times out of 3. This is also why the short set stays small.
- Naming: "system tests" means the dev team's suite `Viedoc.Data.SystemTests`; the suite in this repo is called end-to-end tests.

What is known for step 4, for when it is planned:

- Old pipeline 919 "Viedoc.Data SystemTests" (project Viedoc.Data) cannot be reused as it is: its YAML (`SystemTests/pipelines/system-tests.yml`, branch `phuong/SystemTests`) was deleted when the suite moved to this repo on 2026-10-14, and it only ran the database specs, never the chatbot. Last run 2026-09-08.
- What 919 proved: the self-hosted pool `internaltest-agents` reaches the dev1 MySQL with a read-only secret account.
- No new repo is needed: the YAML goes in this repo, with a new pipeline in project Viedoc POC.
- Open: whether project Viedoc POC may use pool `internaltest-agents`; the Claude CLI and an API key on the agent (LLM cost per run); the MCP token runner `Scripts/mcp-user-tokens.spec.ts` needs a browser and the test account passwords; a trigger on 887 across projects.

### AC test coverage

Every acceptance criterion is covered by the dev team's acceptance tests in the Viedoc.Data repo; the end-to-end suite does not re-test ACs. An acceptance test calls the MCP tool directly, so a green one proves the tool returns the right data for that AC, not that the chatbot reaches the tool; the end-to-end suite covers that link.

- **Source:** the `Requirement` / `AcceptanceCriterion` test traits in `Viedoc.Data.AcceptanceTests`, compared with each PBI's Acceptance Criteria field in ADO.
- **Status:** Not measured. Needs the AC test coverage script (see Gaps). Today only PBIs 105523, 105540 and 105701 have acceptance tests.
- **Watch:** Share of ACs with at least one acceptance test (target 100%); ACs without a test, per PBI.

## Bugs and requirements

How the tickets are doing: bugs still open and questions to the team. Source: ADO work items.

### Open bugs and questions

- **Source:** ADO queries, shown on the Viedoc4 dashboard "[Draft] Viedoc.Data Quality".
- **Status:** Measured.
- **Watch:** Open bugs by state and age (bugs have no severity); open questions to the team and how long they wait for an answer.

## Product health (dev1)

How the running app on dev1 is doing, whatever the tests say. Sources: App Insights `appi-dat-dev1-swc` (readable with an `az` login, checked 2026-10-28) and the CDC checkpoints.

### Data freshness

- **Source:** App Insights metric `app.marten.projection.business.latency`.
- **Status:** Partly measured. The metric stops when the event is stored; the async read models the API reads (`FormReadModel`, `ItemReadModel`, `EventReadModel`, `QueryReadModel`) are not measured. It has no backfill tag and no distribution, so backfill skews the average and there is no true p95. Fixes are listed under Gaps; not raised with the dev team yet.
- **Watch:** Time from a Viedoc action until the MCP can read it.
- **Other signals** for the later stages, in Postgres:
  - Async projection lag: `mt_event_progression` against the high-water mark.
  - Age of the metric views: now − `mv_refresh_log.refreshed_at`, the `dataAsOf` the MCP returns.

### Sync status

Freshness measures how long each change took; it shows nothing when the harvest is stuck, because no events arrive to measure. Sync status shows that.

- **Source:** the CDC checkpoints in Table Storage, one per study and harvester (Study, Site, Subject, Form, Query, ...), readable through `ListCheckpoints` of the CDC API. Fields `CaughtUp`, `CaughtUpAt`, `AtHead`, `AtHeadChangedAt`, `EventsHarvested`.
- **States** (ADR-0042, meant for Cockpit; not checked in the Cockpit UI):

  | State | Rule | Meaning |
  | --- | --- | --- |
  | Pending | No checkpoint | Not started |
  | Backfilling | `!CaughtUp` | Still reading the old data; never caught up yet |
  | Lagging | `CaughtUp && !AtHead` | Caught up once, but the last poll found data not read yet |
  | Caught up | `CaughtUp && AtHead` | Read everything; keeping up with the source |

- **Status:** The state exists but is not on a dashboard. Read access to the dev1 storage account `stdatdev1swc` (role "Storage Table Data Reader") not checked.
- **Watch:** Share of (study, harvester) pairs that are Caught up; a study is in sync when all its harvesters are. Pairs that are Lagging, and for how long (now − `AtHeadChangedAt`).
- **Not available:** a sync percentage. `EventsHarvested` counts what was read, with no total to divide by. A percentage would need the read position in `CheckpointCursorJson` (e.g. `LastAuditId`) compared with the latest audit record of the study in MySQL; not verified.
- **Related check** for whether the data matches the source: counts per study (subjects, forms, queries) in MySQL against the read models.

### Failed or skipped events

- **Source:** App Insights counters `processor.events.duplicate_skipped` and `cdc.harvester.errors`.
- **Status:** Partly measured. Failed events are only logged; there is no `processor.events.failed` counter yet (see Gaps).
- **Watch:** Duplicates skipped by the Processor, CDC harvester errors, and failed events once counted.

### MCP errors per tool

- **Source:** App Insights requests and exceptions of the MCP app.
- **Status:** Not measured. A failing tool still returns HTTP 200 and every request is named `POST /mcp/`, so App Insights cannot show which tool fails (see Gaps).
- **Watch:** Error rate per MCP tool, once recorded.

## Gaps

Without these, some of the above stays green while dev1 is broken, or cannot be measured at all.

- [ ] Post-deploy smoke test on dev1: `deploy.yml` ends at "Deploy via deploy.ps1" with no check against the running app. In progress: Task 108676 "Add smoke test to the deployment pipeline", under TBI 107801 "Shift Left Testing".
- [ ] AC test coverage script: read the `Requirement` / `AcceptanceCriterion` traits and compare with each PBI's Acceptance Criteria field. Today only PBIs 105523, 105540 and 105701 have acceptance tests.
- [ ] End-to-end freshness (Viedoc action to read model updated): add a histogram next to `app.marten.projection.business.latency` in `Telemetry/MartenTelemetry.cs`, recorded as `now - SourceSystemChangeTime` when each async read-model projection (`FormReadModel`, `ItemReadModel`, `EventReadModel`, `QueryReadModel`) has applied the event. `SourceSystemChangeTime` is already on every event header (`EventProcessor.cs`). Today's metric stops at event append. Not raised with the dev team yet.
- [ ] MCP errors per tool: record the tool name and success or failure on each call (a request tag, or a metric such as `mcp.tool.calls` tagged `tool` and `success`). For the dev team; not raised yet.
- [ ] Data freshness accuracy: add a backfill tag to `app.marten.projection.business.latency` and record it as a histogram; can ship with the end-to-end freshness change in `MartenTelemetry.cs`. For the dev team; not raised yet.
- [ ] Failed events: add a `processor.events.failed` counter where the error is logged, so failures can be charted and alerted on; later, a dead-letter store to keep failed events for replay. For the dev team; not raised yet.
- [ ] Sync status on a dashboard: read the CDC checkpoints and chart the four states per study and harvester.
- [ ] End-to-end suite on a pipeline (step 4): wider coverage, and a run after each dev1 deploy with a pass rate for the dashboard.

## Sources

- Viedoc.Data repo, `origin/main`: pipelines in `.azuredevops/pipelines/`, test projects under `systems/*/tests/`, CDC checkpoints in `systems/cdc/` and `docs/adr/0042-harvester-self-reported-backfill-state.md`.
- ADO pipelines and test runs of project Viedoc.Data (861, 887, 919).
- App Insights `appi-dat-dev1-swc`.
- TBI [107801 — Shift Left Testing](https://dev.azure.com/pcgsolutions/Viedoc4/_workitems/edit/107801) and its child tasks.
- `SDLC_Software_Quality_and_AI_Adoption_Guide.pdf` (AI-generated, used only as ideas): DORA, static analysis, requirement volatility, defect rejection rate, AI code acceptance.
