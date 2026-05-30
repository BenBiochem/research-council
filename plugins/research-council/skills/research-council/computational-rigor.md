# Computational rigor checklist

Loaded by the orchestrator when **Mode = computational** (or mixed). Feeds the
**Methodologist·comp** and **Feasibility·compute** seats. The orchestrator does
not dump this whole file into a sub-agent, it picks the 2–4 points most
relevant to the specific artifact and inlines them into the seat prompt.

These are general failure modes for computational science / modeling /
benchmarking. The examples are illustrative, not domain-specific.

---

## Methodologist·comp: inference validity (are you fooling yourself?)

1. **Metric ≠ metric.** Ranking metrics (R, ρ, AUC), calibration metrics (MAE,
   RMSE), and direction metrics (sign/class accuracy) measure different things;
   a method can win one and lose another. Report the metric that matches the
   *claim*, not the most flattering one. Watch the brutal sub-case, e.g.
   accuracy on the borderline/near-threshold examples is often close to chance
   even when the headline correlation looks strong. Watch pooled-vs-grouped
   correlation (Simpson's paradox): a strong pooled number can hide weak
   within-group ranking.

2. **Baseline fairness: apples to apples.** A comparison is valid only if both
   methods see the **same inputs, same preprocessing, same identifiers, and the
   same target quantity**. Comparing method-A computed on one representation to
   method-B computed on a different representation is two measurements, not a
   comparison. Confirm the test set and pipeline are identical before reporting
   a head-to-head.

3. **Data leakage / contamination.** Is the test case (or a near-duplicate) in
   the model's *training* set? Learned predictors can have seen your target.
   Cross-validation that splits items but not near-duplicates/homologs leaks.
   Any tuning that touched the test set invalidates the number.

4. **Cherry-picking / generalization.** A headline from a *favorable* case is
   not the method's number. "0.9 on the easy system" vs "0.7 averaged over the
   full set", the honest figure is the **distribution across all cases**, not
   the best one. If a method only reaches target on a subset, say which subset
   and why.

5. **N, seeds, determinism, normalization.** Stochastic procedures need
   multiple seeds; report mean ± sd, not one run. Know which steps are
   deterministic (you cannot estimate variance from a deterministic step). Watch
   normalization ambiguity (per-unit vs per-aggregate, total ÷ N vs a local
   window); it can move a result by an integer factor and silently make or break
   a claim.

6. **Canonical vs re-implementation.** A re-implementation can manufacture
   artifacts that look like findings (a spurious large-magnitude outlier, a
   depressed correlation) that vanish under the canonical/published
   implementation. Pin the version, prefer the reference implementation, and
   flag any number whose provenance is a re-impl.

7. **Indexing & identifier bugs.** Off-by-one (0-based vs 1-based) silently
   corrupts everything downstream and can halve a correlation without erroring.
   When multiple numbering/identifier conventions exist (source format vs the
   tool's internal indexing), map them explicitly rather than assuming
   alignment. Verify one case by hand.

8. **The right quantity.** What is *actually* computed, and does it match the
   claim? Two quantities that look interchangeable often are not (e.g. an
   equilibrium property vs a rate; a global property vs a local one). A perfect
   number for the wrong quantity is still wrong, and is the easiest mistake to
   miss because the code "works."

---

## Feasibility·compute: can it run, and what is the fastest clean version?

1. **Convergence & protocol soundness.** Will the optimization/sampling
   actually converge? Is the procedure doing what you think, e.g. a global
   refinement when you wanted a local one; independent vs paired handling of two
   states being compared.

2. **Runtime, cost, and "will it finish."** Large benchmarks routinely stall
   and never complete. Estimate wall-time, check for a parallel driver, and run
   the **smallest informative subset first** to derisk before committing the
   full set.

3. **Reproducibility / infra.** Seeds fixed, tool versions pinned, environment
   captured, determinism verified where claimed. A number you cannot regenerate
   is not a result.

4. **Smallest clean test.** What is the *minimal* computation that would change
   your mind? Don't run thousands of cases when a few well-chosen ones answer
   the question. Design the cheap decisive experiment first.

5. **Silent-failure hygiene.** Empty/failed outputs scored as 0 instead of
   dropped; unit mix-ups (e.g. mismatched energy units, or an inverted sign
   convention); truncated files read as complete. These pass without erroring
   and poison the aggregate. Spot-check raw outputs, not just the summary.
