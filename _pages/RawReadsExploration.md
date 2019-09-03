---
title: "Raw read exploration"
layout: archive
permalink: /readsExploration/
---

### fastq format

Let's have a look at the first sequence from our raw read files which are stored in the [fastq format](https://en.wikipedia.org/wiki/FASTQ_format). As we saw in the [lecture](https://github.com/speciationgenomics/presentations/blob/master/NGS_introduction_JM.pdf), each DNA sequence is composed of four lines. Therefore, we need to visualize the first four lines to have a look at the information stored for the first sequence.

First we set the name of the fastq file that we will work with as the variable `FILE`. Then, we copy that file to our directory. Finally, we will examine the first 4 lines. However, we cannot just directly write `head -4 $FILE` like we might with a normal text file because the fastq file is actually compressed. It is thus a binary file which cannot just be read. Luckily, there are many commands that can directly read binary files. Instead of `cat`, which we saw in the [Unix introduction](https://speciationgenomics.github.io/getting_used_to_unix/), we use `zcat`, instead of `grep`, we use `zgrep`. If we want to use any other command, we need to read the file with `zcat` and then pipe the output into our command of choice such as `head` or `tail`.

```shell
FILE="RAD.fastq.gz"
cp /home/data/fastq/${FILE} ./
zcat ${FILE} | head -4
```

These are the four lines which make up the information for each read. You can learn more about what they each mean [here](https://en.wikipedia.org/wiki/FASTQ_format). Now the file is in our directory and readable, let's count the number of lines:

```shell
zcat ${FILE} | wc -l
```

The number of sequences is thus this number divided by 4, or we can count the number of lines starting with the header

```shell
zgrep "@HWI" ${FILE} -c
```

We might think that we could have just counted the number of `@` - i.e. the first symbols for each header.

```shell
zgrep "@" ${FILE} -c
```

However, we see that this does not give us the same number. The reason is that `@` is also used as a symbol for encoding [quality scores](https://en.wikipedia.org/wiki/Phred_quality_score) and thus some quality score lines were also counted.

### Assessing read quality with fastqc

To assess the read quality, we use `fastqc` which is extremely easy to run and can be run with the name of the fastq file as the only argument. It can aslo handle gzipped files.

To get help on `fastqc`:

```shell
fastqc -h | less
```

Let's run `fastqc` on our read subsets:

```shell
fastqc $FILE
```

We can also run it on the whole-genome data files. As we do not need copies of these files in all of your personal directories, we will just write the file names with the paths.

`fastqc` allows an output directory with the `-o` flag. We will thus just work in our home directories and run `fastqc` giving the file name with its path and specifying the output folder as the current directory (i.e. `-o ./`).

```shell
fastqc -o ./ /home/data/fastq/wgs.R1.fastq.gz
fastqc -o ./ /home/data/fastq/wgs.R2.fastq.gz
```

Now, we need to download the html or all files to the local computer for visualization. To download files, mac and linux users can use the command `scp`, Windows users can use `FileZilla` [(see instructions here)](https://speciationgenomics.github.io/logging_on/). You can then open the html file with any internet browser.

### Challenging exercises for the bash wizards and those with extra time left

In the `RAD.fastq.gz `there are some reads with very low GC content. Find the 10 reads with the lowest GC content and check what they are.


Here one very condensed solution: Try to find your own solution first!
```shell
FILE=RAD

cp /home/data/fastq/${FILE} ./

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

As a second exercise, try to generate a new file from the fastqz file containing every 1000th read. This is useful as subsampling is often needed to test software. Fastqc will take very long and a lot of memory if it needs to read in a giant file. It is thus better to subsample if you have large fastq files.

```shell
# Forward (R1) reads
zcat /home/data/fastq/wgs.R1.fastq.gz | awk '{printf("%s",$0); n++; if(n%4==0){
printf("\n")}else{printf("\t")} }' | awk 'NR == 1 || NR % 1000 == 0' | tr "\t" "\n" | gzip > wgs.R1.subsampled.fastq.gz &

# Reverse (R2) reads
zcat /home/data/fastq/wgs.R2.fastq.gz | awk '{printf("%s",$0); n++; if(n%4==0){
printf("\n")}else{printf("\t")} }' | awk 'NR == 1 || NR % 1000 == 0' | tr "\t" "\n" | gzip > wgs.R2.subsampled.fastq.gz &
```
