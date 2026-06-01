---
name: abstract-research
description: >-
  Advanced front-end to research-council that breaks single-thought anchoring.
  Before the council deliberates it generates a divergent spread of candidate
  directions in two tiers: on-graph reframes (orthogonal angles on the current
  problem) and off-graph moves (a completely different paradigm, tool, or
  protocol for the same goal, including ones that require finding a tool you do
  not have yet). The council then critiques across the whole spread, so it
  cannot get stuck improving one approach when a better approach lies off the
  current graph. Runs entirely in Claude via sub-agents; it SUGGESTS what to
  search for but does not search (no egress). Triggers: "abstract-research this",
  "what is our next move", "is there a completely different approach", "are we
  stuck in one way of thinking", "think bigger than improving this". Do NOT use
  for a straightforward critique of one artifact (that is plain research-council)
  or a factual lookup.
---

# Abstract Research

The research-council has one structural blind spot: every seat receives a single
framed question, so the whole panel reasons inside one framing. Worse, a critic
panel pointed at "how do we improve X" tends to only ever suggest *local* moves,
better parameters, a cleaner metric, one more control. It rarely says "stop doing
X this way at all."

Abstract Research fixes that by putting a divergent spread of directions on the
table *before* the council deliberates, including directions that leave the
current approach entirely. Then the council critiques across the spread. The
council cannot get stuck in one thought because it never sees only one.

This skill is a front-end to research-council. It adds the divergence stage and
an abstract-aware synthesis; it reuses the council's seats, panel selection, and
critique mechanics. Run plain `research-council` when you want a critique of one
artifact; run this when you want the council to also weigh *whether you should be
doing it this way at all*.

> No egress, same as the council. The off-graph moves may say "go find a tool
> that does X." This skill SUGGESTS what to search for; it does not perform the
> search. Pursuing those leads (for example with `/hyperresearch`) is a separate,
> user-gated step that does involve web egress. The analysis here stays local.

---

## Step 1: Frame and detect

Do research-council Step 0 (detect MODE: computational / wet-lab / mixed) and
Step 1 (ground the question from `CLAUDE.md` / notes / the artifact). Reframe
into one neutral question, the same way the council does. Announce the mode.

---

## Step 2: Divergence (the new part, two tiers)

Generate a spread of candidate directions for the question. Spawn these as
parallel sub-agents (`subagent_type: "general-purpose"`, one message), each
producing ONE short take (about 120 to 200 words: the direction plus its
tentative answer). Aim for 5 to 7 takes total across the two tiers. They must be
genuinely different, not variations on one.

### Tier A, on-graph reframes (3 to 4)
Different angles *within* the current problem space. Same goal, same general
approach, fresh framing. Use lenses such as:
- the literal, object-level answer,
- a reframe ("the real question is..."),
- an inversion ("what if the opposite is true / what if we removed this?"),
- a first-principles rebuild,
- an analogy transferred from another part of the same field.

### Tier B, off-graph moves (2 to 3): the point of this skill
A completely different paradigm, tool, or protocol that achieves the **same
goal** while abandoning the current approach. Not an improvement of what exists,
a replacement of the whole method. Each off-graph agent is told explicitly:

> Do not improve the current approach. Leave its solution space entirely. Propose
> a different paradigm, technique, instrument, or field that could accomplish the
> same goal. You are allowed, and encouraged, to propose a tool or method the
> user does not currently have. If it does not exist in their stack, name what it
> would need to do and what to search for to find it. "An angle not even on the
> same graph."

Tag each off-graph move with one of:
- **[in-reach]** doable with tools the user already has, or
- **[needs-search]** requires finding or learning a new tool/method; state plainly
  what capability to search for.

---

## Step 3: Council considers the spread

Hand the full set of takes (Tier A + Tier B) to the research-council panel as
"candidate directions on the table." Select and spawn the seats exactly as
research-council Step 2 and Step 4 do for the detected PHASE and MODE, but the
framed question now carries the spread, and each seat works *across* it:

- **Skeptic**: which directions are dead ends? Which off-graph move is a mirage
  (sounds bold, collapses on contact)?
- **Theorist**: does any reframe reveal you are solving the wrong problem? Is an
  off-graph move actually the right question?
- **Methodologist**: for the directions that make a claim, whose evidence or
  validity would actually hold? (load the mode checklist as usual)
- **Feasibility**: reality-check the off-graph moves: for a **[needs-search]**
  move, is the proposed tool plausibly real and findable, and is the switching
  cost worth it? This is where "go search for it" gets grounded.
- **Taste / PI**: would a paradigm shift matter more than incremental
  improvement here, or is the boring local move the honest best next step?

---

## Step 4: Chair synthesis (abstract-aware)

Synthesize once, as the council Chair, then add the two abstract-specific
sections. Output in chat, markdown, no files unless asked:

```
## Abstract Research: {short topic}
**Mode:** {computational | wet-lab | mixed} · **Panel:** {seats that ran}

### The spread
{one line per surviving take, Tier A and Tier B, with [in-reach]/[needs-search] tags on off-graph moves}

### Where the council converges / clashes
{high-confidence points; genuine disagreements, both sides}

### Incremental vs paradigm shift
{the strongest on-graph next move VS the strongest off-graph move, side by side, and the honest tradeoff between them}

### The call
{a clear recommendation: improve along the current graph, or jump to a different graph, and why. Not "it depends."}

### Do first
{ONE concrete next step}

### Off-graph leads to research (optional, involves egress)
{the [needs-search] directions worth chasing, written as explicit search questions you could hand to /hyperresearch. Empty if none survived.}
```

For **mixed** mode, "The call" must still reconcile prediction validity against
experimental feasibility, and note whether the paradigm shift changes that
balance.

---

## Operating rules

- The divergence stage is **upstream and lean**. It produces *framings to be
  critiqued*, not finished answers. The critics still dominate; this is
  anti-anchoring, not brainstorming for its own sake.
- Keep Tier B honest. An off-graph move that is just Tier A in disguise is a
  failure. If you cannot find a genuinely different paradigm, say so plainly
  rather than manufacture one.
- Do not perform external searches here. Surface **[needs-search]** leads as
  questions; let the user decide whether to spend egress on them.
- Everything else follows research-council: lean panel, one synthesis pass, no
  egress, do not run this on trivial questions with one right answer.
