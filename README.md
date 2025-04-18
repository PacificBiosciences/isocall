# ISOCALL

Isocall is a method for calling RNA isoforms from your entire PacBio Kinnex sample set. It consists of the following steps:

- Generate a transcription profile for each sample in the dataset (`isocall profile`)
- Merge the profiles together (`isocall merge`)
- Call isoforms from the resulting merged profile (`isocall call`)

## Building isocall

Isocall is written in Rust. If you don’t already have Rust on your system, you can install it like so:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Once Rust is installed, you can download isocall’s source code and build it like so:

```bash
git clone ssh://git@bitbucket.nanofluidics.com:7999/~edolzhenko/isocall.git
cd isocall/
cargo build --release
```

You can now find the binary in `target/release/isocall`.

## Example

Generate transcription annotation models from RefSeq / ENCODE GTF file:

```bash
isocall models --gff /Volumes/USB1/hg38.ncbiRefSeq.gtf.gz --output ref_seq.gz
```

Suppose your dataset consists of three samples `sampleA`, `sampleB`, and `sampleC`.  Run the profile command to generate a profile for each sample:

```bash
isocall profile --reads sampleA.bam --sample sampleA --output sampleA.gz
isocall profile --reads sampleB.bam --sample sampleB --output sampleB.gz
isocall profile --reads sampleC.bam --sample sampleC --output sampleC.gz
```

Now merge the profiles together:

```bash
isocall merge --txs sampleA.gz sampleB.gz sampleC.gz --output merged.gz
```

And finally call the isoforms:

```bash
isocall call --txs merged.gz --models ref_seq.gz --output-prefix merged
```
