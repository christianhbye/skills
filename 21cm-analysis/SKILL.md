---
name: 21cm-analysis
description: Rules for reducing radio and 21-cm cosmology data and for claiming anything from it (averaging, noise, coordinates, model comparison, validation, scope). Read before analysing data, fitting or validating a model or method, or stating a result.
---

# Analysing 21-cm data

Each rule is an imperative with its reason. Project files add their own examples;
the rules themselves carry no project specifics.

## Reducing data

- **Average with weighted means, Σ w x / Σ w, not medians.** Medians and other
  order statistics are for finding bad data (flagging, masking, outlier
  rejection). As estimates they are nonlinear, so they don't commute with
  calibration or further averaging. They are biased for skewed radiometer data,
  their noise doesn't follow the radiometer equation, and they hide what they
  reject.
- **Give flagged samples zero weight, and name the flag version applied.**
  Build the mask first, then average. Never zero-fill flagged samples or
  interpolate across them silently.
- **Measure noise against the radiometer equation.** Predict it from the
  system power or temperature, the channel width Δν and the integration time
  τ, and state Δν and τ. Where the scatter exceeds the prediction, report both.
  The excess is a finding to explain, not a new noise floor.
- **Work in fundamental coordinates:** time, frequency, fringe rate, delay,
  baseline, and (u, v). Measure signal in Jy, K, uncalibrated correlator power,
  or SNR, and say which noise an SNR divides by. Define any metric you need
  beyond these in terms of these.
- **Index by coordinate, not by array position.** Map channels through the
  frequency axis the data carry. A band-limited load does not start at
  channel 0, and position-based indexing silently shifts every frequency.
- **State the frame of every direction:** true or grid north, topocentric or
  celestial, J2000 or apparent. A map projection's grid north is not true
  north.
- **Know what a mask selects, and check a result with and without it.** A mask
  that keeps a minority of samples, or keeps them unevenly in the variable you
  fit against, selects the data as well as cleaning it.
- **A median-based flag fails once the source is present in more than about
  half the samples.** The median then sits on the source-present level, and
  the source is flagged nowhere. When a source can be present that often,
  measure the background where it is known to be absent (out of view, switched
  off) and flag against that.

## Comparing data with models

- **Compare in the data's coordinates, through the same estimator, at the
  same reference point.** The reference point is the same calibration plane
  and the same instrument response. An estimator whose passband or template
  changes across the groups being compared can manufacture the trend it seems
  to find.
- **Show data, model and residual, and judge the residual against radiometer
  noise.** A model is good enough when the residual is consistent with noise
  over the cells it claims to describe.
- **Score a replacement on everything it would cover,** and separately on the
  part the current method rejects. The subset the current method keeps is where
  it already works, so a comparison there favours it.
- **Converge before you compare.** Report each iterative solve's convergence
  status and residual with its result. Unconverged solves produce trends that
  reverse once they converge.
- **Before interpreting a fitted coefficient, name everything else that would
  produce the same column.** A coefficient is only as specific as its column. A
  parameter exactly degenerate with another is not measured, whatever error the
  fit quotes.
- **A better held-out fit supports a structure only if an alternative with the
  same freedom does worse.** Fit the alternative explanation with as many
  parameters, and score both on the same held-out data.
- **Hold a physical model to simple generic comparators, not flexible ones.**
  Its bar is a line or quadratic in the same variable with the same number of
  parameters (± 1), declared before scoring. A flexible process (a Gaussian
  process, a many-knot spline) describes what is still unexplained. Report it
  alongside, but don't require the physics to beat it.
- **An identified physical model that predicts as well as a generic default
  replaces it.** It need not beat it. Test the replacement itself, not the
  physical model as a small correction on top of the default. What the generic
  default still does better is a remainder to hunt down, not a reason to keep
  the generic default.
- **A held-out test that a fitted parameter can absorb has no power.** If the
  candidate's column is degenerate with a free parameter, the held-out score
  barely moves whatever the truth is. Report the candidate as untestable by
  that test, not rejected, and say what independent constraint could test it.
  Test it on data where nothing absorbs it, or against an external scale.
