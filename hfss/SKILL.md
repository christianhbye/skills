---
name: hfss
description: Use when running Ansys HFSS / Electronics Desktop (AEDT) from a terminal or as an agent, with no GUI - exporting fields, S-parameters or antenna parameters (radiation efficiency, gain, directivity) from a saved solution, launching or monitoring a solve on a remote host, reusing a model someone else built, or reading back an earlier HFSS run.
---

# HFSS from the terminal

AEDT runs headless as `ansysedt -ng -RunScriptAndExit <script.py>`, with the script in AEDT's IronPython. The site details come from the project's `CLAUDE.md`/`AGENTS.md`, which override this skill wherever they differ: the host and how to reach it, where the install and run folders are, and the core and RAM budget. Read those first. If the project names no HFSS host, ask the user. Never guess a login or try a password.

## 1. Before any launch

1. **Look for an export that already exists, before you plan anything.** List the project's HFSS run root on the host (`ls` each run folder and its `exports/` or `parameters/`), and grep the local workspace (docs, products, contribution archives) for the quantity. If an export already holds it, the task is to read and check that export, not to run AEDT. Otherwise, a quantity computed from an existing solution (antenna parameters, fields, S-parameters at a new reference impedance) is a report-only export: open the project, export, close. It needs no solve and no save.
2. **Work on a private copy.** Never open someone else's project or an original in place. Copy the `.aedt` together with its whole `.aedtresults` tree into a new dated run folder, and write `SOURCE_MANIFEST.json` (the source path and the SHA-256 of every file). Check the source hashes again after the run.
3. **Run one AEDT process per project.** Check `ps -eo pid,etimes,pcpu,rss,args | grep -i ansys` and look for `.lock` files next to the `.aedt`. If the project is being solved, read its status files (§ 4) instead of opening it.
4. **Check the host:** `nproc`, `cat /proc/loadavg`, `free -g`, and `df -h` on the run folder. Inside a container these report the whole host, which is what a shared budget is measured against.
5. **Check that AEDT is installed** (`ls <install>`, `which ansysedt`). If the install folder is empty or the licence check fails, stop and tell the user.

## 2. Launch recipe

```
TMPDIR=/tmp/aedt-<run> TMP=... TEMP=... ANSOFT_TEMP_DIRECTORY=... \
OMP_NUM_THREADS=N MKL_NUM_THREADS=N OPENBLAS_NUM_THREADS=N \
taskset -c <N cores> <install>/ansysedt -features=beta -ng -RunScriptAndExit <run>/scripts/<script>.py
```

- **Use a short temp path** (for example `/tmp/aedt-<run>`) for all four variables and for `oDesktop.SetTempDirectory`. AEDT fails at startup to create its MainWin RPC endpoint when the temp path is long.
- **Set the core count with an ACF file** that the script imports for solves, and match it with `taskset` and the thread variables. The file must be wrapped in an outer `$begin 'Configs'` block, or the import fails. Its core settings: `NumEngines=1`, `NumCores=N`, `RAMPercent=<budget>`, `NumGPUs=0`, `UseAutoSettings=false`.
- **Launch detached** so the run survives the SSH session ending: `subprocess.Popen(args, start_new_session=True, stdin=DEVNULL, stdout=log, stderr=STDOUT)`, or `setsid nohup … &`. Write `logs/<step>_launch.json` with the PID, command, cores, MemAvailable and nice value.
- **Write the script in IronPython 2.7:** `from __future__ import print_function`, no f-strings, no numpy. Pass paths in through environment variables rather than hard-coding them.

A complete report-only export, which gives radiation efficiency, gain, directivity and radiated and accepted power at every frequency:

```python
"""Report-only export: no solve, no project save."""
from __future__ import print_function
import os, json, ScriptEnv
OUT, SRC, DESIGN = (os.environ[k] for k in ('RUN_OUT', 'RUN_AEDT', 'RUN_DESIGN'))
SOLUTION = os.environ.get('RUN_SOLUTION', 'Setup1 : Sweep')
ScriptEnv.Initialize('Ansoft.ElectronicsDesktop')
oDesktop.SetTempDirectory(os.environ['TMPDIR'])
project = oDesktop.OpenProject(SRC)
design = project.SetActiveDesign(DESIGN)
rad = design.GetModule('RadField')
os.makedirs(OUT + '/parameters')
freqs = [int(f) for f in os.environ['RUN_FREQS_MHZ'].split(',')]
for f in freqs:
    path = OUT + '/parameters/%04dMHz.txt' % f
    rad.ExportParametersToFile([
        'ExportFileName:=', path, 'SetupName:=', '3D',  # infinite-sphere name
        'IntrinsicVariationKey:=', "Freq='%dMHz'" % f,
        'DesignVariationKey:=', '', 'SolutionName:=', SOLUTION,
        'Quantity:=', ''])
    if not os.path.isfile(path) or os.path.getsize(path) == 0:
        raise RuntimeError('missing export ' + path)
oDesktop.CloseProject(project.GetName())  # no Save call anywhere
with open(OUT + '/EXPORT_DONE', 'w') as fh:
    fh.write(json.dumps({'n': len(freqs), 'solution': SOLUTION}))
```

## 3. Done means verified, not exit code 0

- AEDT can exit 0 after the script raised an exception. Success means the script wrote its `*_DONE` marker *after* checking each output, and the driver's `STATUS.json` says `COMPLETE`, not `ERROR`.
- Check the contents: the angle grids, finite values, full frequency coverage, and the phi seam. Before trusting new exports, reproduce a stored result: regenerate one existing export from the copy and compare it, byte for byte or within a stated tolerance.
- **Patterns need their own convergence check.** A solve that converged on S-parameters alone has not converged its fields. If you need patterns, compare far fields between successive mesh passes.
- **Record the reference impedance.** The default S-parameter report uses the port's own impedance, which may not be 50 Ω. Export Touchstone files at explicitly stated references and record which is which. The report's reference impedance does not tell you where the physical port plane is.

## 4. Watching a running solve

Read the run's `logs/STATUS.json` and the end of its event and console logs, and check the recorded PIDs with `ps`. Never open a second AEDT session on a project to look at it.

## 5. Results leave the host

Copy the products back with their hashes (`sha256sum` at both ends) into the place the project's workflow names for new products, never into read-only data folders. Label them diagnostic simulation output until they have been checked against measured data.

## Common mistakes

| Mistake | Instead |
|---|---|
| Re-solving to get a quantity the saved solution already holds | Do a report-only export (§ 2 example) |
| Opening the original or a teammate's project | Use a hashed private copy |
| A long temp path, then "RPC endpoint" failures at startup | Use a short `/tmp/aedt-<run>` |
| Using all cores, or `-batchsolve` with a guessed core count | Use the project's budget, an ACF, and `taskset` |
| Treating exit 0 as success | Rely on DONE markers and `STATUS.json`, then check the contents |
| Reading S-parameter convergence as pattern convergence | Compare far fields between passes |
| Running paths from old run notes unchanged | Re-check them against the project's current site notes; hosts and mounts move |
