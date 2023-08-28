<!-- 2020-10-02 github-actions,github,netlify,hosting,deployment,cron -->

# Scheduling Netlify site deployment using GitHub Actions

TODO

1. Create a new GitHub Actions workflow file, for example **.github/workflows/deploy.yml**:

```yaml
name: Deploy
on:
  schedule:
    # At 5am every morning
    # https://crontab.guru/#0_5_*_*_*
    - cron: '0 5 * * *'
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Netlify deployment
        run: curl -s -X POST "https://api.netlify.com/build_hooks/${TOKEN}"
        env:
          TOKEN: ${{ secrets.NETLIFY_BUILD_HOOK }}
```

Here, we have:

- A job that triggers a Netlify deployment using a webhook. We don’t have to deploy the same repository where the workflow is located — a webhook from any Netlify site would work here.
-
- The job runs at 5 AM every morning.

**Hint:** [Crontab guru](https://crontab.guru/) is a great tool to create or validate cron schedule expressions.

2. sectet
