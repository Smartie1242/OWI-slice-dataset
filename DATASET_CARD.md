# Dataset Card: OWI Slice Dataset

## Purpose

Annotated OWI web slices for evaluating language identification on noisy,
heterogeneous web text with specific attention to Frisian, Dutch, and random web
content.

## Source

Open Web Index crawl records retrieved with `owilix` and sampled with a
deterministic reservoir sampler using seed `42`.

## Splits And Slices

The dataset has three slices: Frisian, Dutch, and random. Each slice contains
100 sampled pages before correction/enrichment.

## Annotation

Marten annotations are present for all slices. Timo annotations are present for
Frisian and random only. The Dutch slice has no second annotator export.

## Recommended Evaluation File

Use `data/<slice>/enriched.json` for final evaluation and retain the raw,
cleaned, Label Studio, and annotation files for reproducibility.

## License And Release Status

Research dataset release candidate. OWI redistribution terms must be confirmed
before public release.
