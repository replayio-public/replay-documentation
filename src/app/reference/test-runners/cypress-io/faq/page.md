---
title: Frequently asked questions
---

## How do I get debug logs?

{% table %}

- Debug focus
- Command

---

- Cypress
- `DEBUG=replay:cypress:*`

---

- Replay plugin
- `DEBUG=replay:cypress:plugin`

---

- Replay Browser
- DEBUG=cypress:launcher:browsers RECORD_REPLAY_VERBOSE=1

---

- All logs
- DEBUG=cypress:launcher:browsers RECORD_REPLAY_VERBOSE=1

{% /table %}

## How do I group tests ran in a matrix or across multiple runners into the same test run?

By default, each invocation of Cypress is grouped into a test run by a UUID generated when the run begins. To group multiple invocations of Cypress into the same run, set `RECORD_REPLAY_METADATA_TEST_RUN_ID` to the same UUID value and that will be used instead of generating a UUID for each.

Below is an example which runs three test suites using a matrix in Github Actions but groups the results into the same test run in Replay:

```yaml {% fileName=".github/workflows/e2e.yml" %}
jobs:
test-run-id:
  runs-on: ubuntu-latest
  outputs:
    testRunId: ${{ steps.testRunId.outputs.testRunId }}
  steps:
    - id: testRunId
      run: echo testRunId=$(npx uuid) >> "$GITHUB_OUTPUT"
test:
  needs: test-run-id
  runs-on: ubuntu-latest
  strategy:
    matrix:
      product: [frontend, backend, api]
  steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
      with:
        node-version: 16
    - name: Install dependencies
      run: npm ci
    - name: Install Replay Chromium
      run: npx replayio install
    - name: Run
      run: npm run test-${{ matrix.product }}
      env:
        RECORD_REPLAY_METADATA_TEST_RUN_ID: ${{ needs.test-run-id.outputs.testRunId }}
```

## What’s in the `.replay` folder?

`/.replay/runtimes` - this is where the replay browser is installed

`/` - recorded replays are saved locally in the root folder

If you’re trying to cache the browser download, cache `~/.replay/runtimes` or simply cache it before creating replays.

## How do I confirm that the browser is installed correctly

Run `ls -alR ~/.replay/runtimes` to see the contents of the runtimes folder.

## How do I troubleshoot recordings not being created?

To function properly `@replayio/cypress` listens for `after:run` events from Cypress. Cypress's `on` function doesn't support multiple listeners. If you have multiple plugins listening for this event this can prevent them from functioning properly. For that reason, we export `wrapOn` function that you can use like this in your `setupNodeEvents`:

```js
export default defineConfig({
  e2e: {
    setupNodeEvents(cyOn, config) {
      const on = wrapOn(cyOn)
      // ... rest of your plugins-related code
    },
  },
})
```

const on = wrapOn(cyOn)

## What other environment variables can I configure?

`RECORD_REPLAY_METADATA_TEST_RUN_TITLE` - manually apply a test run title, instead of being inferred from the latest commit

`RECORD_REPLAY_DIRECTORY` - set a custom directory (default: `~/.replay`)
