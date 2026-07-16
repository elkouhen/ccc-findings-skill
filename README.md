# ccc-radar-skill

Coding-agent skill for `cccr`: architecture exploration, Semgrep findings,
index management, and optional semantic code search, driven automatically by
the agent.

- [`skills/cccr/SKILL.md`](skills/cccr/SKILL.md) — ownership rules and the
  architecture-first workflow.
- [`skills/cccr/references/settings.md`](skills/cccr/references/settings.md) —
  include/exclude patterns and optional `ccc` settings.
- [`skills/cccr/references/management.md`](skills/cccr/references/management.md) —
  installation, initialization, MCP setup, index refresh, and troubleshooting.
- [`skills/cccr/rules/default/`](skills/cccr/rules/default/) — bundled
  Semgrep rule pack (Java) for bounded file streaming and Kafka
  claim-check/delivery guarantees, run by default on `cccr init` (see
  **Default Rules** in `SKILL.md`); `design-rules.md` in the same directory
  documents the full rule set in prose (including the rules with no
  Semgrep-detectable pattern).
- [`skills/cccr/rules/liveness/`](skills/cccr/rules/liveness/) — bundled
  Semgrep rule pack (Java) for distributed-system blocking points: missing
  HTTP timeouts, blocking waits, synchronous REST calls inside Kafka
  consumers, network calls under a lock — also run by default on
  `cccr init` (see **Default Rules** in `SKILL.md`).
- [`skills/cccr/rules/rest/`](skills/cccr/rules/rest/) — bundled Semgrep
  rule pack (Java) that inventories REST integrations for `cccr
  integrations`/`cccr graph`: Spring routes exposed (`@GetMapping`/.../
  `@RequestMapping` for any HTTP verb), `RestTemplate` call sites, Feign
  declarative clients (`@FeignClient` interface methods, distinguished from
  server routes by their missing method body), and `WebClient` fluent calls
  (`.get().uri(...)`, ...) — not a findings pack, run by default on `cccr
  init` (see **Default Rules** in `SKILL.md`).
- [`skills/cccr/rules/kafka/`](skills/cccr/rules/kafka/) — bundled Semgrep
  rule pack (Java/Spring, plus raw `kafka-clients`) that inventories Kafka
  producers/consumers (`@KafkaListener`, `KafkaTemplate.send`,
  `ProducerRecord`, `KafkaConsumer.subscribe(...)` for non-Spring
  consumers, and `StreamsBuilder.stream(...)`/`KStream.to(...)` for Kafka
  Streams) for `cccr integrations`/`cccr graph`, resolving topic names given
  as Spring properties (`${app.kafka.topics.orders}`, including via a
  `@Value`-annotated field referenced by variable) against
  `application.yml`/`.properties` when present — not a findings pack, run
  by default on `cccr init` (see **Default Rules** in `SKILL.md`).
- [`skills/cccr/rules/kafka-security/`](skills/cccr/rules/kafka-security/)
  — bundled Semgrep rule pack (Java/Spring) for Kafka security findings:
  hardcoded SASL credentials, `PLAINTEXT` security protocol, a
  `JsonDeserializer` trusting all packages, raw Java deserialization — run
  by default on `cccr init` (see **Default Rules** in `SKILL.md`).

## Installation

```bash
npx skills add elkouhen/ccc-radar-skill
```

This installs the `cccr` skill (`skills/cccr/`) for your coding agent. It
still requires the `cccr` CLI itself — see
[Installation in `ccc-radar`](https://github.com/elkouhen/ccc-radar#installation).

For a Java/Spring architecture audit, installation is not complete until the
CLI can locate this skill's rule packs. Set the path explicitly, then use the
preflight before indexing:

```bash
export CCCR_RULES_ROOT="/path/to/ccc-radar-skill/skills/cccr/rules"
cccr init
cccr doctor
cccr index
cccr analyze audit
```

`cccr doctor` distinguishes blocking audit prerequisites (Semgrep,
configuration and REST/Kafka packs), the local-model readiness warning, and
optional `ccc` code search.
The full installation and troubleshooting sequence is in
[`management.md`](skills/cccr/references/management.md).

## MCP Server

The `cccr` MCP server exposes the indexed architecture and findings to a coding
agent. Initialize and index the target repository first, then start the agent
from that repository: the server uses its working directory to locate
`.cccr/config.yml` and `.cccr/findings.db`.

For Codex, register the stdio server once:

```bash
codex mcp add cccr -- cccr mcp
codex mcp get cccr
```

Restart Codex after registration. For Claude Code, add the following server
configuration and restart the client:

```json
{"mcpServers": {"cccr": {"command": "cccr", "args": ["mcp"]}}}
```

Pi requires the community `pi-mcp-adapter` extension because MCP is not built
in. Install it once, then create `.mcp.json` in the indexed repository:

```bash
pi install npm:pi-mcp-adapter
```

```json
{"mcpServers": {"cccr": {"command": "cccr", "args": ["mcp"]}}}
```

Start Pi from that repository and use `/mcp` to inspect the connection.

## Related projects

- [`ccc-radar`](https://github.com/elkouhen/ccc-radar) (`cccr`) — the
  CLI and MCP server this skill drives. It indexes Semgrep findings locally
  and joins them to code search results from `ccc`.
- [`cocoindex-code`](https://github.com/cocoindex-io/cocoindex-code) (`ccc`)
  — the underlying AST-based semantic code search tool that `cccr` extends
  as a companion package (no fork, no internal import at the CLI/MCP level —
  see `ccc-radar`'s ADR-1).

## Provenance

This skill started as an adaptation of cocoindex-code's own
[`skills/ccc/`](https://github.com/cocoindex-io/cocoindex-code/tree/main/skills/ccc)
skill (Apache-2.0): `SKILL.md` is renamed to `cccr` and extended to cover
Semgrep findings, while `references/settings.md` and
`references/management.md` stay close to the upstream material but are adapted
where `cccr` adds its own workflow and defaults. Each file links back to its
source where relevant. See [LICENSE](LICENSE) for the terms this carries over.

## License

[Apache License 2.0](LICENSE), matching the upstream
[`cocoindex-code`](https://github.com/cocoindex-io/cocoindex-code) project.
