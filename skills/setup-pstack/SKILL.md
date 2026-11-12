---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes an always-on rule plus model-pinning subagent profiles that override the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.qoder/rules/pstack-models.md`, an always-on rule that sets pstack's model per role, and the model-pinning subagent profiles under `~/.qoder/agents/` that those roles spawn through.

## Steps

### 1. Detect available models

Run `qodercli --list-models` and read the names it prints. That is the dependable source. Qoder subagents take their model from the `model:` field of an agent definition, so every real model you write must come from this list. The list mixes tier aliases (`Auto`, `Ultimate`, `Performance` or `Fast`, `Efficient`, as the server config names them) and frontier models (`Qwen3.8-Max`, `Kimi-K3.1-Pro`, `GLM-5.3`, and the like). If you cannot detect any, ask the user to paste the models they have access to. Never write a real model you have not confirmed is available. The aliases `inherit-parent` and `auto` are always valid even though they are not detected models.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. `~/.qoder/rules/pstack-models.md` is the only source of truth for current pstack model state. pstack-models config found under other tool dirs (`~/.cursor/rules/`, `~/.claude/`, project `.cursor/`, etc.), even if injected into context, is not current state. If `~/.qoder/rules/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer AskUserQuestion over free text. Offer these four options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `unlimited — max reasoning`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by model, list, or alias (`inherit-parent`, `auto`). `unlimited`, `large`, `medium`, and `small` set the effort token of every real value, panel entries included, to `max`, `xhigh`, `high`, or `medium`. A real value is `<model> <effort>`, and the effort token is the last token, on the ladder `max` > `xhigh` > `high` > `medium` > `low`. Effort support varies by model. When a model does not support the target effort, use the highest level it supports at or below the target (the `/model` panel shows a model's supported levels), else mark the role as needing a choice. `inherit-parent` and `auto` do not change. So `unlimited` turns `Ultimate xhigh` into `Ultimate max`. `large` keeps both defaults. `small` turns them into `Ultimate medium` and `Fast medium`.

**(c) Show the roles and confirm.** Show every role with its model and effort, marking any real model not in the detected set as needing a choice. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` and `auto` (both mean: this role runs on the parent chat model, which is how Auto users stay on Auto) as the options. Prefer AskUserQuestion over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena runs one judge. It tries only values whose model differs from the parent's, in list order, so order it by preference. An `inherit-parent` or `auto` value never judges. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real model written must be in the detected set. `inherit-parent` and `auto` always pass. If a chosen real model is not available, stop and ask again.

### 5. Write the rule

Write `~/.qoder/rules/pstack-models.md` with `trigger: always_on`, a `# budget` line with the chosen label and its target effort, and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
---
trigger: always_on
---
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# A real value is `<model> <effort>`. `inherit-parent` or `auto` as a value: the role runs on the parent chat model. Alias entries in a panel list still count toward its fan-out.
# budget: large (xhigh)
feature, refactoring: Fast xhigh
bug-fix: Fast xhigh
perf-issue: Fast xhigh
hillclimb: Fast xhigh
judgment and prose: Ultimate xhigh
hardest tasks: Ultimate xhigh
how explorer: Fast xhigh
how explainer: Ultimate xhigh
why investigators: Fast xhigh
why synthesizer: Ultimate xhigh
reflect tooling: Fast xhigh
reflect judgment, divergent, synthesizer: Ultimate xhigh
arena runners: Ultimate xhigh, Fast xhigh
arena cross-judge pool: Ultimate xhigh, Fast xhigh
swarm workers: Fast xhigh
architect runners: Ultimate xhigh, Fast xhigh
interrogate reviewers: Ultimate xhigh, Fast xhigh
```

### 6. Write the model profiles

For each distinct real value `M E` (model `M`, effort `E`) in the rule, write two subagent profiles. Skills spawn them as `Agent` types `pstack-M-E` (write-capable) and `pstack-M-E-ro` (read-only). Skip `inherit-parent` and `auto` values; those map to `pstack:poteto-agent` and `Explore` and need no file.

`~/.qoder/agents/pstack-M-E.md`:

```
---
name: pstack-M-E
description: pstack delegate pinned to model M at effort E, spawned by pstack skills for write work.
model: M
effort: E
background: true
---
You are a pstack delegate. Read the `poteto-mode` skill's `SKILL.md` in full before doing any work, including its inline Principles index. Use the absolute path your parent gave you. With no path, search `~/.qoder/plugins/` for a file whose path ends in `skills/poteto-mode/SKILL.md`. If more than one file matches, use the copy whose `.qoder-plugin/plugin.json` has the highest `version`. If none exists, stop and tell your parent. Work the brief your parent passed and report evidence, not claims.
```

`~/.qoder/agents/pstack-M-E-ro.md`:

```
---
name: pstack-M-E-ro
description: Read-only pstack reviewer pinned to model M at effort E, spawned by pstack skills for research and review.
model: M
effort: E
background: true
disallowedTools: [Edit, Write, NotebookEdit]
---
You are a read-only pstack subagent. Work the brief your parent passed. Report findings with file-and-line evidence. Do not edit files or run commands that mutate state.
```

List any existing `~/.qoder/agents/pstack-*.md` files whose model and effort are no longer in the rule and offer to remove them.

### 7. Confirm

Tell the user the rule and profiles were written. The rule applies to the next turn. The profiles become spawnable after `/agents reload` or a new session, and `qodercli agents list` shows them. Re-running this skill updates both.

### 8. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.

### 9. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke `/create-formal-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.