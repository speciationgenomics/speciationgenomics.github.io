---
title: "Treemix"
layout: archive
permalink: /Treemix/
---

`treemix` was written by Joseph K. Pickrell and Jonathan K. Pritchard and is available [here](https://bitbucket.org/nygcresearch/treemix/wiki/Home). If given a set of allele frequencies from a number of populations, it will return the maximum likelihood tree for the set of populations, and optionally attempt to infer a number of admixture events. Note, that the direction of introgression is often wrongly inferred and in general the results of Treemix are more to be taken as a starting point for investigating introgression in more detail than a final result.

Treemix assumes unlinked SNPs and we are thus first going to prune the file for SNPs in high LD. We learned how to do this previously in more detail in the [PCA tutorial](https://speciationgenomics.github.io/pca/), but we will repeat it again here. One could also account for linkage in `treemix` by setting `-k 1000` - i.e setting a large blocksize for the jacknife resampling. However, our current SNP dataset is too large for us anyway. Furthermore, `treemix` does not like missing data. We will therefore remove sites with missing data and perform linkage pruning.

```shell
FILE=dogs
mkdir treemix
cd treemix
vcftools --gzvcf /home/data/vcf/$FILE.vcf.gz --max-missing 1 --recode --stdout | gzip > $FILE.noN.vcf.gz
```
 For LD-pruning, you can use the [LD-pruning script](https://github.com/speciationgenomics/scripts/blob/master/ldPruning.sh) as you already know how to linkage prune.

```shell
wget https://github.com/joanam/scripts/raw/master/ldPruning.sh
chmod +x ldPruning.sh
./ldPruning.sh $FILE.noN.vcf.gz
gzip $FILE.noN.LDpruned.vcf
FILE=$FILE.LDpruned
```

`treemix` requires a special input format. We will use a script that generates this input file from a `vcf` and also a `clust` file - which is a file that provides the information on which sample belongs to which taxon (see [here](https://www.cog-genomics.org/plink/1.9/formats#cluster) for some further information). The `clust` file contains three columns, whereby the first and the second column indicate the name of the individual and the third column indicates the taxon name. We can thus easily generate the `clust` file from our `.pop` file that we generated previously for `ADMIXTOOLS`:

```shell
bcftools query -l $FILE.vcf.gz | awk '{split($1,pop,"."); print $1"\t"$1"\t"pop[2]}' > dogs.clust
```
With this `clust` file ready, we can run the [conversion script](https://github.com/speciationgenomics/scripts/blob/master/vcf2treemix.sh). This script requires plink2treemix.py from [here](https://bitbucket.org/nygcresearch/treemix/downloads/plink2treemix.py):

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

Now we need to download all output files `treemix` has produced to our local machines for visualization of the results in `R`. You should also download the `plotting_funcs.R` script which allows you to plot `treemix` results. This R script is provided with [`treemix`](/usr/local/apps/treemix/1.12/bin/plotting_funcs.R). In addition, you need to download /home/data/vcf/dogs.list which just contains all the populations/breeds (one per line).

First, we need to set the working directory and give a prefix for the file names:

```R
setwd("~/treemix/") # of course this needs to be adjusted
prefix="dogs.LDpruned"
```

We also need to load the following packages:

```R
library(RColorBrewer)
library(R.utils)
source("plotting_funcs.R") # here you need to add the path
```

Now, we can plot the 6 runs of `treemix` side-by-side:

```R
par(mfrow=c(2,3))
for(edge in 0:5){
  plot_tree(cex=0.8,paste0(prefix,".",edge))
  title(paste(edge,"edges"))
}
```

To check which parts of the tree are not well modelled for the different runs, we can plot the residuals. For that we need a file with the different taxa listed.

```R
for(edge in 0:5){
 plot_resid(stem=paste0(prefix,".",edge),pop_order="dogs.list")
}
```

Note, that there are also other ways of inferring admixture. One nice method is making [admixture graphs](https://uqrmaie1.github.io/admixtools/articles/admixtools.html).
