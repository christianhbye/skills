---
name: guided-pr-review
description: Walk the user through a PR you made, one finding or decision at a time, in an iterative session: a one-screen summary with the decisions first, then each item with its evidence and a recommendation, recording each outcome, then applying them and updating the PR. Use after opening or updating a PR (or any reviewable change) when the user wants to review it with you rather than read it alone.
---

# Guided PR review

The user reviews by discussing, not by reading a long report. Lead with what
they must decide, take one item at a time, and let them steer. You do not
review for them: you present, they decide.

## Before the session

1. **Know the change cold.** Read the diff, the spec or description, the
   review findings (independent reviewers, conformance checks, your own known
   gaps) and the primary outputs behind every number you will quote
   (notebook cell, test log, committed file). Re-derive each headline claim
   from the primary output; a subagent's or earlier summary's number is a
   hypothesis until you have. Fix or flag any claim that does not trace.
2. **Build the item list.** One item per decision or finding the user could
   reasonably want to overrule. Drop what passes and what no one would
   decide. Merge items that are the same decision. Order by dependency (a
   choice that changes later items comes first), then by consequence.
   Classify each as **decision** (the user picks: a scope, a threshold, a
   post-hoc rule, a claim's wording), **finding** (something wrong or at risk,
   with a proposed fix), or **note** (for the record; mention only if cheap).
3. **Check the state is reviewable.** The branch is pushed, the PR description
   matches the head, CI status is known, and the working tree is clean. If
   not, say so first and fix it.

## The opening (one screen)

Give, in this order, and nothing else:

- one or two sentences on what the PR is and its label (diagnostic,
  provisional, validated);
- the **decisions and findings, as a numbered list of one-line titles** with
  your recommendation next to each ("1. Keep the 10:28 epoch in the primary
  set (keep)"), so the user sees the whole agenda at once;
- anything blocking a merge (failing checks, unverified claims, missing
  outputs);
- "Start at 1?"

Do not paste the full findings, the diff summary, or the results table here.

## Each item

Present one item, then stop and wait. For each:

- **The question or finding** in a sentence.
- **Evidence**: the number, output, file and line, or test that shows it. Name
  where the user can look. Quote only what the item needs.
- **Options** (two or three, only real ones) and **your recommendation**, with
  the reason and what would change it. If the user's earlier answers decide
  this item, say so and propose the consequence rather than asking again.
- **What happens if they say yes**: the concrete edit, rerun or text change.

Then wait. Answer follow-up questions from evidence, and if you do not know,
check rather than guess. If the user's answer changes the plan (a new item, a
dropped item, a different order), say so and update the agenda. Never bundle
the next item into the same message.

Keep a running **decision log** in the session: item, outcome, who decided,
what it implies. If the user says "do what you recommend for the rest", do so,
and list the defaults you took at the end.

## After the last item

1. **Apply the decisions**, on the PR's branch: code, notebook, text and
   spec changes, rerun what the changes invalidate, run the checks the project
   requires. Do not push or merge beyond what the user has authorised for
   this repo; if it is a shared repo, ask first.
2. **Update the PR description** so it states the final state and the
   decisions taken (and the ones deferred). Update any trackers or registers
   the project keeps (queue, logbook, PR list, notebook register) in the same
   pass, and add the decision log where the project records decisions.
3. **Say what is left**: unmerged, unverified, deferred, and what the user
   must do. If they have approved the merge, say it is theirs to run unless
   they asked you to.

## Rules

- One item at a time. A wall of findings is skimmed; a single decision is
  answered.
- Recommendations are yours; decisions are theirs. Do not treat silence,
  a question, or a tool notification as approval.
- Do not hide the weak parts: lead with a post-hoc rule, a skipped gate or an
  unverified claim rather than burying it.
- Keep the opening short and the items shorter than the evidence behind them.
- Project rules override this skill where they differ (for example, who may
  edit the trackers, whether a shared repo needs the user's check before a
  push, how decisions are recorded).
