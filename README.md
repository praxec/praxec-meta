# praxec-meta

> Self-authoring workflows for [praxec](https://github.com/anthropics/praxec) v0.6+.
>
> Capabilities and orchestrators whose job is to **author** or
> **optimize** other capabilities and orchestrators.

---

## Why this exists

praxec v0.6 introduces a two-tier composition model: typed
**capabilities** (`cap.*`) invoked by lifecycle **orchestrators**
(`flow.*`) via `use:` bindings. Authoring those tiers by hand is
mechanical — pick a verb, draft a snippet contract, declare an
initial state machine, run `praxec check`, iterate.

This library encodes that authoring loop as four meta-orchestrators
operators can run from any praxec-backed agent:

| Orchestrator | What it does |
|---|---|
| `meta/flow.author-capability` | Draft a brand-new capability from a description + the operator's available tools |
| `meta/flow.author-flow` | Draft a brand-new orchestrator that composes existing capabilities |
| `meta/flow.optimize-capability` | Refine an existing capability based on audit history + adversarial review |
| `meta/flow.optimize-flow` | Refine an existing orchestrator (state-shape, branch reduction, etc.) |

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

Tracks the underlying praxec spec version. v0.1 of this repo
targets praxec v0.6.x.

---

## License

MIT — see LICENSE.
