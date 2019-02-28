---
title: "Estimating the site-frequency-spectrum"
layout: archive
permalink: /easysfs/
---

In order to perform demographic analyses with programs such as `fastsimcoal2` or `dadi`,
you need to generate or estimate a **site-frequency spectrum**. There are multiple different ways
to do this but one of the simplest is using [easysfs](https://github.com/isaacovercast/easySFS), a really nice python based utility that is built on top of the `dadi` libraries.

Using `easySFS` is very straighforward. The first thing we will do is declare a `$VCF` variable, to make the downstream commands a bit more clear. We do this like so:

```shell
VCF=cichlid_filtered.vcf.gz
```

Once we have done that, we need to create a population file. For the demonstration here, we will use only cichlids from Makobe. The following `bcftools` command, piped to `grep` and `awk` will produce what we need:

```shell
# make pops
bcftools query -l $VCF | grep "Mak" | awk '{split($0,a,"."); print $1,a[2]}' > pop_file
```

`easySFS` has a nice feature for estimating how to project the populations. In some cases, you might have unbalanced samples (i.e. 20 individuals of species A, 15 of species B) and so it might be worth limiting the number of individuals included to ensure that you get the maximum number of segregating sites. To use this feature, you can use the following command:

```shell
# estimate projections
easySFS.py -i $VCF -p pop_file -a -f --preview
```

However in our case, we have 4 individuals of each species (and thus 8 chromosomes from each). There is no need to downsample or alter our projection. Therefore we run `easySFS` like so:

```shell
# this line will calculate the necessary SFS files
easySFS.py -i $VCF -p pop_file -a -f --proj 8,8
```

This will then produce an SFS in formats suitable for `fastsimcoal2` and `dadi`. You're now ready to go to the next step of actually [using this for demographic inference!](https://speciationgenomics.github.io/fastsimcoal2/)
