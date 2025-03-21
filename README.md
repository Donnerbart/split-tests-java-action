[![Integration tests](https://github.com/Donnerbart/split-tests-java-action/actions/workflows/integration-test.yml/badge.svg)](https://github.com/Donnerbart/split-tests-java-action/actions/workflows/integration-test.yml)
[![Release](https://github.com/Donnerbart/split-tests-java-action/actions/workflows/release.yml/badge.svg)](https://github.com/Donnerbart/split-tests-java-action/actions/workflows/release.yml)

# @donnerbart/split-tests-java-action

Divides a test suite into groups with equal execution time, based on prior test timings.

This ensures optimal parallel execution. Since test file runtimes can vary significantly, splitting them evenly without
considering timing may result in inefficient grouping.

## Usage

```yaml
name: Integration Tests
on:
  pull_request:
    branches: '**'
  workflow_dispatch:
concurrency:
  group: integration-test-${{ github.ref }}
  cancel-in-progress: true
env:
  split-total: 10
jobs:
  generate-split-index-json:
    name: Generate split indexes
    runs-on: ubuntu-latest
    outputs:
      json: ${{ steps.generate.outputs.split-index-json }}
    steps:
      - name: Generate split index list 
        id: generate
        uses: donnerbart/split-tests-java-action/generate-split-index-json@v1
        with:
          split-total: ${{ env.split-total }}

  integration-test:
    name: 'Split #${{ matrix.split-index }}'
    runs-on: ubuntu-latest
    needs:
      - generate-split-index-json
    strategy:
      fail-fast: false
      matrix:
        split-index: ${{ fromjson(needs.generate-split-index-json.outputs.json) }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          path: project

      - name: Checkout JUnit reports
        uses: donnerbart/split-tests-java-action/checkout-junit-reports@v1
        with:
          git-branch: junit-reports/${{ github.base_ref }}
          path: junit-reports

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Split tests
        id: split-tests
        uses: donnerbart/split-tests-java-action@v1
        with:
          split-index: ${{ matrix.split-index }}
          split-total: ${{ env.split-total }}
          glob: '**/project/src/integrationTest/**/*IT.java'
          junit-glob: '**/junit-reports/*.xml'
          format: 'gradle'
          new-test-time: 'average'
          debug: true

      - name: Run integration tests
        run: ./gradlew :integrationTest ${{ steps.split-tests.outputs.test-suite }}

      - name: Upload JUnit report artifact
        uses: actions/upload-artifact@v4
        with:
          name: junit-xml-reports-${{ matrix.split-index }}
          path: '**/test-results/integrationTest/*.xml'

  merge-junit-reports:
    name: Merge JUnit reports
    runs-on: ubuntu-latest
    needs:
      - integration-test
    permissions:
      contents: write
    steps:
      - name: Merge JUnit reports
        uses: donnerbart/split-tests-java-action/merge-junit-reports@v1
        with:
          git-branch: junit-reports/${{ github.base_ref }}
          artifact-name: junit-xml-reports
          split-artifact-pattern: junit-xml-reports-*
```
