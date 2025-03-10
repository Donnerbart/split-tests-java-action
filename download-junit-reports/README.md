# @donnerbart/split-tests-java-action/download-junit-reports

Downloads the JUnit reports from a Git branch and stores the SHA values in a GitHub artifact.
On a job re-run this SHA will be used to check out the same test results to ensure a consistent test distribution.
