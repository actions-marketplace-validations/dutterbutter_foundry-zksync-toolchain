## `foundry-zksync-toolchain` Action

🚀 **This repository is a fork of the `foundry-toolchain` action, adapted to support `foundry-zksync`, which is a fork of Foundry.**  
💡 Full credit goes to the original authors for their work! 🙏

This GitHub Action installs [Foundry-ZKsync](https://github.com/matter-labs/foundry-zksync), the blazing fast, portable and modular
toolkit for ZKsync application development.

### Example workflow

```yml
on: [push]

name: test

jobs:
  check:
    name: Foundry ZKsync project
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive

      - name: Install Foundry-ZKsync
        uses: duttbutter/foundry-zksync-toolchain@v1

      - name: Run tests
        run: forge test --zksync -vvv

      - name: Run snapshot
        run: forge snapshot --zksync
```

### Inputs

- **`cache`** (Optional, Default: `true`)  
  - Whether to cache RPC responses or not.  
  - Type: `bool`

- **`version`** (Optional, Default: `nightly`)  
  - Version to install, e.g., `nightly` or `1.0.0`.  
  - **Note:** Foundry only has nightly builds for the time being.  
  - Type: `string`

- **`cache-key`** (Optional, Default: `${{ github.job }}-${{ github.sha }}`)  
  - The cache key to use for caching.  
  - Type: `string`

- **`cache-restore-keys`** (Optional, Default: `[${{ github.job }}-]`)  
  - The cache keys to use for restoring the cache.  
  - Type: `string[]`

### RPC Caching

By default, this action matches Forge's behavior and caches all RPC responses in the `~/.foundry-zksync/cache/rpc` directory.
This is done to speed up the tests and avoid hitting the rate limit of your RPC provider.

The logic of the caching is as follows:

- Always load the latest valid cache, and always create a new one with the updated cache.
- When there are no changes to the fork tests, the cache does not change but the key does, since the key is based on the
  commit hash.
- When the fork tests are changed, both the cache and the key are updated.

If you would like to disable the caching (e.g. because you want to implement your own caching mechanism), you can set
the `cache` input to `false`, like this:

```yml
- name: Install Foundry ZKsync
  uses: dutterbutter/foundry-zksync-toolchain@v1
  with:
    cache: false
```

### Custom Cache Keys

You have the ability to define custom cache keys by utilizing the `cache-key` and `cache-restore-keys` inputs. This
feature is particularly beneficial when you aim to tailor the cache-sharing strategy across multiple jobs. It is
important to ensure that the `cache-key` is unique for each execution to prevent conflicts and guarantee successful
cache saving.

For instance, if you wish to utilize a shared cache between two distinct jobs, the following configuration can be
applied:

```yml
- name: Install Foundry ZKsync
  uses: dutterbutter/foundry-zksync-toolchain@v1
  with:
    cache-key: custom-seed-test-${{ github.sha }}
    cache-restore-keys: |-
      custom-seed-test-
      custom-seed-
---
- name: Install Foundry ZKsync
  uses: dutterbutter/foundry-zksync-toolchain@v1
  with:
    cache-key: custom-seed-coverage-${{ github.sha }}
    cache-restore-keys: |-
      custom-seed-coverage-
      custom-seed-
```

#### Deleting Caches

You can delete caches via the GitHub Actions user interface. Just go to your repo's "Actions" page:

```text
https://github.com/<OWNER>/<REPO>/actions/caches
```

Then, locate the "Management" section, and click on "Caches". You will see a list of all of your current caches, which
you can delete by clicking on the trash icon.

For more detail on how to delete caches, read GitHub's docs on
[managing caches](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#managing-caches).

#### Fuzzing

Note that if you are fuzzing in your fork tests, the RPC cache strategy above will not work unless you set a
[fuzz seed](https://book.getfoundry.sh/reference/config/testing#seed). You might also want to reduce your number of RPC
calls by using [Multicall](https://github.com/mds1/multicall).

### Summaries

You can add the output of Forge and Cast commands to GitHub step summaries. The summaries support GitHub flavored
Markdown.

For example, to add the output of `forge snapshot --zksync` to a summary, you would change the snapshot step to:

```yml
- name: Run snapshot
  run: NO_COLOR=1 forge snapshot --zksync >> $GITHUB_STEP_SUMMARY
```

See the official
[GitHub docs](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary)
for more information.

### Building

When opening a PR, you must build the action exactly following the below steps for CI to pass:

```console
$ bun install
$ bun run build
```
