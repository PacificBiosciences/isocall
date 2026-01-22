# ISOCALL

Isocall is a method for calling RNA isoforms from your entire PacBio Kinnex sample set. It consists of the following steps:

- Prepare annotations of known isoforms (`isocall prep-isoforms`)
- Generate a transcription profile for each sample in the dataset (`isocall profile`)
- Merge the profiles together (`isocall merge`)
- Call isoforms from the resulting merged profile (`isocall call`)

## Installation

A Linux binary can be downloaded from the releases page.

## Example

Prepare isoform annotations from RefSeq / ENCODE GTF file:

```bash
isocall prep-isoforms --gtf /Volumes/USB1/hg38.ncbiRefSeq.gtf.gz --output ref_seq.isoforms.gz
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

- Use `--skip-mito` parameter of `isocall profile` to skip analysis of `chrM`
- Use `--use-all-chroms` to include non-core chromosomes in `isocall profile` and `isocall call`
- The `--reference` argument requires an indexed FASTA file (with a corresponding `.fai` index)

