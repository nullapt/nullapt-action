# nullapt-action

GitHub Action to sign and publish a [nullapt](https://github.com/nullapt/nullapt) skill to the registry on every version tag.

## Usage

```yaml
# .github/workflows/publish.yml
name: Publish skill
on:
  push:
    tags: ['v*']

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Build your WASM binary first (language-specific)
      - name: Build WASM (Rust)
        run: |
          rustup target add wasm32-wasip1
          cargo build --target wasm32-wasip1 --release
          cp target/wasm32-wasip1/release/*.wasm skill.wasm

      - name: Publish to nullapt registry
        uses: nullapt/nullapt-action@v1
        with:
          skill-json: SKILL.json
          wasm-file: skill.wasm
          private-key: ${{ secrets.NULLAPT_SIGNING_KEY }}
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `skill-json` | Yes | `SKILL.json` | Path to your skill manifest |
| `wasm-file` | Yes | `skill.wasm` | Path to your compiled WASM binary |
| `private-key` | Yes | — | Ed25519 private key PEM (store as a repo secret) |
| `registry` | No | `https://registry.nullapt.dev` | Custom registry URL for self-hosted instances |
| `nullapt-version` | No | `latest` | Pin a specific nullapt CLI version |

## Outputs

| Output | Description |
|---|---|
| `skill-name` | The published skill name (from SKILL.json) |
| `skill-version` | The published skill version (from SKILL.json) |

## Setup

1. Generate a signing key pair:

   ```bash
   nullapt keygen   # writes to ~/.nullapt/keys/ by default
   ```

2. Add the contents of `~/.nullapt/keys/private.pem` as a repository secret named `NULLAPT_SIGNING_KEY`.
3. The public key is embedded in your `SKILL.json` after you run `nullapt sign`.

## Self-hosted registry

```yaml
- uses: nullapt/nullapt-action@v1
  with:
    skill-json: SKILL.json
    wasm-file: skill.wasm
    private-key: ${{ secrets.NULLAPT_SIGNING_KEY }}
    registry: https://my-registry.internal
```
