---
title: "Computing per site Fst"
layout: archive
permalink: /per_site_Fst/
---

In the previous sessions, we investigated population structure and learned how to detect admixture and introgression between populations. Now we will return to our filtered, high-quality variant set stored in our VCF and learn how to calculate population differentiation.

### Mean genome-wide *F*<sub>ST</sub>

As a first step, we will calculate a mean genome-wide *F*<sub>ST</sub> value for our variant set. This is simple to do and there are many tools that can do it. One of the easiest to use is `vcftools` which we learned about when we filtered our variants in the first place.

For speed and simplicity here, we will perform our genome scan on only a single chromosome - **chr20**. Let's set an environmental variable to make it easier to actually run our analyses. We'll also create a directory called `genome_scan` in the process.

```shell
# move to home directory
cd ~
# make a genome scan directory
mkdir genome_scan
# make an environmental variable for the input vcf
VCF=~/vcf/chr20.vcf.gz
```

As we learned earlier, VCFs can store a lot of information, but one thing they do not contain is population information. They can store it indirectly - i.e. in the sample names - but not explicitly in the metadata. This means that in order to calculate pairwise *F*<sub>ST</sub>, we need to first create files that split the populations.

Luckily, we can achieve this quite easily using the `bcftools query` utility. This is actually an exceptionally useful tool and one we will return to again while performing genome scans.

One thing we can do with `bcftools query` is extract the sample names from a VCF. We do that like so:

```shell
# move into the genome scan directory
cd ~/genome_scan
# run bcftools query
bcftools query -l $VCF
```

Here the `-l` flag just tells the tool to print sample names. For this example, we are going to calculate *F*<sub>ST</sub> for the two Pundamilia species pair at Python. So with a few command line tools, we can easily produce the files we need:

```shell
# extract sample names for P.pundamilia
bcftools query -l $VCF | grep "PundPyt" > ppund
# extract sample names for P.pundamilia
bcftools query -l $VCF | grep "NyerPyt" > pnyer
```

All we did here was pipe the sample files to `grep` and then extracted those matching the two population names. We now have two population files that we can use for our  *F*<sub>ST</sub> analyses. We are ready to use `vcftools`

```shell
vcftools --gzvcf ${VCF} \
--weir-fst-pop ppund \
--weir-fst-pop pnyer \
--out ./ppund_pnyer
```

`vcftools` will run for a few moments and then print some information to the screen. It will also save this as a `.log` file. You can see from this it calculates the mean *F*<sub>ST</sub> across all the SNPs we provided on chromsome 20.

So what did we do here? Well we calculated Weir & Cockerham's *F*<sub>ST</sub> for each SNP and to do this, all we needed to do was provide the two sample files (separately!) using the `--weir-fst-pop` flag.

Now that we have created our per-snp *F*<sub>ST</sub> estimates, its a good time to have a look at the output using tools like `head` or `less`. Of course, you can only gain so much information by looking at a bunch of values like this - it makes much more sense to plot them. So for that we turn to `R`.

Download the file to your local machine (using `scp` or `filezilla`) and open up `RStudio`and we will start a new `R` script.

### Visualising per SNP *F*<sub>ST</sub>

We will load the `tidyverse` package, read in the data and then tweak it slightly.

```r
library(tidyverse)

# read in the file
fst <- read_tsv("./ppund_pnyer.weir.fst")

# capdown the headers
names(fst) <- tolower(fst)
names(fst)[grep("fst")] <- "fst"
```

OK so that last step was largely because of personal preference, but it is important to learn how you can manipulate your data in R!

With that, we can plot our data using `ggplot2`:

```shell
ggplot(fst, aes(pos, fst)) + geom_point()
```

This plot is extremely simple but you can see there is a lot of noise - that's because there are a lot of SNPs... and this is only for a single chromosome! As we will see in the next tutorial, it is often easier to perform such analyses on sliding windows across the genome, because then it is easier to see overall trends and patterns in the data. There is less of an issue with the stochasticity of *F*<sub>ST</sub> at neighbouring polymorphic positions.

However before we move on, we will perform a simple outlier analysis.

### Performing a simple outlier analysis

There are many, many methods for performing outlier tests. Some use coalescent simulations, others use Bayesian inference to estimate the posterior probability that a SNP is an outlier and therefore putatively under selection.

However, the simplest way to detect outliers is to use the empirical distribution of *F*<sub>ST</sub> to look for SNPS that exceed an arbitrary threshold of differentiation. This method has been used quite a lot in the literature but it is not without caveats (and more on those later). Typically the threshold is set at either the 95th or 99th percentile of the empirical data.

Once again, we can turn to `R` to do this quite easily. First, let's identify the thresholds from the distribution.

```r
# identify the 95% and 99% percentile
quantiles(fst$fst, c(0.975, 0.995), na.rm = T)
```

Hang on a moment, those values aren't 95 and 99! That's because we are performing a one-tailed test here, so we are not interested in the lower tail of the *F*<sub>ST</sub> distribution.

Now we know how to identify our threshold, let's identify which SNPs are outliers above the 95% threshold. This will also make it possible for us to plot them more easily later. To do this, we will use the `ifelse` function.

```r
# identify the 95% percentile
my_threshold <- quantiles(fst$fst, 0.975, na.rm = T)
# make an outlier column in the data.frame
fst <- fst %>% mutate(outlier = ifelse(fst > my_threshold, "outlier", "background"))
```

What did we do here? `ifelse` takes a logical argument first (i.e. is fst greater than the threshold we set). If this is the case (i.e. it is `TRUE`), it will print "outlier" but if not, it just prints "background".

Now we can see how many outlier SNPs we have versus the background:

```r
fst %>% group_by(outlier) %>% tally()
```

Finally, let's recreate our plot and this time colour the SNPs by their outlier status.

```shell
ggplot(fst, aes(pos, fst, colour = outlier)) + geom_point()
```

Now we can actually see the position of our outlier SNPs along the chromsome. Downstream, we might want to see what genes these occur close to. But for now, think a bit about how much we might trust this test...

### Some caveats to simple outlier tests

Identifying outliers from the empirical distribution is a valid and worthwhile means of exploring the data. However it should always be used with a few caveats in mind.

Firstly, the assumption is that the empirical distribution is normal - often it is wise to plot it to see if this is actually the case.

Secondly the main assumption is that the empiricial distribution approaches the neutral distribution which we would expect if there were no selection acting. This is obviously violated in nearly every use of the test. Since we are using an outlier test to identify SNPs putatively under selection, selection has to be occurring! Some methods use coalescent simulations conditioned on a known demographic history to overcome this.

Thirdly, selectio scans such as this have the highest power when selection is strong and the genetic architecture underlying a trait under a selection is simple (i.e. it is a single locus of major of effect). Their power is much lower when the genomic architecure of a trait is polygenic, when selection is weak or when selection has occurred on standing variation (i.e. soft sweeps).

Finally, and perhaps most importantly outlier tests with arbitrary thresholds can introduce a false dichotomy of positions being under selection or not. Just because a SNP falls under a certain threshold does not mean selection is not operating on it. For example, divergent traits are highly polygenic, selection might be occurring on lots of SNPs but the signal is too weak for us to detect with a method like this.
