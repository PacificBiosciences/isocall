# isocall prep-isoforms

The `prep-isoforms` command converts a reference annotation into the compact
known-isoform database consumed by `isocall call`.

## Basic usage

```bash
isocall prep-isoforms \
    --gtf hg38.ncbiRefSeq.gtf.gz \
    --output ref_seq.isoforms.gz
```

## Required arguments

| Argument | Description |
|----------|-------------|
| `--gtf` | Input GTF annotation file |
| `--output` | Output known-isoforms file (must end with `.gz`; written as BGZF) |

## Input requirements

- The input annotation should be a GTF with exon features.
- Each exon record must include `gene_id` and `transcript_id`.
- `gene_name` is optional. If it is absent, `NA` is written.
- The HTSlib-backed reader accepts plain-text, gzip-compressed, and BGZF-compressed input. `.gtf.gz` remains the most common choice.

## What the command does

1. Reads exon records from the input annotation.
2. Groups exons by transcript.
3. Sorts exons within each transcript.
4. Verifies that transcripts sharing a `gene_id` are all on the same strand.
5. Writes one normalized transcript record per line.

## Output format

The output is a BGZF-compressed, gzip-compatible tab-delimited file with one transcript per line:

```text
chrom  strand  gene_name  gene_id  transcript_id  exon1_start-exon1_end,exon2_start-exon2_end,...
```

This file is intended for use with `isocall call` via `--known-isoforms`.

## Next step

Use the prepared database together with a merged profile:

```bash
isocall call \
    --merged-profile merged.gz \
    --known-isoforms ref_seq.isoforms.gz \
    --reference hg38.fa \
    --output-prefix results
```