- **Untestable is not rejected.** Say where else the term can be tested. When
  you carry it anyway, label it prior-driven wherever it moves a result.
- **A term whose template comes from the fit itself is a diagnostic.** A free
  factor times the model, or a column shaped like the residual, absorbs
  residuals by construction. State its freedom before quoting any numbers.
  Only a constraint independent of the fit (raw data, another sensor, a bench
  measurement) can identify it.
- **Fit the degenerate groups the data constrain, and separate them after the
  fit.** Use one coefficient per multiplicative or additive group with the
  same column shape. Split it into physical pieces only after the fit, with
  stated priors. Never fit separately named parameters the data cannot tell
  apart.
- **Keep an identified parameter fixed at its identified value, and carry the
  data's preferred value as a systematic.** If the design or a measurement
  fixes a parameter and the data prefer another value, don't free it in a fit
  where it is degenerate. Also don't transfer a value fitted in one regime to
  another regime while the cause is unknown. Run the analysis at both values,
  report the shift as a systematic band, and hunt the discrepancy as an
  unidentified term.
- **Segment and explain a discontinuity; don't absorb it.** If the residual is
  clean on either side of a jump, the jump is an event to find. Extra model
  freedom would hide it, and lose the information the jump carries.

## Simulations and derived numbers

- **Tie every simulation result to the configuration that produced it,** and
  regenerate it when the model changes. Key cached results by a fingerprint of
  the parameters that define them, and never fall back to an older key when
  the current one misses: a stale result loads silently and looks current.
- **Derive a quantity that follows from others; don't type it.** Compute it
  from its inputs where it is used. A typed copy drifts from its inputs and
  nothing notices.
- **A linearised margin holds only near the point it was computed at.** A
  requirement divided by a small slope gives an enormous allowance that the
  nonlinear response doesn't support. Check it against the physical range
  (a reflectivity in [0, 1], an angle in [0°, 180°]), and recompute a large
  margin with the full model before quoting it.

## Validating and claiming

- **Report the worst case, not just the typical one.** Give maxima or high
  percentiles and per-channel or per-row counts alongside any mean, and prefer
  showing samples to summary statistics. A defect confined to a few rows or a
  third of the channels leaves band averages untouched.
- **A check must be able to come out the other way.** Before running it, say
  what result would contradict the claim. These are not checks:
  - a fitted parameter compared with itself;
  - a confirmation that reuses the data it confirms;
  - a quantity checked against the same epoch it depends on.
- **Name the scope.** State the window, era, regime, channel range or file set
  a number comes from. Check a second, disjoint sample before stating it
  generally.
- **A control differs from the test in one thing only.** Comparing a condition
  with its absence also compares everything else that changes between them.
  Choose the control that isolates the variable you are testing.
- **A change fixed a problem only if undoing it brings the problem back.**
  When several changes land together, the one written up as the fix is often
  not the one that did it. Revert it alone before crediting it, and check
  causes stated in status notes against the history before repeating them.
- **A spectral signature is not a physical identification.** Which source
  produced a signal is settled by a physical test (an intervention, a geometry,
  a hardware state), not by spacing, shape or timing alone. Until then, say
  "coincides with" or "consistent with".
- **Record dead ends where they would otherwise be repeated.** Write one
  sentence with the numbers that rule the approach out, and keep the code
  that produced them findable.
- **Every diagnostic ends in an outcome:**
  - negative: parked, saying what would reopen it;
  - inconclusive: a concrete next test, queued;
  - positive: adopted, with its implementation path.

  A partly explained effect is adopted *and* its remainder is hunted. Don't
  let a thread end at "interesting".
- **A summary's conclusion must follow from the primary outputs.** Before
  passing on a conclusion, check it against the numbers it rests on and the
  rules above, especially when someone else, or another model, wrote it.
  Arithmetic is usually right. The gate applied and the conclusion drawn are
  where summaries go wrong.
