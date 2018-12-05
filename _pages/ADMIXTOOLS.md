---
title: "ADMIXTOOLS"
layout: archive
permalink: /ADMIXTOOLS/
---

ADMIXTOOLS is a software package developed by the David Reich group. You can find more information here: https://github.com/DReichLab/AdmixTools.

### Generating the input files
ADMIXTOOLS requires eigenstrat input files. To convert a vcf file to eigenstrat, you can use my conversion script. That script just requires the name of the vcf file which can be gzipped or not.

```shell
# move to your home directory
cd ~
# make a directory for ADMIXTOOLS analyses
mkdir ADMIXTOOLS

# mv into that new folder
cd ADMIXTOOLS

# copy the dogs dataset to the new folder
cp /home/data/vcf/dogs.vcf.gz ./

# Convert the vcf to the eigenstrat format required by AMDIXTOOLS
convertVCFtoEigenstrat.sh dogs.vcf.gz
```

Next, we need to make a file that tells ADMIXTOOLS which individuals belong to the same taxa. My script automatically generates dogs.ind which we can use as template for this file by just modifying the third column to indicate the taxon the individual belongs to. If multiple individuals are used per taxon, the same taxon code would be given for each of them. The format is one line per individual, tab-delimited. The first column is the individual namesThe second column indicates the sex and can be M for male, F for female or U for unknown. We do not know or care about the sex here and thus just leave it as U. The third column is the taxon codes. Let's generate a new dogs.pop from the dogs.ind:

```shell
awk '{split($1,pop,"."); print $1"\tU\t"pop[2]}' dogs.ind > dogs.pop
```
Note, it is very important that the order of individuals in this file is not changed.

### D statistics

D statistics, sometimes called ABBA-BABA tests are a simple way to test for hybridization.  Let W, X, Y, Z be four taxa (populations, species, individuals) with a phylogeny ((W,X),Y),Z), with W and X being the focal species, Y the tested species potentially introgressing with W or X, and Z the outgroup used to polarize variants as ancestral (A) or derived (B). Under incomplete lineage sorting only (no hybridization), the number of discordant SNPs grouping W and Y together (BABA patterns) should be roughly equal to the number of SNPs grouping X and Y together (ABBA). The formula implemented in ADMIXTOOLS to compute the D statistic is based on allele frequencies and allows one or multiple individuals per taxon:

For each SNP:
```math
num = (w − x)(y − z )
den = (w + x − 2wx)(y + z − 2yz )
```

Summed across all SNPs:
```math
D = \sum num / \sum den
```

If a single sequence (haploid individual) is used per population, this formula equals to:

D = (n~BABA~ - n~ABBA~) / (n~BABA~+n~ABBA~)

Note, that D statistics are often calculated as (n~ABBA~ - n~BABA~) / (n~BABA~+n~ABBA~). It is thus very important to check which formula is used to correctly interpret the results.

ADMIXTOOLS also outputs z-scores which are the number of standard errors that D deviates from 0. An absolute z score of 3 is generally accepted as significant. If W and X share equal amounts of alleles with Y, D will be 0, or at least the absolute z-score will be below 3. If the D statistic is positive and z>3, W and Y show excess allele sharing. If the D statistic is negative and z<(-3), X and Y show excess allele sharing.

Using ADMIXTOOLS to get a whole-genome point estimate of the D statistic is particularly useful to test multiple different populations for hybridization and to get an overview who hybridized with whom. If it is already known which populations are hybridizing, I would recommend to directly do genome-scans of introgression using fd. Whole-genome estimates of the D statistic work do not require  whole-genome sequencing but work well with RAD data and UCEs. However, I would recommend to not trust results with less than 50 ABBA or 50 BABA patterns. For this tutorial, we will use a dataset of dog species from Freedman et al., 2014, Plos Genetics.
https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1004016

Next, we need to make a file that tells ADMIXTOOLS which combinations of four taxa it should test. This is a simple text file with one combination of 4 taxa per line. You can use blanks or tabs to separate the taxon codes.
```shell
<taxonW> <taxonX> <taxonY> <taxonZ>
```
Let's first test if Basenji and Dingo share equal number of alleles with the Croatian Wolf and then if they share equal number of alleles with the Chinese Wolf. We use the Golden Jackal as outgroup for both tests.
```shell
echo "Basenji Dingo CroatianWolf GoldenJackal" > Dstat_quadruples
echo "Basenji Dingo ChineseWolf GoldenJackal" >> Dstat_quadruples
```

Finally, we need to generate the par file which tells ADMIXTOOLS which input files to use.
```shell
echo "genotypename:dogs.eigenstratgeno" > par.dogs
echo "snpname:      dogs.snp" >> par.dogs
echo "indivname:    dogs.pop" >> par.dogs
echo "popfilename:  Dstat_quadruples" >> par.dogs
```

Now, we are ready to run qpDstat which is the tool of ADMIXTOOLS that computes the D statistiscs. It is very easy to run and just requires the par file.

