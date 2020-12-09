---
title: "ADMIXTURE"
layout: archive
permalink: /ADMIXTURE/
---
`ADMIXTURE` is a clustering software similar to `STRUCTURE` with the aim to infer populations and individual ancestries.
You can find the manual [here](http://software.genetics.ucla.edu/admixture/admixture-manual.pdf).

## Generating the input file

`ADMIXTURE` requires unlinked (i.e. LD-pruned) SNPs in plink format. See the [slides](https://github.com/speciationgenomics/presentations/blob/master/2020-6-ADMIXTURE.pdf) for additional requirements. It is very easy to generate the input file from a VCF containing such SNPs. This time we are using a RAD dataset of the same *Pundamilia* species which includes more than 4 individuals per population and some putative hybrid individuals. Linked sites, monomorphic or multiallelic sites, or sites with more than 25% missing data have already been filtered out. Also sites with MAF smaller than 0.05 or Phred quality lower than 30 were removed. We can use `plink` to generate the `.bed` file which can be read by `ADMIXTURE` (and other files we do not need):

```shell
FILE=Pundamilia.RAD

# Make a directory in the home directory
cd ~
mkdir ADMIXTURE
cd ADMIXTURE

# Generate the input file in plink format
plink --vcf /home/data/vcf/$FILE.vcf.gz --make-bed --out $FILE --allow-extra-chr

# ADMIXTURE does not accept chromosome names that are not human chromosomes. We will thus just exchange the first column by 0
awk '{$1=0;print $0}' $FILE.bim > $FILE.bim.tmp
mv $FILE.bim.tmp $FILE.bim

```
Now, we are ready to run `ADMIXTURE`. We will run it with cross-validation (the default is 5-fold CV, for higher, choose e.g. `cv=10`) and `K=2`.

```shell
admixture --cv $FILE.bed 2 > log2.out
```
 `ADMIXTURE` produced 2 files: `.Q` which contains cluster assignments for each individual and `.P` which contains for each SNP the population allele frequencies.

Let's now run it in a for loop with `K=2` to `K=5` and direct the output into log files

```shell
for i in {2..5}
do
 admixture --cv $FILE.bed $i > log${i}.out
done
```

To identify the best value of k clusters which is the value with lowest cross-validation error, we need to collect the cv errors. Below are three different ways to extract the number of `K` and the `CV` error for each corresponding `K`. Like we said at the start of the course, there are many ways to achieve the same thing in bioinformatics!

```shell
grep "CV" *out | awk '{print $3,$4}' | sed -e 's/(//;s/)//;s/://;s/K=//'  > $FILE.cv.error
grep "CV" *out | awk '{print $3,$4}' | cut -c 4,7-20 > $FILE.cv.error
awk '/CV/ {print $3,$4}' *out | cut -c 4,7-20 > $FILE.cv.error
```

To make plotting easier, we can make a file with the individual names in one column and the species names in the second column. As the species name is in the individual name, it is very easy to extract the species name from the individual name:

```shell
awk '{split($1,name,"."); print $1,name[2]}' ${FILE}.nosex > $FILE.list
```

Now we are ready to plot the results in `R`. To make it a bit easier, Joana Meier has written an `R` script for you that generates the plot. It requires four arguments, the prefix for the `ADMIXTURE` output files (-p <prefix>), the file with the species information (-i <file.list>), the maximum number of K to be plotted (-k 5), and a list with the populations or species separated by commas (-l <pop1,pop2...>). The list of populations provided with -l gives the order in which the populations or species shall be plotted.

You can get it from [here](https://github.com/speciationgenomics/scripts/blob/master/plotADMIXTURE.r) or download it with wget:

```shell
wget https://github.com/speciationgenomics/scripts/raw/master/plotADMIXTURE.r
chmod +x plotADMIXTURE.r
```

Now, let's run it like so:

```shell
Rscript plotADMIXTURE.r -p $FILE -i $FILE.list -k 5 -l PunNyerMak,PunPundMak,PunNyerPyt,PunHybrPyt,PunPundPyt
```

By default, the script generates a tiff file that uses the same prefix as the one provided with -p. In our case $FILE.tiff. This can be changed with -o <output prefix>.

Now we just need to download the `.tiff` file to our local machine to look at it, either using `scp` (Mac/Linux) or `Filezilla` (Windows).

Remember that ADMIXTURE/STRUCTURE plots can be quite misleading as different demographic histories can lead to the same results (see. e.g. [Lawson et al, 2018](https://www.nature.com/articles/s41467-018-05257-7)) These plots are great to detect recent hybrids but they are not ideal to infer complex demographic histories with hybrid origins of entire populations

To evaluate if the ADMIXTURE plot is a good fit, you can use [evaladmix](http://www.popgen.dk/software/index.php/EvalAdmix) and we highly recommend to also use other methods to help infer the demographic history and evidence of hybridisation such as Dstatistics, demographic modeling etc.
