---
title: "Calculating LD decay"
layout: archive
permalink: /ld_decay/
---

One of the most common questions we get before running a genome scan analysis is - what size should my sliding window be? There is really no correct way to answer this as it the answer is dependent on your aims and the purpose/scale of the genome scan.

However, a simple rule of thumb if you are trying to do a genome-wide scan is to set the window to roughly the size where  linkage disequilibrium decays to the genome-wide background. This ensures that your analysis windows can be considered independent (at least to an extent). So, how do we calculate LD decay? Read on to find out!

### Setting up

First things first, we need to set up where we are going to work.

```shell
# move to your home directory
cd ~
# make a plink directory
mkdir ld_decay
# move into it
cd ld_decay
```

We next declare an environmental variable for our vcf. Remember we do this because it reduces arduous typing and makes it easier to adjust future scripts. Here we will use the same vcf we used for our PCA analysis.

```shell
# declare vcf
VCF=~/vcf/cichlid_full_filtered_rename.vcf.gz
```
With that all set up, we are ready to calculate LD with `plink`.

### Calculating LD with `plink`

We previously used `plink` to perform linkage pruning, but the program can also perform LD calculations in its own right. Try the following command (we'll break it down later) to calculate LD for chromosome 1 of the *Pundamilia* genome.

```shell
# calc ld with plink
plink --vcf $VCF --double-id --allow-extra-chr \
--set-missing-var-ids @:# \
--maf 0.01 --geno 0.1 --mind 0.5 --chr 1 \
--thin 0.1 -r2 gz --ld-window 100000 --ld-window-kb 1000 \
--ld-window-r2 0 \
--make-bed --out cichlid_chr1
```

This should run pretty quickly and at least some of it should look familiar to the command we ran previously. What is new this time? We'll explain each option, including the ones we ran before.

* `--vcf` - specified the location of our VCF file.
* `--double-id` - told `plink` to duplicate the id of our samples (this is because plink typically expects a family and individual id - i.e. for pedigree data - this is not necessary for us.
* `--allow-extra-chr` - allow additional chromosomes beyond the human chromosome set. This is necessary as otherwise plink expects chromosomes 1-22 and the human X chromosome.
* `--set-missing-var-ids` - also necessary to set a variant ID for our SNPs. Human and model organisms often have annotated SNP names and so `plink` will look for these. We do not have them so instead we set ours to default to `chromosome:position` which can be achieved in `plink` by setting the option `@:#` - [see here](https://www.cog-genomics.org/plink/1.9/data#set_missing_var_ids) for more info.
* `--maf` - this filters based on minor allele frequency - 0.01 in this case.
* `--geno` - this filters out any variants where more than X proportion of genotypes are missing data. Here we throw out anything with >10% missing dataset.
* `--mind` - this removes any individual with more than 50% missing data.
* `--chr` - an option to choose the chromosome we are running the analysis on. Here we are only performing it on chromosome 1.
* `--thin` - thin randomly thins out the data - i.e. it randomly retains *p* proportion of the data. Here we set that to 0.1 or 10%. This is done to ensure the analysis runs quickly and our output isn't too big as it can quickly get out of hand!
* `--r2` - finally we're on to the options for LD! This tells `plink` to produce squared correlation coefficients. We also provide the argument `gz` in order to ensure the output is compressed. This is very important as it is easy to produce **EXTREMELY** large files.
* `--ld-window` - this allows us to set the size of the lower end of the LD window. In this case, we set it to 100,000 bp - i.e. any sites with < 100,000 sites between them are ignored.
* `--ld-window-kb` - this is the upper end of the LD window. Here we set it to 1000, meaning that we ignore any two sites more than 1 Mb apart in the genome.
* `--ld-window-r2` - the final LD command - this sets a filter on the final output but we want all values of LD to be written out, so we set it to 0.
* `--make-bed` - this just makes a plink bedfile for future analyses
* `--out` Produce the prefix for the output data.

Running this commnd will produce a compressed `*.gz` file that contains the pairwise LD estimates among all variants passing our filtering thresholds. In this example, it is only a single chromosome and quite small, but it is worth remembering that it could be extremely large if run on an entire genome.

### Calculating average LD across set distances

Now, what we need to do next in order to calculate linkage decay is to calculate the **average** LD across intervals in the genome. This is a bit trickier but I have written a python script that should do it for you. You can run it like so:

```shell
# run mark python script
python ld_decay_calc.py -i cichlid.ld.gz -o cichlid_chr1
```
This will take a few moments to run and will produce two files, the pairwise SNPs arranged into distance classes and the average LD across 100 Kb bins. We'll take the latter and then plot it in R to see the LD decay.

```r
## LD decay plotting script

rm(list = ls())

library(tidyverse)

# set path
my_bins <- "./cichlid_chr1.ld_decay_bins"

# read in data
ld_bins <- read_tsv(my_bins)

# plot LD decay
ggplot(ld_bins, aes(distance, avg_R2)) + geom_line() +
  xlab("Distance (bp)") + ylab(expression(italic(r)^2))
```
Doing this, we see a very clear decay to a background LD of 0.1 as we move 100 Kb from a given site. This indicates we can set our sliding windows to 100 Kb with a reasonable justification.

However as a final point, is important to remember that other processes can alter the LD decay curve - for example, population demography and effective population size.
