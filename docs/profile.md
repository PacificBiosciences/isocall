# isocall profile

The `profile` command generates a transcription profile from an
FLNC BAM aligned to the reference genome. A profile consists of exon structures
extracted from read alignments. FLNC BAMs, containing full-length non-concatemer
reads, are produced by `isoseq refine`, then aligned to the reference genome
with `pbmm2`.

## Basic usage

```bash
isocall profile --reads sample.bam --output sample.gz
```

## Required arguments

| Argument   | Description                                     |
|------------|-------------------------------------------------|
| `--reads`  | Path to a reference-genome-aligned FLNC BAM     |
| `--output` | Output file path (must end with `.gz`; written as BGZF) |

## Optional arguments

| Argument           | Default          | Description                       |
|--------------------|------------------|-----------------------------------|
| `--sample`         | extract from BAM | Sample identifier for the profile |
| `--use-all-chroms` | off              | Profile non-core chromosomes      |
| `--skip-mito`      | off              | Skip mitochondrial reads (chrM)   |
| `--io-threads`     | 2                | BGZF/BAM IO threads for input and output |

Passing `--sample` is recommended when you want deterministic sample naming.
The `--io-threads` default of `2` is intentionally conservative: `1` disables the
HTSlib thread pool entirely, while `2` enables parallel BAM/BGZF IO without
grabbing many CPUs by default.

## Filtering steps

The profile command applies several filters to ensure only desired alignments
are processed.

- Only primary alignments are used. The following alignment types are skipped:
  - Unmapped reads
  - Supplementary alignments
  - Secondary alignments
- Reads must have at least 95% of their sequence aligned to the reference
  genome. Reads with lower alignment coverage are skipped.
- By default, non-core contigs are skipped. Use `--use-all-chroms` to include
  all chromosomes and contigs.
- Mitochondrial reads (chrM) are included by default. Use `--skip-mito` to
  exclude them.

## Sample name resolution

The sample identifier is determined in the following order:

1. Use the value provided via `--sample` if specified
2. Extract from the `@RG` header's `SM` field if a unique sample name is present
3. Use BAM filename if neither of the above is available

## Output format

The command produces a BGZF-compressed, gzip-compatible file containing:

1. Header section with metadata:
   - `#ID` line: Sample identifier
   - `#CR` lines: Chromosome names and lengths

2. Body: Encoded read bundles representing exon structures extracted from the
   alignments

These single-sample profiles are designed to be merged
together using the `isocall merge` command.
