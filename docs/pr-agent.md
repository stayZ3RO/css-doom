# PR-Agent review pilot

PR-Agent adds an advisory AI review to small, same-repository pull requests against `main`. Reviews run when a PR is opened, reopened, or marked ready for review. To request another review, post an exact `/review` comment from an account with `OWNER` association. Draft PRs, bot PRs, fork PRs, and PRs with more than 10 changed files are skipped. The action reads the diff through GitHub's API without checking out PR code. Merge decisions stay manual.

## Enable

In **Settings > Secrets and variables > Actions**, add one repository secret named `OPENAI_KEY` with a provider API key. Add a nonsecret repository variable named `PR_AGENT_MODEL` with a supported OpenAI model ID. Use a dedicated provider project and set a spending cap there. GitHub supplies `GITHUB_TOKEN` automatically. Until both settings exist, the workflow succeeds and skips the review step.

The action sends the PR diff to the selected model provider. Use this pilot only for content that is appropriate to send there.

## Cost and disablement

Only the listed PR events and exact `/review` comments with `OWNER` association can start a review. Automatic description and improvement tools are off. The workflow has a 10-file limit, model and output token limits, a 15-minute timeout, and cancels an earlier run for the same PR. PR-Agent's reported cost is an estimate; check the provider bill for actual spend.

To stop reviews, remove `OPENAI_KEY` or `PR_AGENT_MODEL`, or disable **PR-Agent review** on the repository's Actions page.
