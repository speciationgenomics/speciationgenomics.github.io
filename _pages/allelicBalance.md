---
title: "Checking for contamination, PCR errors etc."
layout: archive
permalink: /allelicBalance/
---

Heterozygote positions can be very useful to detect contamination or Illumina barcode switching issues. Without PCR duplicates, the reads at heterozygote positions should be completely independent and the two alleles should be represented with equal number of reads (except for stochastic differences). 

To check the heterozygotes, you can use the script written by David Marques and me and apply it on a dataset of SNPs. Let's try it on a few samples that were RAD-sequenced:

```shell
file=Pundamilia.RAD.subset
checkHetsIndvVCF.sh $file.vcf.gz
```

This script will generate a $file.hetIndStats.pdf that visualizes the allelic balance at heterozygote positions in a separte page for each individual. This file can then be used to detect contaminated indivdiuals or barcode switching problems.

If the dataset is RAD data, it is also possible that some heterozygote positions with highly unbalanced allele counts are in fact due to high levels of PCR duplication and PCR errors. If highly unbalance genotypes are due to contamination by another indivdiual in the dataset or due to Illumina barcode switching, singletons (i.e. positions in the genome with only a single indivdual carrying an alternative allele) are expected to show better allelic balance than positions with higher allele frequency in the dataset. This is because if the reads supporting the underreprented allele come from another individual, they should also be found in that indivual and thus not be singletons in the dataset. However, if the unbalanced heterozygotes are due to PCR errors in PCR duplicates, they should be most pronounced in singletons. This is because if PCR errors are randomly distributed across the genome, they are most likely to hit a monomorphic site because monomorphic sites are most frequent in the genome.

Therefore, it makes sense to run the script a second time, now only on singleton sites:

```shell
vcftools --gzvcf $file.vcf.gz --max-mac 1 --mac 1 --recode --stdout | gzip > $file.mac1.vcf.gz
checkHetsIndvVCF.sh $file.mac1.vcf.gz
```

