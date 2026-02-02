# isocall call

The `call` command performs isoform calling from merged transcriptome profiles. It identifies both known and novel isoforms, applies quality filters, and outputs a GTF file with isoforms.

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
| `--use-all-chroms` | off | Enable analysis of non-core chromosomes |
| `--config` | none | Path to TOML configuration file |

## Configuration file

The `call` command supports configuration via a TOML file, allowing you to set
parameters of various heuristics used by the method. CLI arguments take
precedence over config file values.

A fully documented example configuration file is provided at [examples/config.toml](../examples/config.toml).

### Example configuration

```toml
# config.toml

[sampling]
min_read_support = 3        # Minimum read support for novel isoforms
max_bundles_per_gene = 100  # Max bundles for isoform identification
max_group_size = 10000      # Max bundles before downsampling

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
bin_width = 10              # Bin width for peak detection (bp)
window_bp = 100             # Window for local background (bp)
min_count = 3               # Minimum count at candidate bin
z_min = 4.0                 # Minimum z-score for peak detection

[multi_exonic_caller]
extend_known_ends = false   # Extend known isoform ends based on reads
```

## Sampling Configuration

The sampling section controls read support thresholds and downsampling behavior.

| CLI Argument | Config Key | Default | Description |
|--------------|------------|---------|-------------|
| `--min-read-support` | `min_read_support` | 3 | Minimum read support for calling novel transcripts |
| `--max-bundles-per-gene` | `max_bundles_per_gene` | 100 | Maximum bundles to use for isoform identification per gene |
| `--max-group-size` | `max_group_size` | 10000 | Maximum bundles per group before downsampling warning |

### When to Adjust Sampling Parameters

- Increase `min_read_support`: For higher confidence novel isoform calls (reduces false positives)
- Decrease `min_read_support`: To detect low-abundance novel isoforms (may increase false positives)
- Increase `max_bundles_per_gene`: For more thorough analysis of complex genes (slower)
- Decrease `max_bundles_per_gene`: For faster processing (may miss some isoforms)

## Internal Priming Filter

The internal priming filter detects and removes novel isoforms that are likely
artifacts of internal priming during reverse transcription.

### How It Works

1. Known isoforms are always retained regardless of downstream sequence content.
Novel isoforms with A-rich downstream sequences are flagged as potential internal priming artifacts

2. A sequence is considered "A-rich" if either:
   - The A nucleotide content exceeds the threshold (default: 60%)
   - There is a run of As satisfying the run length threshold (default: 6)

3. Flagged isoforms are not filtered if their TTS is within the rescue distance (default: 25bp) of an annotated TTS from known isoforms

### Parameters

| CLI Argument | Config Key | Default | Description |
|--------------|------------|---------|-------------|
| `--internal-priming-filter` | `enabled` | `on` / `true` | Enable or disable the filter |
| `--internal-priming-filter-window` | `window` | 20 | Size of the downstream window to analyze (bp) |
| `--internal-priming-filter-fraction` | `fraction` | 0.60 | A-content threshold (proportion, 0.0-1.0) |
| `--internal-priming-filter-run-length` | `run_length` | 6 | Minimum consecutive A's to trigger filtering |
| `--internal-priming-filter-rescue-distance` | `rescue_distance` | 25 | Max distance to annotated TTS for rescue (bp) |

## Relative Abundance Filter

The relative abundance filter removes low-abundance novel isoforms that are similar to higher-abundance isoforms. This helps reduce redundant novel isoform calls that may be artifacts.

### How It Works

For each pair of isoforms, if:

1. They are within `max_distance` of each other
2. The lower-abundance transcript's relative abundance falls below the threshold

Then the lower-abundance novel transcript is filtered out. Known (annotated) isoforms are never filtered.

### Parameters

| CLI Argument | Config Key | Default | Description |
|--------------|------------|---------|-------------|
| `--relative-abundance-filter` | `enabled` | `on` / `true` | Enable or disable the filter |
| `--relative-abundance-filter-max-distance` | `max_distance` | 20 | Maximum distance to compare transcripts |
| `--relative-abundance-filter-abundance-base` | `abundance_base` | 0.1 | Base for relative abundance threshold |

## Multi-exonic caller configuration

These parameters control behavior of multi-exon isoform calling. They are only configurable via the config file.

| Config Key | Default | Description |
|------------|---------|-------------|
| `extend_known_ends` | `false` | Whether to extend known isoform terminal exon coordinates based on reads. When enabled, the start/end coordinates of identified known isoforms will be extended to match the longest observed first/last exons from supporting reads. |

## Mono-exonic caller configuration

These parameters control single-exon isoform calling. They are only configurable via the config file.

| Config Key | Default | Description |
|------------|---------|-------------|
| `bin_width` | 10 | Bin width for peak detection (bp) |
| `window_bp` | 100 | Window for local background estimation (bp) |
| `min_count` | 3 | Minimum count at candidate bin |
| `z_min` | 4.0 | Minimum z-score for peak detection |

## Outputs

The command produces two output files:

1. `<prefix>.isoforms.gtf.gz`: GTF file containing all called isoforms
2. `<prefix>.count_matrix.txt`: Read count matrix with the counts of full splice match reads across samples
