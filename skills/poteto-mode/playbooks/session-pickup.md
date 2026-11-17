### Session pickup

**You own the resume point. Read the prior trail, don't redo it.**

1. Locate the prior trail. A local transcript under `~/.qoder/projects/<slug>/`, where `<slug>` is the workspace path with every UTF-16 code unit outside ASCII letters and digits replaced by `-` (so `/home/me/my_proj` becomes `-home-me-my-proj`). When that dashed form is longer than 200 characters, keep the first 200, then `-`, then the base-36 form of `Math.abs(t)`, where `t` starts at `5381` and each UTF-16 code unit of the original path updates it with `t = 33 * t ^ charCodeAt(i)` in signed 32-bit (`Math.abs(-2147483648)` is `2147483648`). Do not glob across `~/.qoder/projects/*/`. That crosses workspace boundaries and reads private chats from unrelated projects. A pushed branch is the other trail. Read the metadata overview and last messages first, then scan back for the decision points. Parse a long transcript in a subagent and keep the reduced timeline in the main thread (the **principle-guard-the-context-window** skill). Pick each subagent's reasoning effort for the job, per the Subagents section.
2. Reconstruct operational state. The branch and worktree, what already landed (`git log`, `git diff` against the base), the open todos, the decisions made. The prior trail is authoritative input. Resist the bias to re-derive it.
3. Diff done vs pending. Compare what shipped against what was planned, name the resume point, do not re-run the prior repro or redo completed work. A "let me verify from scratch" pass means you're treating the trail as untrustworthy when it's authoritative.
4. Route the remaining work to the matching playbook and pick the verdict: continue the execution, ship a finished recommendation, ratify or override a prior conclusion, or postmortem a failed run. The pickup playbook ends here. The routed playbook owns the rest.
5. Verify the inherited claims against the original goal on the real artifact (the **principle-prove-it-works** skill). A passing prior self-report is not the proof.

**Reply:** where the prior agent stopped, what you inherited vs redid (ideally nothing redone), the resume point, and the outcome.
