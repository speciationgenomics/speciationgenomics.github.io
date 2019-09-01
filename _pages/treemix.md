---
title: "Treemix"
layout: archive
permalink: /Treemix/
---

`treemix` was written by Joseph K. Pickrell and Jonathan K. Pritchard and is available [here](https://bitbucket.org/nygcresearch/treemix/wiki/Home). If given a set of allele frequencies from a number of populations, it will return the maximum likelihood tree for the set of populations, and optionally attempt to infer a number of admixture events.

It assumes unlinked SNPs and we are thus first going to prune the file for SNPs in high LD. We learned how to do this previously in more detail in the [PCA tutorial](https://speciationgenomics.github.io/pca/), but we will repeat it again here. One could also account for linkage in `treemix` by setting `-k 1000` - i.e setting a large blocksize for the jacknife resampling. However, our current SNP dataset is too large for us anyway. Furthermore, `treemix` does not like missing data. We can therefore remove sites with missing data and perform linkage pruning all in one go. We will use an [LD-pruning script](https://github.com/speciationgenomics/scripts/blob/master/ldPruning.sh) as you already know how to linkage prune.

```shell
FILE=dogs
vcftools --gzvcf $FILE.vcf.gz --max-missing 1.0 --recode --stdout | gzip > $FILE.noN.vcf.gz
ldPruning.sh $FILE.0.1N.vcf.gz
file=$FILE.ldPruned
```
This reduces our dogs dataset from 13,283,544 sites to 77,216 SNPs.

`treemix` requires a special input format. We will use a script that generates this input file from a `vcf` and also a `clust` file - which is a file that provides the information on which sample belongs to which taxon (see [here](https://www.cog-genomics.org/plink/1.9/formats#cluster) for some further information). The `clust` file contains three columns, whereby the first and the second column indicate the name of the individual and the third column indicates the taxon name. We can thus easily generate the `clust` file from our `.pop` file that we generated previously for `ADMIXTOOLS`:

```shell
awk '{print $1,$1,$3}' $FILE.pop > $FILE.clust
```
With this `clust` file ready, we can run the [conversion script](https://github.com/speciationgenomics/scripts/blob/master/vcf2treemix.sh):

```shell
vcf2treemix.sh $FILE.vcf.gz $FILE.clust
```

Once it is finished, we can already run `treemix`. For `treemix` we need to specify how many migration edges it should add. We want to run it with 0 to 5 migration edges and we set the Golden Jackal as the outgroup. Because we only use a single individual per taxon, we need to use the flag `-noss`. Otherwise, treemix will apply a correction for small sample size which is too conservative. Ideally, we would really have multiple individuals of each taxon.

```shell
for i in {0..5}
do
 treemix -i $FILE.treemix.frq.gz -m $i -o $FILE.$i -root GoldenJackal -bootstrap -k 500 -noss > treemix_${i}_log &
done
```

Now we need to download all output files `treemix` has produced to our local machines for visualization of the results in `R`. You should also download the `plotting_funcs.R` script which allows you to plot `treemix` results.

In `R`, we need to load the following packages:

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
Now, we can plot the 6 runs of `treemix` side-by-side:

```R
par(mfrow=c(2,3))
for(edge in 0:5){
  plot_tree(cex=0.8,paste0(prefix,edge))
  title(paste(edge,"edges"))
}
```

To check which parts of the tree are not well modelled for the different runs, we can plot the residuals. For that we need a file with the different taxa listed.

```R
for(edge in 0:5){
 plot_resid(stem=paste0(prefix,edge),pop_order="dogs.list")
}
```
