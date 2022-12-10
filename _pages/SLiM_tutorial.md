---
title: "Simulating data with SLiM"
layout: archive
permalink: /slim/
---

This tutorial was written by Henry North for the 2021 speciation genomics Physalia course.

SLiM is a software for forward-time simulation. You can read about SLiM and eidos, the language in which SLiM operates, in the [SLiM manual](http://benhaller.com/slim/SLiM_Manual.pdf). There is also a [detailed online tutorial](http://benhaller.com/workshops/workshops.html) on how SLiM works. Here the slides of a short lecture by Henry on [SLiM](https://github.com/speciationgenomics/data/blob/master/SLiM_slides.pdf).

Simulating data is a great way to explore what patterns we would expect in the genomic data under different evolutionary scenarios such as population expansions, a bottleneck or specific selection pressures. We can use neutral simulations to get null distributions for selection statistics, or we can use simulated data to test how well new analysis methods perform.

SLiM is a very userfriendly software. There is also a GUI (graphical user interface). SLiM outputs the simulated data as a vcf file and we can thus apply the same methods (e.g. calculating FST) as for real data.

Here are four very simple SLiM simulations to aid our intuition on the effect of demography on population genomic summary statistics:

First, a model of a stable population of 1000 diploids over 5000 generations. SLiM has a neat function to output .vcf files from a subsample of individuals from the simulation. We can then analyse the output using the same tools that we would use for empirical data.  

*neutral_stable.slim*

```shell
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
1 early()
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

Now we create a model identical to the one above, except that the population expands from 1000 to 7500 at generation 4000:

*neutral_expansion.slim*

```shell
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
1 early()
{
	sim.addSubpop("p1", 1000);
}
// at generation 4000 increase the population size to 7500 immediately

4000 early() { p1.setSubpopulationSize(7500); }

// run to generation 5000
5000 late() {

// randomly subsample 20 individuals for output
allIndividuals = sim.subpopulations.individuals;
 sampledIndividuals = sample(allIndividuals, 20);
 sampledIndividuals.genomes.outputVCF();

}
```

Now a population contraction (from 1000 to 250) at generation 2000:

*neutral_contraction.slim*

```shell
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
1 early()
{
	sim.addSubpop("p1", 1000);
}
// at generation 2000 decrease the population size to 250 immediately

2000 early() { p1.setSubpopulationSize(250); }

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
```shell
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
1 early()
{
	sim.addSubpop("p1", 1000);
}
// at generation 2000 decrease the population size to 250 immediately

4500 early() { p1.setSubpopulationSize(250); }

// run to generation 5000
5000 late() {

// randomly subsample 20 individuals for output
allIndividuals = sim.subpopulations.individuals;
 sampledIndividuals = sample(allIndividuals, 20);
 sampledIndividuals.genomes.outputVCF();

}
```


How do we implement these models? Here is a walk-through of how to run these on the cluster and format the output:

```shell

# Make and go into a working directory called SLiM
cd ~
mkdir SLiM
cd SLiM/

# Get the shell scripts and slim files
cp /home/scripts/SLiM/*.sh .
cp /home/scripts/SLiM/*.slim .

# Let's check the version of slim
slim -version

# Have a look at the slim file
cat neutral_stable.slim

# Running slim is very easy, just specify the slim file
slim neutral_stable.slim > stable.out

# Familiarise yourself with the output file
head stable.out

# The last lines contain the genotype information, make a vcf file of those lines
tail -n+15 stable.out | bgzip -c > stable.vcf.gz

rm stable.out

# Compute FST with vcftools
vcftools --gzvcf stable.vcf.gz --weir-fst-pop pop1.txt --weir-fst-pop pop2.txt --out stable_fst

# have a look at the pop text files that are used to assign individuals to populations
cat pop1.txt
cat pop2.txt

# compute pi (nucleotide diversity) with vcftools
vcftools --gzvcf stable.vcf.gz --site-pi --out stable_pi

# calculate the mean and remove the intermediate file
awk 'NR > 1 {sum+=$3} END {print sum / (NR - 1)}' stable_pi.sites.pi > stable_pi_mean.txt
rm stable_pi.sites.pi

# Compute Tajima's D with vcftools
vcftools --gzvcf stable.vcf.gz --out stable --TajimaD 10000

cat stable.Tajima.D | column -t

awk 'NR==2' stable.Tajima.D | cut -f 4 > stable_tajD.txt
rm stable.Tajima.D

```

This ran one simulation, but we are interested in running many iterations of the same model to generate a null distribution.

Below we implement many iterations of each model and process output in a loop:

run_stable_popsize.sh
```shell
for (( i = 1; i < 101; i++ )); do

  echo "running simulation ${i}"

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

# As above for pi and Tajima's D
cat stable_pi_mean_*.txt >> all_pi_stable.txt
rm stable_pi_mean_*.txt

cat stable_tajD_*.txt >> all_tajD_stable.txt
rm stable_tajD_*.txt
```

You can find all scripts for running the other models on the Speciation Genomics GitHub website:
[run_stable_popsize.sh](https://github.com/speciationgenomics/scripts/blob/master/SLiM_run_stable_popsize.sh)
[run_population_expansion.sh](https://github.com/speciationgenomics/scripts/blob/master/SLiM_neutral_expansion_loop.sh)
[run_population_contraction.sh](https://github.com/speciationgenomics/scripts/blob/master/SLiM_run_population_contraction.sh)
[run_recent_population_contraction.sh](https://github.com/speciationgenomics/scripts/blob/master/SLiM_run_recent_population_contraction.sh)

Visualise the output in R:

plotting.R
```shell

# Collate Tajima's D data
tajD_stable <- read.csv("all_tajD_stable.txt")
colnames(tajD_stable) <- c("tajD_stable")
tajD_expansion <- read.csv("all_tajD_expansion.txt")
colnames(tajD_expansion) <- c("tajD_expansion")
tajD_contraction <- read.csv("all_tajD_contraction.txt")
colnames(tajD_contraction) <- c("tajD_contraction")
tajD_recentContraction <- read.csv("all_tajD_recentContraction.txt")
colnames(tajD_recentContraction) <- c("tajD_recentContraction")
tajD <- cbind(tajD_expansion, tajD_stable, tajD_contraction, tajD_recentContraction)

# Get means
mean(tajD$tajD_expansion)
mean(tajD$tajD_stable)
mean(tajD$tajD_contraction)
mean(tajD$tajD_recentContraction)

# Make boxplots of all models side-by-side
boxplot(tajD)

# Note: Changes in demography alter the mean and variance of Tajima's D.

# Collate fst data
fst_stable <- read.csv("all_fst_stable.txt")
colnames(fst_stable) <- c("fst_stable")
fst_expansion <- read.csv("all_fst_expansion.txt")
colnames(fst_expansion) <- c("fst_expansion")
fst_contraction <- read.csv("all_fst_contraction.txt")
colnames(fst_contraction) <- c("fst_contraction")
fst_recentContraction <- read.csv("all_fst_recentContraction.txt")
colnames(fst_recentContraction) <- c("fst_recentContraction")
fst <- cbind(fst_expansion, fst_stable, fst_contraction, fst_recentContraction)

# Compute means for each model
mean(fst$fst_expansion)
mean(fst$fst_stable)
mean(fst$fst_contraction)
mean(fst$fst_recentContraction)

# Plot the FST distributions for all models side-by-side
boxplot(fst)

# Note: Population expansion and contractions, do not strongly affect Fst.

# Collate pi data
pi_stable <- read.csv("all_pi_stable.txt")
colnames(pi_stable) <- c("pi_stable")
pi_expansion <- read.csv("all_pi_expansion.txt")
colnames(pi_expansion) <- c("pi_expansion")
pi_contraction <- read.csv("all_pi_contraction.txt")
colnames(pi_contraction) <- c("pi_contraction")
pi_recentContraction <- read.csv("all_pi_recentContraction.txt")
colnames(pi_recentContraction) <- c("pi_recentContraction")
pi <- cbind(pi_expansion, pi_stable, pi_contraction, pi_recentContraction)

# Compute means for each model
mean(pi$pi_expansion)
mean(pi$pi_stable)
mean(pi$pi_contraction)
mean(pi$pi_recentContraction)

# Plot boxplots of all models side-by-side
boxplot(pi)

# Note: Changes in demography alter the mean and variance of nucleotide diversity.

```
