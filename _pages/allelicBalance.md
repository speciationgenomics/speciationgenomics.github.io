---
title: "Checking for contamination, PCR errors etc."
layout: archive
permalink: /allelicBalance/
---

Heterozygote positions can be very useful to detect contamination, high PCR error prevalence, Illumina index switching or other issues generating erroneous heterozygotes such as paralogous regions. Without PCR duplicates and other issues, the reads at any genotype should reflect independent copies of the maternal and paternally inherited genes. At heterozygote genotypes, the two alleles should thus be represented with equal number of reads (except for stochastic differences). However, if some of the reads represent PCR duplicates the variance in the number of reads supporting each allele (allelic balance). PCR errors present in multiple PCR duplicates can cause erroneous heterozygote genotypes. If the heterozygote genotype is due to such a PCR error or to contamination or Illumina index switching, the erroneous reads are expected to be rarer than the correct reads. Such heterozygotes should thus show a strong allelic disbalance, whereby the ratio of reads supporting each allele deviates from 50% even at high sequencing depth. If cross-contamination among samples due to lab errors or Illumina index switching is the main reason for heterozygotes with uneven read counts, we would expect singletons (i.e. sites where only a single individual carries an alternative allele) to show better allelic balance than positions with higher minor allele frequency across all samples. This is because the reads supporting the underreprented allele come from another individual should also be found in that individual and can thus not be singletons in the dataset. In contrast, if PCR errors present in multiple PCR duplicates are the main reason, singletons should show the strongest signal. If highly unbalanced genotypes are due to contamination by another individual in the dataset or due to Illumina barcode switching, singletons (i.e. positions in the genome with only a single individual carrying an alternative allele) are expected to show better allelic balance than positions with higher allele frequency in the dataset. This is because if the reads supporting the underreprented allele come from another individual, they should also be found in that individual and thus not be singletons in the dataset. However, if the unbalanced heterozygotes are due to PCR errors in PCR duplicates, they should be most pronounced in singletons. This is because if PCR errors are randomly distributed across the genome, they are most likely to hit a monomorphic site because monomorphic sites are most frequent in the genome.

To check the heterozygotes, you can use the script written by David Marques and me and apply it on a dataset of SNPs. Let's try it on a few samples that were RAD-sequenced:

```shell
FILE=Pundamilia.RAD.subset
checkHetsIndvVCF.sh $FILE.vcf.gz
```

This script will generate a $FILE.hetIndStats.pdf that visualizes the allelic balance at heterozygote positions in a separate page for each individual. This file can then be used to detect contaminated individuals or barcode switching problems.

To distinguish if the heterozygotes with strongly unbalanced read counts are due to cross-contamination or PCR errors in PCR duplicates, we will extracts singleton sites and run the script a second time on this subset:

```shell
vcftools --gzvcf $FILE.vcf.gz --max-mac 1 --mac 1 --recode --stdout | gzip > $FILE.mac1.vcf.gz
checkHetsIndvVCF.sh $FILE.mac1.vcf.gz
```
