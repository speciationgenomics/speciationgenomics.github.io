---
title: "awk"
layout: archive
permalink: /awk/
---

awk is a domain-specific language designed for text manipulation. The abbreviation awk could easily stand for awkward as the syntax is a bit unintuitive. However, it actually stands for the authors Alfred Aho, Peter Weinberger and Brian Kernighan. Awk works on a line-by-line base. It goes through each line in the file and does to the line whatever the user has specified. It is very versatile and can do very different text manipulations such as computing means, combining columns, adding new columns, selecting specific lines etc. The awk command is put in '' (single quotes). Below you can see some examples.

```shell

# First let's make a folder to work in
mkdir awk

# Let's get some files
cp /home/data/awkFiles/* ./

# Let's have a look at these file (note | column -t makes the columns nicely aligned)
head cichlids.info  | column -t
head cichlids.imiss  | column -t
```

In the simplest way of running awk, we can tell awk to print out all lines where a specific condition is true. The condition, e.g. line number is 1, is put between single quotes.

```shell

# Let's get the first line
# in awk, NR means number of line
awk 'NR==1' cichlids.imiss
#
awk 'NR==10' cichlids.imiss # or the tenth line

# Get only every 20th line
awk 'NR%20==0' cichlids.imiss

# awk can be used like grep to select all lines with a specific word such as "TI" which is found in five sample names.
awk '/TI/' cichlids.imiss
# this gives the same output as
grep "TI" cichlids.imiss

# Let's use awk for something else that grep cannot do
# Get all samples with more than 1% missing data
# note: $5 means fifth column
awk '$5>0.01' cichlids.imiss

# you can can also combine the previous codes to select only lines with TI with more than 1% missing data
awk '/TI/ $5>0.01' cichlids.imiss

```

By using curly brackets {} we can specify more complex awk commands.

```shell
# Get just the codes (column 1) of the samples with missing data proportion above 0.01
awk '{if($5>0.01) print $1}' cichlids.imiss
# with if() we can define a specific condition that needs to be fulfilled (e.g. column 5 needs to be greater than 0.01)

# We could also specify now that the letters TI need to be in the first column and the fifth column needs to be higher than 0.01.
awk '{if($1~/TI/ && $5>0.01) print $1}' cichlids.imiss
# Note that && means that both conditions need to be true. You could specify lots of different conditions.

# if we now wanted to print the entire line fulfilling both conditions (containing TI and >1% missing data), we have to specify print $0 ($0 stands for the entire line)
awk '{if($1~/TI/ && $5>0.01) print $0}' cichlids.imiss

```
Everything until now was always performed for each line. If we wanted to do something before or after reading the lines, we can use BEGIN{} and END{}, before or after the main code {}, respectively.

```shell

# Let's use END{} to get the mean missing data proportion. The part in END{} is preformed after going through all lines. a is a variable that will sum up the column five entries (missing data proportions). In the end statement, we specify that it should calculate the sum of missing data proportions divided by the number of lines.
awk '{a+=$5}END{print a/NR}' cichlids.imiss

# Actually the header is still in here, so let's skip the first line
awk '{if(NR>1) a+=$5}END{print a/(NR-1)}' cichlids.imiss

# Now if we want to get the mean missing data proportion per group
# we need to first combine the two files to get the missing data proportion and group information in a single file.

# Let's have a look at their structure again
head cichlids.info
head cichlids.imiss

# We cannot just paste them together as they are sorted in different ways
# Therefore, we need to sort the files First
sort -nk 1 cichlids.info > cichlids.info.sorted

# Or we can just do that directly without rewriting them by using <() which makes linux interpret the output of the command between the brackets as a file
# This is a useful alternative to generating lots of intermediate files
paste cichlids.info.sorted <(sort -nk 1 cichlids.imiss) > cichlids.info.full

# Now annoyingly the header is somewhere in between
cat <(grep INDV cichlids.info.full) <(grep -v INDV cichlids.info.full) > cichlids.info.full.tmp

mv cichlids.info.full.tmp cichlids.info.full

# Let's check how many samples we find in each group
cut -f 2 cichlids.info.full | sort | uniq -c

# Get the mean missing data proportion per group
awk '{missing[$7]+=$5; count[$7]++}END{for(g in missing){print g,missing[g]/count[g]}}' cichlids.info.full  | column -t

# Add a new column with the individual name (column 1, e.g. ind1) and group information (column 7, e.g. piscivore) combined, e.g. ind1_piscivore
# To make it neater to look at, I pipe it into column -t which shows the result with all columns aligned
awk '{print $0"\t"$1"_"$7}' cichlids.info.full | column -t

# replace the fifth column by this new combined individual and group information
awk '{$5=$1"_"$7; print $0}' cichlids.info.full | head | column -t

# You can also modify the numbers, change the proportion of missing data to percent of missing data (x100)
awk '{$5=$5*100; print $0}' cichlids.info.full | head | column -t

# Order the lines of cichlids.imiss according to the order of individuals in cichlids.info
awk 'FNR==NR {file1array[$1] = $0; next} $1 in file1array {print file1array[$1]}' cichlids.imiss cichlids.info
# If specifying two files, NR counts the lines of both files together, whereas FNR starts again at 0 when counting the lines of the second file. Therefore, FNR=NR is only true for the first file. We can read in the lines of the first file into an array called file1array. The key for the array is the first column and thus the individual name. The first file is then printed out in the order of the second file. For each individual (column 1 entry) in the second file, it will print the corresponding line of file one saved in the array.

# We could also write that it should in addition also give the second column of the second file (group information) and write this into a new file.
awk 'FNR==NR {file2array[$1] = $0; next} $1 in file2array {print file2array[$1]"\t"$2}' cichlids.imiss cichlids.info > cichlids.imiss.info

```
