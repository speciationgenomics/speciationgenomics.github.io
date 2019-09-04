---
title: "Sliding window differentiation, variance and introgression"
layout: archive
permalink: /sliding_windows/
---

As SNP Fst values are very noisy, it is better to compute Fst estimates for entire regions. Selection is expected to not only affect a single SNP and the power to detect a selective sweep is thus higher for genomic regions. How large the genomic region should be depends on the SNP density, how fast linkage disequilibrium decays, how recent the sweep is and other factors. It is thus advisable to try different window sizes. Here, we will use 20 kb windows. An alternative is to use windows of fixed number of sites instead of fixed size. We will use scripts written by [Simon Martin](https://simonmartinlab.org/) which you can download [here](https://github.com/simonhmartin/genomics_general).
Note, that the scripts by Simon are written in Python2 (not Python3 which may be standard in your working environment). If the scripts do not run, you may have to adjust the first line "#!/usr/bin/env python" to your Python2 path.

First, let's convert the vcf file into Simon Martin's geno file. You can download the script [here](https://github.com/simonhmartin/genomics_general/raw/master/VCF_processing/parseVCF.py).

```shell
cd ~/genome_scans
FILE="Pundamilia.Kivu.chr20"
cp /home/data/vcf/Pundamilia.Kivu.chr20.vcf.gz ./
parseVCF.py -i $FILE.vcf.gz -o $FILE.geno.gz
```

First, we will calculate pi for each species and Fst and dxy for each pair of species all in one go.
```shell
popgenWindows.py -g $FILE.geno.gz -o $FILE.Fst.Dxy.pi.csv.gz \
   -f phased -w 20000 -m 10000 -s 20000 \
   -p PundPyt 11725.PunPundPyt,11727.PunPundPyt,11728.PunPundPyt,11729.PunPundPyt \
   -p NyerPyt 11719.PunNyerPyt,11986.PunNyerPyt,11992.PunNyerPyt,11546.PunNyerPyt \
   -p NyerMak 11591.PunNyerMak,11593.PunNyerMak,11595.PunNyerMak,11598.PunNyerMak \
   -p PundMak 13069.PunPundMak,10558.PunPundMak,10560.PunPundMak,11297.PunPundMak \
   -p kivu 64253
```

Note that -w 20000 specifies a window size of 20 kb that is sliding by 20kb (-s 20000) and -m 10000 requests these windows to have a minimum number of 10kb sites covered. The way we have encoded the genotypes (e.g. A/T) in our geno.gz file is called "phased" and we specify that with "-f phased" even though our data is actually not phased.

Next, we calculate fd to test for introgression from the original Pundamilia nyererei (NyerMak)  into P. sp. "nyererei-like" (NyerPyt) using the Lake Kivu cichlid as outgroup. fd is a measure of introgression suitable for small windows. As the output file will not retain any information on which combination of species was used for the test, I like to add this information to the file name.

```shell
ABBABABAwindows.py -w 20000 -m 10 -s 20000 -g $FILE.geno.gz \
   -o $FILE.PundP.NyerP.NyerM.Kivu.fd.csv.gz \
   -f phased --minData 0.5 --writeFailedWindows \
   -P1 PundPyt 11725.PunPundPyt,11727.PunPundPyt,11728.PunPundPyt,11729.PunPundPyt \
   -P2 NyerPyt 11719.PunNyerPyt,11986.PunNyerPyt,11992.PunNyerPyt,11546.PunNyerPyt \
   -P3 NyerMak 11591.PunNyerMak,11593.PunNyerMak,11595.PunNyerMak,11598.PunNyerMak \
   -O kivu 64253
```

For this script we need specify that at least 50% of the individuals of each population need to have data for a site to be considered (--minData 0.5) and we reduce m to 10 as it only considers polymorphic sites.

To plot the results, we need to download the output files that the two scripts produced to plot it on our local computers with R. Please download all files found in the genome_scan folder: /home/data/genome_scan/*

Then we can start plotting:

```r
# Prepare input files:

# Read in the file with sliding window estimates of FST, pi and dxy
windowStats<-read.table("Pundamilia_kivu_div_stats.csv",header=T,sep=",")

# Remove windows with less than 10 kb covered
windowStats<-windowStats[windowStats$sites>10000,]


# Get chrom ends for making positions additive
chrom<-read.table("chrEnds.txt",header=T)
chrom$add<-c(0,cumsum(chrom$end)[1:21])
chrom$add<-chrom$add+c(0,cumsum(rep(10000000,times=21)))

# Make the positions of the divergence and diversity window estimates additive
windowStats$mid<-windowStats$mid+chrom[match(windowStats$scaffold,chrom$chr),3]


# Get fd estimates of 20 kb windows for NyerMak into NyerPyt
fd<-read.delim(file = "Pundamilia_ABBABABA.csv",sep=",",header=T,na.strings = "NaN")

# Remove the weird cases of negative fd or fd above 1
fd[(fd$fd<0 | fd$fd>1) & !is.na(fd$fd),"fd"]<-NA

# Make a density plot to show the distribution of number of sites used per window
# And to find a good threshold
plot(density(fd$sitesUsed,na.rm=T))
abline(v=10)

# Set windows with less than 5 sites to NA for all hybridization stats
fd[fd$sitesUsed<10&!is.na(fd$fd),c("fd","fdM","D")]<-c(NA,NA,NA)

# make positions additive
fd$mid<-fd$mid+chrom[match(fd$scaffold,chrom$chr),3]

##########################################################################
# Plotting statistics along the genome:

# Combine 6 plots into a single figure:
par(mfrow=c(5,1),mar=c(0,5,0,1),cex.lab=1,cex.axis=1)

# Plot Fst between species at Makobe Island
plot(windowStats$mid,windowStats$Fst_NyerMak_PundMak,cex=0.5,pch=19,xaxt="n",
     ylab="Fst Makobe",ylim=c(0,1))
abline(h=mean(windowStats$Fst_NyerMak_PundMak,na.rm=T),col="grey")

# Plot Fst between species at Python Island
plot(windowStats$mid,windowStats$Fst_PundPyt_NyerPyt,cex=0.5,pch=19,xaxt="n",
     ylab="Fst Python",ylim=c(0,1))
abline(h=mean(windowStats$Fst_PundPyt_NyerPyt,na.rm=T),col="grey")

# Plot Dxy between species at Makobe Island
plot(windowStats$mid,windowStats$dxy_NyerMak_PundMak,cex=0.5,pch=19,xaxt="n",
     ylab="dxy Makobe",ylim=c(0,0.03))
abline(h=mean(windowStats$dxy_NyerMak_PundMak,na.rm=T),col="grey")

# Plot Dxy between species at Python Island
plot(windowStats$mid,windowStats$dxy_PundPyt_NyerPyt,cex=0.5,pch=19,xaxt="n",
     ylab="dxy Python",ylim=c(0,0.03))
abline(h=mean(windowStats$dxy_PundPyt_NyerPyt,na.rm=T),col="grey")

# Plot fd
plot(fd$mid,fd$fdM,cex=0.5,pch=19,xaxt="n",
     ylab="fdM",ylim=c(0,1))
abline(h=mean(fd$fdM,na.rm=T),col="grey")

# Add the LG names to the center of each LG
axis(1,at=chrom$add+chrom$end/2,tick = F,labels = 1:22)


##############################################################################

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
