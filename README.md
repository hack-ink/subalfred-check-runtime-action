<div align="center">

# [Subalfred check runtime action](https://github.com/hack-ink/subalfred-check-runtime-action)
### Use [`subalfred check runtime`](https://github.com/hack-ink/subalfred) to check your runtime code.

<img width="400" src="https://raw.githubusercontent.com/w3f/Grants-Program/master/static/img/badge_black.svg"/>

</div>

### Usage
> **Take [Polkadot code/repository](https://github.com/paritytech/polkadot) as an example.**

**Requirement.**
1. build your node
2. upload it
3. provide a live chain HTTP RPC endpoint to compare with

**Build and upload.**
```yml
jobs:
  basic-checks:
    name: Build and upload Polkadot
    runs-on: ubuntu-latest
    steps:
      - name: Setup build environment
        run: sudo apt install -y protobuf-compiler
      - name: Fetch latest code
        uses: actions/checkout@v3
      - name: Build
        run: |
          cargo build --release --locked
          mv target/release/polkadot .
      - name: Upload
        uses: actions/upload-artifact@v2
        with:
          # pass this name to `uploaded-artifact`
          name: polkadot
          path: polkadot
```

**Check single runtime.**
```yml
name: Checks
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  runtime-checks:
    name: Task check runtime
    runs-on: ubuntu-latest
    steps:
      - name: Check
        uses: actions/subalfred-check-runtime-action@v0.1.8
        with:
          uploaded-artifact: polkadot
          chain: polkadot
          compare-with: https://rpc.polkadot.io
```

**Check multiple runtimes at once.**
```yml
name: Checks
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  runtime-checks:
    name: Task check runtimes
    strategy:
      matrix:
        target: [
            { chain: polkadot-dev, compare-with: https://rpc.polkadot.io },
            { chain: kusama-dev, compare-with: https://kusama-rpc.polkadot.io },
          ]
    needs: [basic-checks]
    runs-on: ubuntu-latest
    steps:
      - name: Check ${{ matrix.target.chain }}
        uses: hack-ink/subalfred-check-runtime-action@v0.1.8
        with:
          uploaded-artifact: polkadot
          chain: ${{ matrix.target.chain }}
          compare-with: ${{ matrix.target.compare-with }}
```
