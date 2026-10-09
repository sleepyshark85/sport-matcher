# Delivery

Everything that takes a Viedoc release from a merge on `main` to a deployment in all four
production regions: the release pipelines, the declarations they read, and the scripts they run.

**Nothing here is live yet.** The repository is being built beside the current release process,
not on top of it. `Infrastructure` keeps running releases until a real release has gone out on
what is in here.

## Two lanes, and which one you are in

The repository holds two kinds of thing, and the difference is what happens on 2026-10-31.

| | | |
|---|---|---|
| **The machinery** | `pipelines/` `docs/` `tools/` `DECISIONS.md` `AZURE-DEVOPS-AUDIT.md` `audit/` | Permanent. It is what a release runs on |
| **The project** | [`project-daybreak/`](project-daybreak/) | Deleted when Project Daybreak ends, 2026-10-31 |

So two things hold, and `tools/Test-DocumentLinks.ps1` enforces both:

- **Nothing in the machinery points at `project-daybreak/`.** A link into a folder that gets
  deleted is a link that dies. Name the document in prose instead. This file, `CLAUDE.md` and
  `DECISIONS.md` are the exception, because describing both lanes is their job.
- **A document is authored in the lane it belongs to.** Nothing is copied between them. If a
  document turns out to matter after October, it moves to `docs/` and stops being a project file.

Each folder has a short `README.md` saying what goes in it and how it is arranged. Read that before
adding a file.

## Where things are going

```
pipelines/        the pipelines, their templates, scripts, declarations and contracts, and the status site in
                  pipelines/status/ - see pipelines/README.md
docs/             how the machinery is designed — the plan, the pipelines, the specs, the documents
tools/            repository checks, run from a laptop
project-daybreak/ the project's working folder, deleted 2026-10-31
```

**The Bicep lives in a repository of its own, AzureInfrastructure**, with the build that compiles it
into a package. This repository holds the pipelines that apply
the pinned package and check it (`pipelines/README.md`, *The infrastructure path*). It never mixes into
`pipelines/`.

## Two rules that decide how code gets here

**Dependencies point in, never out.** A file we take from `Infrastructure`, `Trialgateway` or
anywhere else is copied in and owned here — nothing in this repository points outward. Product
repositories pointing *at* `Delivery` is the opposite case and is how one definition serves 100+
repos. Taking a template means taking its whole relative-path closure.

**All the pipelines exist from day one.** Every pipeline is created as a shell that runs and
prints the manual step it will one day replace. The chain runs end to end before any of it is
real, and teams replace one shell at a time.

Both are in [`DECISIONS.md`](DECISIONS.md) with the rest, which is where a change is
held to them. **`main` is protected** — every change reaches it through a pull request approved by
the Daybreak core team.

## What to read

| | |
|---|---|
| [`DECISIONS.md`](DECISIONS.md) | The rules every change is held to, and the five things a product team owes. Start here |
| [`docs/PLAN.md`](docs/PLAN.md) | The project: work streams, owners, milestones, what is out of scope |
| [`docs/pipelines.md`](docs/pipelines.md) | What each pipeline is for and the gate it enforces. The list itself is [`pipelines/pipelines.json`](pipelines/pipelines.json), and the order is [`pipelines/run-order.md`](pipelines/run-order.md) |
| [`docs/pipeline-specs.md`](docs/pipeline-specs.md) | One spec per pipeline: stages, scripts, arguments, artifacts, gates |
| [`docs/documents.md`](docs/documents.md) | Every document a release produces, and which pipeline will generate it |
| [`AZURE-DEVOPS-AUDIT.md`](AZURE-DEVOPS-AUDIT.md) | Every change made to shared Azure DevOps state: before, after and why |
| [`audit/`](audit/) | The lists `AZURE-DEVOPS-AUDIT.md` refers to and cannot hold inline, one file per measurement |
| [`docs/db-migrations/`](docs/db-migrations/) | Database migrations as they actually run, per package — what is automated, what isn't, and the convention `db.migrate` runs. One folder per task from here on: `docs/<task>/` |
| [`project-daybreak/`](project-daybreak/) | The project: how we got here, what is open, and the work stream documents |

## The project folder

[`project-daybreak/`](project-daybreak/) is everything Daybreak produced that a release does not
run: [`TODO.md`](project-daybreak/TODO.md) is the one live list of open items,
[`streams/`](project-daybreak/streams/) is a working document per work stream,
[`meetings/`](project-daybreak/meetings/) holds the kick-off and every session since, and
[`shaping/`](project-daybreak/shaping/) is the analysis the plan was built from.

**Text only.** The wall photographs, the decks, the meeting recordings and the signed QA documents
stayed on the machines they were made on. The notes link to them anyway, because naming the file is
how you ask for it. Ask Majd for the file.

**It is deleted at the end of October 2026**, which is why nothing in the machinery points at it.

Project Daybreak runs 1 September – 31 October 2026.
