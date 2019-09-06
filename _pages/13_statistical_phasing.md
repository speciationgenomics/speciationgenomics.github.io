---
title: "Statistical phasing with shapeit2"
layout: archive
permalink: /phasing/
---

There are several applications in speciation genomics where we require **phased data**. For example for selection scans or some demographic analyses. The most accurate way to phase is to sequence (**parent-offspring trios**)[https://genome.cshlp.org/content/23/1/142.full.html]. However this is simply not possible for a large number of study systems. As an alternative, we can instead (statistically phase)[https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3217888/] using computational methods.

There are several different packages available for phasing such as (BEAGLE)[https://faculty.washington.edu/browning/beagle/beagle.html] and (fastPHASE)[http://scheet.org/software.html]. For this tutorial, we will use (shapeit2)[http://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html]. Note that there are more (recent versions of shapeit)[https://jmarchini.org/shapeit3/] available but these typically only offer significant improvements if you have very large sample sizes (i.e. 100s or 1000s of individuals).

Our aim in this tutorial is to phase whole genome resequencing data from house sparrows (*Passer domesticus domesticus*) and a wild subspecies, the Bactrianius sparrow (*Passer domesticus bactrianus*). The dataset is taken from (Ravinet *et al* (2018))[https://royalsocietypublishing.org/doi/full/10.1098/rspb.2018.1246]. We will ultimately use this phased data to perform a selection scan using extended haplotype statistics.

### Getting the data

Before we start phasing, we will create the directory we will be working in.

```shell
# move into home directory
cd ~
# make phasing directory
mkdir phasing
# move inside
cd phasing
```

Now we are ready to download and look at the data.

```shell
cp /home/data/sparrows/sparrows_chr8_unphased.vcf.gz* .
```

Next we will make an environmental variable to make our downstream analysis more straightforward.

```shell
VCF=~/phasing/sparrows_chr8_unphased.vcf.gz
```

Let's take a moment to see how many individuals and SNPs we have here.

```shell
bcftools query -l $VCF | wc -l
bcftools index -n $VCF
```

### Phasing the data

As we mentioned previously, `shapeit` is a tool for (statistically estimating haplotype phase from genotypes)[https://en.wikipedia.org/wiki/Haplotype_estimation]. Phasing is actually quite easy to do - what is much harder is knowing whether you did it correctly...

Before we run our phasing, let's create an output environmental variable using string manipulation. Here is a little reminder:

```shell
# remove the suffix
echo ${VCF%_u*}
# add the phased suffix
echo ${VCF%_u*}_phased
# now we're happy with this, assign a new variable
OUTPUT=${VCF%_u*}_phased
```

With this taken care of, we are ready to run `shapeit`. Note that here we are running the program only for a single chromosome - this is the most practical (and sensible) way to phase - it makes it much easier to paralellise.

We do this like so:

```shell
shapeit --input-vcf $VCF \
 -O $OUTPUT \
--window 0.5 -T 4
```

What did we do with this command?

* `--input-vcf` - this flag allows us to read in the vcf we want to phase.
* `--window` - sets the window size within which the phasing is carried out. The default is 1 Mb, here we set it to 0.5 Mb.
* `-T` - set the number of threads to use for parallelisation - we used 4 here

Next we run the analysis. It will take a few minutes and will print updates to the screen as it runs. Take a short break while it runs... and when it is done we can have a look at the results!

### Examining the phased data and converting it to a phased vcf

Once `shapeit` has finished running we can see it has produced two files, `sparrows_chr8_phased.haps` and `sparrows_chr8_phased.samples`. We can look at these in more detail:

```shell
head sparrows_chr8_phased.samples
```

The `sparrows_chr8_phased.samples` file is simply a list of samples with the ID for each sample repeated in two columns and a third column showing the proportion of missing data. Since there is no missing data in this dataset, this column is just full of zeros.

We can also look at the `.haps` output.

```shell
less sparrows_chr8_phased.haps
```

This is basically a matrix with the first three 5 columns identical to those in a vcf - i.e. chromosome, ID, position, reference allele, alternative allele. After this, each entry is the phased allele for each individual, where `0` is the reference allele and `1` is the alternative.

If we want to use this data downstream, particularly in the `R` package `rehh`, we need to convert it to a vcf. Luckily this is easy using `shapeit`:

```shell
shapeit -convert \
--input-haps ${OUTPUT} \
--output-vcf ${OUTPUT}.vcf
```

We can then compress and index the vcf, like so:

```shell
bgzip ${OUTPUT}.vcf
bcftools ${OUTPUT}.vcf.gz
```

Before moving on to the next step, let's have a quick look at our phased vcf to see how it is different from a standard vcf.

```shell
bcftools view -H ${VCF} | head | cut -f 1-12
bcftools view -H ${OUTPUT}.vcf.gz | head | cut -f 1-12
```

Comparing the two, we can see tht the phased vcf only contains the genotypes - all the other information has been stripped out. Furthermore, in the phased vcf, genotypes are encoded as `0|0`, `0|1` or `1|1` instead of `0/0`, `0/1` or `1/1`. This is because in a vcf, `|` is typically used to denote that the phase of these loci are known. Thus, it is possible to read the haplotype of an individual by reading downwards across loci on either side of the `|`.

### Subsetting the vcf for a selection scans

We will be using our phased vcf for long-range haplotype statistic estimation in `rehh`. While it is possible to split the populations in the vcf apart in R, it is a bit clumbsy to do so. instead, it is easier to split the vcf using `bcftools`. To do this, we first need the sample names

```shell
# set a new variable
VCF=~/phasing/sparrows_chr8_phased.vcf.gz
# look at the sample names
bcftools query -l $VCF
# extract sample names
bcftools query -l $VCF > samples
```

As we learned earlier, this vcf contains data from two house sparrow subspecies, the European house sparrow (from France and Norway) and the wild Bactrianius sparrow (from Kazahkstan and Iran). We split the population data like so:

```shell
grep "^[8F]" samples > house
grep -v "^[8F]" samples > bac
```

Note we used the `-v` flag to do an inverse `grep` - i.e. extract the opposite of this pattern. Next we split the vcf:

```shell
bcftools view -S house -O z -o house_chr8.vcf.gz $VCF
bcftools view -S bac -O z -o bac_chr8.vcf.gz
```

What did we do here? We used the `bcftools index` command to extract the samples for each population. The `-S` flag extracts the samples listed in each file. `-O z` specifies that we want a compressed vcf. Finally `-o` tells `bcftools` where to write the output. Now all we need to do is index the vcfs.

```shell
bcftools index house_chr8.vcf.gz
bcftools index bac_chr8.vcf.gz
```

Now we're ready to read this into `R` for a selection scan analysis.
