# Praxec discoverability — dogfooding finding

**Context:** A frontier agent (Claude), with the `praxec` MCP two-tool surface
available the whole session, hand-rolled and then dispatched build agents to
*create* workflows that **already existed, were loaded into the gateway, and were
returned by a single `praxec.query` search**. Discovery only happened after the
human operator named the repos. This is a first-person account of the failure mode
so the ranking/surfacing can be fixed.

## What was rebuilt-instead-of-discovered

| I was building… | …that already exists (one search away) |
|---|---|
| A JTBD+cog-sci UX-optimization/annealing flow | `cognitive-max/flow.ui.optimal` — "HCI+JTBD design → customer-alignment FMECA → signoff → TDD → ui-green → react review → adversarial → PR, iterated to perfection via grounded UX measurement" |
| A "UX engineer" agent + per-principle analysis capability | `cognitive/cap.plan.ux-design` — "JTBD + cognitive-science lens; falsifiable weak assumptions w/ suggested_test + target_metric" |
| An elicitation→persisted-intent step | `cognitive-max/flow.ux.discovery` — "interview a vague brief → gate on elicitation's deterministic readiness → export spec" |
| The premium *os-grounded vet+fix | `flow.ux.vet-and-fix` — "uxos audit → grounded IntentOS proposals via propose_and_park" |
| An HOQ / cross-matrix capability | `uxos.check_coverage` — "crossmatrix drift vs intent (stale/orphan/uncovered)" |
| Adversarial review / TDD gates / PR (hand-rolled with ad-hoc subagents) | `cap.review.*`, `cognitive/cap.implement.build-loop` (engine-enforced red/green), `meta/flow.author-flow`, `flow.add-ui-feature` |

Cost: three build agents launched and killed; a whole session of ad-hoc scaffolding
for gates that ship as governed flows.

## Root causes

1. **Discovery is pull-only and un-prompted.** Nothing nudges an agent toward an
   existing workflow at the moment of need. The default behavior is to hand-roll.
2. **Flat ranking — no "canonical" signal.** Every workflow scored ~`0.65`:
   `topology 0.000 (no registry tool links)` + `evidence 0.500 (none at/above
   min_runs)`. So `flow.ui.optimal` (the canonical UX flow) ranks identically to
   unrelated capabilities. There is no signal that says "this is THE flow for X."
3. **Cold-start evidence trap.** Ranking leans on run-history (`min_runs`); an
   un-run flow is permanently neutral. Great new flows can't rank up until run,
   but they aren't run because they aren't surfaced. Chicken-and-egg.
4. **Narrow default listing.** `praxec.query {kind: workflow, query: ""}` returned
   ~10 `cognitive-max/*` items; the broader set (frontrails, `meta/`, `cognitive/`)
   only appeared under a *targeted* query. The "show me everything" view
   under-represents what's reachable.
5. **On-disk vs. loaded opacity.** Several packs exist on disk
   (cognitive-architectures, frontrails-praxec-pack, praxec-meta); which are
   *loaded* into the running gateway isn't visible without probing.
6. **Empty `tags`/`aliases`.** The shipped flows carry `tags: []`, `aliases: []`.
   Matching rests on description prose alone.
7. **Meta-flows aren't surfaced at authoring time.** `meta/flow.author-flow` is
   "the canonical way to author a flow" — but nothing intercepts an agent about to
   hand-write YAML (or spin up a build agent) to point at it.

## Recommendations (ranked by leverage)

- **A. Task-triggered surfacing.** When an agent begins a recognizable task
  (author a flow/cap, "review", "UX", "bugfix") the gateway — or a shipped
  AGENTS.md convention — surfaces the top existing matches first. Cheapest form: a
  strong shipped convention: *"Before hand-rolling multi-step work or authoring a
  flow, `praxec.query` for an existing one."*
- **B. Canonical/topology signal.** Let a flow declare `canonical_for: [ux, …]`
  (or be linked in a registry topology) so `topology` isn't 0.000 for the intended
  flow, and boost it so THE flow outranks incidental matches.
- **C. Cold-start priors.** Seed evidence from static signals (passes
  `praxec check`, referenced-by other flows, a curated `blessed` tier) so un-run
  flows aren't permanently neutral.
- **D. Richer default list.** Empty-query `list_workflows` should return a grouped
  cross-namespace overview (by verb/domain), not one namespace's first N.
- **E. Loaded-vs-available transparency.** A query that shows loaded vs.
  on-disk-but-unregistered packs, with the one-liner to register.
- **F. Curate `tags`/`aliases`** on shipped flows (mechanical, high ROI).
- **G. Author-time poka-yoke.** `author-flow`/`author-capability` should run a
  "does this already exist?" search first and require the operator to confirm
  "none fit" before drafting — a guard against rebuilding.

## The meta-point

The pattern recurred at three levels this session — hand-rolled review/gates/PR
(exists as flows), then built UX flows (exist), then almost built a shape catalog
(the flow library *is* the shape exemplars). The tooling is excellent; it is just
**not discoverable at the moment of need.** Fixing surfacing + ranking (A–C) would
convert "rediscovered only when a human points" into "proposed by default."

## Addendum: the human gate elicits without context (live finding #2)

Driving `meta/flow.author-flow` for real, the `picking_shape` state parked on
`meta/cap.gate.human-pick-shape` and surfaced to the human operator via MCP
elicitation as, verbatim: *"Mission wf_… is waiting on you to 'pick'"* — a bare
optional-note field with Accept/Decline. **None of the three candidate shapes,
their names, approaches, or trade-offs were shown.** The operator's reaction:
"not good enough to elicit anything… there's absolutely no context or info."

The gate's entire purpose is *informed* human sign-off, and the candidates WERE
sitting in the workflow context (`context.candidates[]`, each with `name`,
`approach`, `tradeoffs`, `tool_stack`, `state_count`, `requires_new`) — the
elicitation just didn't carry them. Recommendation: human-gate transitions
should declare a `presentation` (or the elicitation should render the gate's
declared context slice — e.g. `$.context.candidates`) so the decision payload
reaches the human. A gate that transmits no context degrades to a rubber stamp,
which silently converts HITL governance into noise.
