# cloud-itonami-lei-apvbkpoulhux7yzlku17

**独立した第三者による分析/アーカイブであり、Antero Resources Corporation と提携・後援関係にありません。**
This is an independent third-party archive/analysis. Not affiliated with,
endorsed by, or sponsored by Antero Resources Corporation.

## What this is

A per-company reference/archive repository in the `cloud-itonami-lei-*`
family (ADR-2607110300, [com-junkawasaki/root design
rationale](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607110300-cloud-itonami-lei-corporate-tos-catalog.md)).
It archives Antero Resources Corporation's publicly published Terms of Service / Terms of Use
text, keyed by the company's ISO 17442 Legal Entity Identifier (LEI), with
full source-url + retrieved-at + sha256 provenance for every entry so the
document's revision history can be tracked over time via git history.

The archive itself (`blueprint.edn`, `NOTICE`, `80-data/public/tos.journal.edn`) is a
**read-only reference/archive** — it proposes or executes nothing on Antero Resources
Corporation's behalf.

**As of 2026-07-24 this repo ALSO carries a governed actor layered on top of that
archive** (`src/tosmonitor/*`, ToSMonitor-LLM ⊣ ToSArchiveGovernor), added as part of a
10-repo validation batch extending the pilot on
`cloud-itonami-lei-2572ibtt8cczw6au4141` (P&G)
([ADR-2607241900](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607241900-cloud-itonami-lei-tos-monitor-actor-pilot.edn),
[ADR-2607242000](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607242000-cloud-itonami-lei-tos-monitor-actor-batch10.edn),
repo-local record: [`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md)).
This does NOT change the archive's own read-only nature above, and it does NOT extend
to every other `cloud-itonami-lei-*` repository — most repos in this family remain
plain archives exactly as ADR-2607110300 describes. See `## Governed actor` below.

## Company identity

| Field | Value |
|---|---|
| Legal name | Antero Resources Corporation |
| LEI | [APVBKPOULHUX7YZLKU17](https://search.gleif.org/#/record/APVBKPOULHUX7YZLKU17) |
| Jurisdiction | US-DE |
| Website | https://www.anteroresources.com |
| Ticker | NYSE:AR |
| Industry (this repo's discovery context) | ISIC 0620 Natural gas extraction |

## Data

See `80-data/public/tos.journal.edn` — an EDN quad-log
`[<doc-id> :attr value <tx> :add]`. See `NOTICE` for copyright/attribution
of the archived third-party text.

## Governed actor

**ToSMonitor-LLM ⊣ ToSArchiveGovernor**, built on this workspace's
[`langgraph-clj`](https://github.com/com-junkawasaki/langgraph-clj) StateGraph runtime,
modeled directly on
[`cloud-itonami-commitment-ledger`](https://github.com/cloud-itonami/cloud-itonami-commitment-ledger)'s
Store/Registry/Advisor/Governor/Phase/Operation/Sim shape (byte-identical code to the
pilot and every other repo in this batch except `tosmonitor.store`'s company/baseline
data).

Given a candidate ToS/legal-document snapshot, the actor proposes whether it materially
diverges from the archived baseline above, and drafts a change summary — but **it never
writes to the archive itself.** `tosmonitor.store`'s `commit-record!` only writes to
this actor's own Store (an in-process ledger), never to `80-data/public/
tos.journal.edn`. A human archivist decides whether to fold a proposal into the real
archive.

**Single actuation, `:tos/change-proposal`, never autonomous.** Every run — whether the
candidate matches the baseline or diverges from it — carries `:stake :actuation/
archive-update` and is permanently excluded from every rollout phase's auto-commit set.
Six HARD governor checks, each independently re-verified rather than taken on the
advisor's self-report:

| Check | Guards against |
|---|---|
| `grounding-violations` | a cited excerpt not actually present in the candidate's own text (hallucinated quotes) |
| `provenance-incomplete-violations` | a candidate missing full-text/source-url/retrieved-at/sha256 |
| `sha256-mismatch-violations` | a candidate's self-reported sha256 not matching its own content (ground-truth recompute, `:clj`-only in V1) |
| `retrieved-at-not-advancing-violations` | proposing a change from input staler than what is already archived |
| `doc-type-unknown-violations` | a doc-type outside this archive family's own observed vocabulary |
| `source-domain-mismatch-violations` | this actor's own distinctive check — a source-url that doesn't belong to the archived company (misattribution risk specific to an LEI-keyed independent archive) |

```bash
kbb -M:dev:run     # clean lifecycle + all six HARD-hold checks + a phase-0 hold + a MemStore->DatomicStore swap
kbb -M:dev:test    # governor contract · phase invariants · store parity · advisor smoke
kbb -M:lint        # clj-kondo (errors fail; CI mirrors this)
```

`kbb -M:dev:run` and the test suite always use the deterministic mock-advisor — no
live fetch of the company's current ToS page and no live `kotoba-server`/CACAO publish
happen anywhere in this actor. `tosmonitor.advisor/llm-advisor` exists as a written,
swappable seam but is not invoked. See
[`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md) for the full
design record.

## License

Repository structure: AGPL-3.0-or-later (see LICENSE). Archived third-party
ToS text: copyright Antero Resources Corporation (see NOTICE).
