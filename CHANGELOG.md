# Changelog

## Unreleased

- **`cap.verify.capability-harness` + `verify.capability-harness` script** — per-affinity capability verification. Runs a golden contract (prompt + verifier) against every binding in `agents.yaml` whose override-key affinity matches the contract. Complements (not replaces) `cap.verify.auth-only-smoke-test`: auth-only proves credentials in ~1s; capability-harness proves the model can do the work for the affinity it's slotted under. Verifier kinds today: `exact_number` (parses first integer, compares equals), `regex_match` (Python re.search). Per-binding output names pass/fail with verifier detail + elapsed ms; exit 0 always (gating decisions live in the orchestrator).
- **`contracts/`** — golden contracts directory. Ships three starters: `reasoning-arithmetic-v1.yaml` (123×456 = 56088, exact-number verifier), `coding-rust-fn-v1.yaml` (Rust `fn add(a,b)` signature, regex verifier), `prose-coffee-mug-v1.yaml` (constrained-length product description, regex verifier as length proxy). Operators author their own contracts at the same shape — the script reads the YAML structure described in the cap docstring.
- **`flow.configure-models` optional capability step** — new input `capability_contract` (path; default `""`). When non-empty, runs `cap.verify.capability-harness` against the just-written agents.yaml using the named contract; empty skips. Guard reads via `$.workflow.input.capability_contract` so the empty-default case doesn't trip `GUARD_UNSET_SLOT`.

## 0.1.0 (unreleased)

First public release. Targets praxec v0.6.x.

- Four meta-orchestrators (`flow.author-capability`, `flow.author-flow`,
  `flow.optimize-capability`, `flow.optimize-flow`).
- Shared cap stack: `cap.research.tool-inventory`,
  `cap.research.lexicon-lookup`, `cap.plan.compose-implementation`,
  `cap.gate.human-pick-shape`, `cap.implement.emit-yaml`,
  `cap.verify.check-config`, `cap.summarize.lexicon-define`,
  `cap.audit.mine-transitions`.
- Stub-shadows of `cap.review.adversarial` + `cap.coordinate.pr-open`
  so this repo is loadable standalone. Operators who also load
  cognitive-architectures should declare these in their top-level
  `overrides:` block.
- Happy-path-only orchestrators (no guarded branches) — same v0.6
  shape-trap workaround as cognitive-architectures.
