# Work Log By Cursor

*Auto-recording your work with Cursor for self review*

It helps in self reviews, 1:1s and status updates. It takes about 2 minutes to set up, and you can backfill your last 6 months.

*Add the rule*
In Cursor, go to *Settings → Rules → User Rules → Add Rule* and paste this:

```
# Worklog Maintenance Rule

When you finish implementing a feature, fix, or significant refactor:
1. Append an entry to `vishal/WORKLOG.md` (create it if it does not exist).
2. Keep the entry concise, focusing on **impact** and **context**, not raw code lines:
 - **Date**: YYYY-MM-DD
 - **service**: service name
 - **Task / Ticket**: Short title or Jira/Issue ID if known.
 - **Problem Solved**: 1 sentence on why this was broken or needed.
 - **What Was Done**: 1-2 bullet points.
 - **Impact**: Expected result (e.g., "reduces query latency", "prevents double-billing edge case").

```

From then on, every time Cursor finishes a feature, fix or refactor with you, it adds an entry to `~/dev/docs/WORKLOG.md`. Change the path if your repos aren't under `~/dev`.

*Backfill your past work (one time)*
First, log in to GitHub CLI in your own terminal so Cursor can read your merged PRs
Then open Cursor in the folder that holds all your repos (for example `~/code`) and paste this prompt into Agent:

```
Backfill my worklog at ~/dev/docs/WORKLOG.md with the features, bug fixes, refactors and docs I did in the last 6 months, using the format from my Worklog rule.
Sources:
1. My merged PRs across all repos: gh search prs --author @me --merged --merged-at ">=<6-months-ago-date>" --limit 200 --json number,title,repository,closedAt,body,url
2. My git commits in every repo under ~/dev (git log --all --author=<my email> --since=<date> --no-merges)
3. My past Cursor chat transcripts for this workspace
Merge them into one entry per logical task, sort oldest first, tag each entry [Feature]/[Bug]/[Refactor]/[Docs]/[Chore/Security], and don't make anything up. At the end, list anything I should double-check.

```

* Review the result*

*Why bother?* When review season comes, you already have a dated list of what you did, why it mattered and the impact. No more digging through Jira and Git the night before.