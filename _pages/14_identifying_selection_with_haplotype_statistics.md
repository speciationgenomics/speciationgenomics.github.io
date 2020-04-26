---
title: "Identifying selection with haplotype statistics"
layout: archive
permalink: /haplotypes/
---
As another measure of selection, we will use tests that rely on extended haplotype lengths. During a selective sweep, a variant rises to high frequency so rapidly that linkage disequilibrium with neighbouring polymorphisms is not disrupted by recombination, giving rise to long haplotypes. In regions of low recombination, all haplotypes are expected to be longer than in regions of high recombination. Therefore, it is important to compare the haplotypes against other haplotypes at the same genomic region. This can either be within a population, whereby an ongoing sweep would lead to a single haplotype being very long compared to the other haplotypes, or between populations whereby the population that experienced a sweep has longer haplotypes than the population which was not affected by selection in that genomic region. Haplotypes are best assessed with phased dataset. Phasing basically means figuring out for heterozygous positions which of the alleles are part of the same haplotype or chromosome (e.g. which of the alleles were inherited together on a chromosome from the mother). For instance if an individual is heterozygous at two SNPs with genotypes AG and TC, phasing would tell us if the allele A at SNP1 one was inherited on the same chromosome like T or like C at the second SNP. Phased genotypes would be given as A|G and C|T meaning that A and C are on the same chromosome (e.g. maternal) and G and T are on the same chromosome (e.g. paternal). In the absence of long reads that span these SNPs, we can use statistical phasing using all sequenced individuals of the same population (the more the better). There are lots of different tools for phasing and most of them also impute missing genotypes. This means that they infer missing genotypes statistically resulting in a dataset without missing data. If you have a linkage map, it is recommended to use it for making phasing more accurate. However, here we will just perform a very basic phasing with [SHAPEIT2](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#gettingstarted) which does not impute genotypes and does not require recombination rate information. To compute detect regions with extended haplotype lengths, we will use the excellent `rehh` R package. For more information see the [vignette](https://cran.r-project.org/web/packages/rehh/vignettes/rehh.html). `rehh` is well maintained, continually updated and has [a very informative tutorial](https://cran.r-project.org/web/packages/rehh/vignettes/rehh.html) which we recommend you also check out.

To run `rehh` and perform our analyses, we need to run things in `R`. You can either download the data from [our github](https://github.com/speciationgenomics) and run it locally on your own machine, or we can use our `RStudio` server. Once `R` is running, we are ready to go!

### Setting up the R environment

The first thing we need to do is clear our `R` environment and load the packages we need. Like so:

```r
# clear environment
rm(list = ls())
# load packages
library(rehh)
library(tidyverse)
```

### Reading in data from vcfs

A nice new addition to `rehh` is the ability to read in (and also filter) your data from a vcf. However, it is still quite tricky to split up individuals so instead we will read in a vcf for each population. We read in data using the `data2haplohh` function:

```r
# read in data for each species
# house
house_hh <- data2haplohh(hap_file = "./house_chr8.vcf.gz",
                   polarize_vcf = FALSE)
# bactrianus
bac_hh <- data2haplohh(hap_file = "./bac_chr8.vcf.gz",
                         polarize_vcf = FALSE)
```

This will take a few moments but once the commands are run, we will have read in our data. It is really important that we set `polarize_vcf` to `FALSE` because we have not used an outgroup genome to set our alleles as derived or ancestral. **NB. Some plots and functions in `rehh` will still refer to ancestral and derived even if you use this option**. Instead `rehh` will use the minor and major alleles (in terms of frequency) from our data.

Next, we will filter our data on a **minor allele frequency** or **MAF**. This is really simple in `rehh` with the `subset` function:

```r
# filter on MAF - here 0.05
house_hh_f <- subset(house_hh, min_maf = 0.05)
bac_hh_f <- subset(bac_hh, min_maf = 0.05)
```

This will remove some sites - and we'll be ready to run our haplotype scans.

### Performing a haplotype genome scan - *iHS*

Before we can calculate the statistics we are interested in - *iHS* and *xpEHH* - we need to calculate *iES* statistics. Luckily this is really easy using the `rehh` function `scan_hh`.

```r
# perform scans
house_scan <- scan_hh(house_hh_f, polarized = FALSE)
bac_scan <- scan_hh(bac_hh_f, polarized = FALSE)
```

Note that we once again set the `polarized` argument to `FALSE`. Next we can use the output of this scan to calculate *iHS*. We do this with the `ihh2ihs` function.

```r
# perform iHS on house
house_ihs <- ihh2ihs(house_scan, freqbin = 1)
```

Note that the `freqbin = 1` argument is set again because we are not using polarized data (i.e. we do not know which allele is ancestral or derived). If we did, `rehh` can apply weights to different bins of allele frequencies in order to test whether there is a significant deviation in the *iHS* statistic. However, since this isn't the case we bin everything into a single category.

Having calculated the statistc, let's plot the results. We can either plot the statistic itself like so:

```r
ggplot(house_ihs$ihs, aes(POSITION, IHS)) + geom_point()
```

Or we can plot the log *P*-value to test for outliers.

```r
# plot
ggplot(house_ihs$ihs, aes(POSITION, LOGPVALUE)) + geom_point()
```

Here a log *P*-value of 6 is equivalent to something like P = 10<sup>-6</sup> - which is quite a conservative threshold for an outlier!

### Performing a haplotype genome scan - *xpEHH*

Next we will calculate *xpEHH* which is the cross-population *EHH* test. This is essentially a test for the probability that if we randomly sampled haplotypes from different populations, we would get different haplotypes. Again, `rehh` makes this simple with the `ies2xpehh` function.

```r
# perform xp-ehh
house_bac <- ies2xpehh(bac_scan, house_scan,
                       popname1 = "bactrianus", popname2 = "house",
                       include_freq = T)
```

Here we provide the names of our previous *iES* scans (`bac_scan` and `house_scan`). We can also provide the function with the names of our populations and finally, if we set `include_freq` to `TRUE`, we get the frequencies of alleles in our output, which might be useful if we want to see how selection is acting on a particular position.

Next, we can plot the *xpEHH* values, like so:

```r
# plot
ggplot(house_bac, aes(POSITION, XPEHH_bactrianus_house)) + geom_point()
```

In this plot, highly negative values suggest selection in population 2 (house in this case) whereas positive values indicate selection in population 1. Alternatively, like with *iHS*, we could plot the log *P* values.

```r
ggplot(house_bac, aes(POSITION, LOGPVALUE)) + geom_point()
```

### Examining haplotype structure around a target of selection

One other nice feature of `rehh` is that we can examine haplotype structure around SNPs we think might be under selection. Before we do that, we need to identify the SNP in our dataset with the strongest evidence of being an *xpEHH* outlier.

```r
# find the highest hit
hit <- house_bac %>% arrange(desc(LOGPVALUE)) %>% top_n(1)
# get SNP position
x <- hit$position
```

Here we also set the position of our putative selection SNP as the object `x`. This is because we need to identify where it occurs in our haplotype objects - unfortunately we cannot use the position for this. In the code below, we find the marker id for both our datasets.

```r
marker_id_h <- which(house_hh_f@positions == x)
marker_id_b <- which(bac_hh_f@positions == x)
```

Now we are ready to plot the bifurcation of haplotypes around our site of selection. We do this like so:

```r
house_furcation <- calc_furcation(house_hh_f, mrk = marker_id)
bac_furcation <- calc_furcation(bac_hh_f, mrk = marker_id)
```

We can also plot both of these to have a look at them:

```r
plot(house_furcation, xlim = c(19.18E+6, 19.22E+6))
plot(bac_furcation, xlim = c(19.18E+6, 19.22E+6))
```

Calculating the furcation pattern also makes it possible to calculate the haplotype length around our signature of selection.

```r
house_haplen <- calc_haplen(house_furcation)
bac_haplen <- calc_haplen(bac_furcation)
```

With the haplotype length calculated, we can now plot this to see how haplotype structure differs between our two populations.

```r
plot(house_haplen)
plot(bac_haplen)
```

Here we can see the blue haplotype is much larger around this target and is also more numerous in the European house sparrow.

### Writing out the data for later used

Finally, before we move on to the last tutorial, we are going to write out the data. We'll also make the column names all smaller letters, to make downstream scripting a bit easier.

Use the following code to achieve this:

```r
# write out house bactrianus xpEHH
house_bac <- tbl_df(house_bac)
colnames(house_bac) <- tolower(colnames(house_bac))
write_tsv(house_bac, "./house_bac_xpEHH.tsv")
```

In the last tutorial, we'll use `R` to identify genes that are close to our outlier SNPs.
