---
title: "Comparative genomics with Orthofinder"
layout: archive
permalink: /orthofinder/
---
This tutorial is written by José Cerca for the 2022 course.

We will be using Orthofinder to identify genes in a region where there is differentiation on the genome. Specifically, I have identified this region at 38 - 38.5 Mb and we will get the genes inside and determine their functinons.

Files we will be using:
A gff file, with annotations and features (house_sparrow_chr1A.gff)
A protein file for the house sparrow, with aminoacid sequences (house_sparrow_proteins_reduced.faa)
A protein file for a closely related model-organism (chicken; downloaded from ensembl; Gallus_gallus.GRCg6a.pep.reduced.faa)


1 - We start by cleaning up the space and setting an working directory.

```
# We make a folder and move inside
cd ~
mkdir Orthofinder_exercise; cd Orthofinder_exercise

# We make a structure for us to work
mkdir 01_data 02_orthofinder_run
```

2 - Let's copy the data, and then clean it.
```
cd ~/Orthofinder_exercise/01_data
cp /home/data/orthofinder/* .
# Take a minute or two to get acquainted with the data.
# Specifically - what do the .faa files contain?
# What does the general feature format file (gff) contain?

# Now, for the 'cleaning'.
# Orthofinder will give you tons of results. When you run it with more than two species it often gets confusing - which gene comes from species A? Is this gene from species C or D?
# I have a solution. Add the species name to each gene :)

# Run this code, but think, what does it do?
sed -i "/^>/ s/>/>House_sparrow_/" house_sparrow_proteins_reduced.faa
sed -i "/^>/ s/>/>chicken_/" Gallus_gallus.GRCg6a.pep.reduced.faa

# Check the files using less to see what it does.
```

4 - Let's run OrthoFinder
```
# Ignore this next line
PATH="/home/ubuntu/bin:/home/ubuntu/.local/bin:/home/ubuntu/miniconda3/bin:/home/ubuntu/miniconda3/condabin:/home/ubuntu/miniconda3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/home/scripts"

# Loading OrthoFinder
orthofinder -h

# Run it on the 01_data folder
orthofinder -f . -t 2
echo " Go drink of tea or take a 2 min break "

# move the folder to the results
mv OrthoFinder ../02_orthofinder_run/

# Before we look at the results, I'd like you to complete the next step.
```

5 - We will find which house sparrow genes are in CHR1A between 38 - 38.5 MB.
```
# CHR1A Rombos vs West : 38 - 38.5 Mb
# Try to understand the code and what it does.
awk '$1=="chr1A"' house_sparrow_chr1A.gff | awk '$4>38000000 && $5<38500000' | awk '$3=="gene"' | sed "s/.*ID=//; s/\-.*//; s/;.*//" | sort -u > genes_chr1A_38Mb_38.5Mb.tsv

# Result
IV00_00019202
IV00_00019209

# Are you confused what the previous code does? Remember, bioinformatics takes time Let's go step - by step:
awk '$1=="chr1A"' house_sparrow_chr1A.gff # here, we say ' hey unix, column one must be chr1A - I'm only interested in that one.
awk '$1=="chr1A"' house_sparrow_chr1A.gff | awk '$4>38000000 && $5<38500000' # Now, we pipe it and ask for ' column 4 needs to be bigger than 38 Mb, and column 5 smaller than 38.5 Mb. Column 4 is the ' start positino, whereas column 5 is the stop position.
awk '$1=="chr1A"' house_sparrow_chr1A.gff | awk '$4>38000000 && $5<38500000' | awk '$3=="gene"' # Now we specify that column 3 must say ' gene ' (so we exclude TEs and exons).
awk '$1=="chr1A"' house_sparrow_chr1A.gff | awk '$4>38000000 && $5<38500000' | awk '$3=="gene"' | sed "s/.*ID=//; s/\-.*//; s/;.*//" # This code is pretty specific to our data - it is just to clean the file, specifically, it removes the information before 'ID=', after '-', and after the after ';'.
awk '$1=="chr1A"' house_sparrow_chr1A.gff | awk '$4>38000000 && $5<38500000' | awk '$3=="gene"' | sed "s/.*ID=//; s/\-.*//; s/;.*//" | sort -u # we sort  it and get unique values
```

6 - Now, we look over the Orthofinder results. Explore the files in the folder 'Comparative_Genomics_Statistics'
```
# go to
cd ../02_orthofinder_run/

# Orthofinder will create a run called: Orthofinder/the_day_you_ran_it/ . Let's move there
cd OrthoFinder/Results*

# Orthofinder will create ' orthogroups ' - groups of genes which are similar, by employing some cool phylogenetic-oriented algorithms. I'll be talking about orthologs and orthogroups from now on.
# Orthofinder gives you plenty of folders. There is just so much you can do with this - I've explored TEs, genes, gene trees, gene expansions. You just need to be create and have a question. Importantly, they have:
Citation.txt # Credit where it's due
Single_Copy_Orthologue_Sequences # If you do phylogenetics, this is single-copy orthologs (1 per species/genome). Really cool.
Comparative_Genomics_Statistics # Cool statistics on duplications, trees etc
Species_Tree # A tree reconstruction based on your data (we have two genomes here so not very informative.
Gene_Duplication_Events # More info on duplicates
Orthogroups # Info Orthogroups, for you to process downstream if you're into analysing gene expansions etc.
Gene_Trees # a tree per each orthogroup
Orthogroup_Sequences # The fasta files orthogroups.
```

7 - We are interested in genes IV00_00019202 and IV00_00019209 - those in regions with a high FST.
```
cd Orthogroups
grep "IV00_00019202" Orthogroups.tsv
grep "IV00_00019209" Orthogroups.tsv

# Orthogroups are groups of (among others), orthologs. Orthologs are "genes in different species that evolved from a common ancestral gene by speciation" - because they share sequence similarity and origin we expect them to have the same functions. Notice that I write "expect" since to really verify this you'll need to do CRISP-R and other things. Comparative genomics is a good hypothesis generator, and a good first step, but be careful in overselling your results. See this paper for a discussion: https://www.sciencedirect.com/science/article/pii/S0169534722001689

# So, in this case it says that the gene OG0000290 is part an Orthogroup with the chicken gene 'ENSGALP00000041112.2'
OG0000290	chicken_ENSGALP00000041112.2	House_sparrow_IV00_00019209-RA

# There is a correponding chicken-ortholog in each. The chicken *faa file has gene names there. Go back to your 01_data folder and see what is there.
# Go back on the data folder and grep for that gene
cd ~/Orthofinder_exercise/01_data
grep "ENSGALP00000041112.2" Gallus_gallus.GRCg6a.pep.reduced.faa

# What is the gene name and symbol? What do we know about that gene?
# What about the second gene?
```
