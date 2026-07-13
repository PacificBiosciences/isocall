# isocall assign-reads

The `assign-reads` command determines which reads support which transcripts. For
every read in the input BAM file, it outputs the transcript(s) that the read is
compatible with.

## Usage

```bash
isocall assign-reads --reads sample.bam --gtf transcripts.gtf --output pairs.tsv.gz
```

The GTF file is expected to be produced by the `isocall call` command. Both
plain and gzip-compressed GTFs are accepted.

## Required arguments

| Argument   | Description                                       |
|------------|---------------------------------------------------|
| `--reads`  | An aligned BAM file                               |
| `--gtf`    | GTF with transcript annotations                   |
| `--output` | Output BGZF-compressed file (must end with `.gz`) |

## Optional arguments

| Argument       | Default | Description                              |
|----------------|---------|------------------------------------------|
| `--io-threads` | 2       | BGZF/BAM IO threads for input and output |

## Read filtering

`assign-reads` uses the same read filtering rules as `isocall profile`:

- Only primary alignments are used.
- Reads must have at least 95% of their sequence aligned to the reference
  genome.

## Assignment rule

A read's exon structure is extracted from its alignment CIGAR string and
compared with the transcripts in the GTF file:

- Read and transcript must be on the same strand.
- Multi-exon reads are assigned to every transcript with the identical intron
  chain (differences at 5′/3′ ends do not affect the match).
- Single-exon reads are assigned to every single-exon transcript that fully
  contains the read.

Note that a read may be assigned to more than one transcript, for example when
several transcripts share an intron chain.

## Output format

A BGZF-compressed, tab-delimited file with one transcript-read pair per line:

```text
transcript_id read_id
```

Pairs are emitted in the order reads are encountered in the BAM; the transcript
ids for a single read are sorted, so the output is deterministic for a given
input.
