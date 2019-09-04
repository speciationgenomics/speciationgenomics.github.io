---
title: "Sliding window Fst, dxy, fd"
layout: archive
permalink: /sliding_windows/
---

As SNP Fst values are very noisy, it is better to compute Fst estimates for entire regions. Selection is expected to not only affect a single SNP and the power to detect a selective sweep is thus higher for genomic regions. How large the genomic region should be depends on the SNP density, how fast linkage disequilibrium decays, how recent the sweep is and other factors. It is thus advisable to try different window sizes. Here, we will use 20 kb windows. An alternative is to use windows of fixed number of sites instead of fixed size. We will use scripts written by [Simon Martin](https://simonmartinlab.org/) which you can download [here](https://github.com/simonhmartin/genomics_general).

First, let's convert the vcf file into Simon Martin's geno file. You can download the script [here](https://github.com/simonhmartin/genomics_general/raw/master/VCF_processing/parseVCF.py).

```shell
cd ~/genome_scans
cp /home/data/vcf/Pundamilia.outgroups.chr20.vcf.gz ./
FILE="Pundamilia.outgroups.chr20"
parseVCF.py -i $FILE.vcf.gz -o $FILE.vcf.geno.gz
```

First, we will calculate pi for each species and Fst and dxy for each pair of species all in one go.
```shell
popgenWindows.py -w 20000 -m 10000 -s 20000 -g $INPUT -o $OUTPUT2 \
   -f phased -minData 0.5 \
   -p PundPyt 11725.PunPundPyt,11727.PunPundPyt,11728.PunPundPyt,11729.PunPundPyt \
   -p NyerPyt 11719.PunNyerPyt,11986.PunNyerPyt,11992.PunNyerPyt,11546.PunNyerPyt \
   -p NyerMak 11591.PunNyerMak,11593.PunNyerMak,11595.PunNyerMak,11598.PunNyerMak \
   -p PundMak 13069.PunPundMak,10558.PunPundMak,10560.PunPundMak,11297.PunPundMak \
   -p kivu 64253
```

Next, we calculate ABBABABA for introgression from the original Pundamilia nyererei (NyerMak)  into P. sp. "nyererei-like" (NyerPyt) using the Lake Kivu cichlid as outgroup:

```shell
ABBABABAwindows.py -w 20000 -m 10000 -s 20000 -g $INPUT -o $OUTPUT1 \
   -f phased --minData 0.5 --writeFailedWindows\
   -P1 PundPyt 11725.PunPundPyt,11727.PunPundPyt,11728.PunPundPyt,11729.PunPundPyt \
   -P2 NyerPyt 11719.PunNyerPyt,11986.PunNyerPyt,11992.PunNyerPyt,11546.PunNyerPyt \
   -P3 NyerMak 11591.PunNyerMak,11593.PunNyerMak,11595.PunNyerMak,11598.PunNyerMak \
   -O kivu 64253
```
