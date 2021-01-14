---
title: "ADMIXTOOLS and admixr"
layout: archive
#permalink: /ADMIXTOOLS_admixr/
---

ADMIXTOOLS is a software package developed by the David Reich group. You can find more information [here](https://github.com/DReichLab/AdmixTools). It can be a little tricky to use for beginners (and indeed, more advanced users too!). A more straightforward way to use ADMIXTOOLS is using the `R` package `admixr` which is well [documented](https://github.com/bodkan/admixr) with clear [tutorials](https://bodkan.net/admixr/articles/tutorial.html). Note that `admixr` does not replace ADMIXTOOLS, but instead requires a local installation of the program to run - it is essentially an interface for the software. To run it for the course, we will be using an [RStudio Server](https://www.rstudio.com/products/rstudio/download-server/).

### Generating the input files

ADMIXTOOLS requires `eigenstrat` input files. To convert a vcf file to eigenstrat, we will use a conversion script written by Joana. That script just requires the name of the vcf file (NB - the script can use uncompressed vcfs too).  Note that if the script fails because it does not like your scaffold names, you can add the option --renameScaff
It will then rename your scaffolds to 1 and make the positions cumulative, i.e. concatenate your scaffolds. It assumes a constant recombination rate of 2 cM/Mb. If that is way off for your species, you may want to change it at the top of the script. We are going to use a dataset of whole genome resequence data from a set of dogs that originally stems comes from [here](https://datadryad.org/resource/doi:10.5061/dryad.sk3p7).

```shell
# move to your home directory
cd ~
# make a directory for ADMIXTOOLS analyses
mkdir admixtools

# move into that new folder
cd admixtools

# copy the dogs dataset to the new folder
cp /home/data/vcf/dogs.vcf.gz ./

# copy the conversion script to your directory
cp /home/scripts/convertVCFtoEigenstrat.sh .

# Convert the vcf to the eigenstrat format required by AMDIXTOOLS
sh convertVCFtoEigenstrat.sh dogs.vcf.gz

```

Have a look in your directory, this will produce a number of files, all named `dogs.pruned`. Before we can progress however, we need to do a little housekeeping to ensure we can read these files in our `R` session. This is a nice moment to test out our `bash` skills.

Take a look at the `dogs.ind` file - you can see the names of each species begin with `1.` or `.2`. We can remove those with `sed`.

```shell
sed -i_bak 's/[12]\.//g' dogs.ind
```

Here we used the `-i` flag with `sed` to perform inline editing - the `_bak` argument to `-i` simply means that we create a backup copy of the original file with the suffix `_bak`.

Now look at the `dogs.ind` file again - there is a third column that just contains `Control`. This is an issue for `admixr` and `ADMIXTOOLS` it will be easier for us downstream if we convert this to the sample names. We can do this easily with awk, like so:

```shell
awk '{print $1"\t"$2"\t"$1}' dogs.ind > dogs.ind2
mv dogs.ind dogs.ind_bk
mv dogs.ind2 dogs.ind
```

Last of all, we need to rename the `dogs.eigenstratgeno` to `dogs.geno` so that `admixr` can read it. We can achieve this easily with `mv`:

```shell
mv dogs.eigenstratgeno dogs.geno
```
### Preparing the R environment

The first step you need to take is to load the `admixr` package in `R`. You can do this like so:

```r
library(admixr)
```

With `admixr` loaded, we are now ready to tell the package where to look for the data. We will set a data prefix so it knows what to look for:

```r
# set data prefix
data_prefix <- "../data_tmp/dogs"
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

Using `ADMIXTOOLS` to get a whole-genome point estimate of the D statistic is particularly useful to test multiple different populations for hybridization and to get an overview who hybridized with whom. If it is already known which populations are hybridizing, we would recommend to directly do genome-scans of introgression using fd. Whole-genome estimates of the D statistic work do not require  whole-genome sequencing and work well with RAD data and UCEs. **However, we would also recommend you treat results with less than 50 ABBA or 50 BABA patterns with caution**. For this tutorial, we will use a dataset of dog species from Freedman et al., 2014, [PloS Genetics](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1004016).

Let's first test if Basenji and Dingo share equal number of alleles with the Croatian Wolf and then if they share equal number of alleles with the Chinese Wolf. We use the Golden Jackal as outgroup for both tests.

```r
test1 <- d(W = "Basenji", X = "Dingo", Y = c("ChineseWolf", "CroatianWolf"), Z = "GoldenJackal", data = snps)
```

We can also test whether Basenji shares an excess of alleles with either the Chinese Wolf or the Israeli Wolf.

```r
test2 <- d(W = "ChineseWolf", X = "IsraeliWolf", Y = "Basenji", Z = "GoldenJackal", data = snps)
```

Let's take a look at the results.

```r
test1
test2
```

Note that `admixr` formats its results using `tidyr` principles.

The number of SNPs differs for different combinations of four taxa because only SNPs covered by all four taxa are used. We find that the first test is not significant (\|z\|<3), whereas the second test suggests that the Chinese Wolf shows excess allele sharing with the Dingo (nABBA > nBABA) and the third test suggests that the Israeli Wolf and the Basenji experienced gene flow. The standard error used to compute the z-score is estimated with a block-jackknife procedure taking linkage among markers into account.


### f4 tests

Sometimes we might want to run an f4 test instead of the D statistic. This is a test of whether four taxa are two pairs of sister taxa ((W,X),(Y,Z)).

Let's try this with the following test:

```r
test3 <- f4(W = "Basenji", X = "Dingo", Y = "CroatianWolf", Z = "ChineseWolf", data = snps)
```

Then we look at the results

```r
test3
```

Given that the z-score is above 3, we interpret this result as evidence for excess allele sharing between either Dingo and Chinese Wolf or Basenji and Croatian Wolf. The f4 test can be useful if no outgroup is available. It can also be used to test for subtle excess allele sharing between two pairs with parallel selection. In the Pundamilia case, we could use it to test for excess allele sharing between the two red and the two blue species. D statistics can be biased if the outgroup is not a real outgroup but shows excess allele sharing with either W or X. In that case, Any taxon used as Y will result in significant D statistics. Ideally, one would thus run each test with different outgroups and also run some control tests with taxa used as Y that are unlikely to have hybridized with W or X. If that is not possible, f4 tests might help to make sure that the D statistic results are robust.

### f4 ratio test

F4 tests can also be used to estimate the admixture proportion if combined in a smart way. `ADMIXTOOLS` has a separate tool for computing f4 ratio tests to estimate admixture proportions. To perform an f4 ratio test, we need five taxa: A, B, X, C, O. The introgressed taxon "X", its sister taxon "C", the source of introgression "B" and its sister taxon "A", and an outgroup "O". The ancestry proportion of B in X is then computed as the ratio of these two f4 tests.

```shell
alpha = f4(A,O; X,C)/ f4(A,O; B, C)
```
As an example, if we want to estimate the Israeli Wolf ancestry proportion of the Basenji, we can compute it using the Basenji as X, the Dingo as C, the Israeli Wolf as B and the Chinese Wolf as A.

Running this in `admixr` is simple:

```r
test4 <- f4ratio(X = "Basenji", A = "ChineseWolf", B = "IsraeliWolf", C = "Dingo", O = "GoldenJackal", data = snps)
```
We can check the result:

```r
test4
```

We can see that the Basenji has 34% of its ancestry from the Israeli Wolf (i.e. 1 - alpha here).
