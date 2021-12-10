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

For a tutorial on long-range haplotype statistics to infer selective sweeps, see [here](https://speciationgenomics.github.io/haplotypes/).

Note that dxy and pi require monomorphic sites to be present in the dataset, whereas Fst and fd are only computed on bi-allelic sites. It is thus important to filter out indels and multi-allelic sites and to keep monomorphic sites (no maf filter).

As SNP Fst values are very noisy, it is better to compute Fst estimates for entire regions. Selection is expected to not only affect a single SNP and the power to detect a selective sweep is thus higher for genomic regions. How large the genomic region should be depends on the SNP density, how fast linkage disequilibrium decays, how recent the sweep is and other factors. It is thus advisable to try different window sizes. Here, we will use 20 kb windows. An alternative is to use windows of fixed number of sites instead of fixed size. We will use scripts written by [Simon Martin](https://simonmartinlab.org/) which you can download [here](https://github.com/simonhmartin/genomics_general).
Note, that the scripts by Simon are written in Python2 (not Python3 which may be standard in your working environment). If the scripts do not run, you may have to adjust the first line "#!/usr/bin/env python" to your Python2 path.

First, let's convert the vcf file into Simon Martin's geno file. You can download the script [here](https://github.com/simonhmartin/genomics_general/raw/master/VCF_processing/parseVCF.py).

```shell
mkdir ~/genome_scans
cd ~/genome_scans

# Copy the vcf file and the information file and a small subset file
cp /home/data/genomeScans/* ./

# Convert the vcf file to geno.gz which is the format that Simons script requires
parseVCF.py -i subset.vcf.gz | bgzip > subset.geno.gz

```

First, we will calculate pi for each species and Fst and dxy for each pair of species all in one go.
```shell
popgenWindows.py -g subset.geno.gz -o subset.Fst.Dxy.pi.csv.gz \
   -f phased -w 20000 -m 10000 -s 20000 \
   -p PundPyt -p NyerPyt -p NyerMak -p PundMak -p kivu 64253 \
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

# Prepare input files:
# Read in the file with sliding window estimates of FST, pi and dxy
windowStats<-read.csv("./genome_scan/Pundamilia_kivu_div_stats.csv",header=T)

# Let's have a look at the FST values
head(windowStats)

# Let's plot FST between the two younger species
require(ggplot2)
ggplot(windowStats,aes(mid,Fst_PundPyt_NyerPyt))+geom_point()

# Annoyingly, it plots all the FST values on top of each other

# Let's just plot chr1, we can use the tidyverse filter function for that
require(tidyverse)
ggplot(windowStats %>% filter(scaffold == "chr1"),aes(mid,Fst_PundPyt_NyerPyt))+geom_point()

# However, we actually want to see all chromosomes next to each other, so let's add a new column with additive values
# Get chrom ends for making positions additive
chrom<-read.table("./genome_scan/chrEnds.txt",header=T)
chrom$add<-c(0,cumsum(chrom$end)[1:21])

# Make the positions of the divergence and diversity window estimates additive
windowStats$mid<-windowStats$mid+chrom[match(windowStats$scaffold,chrom$chr),3]

# Get fd estimates of 20 kb windows for NyerMak into NyerPyt
fd<-read.delim(file = "./genome_scan/Pundamilia_ABBABABA.csv",sep=",",header=T,na.strings = "NaN")

# Remove windows without mid position (windows with 0 sites)
fd<-fd[!is.na(fd$mid),]

# make positions additive
fd$mid<-fd$mid+chrom[match(fd$scaffold,chrom$chr),3]

##########################################################################
# Plotting statistics along the genome:

# Combine 2 plots into a single figure:
par(mfrow=c(2,1),oma=c(3,0,0,0),mar=c(1,5,1,1))

# Plot Fst between species at Makobe Island
plot(windowStats$mid,windowStats$Fst_NyerMak_PundMak,cex=0.5,pch=19,xaxt="n",
     ylab="Fst Makobe",ylim=c(0,1),col=windowStats$scaffold)
abline(h=mean(windowStats$Fst_NyerMak_PundMak,na.rm=T),col="grey")

# Plot Fst between species at Python Island
plot(windowStats$mid,windowStats$Fst_PundPyt_NyerPyt,cex=0.5,pch=19,xaxt="n",
     ylab="Fst Python",ylim=c(0,1),col=windowStats$scaffold)
abline(h=mean(windowStats$Fst_PundPyt_NyerPyt,na.rm=T),col="grey")
# Add the LG names to the center of each LG
axis(1,at=chrom$add+chrom$end/2,tick = F,labels = 1:22)

# Plot Dxy between species at Makobe Island
plot(windowStats$mid,windowStats$dxy_NyerMak_PundMak,cex=0.5,pch=19,xaxt="n",
     ylab="dxy Makobe",ylim=c(0,0.03),col=windowStats$scaffold)
abline(h=mean(windowStats$dxy_NyerMak_PundMak,na.rm=T),col="grey")

# Plot Dxy between species at Python Island
plot(windowStats$mid,windowStats$dxy_PundPyt_NyerPyt,cex=0.5,pch=19,xaxt="n",ylab="dxy Python",ylim=c(0,0.03),col=windowStats$scaffold)
abline(h=mean(windowStats$dxy_PundPyt_NyerPyt,na.rm=T),col="grey")

# Plot fd along the genome
plot(fd$mid,fd$fdM,cex=0.5,pch=19,xaxt="n",
     ylab="fdM",ylim=c(0,1),col=windowStats$scaffold)
abline(h=mean(fd$fdM,na.rm=T),col="grey")

# Add the LG names to the center of each LG
axis(1,at=chrom$add+chrom$end/2,tick = F,labels = 1:22)

# Are dxy and fst correlated?
par(mfrow=c(1,1),mar=c(5,5,1,1),oma=c(1,1,1,1))

plot(windowStats$dxy_NyerMak_PundMak,windowStats$Fst_NyerMak_PundMak,
     xlab="dxy",ylab="Fst",pch=19,cex=0.3)
# ignoring the outliers:
plot(windowStats$dxy_NyerMak_PundMak,windowStats$Fst_NyerMak_PundMak,
     xlab="dxy",ylab="Fst",pch=19,cex=0.3,xlim=c(0,0.01))

# Is dxy correlated with pi, indicating that it is mostly driven by variance in mutation and recombination rates?
plot(rowMeans(cbind(windowStats$pi_PundMak,windowStats$pi_NyerMak),na.rm=T),
     windowStats$dxy_NyerMak_PundMak,cex=0.3,
     xlab="pi",ylab="dxy")
abline(a=0,b=1,col="grey")

# If we correct for mutation rate differences with dxy to kivu
plot(windowStats$dxy_NyerMak_PundMak/rowMeans(cbind(windowStats$dxy_NyerMak_kivu,
                                                    windowStats$dxy_PundMak_kivu),na.rm=T),
     windowStats$Fst_NyerMak_PundMak,ylab="Fst",xlab="dxy normalized",pch=19,cex=0.3)


##############################################################################

# Supplementary


# What are the FST distributions between the two species pairs?
par(mfrow=c(2,2))
boxplot(windowStats$Fst_PundPyt_NyerPyt,windowStats$Fst_NyerMak_PundMak)
abline(h=0)

# Are shared high Fst windows in regions of low recombination?
sharedHighFst<-windowStats[windowStats$Fst_NyerMak_PundMak>0.1 &
                             windowStats$Fst_PundPyt_NyerPyt>0.1, ]
normalFST<-windowStats[windowStats$Fst_NyerMak_PundMak<0.1 &
                             windowStats$Fst_PundPyt_NyerPyt<0.1,]

# Add the ratio of dxy between Makobe to dxy to kivu
sharedHighFst$ratio<-sharedHighFst$dxy_NyerMak_PundMak/sharedHighFst$dxy_NyerMak_kivu
sharedHighFst<-normalFST[!is.infinite(sharedHighFst$ratio)|is.na(sharedHighFst$ratio),]
normalFST$ratio<-normalFST$dxy_NyerMak_PundMak/normalFST$dxy_NyerMak_kivu
normalFST<-normalFST[!is.infinite(normalFST$ratio)|is.na(normalFST$ratio),]

# How many sites of shared high FST (>0.1) are there?
length(sharedHighFst$scaffold)

# Compare stats at windows with high fst to windows with Fst<0.1
vioplot(sharedHighFst$dxy_PundPyt_NyerPyt,normalFST$dxy_NyerMak_PundMak)
vioplot(sharedHighFst$pi_PundPyt,normalFST$pi_PundPyt)
vioplot(sharedHighFst$pi_NyerPyt,normalFST$pi_NyerPyt)
vioplot(sharedHighFst$pi_PundMak,normalFST$pi_PundMak)
vioplot(sharedHighFst$pi_NyerMak,normalFST$pi_NyerMak)

# Do shared high FST windows contain haplotypes predating the Victoria-Kivu split?
vioplot(sharedHighFst$ratio,normalFST$ratio,horizontal = T)

install.packages("beanplot")
require("beanplot")

par(mfrow=c(1,1),mar=c(1,3,1,1),mgp=c(1.6,0.5,0),cex.axis=0.9,cex=1.3,xaxs="i",yaxs="i")
bp<-beanplot(normalFST$ratio,sharedHighFst$ratio,log = "",
             side='both', border='NA',
             col=list('cornflowerblue','blue'),
             ylab='maximum relative divergence' ,what=c(0,1,0,0),xaxt="n",
             ylim=c(0,3),cex=2)
abline(h=1)
par(xpd=F)

# Function to plot the density of dots in a grid
colPlot <- function(dataset=stats,varx,vary,minx=0,miny=0,maxx=0.02,maxy=maxx,title="",xlab=varx,ylab=vary,corr=T){
  rcbpal<-c("#ffeda0","#fed976","#feb24c","#fd8d3c","#fc4e2a","#e31a1c","#bd0026")
  data<-dataset
  rf<-colorRampPalette((rcbpal))
  r<-rf(12)
  xbin<-seq(minx,maxx,length.out = 100)
  ybin<-seq(miny,maxy,length.out = 100)
  # colx=grep(names(data),pattern=varx)
  # coly=grep(names(data),pattern=vary)
  colx=varx
  coly=vary
  freq<-as.data.frame(table(findInterval(x = data[,colx],vec = xbin,all.inside=T),findInterval(x = data[,coly],vec = ybin,all.inside = T)))
  freq[,1] <- as.numeric(as.character(freq[,1]))
  freq[,2] <- as.numeric(as.character(freq[,2]))
  freq2D<-matrix(0,nrow=length(xbin),ncol=length(ybin))
  freq2D[cbind(freq[,1], freq[,2])] <- freq[,3]
  image(xbin,ybin,log10(freq2D),col=r,breaks = seq(minx,max(log10(freq2D)),length.out=length(r)+1),
        xlab=xlab,ylab=ylab,xlim=c(minx,maxx),ylim=c(miny,maxy))
  barval<-round(10^(seq(minx,max(log10(freq2D)),length.out = length(r)+1)))
  title(main=title)
  lm<-summary(lm(data[,coly]~data[,colx]))
  r2<-round(lm$r.squared,3)
  if(corr) legend("top",legend=bquote(r^2 == .(r2)),bty = "n",cex=1.2)
  abline(lm(data[,coly]~data[,colx]))
}

par(mfrow=c(2,2),mar=c(5,5,0,0))

# Are the FST values of the two species pairs correlated?
colPlot(dataset = windowStats,varx = "Fst_PundPyt_NyerPyt",vary = "Fst_NyerMak_PundMak",maxx = 1,maxy=1)
colPlot(dataset = windowStats,varx = "dxy_PundPyt_NyerPyt",vary = "dxy_NyerMak_PundMak",maxx = 0.03,maxy=0.03)

# Are dxy and pi correlated?
colPlot(dataset = windowStats,varx = "pi_NyerPyt",vary = "dxy_PundPyt_NyerPyt",maxx = 0.03,maxy=0.03)
colPlot(dataset = windowStats,varx = "pi_NyerMak",vary = "dxy_NyerMak_PundMak",maxx = 0.03,maxy=0.03)


```
