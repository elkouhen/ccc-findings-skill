# cccf Settings

Two configuration layers matter in practice:

## 1. Project audit config: `.cccf/config.yml`

This is the file created by `cccf init` and consumed by `cccf index`.

```yaml
rules:
  - .cccf/rules/default/a-memoire-fichiers.yaml
  - .cccf/rules/default/b-kafka.yaml
  - .cccf/rules/liveness/java.yaml
  - .cccf/rules/rest/java.yaml
  - .cccf/rules/kafka/java.yaml
include:
  - "**/*"
exclude:
  - ".git/**"
  - ".venv/**"
  - "node_modules/**"
  - ".cccf/**"
min_severity: INFO
embedding_model: Snowflake/snowflake-arctic-embed-xs
semgrep_timeout_s: 120
```

Relevant fields for this skill:

- `rules`: must include the copied local packs for the microservice audit
  workflow if you want `cccf endpoints` / `cccf graph` to work.
- `include` / `exclude`: file perimeter for indexing.
- `min_severity`: applies to findings, not to endpoint-inventory rules.
- `embedding_model`: used by `cccf findings` and the local findings index.

After editing `.cccf/config.yml`, run `cccf index`.

## 2. ccc user-level config

`cccf search` relies on `ccc`, which keeps its own settings outside the
repository (for example `~/.cocoindex_code/global_settings.yml` for embedding
provider/model). Those settings matter only for the semantic code-search part.

If the question is about architecture inventory, Kafka/REST coupling, or known
findings, you can often avoid touching `ccc` settings entirely by using:

1. `cccf summary`
2. `cccf endpoints`
3. `cccf graph`
4. `cccf findings`

Use `cccf search` only when you truly need semantic code retrieval.
