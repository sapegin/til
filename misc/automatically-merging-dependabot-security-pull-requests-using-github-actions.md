<!-- 2020-09-05 github github-actions dependabot mrm -->

# Automatically merging Dependabot security pull requests using GitHub Actions

[Dependabot](https://dependabot.com/) is a tool for automated dependency updates. It creates pull requets for each dependeny update, and GitHub uses it for security updates. The problem is you still need to merge these pull requests. Using [GitHub Actions](https://github.com/features/actions) we can merge pull requests from Dependabot automatically.

To do that, create a new workflow file, **.github/workflows/dependabot.yml**:

```yaml
name: 'Dependabot Automerge'

on: pull_request

jobs:
  worker:
    runs-on: ubuntu-latest

    if: github.actor == 'dependabot[bot]'
    steps:
      - uses: actions/github-script@v3
        with:
          script: |
            github.pulls.createReview({
              owner: context.payload.repository.owner.login,
              repo: context.payload.repository.name,
              pull_number: context.payload.pull_request.number,
              event: 'APPROVE'
            })
            github.pulls.merge({
              owner: context.payload.repository.owner.login,
              repo: context.payload.repository.name,
              pull_number: context.payload.pull_request.number,
              merge_method: 'squash'
            })
```

This will approve and merge all pull requests from Dependabot bot.

---

I’ve also created [an Mrm task](https://mrm.js.org/docs/mrm-task-dependabot) to add this workflow with a single command: `npx mrm dependabot`.

---

Based on the [article by Toufik Airane](https://medium.com/@toufik.airane/automerge-github-dependabot-alerts-with-github-actions-7cd6f5763750).
