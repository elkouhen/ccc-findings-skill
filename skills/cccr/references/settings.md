# cccr Settings

Two configuration layers matter in practice:

## 1. Project audit config: `.cccr/config.yml`

This is the file created by `cccr init` and consumed by `cccr index`.

```yaml
rules:
  - .cccr/rules/default/a-memoire-fichiers.yaml
  - .cccr/rules/default/b-kafka.yaml
  - .cccr/rules/liveness/java.yaml
  - .cccr/rules/rest/java.yaml
  - .cccr/rules/kafka/java.yaml
  - .cccr/rules/kafka-security/java.yaml
include:
  - "**/*"
exclude:
  - ".git/**"
  - ".venv/**"
  - "node_modules/**"
  - ".cccr/**"
min_severity: INFO
embedding_model: Snowflake/snowflake-arctic-embed-xs
semgrep_timeout_s: 120
```

Relevant fields for this skill:

- `rules`: must include the copied REST and Kafka local packs if you want the
  indexed architecture objects, `cccr graph`, and exports to be complete.
- `include` / `exclude`: file perimeter for indexing.
- `min_severity`: applies to findings, not to endpoint-inventory rules.
- `embedding_model`: used by the local findings index and by MCP finding
  queries. It must be available before indexing.

After editing `.cccr/config.yml`, run `cccr index`.

## 2. ccc user-level config

`cccr search` relies on `ccc`, which keeps its own settings outside the
repository (for example `~/.cocoindex_code/global_settings.yml` for embedding
provider/model). Those settings matter only for the semantic code-search part.

If the question is about architecture inventory, Kafka/HTTP coupling, MongoDB
usage, or known findings, you can often avoid touching `ccc` settings entirely
by using:

1. `cccr microservices`
2. `cccr topics` / `cccr apis` / `cccr mongodb`
3. `cccr graph` or `cccr analyze audit`
4. `cccr findings`

Use `cccr search` only when you truly need semantic code retrieval.
