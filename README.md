<h1 align="center">ISOCALL</h1>

<p align="center">Isocall is a method to joint-call RNA isoforms across many PacBio IsoSeq samples.</p>

> [!WARNING]
> **Please note:** Isocall is currently in active development and should be used for experimentation and feedback.

Isocall works as a four-step workflow:

- Prepare known isoforms from a reference annotation with [`isocall prep-isoforms`](docs/prep-isoforms.md)
- Generate one transcriptome profile per sample with [`isocall profile`](docs/profile.md)
- Merge profiles with [`isocall merge`](docs/merge.md)
- Call known and novel isoforms with [`isocall call`](docs/call.md)

## Authors and contributors

| Name                 | Affiliation     |
|----------------------|-----------------|
| Egor Dolzhenko       | PacBio          |
| Jocelyne Bruand      | PacBio          |
| Yerbol Kurmangaliyev | Brandeis U.     |
| Megan Schertzer      | U. of Virginia  |
| Jonathan Belyeu      | PacBio          |
| Ryan Gossart         | Brandeis U.     |
| Elizabeth Tseng      | PacBio          |
| Zev Kronenberg       | PacBio          |

## Installation

A Linux binary is available on the GitHub releases page.

A container image is also available on Quay:
[quay.io/repository/pacbio/isocall](https://quay.io/repository/pacbio/isocall)

## Where Isocall fits in the Iso-Seq analysis workflow

Isocall is an alternative to the clustering (`isoseq cluster`) and collapse
(`isoseq cluster2`) part of the current [Iso-Seq workflow](https://isoseq.how/getting-started.html).
It starts from FLNC BAMs aligned to the same reference genome and performs
multi-sample isoform calling directly. The FLNC BAMs, containing full-length
non-concatemer reads, are produced by `isoseq refine`.

## Quickstart

Start from a GTF annotation, typically provided as `.gtf.gz`, plus one
reference-aligned FLNC BAM per sample.

Prepare known isoforms from RefSeq, GENCODE, or another annotation set:

```bash
isocall prep-isoforms --gtf hg38.ncbiRefSeq.gtf.gz --output ref_seq.isoforms.gz
```

Suppose the dataset consists of three samples, `sampleA`, `sampleB`, and `sampleC`. Generate one profile per sample:

```bash
isocall profile --reads sampleA.bam --sample sampleA --output sampleA.gz
isocall profile --reads sampleB.bam --sample sampleB --output sampleB.gz
isocall profile --reads sampleC.bam --sample sampleC --output sampleC.gz
```

Now merge the profiles together:

```bash
isocall merge --profiles sampleA.gz sampleB.gz sampleC.gz --output merged.gz
```

And finally call the isoforms:

```bash
isocall call --merged-profile merged.gz --known-isoforms ref_seq.isoforms.gz --reference hg38.fa --output-prefix merged
```

The `isocall call` step is tunable. You can relax or adjust calling and
filtering thresholds through CLI parameters or a TOML config file. The default
settings are intentionally more stringent; see
[`docs/call.md`](docs/call.md) and [`examples/config.toml`](examples/config.toml).

This produces:

- `merged.isoforms.gtf.gz`
- `merged.count_matrix.txt`
- `merged.closest_known.txt`

## Requirements and Notes

- `isocall prep-isoforms`, `isocall profile`, and `isocall merge` currently emit BGZF-compressed, gzip-compatible output and expect output paths ending in `.gz`
- The `--reference` argument requires an indexed FASTA file (with a corresponding `.fai` index)
- Passing `--sample` to `isocall profile` is recommended when sample names are not cleanly encoded in the BAM header
- `isocall call` has many adjustable thresholds and filters; if sensitivity is too low for your use case, review the calling parameters before changing the upstream workflow
- Isocall currently does not call novel isoforms on `chrM`; it passes through the reference isoforms for that chromosome

## Command Reference

- [Preparing known isoforms with `isocall prep-isoforms`](docs/prep-isoforms.md)
- [Generating transcription profiles with `isocall profile`](docs/profile.md)
- [Merging profiles with `isocall merge`](docs/merge.md)
- [Calling isoforms with `isocall call`](docs/call.md)
- [Assigning reads to transcripts with `isocall assign-reads`](docs/assign-reads.md)

The CLI also exposes `isocall filter`, but the current implementation is not complete and should not be treated as a supported workflow step yet.

## Need help?

If you notice any missing features, bugs, or need assistance with analyzing the output of isocall, please don't hesitate to open a GitHub issue or reach out to the authors by [email](mailto:edolzhenko@pacificbiosciences.com).

## Support information

Isocall is a pre-release software intended for research use only and not for use in diagnostic procedures. While efforts have been made to ensure that isocall lives up to the quality that PacBio strives for, we make no warranty regarding this software.

As isocall is not covered by any service level agreement or the like, please do not contact a PacBio Field Applications Scientists or PacBio Customer Service for assistance with any isocall release. Please report all issues through GitHub instead. We make no warranty that any such issue will be addressed, to any extent or within any time frame.

### DISCLAIMER

THIS WEBSITE AND CONTENT AND ALL SITE-RELATED SERVICES, INCLUDING ANY DATA, ARE PROVIDED "AS IS," WITH ALL FAULTS, WITH NO REPRESENTATIONS OR WARRANTIES OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, ANY WARRANTIES OF MERCHANTABILITY, SATISFACTORY QUALITY, NON-INFRINGEMENT OR FITNESS FOR A PARTICULAR PURPOSE. YOU ASSUME TOTAL RESPONSIBILITY AND RISK FOR YOUR USE OF THIS SITE, ALL SITE-RELATED SERVICES, AND ANY THIRD PARTY WEBSITES OR APPLICATIONS. NO ORAL OR WRITTEN INFORMATION OR ADVICE SHALL CREATE A WARRANTY OF ANY KIND. ANY REFERENCES TO SPECIFIC PRODUCTS OR SERVICES ON THE WEBSITES DO NOT CONSTITUTE OR IMPLY A RECOMMENDATION OR ENDORSEMENT BY PACIFIC BIOSCIENCES.
