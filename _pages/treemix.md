---
title: "Treemix"
layout: archive
permalink: /Treemix/
---

Treemix was written by Joseph K. Pickrell and Jonathan K. Pritchard (https://bitbucket.org/nygcresearch/treemix/wiki/Home). If given a set of allele frequencies from a number of populations, it will return the maximum likelihood tree for the set of populations, and optionally attempt to infer a number of admixture events.

It assumes unlinked SNPs and we are thus first going to prune the file for SNPs in high LD. One could also account for linkage in treemix by setting "-k 1000" but the current file is too large for us anyways. It also does not like missing data. We thus remove sites with missing data and perform linkage pruning. You can use my LD-pruning script as you already know how to linkage prune.

```shell
FILE=dogs
vcftools --gzvcf $FILE.vcf.gz --max-missing 1.0 --recode --stdout | gzip > $FILE.noN.vcf.gz
ldPruning.sh $FILE.0.1N.vcf.gz
file=$FILE.ldPruned
```
This reduces our dogs dataset from 13,283,544 sites to 77,216 SNPs.

Treemix requires a special input format. I have written a script that generates this input file from a vcf file and a clust file which is a file that provides the information on which sample belongs to which taxon. The clust file contains three columns, whereby the first and the second column indicate the name of the individual and the third column indicates the taxon name. We can thus easily generate the clust file from our .pop file that we generated previously for ADMIXTOOLS:

```shell
awk '{print $1,$1,$3}' $FILE.pop > $FILE.clust
```
With this clust file ready, we can run my conversion script:

```shell
vcf2treemix.sh $FILE.vcf.gz $FILE.clust
```
Once it is finished, we can already run treemix. For treemix we need to specify how many migration edges it should add. We want to run it with 0 to 5 migration edges and we set the Golden Jackal as outgroup. Because we only use a single individual per taxon, we need to use the flag -noss. Otherwise, treemix will apply a correction for small sample size which is too aggressive. Ideally, we would really have multiple individuals of each taxon.

```shell
for i in {0..5}
do
 treemix -i $FILE.treemix.frq.gz -m $i -o $FILE.$i -root GoldenJackal -bootstrap -k 500 -noss > treemix_${i}_log &
done
```

Now we need to download all output files of treemix to the local computer for visualization of the results in R. Please, also download the plotting_funcs.R script which allows you to plot Treemix results.


In R, we need to load the following packages:
```R
library(RColorBrewer)
library(R.utils)
source("plotting_funcs.R") # here you need to add the path
```
Next, we need to set the working directory and give a prefix for the file names:

```R
setwd("~/treemix/") # of course this needs to be adjusted
prefix="dogs.dp10.max0.25N.LDpruned."
```
Now, we can plot the 6 runs of treemix side-by-side:

```R
par(mfrow=c(2,3))
for(edge in 0:5){
  plot_tree(cex=0.8,paste0(prefix,edge))
  title(paste(edge,"edges"))
}
```

To check which parts of the tree are not well modeled for the different runs, we can plot the residuals. For that we need a file with the different taxa listed.

```R
for(edge in 0:5){
 plot_resid(stem=paste0(prefix,edge),pop_order="dogs.list")
}
```
