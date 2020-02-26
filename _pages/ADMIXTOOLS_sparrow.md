---
title: "ADMIXTOOLS and admixr"
layout: archive
permalink: /ADMIXTOOLS_admixr/
---

ADMIXTOOLS is a software package developed by the David Reich group. You can find more information [here](https://github.com/DReichLab/AdmixTools). It can be a little tricky to use for beginners (and indeed, more advanced users too!). A more straightforward way to use ADMIXTOOLS is using the `R` package `admixr` which is well [documented](https://github.com/bodkan/admixr) with clear [tutorials](https://bodkan.net/admixr/articles/tutorial.html). Note that `admixr` does not replace ADMIXTOOLS, but instead requires a local installation of the program to run - it is essentially an interface for the software. To run it for the course, we will be using an [RStudio Server](https://www.rstudio.com/products/rstudio/download-server/).

### Generating the input files

ADMIXTOOLS requires `eigenstrat` input files. To convert a vcf file to eigenstrat, we will use a conversion script written by Joana. That script just requires the name of the vcf file (NB - the script can use uncompressed vcfs too). We are going to use a dataset of whole genome resequencing data from *Passer* sparrows.

```shell
# move to your home directory
cd ~
# make a directory for ADMIXTOOLS analyses
mkdir admixtools

# move into that new folder
cd admixtools

# copy the dogs dataset to the new folder
cp /home/data/sparrows/sparrows_chr8_unphased.vcf.gz ./

# copy the conversion script to your directory
cp /home/scripts/convertVCFtoEigenstrat.sh .

# Convert the vcf to the eigenstrat format required by AMDIXTOOLS
sh convertVCFtoEigenstrat.sh sparrows_chr8_unphased.vcf.gz
```

### Preparing the R environment

The first step you need to take is to load the `admixr` package in `R`. You can do this like so:

```r
library(admixr)
```

With `admixr` loaded, we are now ready to tell the package where to look for the data. We will set a data prefix so it knows what to look for:

```r
# set data prefix
data_prefix <- "../admixtools/sparrows"
```

Once this is done, we can then load the data like so:

```r
# read in data
snps <- eigenstrat(data_prefix)
```

Finally, we can count our SNPs to get an idea of how much workable data we have.

```r
# count SNPs
snp_info <- count_snps(snps)
```

Now we are ready to calculate some statistics!

### D statistics

D statistics, sometimes called ABBA-BABA tests are a simple way to test for hybridization.  Let W, X, Y, Z be four taxa (populations, species, individuals) with a phylogeny ((W,X),Y),Z), with W and X being the focal species, Y the tested species potentially introgressing with W or X, and Z the outgroup used to polarize variants as ancestral (A) or derived (B). Under incomplete lineage sorting only (no hybridization), the number of discordant SNPs grouping W and Y together (BABA patterns) should be roughly equal to the number of SNPs grouping X and Y together (ABBA). The formula implemented in ADMIXTOOLS to compute the D statistic is based on allele frequencies and allows one or multiple individuals per taxon:

For each SNP:
```math
num = (w − x)(y − z )
den = (w + x − 2wx)(y + z − 2yz )
```

Summed across all SNPs:
```math
D = sum(num) / sum(den)
```

If a single sequence (haploid individual) is used per population, this formula equals to:
```math
D = (nBABA - nABBA) / (nBABA+nABBA)
```

Note, that D statistics are often calculated as ABBA-BABA, whereas `ADMIXTOOLS` (and by extension `admixr`) computes BABA-ABBA. It is thus very important to check which formula is used to correctly interpret the results.

`ADMIXTOOLS` also outputs z-scores which are the number of standard errors that D deviates from 0. An absolute z score of 3 is generally accepted as significant. If W and X share equal amounts of alleles with Y, D will be 0, or at least the absolute z-score will be below 3. If the D statistic is positive and z>3, W and Y show excess allele sharing. If the D statistic is negative and z<(-3), X and Y show excess allele sharing.

Using `ADMIXTOOLS` to get a whole-genome point estimate of the D statistic is particularly useful to test multiple different populations for hybridization and to get an overview who hybridized with whom. If it is already known which populations are hybridizing, we would recommend to directly do genome-scans of introgression using fd. Whole-genome estimates of the D statistic work do not require  whole-genome sequencing and work well with RAD data and UCEs. **However, we would also recommend you treat results with less than 50 ABBA or 50 BABA patterns with caution**.

