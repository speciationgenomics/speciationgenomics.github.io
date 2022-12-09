---
title: "Sliding window differentiation, variance and introgression"
layout: archive
permalink: /sliding_windows/
---

There are many different ways to detect regions under divergent selection or that confer barriers to gene flow. In this tutorial, we are going to compute four of them in genomic windows:
- pi, a measure of genetic variation
- Fst, a measure of genomic differentiation
- dxy, a measure of absolute divergence
- fd, a measure of gene flow/introgression

For a tutorial on long-range haplotype statistics to infer selective sweeps, see [here](https://speciationgenomics.github.io/haplotypes/). You may also want to consider more complex methods such as [SweeD](https://academic.oup.com/mbe/article/30/9/2224/999783#74416771) to infer sweeps, or to detect barrier loci: [gIMble](https://europepmc.org/article/ppr/ppr564457), [diem](https://www.biorxiv.org/content/10.1101/2022.03.24.485605v3) or ancestral recombination graph methods [(review)](https://academic.oup.com/genetics/article/221/1/iyac044/6554197).

Note that dxy and pi require monomorphic sites to be present in the dataset, whereas Fst and fd are only computed on bi-allelic sites. It is thus important to filter out indels and multi-allelic sites and to keep monomorphic sites (no maf filter).

As SNP Fst values are very noisy, it is better to compute Fst estimates for entire regions. Selection is expected to not only affect a single SNP and the power to detect a selective sweep is thus higher for genomic regions. How large the genomic region should be depends on the SNP density, how fast linkage disequilibrium decays, how recent the sweep is and other factors. It is thus advisable to try different window sizes. Here, we will use 20 kb windows. An alternative is to use windows of fixed number of sites instead of fixed size. We will use scripts written by [Simon Martin](https://simonmartinlab.org/) which you can download [here](https://github.com/simonhmartin/genomics_general).
Note, that the scripts by Simon are written in Python2 (not Python3 which may be standard in your working environment). If the scripts do not run, you may have to adjust the first line "#!/usr/bin/env python" to your Python2 path.

First, let's convert the vcf file into Simon Martin's geno file. You can download the script [here](https://github.com/simonhmartin/genomics_general/raw/master/VCF_processing/parseVCF.py).

```shell
cd ~
mkdir genome_scans
cd genome_scans

# Copy the vcf file and the information file and a small subset file
cp /home/data/genomeScans/* ./

# Convert the vcf file to geno.gz which is the format that Simons script requires
parseVCF.py -i subset.vcf.gz | bgzip > subset.geno.gz

```

First, we will calculate pi for each species and Fst and dxy for each pair of species all in one go.
```shell
popgenWindows.py -g subset.geno.gz -o subset.Fst.Dxy.pi.csv.gz \
   -f phased -w 20000 -m 10000 -s 20000 \
   -p PundPyt -p NyerPyt -p NyerMak -p PundMak -p kivu \
   --popsFile pop.info
```

Note that -w 20000 specifies a window size of 20 kb that is sliding by 20 kb (-s 20000) and -m 10000 requests these windows to have a minimum number of 10 kb sites covered. The way we have encoded the genotypes (e.g. A/T) in our geno.gz file is called "phased" and we specify that with "-f phased" even though our data is actually not phased. Instead of writing all the individual names into the command, we could give only the species names in the code (e.g. -p pundPyt -p NyerPyt) and with `--popsFile` specify a file that contains a line for each individual with its name and species in a text file.

Next, we calculate fd to test for introgression from the original Pundamilia nyererei (NyerMak) into P. sp. "nyererei-like" (NyerPyt) using the Lake Kivu cichlid as outgroup. fd is a measure of introgression suitable for small windows. As the output file will not retain any information on which combination of species was used for the test, I like to add this information to the file name. We thus have quite a long file name.

```shell
ABBABABAwindows.py -w 20000 -m 10 -s 20000 -g subset.geno.gz \
   -o subset.fd.PundP.NyerP.NyerM.Kivu.csv.gz \
   -f phased --minData 0.5 --writeFailedWindow \
   -P1 PundPyt -P2 NyerPyt -P3 NyerMak -O kivu \
   --popsFile pop.info
```

To speed up the calculation of these statistics, the script can be run on multiple threads by specifying -T <thread number>.

For this script we need to specify that at least 50% of the individuals of each population need to have data for a site to be considered (-\-minData 0.5) and we reduce m to 10 as it only considers polymorphic sites.

To plot the results, we need will use the full dataset and read it into R. You can download all files found in the genome_scan folder: /home/data/genome_scan/*

Then we can start plotting:

```r
rm(list = ls())

rm(list = ls())

# Prepare input files:
# Read in the file with sliding window estimates of FST, pi and dxy
windowStats<-read.csv("genomeScans/Pundamilia_kivu_div_stats.csv",header=T)

# Read in the fd estimates of 20 kb windows for NyerMak into NyerPyt (P1=PundPyt, P2=NyerPyt, P3=NyerMak, outgroup=Kivu cichlid)
fd<-read.csv("genomeScans/Pundamilia_ABBABABA.csv",header=T,na.strings = "NaN")

# Let's have a look at the FST and fd datasets
head(windowStats)
head(fd)

# Let's plot FST between the two younger species
require(ggplot2)
ggplot(windowStats,aes(mid,Fst_PundPyt_NyerPyt))+geom_point()
# Annoyingly, it plots all the FST values on top of each other

# Let's just plot chr1, we can use the tidyverse filter function for that
require(tidyverse)
ggplot(windowStats %>% filter(scaffold == "chr1"),
       aes(mid,Fst_PundPyt_NyerPyt))+geom_point()

# Let's see if the FST peaks show evidence of excess allele sharing (introgression or selection on the same standing variation) with NyerMak
require(gridExtra)
g1<-ggplot(windowStats%>% filter(scaffold == "chr1"),
           aes(mid,Fst_PundPyt_NyerPyt))+geom_point()
g2<-ggplot(fd %>% filter(scaffold == "chr1"),aes(mid,fdM))+geom_point()
grid.arrange(g1, g2, nrow=2)

# Combine the two datasets to check if the high FST outliers have high fd values
allStats<-merge(windowStats,fd,by=c("scaffold","mid"))
ggplot(allStats %>% filter(scaffold == "chr1"),
       aes(fdM,Fst_PundPyt_NyerPyt))+geom_point()

# We actually want to see all chromosomes next to each other,
# so let's add a new column with additive values
# Get chrom ends for making positions additive
# (this is the first two columns of the fasta.fai file of the reference genome)
chrom<-read.table("genomeScans/chrEnds.txt",header=T)

# Make a cumulative sum of the chromosome lengths and
# add a bit more to get gaps between the chromosomes
chrom$add<-c(0,cumsum(chrom$end+10000000)[1:21])

# Make the positions of the divergence and diversity window estimates additive
windowStats$mid<-windowStats$mid+
  chrom[match(windowStats$scaffold,chrom$chr),3]

# make fd positions additive
fd$mid<-fd$mid+chrom[match(fd$scaffold,chrom$chr),3]

# Plotting statistics along the genome:
# Combine 2 plots into a single figure:

par(mfrow=c(2,1),oma=c(3,0,0,0),mar=c(1,5,1,1))

# Plot Fst between species at Makobe Island
plot(windowStats$mid,windowStats$Fst_NyerMak_PundMak,cex=0.5,pch=19,xaxt="n",
     ylab="Fst Makobe",ylim=c(0,1),col="blue")
abline(h=mean(windowStats$Fst_NyerMak_PundMak,na.rm=T),col="grey")

# Plot Fst between species at Python Island
plot(windowStats$mid,windowStats$Fst_PundPyt_NyerPyt,cex=0.5,pch=19,xaxt="n",
     ylab="Fst Python",ylim=c(0,1),col="blue")
abline(h=mean(windowStats$Fst_PundPyt_NyerPyt,na.rm=T),col="grey")
# Add the LG names to the center of each LG
axis(1,at=chrom$add+chrom$end/2,tick = F,labels = 1:22)

# Dxy
# Plot Dxy between species at Makobe Island
plot(windowStats$mid,windowStats$dxy_NyerMak_PundMak,cex=0.5,pch=19,xaxt="n",
     ylab="dxy Makobe",ylim=c(0,0.03),col="blue")
abline(h=mean(windowStats$dxy_NyerMak_PundMak,na.rm=T),col="grey")

# Plot Dxy between species at Python Island
plot(windowStats$mid,windowStats$dxy_PundPyt_NyerPyt,cex=0.5,pch=19,xaxt="n",
     ylab="dxy Python",ylim=c(0,0.03),col="blue")
abline(h=mean(windowStats$dxy_PundPyt_NyerPyt,na.rm=T),col="grey")

# Plot fd along the genome
par(mfrow=c(1,1),oma=c(3,0,0,0),mar=c(1,5,1,1))
plot(fd$mid,fd$fdM,cex=0.5,pch=19,xaxt="n",
     ylab="fdM",ylim=c(0,1),col="blue")
abline(h=mean(fd$fdM,na.rm=T),col="grey")

# Add the LG names to the center of each LG
axis(1,at=chrom$add+chrom$end/2,tick = F,labels = 1:22)

# Are the Fst values of Makobe and Python correlated?
plot(windowStats$dxy_NyerMak_PundMak,
     windowStats$dxy_PundPyt_NyerPyt,
     xlab="dxy Makobe",ylab="dxy Python",
     pch=19,cex=0.3)

# This high correlation is likely due to mutation rate variation
# driving the dxy values. If that is the case we would also
# expect pi and dxy to be highly correlated:
plot(rowMeans(cbind(windowStats$pi_PundPyt,windowStats$pi_NyerPyt)),
     windowStats$dxy_PundPyt_NyerPyt,
     xlab="dxy Makobe",ylab="dxy Python",
     pch=19,cex=0.3)

# Yes, super highly correlated! Ok, so let's use the kivu cichlid
# to account for variation in mutation rate:
windowStats$dxy_Pyt_corr<-windowStats$dxy_PundPyt_NyerPyt/
  rowMeans(cbind(windowStats$dxy_NyerPyt_kivu,
                 windowStats$dxy_PundPyt_kivu))

plot(windowStats$mid,windowStats$dxy_Pyt_corr,
     cex=0.5,pch=19,xaxt="n",
     ylab="dxy Python",col="blue")

# Are FST and dxy corrected for mutation rate correlated?
plot(windowStats$Fst_PundPyt_NyerPyt,windowStats$dxy_Pyt_corr,
     cex=0.5,pch=19,xlab="FST Python",ylab="corr dxy Python",col="blue")
# -> all high FST windows also show high corrected dxy


# Let's compare four selection stats on chr1
g1<-ggplot(windowStats%>% filter(scaffold == "chr1"),
           aes(mid,Fst_PundPyt_NyerPyt))+geom_point()
g2<-ggplot(windowStats%>% filter(scaffold == "chr1"),
           aes(mid,dxy_Pyt_corr))+geom_point()
g3<-ggplot(fd %>% filter(scaffold == "chr1"),
           aes(mid,fdM))+geom_point()
g4<-ggplot(windowStats %>% filter(scaffold == "chr1"),
           aes(mid,pi_PundPyt-pi_NyerPyt))+geom_point()

grid.arrange(g1, g2, g3, g4, nrow=4)


```
