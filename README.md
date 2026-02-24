<h1 align="center">ISOCALL</h1>

<p align="center">Isocall is a method to joint-call RNA isoforms across many PacBio IsoSeq samples.</p>

> [!WARNING]
> **Please note:** Isocall is currently in active development and should be used for experimentation and feedback.

Isocall workflow consists of the following steps:

- Prepare annotations of known isoforms (`isocall prep-isoforms`)
- Generate a transcription profile for each sample in the dataset (`isocall profile`)
- Merge the profiles together (`isocall merge`)
- Call isoforms from the resulting merged profile (`isocall call`)

## Authors and contributors

| Name                 | Affiliation     |
|----------------------|-----------------|
| Egor Dolzhenko       | PacBio          |
| Jocelyne Bruand      | PacBio          |
| Yerbol Kurmangaliyev | Brandeis U.     |
| Megan Schertzer      | U. of Virginia  |
| Ryan Gossart         | Brandeis U.     |
| Elizabeth Tseng      | PacBio          |
| Zev Kronenberg       | PacBio          |

## Installation

A Linux binary is available in the releases page.

## Example

Prepare annotations of known isoform from RefSeq, ENCODE, or other GTF file:

```bash
isocall prep-isoforms --gtf hg38.ncbiRefSeq.gtf.gz --output ref_seq.isoforms.gz
```

Suppose your dataset consists of three samples `sampleA`, `sampleB`, and `sampleC`.  Run the profile command to generate a profile for each sample:

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

## Notes

- Use `--use-all-chroms` to include non-core chromosomes in `isocall profile` and `isocall call`
- The `--reference` argument requires an indexed FASTA file (with a corresponding `.fai` index)
- Isocall currently does not call novel isoforms on `chrM`; it passes through the reference isoforms for that chromosome

## Reference

- [Generating transcription profiles with `isocall profile`](docs/profile.md)
- [Merging profiles with `isocall merge`](docs/merge.md)
- [Calling isoforms with `isocall call`](docs/call.md)

## Need help?

If you notice any missing features, bugs, or need assistance with analyzing the output of isocall, please don't hesitate to open a GitHub issue or reach out to the authors by [email](mailto:edolzhenko@pacificbiosciences.com).

## Support information

Isocall is a pre-release software intended for research use only and not for use in diagnostic procedures. While efforts have been made to ensure that isocall lives up to the quality that PacBio strives for, we make no warranty regarding this software.

As isocall is not covered by any service level agreement or the like, please do not contact a PacBio Field Applications Scientists or PacBio Customer Service for assistance with any isocall release. Please report all issues through GitHub instead. We make no warranty that any such issue will be addressed, to any extent or within any time frame.

### DISCLAIMER

THIS WEBSITE AND CONTENT AND ALL SITE-RELATED SERVICES, INCLUDING ANY DATA, ARE PROVIDED "AS IS," WITH ALL FAULTS, WITH NO REPRESENTATIONS OR WARRANTIES OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, ANY WARRANTIES OF MERCHANTABILITY, SATISFACTORY QUALITY, NON-INFRINGEMENT OR FITNESS FOR A PARTICULAR PURPOSE. YOU ASSUME TOTAL RESPONSIBILITY AND RISK FOR YOUR USE OF THIS SITE, ALL SITE-RELATED SERVICES, AND ANY THIRD PARTY WEBSITES OR APPLICATIONS. NO ORAL OR WRITTEN INFORMATION OR ADVICE SHALL CREATE A WARRANTY OF ANY KIND. ANY REFERENCES TO SPECIFIC PRODUCTS OR SERVICES ON THE WEBSITES DO NOT CONSTITUTE OR IMPLY A RECOMMENDATION OR ENDORSEMENT BY PACIFIC BIOSCIENCES.
