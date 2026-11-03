# Hackday: Viedoc.Data quality board — presentation script

About 7–8 minutes. Open the board while presenting: https://claude.ai/artifact/56pXGDx3ee4EJAL1KHdRGt (share it with the team first).

## 1. Why (1 min)

How do you monitor the quality of a data platform?

That's the question I wanted to answer in this hackday. Two things matter to me here. First: when we release, how can we say we're confident, and back that up with numbers instead of a feeling? Second: how do we spot a quality problem early, while it's still cheap to fix, instead of finding it after a customer does?

For both, we need the right metrics. Metrics that tell us, quickly and clearly, whether the product is healthy.

But is "confident" just "all the tests pass"? A green build only says the tests we have passed. It doesn't say whether those tests cover enough, which parts are missing, or where we should add more. And tests don't see everything: data that arrives late or incomplete, a question nobody answered, or an app that is slow or throwing exceptions on dev1.

So to get a complete picture of quality, monitoring the tests is not enough on its own. Next to the tests, we need to watch other quality characteristics too: the quality of the data itself, our process around bugs and requirement clarification, and the issues, exceptions and performance that App Insights already records.

So in this hackday I tried to find those points: what we can measure to see the health and the quality of Viedoc.Data. I went through all our pipelines, the test projects, App Insights and the CDC code.

Then I put everything I found on one dashboard. That's the part I care about most. Today this information is spread out: test results in one pipeline, errors in App Insights, bugs and questions in ADO. Each of us sees a piece of it, and nobody sees the whole thing. With one dashboard, the whole team looks at the same picture. We all know where the quality stands today, we spot a problem as soon as it shows up, and we can act on it early, together.

I split the monitoring into four main areas:

- The first part, **Tests**, focuses on monitoring the tests in our system: where they run, whether they pass, and whether every acceptance criterion has one.
- The second part, **Data quality**, focuses on the data itself: is what the MCP returns right, complete and up to date with Viedoc.
- The third part, **Bugs and requirements**, focuses on our tickets: the bugs and the questions that are still open.
- The fourth part, **Product health**, focuses on the running app on dev1: errors, slow responses and events that go wrong, whatever the tests say.

Everything on this page is mock data. The layout is what matters. The dashed grey panels are things we can't measure yet; some of them show an example of what they would look like.

## 2. The summary at the top (20 s)

The first row is the summary: a quick update on all four areas, one card each. Each card has a status label and the few numbers that matter most. Green means fine, orange or red means look closer, grey and dashed means not measured yet. If you only have ten seconds, this row is what you read.

Below it, each area has its own part with the details. I'll go through them one by one, from top to bottom.

## 3. Tests (2 min)

*Overview:* this part answers "do our tests run, and do they pass?" At the top you see four cards, one per point where we test, left to right in the order a change goes through them. Each card shows the pipeline, its recent runs as small bars, and the pass rate. Below them: the test count per suite, and AC test coverage.

We test at four points, from a pull request to the running app.

- **Before merge to main**: pipeline 861 runs all the dev tests on every PR. That's unit, integration, system, acceptance and convention tests, plus vitest for Cockpit.
- **Before each deploy**: 887 runs the same tests again. It's the last gate before deploy, and the only one that tests the exact code we ship.
- **Right after each deploy**: nothing yet. A deploy ends at "Deploy via deploy.ps1" and nobody checks that the app actually runs. Task 108676 covers this. The card shows an example of what we'd see.
- **On dev1, through the chatbot**: the end-to-end suite asks real questions as test users and checks the answers against MySQL. Today we run it by hand, per ticket.

For each step we watch pass rate, run duration, and how the test count per suite changed over the last 14 days. A drop with a green build means someone deleted tests.

Two things worth knowing:

- A skipped test fails the build, so a green run always has zero skips.
- On October 30 the deploy pipeline timed out three times in a row in the Test step. Those runs published no .NET results, so a test trend chart would look fine for those runs. That's why run duration is on the board.

Last in this part, **AC test coverage**: every acceptance criterion should have an acceptance test. Eight work items have acceptance tests today. We can check coverage straight from the code, without running anything: each acceptance test is tagged with its PBI and AC number, so we compare those tags with the AC list on each PBI. The documentation pipeline already does this, but only for the PBI someone runs it for.

The headline counts PBIs whose In Sprint State is Done, because a PBI still in work can lack tests. The list shows only the PBIs with a gap, Done ones first: those went out without a test for every AC.

## 4. Data quality (1.5 min)

*Overview:* this part answers "can people trust the data the MCP gives them?" Four panels: correctness and completeness side by side, then freshness and sync status.

The data is what this product delivers, so it gets its own part. Three questions: is the data right, is it complete, and is it current? Freshness and sync status both answer the last one.

- **Correctness**: do the chatbot's answers match MySQL? The end-to-end suite checks this, but only when we run it by hand.
- **Completeness**: did every record arrive? An hourly check, with no chatbot, compares every key in MySQL with the read models, for the studies that are caught up. A mismatch that lasts several runs is a bug, not a change still on its way. Nobody does this yet; the panel shows an example.
- **Freshness**: the AC says a change in Viedoc must be readable through the MCP within 5 minutes. So the target is 100% within 5 minutes, and we count the changes that were late. Today's metric stops when the event is stored. It doesn't reach the read models the API uses, and it can't tell backfill from live changes.
- **Sync status**: freshness says nothing when the harvest is stuck, because no events arrive to measure. CDC already stores a state per study and harvester: pending, backfilling, lagging or caught up. We just need to show it.

## 5. Bugs and questions (30 s)

*Overview:* this part answers "what is still open on our tickets?" Two tables side by side: open bugs on the left, questions waiting for the team on the right.

These are open bugs and open questions to the team, oldest first, with how long each has waited. This one already works from ADO queries.

## 6. Product health on dev1 (1 min)

*Overview:* this part answers "is the running app on dev1 working right now?" Four panels: failed events and harvester errors, duplicate events, MCP response time, and MCP errors per tool.

This part is about whether the running app works, whatever the tests say.

- **Failed events and harvester errors**: we count harvester errors. Failed events in the Processor are only logged, there's no counter yet.
- **Duplicate events**: the Processor counts the duplicate events it drops. A jump there means the CDC is sending the same changes again.
- **MCP response time**: how long a user waits. App Insights already has it: half the requests take under 0.2 s, but 1 in 20 takes over 14 s. We should agree on a limit, for example 10 seconds, and watch the share of requests under it.
- **MCP errors per tool**: a failing tool still returns HTTP 200, and every request is named `POST /mcp/`. So we can't see which tool fails.

## 7. Next steps (1 min)

*Overview:* the last section on the board, "Not measured yet", lists every gap that keeps a panel grey or partial.

If the team sees the value in this and agrees, this is how we can monitor the quality of Viedoc.Data. It's not complete yet: there are still gaps, and some of them need updates on the developer side, for example:

1. A smoke test after deploy (already planned in 108676).
2. Record the lag of each change up to the read models, with a backfill tag.
3. A `processor.events.failed` counter.
4. The tool name and success on each MCP call.

We'd also need to agree on how long a user may wait for an answer, so the response time panel has a limit to measure against.

On the testing side, the next pieces are the AC test coverage script, the hourly completeness check, and running the end-to-end suite on a pipeline after each dev1 deploy.

## 8. Close (15 s)

The details are in `Viedoc.Data.Testing/docs/QUALITY_KPIS.md` in the PlaywrightAutomationTest repo. I'd love feedback on what's missing, and what you'd actually look at every week.
