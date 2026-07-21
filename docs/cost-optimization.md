# Cost Optimization: RTK + Ponytail

Two opt-in tools that reduce agent token spend without changing output
quality. RTK compresses input (CLI output the agent reads); Ponytail
compresses output (the agent's own reasoning and code).

## Benchmark results

30 matched pairs, same issues, same repos, same models — with and
without RTK + Ponytail enabled:

| Phase  | Baseline avg | RTK+Ponytail avg | Savings |
|--------|--------------|-------------------|---------|
| Triage | $0.44        | $0.33             | 28.7%   |
| Code   | $2.80        | $2.12             | 24.2%   |
| Review | $1.15        | $1.08             | ~6%     |
| Fix    | $0.92        | $0.87             | ~6%     |

Review and Fix agents showed marginal savings because they operate on
narrower context (a single PR diff) where RTK has less to prune.

## What each tool does

### RTK (input-side compression)

[RTK](https://github.com/rtk-ai/rtk) is a Rust CLI proxy that wraps
commands like `git`, `find`, `ls`, and `cat`. It returns the same
information in a token-optimized format — 60–90% fewer tokens on
typical dev operations.

The `plugins/rtk/rtk-rewrite.sh` hook intercepts Bash tool calls and
transparently rewrites them to use `rtk`.

**Prerequisite:** The `rtk` binary (>= 0.23.0) must be installed in the
sandbox image. See [RTK install docs](https://github.com/rtk-ai/rtk#installation).

### Ponytail (output-side compression)

The `skills/ponytail` skill injects a "lazy senior developer" persona
that produces minimal, correct output. It enforces:

- **YAGNI ladder** — stdlib before custom code, reuse before new code,
  one line before fifty
- **Output compression** — code first, at most three lines of
  explanation
- **Root-cause focus** — fix the shared function once, not every caller

No prerequisites — it's a pure prompt skill.

## Enabling

### Ponytail only (no prerequisites)

Add `ponytail` to the agent's `skills:` array:

```yaml
# agents/code.md frontmatter
skills:
  - code-implementation
  - ponytail
```

Or in the harness config:

```yaml
# harness/code.yaml
skills:
  - skills/code-implementation
  - skills/ponytail
```

### RTK + Ponytail (requires rtk binary in image)

Add the RTK plugin to the harness config:

```yaml
# harness/code.yaml
plugins:
  - plugins/gopls-lsp
  - plugins/rtk

skills:
  - skills/code-implementation
  - skills/ponytail
```

The RTK plugin's `rtk-rewrite.sh` hook must be configured as a
PreToolUse hook in the sandbox's Claude Code settings. The hook
degrades gracefully — if `rtk` is not on PATH, it logs a warning and
passes commands through unchanged.

## Trade-offs

- **Ponytail** may produce shorter commit messages and PR descriptions.
  The code-implementation skill's commit format rules take precedence
  when they conflict.
- **RTK** returns filtered CLI output. For debugging sandbox
  infrastructure issues, use `rtk proxy <cmd>` to bypass filtering and
  see raw output.
- Review and Fix agents see small savings (~6%) because their context
  window is already narrow (single PR diff). The triage and code agents
  benefit most.

## Which agents benefit most

| Agent      | RTK benefit | Ponytail benefit | Combined |
|------------|-------------|------------------|----------|
| Triage     | High        | Medium           | ~29%     |
| Code       | High        | High             | ~24%     |
| Review     | Low         | Low              | ~6%      |
| Fix        | Low         | Low              | ~6%      |
| Prioritize | Medium      | Medium           | untested |
| Retro      | Medium      | Medium           | untested |
