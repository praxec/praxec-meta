# praxec-meta

> Self-authoring workflows for [praxec](https://github.com/anthropics/praxec) 0.0.14+.
>
> Capabilities and flows whose job is to **author** or **optimize**
> other capabilities and flows.

---

## Why this exists

praxec composes work in two authored tiers: typed **capabilities**
(`cap.*`) invoked by lifecycle **flows** (`flow.*`) via `use:`
bindings. Flows may also compose other flows (V11 is relaxed — a flow
can invoke a nested flow, bounded at runtime by a sub-workflow depth
cap). Authoring those tiers by hand is mechanical — pick a verb, draft
a snippet contract, declare an initial state machine, run `praxec
check`, iterate.

This library encodes that authoring loop as four meta-flows operators
can run from any praxec-backed agent:

| Flow | What it does |
|---|---|
| `meta/flow.author-capability` | Draft a brand-new capability from a description + the operator's available tools |
| `meta/flow.author-flow` | Draft a brand-new flow that composes existing capabilities (and/or other flows) |
| `meta/flow.optimize-capability` | Refine an existing capability based on audit history + adversarial review |
| `meta/flow.optimize-flow` | Refine an existing flow (state-shape, branch reduction, etc.) |

A fifth flow, `meta/flow.configure-models`, writes the operator's
`.praxec/models.yaml` (the `gateway.models_yaml` file the resolver
reads for `kind: agent` and affinity-resolved `kind: llm` steps).

The point: **what each operator can reach for differs.** One operator
has a dozen MCP servers; another has only CLI tooling and a small
script library. The first state of every meta-workflow is a
tool-inventory cap that introspects what's actually loaded — the rest
of the orchestrator composes against that inventory rather than a
hardcoded stack.

---

## Install

Clone next to your existing praxec repos:

```
git clone <this repo> /repos/praxec-meta
```

Then in your gateway config:

```yaml
# gateway.yaml
version: "1.0.0"
repos:
  - path: /repos/cognitive-architectures   # (or whatever resource repos you use)
  - path: /repos/praxec-meta
```

Validate the load:

```
praxec check --config gateway.yaml
```

You should see the four `meta/flow.*` ids alongside whatever the
other repos provide.

---

## Shared cap stack

The four orchestrators share a small reusable cap stack — these are
where the introspection / generation / verification live:

| Cap | Used by | Verb |
|---|---|---|
| `cap.research.tool-inventory`     | all 4   | research |
| `cap.research.lexicon-lookup`     | all 4   | research |
| `cap.plan.compose-implementation` | all 4   | plan     |
| `cap.gate.human-pick-shape`       | all 4   | gate     |
| `cap.implement.emit-yaml`         | all 4   | implement |
| `cap.verify.check-config`         | all 4   | verify   |
| `cap.review.adversarial`          | all 4   | review (from cognitive-architectures if available, stubbed here otherwise) |
| `cap.audit.mine-transitions`      | optimize-* only | audit |
| `cap.summarize.lexicon-define`    | author-* (for new vocabulary) | summarize |
| `cap.coordinate.pr-open`          | all 4 (terminal step) | coordinate (from cognitive-architectures if available, stubbed here otherwise) |

`cap.review.adversarial` and `cap.coordinate.pr-open` are duplicated
here as thin stubs so this repo is loadable standalone; operators
who also load `cognitive-architectures` can declare those names in
their top-level `overrides:` block to win the collision with the
richer cognitive-architectures versions.

---

## Versioning

Tracks the underlying praxec version. This repo (0.3.0) targets
praxec 0.0.14.

---

## License

MIT — see LICENSE.
