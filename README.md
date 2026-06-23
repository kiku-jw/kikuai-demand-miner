# KikuAI Demand Miner

Local-first demand miner for public discussion signals. The current MVP mines
public Hacker News stories and comments for repeated user pain, writes a demand
report, and can emit only strong clusters into the scanner candidate format.

This is discovery tooling, not a money predictor. A demand cluster is evidence
that people discussed a pain. It still needs normal proof-card validation before
any build decision.

Primary CTA: run a capped Hacker News demand report locally.

```bash
python3 scripts/opportunity_scanner.py --week 2026-W24 hn-demand --max-stories 80 --comments-per-story 20 --max-clusters 10 --max-candidates 5
```

Expected result: a Markdown demand report and optional JSONL scanner candidates
under `data/`.

## What It Does

- Reads public Hacker News feeds through the official Firebase API.
- Uses capped story/comment traversal.
- Skips deleted and dead HN items.
- Extracts complaint, workaround, question, and tool-seeking phrases.
- Clusters pain at thread level to avoid one thread flooding the queue.
- Scores clusters across recurrence, buyer clarity, workaround clarity,
  no-call product angle, async distribution hint, legal/privacy safety, and
  hype risk.
- Writes a human Markdown report.
- Optionally emits strong clusters as JSONL candidates for the scanner ledger.

## Quick Start

Run a report without ingesting candidates:

```bash
python3 scripts/opportunity_scanner.py \
  --week 2026-W24 \
  hn-demand \
  --max-stories 80 \
  --comments-per-story 20 \
  --max-clusters 10 \
  --max-candidates 5
```

Collect and ingest only strong clusters:

```bash
python3 scripts/opportunity_scanner.py \
  --week 2026-W24 \
  hn-demand \
  --max-stories 80 \
  --comments-per-story 20 \
  --max-clusters 10 \
  --max-candidates 5 \
  --ingest
```

Run tests:

```bash
python3 -m unittest discover -s tests
```

## Outputs

```text
data/sources/hn/<week>-demand-candidates.jsonl
data/reports/<week>-demand-miner.md
```

Runtime `data/` is ignored by Git.

The report is a triage artifact. It does not validate payment intent, legal
fit, distribution, or whether the product should be built.

## Source Boundaries

- Public Hacker News API only.
- No hidden scraping.
- No private data.
- No Telegram sending by default.
- Report-only clusters do not become recommendations.

## License

MIT.