Let's first test if house sparrows from Europe and Spanish sparrows share more alleles with one another than expected by chance. Her we use the tree sparrow as an outgroup.

```r
d1 <- d(W = "house-european_house-uk", X = "bactrianus-bactrianus-kazahkstan", Y =  "spanish-spanish-cv", Z = "tree", data = snps)
```
Look at the results. Note that `admixr` formats its results using `tidyr` principles.

The number of SNPs differs for different combinations of four taxa because only SNPs covered by all four taxa are used. The standard error used to compute the z-score is estimated with a block-jackknife procedure taking linkage among markers into account.

A nice thing about `admixr` is that it makes it really easy to perform tests on multiple populations in one go. For example, what if we want to perform *D* statistic tests fpr all European house and Italian sparrow populations? To do that, all we need to do is use some of our `R` skills in order to extract the population names.

```r
pops <- snp_info %>% filter(grepl("^house|^italian", label)) %>% pull(label) %>% unique()
```

Then we can run the command as we did before, just with `W = pops` instead.

```r
d2 <- d(W = pops, X = "bactrianus-bactrianus-kazahkstan", Y = "spanish-spanish-cv", Z = "tree", data = snps)
```

That's a lot of tests, but of course because we are using R it is very easy to plot our results...

### f4 tests

Sometimes we might want to run an f4 test instead of the D statistic. This is a test of whether four taxa are two pairs of sister taxa ((W,X),(Y,Z)).

Let's try this with the following test:

```r
f1 <- f4(W = "house-european_house-uk", X = "bactrianus-bactrianus-kazahkstan",
         Y = "spanish-spanish-kazahkstan", Z = "spanish-spanish-cv", data = snps)
```

Then we look at the results

```r
f1
```

 The f4 test can be useful if no outgroup is available. It can also be used to test for subtle excess allele sharing between two pairs with parallel selection. In the Pundamilia case, we could use it to test for excess allele sharing between the two red and the two blue species. D statistics can be biased if the outgroup is not a real outgroup but shows excess allele sharing with either W or X. In that case, Any taxon used as Y will result in significant D statistics. Ideally, one would thus run each test with different outgroups and also run some control tests with taxa used as Y that are unlikely to have hybridized with W or X. If that is not possible, f4 tests might help to make sure that the D statistic results are robust.

 Let's try some other examples:

 ```r
f2 <- f4(W = "house-european_house-uk", X = "bactrianus-bactrianus-kazahkstan",
         Y = "spanish-spanish_admix-badajoz_spain", Z = "spanish-spanish-cv", data = snps)
f2

f3 <- f4(W = "house-european_house-uk", X = "bactrianus-bactrianus-kazahkstan",
         Y = "spanish-spanish-cv", Z = "tree", data = snps)
f3
```

### f4 ratio test

F4 tests can also be used to estimate the admixture proportion if combined in a smart way. `ADMIXTOOLS` has a separate tool for computing f4 ratio tests to estimate admixture proportions. To perform an f4 ratio test, we need five taxa: A, B, X, C, O. The introgressed taxon "X", its sister taxon "C", the source of introgression "B" and its sister taxon "A", and an outgroup "O". The ancestry proportion of B in X is then computed as the ratio of these two f4 tests.

```shell
alpha = f4(A,O; X,C)/ f4(A,O; B, C)
```
As an example, we will estimate the proportion of Spanish sparrow ancestry in a mainland population of Italian sparrows.

Running this in `admixr` is simple:

```r
f4r1 <- f4ratio(X = "italian-italian_mainland-crotone", A =  "spanish-spanish-cv", B = "spanish-spanish-kazahkstan", C = "house-european_house-uk", O = "tree", data = snps)
```
We can check the result:

```r
f4r1
```

Here we can see that the Italian population collected from Crotone has 44% of its genome derived from the Spanish sparrow. But we heard earlier that this varied across the distribution of the Italian sparrow. So why don't we run this test again for each population all at once?

First, we get the Italian population information out:

```r
italians <- pops %>% grep("italian", ., value = T)
```

Then we run the test for all at once

```r
f4r2 <- f4ratio(X = italians, A =  "spanish-spanish-cv", B = "spanish-spanish-kazahkstan", C = "house-european_house-uk", O = "tree", data = snps)
```

Once its done, we can check the results!

```r
f4r2
```

And of course this being R, we can also plot these results...
