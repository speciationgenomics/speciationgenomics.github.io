---
title: "awk"
layout: archive
permalink: /awk/
---

```shell

# First let's make a folder to work in
mkdir awk

# Let's get some files
cp /home/data/awkFiles/* ./

# Let's have a look at these file
head cichlids.info  | column -t
head cichlids.imiss  | column -t

# Let's get the first line
# in awk, NR means number of line
awk 'NR==1' cichlids.imiss
awk 'NR==10' cichlids.imiss # or the tenth line

# Get only every 20th line
awk 'NR%20==0' cichlids.imiss

# awk can be used like grep to select all lines with a specific word
awk '/INDV/' cichlids.imiss
# this gives the same output as
grep "INDV" cichlids.imiss

# Let's use awk for something else that grep cannot do
# Get all samples with more than 1% missing data
# note: $5 means fifth column
awk '$5>0.01' cichlids.imiss

# With curly brackets {} we can specify more complex awk commands.

# Get just the codes of these samples (column 1) with missing data proportion above 0.01
awk '{if($5>0.01) print $1}' cichlids.imiss
# with if() we can define a specific criterium that needs to be fulfilled (e.g. column 5 needs to be greater than 0.01)

# To get the mean missing data proportion, we can use the END{} statement. The part in END{} is preformed after going through all lines.
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

# Order the lines of cichlids.info according to the order of individuals in cichlids.imiss
awk 'FNR==NR {file2array[$1] = $0; next} $1 in file2array {print file2array[$1]}' cichlids.imiss cichlids.info

```
