# cccr Management

`cccr` has two workflows: architecture exploration and audit require `cccr`,
Semgrep, and the configured local embedding model for the findings index.
Semantic code search additionally requires `ccc`.

- `cccr` — architecture objects, findings, graph exports, and MCP
- `semgrep` — scan engine used by `cccr index`
- `ccc` — semantic code search used by `cccr search`

## Installation

Install the architecture-audit prerequisites (install `uv` and `pipx` first if
they are absent):

```bash
uv tool install ccc-radar
pipx install semgrep
env -u SSL_CERT_FILE uvx --from huggingface_hub hf download jinaai/jina-code-embeddings-1.5b --local-dir ~/models/jina-code-embeddings-1.5b
```

Install optional code search with `uv tool install cocoindex-code`.

`cccr search` is the only command that requires `ccc`; architecture object
commands, `cccr graph`, `cccr analyze`, `cccr findings`, and `cccr index` do
not require it.

## Project Initialization

For the Java/Spring/Maven audit workflow owned by this skill:

1. Set `CCCR_RULES_ROOT` to this skill's `skills/cccr/rules` directory (or pass
   it with `cccr init --rules-root`).
2. Run `cccr init`, then `cccr doctor`; do not audit a graph while packs are missing.
3. Run `cccr index`, then `cccr microservices` / `cccr integrations` /
   `cccr graph` / `cccr analyze audit`.

`cccr init` copies the packs into the project and records their source hashes
in `.cccr/rules/manifest.json`; the indexed audit is therefore reproducible
even if the skill is upgraded later.

Example:

```bash
export CCCR_RULES_ROOT="/path/to/ccc-radar-skill/skills/cccr/rules"
cccr init
cccr doctor
cccr index
```

If `.cccr/config.yml` already exists, do not recreate it silently; reuse it.

## MCP Setup

The MCP server reads the project from its process working directory. Initialize
and index the project first, then launch the coding agent from that repository.

Register the server in Codex once:

```bash
codex mcp add cccr -- cccr mcp
codex mcp get cccr
```

Restart Codex after registration. For Claude Code, register this stdio server
in its MCP configuration:

```json
{"mcpServers": {"cccr": {"command": "cccr", "args": ["mcp"]}}}
```

Pi does not include MCP support by default. Install the community adapter once,
then create `.mcp.json` in the indexed repository:

```bash
pi install npm:pi-mcp-adapter
```

```json
{"mcpServers": {"cccr": {"command": "cccr", "args": ["mcp"]}}}
```

Start Pi from the target repository and use `/mcp` to inspect the connection.

## Refreshing the Index

- After code changes: run `cccr index`.
- For architecture questions: prefer `cccr microservices`, `cccr topics`,
  `cccr apis`, `cccr mongodb`, `cccr graph`, or `cccr findings` before
  `cccr search`.
- After major refactors that affect code search too: refresh `cccr index`, then
  use `cccr search`.

## Troubleshooting

- `cccr` missing: install `ccc-radar`.
- `semgrep` missing: install `semgrep`.
- `ccc` missing: install `cocoindex-code`; only blocks `cccr search`.
- missing architecture packs: set `CCCR_RULES_ROOT` before `cccr init`, or use
  explicit `--rules` in an existing configuration.
- an absent-index error: initialize/configure the repository, then run
  `cccr index`.
- Embedding signature / dimension mismatch: re-run `cccr index --full`.
