---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes an always-on rule plus model-pinning subagent profiles that override the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.qoder/rules/pstack-models.md`, an always-on rule that sets pstack's model per role, and the model-pinning subagent profiles under `~/.qoder/agents/` that those roles spawn through.

## Steps

### 1. Detect available models

Run `qodercli --list-models` and read the names it prints. That is the dependable source. Qoder subagents take their model from the `model:` field of an agent definition, so every real model you write must come from this list. The list mixes tier aliases (`Auto`, `Ultimate`, `Performance` or `Fast`, `Efficient`, as the server config names them) and frontier models (`Qwen3.8-Max`, `Kimi-K3.1-Pro`, `GLM-5.3`, and the like). If you cannot detect any, ask the user to paste the models they have access to. Never write a real model you have not confirmed is available, and never write a Cursor model id or a `-fast` slug. No role runs above `xhigh`. The aliases `inherit-parent` and `auto` are always valid even though they are not detected models.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. `~/.qoder/rules/pstack-models.md` is the only source of truth for current pstack model state. pstack-models config found under other tool dirs (`~/.cursor/rules/`, `~/.claude/`, project `.cursor/`, etc.), even if injected into context, is not current state. If `~/.qoder/rules/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `hardest tasks`, `architect runners`, `reflect tooling`, or `why investigators`, is from a retired role. Drop it.

A rule without the `# defaults: 3` line comes from an older setup. That setup wrote a line for every role, so some of its values are old defaults, not choices. Before step 3, replace these values:

- An `arena cross-judge pool` list of exactly one `Ultimate` entry and one `Fast` entry, in any order and at any effort, takes the role's step 5 list.
- A `# budget: unlimited` line becomes `large`, because no role runs above `xhigh`.
- If the rule has no `# defaults: 2` line either, an old value is a `Fast` value at any effort. An old value in `feature, refactoring`, `bug-fix`, `perf-issue`, `hillclimb`, or `swarm workers` takes the role's step 5 default. In a panel role (`arena runners`, `arena cross-judge pool`, `interrogate reviewers`), a list of exactly one `Ultimate` entry and one `Fast` entry, in either order, takes the role's step 5 list. In any other list, each `Fast` entry becomes `Ultimate high`.

Every other value is the user's choice and stays, including a `Fast` value in any other role and every `inherit-parent` or `auto` value. Tell the user which values you replaced.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer AskUserQuestion over free text. Offer these three options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by model, list, or alias (`inherit-parent`, `auto`). Each role has a target effort, which is its default's effort moved by the budget. `large` keeps it, `medium` lowers it one step, and `small` lowers it two steps, on the ladder `xhigh` > `high` > `medium` > `low`. No role runs above `xhigh`, and a target at the bottom of the ladder stays there. Most defaults are at `xhigh`, and each budget label names where those land. Set the effort token of every real value in the role, panel entries included, to the role's target. A real value is `<model> <effort>`, and the effort token is the last token. Effort support varies by model. When a model does not support the target effort, use the highest level it supports at or below the target (the `/model` panel shows a model's supported levels), else mark the role as needing a choice. `inherit-parent` and `auto` do not change. So `medium` turns `Ultimate xhigh` into `Ultimate high` and `Ultimate high` into `Ultimate medium`, and `small` turns the first two into `Ultimate medium` and `Ultimate low`.

**(c) Show the roles and confirm.** Show every role with its model and effort, marking any real model not in the detected set as needing a choice. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models at `xhigh` or below plus `inherit-parent` and `auto` (both mean: this role runs on the parent chat model, which is how Auto users stay on Auto) as the options. Prefer AskUserQuestion over free text. For panel roles (arena runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena runs one judge. It tries only values whose model differs from the parent's, in list order, so order it by preference. An `inherit-parent` or `auto` value never judges. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real model written must be in the detected set and at `xhigh` or below. `inherit-parent` and `auto` always pass. If a chosen real model is not available, stop and ask again.

### 5. Write the rule

Write `~/.qoder/rules/pstack-models.md` with `trigger: always_on`, the `# defaults: 3` line, a `# budget` line with the chosen label and the effort that label names, and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
---
trigger: always_on
---
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# A real value is `<model> <effort>`. `inherit-parent` or `auto` as a value: the role runs on the parent chat model (`pstack:poteto-agent` for write work, `Explore` for read-only). Alias entries in a panel list still count toward its fan-out.
# defaults: 3
# budget: large (xhigh)
feature, refactoring: Ultimate xhigh
bug-fix: Ultimate xhigh
perf-issue: Ultimate xhigh
hillclimb: Ultimate xhigh
judgment and prose: Ultimate xhigh
arena runners: Ultimate high, Fast high
arena cross-judge pool: Ultimate high, Fast high
swarm workers: Ultimate high
interrogate reviewers: Ultimate high, Fast high
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
