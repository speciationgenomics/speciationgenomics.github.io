---
title: "ADMIXTURE"
layout: archive
permalink: /ADMIXTURE/
---
ADMIXTURE is a clustering software similar to STRUCTURE with the aim to infer populations and individual ancestries.
You can find the manual here: http://software.genetics.ucla.edu/admixture/admixture-manual.pdf

## Generating the input file
ADMIXTURE requires unlinked (i.e. LD-pruned) SNPs in plink format. It is very easy to generate the input file from a VCF file containing such SNPs. This time we are using a RAD dataset of the same *Pundamilia* species which includes more than 4 individuals per population and some putative hybrid individuals. Linked sites, monomorphic or multiallelic sites, or sites with more than 25% missing data have already been filtered out. Also sites with maf smaller than 0.05 or quality lower than 30 were removed. We can use plink to generate the .bed file which can be read by ADMIXTURE (and other files we do not need):

```shell
FILE=Pundamilia.RAD
plink --vcf $FILE.vcf.gz --make-bed --out $FILE --allow-extra-chr
```
Now, we are ready to run ADMIXTURE. We will run it with cross-validation (default=5-fold CV, for higher, choose e.g. cv=10):
```shell
for i in {2..5}
do
 admixture --cv $FILE.bed $i > log${i}.out
done
```
ADMIXTURE produced a log file that we called log${i}.out and two files for each k: .Q which contains cluster assignments for each individual and .P which contains for each SNP the population allele frequencies.

To identify the best value of k clusters which is the value with lowest cross-validation error, we need to collect the cv errors. I am showing here three different versions to extract the K number and the CV error for each K.
```shell
grep "CV" *out | awk '{print $3,$4}' | sed -e 's/(//;s/)//;s/://;s/K=//'  > $FILE.cv.error
grep "CV" *out | awk '{print $3,$4}' | cut -c 4,7-20 > $FILE.cv.error
awk '/CV/ {print $3,$4}' *out | cut -c 4,7-20 > $FILE.cv.error
```

To make plotting easier, we can make a file with the individual names in one column and the species names in the second column. As the species name is in the individual name, it is very easy to extract the species name from the individual name:
```shell
awk '{split($1,name,"."); print $1,name[2]}' ${file}.nosex > Pundamilia.list
```
Now we are ready to plot the results in R. To make it a bit easier, I have written an R script for you to do that you can just open in R-Studio and we will run it together line by line. Those that have more background on R, should look at this script and try to understand it.

You can get it from here:
```shell
cp /home/ADMIXTURE/ADMIXTURE_plot.r ./
```

First, we need to download all files to your local computers, either using scp (Mac/Linux) or Filezilla (Windows).

```r

# Specify the working directory, where your ADMIXTURE output files are located:
setwd()

# We will define the file prefix as a variable called prefix
prefix="Pundamilia.RAD"

# Get individual names in the correct order
labels<-read.table("Pundamilia.list",sep=" ")

# Name the columns
names(labels)<-c("ind","pop")

# Add a column with population indices to order the barplots
labels$n<-as.factor(labels$pop)
levels(labels$n)<-c(4,2,5,1,3)
labels$n<-as.integer(as.character(labels$n))

# read in the different admixture output files
tbl2=read.table(paste0(prefix,".2.Q"))
tbl3=read.table(paste0(prefix,".3.Q"))
tbl4=read.table(paste0(prefix,".4.Q"))
tbl5=read.table(paste0(prefix,".5.Q"))

par(mfrow=c(4,1),mar=c(0,1,0,0),oma=c(2,1,9,1),mgp=c(0,0.2,0))
barplot(t(as.matrix(tbl2[order(labels$n),])), col=rainbow(n=2),xaxt="n", border=NA,ylab="K=2",yaxt="n")
barplot(t(as.matrix(tbl3[order(labels$n),])), col=rainbow(n=3),xaxt="n", border=NA,ylab="K=3",yaxt="n")
barplot(t(as.matrix(tbl4[order(labels$n),])), col=rainbow(n=4),xaxt="n",  border=NA,ylab="K=4",yaxt="n")
barplot(t(as.matrix(tbl5[order(labels$n),])), col=rainbow(n=5),xaxt="n", xlab="Individual #", border=NA,ylab="K=5",yaxt="n")


# plot cv errors
cv<-read.table(paste0(prefix,".cv.error"),header=T)
par(mfrow=c(1,1),mar=c(3,3,1,1))
plot(cv[,1],cv[,2],pch=19,cex=2,ylab="cross-validation error",xlab="K")

# The best k is clearly 4, let's plot just this one:
bp<-barplot(t(as.matrix(tbl4[order(labels$n),])), col=rainbow(n=4),xaxt="n",  border=NA,ylab="K=4",yaxt="n",space=spaces)
axis(3,at=bp,labels=labels$ind[order(labels$n)],las=2,tick=F,cex=0.6)



# Or a nicer version with better spaces
rep<-as.vector(table(labels$n))
spaces<-0
for(i in 1:length(rep)){spaces=c(spaces,rep(0,rep[i]-1),0.5)}
spaces<-spaces[-length(spaces)]

bp<-barplot(t(as.matrix(tbl2[order(labels$n),])), col=rainbow(n=2),xaxt="n", border=NA,ylab="K=2",yaxt="n",space=spaces)
axis(3,at=bp,labels=labels$ind[order(labels$n)],las=2,tick=F,cex=0.6)
barplot(t(as.matrix(tbl3[order(labels$n),])), col=rainbow(n=3),xaxt="n", border=NA,ylab="K=3",yaxt="n",space=spaces)
barplot(t(as.matrix(tbl4[order(labels$n),])), col=rainbow(n=4),xaxt="n",  border=NA,ylab="K=4",yaxt="n",space=spaces)
barplot(t(as.matrix(tbl5[order(labels$n),])), col=rainbow(n=5),xaxt="n", xlab="Individual #", border=NA,ylab="K=5",yaxt="n",space=spaces)

```
Note, individuals labelled as PunHybrPyt are individuals from Python Island that look phenotypically intermediate. Such individuals are very rare. ADMIXTURE can be used to test if they really are hybrids. This seems to be the case in at least one of them.
