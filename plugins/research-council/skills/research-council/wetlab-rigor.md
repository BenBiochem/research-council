# Wet-lab rigor checklist

Loaded by the orchestrator when **Mode = wet-lab** (or mixed). Feeds the
**Methodologist·wet** and **Feasibility·bench** seats. The orchestrator picks
the 2–4 points most relevant to the specific design and inlines them into the
seat prompt, it does not dump the whole file.

General bench-experiment failure modes for the life sciences. Examples are
illustrative, not domain-specific.

---

## Methodologist·wet: inference validity (will the experiment support the claim?)

1. **Controls: the result is the control.** Every claim needs its comparator:
   a **negative** (no-input / vehicle / buffer-only), an **inactive/scrambled
   variant** as the specificity control (to prove the signal is the effect, not
   background), and a **positive** that proves the assay can detect the effect
   at all. A measurement without its control is not evidence.

2. **Replicates: biological vs technical.** Technical replicates (same sample,
   re-measured) quantify instrument/handling noise. **Biological replicates**
   (independent preparations) quantify the actual claim. n = 3 technical ≠
   n = 3 biological; only the latter supports "the system does X." Always state
   which, and design for biological.

3. **Power / detectability.** Is the predicted effect size above the assay's
   noise floor? How many replicates resolve the difference you expect? An
   underpowered "no difference" is not a negative result.

4. **Batch effects & confounds.** Preparation batch, day, operator, plate
   position, instrument drift, all of which vary. Block or randomize them. Beware
   effects that *masquerade* as your signal: differences in input amount,
   maturation/equilibration lag, a tag or carrier altering behavior, or an
   abundance difference read as an activity difference.

5. **Assay validation & dynamic range.** Is the readout linear over the range
   you use? Is it reporting what you think? Many proxy readouts report a
   correlate (size, proximity, abundance) rather than the property of interest.
   Build a standard curve; check for saturation and for signal at the bottom of
   the range.

6. **Readout artifacts.** Background signal, interference at high concentration,
   decay/bleaching over time, edge effects. Separate "more of the thing" from
   "more active thing."

---

## Feasibility·bench: can it be done cleanly, and does it respect the constraints?

1. **Respect declared method constraints (do not invent your own).** If the
   project's notes/`CLAUDE.md` or the user's message declares standing method
   constraints (techniques to **avoid** or **prefer**), enforce them: flag any
   step that violates an avoid-constraint and propose a compliant alternative.
   **Absent a declared constraint, apply neutral feasibility judgment and do NOT
   impose technique preferences of your own**; there is no universally "right"
   method, and pushing an unstated preference biases the council.

   > Constraint format (optional, user-supplied, this is the *mechanism*, not a
   > default): a short block such as
   > `CONSTRAINTS: avoid <technique A>, <technique B>; prefer <technique C>`
   > in the project `CLAUDE.md` or in the request. The skill ships with no
   > built-in preferences; the placeholders are filled only by you.

2. **Physical feasibility.** Reagents, equipment, and time per round; is the
   construct/sample makeable; expression/solubility/stability risk; realistic
   yield and throughput.

3. **The cleanest experiment.** What single experiment most cleanly isolates the
   one variable? Reject designs where two things change at once (you will not
   know which caused the effect). Prefer a constant, well-characterized readout
   across conditions.

4. **Architecture confounds.** When comparing construct/format architectures,
   each carries its own confound (stoichiometry, coupling, processing). Hold the
   reporter/readout constant across conditions so the architecture is the only
   variable.

5. **Throughput & cost.** Match the readout's throughput to the number of
   conditions; a plate-scale readout beats a low-throughput gel/blot when many
   conditions or replicates are needed; reserve low-throughput confirmation for
   a few decisive samples.
