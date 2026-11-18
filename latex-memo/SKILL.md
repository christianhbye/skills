---
name: latex-memo
description: Write, change and rebuild a LaTeX memo so that its figures, quoted numbers and PDF all trace to code that regenerates them, with clean provenance. Use when creating a memo, editing its text or the script or notebooks that produce it, or when an input it reads changes.
---

# LaTeX memos

A memo records what a project has learned. Each figure and quoted number is
produced by code, so the text, the figures and the data can't drift apart.
This skill is portable. Each project's agent instructions (`AGENTS.md` or
`CLAUDE.md`) carry a § Memos, or point from there to the file that holds the
project's memo settings:

- the Python environment and the TeX engine, with the exact commands;
- where the data live, and any environment variables the scripts read;
- the memo layout and numbering (directory and file names);
- the **products generator**: a products script in the memo directory, or
  the notebooks or scripts elsewhere that write the memo's figures and numbers;
- the provenance the memo must carry;
- the bibliography: a shared `.bib`, or `thebibliography` per memo;
- the macro that marks open points (`\placeholder{}`, `\todo{}`, ...);
- the revision model: **living** (rewrite in place) or **append-only**
  (amend, or supersede with a new memo);
- the repository layout: whether there is a shared checkout or a hub remote,
  and what a worktree needs that git does not carry (gitignored data, sibling
  repositories), and any record of branches and worktrees to update;
- the commit and merge path: who approves, and whether a merge is a local
  fast-forward or a pull request the owner merges.

**Project settings override this skill** wherever they differ. The defaults
below apply only where the project says nothing. If a setting is missing and
the default doesn't obviously fit, ask, then add it to the project's § Memos.

Claims in a memo follow `21cm-analysis`, figures follow `21cm-figures`, and the
text follows `technical-prose`.

## Layout

Default: each memo is a directory `memos/memo-NNN-<topic>/` holding:

- `memo-NNN.tex`, and the committed `memo-NNN.pdf`. The number goes on the
  title page, so a circulated PDF identifies itself.
- **One products script**, `make_figures.py` (some projects call it
  `make_products.py`). It writes everything the memo shows:
  - `figures/*.pdf`, as vector figures;
  - `numbers.tex`, with `numbers.json` beside it (some projects put both in
    `tables/`): a `\newcommand` for every number the text quotes;
  - any generated tables;
  - a provenance record (see Provenance).

  Taking figure names as arguments (`make_figures.py <fig> …`) to re-run only
  some figures is fine, provided it merges into `numbers.json` rather than
  replacing it.
- Optional slow scripts (`validate_*.py`, fits, simulations). Their outputs
  (`.npz`, `.json`) sit in the memo directory, are committed, and are read by
  the products script.
- Optional `README.md`: a file table, and dated lines under `## Recent changes`.

Where the project's generator is notebooks or scripts outside the memo
directory, they write the figures (and `numbers.tex`, if the project uses
one) into the memo directory, and the rest of this skill reads "products
script" as that generator.

Number memos in order: the next unused number in the project's format, never
reused. A memo that is superseded keeps its number and says which memo
replaces it.

**Before renaming or moving a memo directory, check that nothing is running in
it** (`ps -ef | grep <memo dir>`, and any other agent sessions). A running script
resolved its paths when it started, and its final writes will fail if the
directory moves. If something is running, leave a symlink from the old path
until it finishes. Then update every path that names the memo, including other
memos that read its outputs.

## Writing

- **Quote every computed number from the generator's output,** never from
  memory or a session. Where the generator writes `numbers.tex`, quote through
  its macros and never type a measured number into the text. Macro names may
  contain letters only, so use `A`, `B`, `C`… suffixes rather than digits. If a
  macro renders oddly (`0.00\%`), fix the format in the script.
- **LaTeX rows written from Python end in `+ r"\\"`.** A single backslash
  breaks the build.
- **Read inputs only from durable places:** tracked products; gitignored
  products, which the script checksums; and validation outputs in the memo
  directory. Never read from or cite a `scratch/` directory or a session
  scratchpad: the next reader can't regenerate from them.
- **Put inputs that are not yet known in one marked block** in the generator
  (e.g. `OBS = dict(...)`, tagged `PLACEHOLDER`). Show them in the text with
  the project's open-point macro (default `\placeholder{}`, in orange), and
  list them in an inputs table with their source and status.
- **When a result is refined or overturned, follow the project's revision
  model.**
  - *Living* (default): rewrite the affected sections in place; don't append
    a correction.
  - *Append-only*: leave the earlier text and its provenance as they were.
    Add a dated amendment, or, where the conclusions change, write a new memo
    that supersedes this one, and mark both.

  Either way:
  - update the status (and version and date, if the title page carries them);
  - add a dated line to the memo's changelog for any substantive change, saying
    what changed and what was learned, and naming the corrected claim if one
    was overturned;
  - keep the status box at the top current.
- **Keep sections on caveats and approximations and on next steps.**
- **Cite only from bibliographic metadata you actually retrieved** (ADS,
  arXiv, DOI). Never invent an author, title, volume or page number; if the
  metadata isn't in hand, leave an open-point marker instead of a citation.
  Use the project's bibliography; with none set, prefer
  `\begin{thebibliography}` over BibTeX.

## Provenance

Use the project's provenance format where it sets one (for example a table of
the notebooks, code commits and product directories behind the memo). Default:
the products script records what it was run from, in `numbers.tex` or in a
table it writes:

