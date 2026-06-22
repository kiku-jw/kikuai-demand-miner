# Opportunity Scanner Source Expansion

## Purpose

Phase 7 broadens discovery only after the ledger, labels, deep review,
aggregation, digest, and calibration layers work. Every new source must satisfy
the same candidate input contract and must preserve public-only provenance.

## GitLab Source MVP

Collect public GitLab projects:

```bash
python3 scripts/opportunity_scanner.py \
  --week 2026-W23 \
  gitlab-search \
  --search 'dashboard' \
  --max-candidates 10 \
  --issues-per-project 3
```

Collect and ingest:

```bash
python3 scripts/opportunity_scanner.py \
  --week 2026-W23 \
  gitlab-search \
  --search 'dashboard' \
  --max-candidates 10 \
  --issues-per-project 3 \
  --ingest
```

By default the output path is:

```text
data/sources/gitlab/<week>-candidates.jsonl
```

Set `GITLAB_TOKEN` or `GL_TOKEN` only to improve public API access. The
collector still uses `visibility=public` and records `auth_required=false`.

## Source Contract

The GitLab source uses public REST API endpoints:

- `GET /projects`
- `GET /projects/:id/issues`

Candidate metadata includes:

```yaml
raw_metadata:
  provider: gitlab
  id:
  path_with_namespace:
  repo_key:
  fork_family_key:
  forked_from_path_with_namespace:
  fork:
  visibility:
  archived:
  star_count:
  forks_count:
  open_issues_count:
  last_activity_at:
  topics:
  collection:
    api_surface: gitlab-rest
    api_version: v4
    visibility: public
    auth_required: false
    endpoint_kinds:
      - projects
      - project-issues
```

`repo_key` is normalized from `web_url`. `fork_family_key` uses
`forked_from_project.path_with_namespace` when available, then the project
itself.

## Caps

- `--max-candidates` limits candidates per run.
- `--per-page` is clamped to `1..100`.
- `--issues-per-project` limits issue excerpts.
- Projects whose API `visibility` is not `public` are skipped.

## Current Source Set

Implemented:

- GitHub public REST source.
- GitLab public REST source.

Future sources should be added only after a source contract doc and tests:

- curated lists / awesome lists
- ecosyste.ms or GH Archive
- marketplaces and plugin stores
- Reddit/Hacker News/Product Hunt only with specific query paths
- pasted research reports from Operator

## Verification

```bash
python3 -m py_compile scripts/opportunity_scanner.py tests/test_opportunity_scanner.py
python3 -m unittest discover -s tests
```

Covered invariants:

- GitLab source enforces public visibility
- GitLab source caps collection
- GitLab source maps candidates into the common contract
- GitLab source captures project issue excerpts
- GitLab source contributes repo/fork-family identity keys

Live smoke test performed on 2026-06-02:

```bash
python3 scripts/opportunity_scanner.py \
  --data-dir /tmp/opportunity-gitlab-live \
  --week 2026-W23 \
  gitlab-search \
  --search 'dashboard' \
  --max-candidates 1 \
  --per-page 1 \
  --issues-per-project 0 \
  --ingest
```

Result: one public GitLab project collected, ingested, and reported.

## Hacker News Demand Miner MVP

Mine public Hacker News discussions for repeated user pain before spending
frontier-agent time on interpretation:

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

By default the output paths are:

```text
data/sources/hn/<week>-demand-candidates.jsonl
data/reports/<week>-demand-miner.md
```

The collector uses the public Hacker News Firebase API:

- `/v0/askstories.json`
- `/v0/showstories.json`
- `/v0/newstories.json` when explicitly requested
- `/v0/item/<id>.json`

Candidate metadata includes:

```yaml
source: hn-demand
source_url: https://news.ycombinator.com/item?id=<story_id>
project_url: https://news.ycombinator.com/item?id=<story_id>
repository: ""
license: unknown
raw_metadata:
  provider: hacker-news
  source_type: demand-cluster
  cluster_id:
  story_ids:
  comment_ids:
  story_count:
  comment_count:
  points_total:
  score:
    total:
    verdict:
    dimensions:
    hard_reasons:
  collection:
    api_surface: hacker-news-api
    visibility: public
    auth_required: false
search_lanes:
  demand_pain_cluster: true
```

Scoring is intentionally conservative and transparent. Each dimension is
`0..2`:

- pain recurrence
- buyer clarity
- current workaround clarity
- no-call product angle
- async distribution hint
- legal/privacy safety
- not hype/novelty-only

Clusters scoring `10+` become candidates unless a hard reject fires. Clusters
scoring `7..9` stay report-only. Lower-scoring clusters are noise. Hard rejects
include sensitive/legal/financial/crypto/trading/copyright/downloader/ToS
patterns and unclear buyer signals.

Telegram behavior does not change. HN report-only clusters and raw candidates
do not appear in Telegram. The digest still sends only ideas that survive the
existing final filters as `proof-card`, `PRD-lite`, or Operator-approved proof.

Reddit remains parked until the source contract is stricter than "scan
subreddits." A Reddit source must define exact public endpoints, allowed
communities, rate limits, deleted-content handling, ToS boundaries, and
anti-spam distribution rules before implementation.
