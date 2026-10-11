---
name: research-council
description: >-
  Pressure-test a research decision, benchmark, experiment, result, or paper
  claim through a critic-weighted panel (Skeptic/Reviewer-2, Methodologist,
  Theorist, Taste, Naive-reader, Feasibility), then synthesize one verdict.
  Auto-detects computational vs wet-lab work and routes the rigor lenses
  accordingly. Runs in Claude via sub-agents; the artifact is never sent to a
  third-party model or API, so it is safe for unpublished work. Triggers:
  "council this", "reviewer 2 this", "pressure-test/stress-test/red-team this",
  "is this benchmark/experiment sound", "what will a reviewer attack", "should
  I pursue X or Y", "am I fooling myself". Do NOT trigger on factual lookups,
  simple implementation, or questions answered by just running the computation
  or the experiment.
---

# Research Council

A single AI answer hides its own blind spots. The council pressure-tests one
research artifact through several **independent critic lenses**, then
synthesizes where they converge, where they clash, what the Chair found that
the seats missed, and the one thing to do next.

This is adapted from Andrej Karpathy's [LLM Council](https://github.com/karpathy/llm-council),
with three deliberate departures tuned for research, computation, and
paper-writing:

1. **Critic-weighted, not idea-weighted.** In rigorous science the bottleneck
   is *selecting and killing* ideas, not generating them. The panel is built
   from seats that override the assistant's default agreeableness (Skeptic,
   Methodologist, Naive-reader): the lenses where a forced perspective
   genuinely changes the output. Pure "upside/expansion" seats are omitted.
2. **Phase- and mode-aware, not a fixed roster.** Different work activates
   different seats, and **computational vs wet-lab work fails in different
   ways**, so the rigor lenses specialize. This is the core feature.
3. **One model, no third-party egress.** Every seat is a Claude sub-agent. The
   artifact goes only to Claude, the model already running the session; no other
   model or API sees it. The bundled `council-seat` agent is limited to
   Read/Glob/Grep, so a seat cannot browse the web.

> One-model honesty: these seats are Claude prompted several ways, not several
> independent models. That is real value for *critique* (overriding default
> helpfulness shifts the output) and weaker for *independent-knowledge
> triangulation*. Agreement among seats is correlated, so it is not
> confirmation. A finding earns weight from the evidence it cites in the
> artifact, never from how many seats raised it.

---

## Step 0: Detect MODE and PHASE  (this is the routing core)

Before anything else, classify the artifact on two axes. Do this silently and
fast (no sub-agents yet).

### MODE: computational, wet-lab, or both

**COMPUTATIONAL**: the work lives in silico. Signals: prediction / benchmark /
model / dataset; a metric (R, ρ, MAE, RMSD, AUC, accuracy, ROC); an algorithm,
pipeline, or script; simulation / sampling / optimization / scoring; seeds,
convergence, cross-validation, train/test splits, hyperparameters.

**WET-LAB**: the work happens at the bench. Signals: a physical assay or
experiment; sample preparation / purification; cloning, constructs, reagents;
gels, spectroscopy, fluorescence, chromatography, microscopy; replicates,
controls, buffers, concentrations, incubation, organisms, yield.

**BOTH (mixed)**: the decision couples the two: a computational prediction that
will be (or should be) tested experimentally, "should I trust this prediction
enough to run the assay," a design that is modeled then built. Treat mixed as
first-class, do not collapse it to one side.

**If genuinely unclear after reading the artifact**, ask exactly ONE
clarifying question, "Is this computational, wet-lab, or both?", then proceed.

Announce the detected mode in one line so the user can correct it:
`Mode: computational` / `Mode: wet-lab` / `Mode: mixed (computational + wet-lab)`.

### PHASE: what kind of decision is this

- **Pursue / prioritize**: "should I do this at all," "which direction," "is
  this worth the time."
