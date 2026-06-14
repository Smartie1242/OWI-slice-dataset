# Dataset Card: OWI Slice Dataset

## Purpose

Annotated OWI web slices for evaluating language identification on noisy,
heterogeneous web text, with specific attention to Frisian and Dutch.

## Source And Sampling

Open Web Index records were retrieved from the 2026-02-03 main public dataset
with `owilix`. Frisian and Dutch use the `fry` and `nld` language paths; the
random slice has no language-path restriction. Each slice was sampled to 100
records with deterministic reservoir sampling using seed 42.

## Annotation

The repository uses anonymized annotator filenames. `annotator1` is present for
all slices; `annotator2` is present for Frisian and random only. Agreement is
96/100 for Frisian and 98/100 for random. Dutch has no second annotation.
Disagreements were manually resolved into `corrected.json`.

## Recommended Evaluation File

Use `data/<slice>/enriched.json`. Mixed-language labels are retained for
provenance but excluded from the single-label detector evaluation used by the
research project.

## License And Release Status

Research dataset release candidate. See `LICENSE.md` for Open Web Index licence
context and reuse notes. Redistribution terms must be confirmed before public
release.