- the git commit of the memo's repository, with `-dirty` if anything tracked
  was modified when it started (`git describe --always --dirty`);
- the same for each separately versioned package it imports;
- a checksum of every gitignored input it reads.

So that the stamp is clean, commit the source before regenerating (step 4).
Where the generator lives in another repository, commit it there first and
cite that commit.

## Steps

1. **Work on a branch,** in a worktree if the checkout is shared, recording
   the branch or worktree wherever the project tracks them before creating
   it. Default location: `git worktree add -b <branch> <scratchpad>/wt-<name>`.
   Supply whatever the worktree lacks, as the project's § Memos says: symlinks
   to gitignored data, environment variables, or `PYTHONPATH` for a sibling
   package. Branch-only data products go inside the worktree (via the
   generator's `--out-dir`) or wherever the project puts branch drafts, never
   into the shared checkout. Untracked copies there would block switching it
   back to the main branch after the merge.

2. **Edit the source:** the `.tex`, the generator, and any validation
   script, following Writing above. Update the memo's `README.md` or the
   project's memo register if it has one.

3. **Run any validation script that changed.** These are slow: run them in the
   background and wait for the completion notification. Commit their outputs.

4. **Commit the source first,** with explicit pathspecs, following the
   project's and the harness's rules for commit messages and attribution:
   `git commit -m "memo NNN: <what changed>" -- <paths>`. Before step 5,
   `git status --short` must show no modified tracked file. If outputs from an
   earlier run are left over, discard them:
   `git checkout -- <memo>/figures <memo>/numbers.* <memo>/memo-NNN.pdf`.

5. **Regenerate and build.** Run the generator, then the TeX command from
   the project's § Memos. With `pdflatex`, run it twice so references resolve;
   `latexmk` and `tectonic` rerun on their own. Keep the log
   (`tectonic --keep-logs`). The build log should have no errors, no undefined
   references, and no overfull boxes. Fix overfull boxes this way:
   - a long `\texttt{path}` becomes `\nolinkurl{...}`;
   - a long inline equation becomes display math.

6. **Check before committing products:**
   - `git diff <memo>/numbers.tex` (or the provenance table): only the numbers
     you meant to change moved. Any other change means an input or package
     changed under you. Find out why before going on.
   - The provenance record has no `-dirty`.
   - Read the changed passages as rendered. Extract the PDF text with PyMuPDF
     (`import fitz; ''.join(p.get_text() for p in fitz.open(pdf))`), then grep
     it for the passage.
   - Look at the pages:
     `pdftoppm -png -r 70 memo-NNN.pdf <scratchpad>/page`, then read the images.
   - Run the mechanical audit:
     `python ~/skills/latex-audit/scripts/latex_audit.py <memo>/memo-NNN.tex --log <memo>/memo-NNN.log`,
     adding `--extra-placeholder <macro>` for each open-point macro the project
     uses beyond `\todo`, `\tbd` and `\placeholder`. It should report no
     serious findings other than open points you meant to leave.
   - Check printed text size:
     `python ~/skills/21cm-figures/scripts/pdf_min_font.py <memo>/memo-NNN.pdf`.
   - For a full review before circulating, follow `latex-audit`.

7. **Commit the products** (figures, numbers, tables, PDF) as
   `memo NNN: regenerate products at <source commit>`. Push the branch if there
   is a hub and the project allows it. Report what changed, with the key
   numbers.

8. **Merge the way the project says.** If it uses pull requests, open one (if
   the project allows it) and leave the merge to the owner. Default, for a
   local merge: merge only on the owner's direct OK in the session. If the
   main branch has not moved, fast-forward it, then remove the worktree and
   delete the branch. If it has moved:
   - merge the main branch into yours in the worktree;
   - repeat steps 5–6 there;
   - fast-forward main.

   Update a shared checkout only if it is on main, clean, and has no local
   commits. Otherwise leave it alone and say so.

## Figures

`21cm-figures` holds the rules. For memos specifically:

- **Show measured quantities in their measurement domain,** rather than
  abstracted statistics such as fractions above a threshold or summary scores.
  Use a summary statistic only for a scan over many configurations, and then
  show it next to at least one direct comparison of a representative case.
- **Compare data and model with the residual on shared axes.** For 2-D data,
  use a row of three panels: data | model | residual, with one colour scale
  shared by data and model, and a diverging scale for the residual. For 1-D
  data, use three stacked panels, or data and model overlaid with the residual
  below. Show the noise level on the residual.
- **Write vector PDFs from the generator only,** named as the project names
  them. Use a white background and `pdf.fonttype=42`, and draw at the printed
  width so that text prints at 7–8 pt or larger.

```python
fig, ax = plt.subplots(1, 3, figsize=(7.0, 2.6), sharex=True, sharey=True,
                       constrained_layout=True)
norm = LogNorm(vmin, vmax)                       # shared by data and model
for a, img, t in zip(ax[:2], (data, model), ("data", "model")):
    im = a.imshow(img.T, origin="lower", extent=ext, norm=norm, cmap=SEQ)
    a.set_title(t)
fig.colorbar(im, ax=ax[:2], shrink=0.8)
lim = np.nanmax(np.abs(resid))
imr = ax[2].imshow(resid.T, origin="lower", extent=ext, cmap=DIVERGING,
                   vmin=-lim, vmax=lim)
ax[2].set_title("data − model")
fig.colorbar(imr, ax=ax[2], shrink=0.8)
```
