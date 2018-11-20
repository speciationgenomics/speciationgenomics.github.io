---
title: "Getting used to Unix"
layout: archive
permalink: /getting_used_to_unix/
---

## Getting used to Unix

### What is Unix?

[Unix is an operating system or OS](https://en.wikipedia.org/wiki/Unix). Much like Windows, Mac OS X and Ubuntu it is a way to interact to interact with your computer. Unlike those systems though, it is not a graphical user interface (**GUI**). The Unix kernel is the essentially the 'hub' of the operating system and you can interact with it using the command line. Don't worry if this doesn't make sense at first, it will do soon...

You might actually already have some experience with the command line. For example, you might have used MS-DOS which is another command-line OS. Alternatively, if you have a Mac or Linux computer, you might actually have already used Unix, since both of these operating systems are actually built on top of it.

### Why bother to learn Unix as a biologist?

It might seem pretty stupid that in 2018, when you have a ridiculously powerful smartphone in your pocket, that it is necessary to learn how to interact with a computer by typing. Why can't we use an app with a well designed interface? The simple answer to this is that despite it's outdated appearance, Unix and the command line is extremely powerful and flexible. You can combine commands and tools to do things with your data that would simply be far too fiddly with a desktop based computer system.

Of course, the best way to actually learn about why and how Unix can be so powerful to actually use it. So in this tutorial, we will get used to some of the basics of interacting with the command prompt.

### Getting started

Although Mac and Linux computers have a Unix basis, typically when working in bioinformatics we would log into a computing cluster such as the one we have prepared for this course. When you are back in your home institutions, you can also use this to log into the computing nodes there. How do we log in?

#### Mac OS X and Linux users

If you are using a Mac or Linux machine, you will need to open a `terminal` window and then type `ssh`.

`ssh` stands for [secure shell](https://en.wikipedia.org/wiki/Secure_Shell) and is a way of interacting with remote servers. You need to log on to the server for this course like so:

```
ssh username@login.node
```
You will then be prompted for a password and you should be inside the server!

#### Windows users

If you are using a Windows machine, you will need to log on using [PuTTY](https://www.putty.org/) since there is no native `ssh` client.

### Terminal and the prompt

However you logged in, once you are on the cluster, you should see a terminal window. This is your main point of access to the cluster and the way you interact with Unix. In short, when we talk about the command-line, this is what we mean. It might look a little daunting at first, but it won't take too long to get to grips with.

The first thing you will see in the terminal is a **prompt**. It looks like this

```
[mark@ourserver ~]$
```
This will appear on every line and contains some important information. It shows the your username and where you are in the system (NEED TO CHECK IF THAT IS THE CASE). Finally, there is the `$` character. This denotes the end of the prompt and it is letting us know that it is waiting for us to tell it to do something.

You can try hitting enter repeatedly and you will notice that the same line repeats again and again. Clearly it wants some instructions, so let's give it some.

### Your first Unix commands

The first thing we are going to learn about is how to perform some basic Unix operations and how we can navigate the Unix file system.

To start with, we should be in our home directory. This is where we end up automatically when we log in but just to make sure, let's see what commands we would use to actually get there:

```
cd ~
```
`cd` simply stands for change directory and in Unix, the `~` is shorthand for the home directory.

You could also do the following:

```
cd $HOME
```
Here we used `cd` again but with an **environmental variable** for the home directory. We will return to what this means later.

What's in our home directory? We can have a look like so:

```
ls
```

`ls` simply means list. In this instance, you won't see anything because well, there's nothing in your home directory. So let's change that. Type the following:

```
mkdir stuff1
```
Here we use the `mkdir` command to make a directory. We called it `stuff1`. We can also make multiple directories at once:

```
mkdir stuff2 stuff3
```

Now we have three different directories contained within our home directory. Let's check that using `ls` again.

You should now see these directories listed! Let's move into one of them:

```
cd stuff1
```
Now we are inside stuff1. We have moved directory - we are now working inside a different directory to our home directory. How can we prove this to ourselves? Try:

```
pwd
```
The `pwd` command stands for **print working directory** and is a useful way of writing to the screen where in the Unix filesystem we are. It will print a file path to the screen, like so:

```
/mark/home/stuff1/
```
Is there anything inside this directory? We can check again using `ls`.

Obviously not since we just created it. So let's make some files to fill it up.

```
touch file1 file2
```
Try `ls` again - now you will see two files listed. Note that `touch` is not really a great way to create files - it is usually used to update the last time a file was accessed (i.e. literally 'touch' it). But for here, it is a quick and easy way to make an empty file for demonstration.

Hang on, the files are empty? We can check this using the following commands:

```
cat file1
cat file2
```
`cat` is a useful Unix command that prints files into the standard output of the terminal. It stands for **concatenate** and we will be using it a lot in the near future.

Of course because these files are empty, `cat` doesn't do anything. So lets change that and edit one of them. We can do this using `nano`, a simple command-line text editor. Use the following command to open `file1` with `nano`:

```
nano file1
```

Then add the following text:

```
Hello world!
```
You'll then need to exit `nano` and save the changes (there will be an onscreen prompt explaining how).

Next try printing the contents of the file we edited:

```
cat file1
```

So now we know how to create directories, move into them and create files, edit those files and also print them to the screen. We also learned how to list the files in a directory, in order to get an idea of the file structure we are working with. Next, we will learn how to move and copy files.

### Moving and copying

We should still be in `stuff1`. Let's move `file1` into one of the other directories we created.

```
mv file1 ../stuff2/
```
There are some things to unpick here. Firstly, `mv` is the command to move files (standing er... for move!). The second argument is what we want to move, `file1` in this case and the third argument is where we want to move it to. Here we used `../stuff2/`.

This third argument is a relative path. In Unix, `./` signifies the current directory. `../` signifies the directory above the one you are in. So, since we are in `stuff1`, `../` means in the home directory. We can test this with `ls`. Compare the outcomes of these commands:

```
ls ./
ls ../
ls $HOME
ls $HOME/stuff2/
```

That last command should show us that `file1` has indeed been moved into `stuff2`. We will come back to the subjects of paths again soon.

`mv` is a handy utility - it can also be used to rename file. For instance, we can do this:

```
mv file2 file10
```
If you then use `ls`, you'll see the file name has changed. This very useful as you will need to rename files a lot in bioinformatics. Actually this is a good point to mention that you should try and keep your filenames short and clear - long file names are a nightmare for you and your collaborators!

One last note on `mv` is that it is a powerful command and it can easily be used to accidentally overwrite files. You need to be careful when you use it to make sure that doesn't happen.

An alternative to moving files is to copy them, which you might need to do from time to time. Let's move into `stuff2` and copy `file1`.

```
cd ../stuff2
cp file1 file1_copy
```
Here we use `cp` to copy our file, with the first argument being the file we want to copy and the second being the name we want to copy it to. Clearly you can see that here we gave the file a new name, but it doesn't have to be this way. Let's move into `stuff3` and demonstrate:

```
cd ../stuff3
cp ../stuff2/file1_copy ./
```
With this `cp` command, we first use `../stuff2/file1_copy` to tell the tool where the file we want to copy is, then we use the `./` to specify we want to copy it to the directory we are currently in. `ls` will confirm the file is now in our current directory, complete with the same name.

### Cleaning up

We have obviously created a lot of files and directories in the process of this tutorial. What if we want to get rid of them? This is actually quite an important skill and one that is often overlooked. Believe us, when you work with lots of data, getting rid of files you don't need is very, very important. Cluttered directories are an absolute nightmare to deal with.

We can easily remove files with the `rm` command. For instance:

```
rm -i file1_copy
```

Here there is a **flag** after `rm` - `-i` which simply tells the command to ask permission before deleting. Indeed, when you run the command above, you will receive a prompt asking you if you really want to delete a file.

Typing `rm` for each file is a hassle. How can we do this faster? Let's move into `stuff2` and see:

```
cd ../stuff2
rm -i *
```
Here we used an **asterisk wildcard** (a topic we will return to) in order to tell `rm` to delete **EVERYTHING** in a directory. As you will see - it is very important to be careful with such a powerful command.

Let's move back up to the home directory and delete directories.

```
cd ../
rmdir stuff3 stuff2
```
To delete a directory, we need to use the command `rmdir` which is just remove directory. This works well in this case, but what if we do the following?

```
rmdir stuff1
```
This will return an error because `stuff1` is not empty. So instead, we need to delete it and all it's contents. We can do this easily like so:

```
rm -r stuff1/*
```
In this case, the `-r` flag tells `rm` to run recursively - i.e. within the directories contained within our target, which is `stuff1/*` - i.e. `stuff1` and all the files it contains. If you use `ls` you should now see that your home directory is completely empty.

There are more command flags than `-i` and `-r` for `rm`. They are common on a lot of progrmas and you can often use `man` and `--help` to get a guide to them. For example:

```
man rm
rm --help
```
The last and most important point of this introductory tutorial is to be aware of how dangerous a command like rm can be. If you use rm indiscriminately in the wrong folder the results can be disastrous. A colleague once deleted all his data by using `rm` * in his home directory. Adding the -i flag means that `rm` will always be used interactively, in other words it will ask your permission before deleting files.

### Navigating in Unix

To a beginner, the Unix file system is big, large and confusing. Don’t worry though, there are plenty of good resources out there that can [demystify](https://en.wikipedia.org/wiki/Unix_filesystem) it.

Most of your navigation will be done using `cd` - the change directory command we learned about in the previous session. The three most important `cd` commands to remember are:

```
cd /
cd ~
cd
```

The first of these will take you to the root directory (`/` at the beginning of a file path means root). The second will take you to your home directory which is denoted by a tilde (`~`). However a nice thing with `cd` is that you don’t even need to use the tilde character to go to your home directory. Just `cd` alone will work. Try them if you want and use `pwd` and `ls` to see the results. There are some other useful tricks with `cd` too. Use the following code to make some directories.

```
cd
mkdir unix_test
mkdir unix_test/dir1 unix_test/dir2
```

What have we done here? Well we simply created a directory with two sub directories in our home  directory - again remember we can use `~` as a shortcut for this. What we’re going to do next is navigate to `dir1` and demonstrate to ourselves the flexibility of `cd`. As often is the case, the first route is the longest…

```
cd unix_test
cd dir1
```

OK great. We’re in dir1 now. But how could we have gotten there quicker? Well first of all let’s go back to the home directory. You already know one way to do this, so let’s try another.

```
cd ..
cd ..
```

Adding the two periods after cd allows you to jump back one directory. You could also have achieved the same result using this:

```
cd ../../
```

It’s really important you remember that these two periods do this. Since we’re on the subject, a single period means the current directory and can be used in other commands too. For example

```
ls .  # will list files in the current directory
ls .. # will list files in the directory one level above
```
You can see from this `ls` and our other `cd` examples that in Unix, all operations are **relative** to the directory you are operating in.

Let’s return to `cd` and navigate back to `dir1`. This time we’ll do it the quick way.

```
cd ~/unix_test/dir1
```

All we did there was use the filepath to navigate to our directory. We could also use the same method to jump from `dir1` to `dir2`.

```
cd ~/unix_test/dir2
```
In this case, we used the **absolute filepath**.

Note that the environmental variable `$HOME` can be substituted for the `~` so actually, we could have also written this like so:

```
cd $HOME/unix_test/dir2

```
We can also use a relative filepath – i.e. the two periods – to jump back to dir1.

```
cd ../dir1
```

All this is saying is, go back one level and then enter `dir1`. So a crucial point - by default, operations in Unix are relative but we can also specify absolute paths to where we want to naviage or operate. These are important things to keep in mind when we write scripts and progress with programming in Unix.


Before we move on to the next section, use `touch` to create files - `file1` and `file2` in `dir1`.

### Wild cards and pattern matching

Earlier we briefly touched on the topic of wildcards. Pattern matching is a major strength of Unix but it can quickly get confusing. We will cover the basics for now but this is a topic we will repeatedly turn to throughout the course.

For now, let's move into `dir1` and create some files:

```
cd ~/unix_test/dir1
touch abc.txt abc.jpg xyz.txt xyz.jpg cat.txt car.txt
```

Now we’re going to use the asterisk wildcard with `ls`. When you use `*` it is basically a placeholder that says “find anything that fits this pattern”. Keep in mind though that these techniques can be used with other commands like `mv` and `cp` too.

First of all, lets show all text files.

```
ls *.txt
```

Then we’ll show all files with the name `xyz`.

```
ls xyz*
```

What about if we want to list all files except those with `xyz` in the name?

```
ls -Ixyz*
```

This example requires the `-I` flag to `ls` - i.e. ignore. This is one way to that, but you could also use more formal pattern matching which is more flexible and more powerful as it can be used with other commands such as `mv`.

```
ls [^x]*
```

Here we are essentially saying 'show me everything except things that start with x'.

We can easily extend this to make it exclude objects that do not start with x or a. Like so:

```
ls [^xa]*
```

Or only files that start with ‘c’?

```
ls c*
```

Or all files where the name contains ‘c’?

```
ls *c*
```

Finally here’s a little example of how to use something like this with copy – i.e. the `cp` command. We want to copy all `.txt` files from `dir1` to `dir2`. As we learned previously, `cp` acts much like `mv`, except it only copies files. You can still accidentally overwrite things though so beware!

```
cp *.txt ../dir2
ls ../dir2
```

### Notes and extras

For simplicity, we will not explicitly refer to the prompt in these tutorials. There are a couple of things you should remember though. Firstly the prompt character can vary (i.e. %, #, $). Secondly the prompt is really useful for getting your bearings and letting you know where you are. Finally you can customise the prompt and [make it look pretty with some nice colours](http://jamiedubs.com/ps1-collection-customize-your-bash-prompt), or get it to [say whatever you would like it to](http://www.cyberciti.biz/tips/howto-linux-unix-bash-shell-setup-prompt.html).

### Additional resources

There are a great many resources out there for learning basic Unix commands. One of the best is the [Unix Primer for Biologists](http://korflab.ucdavis.edu/unix_and_Perl/) by Keith Bradnam and Ian Korf. A large proportion of these tutorials were inspired by this resource.

Some other nice resources are the [Command Prompt for Beginners](http://lifehacker.com/5633909/who-needs-a-mouse-learn-to-use-the-command-line-for-almost-anything) at Lifehacker and the Ubuntu [UsingTheTerminal](https://help.ubuntu.com/community/UsingTheTerminal) guide. If you have experience with the MS-DOS command prompt, then this [quick translation table](http://www.yolinux.com/TUTORIALS/unix_for_dos_users.html) might also be useful.

Finally practicing Unix is straightforward for Mac OS X and Linux users but less so for those of you with Windows machines. If you’re just trying to learn the basics, you could do a lot worse than use [cygwin](http://www.cygwin.com/). Mats Töpel has a [nice tutorial on his blog](https://figshare.com/articles/Cygwin_installation_instructions/824298).

### What's next?

With some Unix basics, you're now ready to take it up a notch and learn some more advanced Unix techniques.