- **Design**: a benchmark or experiment that has not run yet ("is this design
  sound," "what will confound this").
- **Interpret**: results are in; what do they mean, are they real.
- **Write / frame**: a claim, figure, paragraph, abstract, or positioning
  decision for a paper / talk.

---

## Step 1: Ground the council (fast, targeted)

The critique is only as good as its context. Spend ~30 seconds, no more:

- Read the project's `CLAUDE.md` / README / notes if present; they carry the
  prior results, real numbers, and constraints that turn generic critique into
  *specific* critique.
- `Glob`/`Read` only the 1-3 files the artifact needs (the script, the figure
  data, the manuscript section). Do not survey the whole repo. Note their paths:
  seats read the originals (Step 3), they do not critique your summary of them.
- If the user pasted the artifact inline, that is the primary object.
- **Standing constraints:** if the project's `CLAUDE.md`/notes or the user's
  message declares method constraints (techniques to avoid or prefer, see the
  `CONSTRAINTS` note in the mode checklists), pass them to the Feasibility seat
  so it enforces them. Absent any declared constraint, do not invent technique
  preferences.

---

## Step 2: Select the panel  (efficiency lives here)

Do **not** spawn every seat. Spawn the smallest panel that covers the failure
surface for this PHASE × MODE. Target **3-4 seats**, cap at 5. Synthesis is
done once by you (the Chair); it is not a spawned seat.

| PHASE | Base seats | + add for COMPUTATIONAL | + add for WET-LAB |
|-------|-----------|------------------------|-------------------|
| Pursue / prioritize | Taste, Theorist, Skeptic | (Methodologist if a specific method is on the table) | (Feasibility if a specific experiment is on the table) |
| Design | Skeptic, Theorist | **Methodologist·comp**, **Feasibility·compute** | **Methodologist·wet**, **Feasibility·bench** |
| Interpret | Skeptic, Theorist | **Methodologist·comp** | **Methodologist·wet** |
| Write / frame | Skeptic (as Reviewer-2), Naive-reader, Taste | **Methodologist·comp** | **Methodologist·wet** |

Add Taste to Design or Interpret when the significance of the result is itself
in question.

**MIXED mode:** run the COMPUTATIONAL Methodologist **and** the WET-LAB
Feasibility seat together (plus Skeptic + Theorist). The Chair's job is then to
reconcile *prediction validity* against *experimental feasibility*.

The **Methodologist** and **Feasibility** seats are *mode-specialized*: before
writing their prompts, `Read` the matching checklist (it lives in this skill's
own directory, alongside this `SKILL.md`) and inline its 2-4 most relevant
points:

- computational → `computational-rigor.md`
- wet-lab → `wetlab-rigor.md`

Read only the checklist(s) the panel actually uses.

---

## Step 3: Frame the question

Build one prompt every seat receives, with three parts:

1. **The user's request, verbatim.** Seats see the original words, so a seat can
   notice if your framing drifted from what was asked.
2. **A neutral framing:** the core decision, key context, the real numbers and
   constraints from grounding, and what is at stake. Do not inject your own
   opinion or steer.
3. **The artifact:** inline text, or the file paths the seat may `Read`. Never
   only your summary of it.

**Print the framed question to the user before spawning**, so a bad frame is
visible and correctable. Every seat gets the identical text.

---

## Step 4: Convene the panel (parallel, one round)

Spawn the selected seats **simultaneously**, all in one message. Use the bundled
agent `research-council:council-seat` (Read/Glob/Grep only). If that agent type
is unavailable, use `general-purpose` and note "tool-lock not applied" in the
Panel line, since a general-purpose agent can reach the web. Each seat is
independent; no seat sees another's answer in this round.

**Seat prompt template** (fill `{SEAT NAME}`, `{SEAT CHARTER}`,
`{MODE CHECKLIST POINTS}` for Methodologist/Feasibility only, `{FRAMED QUESTION}`):

```
You are the {SEAT NAME} on a research council. Argue your lane at full strength
and do not soften to be kind. Other seats cover the angles you ignore; synthesis
comes later.

Your charter:
{SEAT CHARTER}

{MODE CHECKLIST POINTS}

The artifact brought to the council:
---
{FRAMED QUESTION}
---
If the framing above distorts the user's verbatim request, say so; that is a
finding.

Rules: every flaw you raise must quote the line, or state the fact, in the
artifact that it rests on. A flaw you cannot anchor is a suspicion; label it as
one. "No material flaw in my lane" is a valid and valued answer; say what would
change your mind. Do not invent a problem to fill the space.

Respond in 150-250 words, no preamble, in this form:
Lane verdict: sound | flawed | cannot tell
Findings, one per line: [fatal|major|minor] claim | evidence quote | cheapest
check that would confirm or kill it
```

Collect all responses. **Quorum:** if fewer than 2 seats return, stop and say
so; do not synthesize from one seat. Otherwise list failed seats in the Panel
line and continue.

**No sub-agent tool** (for example claude.ai web): run each seat in turn in this
context, keep their answers separate, and print `simulated in one context, not
independent` in the Panel line. Treat the result as weaker.

---

## Step 5: Chair synthesis (you do this; one pass)

You are the Chair. Read every seat response and audit it before reporting it:

- **Check the anchors.** For each fatal/major finding, confirm its evidence
  quote exists in the artifact and supports the claim. Findings that fail go to
  "Gaps and unverified", not to the shared findings.
- **Rank by evidence, not by votes.** Seats share one model, so agreement is
  correlated. A single well-anchored finding outranks three unanchored ones.
- **Look for what the panel missed**, inline: for each seat, what did the others
  catch that it missed, and what did all of them miss?
- **Sound is a valid outcome.** If every anchored finding is minor, say the
  artifact is sound enough to proceed, and name the residual risks.
- The Chair may overrule the majority when the dissenting reasoning is stronger;
  say so and say why.

**Deep mode** (only if the user said "deep"/"deliberate" or the stakes are very
high): run ONE more round. Re-spawn the seats with the other seats' anonymized
responses and three questions (strongest response? biggest blind spot? what did
all miss?), then synthesize. Skip by default; it doubles cost for marginal gain
on most questions.

---

## Step 6: Output (in chat, markdown, no files)

Present directly in the conversation. **Do not generate HTML or write any file
unless the user explicitly asks for a saved transcript** (if asked, write
`council-<unix-ts>.md` to the project's working dir). Use this structure:

```
## Research Council: {short topic}
**Mode:** {computational | wet-lab | mixed} · **Phase:** {phase}
**Panel:** {seats that ran}; failed: {none | names}; {independent | simulated in one context, not independent | tool-lock not applied}

### Shared findings
{findings raised by 2+ seats, each with its evidence quote. Agreement is correlated, so the quote carries the weight, not the count.}

### Contention
{real clashes; present both sides and why each is reasonable. Do not smooth over.}

### Gaps and unverified
{what the Chair found that the seats missed; unanchored seat suspicions; prior-work claims nobody could check (no web access, so novelty is judged from memory).}

### The call
{a clear, direct recommendation. "Sound enough to proceed" is a legitimate call. The Chair may overrule the majority, with reason.}

### What would change the call
{one or two observations or results that would flip it.}

### Do first
{ONE concrete next step. Not a list.}
```

For **mixed** mode, "The call" MUST explicitly reconcile the computational
verdict with the wet-lab verdict (e.g., "the prediction is sound enough to act
on, but the cleanest test is X, not the assay you proposed").

---

## The seats (charters)

Spawn only the ones Step 2 selected.

### Skeptic  (the adversary / Reviewer-2)
Search for the flaw that sinks the artifact: the confound, the null, the
alternative explanation, the unstated assumption that, if false, undoes the
claim. Report the strongest flaw that survives contact with the artifact's own
text. In **Write/frame** phase you ARE Reviewer 2: review the claim as a hostile
but fair referee who will recommend reject if the weakness is real. Never soften
to be kind. Never invent a flaw either: if the best you find is minor, say so.

### Methodologist  (mode-specialized: prevents fooling yourself)
Owns *inference validity*. Is the conclusion actually supported by the
evidence/analysis? **Load the mode checklist.** Computational: metric choice,
baseline fairness, leakage, N/seeds, cherry-picking, reproducibility, "is this
even the right quantity." Wet-lab: controls, replicate type, power, batch
effects, assay validation, dynamic range, readout confounds. This is the most
under-weighted seat in most groups; give it teeth.

### Theorist  (first-principles: prevents measuring the wrong thing)
Ignore the surface question. What are we *actually* trying to establish, and
does the artifact bear on it? Compare the user's verbatim request with the
framing and the artifact. Strip assumptions, rebuild the problem. The most
valuable output here is sometimes "you are asking/measuring the wrong thing."

### Taste / PI  (significance: prevents working on the wrong problem)
Is this worth doing at all? Significance over mere correctness. What does the
field gain if this fully succeeds, and is that gain worth the cost and risk?
Will the strongest result still be a footnote? Without web access you judge
novelty from memory, so end with a line `Prior-work claims I could not verify:`.

### Naive-reader  (legibility: prevents nobody understanding/believing it)
Zero assumed context. React only to what is on the page. Where does the
writing/figure assume knowledge the reader lacks? What term, axis, or claim
will an outside reader misread or distrust? Most valuable in Write/frame phase.

### Feasibility  (mode-specialized: prevents the un-runnable plan)
Owns *can this actually be done, and what is the fastest clean version*.
**Load the mode checklist.** Computational ("compute realist"): will it
converge/finish, runtime and cost, reproducibility, is the pipeline sound, what
is the smallest experiment that answers the question. Wet-lab ("bench realist"):
is it physically doable with reasonable reagents/time, are the controls
runnable, what is the cleanest experiment, and does it respect any declared
project method constraints.

---

## Operating rules

- **Spawn the selected seats in parallel, in one message.** Sequential spawning
  wastes time and risks bleed between seats.
- **Keep it lean.** 3-4 seats is the target. More seats produce consensus-mush
  and longer runs, not better science. Synthesis is yours, not a seat.
- **Detect mode before reading checklists**: load only the checklist(s) in play.
- **Specific and anchored beats balanced.** A seat that quotes the exact flaw
  earns its slot; a seat that hedges or invents does not.
- **Don't council the trivial.** If the question has one right answer or is
  settled by simply running the computation/experiment, skip the council and
  just answer (or run it).
- **No third-party egress.** Never route the artifact to an external model or
  API. Sub-agents only.
