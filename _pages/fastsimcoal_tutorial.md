---
title: "Demographic modeling with fastsimcoal2"
layout: archive
permalink: /fastsimcoal2/
---
`fastsimcoal2` is an extremely flexible demographic modelling software developed by the group of [Laurent Excoffier at University of Bern](http://www.cmpg.iee.unibe.ch/about_us/team/researchers/prof_dr_excoffier_laurent/index_eng.html). It uses the site frequency spectrum (SFS) to fit model parameters to the observed data by performing coalescent simulations. You can find the manual and more information [here](http://cmpg.unibe.ch/software/fastsimcoal2/).

### Preparing the input files

To run `fastsimcoal2` we need three input files which are all named in a consistent way. They are all just plain text files.

* observed SFS - `${PREFIX}_jointDAFpop1_0.obs`
* template file defining the demographic model - `${PREFIX}.tpl`
* estimation file defining the parameters - `${PREFIX}.est`

### Observed SFS

The observed SFS can be a derived SFS (i.e. also known as DAF or an unfolded SFS) if the ancestral state is unknown or a minor allele frequency SFS (i.e. MAF or folded SFS) otherwise. `fastsimcoal2` will identify the kind of SFS by its name suffix and the command flag `-m` or `-d`. For a single population, it expects the name `${PREFIX}_DAFpop0.obs` or `${PREFIX}_MAFpop0.obs`.

For two populations, the file names should end with `jointDAFpop1_0.obs` or `jointMAFpop1_0.obs`. If more than two observed populations are used, pairwise MAFs or DAFs can be provided following the naming scheme above or a multidimensional SFS can be used ending with `_DSFS.obs` or `_MSFS.obs` and specifying the `-multiSFS` flag when running the program. See the [fastsimcoal2 manual](http://cmpg.unibe.ch/software/fastsimcoal2/) for further details. The `.obs` file can be generated with `Arlequin`, `angsd`, `easySFS` (as in this [tutorial](https://speciationgenomics.github.io/easysfs/)) or several other tools.

### Template file

The template file describes the demographic model and the parameters of interest. Values that should be estimated (i.e. model parameters such as migration rate, splitting time etc) are given as different keywords. It is best practise to use capital letters for parameters names and it is important to avoid any names that are part of another parameter name or a function (e.g. `log`, `min`, `FREQ`).

As fastsimcoal2 is a coalescent simulator, all models need to be specified **backwards in time**. This means that the most recent event is specified first. The template file is best produced by modifying an example `tpl` file in a text editor.

In this tutorial, we will first write a model of two species that diverged in the face of gene flow and then evolved complete reproductive isolation. Unfortunately, we do not have a good estimate for the mutation rate, but we have an estimate for the (maximum possible) splitting time. Here we will set the split between the populations to 6000 generations.

**Question for discussion:** Why do we need to fix a parameter (e.g. splitting time) if we do not have a reliable mutation rate estimate?

### Estimation file

All keywords introduced in the template file need to be defined in the estimation file. For each keyword, the parameter distribution (uniform or log-uniform) and search range (min and max) are given on a single line. Each parameter can be an integer or a float, as specified by a first indicator variable.

It is also possible to specify rules which limit the possible parameter values. Typically, they are used to restrict a parameter to be smaller than another one. As an example, the parameter specifying the size before the bottleneck may be restricted to be at least ten times as large as the population size at the bottleneck. In more complex models, you might use rules to specify that a species pair may be required to split before another species pair.

Lastly, complex parameters can be used to compute parameter values with simple operations such as computing the ratio between two simple parameters or specifying a parameter as the minimum of two parameters. In addition, for each simple or complex parameter, one needs to specify if the parameter value should be written into an output file or not.

As with the template file, the estimation file is best produced by modifying an example `est` file in a text editor.

## Running fastsimcoal2

Once we have all input files ready, it is time to run `fastsimcoal`. In addition to the input files, we need to specify how many simulations and iterations `fastsimcoal` should perform, when to stop and how many threads (i.e. CPUs) can be used in parallel.

Let's run `fastsimcoal` with our model of two populations. First we make a `fastsimcoal` directory and in that one a directory for our model of `early_geneflow`:

```shell
PREFIX="early_geneflow"
mkdir fastsimcoal
cd fastsimcoal
mkdir early_geneflow
cd early_geneflow
cp /home/fastsimcoal/early_geneflow/* ./
fsc -t ${PREFIX}.tpl -e ${PREFIX}.est -m -0 -C 10 -n 10000 -L 40 -s 0 -M
```

This command runs `fastsimcoal` using a MAF (`-m`) while ignoring monomorphic sites (`-0`) and SFS entries with less than 10 SNPs (`-C`). This means that entries with less than 10 SNPs are pooled together. This option is useful when there are many entries in the observed SFS with few SNPs and with a limited number of SNPS to avoid overfitting.

`fastsimcoal` will also perform (`-n`) 10,000 simulations to estimate the expected SFS in each cycle and will run (`-L`) 40 optimization (ECM) cycles to estimate the parameters. Ideally, the number of iterations should be something between 200,000 and 1,000,000 but to make it faster, we are now only running it for 100,000 iterations. We also specify (`-M`) that we want to perform parameter estimation. With `-s 0`, we can tell `fastsimcoal` to output SNPs.

Once `fastsimcoal` is finished, we can have a look at the output files.
It produced a folder called `${PREFIX}` which contains a number of new files. The most relevant files are the following:

* `${PREFIX}.bestlhoods:` file with the Maximum likelihood estimates for each parameter specified "output" in the `est` file and the model likelihoods.
* `${PREFIX}.\_jointMAFpop1\_0.txt`: file with the expected SFS obtained with the parameters that maximized the likelihood during optimization. This is needed to visually check the fit of the expected SFS to the observed SFS. This file has the same suffix as the observed SFS provided.
* `${PREFIX}.simparam`: file with an example of the settings to run the simulations. This is useful to check when you have errors. Many times errors in specification of models can be detected in this file.
* `${PREFIX}\_maxL.par`: Model specification file with the best parameter estimates. It is basically the tpl file with the keywords replaced by estimated values. This file is useful if you want to simulate data under the best model using Arlequin.

Note, the bestlhoods file contains two different likelihoods: `MaxObsLhood`: is the maximum possible value for the likelihood if there was a perfect fit of the expected to the observed SFS, i.e. if the expected SFS was the relative observed SFS.
`MaxEstLhood`: is the maximum likelihood estimated according to the model parameters.

The better the fit, the smaller the difference between `MaxObsLhood` and `MaxEstLhood`.

### Finding the best parameter estimates

`fastsimcoal2` should not just be run once because it might not find the global optimum of the best combination of parameter estimates right away. It is better to run it 100 times or more. Of these runs, select the one with the highest likelihood which is the run with the best fitting parameter estimates for this model.

Due to time constraints, we will only run this model 5 times. Note, I added the flag `-q` for "quiet" which reduces the amount of information fsc writes to stdout.

```shell
 for i in {1..5}
 do
   mkdir run$i
   cp ${PREFIX}.tpl ${PREFIX}.est ${PREFIX}_jointMAFpop1_0.obs run$i"/"
   cd run$i
   fsc -t ${PREFIX}.tpl -e ${PREFIX}.est -m -0 -C 10 -n 10000 -L 40 -s0 -M -q
   cd ..
 done
```

To find the best run, i.e. the run with the highest likelihood, or better the smallest difference between the maximum possible likelihood (`MaxObsLhood`) and the obtained likelihood (`MaxEstLhood`), we can check the `.bestlhoods` files.

```shell
cat run{1..5}/${PREFIX}/${PREFIX}.bestlhoods | grep -v MaxObsLhood | awk '{print NR,$8}' | sort -k 2
```

I wrote a script for you that automatically extracts the files of the best run and copies them into a new folder which it calls `bestrun`. Just run it in the directory where all the folders run are located:

```shell
fsc-selectbestrun.sh
```

### Model comparison with AIC

In order to find the best model, the likelihoods of the best run of each model should be compared. Comparing raw likelihoods is problematic, because a model with more parameters will always tend to result in a better fit to the data. Therefore, the (Akaike information criterium)[https://en.wikipedia.org/wiki/Akaike_information_criterion] or AIC is typically calculated to determine if the models differ in their likelihoods accounting for the number of parameters in each model.

```shell
cd bestrun/
calculateAIC.sh early_geneflow
```

### Visualize the model fit

To visualize the fit of the simulated SFS to the data, we can use an `R` script that David Marques wrote - `SFStools.r` - which you can download here)[https://github.com/marqueda/SFS-scripts/].

To visualize the model with the best parameter estimates, we can use one of my R scripts - `plotModel.r`.

```shell
SFStools.r -t print2D -i early_geneflow
plotModel.r -i early_geneflow -p NyerMak,PundMak
```
Now, let's download the pdfs these scripts generated.

```shell
scp -i c.pem user@<ip>:~/fastsimcoal/early_geneflow/bestrun/*pdf ./
```

### Model comparison with Likelihood distributions

One drawback of the composite likelihoods in model tests based on AIC is that it can overestimate the support for the most likely model if the SNPs are not independent (here they are not LD-pruned). Another way to infer if the models are really different and do not just differ because of stochasticity in the likelihood approximation, is to get likelihood distributions for each model. This is done by running each model with the best parameter values multple times (ideally about 100 times). The likelihoods will differ because `fastsimcoal` does not compute the likelihood but rather approximates it with simulations. If the ranges of likelihoods of two models overlap, it means that they do not differ significantly, i.e. provide an equally good fit to the observed data.

Let's recompute the likelihood for the best run for the `early_geneflow` model.

```shell

model="early_geneflow"
cd ~/fastsimcoal/$model/bestrun

# create temporary obs file with name _maxL_MSFS.obs
cp ${model}_jointMAFpop1_0.obs ${model}_maxL_jointMAFpop1_0.obs

# Run fastsimcoal 20 times (in reality better 100 times) to get the likelihood of the observed SFS under the best parameter values with 1 mio simulated SFS.
for iter in {1..20}
do
 fsc -i ${model}_maxL.par -n1000000 -m -q -0
 # Fastsimcoal will generate a new folder called ${model}_maxL and write files in there

 # collect the lhood values
 sed -n '2,3p' < ./${model}_maxL/${model}_maxL.lhoods  >> ./${model}.lhoods

 # delete the folder with results
 rm -r ${model}_maxL/
done

```

We would now repeat this for different models: ongoing gene flow, a model without gene flow, a model of secondary contact (recent gene flow) and a model with different amounts of gene flow in the past and in recent times.

Due to time constraints, we will give you the results of these models. They are in the same format as the folder we generated for the model with early gene flow. Load them into your modeling folder. Note, the `-r` flag stands for "recursive" and allows to also copy directories and their contents.

```shell
cp -r /home/fastsimcoal/extramodels/* ./
mkdir lhoods
cd lhoods
cp -r /home/fastsimcoal/lhoods/* ./
cp ../early_geneflow/bestrun/early_geneflow.lhoods ./
```

Now, let's download the likelihoods and plot them: (NB: Windows users must use `Filezilla`)

```shell
scp -i c.pem user@<ip>:~/fastsimcoal/lhoods/* ./
```

On the local computer, we can plot the likelihoods as boxplots.

```r

setwd()

# Read in the likelihoods
early_geneflow<-read.table("early_geneflow.lhoods")
ongoing_geneflow<-read.table("ongoing_geneflow.lhoods")
diff_geneflow<-read.table("diff_geneflow.lhoods")
recent_geneflow<-read.table("recent_geneflow.lhoods")
no_geneflow<-read.table("no_geneflow.lhoods")

# Plot the likelihoods

par(mfrow=c(1,1))
boxplot(range = 0,diff_geneflow$V1,recent_geneflow$V1,early_geneflow$V1,ongoing_geneflow$V1,
        no_geneflow$V1,
        ylab="Likelihood",xaxt="n")
axis(side=1,at=1:5, labels=c("early+recent","recent","early","constant","no"))

```
