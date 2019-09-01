---
title: "REHH"
layout: archive
permalink: /REHH/
---

As another measure of selection, we will use tests that rely on extended haplotype lengths. During a selective sweep, a variant rises to high frequency so rapidly that linkage disequilibrium with neighbouring polymorphisms is not disrupted by recombination, giving rise to long haplotypes. In regions of low recombination, all haplotypes are expected to be longer than in regions of high recombination. Therefore, it is important to compare the haplotypes against other haplotypes at the same genomic region. This can either be within a population, whereby an ongoing sweep would lead to a single haplotype being very long compared to the other haplotypes, or between populations whereby the population that experienced a sweep has longer haplotypes than the population which was not affected by selection in that genomic region.

To compute detect regions with extended haplotype lengths, we will use the rehh R package. For more information see the [vignette](https://cran.r-project.org/web/packages/rehh/vignettes/rehh.html).

The rehh package can read lots of different formats, including phased vcf files. We will use the vcf file that was phased in the SHAPEIT2 tutorial.

### Running rehh
