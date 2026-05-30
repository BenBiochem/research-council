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

<img width="1002" height="731" alt="Screenshot from 2026-05-30 02-04-15" src="https://github.com/user-attachments/assets/f9d62897-1d32-4f62-a7e8-2b34cb302090" />

**Mixed** mode runs the computational Methodologist and the wet-lab Feasibility
seat together, and the Chair reconciles *prediction validity* against
*experimental feasibility*. Panels stay lean (3–4 seats); synthesis is done once.

## Install

**As a plugin**: point your Claude Code plugin config at this repo, or clone it
into your plugins directory.

**Manually**: copy the skill folder into your personal skills directory:

```bash
cp -r skills/research-council ~/.claude/skills/
```

The skill is self-contained: `SKILL.md` plus two mode checklists
(`computational-rigor.md`, `wetlab-rigor.md`) loaded conditionally.

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

## Credit

Created by **Benjamin E. Peterson**. If you build on this, please keep the credit
(and retain the MIT notice) — see the license below.

Methodology adapted from Andrej Karpathy's
[LLM Council](https://github.com/karpathy/llm-council).

## License

MIT © 2026 Benjamin E. Peterson — see [LICENSE](LICENSE). The MIT notice must be
retained in copies and derivative works, which preserves attribution.
