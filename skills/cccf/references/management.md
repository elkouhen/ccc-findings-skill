# cccf Management

`cccf` depends on three executables for the full workflow:

- `cccf` — findings / endpoints / graph / MCP
- `semgrep` — scan engine used by `cccf index`
- `ccc` — semantic code search used by `cccf search`

## Installation

Install all three when the environment is missing them:

```bash
uv tool install ccc-findings
uv tool install cocoindex-code
pipx install semgrep
```

`cccf search` is the only command that requires `ccc`; `cccf summary`,
`cccf endpoints`, `cccf graph`, `cccf findings`, and `cccf index` do not.

## Project Initialization

For the Java/Spring/Maven audit workflow owned by this skill:

1. Copy the local rule packs into the target repo under `.cccf/rules/`.
2. Run `cccf init` with explicit `--rules` for `default`, `liveness`, `rest`,
   and `kafka`.
3. Run `cccf index`.

Example:

```bash
cccf init \
  --rules .cccf/rules/default/a-memoire-fichiers.yaml \
  --rules .cccf/rules/default/b-kafka.yaml \
  --rules .cccf/rules/liveness/java.yaml \
  --rules .cccf/rules/rest/java.yaml \
  --rules .cccf/rules/kafka/java.yaml
cccf index
```

If `.cccf/config.yml` already exists, do not recreate it silently; reuse it.

## Refreshing the Index

- After code changes: run `cccf index`.
- After asking architecture-level questions: prefer `cccf summary`,
  `cccf endpoints`, `cccf graph`, or `cccf findings` before `cccf search`.
- After major refactors that affect code search too: refresh `cccf index`, then
  use `cccf search`.

## Troubleshooting

- `cccf` missing: install `ccc-findings`.
- `semgrep` missing: install `semgrep`.
- `ccc` missing: install `cocoindex-code`; only blocks `cccf search`.
- `Index absent. Lancez d'abord: cccf index`: initialize/configure the repo,
  then run `cccf index`.
- Embedding signature / dimension mismatch: re-run `cccf index --full`.
