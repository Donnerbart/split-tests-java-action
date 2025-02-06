# @donnerbart/split-tests-java-action/generate-split-index-json

Generates the list of split index for the test matrix.

## Usage

```yaml
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
          split-total: 10

  integration-test:
    name: "Test #${{ matrix.split-index }}"
    runs-on: ubuntu-latest
    needs:
      - generate-split-index-json
    strategy:
      fail-fast: false
      matrix:
        split-index: ${{ fromjson(needs.generate-split-index-json.outputs.json) }}
```
