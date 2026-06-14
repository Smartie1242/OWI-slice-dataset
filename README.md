# OWI Slice Dataset

Annotated Open Web Index (OWI) web slices used for language-identification
evaluation in the [Resiliparse research project](https://github.com/Smartie1242/resiliparse-research).

## Status And License

This is a research dataset release candidate. Reuse is governed by `LICENSE.md`,
including the Open Web Index Licence context for OWI-derived text. Until public
redistribution terms are confirmed, treat the repository as research-use
material rather than a generic permissive open-data release.

## Contents

Each active slice contains 100 sampled OWI records:

- `data/frisian/`: records from the OWI Frisian (`fry`) language path.
- `data/dutch/`: records from the OWI Dutch (`nld`) language path.
- `data/random/`: records sampled without a language-path restriction.

Files progress through the annotation workflow as follows:

```text
raw.jsonl
cleaned.json
labelstudio.json
annotations/annotator1.json
annotations/annotator2.json  # Frisian and random only
diff.json                    # Frisian and random only
corrected.json
enriched.json
```

The public repository anonymizes the research filenames: `annotator1.json`
corresponds to the primary annotator and `annotator2.json` to the second
annotator. Dutch has only `annotator1.json`; do not infer a Dutch second
annotation from `archive/combined_annotation/annotator2.json`. That archived
file is the historical combined Frisian/random export.

All active dataset files were verified byte-for-byte against the research copy,
accounting for this annotation filename mapping. Final evaluation uses
`data/<slice>/enriched.json`.

## Retrieve New Slices

The research data was retrieved with `owilix` from
[`owi-cli`](https://opencode.it4i.eu/openwebsearcheu-public/owi-cli):

```bash
git clone https://opencode.it4i.eu/openwebsearcheu-public/owi-cli.git
cd owi-cli
uv sync

OWI_SELECTOR="all:2026-02-03/collectionName=main/access=public"
uv run owilix remote pull "$OWI_SELECTOR"

# Use the local identifier reported by the pull. The research run used:
OWI_DATASET="Index-main.owi@it41-2026-02-03:2026-02-03"
```

From this repository, sample with seed 42:

```bash
mkdir -p data/frisian data/dutch data/random

owilix --format jsonl query less -L "$OWI_DATASET" --files "**/language=fry/**/*.parquet" \
| awk -v k=100 -v seed=42 -f scripts/reservoir.awk \
> data/frisian/raw.jsonl

owilix --format jsonl query less -L "$OWI_DATASET" --files "**/language=nld/**/*.parquet" \
| awk -v k=100 -v seed=42 -f scripts/reservoir.awk \
> data/dutch/raw.jsonl

owilix --format jsonl query less -L "$OWI_DATASET" \
| awk -v k=100 -v seed=42 -f scripts/reservoir.awk \
> data/random/raw.jsonl
```

## Annotation Workflow

Install [`rsp_eval`](https://github.com/Smartie1242/rsp_eval), then run its
commands from a working directory containing the canonical `data/OWI_slice`
layout. Copy or link this repository's `data/` directory as `data/OWI_slice/`.

Prepare Label Studio imports:

```bash
python -m rsp.cli.prepare_datasets \
  --input data/OWI_slice/frisian/raw.jsonl \
  --input data/OWI_slice/dutch/raw.jsonl \
  --input data/OWI_slice/random/raw.jsonl \
  --output data/OWI_slice
```

Use `label-studio-config.xml` from `rsp_eval` or the research repository.
Export the first annotation for every slice and the second annotation for
Frisian and random. Compare double annotations:

```bash
python -m rsp.cli.compare_annotations \
  data/OWI_slice/frisian/annotations/annotator1.json \
  data/OWI_slice/frisian/annotations/annotator2.json \
  --out data/OWI_slice/frisian/diff.json

python -m rsp.cli.compare_annotations \
  data/OWI_slice/random/annotations/annotator1.json \
  data/OWI_slice/random/annotations/annotator2.json \
  --out data/OWI_slice/random/diff.json
```

The verified agreement is 96/100 for Frisian and 98/100 for random. Dutch has
no double-annotation agreement rate. Resolve disagreements manually, save each
slice's `corrected.json`, then enrich:

```bash
python -m rsp.cli.owi_preprocessing --slice-dir data/OWI_slice
```

Annotations labelled `Mixed languages` remain in the provenance files but are
excluded from the research's single-label detector evaluation. Unknown or
otherwise unevaluable labels are also skipped defensively by analysis commands.

## Release Checklist

- Confirm OWI redistribution terms before public release.
- Preserve the raw, annotation, correction, and enrichment stages.
- Record a release tag or commit and checksums when the dataset is frozen.
- Do not include detector outputs, thesis drafts, or personal working notes.
