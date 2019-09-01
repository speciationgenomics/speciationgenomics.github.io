---
title: "REHH"
layout: archive
permalink: /REHH/
---

As another measure of selection, we will use tests that rely on extended haplotype lengths. During a selective sweep, a variant rises to high frequency so rapidly that linkage disequilibrium with neighbouring polymorphisms is not disrupted by recombination, giving rise to long haplotypes. In regions of low recombination, all haplotypes are expected to be longer than in regions of high recombination. Therefore, it is important to compare the haplotypes against other haplotypes at the same genomic region. This can either be within a population, whereby an ongoing sweep would lead to a single haplotype being very long compared to the other haplotypes, or between populations whereby the population that experienced a sweep has longer haplotypes than the population which was not affected by selection in that genomic region.

To compute detect regions with extended haplotype lengths, we will use the rehh R package. For more information see the [vignette](https://cran.r-project.org/web/packages/rehh/vignettes/rehh.html).

Haplotypes are best assessed with phased dataset. Phasing basically means figuring out for heterozygous positions which of the alleles are part of the same haplotype or chromosome (e.g. the chromosome inherited from the mother). For instance if an individual is heterozygous at two SNPs with genotypes AG and TC, phasing would tell us if the allele A at SNP1 one was inherited on the same chromosome like T or like C at the second SNP. Phased genotypes would be given as A|G and C|T meaning that A and C are on the same chromosome (e.g. maternal) and G and T are on the same chromosome (e.g. paternal). In the absence of long reads that span these SNPs, we can use statistical phasing using all sequenced individuals of the same population (the more the better). There are lots of different tools for phasing and most of them also impute missing genotypes. This means that they infer missing genotypes statistically resulting in a dataset without missing data. If you have a linkage map, it is recommended to use it for making phasing more accurate. However, here we will just perform a very basic phasing with [SHAPEIT2](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#gettingstarted) which does not impute genotypes and does not require recombination rate information.

### Phasing with SHAPEIT2

´´´shell

´´´´

### Running rehh
