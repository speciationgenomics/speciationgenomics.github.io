---
title: "Trimming reads and removing adapter sequences"
layout: archive
permalink: /Trimmomatic/
---

Sometimes Illumina adapter sequences are still present in some reads because adapters can form adapter dimers and then one of them gets sequenced or if a DNA fragment is shorter than the read length, the sequencer continues to "read-through" into the adapter at the end of the DNA fragment. In the latter case the forward and the reverse read will contain adapter sequences, which is called a "palindrome". While a full adapter sequence can be identified relatively easily, reliably identifying a short partial adapter sequence is inherently difficult. However, if there is a short partial adapter present at the end of the forward read and the beginning of the reverse read, that is a good sign for a "palindrome" sequence.

Reads that start or end with very low quality can be aligned better if the bad quality parts are trimmed off. We will use [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) to trim reads and remove adapter sequences. As we have paired reads, we will run it in Paired-end (PE) mode which requires 2 input files (for forward and reverse reads) and 4 output files (for forward paired, forward unpaired, reverse paired and reverse unpaired reads).

The current processing steps are:
* ILLUMINACLIP: Cut adapter and other illumina-specific sequences from the read.
* SLIDINGWINDOW: Perform a sliding window trimming, cutting once the average quality within the window falls below a threshold.
* LEADING: Cut bases off the start of a read, if below a threshold quality
* TRAILING: Cut bases off the end of a read, if below a threshold quality
* CROP: Cut the read to a specified length
* HEADCROP: Cut the specified number of bases from the start of the read
* MINLEN: Drop the read if it is below a specified length

In Trimmomatic, different processing steps take one or more settings, delimited by ':' (a colon):
"ILLUMINACLIP:<fastaWithAdapters>:<seed mismatches>:<palindrome clip threshold>:<simple clip threshold>:<minAdapterLength>"

See the Trimmomatic [manual](http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf) for all other options.

```shell
java -jar /home/scripts/trimmomatic.jar PE \
 /home/data/wgs_100k/10558.PunPundMak.R1.100k.fastq.gz \
/home/data/wgs_100k/10558.PunPundMak.R2.100k.fastq.gz \
10558.PunPundMak.R1.100k.paired.fastq.gz 10558.PunPundMak.R1.100k.unpaired.fastq.gz \ 10558.PunPundMak.R2.100k.paired.fastq.gz 10558.PunPundMak.R2.100k.unpaired.fastq.gz \
ILLUMINACLIP:/home/scripts/Trimmomatic-0.39/adapters/TruSeq3-PE.fa/:2:30:10 \
LEADING:5 TRAILING:5 SLIDINGWINDOW:5:10 MINLEN:50
```
