# @donnerbart/split-tests-java-action/checkout-junit-reports

Checks out the JUnit reports from a Git branch and stores the SHA values in a GitHub artifact.
On a job re-run this SHA will be used to check out the same test results to ensure a consistent test distribution.

## Required permissions

This action queries the workflow run's artifacts via the GitHub REST API, so the job's `GITHUB_TOKEN` must have at least `actions: read`. Modern repositories grant this by default, but workflows that explicitly restrict permissions need to include it:

```yaml
permissions:
  actions: read
  contents: read
```
