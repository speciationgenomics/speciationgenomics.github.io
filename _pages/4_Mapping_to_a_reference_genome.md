---
title: "Mapping to a reference genome"
layout: archive
permalink: /mapping_reference/
---

One of the first steps we need to take along our pathway to population/speciation genomic analyses is mapping our data to a reference genome.

Using alignment software, we essentially find where in the genome our reads originate from and then once these reads are aligned, we are able to either call variants or construct a consensus sequence for our set of aligned reads.

### Getting access to the reference genome

We will be aligning our sequence data to the *Pundamilia nyererei* reference genome, first published by [Brawand *et al.* (2014)](https://www.nature.com/articles/nature13726) and more recently updated using a linkage map by [Feulner *et al.* (2018)](http://www.g3journal.org/content/8/7/2411).

You will find a copy of the reference genome in the `/home/data/reference/` directory. Make a `reference` directory in your own home directory and then copy the reference genome like so:

```shell
cd ~
mkdir reference
cd reference
cp /home/data/reference/P_nyererei_v2.fasta.gz .
```
The file we copied is compressed with `gzip`, so before we can do anything with it, we need to decompress it. Usually we avoid decompressing files but it was compressed here because of it's large size and we need it to be uncompressed for it to be used properly in our analysis. So to do this we simply use `gunzip`:

```shell
gunzip P_nyererei_v2.fasta.gz
```

In order to align reads to the genome, we are going to use `bwa-mem2` which is a very fast and straightforward aligner. See [here](https://github.com/bwa-mem2/bwa-mem2) for more details on `bwa-mem2`.

#### Indexing the reference genome

Before we can actually perform an alignment, we need to index the reference genome we just copied to our home directories. This essentially produces an index for rapid searching and aligning. We use the `bwa-mem2 index` tool to achieve this. However, the command takes some time to run, so we will use a useful Unix utility called `screen` to run it in the background (we will return to **why** it is advantageous to use `screen` momentarily).

Use the following command to start a screen:

```shell
screen -S index
```
The terminal should flicker and you will now be in another screen. You can prove this to yourself with the following command:

```shell
screen -ls
```
This will list the running screens and should show you one called `index`. If you are inside this screen, you will also see that it is listed as attached.

Now that you are sure you are inside the screen, run the following command.

```shell
bwa index P_nyererei_v2.fasta
```
Let's breakdown what we actually did here. As you might imagine `bwa-mem2 index` is a tool for indexing. We simply specify the reference fasta file from which to build our genome index.

As you will have noticed by now, this command takes quite some time to run. Rather than sit there and wait, you can press `ctrl + ad` keys all together to shift back to the main screen (i.e. the one you started in).

##### Why screen?

While we are waiting for the index to complete, this is a good moment to discuss the benefits of `screen`. You've already seen that screen allows you to run processes in the background, but in addition to that, screen also protects your session. If you are running an analysis and the terminal you are in crashes or your computer stops or something, it will send a kill signal to your analysis and stop it. However, if you have placed that analysis in another screen, this kill signal will not reach it - meaning you can safely log out and log back in to a node and your work will be running. This is just one way to run an analysis that will take a long time.

To return to your screen at any time and check the analysis, use the following:

```shell
screen -rd
```
If you have multiple screens, you will need to specify the ID number - to discover this, use `screen -ls`.

##### Back to the indexing

OK so hopefully by now, the indexing should have completed - on our cluster it takes about 15 mins. If not, then kill the process using `ctrl + c` and exit the screen by typing `exit`.

To save time and to allow everyone to proceed at the same pace, we will instead copy some already built indexes to our home directory

```shell
cp /home/data/reference/P_nyererei_v2.fasta.* .
```

Use `ls` to take a look, but this will have copied in about 5 files all with the `P_nyererei_v2.fasta.` prefix that we will use for a reference alignment.

When `bwa-mem2` aligns reads, it needs access to these files, so they should be in the same directory as the reference genome. Then when we actually run the alignment, we tell `bwa-mem2` where the reference is and it does the rest. To make this easier, we will make a variable pointing to the reference.

```shell
REF="~/reference/P_nyererei_v2.fasta"
```

#### Performing a paired end alignment

Now we are ready to align our sequences! To simplify matters, we will first try this on a single individual. First, we will create a directory to hold our aligned data:

```shell
cd ~
mkdir align
cd align
```
As a side note, it is good practice to keep your data well organised like this, otherwise things can become confusing and difficult in the future.

To align our individual we will use `bwa-mem2`. You might want to first have a look at the options available for it simply by calling `bwa-mem2`. We are actually going to use `bwa-mem2 mem` which is the best option for short reads.

As we mentioned above, we will use the individual - `10558.PunPundMak` which we have already trimmed. There are two files for this individual - `R1` and `R2` which are forward and reverse reads respectively.

Let's go ahead and align our data, we will break down what we did shortly after. Note that we run this command from the home directory.

```shell
bwa-mem2 mem -t 4 $REF \
/home/data/wgs_raw/10558.PunPundMak.R1.fastq.gz \
/home/data/wgs_raw/10558.PunPundMak.R2.fastq.gz > 10558.PunPundMak.sam
```
Since we are only using a shortened fastq file, with 100K reads in it, this should just take a couple of minutes. In the meantime, we can breakdown what we actually did here.

* `-t` tells `bwa-mem2` how many threads (cores) on a cluster to use - this effectively determines its speed.
* Following these options, we then specify the reference genome, the forward reads and the reverse reads. Finally we write the output of the alignment to a **SAM file**.

Once your alignment has ended, you will see some alignment statistics written to the screen. We will come back to these later - first, we will learn about what a SAM file actually is.

#### SAM files

Lets take a closer a look at the output. To do this we will use `samtools`. More details on `samtools` can be found [here](http://www.htslib.org/doc/samtools.html)

```shell
cd align
samtools view -h 10558.PunPundMak.sam | head
samtools view 10558.PunPundMak.sam | head
```

This is a SAM file - or sequence alignment/map format. It is basically a text format for storing an alignment. The format is made up of a header where each line begins with `@`, which we saw when we used the `-h` flag and an alignment section.

The header gives us details on what has been done to the SAM, where it has been aligned and so on. We will not focus on it too much here but there is a very detailed SAM format specification [here](https://samtools.github.io/hts-specs/SAMv1.pdf).

The alignment section is more informative at this stage and it has at least 11 fields. The most important for now are the first four. Take a closer look at them.

```shell
samtools view 10558.PunPundMak.sam | head | cut -f 1-4
```

Here we have:

* The sequence ID
* The flag - these have various meanings, 0 = mapped, 4 = unmapped
* Reference name - reference scaffold the read maps to
* Position - the mapping position/coordinate for the read

**Note!!** The other fields are important, but for our purposes we will not examine them in detail here.

Now, lets look at the mapping statistics again:

```shell
samtools flagstat 10558.PunPundMak.sam
```

This shows us that a total of 200k reads were read in (forward and reverse), that around 94% mapped successfully, 81% mapped with their mate pair, 1.94% were singletons and the rest did not map.

#### BAM files

One problem with SAM files is that for large amounts of sequence data, they can rapidly become HUGE in size. As such, you will often see them stored as BAM files (Binary Aligment/Mapping format). A lot of downstream analyses require the BAM format so our next task is to convert our SAM to a BAM.

```shell
samtools view 10558.PunPundMak.sam -b -o 10558.PunPundMak.bam
```

Here the `-b` flag tells `samtools` to output a BAM and `-o` identifies the output path. Take a look at the files with `ls -lah` - you will see a substantial difference in their size. This would be even more striking if we were to map a full dataset.

You can view bamfiles in the same way as before.

```shell
samtools view 10558.PunPundMak.bam | head
```
Note that the following will not work (although it does for SAM) because it is a binary format

```shell
head 10558.PunPundMak.bam
```

Before we can proceed to more exciting analyses, there is one more step we need to perform - sorting the bamfile. Most downstream analyses require this in order to function efficiently

```shell
samtools sort 10558.PunPundMak.bam -o 10558.PunPundMak_sort.bam
```

Once this is run, we will have a sorted bam. One point to note here, could we have done this is a more efficient manner? The answer is yes, actually we could have run all of these commands in a single line using pipes like so:

```shell
bwa-mem2 mem -t 4 $REF /home/data/wgs_raw/10558.PunPundMak.R1.fastq.gz /home/data/wgs_raw/10558.PunPundMak.R2.fastq.gz | samtools view -b | samtools sort -T 10558.PunPundMak > ./align/10558.PunPundMak_sort.bam
```

However as you may have noticed, we have only performed this on a single individual so far... what if we want to do it on multiple individuals? Do we need to type all this everytime? The answer is no - we could do this much more efficiently using the bash scripting tools we learned earlier today.

### Working with multiple individuals

There are actually multiple ways to work with multiple individuals. One way is to use a bash script to loop through all the files and align them one by one. We will examine this way in detail together. An alternative way is to use some form of parallelisation and actually map them all in parallel. We will also demonstrate an example of this, but it is quite advanced and is really only to give you some familiarity with the approach.

#### Using a bash script to loop through and align individuals

We are going to work together to write a bash script which you can use to map multiple individuals one by one. It is typically a lot more straightforward to write a script locally on your machine and then upload it to the cluster to make sure it does what you want. There are plenty of different scripting and code specific text editors out there but a free, multi-platform, open source editor we recommomend is [Atom](https://atom.io/).

The first thing we will do is initiate the script with the line telling the interpreter that the file must be treated as a bash script:

```shell
#!/bin/sh
```
Next, we declare some variables in our script. As we learned from our adventures with `bwa mem` above, we need to point to the reference genome. We do this like so:

```shell
REF="~/reference/P_nyererei_v2.fasta"
```
Next, we will declare an array to ensure that we have all our individuals

```shell
INDS=($(for i in /home/data/wgs_raw/*R1.fastq.gz; do echo $(basename ${i%.R*}); done))
```
As we learned before, this will create a list of individuals which we can then loop through in order to map each individual. Here we used bash substitution to take each forward read name, remove the directory and leave only the individual name.

If you want to, decleare the array in your command line and then test it (i.e. type `echo ${IND[@]}`). You will see that we have only individual names, which gives us some flexibility to take our individual name and edit it inside our `for` loop (i.e. it makes defining input and output files much easier.

Next we will add the actual `for` loop to our script. We will use the following:

```shell
for IND in ${INDS[@]};
do
	# declare variables
	FORWARD=/home/data/wgs_raw/${IND}.R1.fastq.gz
	REVERSE=/home/data/wgs_raw/${IND}.R2.fastq.gz
	OUTPUT=~/align/${IND}_sort.bam

done
```
In this first version of our loop, we are making the `$FORWARD`, `$REVERSE` and `$OUTPUT` variables, making this much easier for us to edit later. It is a good idea to test the variables we are making in each loop, using the `echo` command. Feel free to test this yourself now - you can just copy and paste from your script into the command line to make it easier.

After we have tested the loop to make sure it is working properly, all we have to do is add the `bwa mem` command we made earlier but with our declared variables in place.

```shell
for IND in ${INDS[@]};
do
	# declare variables
	FORWARD=/home/data/wgs_raw/${IND}.R1.fastq.gz
	REVERSE=/home/data/wgs_raw/${IND}.R2.fastq.gz
	OUTPUT=~/align/${IND}_sort.bam

	# then align and sort
	echo "Aligning $IND with bwa"
	bwa-mem2 mem -t 4 $REF $FORWARD \
	$REVERSE | samtools view -b | \
	samtools sort -T ${IND} > $OUTPUT

done
```

With this completed `for` loop, we have a command that will create variables, align and sort our reads and also echo to the screen which individual it is working on, so that we know how well it is progessing.

Although it takes a bit more work to make a script like this, it is worth it. This script is very general - you would only need to edit the variables in order to make it work on almost any dataset on any cluster. Reusability (and clarity) of scripts is something to strive for.

Now thatwe have built our script, it is time to use it. Save it and name it `align_sort.sh` We will move it on to the cluster, either using `scp` or `filezilla`, a process explained in [this tutorial](). Be sure to move it to your `home` folder.

Once the script is on the cluster, open a screen and call it `align`; (i.e. `screen -S align`). Then run the script like so:

```shell
bash align_sort.sh
```

You will now see the script running as the sequences align. Press `Ctrl + A + D` in order to leave the screen. Now is a good time to take a break as you wait for the job to complete.

#### Advanced: simultaneously aligning using `parallel`

An alternative to mapping each individual one by one is to use `parallel`, a utility which lets you run commands in parallel. `parallel` is actually very easy to use.

First we need to make a list of individuals. We can do that using similar code to that we used above to declare an array:

```shell
for i in /home/data/wgs_raw/*R1.fastq.gz; do echo $(basename ${i%.R*}); done > inds
```
So now we have created a file called `inds` with each individual on a separate line. Use `cat` to check it.

Next we will look at how `parallel` works. The following command will take `inds` as an input file and then print all values contained to the file at once, splitting the job across different cores on the cluster.

```shell
parallel 'echo {}' :::: inds
```
Now, you won't really notice the benefits of paralell like this, but at least you know understand the syntax. The `{}` is just a placeholder, allowing paralell to read in from the file which we define after `::::`.

Next we need to write a quick script - one that will take an individual name from the command line and perform the alignment on it. We can use a modified version of the script we wrote in the last section.

```shell
#!/bin/sh

# align a single individual
REF=~/reference/P_nyererei_v2.fasta

# declare variables
IND=$1
FORWARD=/home/data/wgs_raw/${IND}.R1.fastq.gz
REVERSE=/home/data/wgs_raw/${IND}.R2.fastq.gz
OUTPUT=~/align/${IND}_sort.bam

# then align and sort
echo "Aligning $IND with bwa"
bwa-mem2 mem -t 4 $REF $FORWARD \
$REVERSE | samtools view -b | \
samtools sort -T ${IND} > $OUTPUT
```

This script is more or less identical to that above but it no longer contains a `for` loop and furthermore, it defines the `$IND` variable from the command line as `$1`. Save the script as `parallel_align.sh` and move it to the cluster.

This means that the script could be run like so:

```shell
sh parallel_align.sh 10558.PunPundMak
```
This would actually work, but only for one individual. To ensure that we use all the available computing cores to run align on all individuals simultaneously, we use `parallel` like so:

```shell
parallel `sh parallel_align.sh {}` :::: inds
```

This takes a few minutes but it is well worth learning how to use parallel in this way, it can significantly speed up your analyses.


Finally, we need to remove duplicate reads from the dataset to avoid PCR duplicates and technical duplicates which inflate our sequencing depth and give us false certainty in the genotype calls. We can use [Picard Tools](https://broadinstitute.github.io/picard/) to do that. Again, we can parallelize this to process 10 bam files at once.

```shell
cat inds | parallel --verbose -j 10 \
java -Xmx1g -jar /home/scripts/picard.jar \
 MarkDuplicates REMOVE_DUPLICATES=true \
 ASSUME_SORTED=true VALIDATION_STRINGENCY=SILENT \
 MAX_FILE_HANDLES_FOR_READ_ENDS_MAP=1000 \
 INPUT={}.bam \
 OUTPUT={}.rmd.bam \
 METRICS_FILE={}.rmd.bam.metrics

# Now we need to index all bam files again and that's it!
samtools index *.rmd.bam
 ```
