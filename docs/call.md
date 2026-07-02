# isocall call

The `call` command performs isoform calling from merged transcriptome profiles.
It identifies known and novel isoforms, applies quality filters, and outputs a
GTF file with isoform annotations along with a per-sample count matrix.

## Basic usage

```bash
isocall call \
    --merged-profile merged.gz \
    --known-isoforms ref_seq.isoforms.gz \
    --reference hg38.fa \
    --output-prefix results
```

## Required arguments

| Argument | Description |
|----------|-------------|
| `--merged-profile` | Path to the merged transcriptome profile (from `isocall merge`) |
| `--known-isoforms` | Path to known isoforms database (from `isocall prep-isoforms`) |
| `--reference` | Path to reference genome FASTA file (must be indexed with `.fai`) |
| `--output-prefix` | Prefix for output files |

## Important optional arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `--threads` | 1 | Number of threads to use |
| `--config` | none | Path to TOML configuration file |


## Configuration file

The `call` command supports configuration via a TOML file, allowing you to set
parameters of various heuristics used by the method. CLI arguments take
precedence over config file values.

A fully documented example configuration file is provided at [examples/config.toml](../examples/config.toml).

The `--internal-priming-filter` and `--relative-abundance-filter` flags accept
`on` or `off`. Setting either flag to `off` disables that filter regardless of
the config file.

### Example configuration

```toml
# config.toml

[sampling]
min_reads_per_isoform = 3           # Minimum reads required to consider an isoform
min_read_fraction = 0.99            # Fraction of reads to retain for each gene
max_bundles_per_gene = 10000        # Upper bound on bundles retained per gene

[internal_priming_filter]
enabled = true              # Set to false to disable the filter
window = 20                 # Downstream window size (bp)
fraction = 0.60             # A-content threshold (0.0-1.0)
run_length = 6              # Consecutive A threshold
rescue_distance = 25        # Distance to annotated TTS for rescue (bp)

[relative_abundance_filter]
enabled = true              # Set to false to disable the filter
max_distance = 20           # Max distance to compare transcripts
abundance_base = 0.1        # Relative abundance threshold

[mono_exonic_caller]
max_3p_gap = 20             # Max gap (bp) between adjacent 3′ ends within one cluster
min_support = 10            # Minimum total read count for a cluster to be called
five_prime_trim_fraction = 0.05 # Fraction of outermost 5′ read-end observations to trim

[multi_exonic_caller]
extend_known_ends = false   # Extend known isoform ends based on reads
terminal_trim_fraction = 0.10 # Ignore the most extreme 10% of read-supported ends on each side
```

## Sampling configuration

The sampling section controls read support thresholds and downsampling behavior.

| CLI Argument | Config Key | Default | Description |
|--------------|------------|---------|-------------|
| `--min-reads-per-isoform` | `min_reads_per_isoform` | 3 | Minimum reads required to consider an isoform |
| `--min-read-fraction` | `min_read_fraction` | 0.99 | Sample bundles until they cover at least this fraction of reads for each gene |
| `--max-bundles-per-gene` | `max_bundles_per_gene` | 10000 | Upper bound on bundles retained for each gene after read-fraction sampling |

Note that bundles overlapping multiple genes are assigned to the highest-expressed gene for sampling.

### Adjusting sampling parameters

- Increase `min_reads_per_isoform`: For higher confidence novel isoform calls (reduces false positives)
- Decrease `min_reads_per_isoform`: To detect low-abundance novel isoforms (may increase false positives)
- Increase `min_read_fraction`: For more thorough analysis of complex genes (slower)
- Decrease `min_read_fraction`: For faster processing (may miss low-support isoforms)
- Increase `max_bundles_per_gene`: For more thorough analysis of complex genes (slower)
- Decrease `max_bundles_per_gene`: For faster processing (may miss some isoforms)

## Internal priming filter

The internal priming filter detects and removes novel isoforms that are likely
artifacts of internal priming during reverse transcription. It works like so:

1. Known isoforms are always retained regardless of downstream sequence content.
   Novel isoforms with A-rich downstream sequences are flagged as potential
   internal priming artifacts.

2. A sequence is considered "A-rich" if either:
   - The A nucleotide content exceeds the threshold (default: 60%)
   - There is a run of As satisfying the run length threshold (default: 6)

3. Flagged isoforms are not filtered if their TTS is within the rescue
   distance (default: 25 bp) of an annotated TTS from known isoforms.

### Parameters

| CLI Argument | Config Key | Default | Description |
|--------------|------------|---------|-------------|
| `--internal-priming-filter` | `enabled` | `on` / `true` | Enable or disable the filter |
| `--internal-priming-filter-window` | `window` | 20 | Size of the downstream window to analyze (bp) |
| `--internal-priming-filter-fraction` | `fraction` | 0.60 | A-content threshold (proportion, 0.0-1.0) |
| `--internal-priming-filter-run-length` | `run_length` | 6 | Minimum consecutive A's to trigger filtering |
| `--internal-priming-filter-rescue-distance` | `rescue_distance` | 25 | Max distance to annotated TTS for rescue (bp) |

## Relative abundance filter

The relative abundance filter removes low-abundance novel isoforms that are
similar to higher-abundance isoforms. This helps reduce redundant novel isoform
calls that may be artifacts. For each pair of isoforms (higher-abundance vs
lower-abundance), the filter does the following:

1. Calculates distance between the two transcripts from their exon chains: it
sums junction/UTR boundary differences (in bp) and the length of exons that do
not overlap between the two.

