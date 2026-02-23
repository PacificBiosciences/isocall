# isocall profile

The `profile` command generates a transcription profile from aligned IsoSeq HiFi
reads. A profile consists of exon structures extracted from read alignments.

## Basic usage

```bash
isocall profile --reads sample.bam --output sample.gz
```

## Required arguments

| Argument   | Description                                     |
|------------|-------------------------------------------------|
| `--reads`  | Path to IsoSeq BAM file with aligned HiFi reads |
| `--output` | Output file path (must end with `.gz`)          |

## Optional arguments

| Argument           | Default          | Description                       |
|--------------------|------------------|-----------------------------------|
| `--sample`         | extract from BAM | Sample identifier for the profile |
| `--use-all-chroms` | off              | Profile non-core chromosomes      |
| `--skip-mito`      | off              | Skip mitochondrial reads (chrM)   |

## Filtering steps

The profile command applies several filters to ensure only desired alignments
are processed.

- Only primary alignments are used. The following alignment types are skipped:
  - Unmapped reads
  - Supplementary alignments
  - Secondary alignments
- Reads must have at least 95% of their sequence aligned to the reference
  genome. Reads with lower alignment coverage are skipped.
- By default, only "core" chromosomes are included (chr1-chr22, chrX, chrY).
  Use `--use-all-chroms` to include all chromosomes.
- Mitochondrial reads (chrM) are included by default. Use `--skip-mito` to
  exclude them.

## Sample name resolution

The sample identifier is determined in the following order:

1. Use the value provided via `--sample` if specified
2. Extract from the `@RG` header's `SM` field if a unique sample name is present
3. Use BAM filename if neither of the above is available

## Output format

The command produces a gzip-compressed file containing:

1. Header section with metadata:
   - `#ID` line: Sample identifier
   - `#CR` lines: Chromosome names and lengths

2. Body: Encoded read bundles representing exon structures extracted from the
   alignments

This transcription profiles output by this command are designed to be merged
together using the `isocall merge` command.
