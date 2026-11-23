# Plan: Run the Weekly Search & Embeddings Review from a DGX Spark

Status: proposal. Date: 2026-11-23.

## Why move
The cloud routine's network policy blocks `arxiv.org` and `xcancel.com`, so reports rely on web-search snippets and cannot read cs.IR listings or the `@bclavie` / `@jobergum` timelines. A Spark on a normal network removes that limit.

## Goals
- Weekly dated report `YYYY-MM-DD-search-and-embeddings.md` committed and pushed to `falloutdurham/weeklyreview` (`master`).
- Direct reads of: arXiv cs.IR (new + recent), `@bclavie`, `@jobergum`, plus a few extra accounts.
- Phone notification when the report lands, and on failure.
- Same focus: late interaction, token pruning/compression, query generation, embeddings.

## Non-goals
- Replacing the report format (keep last week's structure plus a "new since last week" delta).
- Anything that needs the Spark to be reachable from the internet.

## Architecture
```
systemd timer (Mon 07:00)
  -> run-weekly.sh
       1. git pull
       2. fetch.py        (arXiv API + xcancel -> JSON in work dir)
       3. [optional] prefilter.py  (local LLM scores abstracts, keeps top ~40)
       4. claude -p       (reads JSON + previous report, writes new report)
       5. git add/commit/push
       6. notify.sh       (ntfy / Telegram / email)
```

## Components

### 1. Setup (one-off)
- Install Claude Code CLI on the Spark and log in.
- Clone the repo to `~/weeklyreview`; add a deploy key (write access, this repo only) or a fine-grained token.
- Python env with `requests`, `feedparser` (or `arxiv`), `beautifulsoup4`.
- Pick a notifier. Recommended: ntfy (self-hostable, one curl). Alternatives: Telegram bot, SMTP.

### 2. Data fetch (`fetch.py`)
- **arXiv:** use the API (`export.arxiv.org/api/query`), `cat:cs.IR` sorted by submittedDate, last 7-8 days, plus keyword queries (`late interaction`, `ColBERT`, `multi-vector`, `token pruning`, `query expansion`, `query generation`, `ColPali`, `embedding`). Respect the 3-second delay between requests. Dedupe by arXiv id, save title/abstract/authors/links.
- **Twitter via xcancel:** fetch `https://xcancel.com/<user>` (and `/with_replies` if useful) for each account. Retry with a fallback Nitter instance list; if all fail, record the failure in the JSON rather than aborting.
- **Accounts:** `bclavie`, `jobergum`, plus candidates to confirm: `lateinteraction`, `mixedbreadai`, `jxmnop`, `nthngdy`, `antoine_chaffin` (PyLate), `lintool` (Jimmy Lin), `tomaarsen`, `thien_tran`. Keep the list in `accounts.txt`.
- Output: `work/YYYY-MM-DD/{arxiv.json,tweets.json,fetch.log}`.

### 3. Optional local pre-filter (`prefilter.py`)
- cs.IR gets roughly 150-250 papers a week; sending all abstracts to Claude is wasteful.
- Run a local model on the Spark (e.g. via Ollama or vLLM) to score each abstract 0-5 against the focus topics and keep the top ~40, always keeping keyword hits.
- Skip this step initially; add it if token use is a concern. The prompt must work without it.

### 4. Report generation
- `claude -p` with the prompt stored in `prompt.md` (the existing scheduled prompt plus: "inputs are in `work/<date>/`; do not repeat items from the previous report; flag anything you could not verify").
- Restrict permissions: allow Read/Write in the repo and work dir, Bash for `git` only; no arbitrary network tools needed because fetching is done beforehand.
- Run in a dedicated directory/user so unattended permissions stay narrow.

### 5. Publish
- Commit with a plain message, push to `master`. Fail loudly if the working tree is dirty or push is rejected.

### 6. Notify
- Success: one-line summary plus the top 3 papers and the report link.
- Failure: stage that failed and last 20 lines of the log.
- Silence is not acceptable on failure; use a `trap ERR` in the script.

## Scheduling
- systemd user timer, `OnCalendar=Mon *-*-* 07:00`, `Persistent=true` so a missed run (box off) fires on next boot.
- Service has a 30-minute timeout.

## Risks and mitigations
| Risk | Mitigation |
|---|---|
| xcancel blocks bots / is down | Fallback instances; report states which feeds failed |
| arXiv API rate limits | Single sequential client, 3 s delay, cache by id |
| Spark off or asleep | `Persistent=true`; disable suspend; notify on staleness (cron check for no report in 9 days) |
| Hallucinated papers | Prompt requires every item to come from the fetched JSON; include arXiv ids and verify they exist in `arxiv.json` before commit |
| Credential exposure | Repo-scoped deploy key; secrets in `~/.config`, not in the repo |
| Unattended agent doing too much | Narrow permission allowlist; separate OS user |

## Milestones
1. Manual run: `fetch.py` produces JSON; confirm xcancel and arXiv both work from the Spark.
2. Manual `claude -p` run produces a report matching the current format.
3. Add commit/push and notifier; dry run to a test branch.
4. Enable timer; watch two weeks.
5. Decide on pre-filter and extra accounts based on report quality and token cost.

## Open questions
- Notifier preference (ntfy / Telegram / email)?
- Keep the cloud routine as a backup, or disable it once the Spark works?
- Which extra Twitter accounts to follow?
- Use a local model for pre-filtering, or send everything to Claude?
- Should the report also be published as an artifact, or stay as markdown in the repo?
