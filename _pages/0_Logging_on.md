---
title: "Logging on to the cluster"
layout: archive
permalink: /logging_on/
---

Typically when working in bioinformatics with large datasets we would log into a computing cluster such as the one we have prepared for this course. Outside the course, you would work on the cluster/server of your home institution and log in in a very similar way as we show below. You likely would not need a keyfile for your home institution, but would have a password instead. How do we log in for the course?


#### Logging on

If you are using a Mac or Linux machine, you will need to open a `terminal` window. If you are using a Windows machine, start [MobaXterm](https://mobaxterm.mobatek.net/) or "Ubuntu on Windows" to get a `terminal`. For MobaXterm users, we recommend to go to "Settings", then "Configuration" and change the "Persistent home directory" to a directory of your choice. In the same Configuration window, also go to "SSH" and tick the box "SSH keepalive".


To connect to the Amazon cloud, we use the command `ssh`.

`ssh` stands for [secure shell](https://en.wikipedia.org/wiki/Secure_Shell) and is a way of interacting with remote servers. You will need to log in to the cluster using a keyfile that has been generated for you and sent via email.

To log on, we will need the keyfile you got from Physalia. If this was the cluster/server of your institution, you would not need that but use a password instead.

First, copy the keyfile into your home directory (change the number (123) to the user number assigned to you).

```shell
cp c123.pem ~
```

If you get an error about permissions, try the following:

```shell
chmod go= c123.pem
chmod -R u+x c123.pem
```

Then you should be able to log in with `ssh` whatever your working directory is. You need to provide `ssh` with the path to your key file, which you can do with the `-i` flag. This basically points to your identity file or keyfile (here shown for user 123). For example:

```shell
ssh -i "~/c123.pem" user123@54.245.175.86
```

Of course you will need to change the log in credentials shown here (i.e. the username and keyfile name) with your own. **Also be aware that the cluster IP address will change everyday**. We will update you on this each day.

The first time logging in you will be prompted to accept an RSA key - just type `yes`!

#### Downloading and uploading files

Occassionally, we will need to shift files between the cluster and our local machines. To do this, we can use a command utility called `scp` or [secure copy](https://en.wikipedia.org/wiki/Secure_copy). It works in a similar way to `ssh`. Let's try making a dummyfile in our local home directory and then uploading it to our home directory on the cluster.

```shell
# make a file
touch test_file
# upload a file from your computer to the cluster
scp -i "~/c123.pem" test_file user123@54.245.175.86:~/
```
Just to break this down a little we are simply copying a file, `test_file` in this case to the cluster. After the `:` symbol, we are specifying where on the cluster we are placing the file, here we use `~/` to specify the home directory.

Copying files back on to our local machine is just as straightforward. You can do that like so:

```shell
# download a file from the cluster to your computer
scp -i "~/c123.pem" user123@54.245.175.86:~/test_file ./
```
Where here all we did was use `scp` with the cluster address first and the location (our working directory) second - i.e. `./`

#### Making life a bit easier

If you are logging in and copying from a cluster regularly, it is sometimes good to use an `ssh` alias. Because the cluster IP address changes everyday, we will not be using these during the course. However, if you would like some information on how to set them up, see [here](https://markravinet.github.io/CEES_tips_&_tricks.html)
