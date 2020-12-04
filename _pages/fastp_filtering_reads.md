---
title: "Trimming reads and removing adapter sequences and polyG tails"
layout: archive
permalink: /fastp/
---

Sometimes Illumina adapter sequences are still present in some reads because adapters can form adapter dimers and then one of them gets sequenced or if a DNA fragment is shorter than the read length, the sequencer continues to "read-through" into the adapter at the end of the DNA fragment. In the latter case the forward and the reverse read will contain adapter sequences, which is called a "palindrome". While a full adapter sequence can be identified relatively easily, reliably identifying a short partial adapter sequence is inherently difficult. However, if there is a short partial adapter present at the end of the forward read and the beginning of the reverse read, that is a good sign for a "palindrome" sequence.
To anyone using Novaseq, or any NextSeq Illumina technology. Watch out for overrepresented polyG sequences (weirdly long sequences of GGGGGGG), particularly in the reverse reads. This is a problem of the latest Illumina instruments that use a [two-colour system](https://sequencing.qcfail.com/articles/illumina-2-colour-chemistry-can-overcall-high-confidence-g-bases/) to infer the bases. A lack of signal is called as G with high confidence. These polyG tails need to be removed or the read will not map well to the reference genome.
Reads that start or end with very low quality can be aligned better if the bad quality parts are trimmed off. We will use [fastp](https://github.com/OpenGene/fastp) to fix all of these issues. fastp can remove low quality reads, adapters and polyG tails. It even automatically detects what adapters were used.

```shell
forward=wgs.R1
reverse=wgs.R2

fastp --in1 ${forward}.fastq.gz --in2 ${reverse}.fastq.gz --out1 ${forward}.trimmed.fastq.gz --out2 ${reverse}.trimmed.fastq.gz --unpaired1 ${forward}.unpaired.fastq.gz --unpaired2 ${reverse}.unpaired.fastq.gz --reads_to_process 1000000 -l 50 &> $forward.log
```
### Parameters:
*in1 and in2: specify your files of forward (1) reads and of the reverse (2) reads.
*out1 and out2: specify the output files for forward and reverse reads that are still Paired.
*unpaired1 and unpaired2: specify the output files for reads that are not paired anymore.
*reads_to_process: specify how many reads should be processed. As processing the entire file can take long, we will set this to 1 million. Importantly, this is only used for testing purposes. For the real data, you should of course remove this options.
*l 50: this specifies that if a read is shorter than 50 basepairs after all filters, it should be removed.


### PolyG tail trimming
This feature is enabled for NextSeq/NovaSeq data by default, and you can specify -g or --trim_poly_g to enable it for any data, or specify -G or --disable_trim_poly_g to disable it.

### Removal of adapter sequences
Adapter trimming is enabled by default, but you can disable it by -A or --disable_adapter_trimming. Adapter sequences can be automatically detected for both PE/SE data.

### Length filter
Reads below the length threshold (e.g. due to adapter removal) are removed. Length filtering is enabled by default. The minimum length requirement is specified with -l or --length_required.

### Quality filtering
Quality filtering is enabled by default, but you can disable it by -Q or disable_quality_filtering. Currently it supports filtering by limiting the N base number (-n, --n_base_limit), and the percentage of unqualified bases.
To filter reads by its percentage of unqualified bases, two options should be provided:
    -q, --qualified_quality_phred     the quality value that a base is qualified. Default 15 means phred quality >=Q15 is qualified.
    -u, --unqualified_percent_limit   how many percents of bases are allowed to be unqualified (0~100). Default 40 means 40%
