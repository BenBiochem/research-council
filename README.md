# Research Council
<img width="1122" height="1402" alt="Research council: Six minds, one idea" src="https://github.com/user-attachments/assets/d024b3a7-c309-4a8a-8daa-dfd1114d9d99" />
A Claude skill that pressure-tests one research artifact through a critic-weighted
panel: Skeptic/Reviewer-2, Methodologist, Theorist, Taste/PI, Naive-reader,
Feasibility. Then synthesizes where they converge, where they clash, what they
all missed, and the one thing to do next.

Built for **research, computation, and paper-writing**. Adapted from Andrej
Karpathy's [LLM Council](https://github.com/karpathy/llm-council), with three
deliberate departures:

- **Critic-weighted, not idea-weighted.** In rigorous work the bottleneck is
  *selecting and killing* ideas, not generating them. The panel is built from
  the lenses that override an assistant's default agreeableness, where a forced
  perspective genuinely changes the output. No "growth/upside" seats.
- **Computational vs wet-lab aware.** The skill detects whether the work is
  *in silico* or *at the bench* (or both) and routes the Methodologist and
  Feasibility seats to the matching rigor checklist. This is the core feature.
- **No egress.** Every seat is a Claude sub-agent. Nothing leaves your machine,
  no third-party model sees the artifact. Safe on unpublished data, manuscripts,
  and designs.

> One-model honesty: the seats are Claude prompted several ways, not several
> independent models. That is real value for *critique* and weaker for
> *independent-knowledge triangulation*, the verdict leans on the reasoning
> surfaced, not on seats agreeing.

**Also included:** `abstract-research`, a companion mode that keeps the council
from getting stuck in one framing by putting *off-graph* alternatives on the
table first (a different paradigm or tool entirely, not just a new angle). See
[Advanced: abstract-research](#advanced-abstract-research) at the bottom.

## What it's for

Use it on decisions where being wrong is expensive:

- "Is this benchmark a fair comparison, or am I fooling myself?"
- "What will Reviewer 2 attack in this claim / figure / paragraph?"
- "Is this experiment sound controls, replicates, confounds?"
- "Should I pursue X or Y?" / "Which approach?"
- "My model predicts X, is it worth validating at the bench, and how?" (mixed)

Not for factual lookups, simple implementation, or questions answered by just
running the computation/experiment.

## Routing

| Phase | Base seats | + Computational | + Wet-lab |
|-------|-----------|-----------------|-----------|
| Pursue / prioritize | Taste, Theorist, Skeptic | (Methodologist) | (Feasibility) |
| Design | Skeptic, Theorist | Methodologist·comp, Feasibility·compute | Methodologist·wet, Feasibility·bench |
| Interpret | Skeptic, Theorist | Methodologist·comp | Methodologist·wet |
| Write / frame | Skeptic (Reviewer-2), Naive-reader | Methodologist·comp | Methodologist·wet |

**Mixed** mode runs the computational Methodologist and the wet-lab Feasibility
seat together, and the Chair reconciles *prediction validity* against
*experimental feasibility*. Panels stay lean (3–4 seats); synthesis is done once.

## Graphical Explanation
<img width="1448" height="1086" alt="Research council flowchart diagram" src="https://github.com/user-attachments/assets/bfa99581-a582-43df-b940-8c10ef04c83c" />

## Install

**As a plugin**: point your Claude Code plugin config at this repo, or clone it
into your plugins directory.

**Manually**: copy the skill folders into your personal skills directory:

```bash
cp -r plugins/research-council/skills/* ~/.claude/skills/
```

This installs both skills: `research-council` (the council itself: a `SKILL.md`
plus two mode checklists loaded conditionally) and `abstract-research` (the
divergence front-end described below).

## Usage

In any Claude conversation, trigger it with:

- "council this …"
- "reviewer 2 this …"
- "pressure-test / stress-test / red-team this …"
- "is this benchmark/experiment sound?"

Add **"deep"** for an extra anonymized cross-examination round on high-stakes
calls (doubles cost; off by default).

### Optional: project constraints

The skill imposes **no** method preferences of its own. If your project has
standing constraints (techniques to avoid or prefer), declare them and the
Feasibility seat will enforce them and flag violations, otherwise it applies
neutral feasibility judgment. Put a line in your project `CLAUDE.md` or your
request:

```
CONSTRAINTS: avoid <technique A>, <technique B>; prefer <technique C>
```

## Advanced: `abstract-research`

`research-council` has one structural blind spot: every seat gets a single framed
question, so the panel reasons inside one framing. A critic panel pointed at "how
do we improve X" also tends to only suggest *local* moves; it rarely says "stop
doing X this way at all."

`abstract-research` is a front-end that fixes both. Before the council
deliberates, it puts a divergent spread of candidate directions on the table in
two tiers:
<img width="1450" height="1085" alt="Abstract-research and cosmic thoughts" src="https://github.com/user-attachments/assets/68db6247-7b00-40ad-ab42-f3eda0d49580" />
- **On-graph reframes:** orthogonal angles on the current problem (literal,
  reframe, inversion, first-principles, analogy).
- **Off-graph moves:** a completely different paradigm, tool, or protocol for the
  *same goal*, abandoning the current approach rather than improving it. "An
  angle not even on the same graph." These may propose a tool you do not have yet
  and name what to search for to find it.

The council then critiques across the whole spread, so it cannot get stuck
improving one approach when a better one lies off the current graph. The verdict
weighs the strongest incremental move against the strongest paradigm shift and
makes a call.

Trigger it with "abstract-research this", "what is our next move", or "is there a
completely different approach". Same no-egress guarantee as the council: it
*suggests* what to search for, it does not search. Off-graph leads that need a
new tool come out as explicit research questions you can optionally pursue with a
deep-research tool (which does involve web egress).

## Credit

Created by **Benjamin E. Peterson**. If you build on this, please keep the credit
(and retain the MIT notice); see the license below.

Methodology adapted from Andrej Karpathy's
[LLM Council](https://github.com/karpathy/llm-council).

## License

MIT © 2026 Benjamin E. Peterson. See [LICENSE](LICENSE). The MIT notice must be
retained in copies and derivative works, which preserves attribution.
