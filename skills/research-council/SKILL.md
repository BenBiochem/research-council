---
name: research-council
description: >
  Convene a research council — a CRITIC-WEIGHTED panel of specialist lenses
  (Skeptic/Reviewer-2, Methodologist, Theorist, Taste/PI, Naive-reader,
  Feasibility) that independently pressure-test a research decision, a
  computational benchmark, an experimental design, a results interpretation,
  or a paper claim, then synthesize one verdict. It AUTOMATICALLY DETECTS
  whether the work is COMPUTATIONAL or WET-LAB (or both) and routes to the
  matching Methodologist + Feasibility lenses and rigor checklist. Runs
  entirely inside Claude via sub-agents — no external APIs, no data egress
  (safe for unpublished work). MANDATORY TRIGGERS: "council this", "research
  council", "convene the council", "pressure-test this", "stress-test this",
  "red-team this", "reviewer 2 this". STRONG TRIGGERS (when paired with a real
  research decision/artifact): "is this benchmark sound", "is this experiment
  sound", "what will a reviewer attack", "critique this claim/figure/paragraph",
  "should I pursue X or Y", "which approach", "am I fooling myself". Do NOT
  trigger on factual lookups, simple implementation tasks, or empirical
  questions that are answered by just running the computation or experiment.
---

# Research Council

A single AI answer hides its own blind spots. The council fixes that by
pressure-testing one research artifact through several **independent critic
lenses**, then synthesizing where they converge, where they clash, what they
all missed, and the one thing to do next.

This is adapted from Andrej Karpathy's [LLM Council](https://github.com/karpathy/llm-council),
with three deliberate departures tuned for research, computation, and
paper-writing:

1. **Critic-weighted, not idea-weighted.** In rigorous science the bottleneck
   is *selecting and killing* ideas, not generating them. The panel is built
   from seats that override the assistant's default agreeableness (Skeptic,
   Methodologist, Naive-reader) — the lenses where a forced perspective
   genuinely changes the output. Pure "upside/expansion" seats are omitted.
2. **Phase- and mode-aware, not a fixed roster.** Different work activates
   different seats, and **computational vs wet-lab work fails in different
   ways**, so the rigor lenses specialize. This is the core feature.
3. **One model, no egress.** Every seat is a Claude sub-agent. Nothing leaves
   the machine — no third-party model sees the artifact. Use this freely on
   unpublished data, manuscripts, and designs.

> One-model honesty: these seats are Claude prompted several ways, not several
> independent models. That is real value for *critique* (overriding default
> helpfulness shifts the output) and weaker for *independent-knowledge
> triangulation*. The verdict should lean on the reasoning surfaced, not treat
> agreement among seats as independent confirmation.

---

## Step 0 — Detect MODE and PHASE  (this is the routing core)

Before anything else, classify the artifact on two axes. Do this silently and
fast (no sub-agents yet).

### MODE — computational, wet-lab, or both

Read the user's question and any artifact, then classify:

**COMPUTATIONAL** — the work lives in silico. Signals: prediction / benchmark /
model / dataset; a metric (R, ρ, MAE, RMSD, AUC, accuracy, ROC); an algorithm,
pipeline, or script; simulation / sampling / optimization / scoring / energy
function; seeds, convergence, cross-validation, train/test splits, hyper-
parameters; any "in silico" result.

**WET-LAB** — the work happens at the bench. Signals: a physical assay or
experiment; sample preparation / purification; cloning, constructs, reagents;
gels, spectroscopy, fluorescence, chromatography, microscopy; a quantity
measured from an instrument; replicates, controls, buffers, concentrations,
incubation, organisms, yield.

**BOTH (mixed)** — the decision couples the two: a computational prediction
that will be (or should be) tested experimentally, "should I trust this
prediction enough to run the assay," a design that is modeled then built. Many
real projects are mixed — treat mixed as first-class, do not collapse it to
one side.

**If genuinely unclear after reading the artifact**, ask exactly ONE
clarifying question — "Is this computational, wet-lab, or both?" — then proceed.
Do not ask more than one.

Announce the detected mode in one line so the user can correct it:
`Mode: computational` / `Mode: wet-lab` / `Mode: mixed (computational + wet-lab)`.

