
## Quick start

```bash
pip install -r requirements.txt
export PIPELINE_HASH_SALT=local-test-salt   # PowerShell: $env:PIPELINE_HASH_SALT = "..."
python -m pytest -q
python run_pipeline.py --config config/sales.yaml --dry-run
python run_pipeline.py --config config/sales.yaml
python run_pipeline.py --config config/customers.yaml
```

Full setup, commands, and troubleshooting: [docs/HOW_TO_USE.md](docs/HOW_TO_USE.md)

Adding your own dataset: [docs/ADDING_A_DATASET.md](docs/ADDING_A_DATASET.md)

## Design highlights

- **Config-driven:** new datasets need a YAML file, not new code. Config is validated, and unknown keys are rejected.
- **Named data quality rules** (`not_null`, `min`, `max`, `allowed`, `regex`, `unique`) instead of arbitrary expressions.
- **Quarantine and thresholds:** bad rows are kept for review, and the run stops if the reject rate is too high.
- **Schema contracts:** added, removed, or re-typed columns fail the run until reviewed.
- **Privacy:** configured PII columns are salted-SHA-256 hashed before any file is written.
- **Idempotent runs:** running twice produces the same output.

Reasoning for these choices is in [docs/adr/](docs/adr/).

## Known limitations

- Incremental loading does not remove rows that were deleted from a source. Use `--full-refresh` to rebuild.
- Data is processed in memory, so very large files would need chunked processing.
- Scheduling is handled by GitHub Actions. There is no Airflow or Databricks Workflows version yet.
- No alerting yet. The run report and `quality.json` are the places to add it.
- Hashed PII is pseudonymous. Anyone with the salt can test guesses, so keep the salt secret.

## Testing

```bash
python -m pytest -q        # unit and end-to-end tests
ruff check .               # lint
```

Tests write only to a temporary folder, so they never change your data or contracts.

## License
