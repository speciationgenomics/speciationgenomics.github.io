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
plink --vcf $file.vcf.gz --make-bed --out $file --allow-extra-chr
```
Now, we are ready to run ADMIXTURE. We will run it with cross-validation (5-fold CV):
```shell
for i in {2..5}
do
 admixture --cv $file.bed $i -j2 | tee log${i}.out
done
```
Note, piping the stdout into tee allows us to see what admixture writes into the terminal while also saving a copy of that output into the file log$i.out.

ADMIXTURE produced two files for each k: .Q which contains cluster assignments for each individual and .P which contains for each SNP the population allele frequencies.

To identify the best value of k clusters which is the value with lowest cross-validation error, we need to collect the cv errors:
```shell
grep "CV" *out | awk '{print $3,$4}' | sed -e 's/(//;s/)//;s/://;s/K=//'  > $file.cv.error
```
Now we are ready to plot the results in R:
```r
library("stringr")
prefix="Pundamilia.RAD.12"
# Get individual names in the correct order
labels<-read.table(paste0(prefix,".nosex"))
names(labels)<-c("ind","pop")

# Replace the second column by the population information extracted from the individual name
labels$pop<-str_split(labels$pop, "\\.",simplify=T)[,2]

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

# plot cv errors
cv<-read.table(paste0(prefix,".cv.error"),header=T)
par(mfrow=c(1,1),mar=c(3,3,1,1))
plot(cv[,1],cv[,2],pch=19,cex=2,ylab="cross-validation error",xlab="K")

# The best k is clearly 4, let's plot just this one:
bp<-barplot(t(as.matrix(tbl4[order(labels$n),])), col=rainbow(n=4),xaxt="n",  border=NA,ylab="K=4",yaxt="n",space=spaces)
axis(3,at=bp,labels=labels$ind[order(labels$n)],las=2,tick=F,cex=0.6)

```
Note, individuals labelled as PunHybrPyt are individuals from Python Island that look phenotypically intermediate. Such individuals are very rare. ADMIXTURE can be used to test if they really are hybrids. This seems to be the case in at least one of them.
