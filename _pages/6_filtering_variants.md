---
title: "Filtering and handling VCFs"
layout: archive
permalink: /filtering_vcfs/
---

In the last session, we learned how to call variants and handle VCFs. In this session, we are going to focus on how to filter VCFs. This might seem like a relatively straightforward task but it is actually exceptionally important and something you should spend a lot of time thinking carefully about.

### Getting access to the real data

At this point our 'toy' dataset breaks down as we do not have enough reads or coverage to call a decent variant set. Recall from the last steps of the last tutorial that most of genotpyes were missing.

To rememdy this, we are going to use a vcf we prepared on your behalf using the full set of reads. To get the full dataset into your directory, use the following commands:

```shell
# move to your vcf directory
cd ~/vcf
# copy the data file from the data directory
cp $~/data/vcf/cichlid_full.vcf.gz* ./
```

This will copy both the vcf and its index to your directory. Let's take a moment to see how big the file is:

```shell
ls -lh *.vcf.gz
```
You will see that this VCF is 1.5G in size, which is quite large but real analyses with many individuals can run to much larger sizes than this (for instance, we regularly work with VCF files containing hundreds of individuals which are >300 G).


### How many unfiltered variants?

One thing we didn't check yet is how many variants we actually have. Each line in the main output of a vcf represents a single call so we can use the following code to work it out:

```shell
bcftools view -H cichlid_full.vcf.gz | wc -l
```
This command will essentially print every line of the VCF, so if your file is large, it will take quite a long time to run this. For this reason, it might be best to run it inside a screen and then we can check it again later.

Either way, we have close to 8 million variants in our full VCF. That's a substantial number! But chances are many of them are not useful for our analysis because they occur in too few individuals, their minor allele frequency is too low or they are covered by insufficient depth.

At present, we have applied no filters at all. This is intentional - we want to see what happens when filters are applied. However, it is also a good idea to perform an initial analysis, to get an idea of how to set filters. However as we have just seen, it takes time to perform operations on a large VCF.

For this reason, it is a good idea to subsample our variant calls and get an idea of the general distribution of a few key attributes of the data.

### Randomly subsampling a VCF

How can we get an idea of how to set filters? We could just take the first 100 000 variants in our vcf and get an idea of how they look for some basic statistics (i.e. minor allele frequency, depth of coverage and so on). But would this be accurate? What if there is some bias on the first chromosome in the genome and everything there mapped well?

