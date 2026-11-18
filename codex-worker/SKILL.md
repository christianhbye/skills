---
name: codex-worker
description: Use when handing a task to a Codex worker (`codex exec`) from a Claude session, resuming one, or reading back its result — including choosing the Codex model, writing its brief, and checking whether it finished, stopped on a question, or hit a usage limit.
---

# Codex workers

A Codex worker is a background `codex exec` run with a written brief. The orchestrator (a Claude session) launches it, then audits its result against primary outputs before anything reaches trackers, PRs or CHB. Codex cannot launch Claude, reads `AGENTS.md` (never Claude memory), and must not write the project's trackers. Project specifics (products directory, product-location rules, tracker files) come from the project's `CLAUDE.md`/`AGENTS.md`, which override this skill where they differ.

## 1. Choose model, effort and thread

| Situation | Model | Effort (`-c model_reasoning_effort=…`) |
|---|---|---|
| Mechanical: path repoints, reruns, PR plumbing, register/README edits, dry runs | `gpt-6.2-sol` | `'"medium"'` |
| Most work: notebooks, analysis code, fixes, reviews | `gpt-6.2-sol` | `'"high"'` (the config default) |
| Particularly challenging (hard debugging, subtle math, new methods) | `gpt-6.2-sol` | `'"xhigh"'` |
| High-importance (CHB's call, or the headline calibration/science) | `gpt-6.2-astra` (the config default model) | `'"high"'` |
| **Very few cases only:** very important scientific work with a bounded scope (e.g. one derivation, one calibration step, one spec's math), not open-ended or multi-part tasks | `gpt-6.2-astra` | `'"xhigh"'` |

Always pass `-m` and the effort explicitly (CHB, 2026-11-18). Astra + xhigh is the most expensive combination: name the reason in the brief's header when you use it. Pick the effort by how hard the task is, the model by how much rides on it.

Thread: a new task gets a fresh `codex exec`; a short follow-up on the SAME task (answering its stop question, one fix) resumes with `codex exec resume <thread-id>` at the same model and effort. A new task gets a fresh thread even if an old thread "has context": each resumed turn re-reads the whole history (one resumed thread used ~2M tokens in a day, 2026-11-17). Thread ids: `~/.codex/sessions/YYYY/MM/DD/rollout-*-<id>.jsonl` (grep for the branch name). At most one or two Codex workers at a time.

## 2. Launch (exact form)

Write the brief to `<scratchpad>/<task>_prompt.md`, then run with the Bash tool, `run_in_background: true` (`<products>` = the project's products directory, e.g. `~/Documents/research/papers/eigsep_analysis/products`):

```bash
S=<scratchpad>; W=<worktree>
cd "$W" && OPENBLAS_NUM_THREADS=1 codex exec -m gpt-6.2-sol -c model_reasoning_effort='"high"' -C "$W" \
  -c sandbox_mode='"workspace-write"' -c sandbox_workspace_write.network_access=true \
  -c 'sandbox_workspace_write.writable_roots=["/home/christian/.cache/uv","/home/christian/.local/share/jupyter","/tmp","<products>"]' \
  -o "$S/<task>_final.md" - < "$S/<task>_prompt.md" > "$S/<task>_log.txt" 2>&1
```

- The products root is needed for any notebook or product write (add other output dirs the task writes outside the worktree); the uv cache and jupyter roots for `uv run` and kernels.
- `> log 2>&1` is required: limit and sandbox errors only appear there.
- `danger-full-access` is refused by Claude Code's classifier; never work around it.
- Resume form: `codex exec resume <thread-id> -m <same model> -c …(same -c flags)… -o … - < prompt > log 2>&1`, run from the worktree.
- Before launching: the worktree exists and is recorded where the project records branches (EIGSEP, MISTIC: `docs/branches.md`, updated on the workspace's `main` first); laptop load is sane; for shared-host compute the brief points at the project's compute rules (EIGSEP: the `limbo-compute` skill).

## 3. The brief: these parts, in this order, nothing else

1. **Task** — one paragraph: the problem and the deliverable (e.g. "a PR to main that …").
2. **Where** — worktree path, branch, base commit; "work only here".
3. **Read first** — `AGENTS.md` (workspace and repo) and any spec/plan path. Point at rules; do not restate them.
4. **Scope and decisions already made** — what changes, what must not change, choices CHB or the orchestrator already took (with the MOD/decision ref).
5. **Stop rules** — "if a modelling choice is open, or <specific condition>, stop and report the options instead of choosing."
6. **Finish** — tests (focused + full configured suite at the exact final head), commit/push/PR, PR body shape (problem / what changes / what doesn't / numbers / suite).
7. **Limits** — one BLAS thread, RAM cap, no workspace tracker edits, delete scratch.
8. **Final report** — "≤ N lines: PR number, head SHA, key numbers with the results.json fields they come from, anything stopped on."

Product locations follow the project's rule. In the EIGSEP workspace, a worker on a branch writes drafts to `products/_branches/<branch>/` (what `paths.product_dir()` does on a branch) and only a run on `main` writes canonical `products/NNN_*`: never tell a branch worker to write canonical products.

## 4. When it exits

1. Read `<task>_final.md`. Then `tail -40 <task>_log.txt`.
2. Classify:
   - **Usage limit** — log contains `You've hit your usage limit … try again at <date>` (final file often empty). Record the reset time in the logbook; the worktree may hold uncommitted partial edits: hand the rest to a Claude worker (brief it to review `git diff` first) or wait for the reset. Do not just relaunch Codex.
   - **Stopped on a question** — the report asks for a choice. Decide it if it is the orchestrator's (record it as a provisional decision under its MOD), otherwise ask CHB; then resume the same thread with the answer.
   - **Done** — continue to 3.
3. Audit against primary outputs, not the summary: `git status`, `git log`, `git diff --stat` on the branch; open the products' `results.json` and check the quoted numbers; look at the key figures; confirm the suite record (counts, exact head). Codex's arithmetic is usually right; its conclusions often are not.
4. Then the project's trackers (orchestrator only; EIGSEP: `docs/prs.md`, `docs/branches.md`, logbook, queue line); delete the scratch prompt/final/log files.

## Common mistakes

| Mistake | Fix |
|---|---|
| No `-m`, so astra (expensive) runs routine work | Always pass `-m gpt-6.2-sol` unless the work is high-importance |
| No effort flag, or xhigh by habit | Always pass the effort: medium mechanical, high default, xhigh only when particularly challenging; astra + xhigh only for very important, bounded-scope science |
| Resuming yesterday's long thread for a new task | Fresh thread; resume only for follow-ups on the same task |
| Missing products writable root | Use the exact launch block above |
| No log redirect; limit hit goes unnoticed | `> <task>_log.txt 2>&1`, grep it on exit |
| Brief restates AGENTS.md and the CI policy | Point at AGENTS.md; brief = the 8 parts above |
| Branch worker told to write canonical products | Follow the project's product-location rule (EIGSEP: branch → `products/_branches/<branch>/`) |
| Relaying the worker's summary to CHB unchecked | Audit step 3 first |
