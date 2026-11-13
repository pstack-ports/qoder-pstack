# Qoder compatibility

pstack 0.15.36 runs on Qoder. The core poteto-mode workflow is compatible. This file records what is verified, what needs setup, and what is out of scope.

## Verified compatible

- The manifest `.qoder-plugin/plugin.json` and `marketplace.json` use the Qoder plugin format.
- Both agents, `poteto-agent` and `comment-sicko`, register and appear in the Agent tool list.
- The plugin ships 61 skills. Four are model-invocable: `control-cli`, `control-ui`, `deslop`, and `setup-pstack`. The other 57 set `disable-model-invocation: true`, so Qoder hides them from model invocation but keeps them slash-invocable and file-routable.
- The skills use Qoder tool names such as `Agent`, `TaskCreate`, and `Skill`. No Cursor tool name or `cursor-agent` CLI reference remains.
- The default models `Fast` and `Ultimate` are valid names in `qodercli --list-models`.
- Every `subagent_type` reference points to a registered agent.

## Bugbot detection

`skills/poteto-mode/scripts/watch-pr/github.ts` detects bugbot comments without a host-specific account or marker. `isBugbot` returns true when the author login contains `bugbot`, or when the body carries a bot marker: `automation_id`, `run_id`, `agentic security review`, or `bugbot`. `passKey` reads `RUN_ID:` and `AUTOMATION_ID:` with any prefix, so `CURSOR_AUTOMATION_ID:` and `QODER_AUTOMATION_ID:` both parse.

Qoder's exact bugbot login and marker are unverified. Detection covers any host whose bot login contains `bugbot` or whose comments carry these markers. A host that uses a different marker needs a new entry in the `isBugbot` body-token list and the `passKey` regex list.

## Model setup

Run `/setup-pstack` to write `~/.qoder/rules/pstack-models.md` and the model-pinning profiles under `~/.qoder/agents/`. Until you run it, roles use the documented auto fallback: `Explore` for read-only work and `poteto-agent` for write work. The fallback works. The pinned-model path stays inactive until setup runs.

## Out of scope

The benny Slack automations are not a compatibility target. Qoder has no surface that creates a Slack-triggered automation, so the pack stays dormant and unsupported. `automations/benny/FOR_AGENTS.md` records this limit.