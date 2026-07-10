# Changelog

## Unreleased

### praxec 0.0.14 alignment (0.3.0)

- **`plan.suggest-bindings`: reasoning models are safe as chain leads/rungs.** The binding-composer skill now states that a *reasoning* model may lead a `default:`/override list or serve as any CoR rung, because the engine caps per-turn reasoning on every `kind: agent` step that sets no explicit `reasoning_effort` (`ReasoningTuning.default_effort`, `low` by default) — so a reasoning model no longer burns its whole turn budget on hidden reasoning and returns empty content (which the agent loop would classify as an `AGENT_NO_RESULT` stall). The skill now forbids demoting/deprioritizing/burying a model for latency that is actually reasoning-token burn, and tells the composer to prefer the engine reasoning default over hardcoding a per-model `reasoning_effort` (noting `medium` is a no-op ≡ provider default; the only value that tightens the cap is `low`). No vendor/model is hardcoded as good or bad.
- **`agents.yaml` → `models.yaml` cutover.** The engine's canonical model-binding file is now `.praxec/models.yaml` (the `gateway.models_yaml` key, loaded via `ModelsFile::from_path`). `flow.configure-models` now defaults `target_path` to `.praxec/models.yaml`. Renamed `cap.implement.write-agents-config` → `cap.implement.write-models-config` and script `install.agents-config` → `install.models-config`. Smoke/capability caps take `models_config_path` (was `agents_config_path`). The round-trip validator is `px validate-agents-config` (a `px`-binary subcommand; its help documents it as `validate-models-config`) — override the binary via `$PX_BIN`.
- **`emit`-step `kind` discriminator fix.** All four author/optimize flows passed the goal string / target id as the emit cap's `kind` input; that input is a closed enum `[capability, orchestrator]`, so it now receives the correct literal (`capability` for author/optimize-capability, `orchestrator` for author/optimize-flow).
- **`check-config` now checks the parent gateway config.** `cap.verify.check-config` took `config_path` (the emitted file in isolation, which has no parent context for V12/V13/V14/contract-hash/connection checks) and relied on a `CONFIG_PATH` env var the script executor never sets. It now takes `gateway_config_path`, passed as a positional script arg; the author/optimize flows thread a `gateway_config_path` input (default `praxec.yaml`). NOTE: the emit step still does not persist the YAML to disk — see the open gap below.
- **Script-input env→arg fix.** `cap.audit.mine-transitions` (and `check-config`) relied on env vars (`DEFINITION_ID`/`SINCE`/`CONFIG_PATH`) that the script executor does not inject; inputs now arrive via positional `args:`. JSON stdout is now projected from `$.output.json` (was `$.output`).
- **Two-tool MCP surface.** The gateway exposes exactly `praxec.query` / `praxec.command`. The lexicon caps (`cap.research.lexicon-lookup`, `cap.summarize.lexicon-define`) now target those tools with a `map:` block and `subject: "lexicon:<term>"` shape (was `tool: gateway.lexicon.*` + an ignored `arguments:` block). `cap.coordinate.pr-open` likewise switched `arguments:` → `map:`.
- **Affinity / provider enum refresh.** Added the `agentic` affinity (plan skill + capability-harness). `models.yaml` provider slugs corrected: Google is `gemini` (not `google`); local providers route through `custom{endpoint}` / `llamacpp`; `openrouter` / `bedrock` added.
- **V11 relaxed + authoring-surface notes.** The emit skill now documents that flows may nest flows (V11 relaxed, depth-bounded), the connection schema (`kind:` + `command:`/`url:`), the two-tool surface, and that cap + flow are the only engine authoring tiers (no `hop_slot` primitive in core).

### Earlier (pre-alignment)

- **`cap.verify.capability-harness` + `verify.capability-harness` script** — per-affinity capability verification. Runs a golden contract (prompt + verifier) against every binding in `models.yaml` whose override-key affinity matches the contract. Complements (not replaces) `cap.verify.auth-only-smoke-test`: auth-only proves credentials in ~1s; capability-harness proves the model can do the work for the affinity it's slotted under. Verifier kinds today: `exact_number` (parses first integer, compares equals), `regex_match` (Python re.search). Per-binding output names pass/fail with verifier detail + elapsed ms; exit 0 always (gating decisions live in the flow).
- **`contracts/`** — golden contracts directory. Ships three starters: `reasoning-arithmetic-v1.yaml` (123×456 = 56088, exact-number verifier), `coding-rust-fn-v1.yaml` (Rust `fn add(a,b)` signature, regex verifier), `prose-coffee-mug-v1.yaml` (constrained-length product description, regex verifier as length proxy). Operators author their own contracts at the same shape — the script reads the YAML structure described in the cap docstring.
- **`flow.configure-models` optional capability step** — new input `capability_contract` (path; default `""`). When non-empty, runs `cap.verify.capability-harness` against the just-written models.yaml using the named contract; empty skips. Guard reads via `$.workflow.input.capability_contract` so the empty-default case doesn't trip `GUARD_UNSET_SLOT`.

## 0.1.0 (unreleased)

First public release. Targeted praxec v0.6.x (superseded by the
0.0.x line — see the alignment entry above).

- Four meta-flows (`flow.author-capability`, `flow.author-flow`,
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
