# Changelog

## 2026.10.11

research-council
- Seat prompt: findings must quote the line of the artifact they rest on; "no
  material flaw in my lane" is a valid answer; the Skeptic no longer assumes a
  fatal flaw exists.
- Seats return a lane verdict (sound, flawed, cannot tell) and severity-tagged
  findings with the cheapest check that would confirm or kill each one.
- Chair checks every fatal or major finding against the artifact before
  reporting it, and "sound enough to proceed" is a valid call.
- Output: "Convergence: high confidence" is now "Shared findings" with evidence
  quotes; "Blind spots" is now "Gaps and unverified"; new "What would change
  the call"; the Panel line reports failed seats and whether the panel was
  independent.
- Seats receive the user's verbatim request and the artifact (or file paths),
  not only the Chair's summary. The framed question is printed before spawning.
- Quorum rule (stop below 2 seats) and a single-context fallback when no
  sub-agent tool exists.
- New `council-seat` agent restricted to Read, Glob, Grep.
- Taste added to the Write / frame panel; Taste now lists prior-work claims it
  could not verify.
- Checklists: removed project-specific wording and a technique preference from
  the wet-lab Feasibility list; added a null-model and uncertainty item
  (computational) and a blinding and exclusion-rules item (wet-lab).

abstract-research
- Divergence reduced to 4 takes (2 per tier); cost note added.
- Tools proposed by off-graph moves are marked unverified until searched.

Repo
- README: concrete install commands, sub-agent requirement, accurate egress
  wording.
- Plugin version bumped (previously predated abstract-research).
