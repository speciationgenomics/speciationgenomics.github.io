---
title: "Raw read explorations"
layout: archive
permalink: /readsExploration/
---

### fastq format

Let's have a look at the first sequence: As we saw in the lecture, each DNA sequence is composed of four lines. Therefore, we need to visualize the first four lines to have a look at the first sequence.

```shell
FILE="RAD1.fastq.gz"
zcat /home/data/fastq/ | head -4
```

Let's count the number of lines in the file:

```shell
zcat /home/data/fastq/${FILE} | wc -l
```

The number of sequences is thus this number divided by 4, or we can count the number of lines starting with the header

```shell
zgrep "@HWI" /home/data/fastq/${FILE} -c
```

We might think that we could have just counted the number of "@".
```shell
zgrep "@" /home/data/fastq/${FILE} -c
```
However, we see that this does not give us the same number. The reason is that @ is also a quality score and thus some quality score lines were also counted.


### Assessing read quality with fastqc

To assess the read quality, we use fastqc which is extremely easy to run and takes only a single argument, the name of the fastq file. It can handle gzipped files.

To get help on fastqc:
```shell
fastqc -h
```

Let's run fastqc on our read subsets:
```shell
fastqc 10558.PunPundMak.R1.subsampled.fastq.gz
fastqc 10558.PunPundMak.R2.subsampled.fastq.gz
```

In addition, we will get subsets of reads from two RAD datasets that have been single-end sequenced. As we do not need copies of these files in all of your personal directories, we want to find out if fastqc allows specifying an output path. As with most tools, we can get help on the possible arguments with the -h flag:
```shell
fastqc -h | less
```

We see that fastqc indeed allows an output directory with -o. We will thus just work in the personal directory and run fastqc giving the file name with its path and specifying the output folder as the current directory (-o ./).

```shell
fastqc -o ./ /home/data/RAD1.fastq.gz
fastqc -o ./ /home/data/RAD2.fastq.gz
```

Now, we need to download the html or all files to the local computer for visualization. You can open the html file with any internet browser.


### Challenging exercises for the bash wizards and those with extra time left

In RAD2 there are some reads with very low GC content. Find the 10 reads with the lowest GC content and check what they are.


Here one very condensed solution: Try to find your own solution first!
```shell
file=RAD2

#Add GC content to each read in fastq file to check reads with highest or lowest GC contents:
zcat ${FILE}.fastq.gz | awk 'NR%4==2' | awk '{split($1,seq,""); gc=0; at=0; n=0; for(base in seq){if(seq[base]=="A"||seq[base]=="T") at++; else if(seq[base]=="G"||seq[base]=="C") gc++; else n++}; print $0,gc/(at+gc+n)*100,n}' > ${FILE}.gc

#Lowest GC content:
sort -k 2 -t " " ${FILE}.gc | head

#Highest GC content:
sort -k 2 -t " " ${FILE}.gc | tail

#Get the worst 10 sequences with all information:
zcat ${FILE}.fastq.gz | grep -f <(sort -k 2 -t " " ${FILE}.gc | tail | cut -d" " -f 1) -A 2 -B 1 > ${FILE}.lowGC

# Make a new fastq file with these reads:
grep -v "^--" ${FILE}.lowGC | gzip > ${FILE}.lowGC.fastq.gz
```

### Subsampling reads
Generate a new file from the fastqz file containing every 1000th read.

```shell
# Forward (R1) reads
zcat /home/data/fastq/10558.PunPundMak.R1.fastq.gz | awk '{printf("%s",$0); n++; if(n%4==0){
printf("\n");}else{printf("\t");} }' | awk 'NR == 1 || NR % 1000 == 0' | tr "\t" "\n" | gzip > 10558.PunPundMak.R1.subsampled.fastq.gz &

# Reverse (R2) reads
zcat /home/data/10558.PunPundMak.R1.fastq.gz | awk '{printf("%s",$0); n++; if(n%4==0){
printf("\n");}else{printf("\t");} }' | awk 'NR == 1 || NR % 1000 == 0' | tr "\t" "\n" | gzip > 10558.PunPundMak.R1.subsampled.fastq.gz &
```
