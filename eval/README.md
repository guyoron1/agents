# Running evals locally

This directory contains functional tests for fullsend agents. Each
agent has its own subdirectory with an `eval.yaml` config and a `cases/`
directory of test cases.

## Prerequisites

### Tools

The following must be on `PATH`:

- **fullsend** — the fullsend CLI binary
- **openshell** — sandbox runtime
- **gh** — GitHub CLI
- **yq**, **jq** — YAML/JSON processors
- **git**, **uuidgen** — standard utilities
- **python3** with the `agent_eval` library installed:
  ```bash
  pip install -e eval/.agent-eval-harness
  ```

### Harness submodule

The eval harness scripts live in `eval/.agent-eval-harness`. Initialize
the submodule before running:

```bash
git submodule sync eval/.agent-eval-harness
git submodule update --init eval/.agent-eval-harness
```

### GCP credentials

A GCP service account with Vertex AI access is required for model
calls during both execution and scoring.

### GitHub token

`GH_TOKEN` must have `repo` and `delete_repo` scopes. The eval creates
ephemeral GitHub repos in the `EVAL_ORG` org and deletes them after
each test case. If `GH_TOKEN` is unset, `run-functional.sh` falls back
to `gh auth token`.

## Environment variables

| Variable | Required | Description |
|----------|----------|-------------|
| `EVAL_ORG` | Yes | GitHub org for ephemeral repos (e.g. `halfsend`) |
| `GOOGLE_APPLICATION_CREDENTIALS` | Yes | Path to GCP service account JSON |
| `ANTHROPIC_VERTEX_PROJECT_ID` | Yes | GCP project ID for Vertex AI |
| `GOOGLE_CLOUD_PROJECT` | Yes | GCP project ID |
| `CLOUD_ML_REGION` | Yes | Vertex AI region (e.g. `us-east5`) |
| `GH_TOKEN` | No | GitHub token; defaults to `gh auth token` |
| `FULLSEND_DIR` | No | Path to the fullsend scaffold directory; defaults to the repo root |
| `AGENT_EVAL_HARNESS_DIR` | No | Path to the agent-eval-harness checkout; defaults to `eval/.agent-eval-harness` |

## Running

```bash
GOOGLE_CLOUD_PROJECT=<project> \
ANTHROPIC_VERTEX_PROJECT_ID=<project> \
CLOUD_ML_REGION=<region> \
GOOGLE_APPLICATION_CREDENTIALS=<path> \
EVAL_ORG=<org> \
bash eval/run-functional.sh <agent>
```

Replace `<agent>` with the agent name (`review`, `triage`, etc.). The
script runs three phases:

1. **Create workspaces** — sets up case directories
2. **Execute** — creates ephemeral GitHub repos, runs the agent against
   each test case, and tears down the repos
3. **Score** — evaluates agent output using LLM judges and deterministic
   checks defined in `eval.yaml`

Results are written to `eval/runs/<agent>/<run-id>/`.

### Linting cases

Validate that all test cases have the required annotations before
running:

```bash
bash eval/lint-cases.sh <agent>
```

## Test case structure

Each case directory under `eval/<agent>/cases/` contains:

- `input.yaml` — fixture definition (forge, fixture type, title, body,
  PR files)
- `annotations.yaml` — expected outcomes (labels, review expectations,
  `max_turns`, `max_cost_usd`)
- `repo/` (optional) — base repo contents pushed to main before the
  fixture is created

## Known issues

- **Self-review 422.** The runner reuses `GH_TOKEN` as `REVIEW_TOKEN`.
  If the token owner is also the PR author, GitHub rejects
  `REQUEST_CHANGES` reviews on your own PR. Use a token from a
  different account or a GitHub App installation token.
  See [#245](https://github.com/fullsend-ai/agents/issues/245).

- **fullsend `UploadFile` bug.** In fullsend v0.31.0, `UploadFile`
  fails when the source filename matches the destination basename.
  See [fullsend-ai/fullsend#5231](https://github.com/fullsend-ai/fullsend/issues/5231).

- **`checkStatus` drops string errors.** fullsend's `checkStatus` does
  not handle string-typed error responses from the GitHub API, causing
  silent failures.