### PHASE — what kind of decision is this

Pick the closest:

- **Pursue / prioritize** — "should I do this at all," "which direction,"
  "is this worth the time."
- **Design** — a benchmark or an experiment that has not run yet
  ("is this design sound," "what will confound this").
- **Interpret** — results are in; what do they mean, are they real.
- **Write / frame** — a claim, figure, paragraph, abstract, or positioning
  decision for a paper / talk.

---

## Step 1 — Ground the council (fast, targeted)

The critique is only as good as its context. Spend ~30 seconds, no more:

- Read the project's `CLAUDE.md` / README / notes if present — they carry the
  context (prior results, real numbers, constraints) that turns generic
  critique into *specific* critique. "This metric is weak" is useless; "your
  reported accuracy on the borderline cases is near chance, and a referee will
  cite that" is the product.
- `Glob`/`Read` only the 1–3 files the specific artifact needs (the script,
  the figure data, the manuscript section, a referenced result). Do not survey
  the whole repo.
- If the user pasted the artifact inline, that is your primary object.
- **Standing constraints:** if the project's `CLAUDE.md`/notes or the user's
  message declares method constraints (techniques to avoid or prefer — see the
  `CONSTRAINTS` note in the mode checklists), pass them to the Feasibility seat
  so it enforces them. Absent any declared constraint, do not invent technique
  preferences.

---

## Step 2 — Select the panel  (efficiency lives here)

Do **not** spawn every seat. Spawn the smallest panel that covers the failure
surface for this PHASE × MODE. Target **3–4 seats**, cap at 5. Synthesis is
done once by you (the Chair) at the end — it is not a spawned seat.

| PHASE | Base seats | + add for COMPUTATIONAL | + add for WET-LAB |
|-------|-----------|------------------------|-------------------|
| Pursue / prioritize | Taste, Theorist, Skeptic | (Methodologist if a specific method is on the table) | (Feasibility if a specific experiment is on the table) |
| Design | Skeptic, Theorist | **Methodologist·comp**, **Feasibility·compute** | **Methodologist·wet**, **Feasibility·bench** |
| Interpret | Skeptic, Theorist | **Methodologist·comp** | **Methodologist·wet** |
| Write / frame | Skeptic (as Reviewer-2), Naive-reader | **Methodologist·comp** | **Methodologist·wet** |

**MIXED mode:** run the COMPUTATIONAL Methodologist **and** the WET-LAB
Feasibility seat together (plus Skeptic + Theorist). The Chair's job is then to
reconcile *prediction validity* against *experimental feasibility* — that
reconciliation is the whole point of mixed mode.

