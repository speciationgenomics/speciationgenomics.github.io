---
title: "Estimating the site-frequency-spectrum"
layout: archive
permalink: /easysfs/
---

In order to perform demographic analyses with programs such as `fastsimcoal2` or `dadi`,
you need to generate or estimate a **site-frequency spectrum** (SFS). There are multiple different ways
to do this but one of the simplest is using [easysfs](https://github.com/isaacovercast/easySFS), a really nice python based utility that is built on top of the `dadi` libraries.

Using `easySFS` is very straighforward. The first thing we will do is declare a `$VCF` variable, to make the downstream commands a bit more clear. We do this like so:

```shell
# As always, we work in a separate directory for this analysis
mkdir SFS
cd SFS

# We will work with a subset of sites that are already filtered for low missing data proportions:
cp /home/data/vcf/cichlid_subset_filtered.vcf.gz ./
VCF="cichlid_subset_filtered.vcf.gz"

```

Once we have done that, we need to create a population file. For the demonstration here, we will use only cichlids from Makobe. The following `bcftools` command, piped to `grep` and `awk` will produce what we need:

```shell
# make pops
bcftools query -l $VCF | grep "Mak" | awk '{split($0,a,"."); print $1,a[2]}' > pop_file
```

`easySFS` has a nice feature for estimating how to downsample the populations to retain the highest number of sites. The SFS cannot be computed with sites that contain missing data. By projecting down the number of individuals per population, more sites can be recovered. Alternatively, if your dataset has a lot of missing data and low coverage, you may want to generate the SFS with [ANGSD](http://www.popgen.dk/angsd/index.php/ANGSD) which takes genotype uncertainties into account. For easySFS, to identify the best projection value, you can use the following command:

```shell
# estimate projections
easySFS.py -i $VCF -p pop_file -a -f --preview
```

However in our case, we have very little missing data. There is no need to downsample. Therefore we run `easySFS` like so:

```shell
# this line will calculate the necessary SFS files
easySFS.py -i $VCF -p pop_file -a -f --proj 8,8
```

This will then produce an SFS in formats suitable for `fastsimcoal2` and `dadi`. You're now ready to go to the next step of actually [using this for demographic inference!](https://speciationgenomics.github.io/fastsimcoal2/)
