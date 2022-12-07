---
title: "Simulating data with SLiM"
layout: archive
permalink: /slim/
---

This tutorial was written by Henry North for the 2021 course.

SLiM is software for forward-time simulation. You can read about SLiM and eidos, the language that SLiM operates in, in the [SLiM manual](http://rmarkdown.rstudio.com). There is also a [detailed online tutorial](http://benhaller.com/workshops/workshops.html) on how SLiM works. Here a short lecture on [SLiM](https://github.com/speciationgenomics/data/blob/master/SLiM_slides.pdf).



Here are four very simple models SLiM models.

First, a model of a stable population of 1000 diploids over 5000 generations. SLiM has a neat function to output .vcf files from a subsample of individuals from the simulation. We can then analyse the output using the same tools that we would use for empirical data.  

*neutral_stable.slim*
```{eidos, eval=F}
// Constant popiulation size of 1000 with no selection
initialize()
{
	// set the overall mutation rate
	initializeMutationRate(1e-5);

	// m1 mutation type: neutral
	initializeMutationType("m1", 0.5, "f", 0.0);

	// g1 genomic element type: uses m1 for all mutations
	initializeGenomicElementType("g1", m1, 1.0);

	// Chromosome of length 10 kb, homogenous (all of type g1)
	initializeGenomicElement(g1, 0, 9999);

	// uniform recombination along the chromosome
	initializeRecombinationRate(1e-8);
}

// create a population of 1000 individuals
1
{
	sim.addSubpop("p1", 1000);
}

// run to generation 5000
5000 late() {

// randomly subsample 20 individuals for output
allIndividuals = sim.subpopulations.individuals;
 sampledIndividuals = sample(allIndividuals, 20);
 sampledIndividuals.genomes.outputVCF();

}

```

The above model, expcept that the population expands from 1000 to 7500 at generation 4000:

*neutral_expansion.slim*
```{eidos, eval=F}
// Instantaneous population expansion from 1000 to 7500 at generation 4000 with no selection
initialize()
{
	// set the overall mutation rate
	initializeMutationRate(1e-5);

	// m1 mutation type: neutral
	initializeMutationType("m1", 0.5, "f", 0.0);

	// g1 genomic element type: uses m1 for all mutations
	initializeGenomicElementType("g1", m1, 1.0);

	// Chromosome of length 10 kb, homogenous (all of type g1)
	initializeGenomicElement(g1, 0, 9999);

	// uniform recombination along the chromosome
	initializeRecombinationRate(1e-8);
}

// create a population of 1000 individuals
1
{
	sim.addSubpop("p1", 1000);
}
// at generation 4000 increase the population size to 7500 immediately

4000 { p1.setSubpopulationSize(7500); }

// run to generation 5000
5000 late() {

// randomly subsample 20 individuals for output
allIndividuals = sim.subpopulations.individuals;
 sampledIndividuals = sample(allIndividuals, 20);
 sampledIndividuals.genomes.outputVCF();

}
```

A population crontraction (from 1000 to 250) at generation 2000:

*neutral_contraction.slim*
```{eidos, eval=F}
// Instantaneous population contraction from 1000 to 250 at genneration 2000
initialize()
{
	// set the overall mutation rate
	initializeMutationRate(1e-5);

	// m1 mutation type: neutral
	initializeMutationType("m1", 0.5, "f", 0.0);

	// g1 genomic element type: uses m1 for all mutations
	initializeGenomicElementType("g1", m1, 1.0);

	// Chromosome of length 10 kb, homogenous (all of type g1)
	initializeGenomicElement(g1, 0, 9999);

	// uniform recombination along the chromosome
	initializeRecombinationRate(1e-8);
}

// create a population of 1000 individuals
1
{
	sim.addSubpop("p1", 1000);
}
// at generation 2000 decrease the population size to 250 immediately

2000 { p1.setSubpopulationSize(250); }

// run to generation 5000
5000 late() {

// randomly subsample 20 individuals for output
allIndividuals = sim.subpopulations.individuals;
 sampledIndividuals = sample(allIndividuals, 20);
 sampledIndividuals.genomes.outputVCF();

}
```

Finally, the same population contraction but closer to the end of the simulation (at generation 4500):
*neutral_recentContraction.slim*
```{eidos, eval=F}
// Instantaneous population contraction from 1000 to 250 at genneration 4500
initialize()
{
	// set the overall mutation rate
	initializeMutationRate(1e-5);

	// m1 mutation type: neutral
	initializeMutationType("m1", 0.5, "f", 0.0);

	// g1 genomic element type: uses m1 for all mutations
	initializeGenomicElementType("g1", m1, 1.0);

	// Chromosome of length 10 kb, homogenous (all of type g1)
	initializeGenomicElement(g1, 0, 9999);

	// uniform recombination along the chromosome
	initializeRecombinationRate(1e-8);
}

// create a population of 1000 individuals
1
{
	sim.addSubpop("p1", 1000);
}
// at generation 2000 decrease the population size to 250 immediately

4500 { p1.setSubpopulationSize(250); }

// run to generation 5000
5000 late() {

// randomly subsample 20 individuals for output
allIndividuals = sim.subpopulations.individuals;
 sampledIndividuals = sample(allIndividuals, 20);
 sampledIndividuals.genomes.outputVCF();

}
```


How do we implement these models? Here is a step-by-step walkthrough of how to run these on the cluster and format the output:

```{bash,eval=F}
cd ~

mkdir SLiM

cd SLiM/

cp /home/scripts/SLiM/*.sh .
cp /home/scripts/SLiM/*.slim .

slim -version

cat neutral_stable.slim

slim neutral_stable.slim > stable.out

head stable.out

tail -n+15 stable.out | bgzip -c > stable.vcf.gz

rm stable.out

vcftools --gzvcf stable.vcf.gz --weir-fst-pop pop1.txt --weir-fst-pop pop2.txt --out stable_fst

cat pop1.txt
cat pop2.txt

vcftools --gzvcf stable.vcf.gz --site-pi --out stable_pi

# calculate the mean and remove the intermediate file
awk 'NR > 1 {sum+=$3} END {print sum / (NR - 1)}' stable_pi.sites.pi > stable_pi_mean.txt
# rm stable_pi.sites.pi


vcftools --gzvcf stable.vcf.gz --out stable --TajimaD 10000

cat stable.Tajima.D | column -t
# rm stable.log

awk 'NR==2' stable.Tajima.D | cut -f 4 > stable_tajD.txt
# rm stable.Tajima.D

```

This ran one simulation, but we are more interested in running many iterations of the same model to generate a null distribution.

Below we implement many iterations of each model and process output in a loop:

run_stable_popsize.sh
```{bash, eval=F}
for (( i = 1; i < 101; i++ )); do

  echo "running simulation ${i} of 100"

  ### 1: Run the simulation
  slim neutral_stable.slim > stable_${i}.out # run the simulation
  tail -n+15 stable_${i}.out | bgzip -c > stable_${i}.vcf.gz # zip and tidy up the vcf output
  rm stable_${i}.out # remove a temporary file (raw SLIM output)

  ### 2: Calculate summary statistics

  ## Calculate Fst from the VCF

  vcftools --gzvcf stable_${i}.vcf.gz --weir-fst-pop pop1.txt --weir-fst-pop pop2.txt --out stable_fst_${i} # produces a log file
  grep "weighted" stable_fst_${i}.log | awk '{print $7}' > fst_stable_${i}.txt # Extract the fst value from the vcftools log file
  rm stable_fst_${i}.weir.fst # remove a temporary file (genome-wide FST; we don't need this)
  rm stable_fst_${i}.log # remove a temporary file (the vcftools log file)


  ## Calculate nucleotide diversity from the VCF

  vcftools --gzvcf stable_${i}.vcf.gz --site-pi --out stable_pi_${i}
  awk 'NR > 1 {sum+=$3} END {print sum / (NR - 1)}' stable_pi_${i}.sites.pi > stable_pi_mean_${i}.txt  # calculate the mean
  rm stable_pi_${i}.log # remove intermediate logfile
  rm stable_pi_${i}.sites.pi # Remove the intermediate file

  ## Calculate Tajima's D from the VCF
  vcftools --gzvcf stable_${i}.vcf.gz --out stable_${i} --TajimaD 10000
  rm stable_${i}.log # remove the intermediate logfile
  awk 'NR==2' stable_${i}.Tajima.D | cut -f 4 > stable_tajD_${i}.txt # extract out the value of D
  rm stable_${i}.Tajima.D # remove the intermediate file

  rm stable_${i}.vcf.gz # remove a temporary file (vcf; we don't need this any more)

done


### Collate Fst data
cat fst_stable_*.txt >> all_fst_stable.txt # Put all 100 fst values into one text output
rm fst_stable_*.txt # remove the individual fst text files

# As above for pi and D
cat stable_pi_mean_*.txt >> all_pi_stable.txt
rm stable_pi_mean_*.txt

cat stable_tajD_*.txt >> all_tajD_stable.txt
rm stable_tajD_*.txt
```

run_population_expansion.sh
```{bash, eval=F}
for (( i = 1; i < 101; i++ )); do

  echo "running simulation ${i} of 100"

  ### 1: Run the simulation
  slim neutral_expansion.slim > expansion_${i}.out # run the simulation
  tail -n+15 expansion_${i}.out | bgzip -c > expansion_${i}.vcf.gz # zip and tidy up the vcf output
  rm expansion_${i}.out # remove a temporary file (raw SLIM output)

  ### 2: Calculate summary statistics

  ## Calculate Fst from the VCF

  vcftools --gzvcf expansion_${i}.vcf.gz --weir-fst-pop pop1.txt --weir-fst-pop pop2.txt --out expansion_fst_${i} # produces a log file
  grep "weighted" expansion_fst_${i}.log | awk '{print $7}' > fst_expansion_${i}.txt # Extract the fst value from the vcftools log file
  rm expansion_fst_${i}.weir.fst # remove a temporary file (genome-wide FST; we don't need this)
  rm expansion_fst_${i}.log # remove a temporary file (the vcftools log file)


  ## Calculate nucleotide diversity from the VCF

  vcftools --gzvcf expansion_${i}.vcf.gz --site-pi --out expansion_pi_${i}
  awk 'NR > 1 {sum+=$3} END {print sum / (NR - 1)}' expansion_pi_${i}.sites.pi > expansion_pi_mean_${i}.txt  # calculate the mean
  rm expansion_pi_${i}.log # remove intermediate logfile
  rm expansion_pi_${i}.sites.pi # Remove the intermediate file

  ## Calculate Tajima's D from the VCF
  vcftools --gzvcf expansion_${i}.vcf.gz --out expansion_${i} --TajimaD 10000
  rm expansion_${i}.log # remove the intermediate logfile
  awk 'NR==2' expansion_${i}.Tajima.D | cut -f 4 > expansion_tajD_${i}.txt # extract out the value of D
  rm expansion_${i}.Tajima.D # remove the intermediate file

  rm expansion_${i}.vcf.gz # remove a temporary file (vcf; we don't need this any more)

done


### Collate Fst data
cat fst_expansion_*.txt >> all_fst_expansion.txt # Put all 100 fst values into one text output
rm fst_expansion_*.txt # remove the individual fst text files

# As above for pi and D
cat expansion_pi_mean_*.txt >> all_pi_expansion.txt
rm expansion_pi_mean_*.txt

cat expansion_tajD_*.txt >> all_tajD_expansion.txt
rm expansion_tajD_*.txt
```

run_population_contraction.sh
```{bash, eval=F}
for (( i = 1; i < 101; i++ )); do

  echo "running simulation ${i} of 100"

  ### 1: Run the simulation
  slim neutral_contraction.slim > contraction_${i}.out # run the simulation
  tail -n+15 contraction_${i}.out | bgzip -c > contraction_${i}.vcf.gz # zip and tidy up the vcf output
  rm contraction_${i}.out # remove a temporary file (raw SLIM output)

  ### 2: Calculate summary statistics

  ## Calculate Fst from the VCF

  vcftools --gzvcf contraction_${i}.vcf.gz --weir-fst-pop pop1.txt --weir-fst-pop pop2.txt --out contraction_fst_${i} # produces a log file
  grep "weighted" contraction_fst_${i}.log | awk '{print $7}' > fst_contraction_${i}.txt # Extract the fst value from the vcftools log file
  rm contraction_fst_${i}.weir.fst # remove a temporary file (genome-wide FST; we don't need this)
  rm contraction_fst_${i}.log # remove a temporary file (the vcftools log file)


  ## Calculate nucleotide diversity from the VCF

  vcftools --gzvcf contraction_${i}.vcf.gz --site-pi --out contraction_pi_${i}
  awk 'NR > 1 {sum+=$3} END {print sum / (NR - 1)}' contraction_pi_${i}.sites.pi > contraction_pi_mean_${i}.txt  # calculate the mean
  rm contraction_pi_${i}.log # remove intermediate logfile
  rm contraction_pi_${i}.sites.pi # Remove the intermediate file

  ## Calculate Tajima's D from the VCF
  vcftools --gzvcf contraction_${i}.vcf.gz --out contraction_${i} --TajimaD 10000
  rm contraction_${i}.log # remove the intermediate logfile
  awk 'NR==2' contraction_${i}.Tajima.D | cut -f 4 > contraction_tajD_${i}.txt # extract out the value of D
  rm contraction_${i}.Tajima.D # remove the intermediate file

  rm contraction_${i}.vcf.gz # remove a temporary file (vcf; we don't need this any more)

done


### Collate Fst data
cat fst_contraction_*.txt >> all_fst_contraction.txt # Put all 100 fst values into one text output
rm fst_contraction_*.txt # remove the individual fst text files

# As above for pi and D
cat contraction_pi_mean_*.txt >> all_pi_contraction.txt
rm contraction_pi_mean_*.txt

cat contraction_tajD_*.txt >> all_tajD_contraction.txt
rm contraction_tajD_*.txt
```

run_recent_population_contraction.sh
```{bash, eval=F}
for (( i = 1; i < 101; i++ )); do

  echo "running simulation ${i} of 100"

  ### 1: Run the simulation
  slim neutral_recentContraction.slim > recentContraction_${i}.out # run the simulation
  tail -n+15 recentContraction_${i}.out | bgzip -c > recentContraction_${i}.vcf.gz # zip and tidy up the vcf output
  rm recentContraction_${i}.out # remove a temporary file (raw SLIM output)

  ### 2: Calculate summary statistics

  ## Calculate Fst from the VCF

  vcftools --gzvcf recentContraction_${i}.vcf.gz --weir-fst-pop pop1.txt --weir-fst-pop pop2.txt --out recentContraction_fst_${i} # produces a log file
  grep "weighted" recentContraction_fst_${i}.log | awk '{print $7}' > fst_recentContraction_${i}.txt # Extract the fst value from the vcftools log file
  rm recentContraction_fst_${i}.weir.fst # remove a temporary file (genome-wide FST; we don't need this)
  rm recentContraction_fst_${i}.log # remove a temporary file (the vcftools log file)


  ## Calculate nucleotide diversity from the VCF

  vcftools --gzvcf recentContraction_${i}.vcf.gz --site-pi --out recentContraction_pi_${i}
  awk 'NR > 1 {sum+=$3} END {print sum / (NR - 1)}' recentContraction_pi_${i}.sites.pi > recentContraction_pi_mean_${i}.txt  # calculate the mean
  rm recentContraction_pi_${i}.log # remove intermediate logfile
  rm recentContraction_pi_${i}.sites.pi # Remove the intermediate file

  ## Calculate Tajima's D from the VCF
  vcftools --gzvcf recentContraction_${i}.vcf.gz --out recentContraction_${i} --TajimaD 10000
  rm recentContraction_${i}.log # remove the intermediate logfile
  awk 'NR==2' recentContraction_${i}.Tajima.D | cut -f 4 > recentContraction_tajD_${i}.txt # extract out the value of D
  rm recentContraction_${i}.Tajima.D # remove the intermediate file

  rm recentContraction_${i}.vcf.gz # remove a temporary file (vcf; we don't need this any more)

done


### Collate Fst data
cat fst_recentContraction_*.txt >> all_fst_recentContraction.txt # Put all 100 fst values into one text output
rm fst_recentContraction_*.txt # remove the individual fst text files

# As above for pi and D
cat recentContraction_pi_mean_*.txt >> all_pi_recentContraction.txt
rm recentContraction_pi_mean_*.txt

cat recentContraction_tajD_*.txt >> all_tajD_recentContraction.txt
rm recentContraction_tajD_*.txt
```


Visualise output in R:

```{r, eval=F}
# Collate TajD data
tajD_stable <- read.csv("SLIMV2/all_tajD_stable.txt")
colnames(tajD_stable) <- c("tajD_stable")
tajD_expansion <- read.csv("SLIMV2/all_tajD_expansion.txt")
colnames(tajD_expansion) <- c("tajD_expansion")
tajD_contraction <- read.csv("SLIMV2/all_tajD_contraction.txt")
colnames(tajD_contraction) <- c("tajD_contraction")
tajD_recentContraction <- read.csv("SLIMV2/all_tajD_recentContraction.txt")
colnames(tajD_recentContraction) <- c("tajD_recentContraction")
tajD <- cbind(tajD_expansion, tajD_stable, tajD_contraction, tajD_recentContraction)
mean(tajD$tajD_expansion)
mean(tajD$tajD_stable)
mean(tajD$tajD_contraction)
mean(tajD$tajD_recentContraction)
boxplot(tajD)

# Collate fst data
fst_stable <- read.csv("SLIMV2/all_fst_stable.txt")
colnames(fst_stable) <- c("fst_stable")
fst_expansion <- read.csv("SLIMV2/all_fst_expansion.txt")
colnames(fst_expansion) <- c("fst_expansion")
fst_contraction <- read.csv("SLIMV2/all_fst_contraction.txt")
colnames(fst_contraction) <- c("fst_contraction")
fst_recentContraction <- read.csv("SLIMV2/all_fst_recentContraction.txt")
colnames(fst_recentContraction) <- c("fst_recentContraction")
fst <- cbind(fst_expansion, fst_stable, fst_contraction, fst_recentContraction)
mean(fst$fst_expansion)
mean(fst$fst_stable)
mean(fst$fst_contraction)
mean(fst$fst_recentContraction)
boxplot(fst)

# Collate pi data
pi_stable <- read.csv("SLIMV2/all_pi_stable.txt")
colnames(pi_stable) <- c("pi_stable")
pi_expansion <- read.csv("SLIMV2/all_pi_expansion.txt")
colnames(pi_expansion) <- c("pi_expansion")
pi_contraction <- read.csv("SLIMV2/all_pi_contraction.txt")
colnames(pi_contraction) <- c("pi_contraction")
pi_recentContraction <- read.csv("SLIMV2/all_pi_recentContraction.txt")
colnames(pi_recentContraction) <- c("pi_recentContraction")
pi <- cbind(pi_expansion, pi_stable, pi_contraction, pi_recentContraction)
mean(pi$pi_expansion)
mean(pi$pi_stable)
mean(pi$pi_contraction)
mean(pi$pi_recentContraction)
boxplot(pi)
```