The **Methodologist** and **Feasibility** seats are *mode-specialized*: before
writing their sub-agent prompts, `Read` the matching checklist (it lives in
this skill's own directory, alongside this `SKILL.md`) and inline its 2–4 most
relevant points into the prompt so the sub-agent stays focused:

- computational → `computational-rigor.md`
- wet-lab → `wetlab-rigor.md`

(Read only the checklist(s) the panel actually uses — that is why the mode is
detected first.)

---

## Step 3 — Frame the question

Reframe the user's raw question + grounding into one neutral prompt every seat
receives. Include: (1) the core decision, (2) key context from the user,
(3) key grounding from files/notes (the real numbers, constraints, prior
results), (4) what is at stake. Do not inject your own opinion or steer it.
Save this framed question; every seat gets the identical text.

---

## Step 4 — Convene the panel (parallel, one round)

Spawn the selected seats **simultaneously** as sub-agents
(`subagent_type: "general-purpose"`, all in one message for true parallelism).
Each seat is independent — no seat sees another's answer in this round.

**Seat prompt template** (fill `{SEAT NAME}`, `{SEAT CHARTER}`,
`{MODE CHECKLIST POINTS — methodologist/feasibility only}`, `{FRAMED QUESTION}`):

```
You are the {SEAT NAME} on a research council. You are not balanced and you do
not hedge — your job is to represent your angle as hard as it will go. Other
seats cover the angles you ignore; synthesis comes later.

Your charter:
{SEAT CHARTER}

{MODE CHECKLIST POINTS}   # included only for Methodologist and Feasibility seats

The artifact brought to the council:
---
{FRAMED QUESTION}
---

Respond in 150–250 words. No preamble. Be specific and concrete — name the
exact flaw, the exact confound, the exact missing control or metric, the exact
sentence a reader will trip on. Vague critique is failure. If you genuinely
find nothing wrong in your lane, say so plainly and say what would change your
mind.
```

Collect all responses. If a seat fails, note it and continue.

---

## Step 5 — Chair synthesis (you do this; one pass, includes the blind-spot check)

You are the Chair. Read every seat response. Do the cross-examination inline
(no separate review round in standard mode): for each seat, ask "what did the
others catch that this seat missed, and what did the panel as a whole miss?"
The Chair may overrule the majority when the dissenting reasoning is stronger —
say so and say why.

**Deep mode** (only if the user said "deep"/"deliberate" or the stakes are very
high): after the first round, run ONE additional cross-examination round —
re-spawn the seats with the other seats' anonymized responses and the three
questions (strongest response? biggest blind spot? what did all miss?), then
synthesize. Skip this by default; it doubles cost for marginal gain on most
questions.

---

## Step 6 — Output (in chat, markdown, no files)

Present directly in the conversation. **Do not generate HTML or write any file
unless the user explicitly asks for a saved transcript** (if asked, write
`council-<unix-ts>.md` to the project's working dir). Use this structure:

```
## Research Council — {short topic}
**Mode:** {computational | wet-lab | mixed} · **Phase:** {phase} · **Panel:** {seats that ran}

### Convergence — high confidence
{points multiple seats reached independently}

### Contention — genuine disagreement
{real clashes; present both sides and why each is reasonable. Do not smooth over.}

### Blind spots the panel caught
{what only surfaced via cross-examination; what individual seats missed}

### The call
{a clear, direct recommendation — not "it depends." Chair may overrule the majority, with reason.}

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
Assume the artifact has a fatal flaw and find it. Attack the central claim:
what is the confound, the null, the alternative explanation, the unstated
assumption that, if false, sinks everything? In **Write/frame** phase you ARE
Reviewer 2 — review the claim as a hostile but fair referee who will recommend
reject if the weakness is real. Never soften to be kind; softening here is the
failure mode.

### Methodologist  (mode-specialized — prevents fooling yourself)
Owns *inference validity*. Is the conclusion actually supported by the
evidence/analysis? **Load the mode checklist.** Computational: metric choice,
baseline fairness, leakage, N/seeds, cherry-picking, reproducibility, "is this
even the right quantity." Wet-lab: controls, replicate type, power, batch
effects, assay validation, dynamic range, readout confounds. This is the most
under-weighted seat in most groups — give it teeth.

### Theorist  (first-principles — prevents measuring the wrong thing)
Ignore the surface question. What are we *actually* trying to establish, and
does the artifact bear on it? Strip assumptions, rebuild the problem. The most
valuable output here is sometimes "you are asking/measuring the wrong thing."

### Taste / PI  (significance — prevents working on the wrong problem)
Is this worth doing at all? Significance over mere correctness. What does the
field gain if this fully succeeds, and is that gain worth the cost and risk?
Will the strongest result still be a footnote, or does it move something? No
other seat supplies this.

### Naive-reader  (legibility — prevents nobody understanding/believing it)
Zero assumed context. React only to what is on the page. Where does the
writing/figure assume knowledge the reader lacks? What term, axis, or claim
will an outside reader misread or distrust? Catches the curse of knowledge that
experts cannot see. Most valuable in Write/frame phase.

### Feasibility  (mode-specialized — prevents the un-runnable plan)
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
- **Keep it lean.** 3–4 seats is the target. More seats produce consensus-mush
  and longer runs, not better science. Synthesis is yours, not a seat.
- **Detect mode before reading checklists** — load only the checklist(s) in play.
- **Specific beats balanced.** A seat that names the exact flaw earns its slot;
  a seat that hedges does not.
- **Don't council the trivial.** If the question has one right answer or is
  settled by simply running the computation/experiment, skip the council and
  just answer (or run it).
- **No egress.** Never route the artifact to an external model/API. Sub-agents
  only.