A far better idea is to randomly sample your VCF. Luckily there is a tool to do exactly this and it is part of the extremely useful [`vcflib` pipeline](https://github.com/vcflib/vcflib). Using it is also very simple. Here we will use it to extract ~100 000 variants at random from our unfiltered VCF.

```shell
bcftools view cichlid_full.vcf.gz | vcfrandomsample -r 0.012 > cichlid_subset.vcf.gz

```
Note that `vcfrandomsample` cannot handle an uncompressed VCF, so we first open the file using `bcftools` and then pipe it to the `vcfrandomsample` utility. We set only a single parameter, `-r` which is a bit confusingly named for the rate of sampling. This essentially means the fraction of variants we want to retain.

This will give us at least 95-100 K variants, depending on the random seed used to start the process. This means that everyone in the class should get different random subsets - providing us a nice demonstration for the next step. Before proceeding though, we should compress and index our new subset VCF to make it easier to access:

```shell
# compress vcf
bgzip cichlid_subset.vcf
# index vcf
bcftools index cichlid_subset.vcf.gz
```

### Generating statistics from a VCF

In order to generate statistics from our VCF and also actually later apply filters, we are going to use `vcftools`, [a very useful and fast program for handling vcf files](https://vcftools.github.io/examples.html).

Determining how to set filters on a dataset is a bit of a nightmare - it is something newcomers (and actually experienced people too) really struggle with. There also isn't always a clear answer - if you can justify your actions then there are often multiple solutions to how you set your filters. What is important is that you get to know your data and from that determine some sensible thresholds.

Luckily, `vcftools` makes it possible to easily calculate these statistics. In this section, we will analyse our VCF in order to get a sensible idea of how to set such filtering thresholds. The main areas we will consider are:

* **Depth:** You should always include a minimum depth filter and ideally also a maximum depth one too. Minimum depth cutoffs will remove false positive calls and will ensure higher quality calls too. A maximum cut off is important because regions with very, very high read depths are likely repetitive ones mapping to multiple parts of the genome.
* **Quality** Genotype quality is also an important filter - essentially you should not trust any genotype with a Phred score below 20 which suggests a less than 99% accuracy.
* **Minor allele frequency** MAF can cause big problems with SNP calls - and also inflate statistical estimates downstream. Ideally you want an idea of the distribution of your allelic frequencies but 0.05 to 0.10 is a reasonable cut-off. You should keep in mind however that some analyses, particularly demographic inference can be biased by MAF thresholds.
* **Missing data** How much missing data are you willing to tolerate? It will depend on the study but typically any site with >25% missing data should be dropped.

#### Setting up

Before we calculate our stats, lets make a little effort to make our commands simpler and also to ensure the output is written to the right place. First we need to make a directory for our results.

```shell
mkdir ~/vcftools
```
Next we will declare to variables to save us some typing below.

```shell
SUBSET_VCF=~/vcf/cichlid_subset.vcf.gz
OUT=~/vcftools/cichlid_subset

```

#### Calculate allele frequency

First we will calculate the allele frequency for each variant. The `--freq2` just outputs the frequencies without information about the alleles, `--freq` would return their identity.

```shell
vcftools --gzvcf SUBSET_VCF --freq2 --out $OUT
```

#### Calculate mean depth per individual

Next we calculate the mean depth of coverage per individual.

```shell
vcftools --gzvcf SUBSET_VCF --depth --out $OUT
```
#### Calculate mean depth per variant

Similarly, we also estimate the mean depth of coverage for each variant.

```shell
vcftools --gzvcf SUBSET_VCF --site-mean-depth --out $OUT
```
#### Calculate site quality

We additionaly extract the site quality score for each variant.

```shell
vcftools --gzvcf SUBSET_VCF --site-quality --out $OUT
```

#### Calculate proportion of missing data per individual

Another individual level statistic - we calculate the proportion of missing data per sample.

```shell
vcftools --gzvcf SUBSET_VCF --missing-indv --out $OUT
```

#### Calculate proportion of missing data per site

And more missing data, just this time per site rather than per individual.

```shell
vcftools --gzvcf SUBSET_VCF --missing-site --out $OUT
```
---

With the statistics calculated, take a moment to have a quick look at the output in the `~/vcftools/` directory. Go [here]() to learn more about plotting these variables in R and to understand where and how we should set cut-offs. With that, we can then turn to actually applying our filters to our VCF.

### Filtering the VCF

Now we have an idea of how to set out thresholds, we will do just that. Run the following `vcftools` command on the data to produce a filtered vcf. We will investigate the options after the filtering is run.

```shell
# move to the vcf directory
cd vcf
# perform the filtering with vcftools
vcftools --gzvcf cichlid_full.vcf.gz \
--remove-indels --maf 0.05 --max-missing 0.75 \
--minDP 10 --maxDP 200 --recode --stdout | gzip -c > \
cichlid_full_filtered.vcf.gz
```

What have we done here?

* `--gvcf` - input path -- denotes a gzipped vcf file
* `--remove-indels` - remove all indels (SNPs only)
* `--maf` - set minor allele frequency - 0.05 here
* `--max-missing` - set minimum missing data. A little counterintuitive - 0 is totally missing, 1 is none missing. Here 0.75 means we will tolerate 25% missing data.
* `--recode` - recode the output - necessary to output a vcf
* `--stdout` - pipe the vcf out to the stdout (easier for file handling)

Now, how many variants remain? There are two ways to examine this - look at the vcftools log or the same way we tried before.

```shell
cat out.log
bcftools view -H cichlid_full_filtered.vcf.gz | wc -l
```

You can see we have substantially filtered our dataset!

---
