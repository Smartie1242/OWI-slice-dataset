# OWI Slice Dataset

This repository contains annotated Open Web Index (OWI) web slices used for
language-identification evaluation in the Resiliparse research project.

## Status And License

Research dataset release candidate. Reuse is governed by the terms described in `LICENSE.md`, including the Open Web Index Licence (OWIL) context for OWI-derived text. Until redistribution terms are confirmed for public release, treat this repository as research-use material rather than a generic permissive open-data release.

## Dataset Contents

* `data/frisian/`: 100 OWI pages sampled from the OWI Frisian language path.
* `data/dutch/`: 100 OWI pages sampled from the OWI Dutch language path.
* `data/random/`: 100 OWI pages sampled from the OWI crawl without a language
path restriction.
* `archive/combined\_annotation/timo.json`: old combined Frisian/random Timo
export. This is not a Dutch annotation export.
* `scripts/reservoir.awk`: deterministic reservoir sampler used to create the
raw slices.

Each active slice can contain:

* `raw.jsonl`: sampled OWI records.
* `cleaned.json`: cleaned documents for annotation.
* `labelstudio.json`: Label Studio import tasks.
* `annotations/marten.json`: Marten's Label Studio export.
* `annotations/timo.json`: Timo's export, present for Frisian and random only.
* `diff.json`: annotation disagreements, present for multi-annotator slices.
* `corrected.json`: manually corrected final annotations.
* `enriched.json`: corrected annotations enriched with source text metadata.

The Dutch slice is Marten-only in this release. Do not infer or fabricate a
Dutch `annotations/timo.json` from the combined export.

## Rebuilding The Dataset

The original OWI data was retrieved with `owilix` from the `owi-cli` project.
Install that tool separately when fresh OWI slices are needed:

```bash
git clone https://opencode.it4i.eu/openwebsearcheu-public/owi-cli.git
cd owi-cli
uv sync
uv run owilix --help
```

Frisian:

```bash
mkdir -p data/frisian
owilix --format jsonl query less -L all --files "\*\*/language=fry/\*\*/\*.parquet" \\
| awk -v k=100 -v seed=42 -f scripts/reservoir.awk \\
> data/frisian/raw.jsonl
```

Dutch:

```bash
mkdir -p data/dutch
owilix --format jsonl query less -L all --files "\*\*/language=nld/\*\*/\*.parquet" \\
| awk -v k=100 -v seed=42 -f scripts/reservoir.awk \\
> data/dutch/raw.jsonl
```

Random:

```bash
mkdir -p data/random
owilix --format jsonl query less -L all \\
| awk -v k=100 -v seed=42 -f scripts/reservoir.awk \\
> data/random/raw.jsonl
```

## Annotation Workflow

Run these commands from the Resiliparse research repository's `code/` directory.

Prepare Label Studio files:

```bash
python -m rsp.cli.prepare\_datasets \\
  --input data/OWI\_slice/frisian/raw.jsonl \\
  --input data/OWI\_slice/dutch/raw.jsonl \\
  --input data/OWI\_slice/random/raw.jsonl \\
  --output data/OWI\_slice
```

Compare annotator exports for multi-annotator slices:

```bash
python -m rsp.cli.compare\_annotations data/OWI\_slice/random/annotations/annotator1.json data/OWI\_slice/random/annotations/annotator2.json --out data/OWI\_slice/random/diff.json
python -m rsp.cli.compare\_annotations data/OWI\_slice/frisian/annotations/annotator1.json data/OWI\_slice/frisian/annotations/annotator2.json --out data/OWI\_slice/frisian/diff.json
```

After manual correction, enrich corrected annotations:

```bash
python -m rsp.cli.owi\_preprocessing --slice-dir data/OWI\_slice
```

Final evaluation inputs are the `enriched.json` files in each slice folder.