```shell
qpDstat -p par.dogs
```
The results have the following format -
result:   Pop1 (W)  Pop2 (X) : Pop3 (Y)  Pop4 (Z)  D-stat	z-score	nBABA	nABBA nSNPs

Here, we get:
result:    Basenji      Dingo CroatianWolf GoldenJackal     -0.0033     -0.776  208289 209657 6737777
result:    Basenji      Dingo ChineseWolf GoldenJackal     -0.0346     -6.318  206269 221058 6786283

The number of SNPs differs for different combinations of four taxa because only SNPs covered by all four taxa are used. We find that the first test is not significant (|z|<3), whereas the second test suggests that the Chinese Wolf shows excess allele sharing with the Dingo (nABBA > nBABA). The standard error used to compute the z-score is estimated with a block-jackknife procedure taking linkage among markers into account.


## Unfortunately, we do not have enough time for the f4 tests and f4 ratio tests during the course, but feel free to try it on your own:

### f4 tests

If we want to run an f4 test instead of the D statistic, whereby the four taxa are two pairs of sister taxa ((W,X),(Y,Z)), we can add "f4mode: YES" to the par file.

Let's try this with the following test:

```shell
echo "Basenji Dingo CroatianWolf ChineseWolf" >> f4_quadruples
echo "genotypename: "dogs".eigenstratgeno" > par.f4tests
echo "snpname:      "dogs".snp" >> par.f4tests
echo "indivname:    "dogs".pop" >> par.f4tests
echo "popfilename:  f4_quadruples" >> par.f4tests
echo "f4mode: YES" >> par.f4tests
```
And we can then again run it with:
```shell
qpDstat -p par.f4tests
```
To make the result more informative, we can modify the output a bit with awk and write it into a file:
```shell
qpDstat -p par.f4tests  | \
 awk 'BEGIN{print "W X Y Z D Z.value nBABA nABBA nSNPs"}/result/{print $2,$3,$4,$5,$6,$7}'  > f4.results
```

This file then contains:
W	X	Y	D	Z.value	nBABA nABBA nSNPs
Basenji      Dingo CroatianWolf ChineseWolf      0.001968      4.661  215047 201721 6772673

Given that the z-score is above 3, we interpret this result as evidence for excess allele sharing between either Dingo and Chinese Wolf or Basenji and Croatian Wolf. The f4 test can be useful if no outgroup is available. It can also be used to test for subtle excess allele sharing between two pairs with parallel selection. In the Pundamilia case, we could use it to test for excess allele sharing between the two red and the two blue species. D statistics can be biased if the outgroup is not a real outgroup but shows excess allele sharing with either W or X. In that case, Any taxon used as Y will result in signficant D statistics. Ideally, one would thus run each test with different outgroups and also run some control tests with taxa used as Y that are unlikely to have hybridized with W or X. If that is not possible, f4 tests might help to make sure that the D statistic results are robust.

### f4 ratio test
F4 tests can also be used to estimate the admixture proportion if combined in a smart way. ADMIXTOOLS has a separate tool for computing f4 ratio tests to estimate admixture proprotions. To perform an f4 ratio test, we need five taxa: A, B, X, C, O. The introgressed taxon "X", its sister taxon "C", the source of introgression "B" and its sister taxon "A", and an outgroup "O". The ancestry proportion of B in X is then computed as the ratio of these two f4 tests.

```shell
\alpha = f4(A,O; X,C)/ f4(A,O; B, C)
```
As an example, if we want to estimate the Israeli Wolf ancestry proportion of the Basenji, we can compute it using the Basenji as X, the Dingo as C, the Israeli Wolf as B and the Chinese Wolf as A.

ADMIXTOOLS requires a line of the f4 ratio test taxa in the following format (colons are ignored):
A 0 : X C:: A 0 : B C

We thus have to write the taxa as follows:
echo "ChineseWolf GoldenJackal : Basenji Dingo :: ChineseWolf GoldenJackal : IsraeliWolf Dingo" > f4ratio.test

As always, we need to make a par file for ADMIXTOOLS specifying that file as popfilename.

```shell
echo "genotypename: "dogs".eigenstratgeno" > par.f4ratio
echo "snpname:      "dogs".snp" >> par.f4ratio
echo "indivname:    "dogs".pop" >> par.f4ratio
echo "popfilename:  f4ratio.test" >> par.f4ratio

```
We can then run the f4 ratio test with the following command:

```shell
qpF4ratio -p par.f4ratio >logfile
```


The results have the following format:
result: A O X C:  A O B C alpha std.err Z(null=0)

We get the following result which we interpret as that the Basenji got a 34% ancestry contribution of the Israeli Wolf.
```shell
result: ChineseWolf GoldenJackal    Basenji      Dingo  : ChineseWolf GoldenJackal IsraeliWolf      Dingo     0.660327     0.092472      7.141
```