2. If the two transcripts are within `max_distance` of each other (and the
lower-abundance one is novel), the filter compares relative abundance to a
distance-dependent threshold:
   - Relative abundance = (supporting read count of the lower-abundance
   transcript) / (supporting read count of the higher-abundance transcript).
   - Threshold = `abundance_base`^((distance − 1) / 5). The cutoff is
   stricter when transcripts are very similar and looser when they are less
   similar.

3. If relative abundance is below the threshold, the lower-abundance novel
transcript is filtered out.

### Parameters

| CLI Argument | Config Key | Default | Description |
|--------------|------------|---------|-------------|
| `--relative-abundance-filter` | `enabled` | `on` / `true` | Enable or disable the filter |
| `--relative-abundance-filter-max-distance` | `max_distance` | 20 | Maximum structural distance to compare transcripts (only pairs within this distance are considered) |
| `--relative-abundance-filter-abundance-base` | `abundance_base` | 0.1 | Base for the distance-dependent threshold: filter when relative abundance <= `abundance_base`^((distance−1)/5) |

## Multi-exonic caller configuration

These parameters control behavior of multi-exon isoform calling. They are only configurable via the config file.

| Config Key | Default | Description |
|------------|---------|-------------|
| `extend_known_ends` | `false` | Whether to extend known isoform terminal exon coordinates based on reads. When enabled, identified known isoforms are extended, but never shrunk, according to the specified trim fraction. |
| `terminal_trim_fraction` | `0.10` | Fraction of the most extreme read-supported ends to ignore on each side when calling multi-exon isoform ends. `0.10` ignores the outermost 10%; `0.0` uses the full observed span. |

## Mono-exonic caller configuration

These parameters control single-exon isoform calling. They are only configurable
via the config file.

Reads are sorted by their 3′-end and grouped into clusters while the distance
between adjacent 3′ ends is below `max_3p_gap`. A cluster containing at least
`min_support` reads, produces an isoform candidate whose 3' end is the mode
of 3' read ends and 5' is obtained by trimming `five_prime_trim_fraction` of
the outermost 5′ read ends.

| Config Key | Default | Description |
|------------|---------|-------------|
| `max_3p_gap` | 20 | Maximum gap (bp) between adjacent 3′ ends within one cluster |
| `min_support` | 10 | Minimum total read count for a cluster to be called |
| `five_prime_trim_fraction` | 0.05 | Fraction of outermost 5′ read-end observations to trim |

## Outputs

The command produces three output files:

1. `<prefix>.isoforms.gtf.gz`: BGZF-compressed, gzip-compatible GTF file
   containing all called isoforms
2. `<prefix>.count_matrix.txt`: Per-sample read counts for each reported
    transcript. Multi-exon transcripts count reads with the same full intron
    chain; single-exon transcripts count single-exon reads fully contained
    within the transcript exon.
3. `<prefix>.closest_known.txt`: A list of reported novel isoforms and the
   closest known isoforms (see below).

### Novel identifiers

Novel transcripts assigned to an annotated gene use the annotated gene
identifier followed by a transcript counter:

```text
<gene_id>|T1
<gene_id>|T2
```

For example, a novel transcript assigned to `GENE1` is reported as
`GENE1|T1`.

Novel genes use their genomic span and strand:

```text
GENE|<seqid>:<start>-<end>|<f/r>
```

`seqid` is the reference contig, `start` is the 1-based inclusive start of the
first exon assigned to the novel gene, `end` is the inclusive end of the last
exon assigned to the novel gene, and `f`/`r` indicates forward or reverse
strand. Novel transcripts on novel genes append the same transcript counter:

```text
GENE|chr1:100-250|f|T1
GENE|chr1:100-250|f|T2
```

### Closest known isoforms

`<prefix>.closest_known.txt` contains one row per reported novel isoform. For
each novel isoform, Isocall compares it with candidate annotated isoforms and
reports the closest match.

| Column                   | Description                                     |
|--------------------------|-------------------------------------------------|
| `novel_gene_id`          | Assigned gene identifier for the novel isoform  |
| `novel_isoform_id`       | Novel isoform identifier                        |
| `novel_isoform_type`     | `monoexonic` or `multiexonic`                   |
| `closest_known_distance` | Inclusion distance to the closest known isoform |
| `known_gene_id`          | Closest known isoform gene identifier, or `NA`  |
| `known_isoform_id`       | Closest known isoform identifier, or `NA`       |
| `known_isoform_type`     | `monoexonic`, `multiexonic`, or `NA`            |

The distance is measured in bases. For a fixed novel isoform and a fixed known
isoform, it is the smallest number of bases that must be added to or removed
from the novel isoform's exonic sequence so that the edited novel isoform is
compatible with the known isoform.

Compatibility is defined by the known isoform:

- Known monoexonic: The edited novel isoform must become one non-empty exon
  fully contained within the known exon. Overhang past either known exon
  boundary is not allowed.
- Known multiexonic: The edited novel isoform may match any contiguous block of
  one or more known exons. Internal splice junctions and internal exon
  boundaries inside the selected block must match the known coordinates exactly.
  If the block starts at the first known exon, overhang past that outer terminal
  boundary is allowed without penalty. If the block ends at the last known exon,
  overhang past that outer terminal boundary is also allowed without penalty.
  At non-terminal block boundaries, the edited boundary must remain within the
  selected known exon.

Isocall reports the known isoform with the minimum distance. If no compatible
annotated isoform exists within the assigned gene, the known-isoform fields are
reported as `NA`. If multiple known isoforms have the same minimum distance,
Isocall prefers a same-class match (`monoexonic` to `monoexonic` or
`multiexonic` to `multiexonic`) and then breaks any remaining tie by transcript
identifier.
