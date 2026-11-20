# Qoder compatibility

pstack 0.15.46 runs on Qoder. The core poteto-mode workflow is compatible. This file records what is verified, what needs setup, and what is out of scope.

## Verified compatible

- The manifest `.qoder-plugin/plugin.json` and `marketplace.json` use the Qoder plugin format.
- Both agents, `poteto-agent` and `comment-sicko`, register and appear in the Agent tool list.
- The plugin ships 61 skills. Four are model-invocable: `control-cli`, `control-ui`, `deslop`, and `setup-pstack`. The other 57 set `disable-model-invocation: true`, so Qoder hides them from model invocation but keeps them slash-invocable and file-routable.
- The skills use Qoder tool names such as `Agent`, `TaskCreate`, and `Skill`. No Cursor tool name or `cursor-agent` CLI reference remains.
- The default models `Fast` and `Ultimate` are valid names in `qodercli --list-models`. Code, prose, and judgment default to `Ultimate xhigh`. Trivial mechanical edits and swarm workers default to `Ultimate high`. Review panels default to `Ultimate high` and `Fast high`. No role runs above `xhigh`.
- Every `subagent_type` reference points to a registered agent.

## Bugbot detection

`skills/poteto-mode/scripts/watch-pr/github.ts` detects bugbot comments without a host-specific account or marker. `isBugbot` returns true when the author login contains `bugbot`, or when the body carries a bot marker: `automation_id`, `run_id`, `agentic security review`, or `bugbot`. `passKey` reads `RUN_ID:` and `AUTOMATION_ID:` with any prefix, so `CURSOR_AUTOMATION_ID:` and `QODER_AUTOMATION_ID:` both parse.

Qoder's exact bugbot login and marker are unverified. Detection covers any host whose bot login contains `bugbot` or whose comments carry these markers. A host that uses a different marker needs a new entry in the `isBugbot` body-token list and the `passKey` regex list.

## Model setup

Run `/setup-pstack` to write `~/.qoder/rules/pstack-models.md` and the model-pinning profiles under `~/.qoder/agents/`. The rule carries a `# defaults: 3` line and a `# budget` line. Budgets are `large` (`xhigh`), `medium` (`high`), and `small` (`medium`). No role runs above `xhigh`.

Upstream 0.15.44 and 0.15.45 moved the Cursor Gemini panel seat from `gemini-4-carbon-high` to `gemini-4-neon-high`, then to `gemini-4-neon-xhigh`, and added migration rules for those slugs. This port's second panel seat is `Fast`, already at `high`. That seat does not follow the Gemini effort change. Those slugs and their migration rules are not written into the defaults, the skills, or the setup-pstack rule. Panel defaults stay `Ultimate high` and `Fast high`.

A rule without the `# defaults: 3` line is from an older setup. On the next run, a line whose role is not in the current shape is dropped. That includes the retired roles `hardest tasks`, `architect runners`, `reflect tooling`, `reflect judgment, divergent, synthesizer`, `how explorer`, `why investigators`, and `why synthesizer`. A `# budget: unlimited` line becomes `large`, because no role runs above `xhigh`. An `arena cross-judge pool` that is exactly one `Ultimate` entry and one `Fast` entry takes the current default list. If the rule also has no `# defaults: 2` line, a `Fast` value at any effort in `feature, refactoring`, `bug-fix`, `perf-issue`, `hillclimb`, or `swarm workers` takes that role's default, and a panel list that is exactly one `Ultimate` entry and one `Fast` entry takes that role's default list. In any other list from that older rule, each `Fast` entry becomes `Ultimate high`. Every other value stays as the user's choice. Until you run setup, roles use the documented auto fallback: `Explore` for read-only work and `poteto-agent` for write work. The fallback works. The pinned-model path stays inactive until setup runs.

## Transcript layout

Qoder writes transcripts at `~/.qoder/projects/<slug>/<session-id>.jsonl`. Subagent transcripts sit under `~/.qoder/projects/<slug>/<session-id>/subagents/`. `<slug>` is qodercli 1.1.89 `Vf(cwd)`, the same function `join(configDir, "projects", Vf(cwd), sessionId + ".jsonl")` uses. The dash step runs on UTF-16 code units, the same units `charCodeAt` reads, so a non-BMP character becomes two dashes. Every code unit that is not an ASCII letter or digit becomes `-`, so `/home/me/my_proj` is `-home-me-my-proj`. When the dashed form is longer than 200 characters, the slug is the first 200 characters, a `-`, and the base-36 form of `Math.abs(t)`. Hash the original path, not the dashed string. `t` starts at `5381`. Each UTF-16 code unit updates it with `t = 33 * t ^ charCodeAt(i)`, and `^` is signed 32-bit. `Math.abs(-2147483648)` is `2147483648`.

## Out of scope

The benny Slack automations are not a compatibility target. Qoder has no surface that creates a Slack-triggered automation, so the pack stays dormant and unsupported. `automations/benny/FOR_AGENTS.md` records this limit.
