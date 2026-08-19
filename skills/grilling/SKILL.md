---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

<!-- Local enhancement: upstream update check. Runs at most once per 7-day window, never blocks, never auto-updates. -->
## Update check (run before the interview starts)

Check for upstream updates before beginning, but keep it fast and non-blocking:

1. Read the `.source` file sitting next to this SKILL.md (same directory). It is a shared manifest covering this skill **and the four mattpocock/skills skills vendored under `skills/devflow/`** (domain-modeling, tdd, to-spec, to-tickets). If its `last_checked` is within the last 7 days, skip the check entirely and go straight to the interview.
2. Otherwise, for **every file in `tracked_files`**: fetch the remote file at its `upstream_raw_url` and compare it **content-for-content** against the local file — strip the local-enhancement block between the two comment markers, then diff the remote content against the remaining local upstream body. Do not rely on commit shas — compare the actual text.
3. Update `last_checked` to today's date (YYYY-MM-DD) in `.source` regardless of the result.
4. If any contents differ, do not just dump the diff. Fetch the official release notes / CHANGELOG (use `upstream_latest_release_url` and `upstream_changelog_url` from `.source`), find what changed for each affected skill, cross-check it against the actual content diff, then give the user a short recommendation per skill — what the change means, whether it's worth updating, and any risks. Flag any deliberately customised local files so the user knows the diff may be intentional. Let them decide. Never modify any SKILL.md without explicit user confirmation.
5. If the check fails for any reason (network, API, missing file), skip it silently and proceed — never block or delay the interview over an update check.

<!-- End of local enhancement. The rest of this file is the upstream skill body, do not edit unless the user asks. -->

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Each question should be formatted like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.
