# Codex Code Review Github Action

A GitHub Action that runs [OpenAI Codex](https://github.com/openai/codex-action) as an automated PR code reviewer. Codex reads the PR diff, applies a review-guidelines prompt, and posts its findings back as a PR comment.

## Setup

1. Copy these two files into your repo (preserving paths):
   - `.github/workflows/codex-review.yml`
   - `.github/codex/review-prompt.md`
2. Add an `OPENAI_API_KEY` secret under **Settings → Secrets and variables → Actions**.
3. Open a PR. The workflow runs and a single review comment appears once Codex finishes.

## How it works

- **Trigger:** every PR push (`opened`, `synchronize`, `reopened`).
- **Codex run:** `openai/codex-action@v1` checks out the PR's merge commit, pre-fetches the base ref so the prompt's `git merge-base` works, and runs `codex exec` with `prompt-file: .github/codex/review-prompt.md`.
- **Comment:** a second job posts Codex's final message to the PR via `actions/github-script@v7`.

## Fork PRs

GitHub does not pass repository secrets to workflows triggered by pull requests from forks (only a read-only `GITHUB_TOKEN`). That means `OPENAI_API_KEY` will be empty on fork PRs and the Codex step will fail. This workflow is therefore intended for same-repo PRs or PRs from trusted contributors with branch access. Don't switch the trigger to `pull_request_target` to "fix" this — combining `pull_request_target` with checking out PR code is a well-known security foot-gun.

## Customizing the review

Edit `.github/codex/review-prompt.md`. The prompt is a lightly adapted copy of the [`review-code` skill](https://github.com/zcaceres/skills/tree/main/skills/review-code) — change the guidelines, output format, or framing as you like. Anything Codex writes as its final message becomes the PR comment verbatim, so keep the output format aligned with what reads well on GitHub.

To change the model, add `model: gpt-5-codex` (or similar) under `with:` in the workflow. See the [action's input list](https://github.com/openai/codex-action#inputs) for other knobs (`sandbox`, `effort`, `allow-users`, etc.).

## License

MIT
