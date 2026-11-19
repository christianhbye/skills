# skills

Agent procedures and portable rules, one directory per skill with a `SKILL.md`
(frontmatter `name` and `description`, then the steps). This directory,
`~/skills`, is the one copy. Each skill is symlinked into `~/.agents/skills`
(Codex), `~/.claude/skills` (`claude`) and `~/.claude-ral/skills`
(`claude-ral`), so every session in every repo loads it. A new skill needs the
same three links:

    n=<name>; for t in ~/.agents/skills ~/.claude/skills ~/.claude-ral/skills; do ln -sfn ~/skills/$n $t/$n; done

A skill's helper scripts sit in its `scripts/` directory; run them by path
(`python ~/skills/<skill>/scripts/<script>.py`), or put that directory on
`sys.path` to import them. Project settings (a project's `AGENTS.md` or
`CLAUDE.md`, e.g. its § Memos) override a skill wherever the two differ.

| Skill | Use it to |
|---|---|
| `21cm-analysis` | reduce data and make claims: weighted means, radiometer noise, model comparison, simulations, validation, scope (portable) |
| `21cm-figures` | make figures: axes, scales, colour maps, model-comparison panels, waterfalls, printed text size, captions (portable) |
| `codex-worker` | hand a task to a Codex worker from a Claude session: model and effort choice (`gpt-6.2-sol` at high by default; astra and xhigh by importance and difficulty), exact launch, brief shape, usage-limit and stop handling, audit before trusting |
| `technical-prose` | write and revise document text: direct, said once, quantitative; signs of padding and unvetted generated text (portable) |
| `latex-audit` | review a LaTeX memo, paper or proposal and report its problems without editing (portable) |
| `latex-page-fit` | bring a LaTeX document within a page limit without losing content or breaking format rules (portable) |
| `latex-memo` | write, change and rebuild a memo: generated figures and numbers, provenance, branch → build → checks → merge (portable; project settings in the project's § Memos) |
| `guided-pr-review` | review a PR you made with the user, one finding or decision at a time, then apply the outcomes and update the PR (portable) |
| `hfss` | run Ansys HFSS/AEDT headless: reuse existing exports, private hashed copies, the non-graphical launch recipe, report-only antenna-parameter export, done markers over exit codes (portable; host and budget in the project's settings) |
