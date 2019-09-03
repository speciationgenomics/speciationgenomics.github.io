---
title: "Variant calling"
layout: archive
permalink: /variant_calling/
---

If everything has worked correctly up to this point, we now have a set of sequence reads that are aligned to our reference genome and stored as bam files. What we want to do now is to call variants from these alignments.

**Disclaimer: There are many ways to perform variant calling for short read data - bcftools, GATK, FreeBayes, Stacks (RAD-seq)** We will be running through a single example today but it you should evaluate which tool will be best for your purposes with your own data.

For this tutorial, we will use `bcftools` which is designed by the same team behind `samtools` - [they are part of the same pipeline](http://www.htslib.org/). `bcftools` is itself a comprehensive pipeline and produces a variant call format (VCF) that is used in many downstream analyses.

### Indexing the reference... again

The first thing we need to do is index our reference genome again. This actually needs to be done with `samtools`. Return to the home directory and perform the following actions

```shell
cd ~/reference/
samtools faidx P_nyererei_v2.fasta
```

This will create a fasta index, denoted by the `.fai` suffix. Let's take a closer look at that index.

```shell
head P_nyererei_v2.fasta.fai
```

You don't need to worry about this in great deal but for clarity, the columns are: chromsome name, chromosome length, offset of the first base, fasta line length, fasta line length + 1.

### Calling variants

To call variants, we will first use the `samtools mpileup` tool to pileup our BAM files. What does this mean exactly? Well we will take all reads at a given position and call variants from the reads covering that position. We need to do this for all individuals.

After performing the pileup, we than pass the output to `bcftools call` which will actually call variants. To see the options available to each part of the pipeline, just type their names into the command line.

Rather than perform each step previously as we did before, here we will use pipes - `|` - to pass data from each part of the pipeline to the next. Rather than explaining every point now, it's best we just perform the calling and break it down later.

First of all, use `screen` to make a screen called variant and move into it.

Inside the screen, declare a variable for the reference genome.

```shell
REF=~/reference/P_nyererei_v2.fasta
```

Next we run the `bcftools mpileup` and `bcftools call` command. We will break it down after.

```shell
cd ~
bcftools mpileup -a AD,DP,SP -Ou -f $REF \
./align/*_sort.bam | bcftools call -f GQ,GP \
-mO z -o ./cichlid.vcf.gz
```

While this is running, let's go through the options and get an idea of what we did.

For `bcftools mpileup`:

* `-a` - Annotate the vcf - here we add allelic depth (`AD`), genotype depth (`DP`) and strand bias (`SP`).
* `-O` - the output type. Here it is `u` which means we do not compress the output.
* `-f` - specify the reference genome to call variants against.

For `bcftools call`:

* `-f` - format fields for the vcf - here they are genotype quality (`GQ`) and genotype probability (`GP`).
* `-v` - output variant sites only - i.e. ignore non-variant parts of the reads
* `-m`- use bcftools multiallelic caller
* `-O`- specify the output type, here it is `z` - i.e. gzipped (compressed) vcf
* `-o` output path

In essence, we have piled up each read against each genome position and then called variants where there are enough alternative reads to support the presence of a variant.

If the command is still running (it takes some time), detach your screen using `ctrl + a + d`. We will return to our VCF later.

#### Exploring vcf files

First, a bit of housekeeping. Make a vcf directory and move the vcf into it.

```shell
mkdir vcf
mv cichlid.vcf.gz ./vcf
cd vcf
```

vcf stands for 'variant call format' and is a standard format used for variant calling and in population genomics. Again a detailed specification can be found [online](https://samtools.github.io/hts-specs/VCFv4.2.pdf). It can take a bit of getting used to but it is widely supported and very useful. Let's have a look at the vcf.

```shell
bcftools view -h cichlid.vcf.gz
```

A lot of information will flash by - this is the vcf header. Like the SAM file, the header contains information on what has been done to the vcf. The last line is particularly important as it shows what each field in the main body of the vcf is and it also gives the individual names. Try the following:

```shell
bcftools view -h cichlid.vcf.gz | head
bcftools view -h cichlid.vcf.gz | tail
```

Here `-h` means show me only the header. You can also see `-H` to see only the raw calls. For some more information on the vcf format, see here.

A nice feature of vcf files is that you can access almost any part of the genome you are interested in. To do this though, you need to index the vcf first. Simply do the following

```shell
bcftools index cichlid.vcf.gz
```

Let's see what variants are present at the start of chr2:

```shell
bcftools view -H -r chr2 cichlid.vcf.gz | head
```

Here the `-r` flag specfies the region of the genome to examine. We can achieve the same effect by specifying the base pair coordinates

```shell
bcftools view -H -r chr2:100000-200000 cichlid.vcf.gz
```
Now you're probably wondering what exactly all this data actually means. Let's explore in a bit more detail.

```shell
bcftools view -h cichlid.vcf.gz | tail -1 | cut -f 1-9
bcftools view -H cichlid.vcf.gz | head -1 | cut -f 1-9
```

These are the first 11 fields of the vcf and they are always present. What do they mean?

* `CHROM` - chromosome or scaffold id from the reference genome
* `POS` - base pair reference position
* `ID` - SNP id - blank in this case
* `REF`- Reference base - A,C,G,T or N. More than one indicates an indel
* `ALT` - Alternaate base - the alternate base called as a variant
* `QUAL` - Phred quality score for alternate base call (site not individual)
* `FILTER` - filter status - if PASS then all filters passed
* `INFO` - additional info on each site - explanation stored in header
* `FORMAT`- the format for the genotype fields

There is a lot of information here! Don't worry too much if it doesn't make perfect sense immediately - familiarity with this format will come with time and experience.

Let's look at the the first site again in detail:

```shell
bcftools view cichlid.vcf.gz | grep -m 1 -A 1 "#CHROM" | cut -f 1-7
```

You should see this:

```shell
#CHROM  POS     ID      REF     ALT     QUAL    FILTER
chr1    1046    .       C       T       19.9754 .
```

Which means we have a variant at 1046 on chr1. The reference base is C, alternate base is T. There are no filters.

Have a look at the info field with the following code

```shell
bcftools view -H cichlid.vcf.gz | head -1 | cut -f 8
```
You can find out what these mean by grepping the header.

```shell
bcftools view -h cichlid.vcf.gz | grep "INFO"
```

So for example, DP here means the raw read depth for the entire site.

What about the actual genotype information? Firstly it is wise to look at the format here.

```shell
bcftools view -H cichlid.vcf.gz | head -1 | cut -f 9
```

To investigate what these mean, grep the header again.

```shell
bcftools view -h cichlid.vcf.gz | grep "##FORMAT"
```

Now let's take a look at the call for a single individual.

```shell
bcftools view -H cichlid.vcf.gz | head | cut -f 20
```

This will return:

```shell
1/1:41,3,0:1:0,1,0:65,12,0:12
```

First we have the genotype. Here `0` always denotes the reference, `1` the alternate base so we can see this individual is homozygous for the `ALT` base. We will ignore the genotype likelhood for now. You can see there is 1 read in total covering this site - which we are seeing because we used a reduced dataset.

Is there an easier way to view genotypes? Yes there is - using the `bcftools query` utility.

This allows you to look at the genotypes like so:

```
bcftools query -f '%CHROM\t%POS\t%REF\t%ALT[\t%GT]\n' cichlid.vcf.gz | head
```
You can also translate them into actual basecalls. Try this:

```
bcftools query -f '%CHROM\t%POS\t%REF\t%ALT[\t%TGT]\n' cichlid.vcf.gz | head
```
The `bcftools query` utility is very powerful and a useful tool to know about for file conversion in your own work.

Looking at the genotypes, you will see they are mostly missing. One of the reasons for this is the fact we used such a tiny dataset. So in the next section we will look at a proper set of variant calls from multiple individuals.

---
