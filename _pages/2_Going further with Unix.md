---
title: "Going further with Unix"
layout: archive
permalink: /going_further_unix/
---

In the last session, we learned the basics of Unix and how we can use it to create, move and manipulate files. In this session, we will delve further into Unix and its associated programming language, [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) in order to see how we can perform some quite powerful and sophisticated operations on files. First of all, we'll introduce some key command line tools and how you can use them in combination with one another.

### Combining commands with pipes

There are a huge number of different Unix command line utilities and tools. What is especially useful with Unix is how easy it is to combine them all. To do this, we need to use a pipe, denoted by a vertical bar - i.e. `|`.

We will use a very simple pipe here to demonstrate the principal but throughout the course, we will use pipes in more complicated examples, always with a detailed explanation. First of all, let's create some files.

```shell
cd ~/home/dir2
touch file1.txt file2.txt file3.txt
```
So in this instance, we know we created three text files. But what if we want to actually check this? Well we already learned that `ls` will show us the files in a directory.

```
ls *.txt
```
But this will just show the files, it won't count them for us. Of course in such a simple example, we can actually just count them ourselves - but this will rarely be practical in a proper bioinformatics context. So let's use a pipe to do the work for us.

```
ls *.txt | wc -l
```
We already saw what the first part of this command does - it lists the files. This list is then piped into `wc` which is a utility to count words and lines (i.e. word count). Last but not least, the `-l` flag tells `wc` to count lines and not words.

Pipes can make your Unix commands extremely efficient. A little later on, you will see how we can use pipes to perform all the separate components of a SNP calling pipeline with a single line of code!

### Building up your Unix toolkit

There are a huge number of extremely helpful Unix command line tools. We have already encountered some - i.e. `pwd`, `wc`, `ls` and so on. There are many more, and sadly we won't be able to cover them all here. Nontheless, when we introduce new tools throughout the course, we will do our best to explain them in detail.

For now though, we will introduce six tools which are really essential and that you will encounter numerous times. These are `head`, `tail`, `grep`, `sed`, `cut` and `awk`. Each of these tools could have a tutorial dedicated to them in their own right and they take some time to get used to - but it is worth it, they can really make your life a lot, lot easier when working with bioinformatic data!

#### git to get data

Before we introduce you to the Unix tools, we are going to explore a little trick that we will use *throughout* this course - cloning and pulling repositories from the [course github page](https://github.com/speciationgenomics). This is a way for use to easily distribute files and update exercises where necessary. Luckily, it's also extremely simple and straightforward to use. Let's clone a repository now to get some text files that we can demonstrate the Unix command line tools with.

```
# move into your home directory
cd ~
# clone the appropriate repository
git clone https://github.com/speciationgenomics/unix_exercises.git
```
When you run the `git clone` command, you will essentially clone a directory of files we have prepared (and hosted) for you on Github into your own home directory. If you use `ls` you should now see a directory called `unix_exercises`.

Take a look inside. You will see three files.

* **iris_data.tsv** - a tab separated dataset that was used by [Ronald Fisher to formulate linear discrimination analysis](https://en.wikipedia.org/wiki/Iris_flower_data_set).
* **mobydick.txt** - the entire text of [Herman Melville's](https://en.wikipedia.org/wiki/Moby-Dick) novel stored as a text file.
* **udhr.txt** - the text of the [Universal Declaration of Human Rights](https://en.wikipedia.org/wiki/Universal_Declaration_of_Human_Rights).

#### head

`head` will display the 'head' of a file - i.e. the first 10 lines by default. This command is essential if you are going to be working with bioinformatic data as often your files are millions of lines long and you just want to have a quick peek at what it contains.

For example, we can use `head` on the `udhr.txt` file to see the first 10 lines

```
head udhr.txt
```

We can also specify exactly how many lines we want to display. For example, if we want to see 20 lines:

```
head -20 udhr.txt
```

#### tail

`tail` is much the same as `head`, except it operates at the other end - i.e. it shows you the **last** ten lines of a file. It is also useful for skipping the start of a file.

First of all, let's look at the last 10 lines of the `udhr.txt` file.

```
tail udhr.txt
```
Or the last 20?

```
tail -20 udhr.txt
```
And if you would like to skip a line, we can use tail for this too.

Let's first just extract the first 10 lines of the declaration using `head`.

```
head udhr.txt > my_file.txt
```

Now if we use `tail` with `-n` flag, we can skip lines from the start of the file. For example:

```
tail -n+3 my_file.txt
```
The `-n+3` argument skips the first three lines of the file. This is very useful for removing lines you are not interested in.

#### grep

`grep` allows you to search for text data and strings. It is very useful for checking whether a pattern occurs in your data and also counting for occurences.

As an example, let's search the text of Moby-Dick for the word "whale".

```
grep --colour "whale" mobydick.txt
```
This will return every line of Moby-Dick where the word whale is mentioned. We used the `--colour` flag (**N.B.** `--color` will. work too if you want to spell it like that!) to return the results with a highlight, which makes it a bit easier to see. Feel free to compare without this flag if you'd like.

There are a couple of useful `grep` tricks here. We can return all the lines **without** the word "whale" if we want.

```
grep -v "whale" mobydick.txt
```
Here, `-v` just means invert the search. Alternatively, we can look for the word whale and print lines that come after each match.

```
grep --colour -A 2 "whale" mobydick.txt
```

The `-A 2` flag just says, print the two lines after each match. We can do the same for the lines before the match with `-B`.

```
grep --colour -B 2 "whale" mobydick.txt
```

Whereas if we use `-C`, we can get the number of lines we want either side of a match.

```
grep --colour -C 3 "whale" mobydick.txt
```

This will return 3 lines before and after each match, which is equivalent to `-B 3 -A 3`.

Finally we can also use `grep` to count occurrences of a word. How many times do you think the word "whale" appears in Moby-Dick? You can use `grep` to find out...

```
grep -c "whale" mobydick.txt
```

Actually though, this is not quite accurate since searching for `"whale"` does not include instances where the word starts with a capital letter. We can solve this by altering our search term a little:

```
grep -c "[Ww]hale" mobydick.txt
```

Here the `[Ww]` simply means we are looking for any matches where the first letter is either "W" or "w".

#### sed

`sed` is similar to grep in that it allows you to search through text and replace it. Again this makes it very powerful for altering text data very rapidly.

Let's extract some text from `mobydick.txt` to demonstrate `sed`. To do this, we will `grep` all sentences with the name "Ishmael" (the main character in the novel) on.

```
grep "Ishmael" mobydick.txt > ishmael.txt
```

Have a look at the file we created. Quite clearly, Ishmael is mentioned a lot less than the whale he is chasing... Anyway, let's have a look at what we can do with `sed`. Firstly, it is possible to look at a specific line of our text:

```
sed -n 3p ishmael.txt
```

In this command, `3p` is just telling the `-n` flag we want to see the third line. We could also extract lines 3-5 like so:

```
sed -n 3-5p ishmael.txt
```

But `sed` can actually do much more than this. For example, it can replace text. Let's replace all instances of "Ishmael" with another name, like "Dave":

```
sed 's/Ishmael/Dave/g' ishmael.txt
```

Which certainly changes the gravitas of the text. This is just a small demonstration of what it is possible to do with `sed`. It is a very useful tool, especially for file conversion and [well worth getting more familiar with](https://www.digitalocean.com/community/tutorials/the-basics-of-using-the-sed-stream-editor-to-manipulate-text-in-linux).


#### cut

`cut` is a command-line utility which allows you to cut a specific column out of a file. This is a particularly useful command for accessing specfic parts of datafiles.

`cut` is probably the more straightforward of the tools here. We can use it to get some columns of the `iris_data.tsv`.

```
cut -f 1 iris_data.tsv | head
```
Note that we piped the output to `head` to make it clearer. We could also extract multiple columns:

```
cut -f 3,5 iris_data.tsv | head
```

Note that `cut` expects a tab-delimited file by default. So if your file is comma-separated or uses spaces, you need to use the `-d` flag to specify it. You can see examples (and more information on `cut` by using `man cut` to view the manual. Note that this works for most command-line tools too.

#### awk

`awk` is not so much a command-line tool, rather a full programming language. It is extremely flexible and can actually be used to do many of the things the previous commands do too. However, it is particularly useful for in-line editing and converting file formats. We can get an idea of how it works with the `iris_data.tsv`.

For example, let's use it in a similar way to `cut` and just print a single column of the data.

```
awk '{print $1}' iris_data.tsv | head
```

`awk` essentially iterates through each row of the file we provided it. So we can also get it to print additional values next to the column we extract. For example:

```
awk '{print $1,"\t"50}' iris_data.tsv | head
```

Here we added a tab space with `"\t"` and told `awk` to print 50 for each row.

We could also do something like add 1 to each value of a specific column. For example:

```
awk '{print $3,"\t"$3+1}' iris_data.tsv | head
```
Here we printed column 3 and also column 3 but with 1 added to all the values in it.

`awk` can do even more than this. It is definitely worth checking out [a few tutorials on it](http://www.hcs.harvard.edu/~dholland/computers/awk.html). We will leave our introduction to `awk` and the other tools here - but we will return to them and their uses throughout our bioinformatics training.

### Bash - a Unix based programming language

Unix is the operating system we are working in with the command line, but we interact with Unix using the **bash programming language**. Bash is not the prettiest or most straightforward of languages to use but it can be exceptionally powerful, even in simple cases.

Since it is a programming language, more often than not we will be writing **bash scripts**. We will deal with this shortly but beforehand, it is worth learning about some fundamental features of the bash language. These will help us build towards creating efficient and effective scripts.

#### Declaring variables

Variables in programming languages are just things that we refer to in the environment we are working in. One way to think of them is like naming objects in real life. Imagine you want to tell a (very literal) colleague to move a table for you. Now, imagine trying to do that without a word for table. You might be able to get them to do what you want each time by saying, "OK, turn the four-legged object with a horizontal flat surface 90 degrees and carry it over there" or something similar, but it would be a lot easier if you could just say "Move the table".

This is basically what you are doing when declaring variables. In otherwords, you can think of a variable as short hand for something you want your code to refer to. In bash, we declare a variable like so:

```
EXAMPLE="Hello world"
```

Note that the variable doesn't **have** to be allcaps but that this is the convention in bash. It is worth sticking to this convention because all command line tools have lowercase names. This makes your code much easier to read (which trust us, is extremely important when you come back to it!)

Recalling variables is also simple, we just precede our variable name with `$`. To print it to the screen, we must use the utility `echo`:

```
echo $EXAMPLE
```
We can also declare multiple variables and combine them together:

```
NAME="my name is Hal"
echo ${EXAMPLE} ${NAME}
```
Note that we wrap the variable names here in curly brackets in order to preserve them. This is not always necessary but it does ensure your code is interpreted properly. This makes more sense if we combine them in a string, i.e. to make them a full sentence.

```
echo "${EXAMPLE}, ${NAME}"
```
Although these examples are actual words, more often than not, you will use variables to store the names of files. Using variables in your scripting is a really useful way to make your code very efficient. Doing this means you can change what an entire script does just by changing a single variable.

One last note on variables - we have actually already encountered one before this section of the tutorial - i.e. the `$HOME` variable. This is one of multiple **environmental variables** which are stored in our Unix environment. You can see all of them using the `env` command. It is best to **not** set any variables with the same names as these.

#### String manipulation

Now that we have learned to create variables, we can also explore how to manipulate them. This is not always straightforward in bash, but again it is really worth learning how to do this as it can make script writing much more straightforward. In most cases we will use string manipulation on filenames and paths, so we will use this as an example now.

First, let's declare a variable. We'll make a dummy filename in this instance:

```
FILE="$HOME/an_example_file.txt"
```
Let's echo this back to the screen:

```
echo $FILE
```
An important point to note here is that the `$HOME` variable has been interpreted so that we now have the entire file path.

Let's say we just want the actual filename, i.e. without the directory or path? We can use `basename`:

```
basename $FILE
```
Alternatively, we could remove the filename and keep only the directory or path:

```
dirname $FILE
```
For now though, we want to operate on the filename itself, so let's redeclare the variable so it is *only* the filename.

```
FILE=$(basename $FILE)
echo $FILE
```
Note that here we have to wrap the `basename $FILE` command in `$()` because it is an actual command.

OK so onto some proper string manipulation. Let's remove the `.txt` suffix.

```
echo ${FILE%.*}
```
What did we do here? First we have to wrap the entire variable name in curly brackets - this will not work without them. The `%` denotes that we want to delete everything after the next character, which in this case is `.*` - i.e. everything after the period. Note that the following would have also worked:

```
echo ${FILE%.txt}
```

We don't need to limit ourselves to the suffix. We could also delete everything after the last underscore. Like so:

```
echo ${FILE%_*}
```
We could also set it so that we delete everything after the *first* underscore:

```
echo ${FILE%%_*}
```
 We can also delete from infront of the characters in our string manipulation example. For example:

```
echo ${FILE#*.}
```
This deletes everything up to and including the period character. We could also do the same with the underscores:

```
echo ${FILE#*_}
echo ${FILE##*_}
```

Where again, a single `#` states we want to delete only after the last occurrence and a double `##` denotes we want to delete everything after the first occurrence.

You might be wondering, what exactly is the point of this? Well altering filenames is very important in most bioinformatics pipelines. So for example, with simple string manipulation you can change the suffix of a filename quickly and easily:

```
echo $FILE
echo ${FILE%.*}.jpg
```
One last point here; string manipulation in bash is not straightforward. It takes a lot of practice to get right and remember properly. We google [this excellent tutorial](https://www.tldp.org/LDP/abs/html/string-manipulation.html) nearly all the time!

#### Bash control flow

[Control flow](https://en.wikipedia.org/wiki/Control_flow) is an important part of many different programming languages. It is essentially a way of controlling how code is carried out.

Imagine you have to perform the same operation on many different files - do you want to type out a command for each and everyone of them? Of course not! This is why you might use control flow to repeat a command multiple times. There are many different types of control flow, but for now we will focus on the most common one - a `for` loop.

Let's have a look at a simple example:

```
for i in {1..10}
do
echo "This is $i"
done
```

All this is is saying is that for each number between 1 and 10, echo a "This is 1", "This is 2" and so on to the screen. `do` and `done` initiate and stop the loop respectively.  Here, the variable `i` is used within the loop but this is completely arbitrary - you can use whatever variable you would like. Indeed, it is often much more convenient to use a variable that makes sense to you. For example:

```
for NUMBER in {1..10}
do
echo "This is $NUMBER"
done
```
It doesn't just have to be numbers either. You can use a loop to iterate across multiple strings too. For example:

```
for NAME in Mario Link Luigi Peach Zelda
do
echo "My name is $NAME"
done
```

Of course, this is a silly example, but you could easily substitute this with filenames - making it quite clear why control flow is an essential skill for effective bash programming in bioinformatics.

#### Declaring arrays

Imagine we want to run a `for` loop on some text files. We'll make five of them to demonstrate - and we can actually do this with a `for` loop too:

```
for i in {1..5}
do
touch file_${i}.txt
done
```
Use `ls` after running this code and you'll see five text files.

Now, what if we want to do something simple like go through all of them and print their names to the screen? We could do it like this:

```
for FILE in *.txt
do
echo $FILE
done
```
This works really well for this simple example. However as your code becomes more advanced, it is easy for something like this to become quite dangerous. Imagine for example that in our `for` loop, we create a new `.txt` file each time? We would be in danger of creating an infinite loop, that continually prints the names of the new files it creates.

For this reason, it is **best practice** in bash to use **arrays**. These are essentially predefined lists of variables. They are easy to make too. Let's try a simple example.

```
ARRAY=(Link Zelda Gannon)
```
Now we can try printing this to the screen:

```
echo $ARRAY
```
This only prints the first value of our array. Actually, arrays have indexes, so we can print any value we specify like so:

```
echo ${ARRAY[0]}
echo ${ARRAY[1]}
echo ${ARRAY[2]}
```
Notice that like python (and unlike R) everything in bash is zero-indexed - i.e. the first variable is zero and so on.

What if we want to print everything in the array?

```
echo ${ARRAY[@]}
echo ${ARRAY[*]}
```
Either of these will work fine.

We can also loop through the array, like so:

```
for CHARACTER in ${ARRAY[@]}
do
echo $CHARACTER
done
```

The purpose of the array here is that it ensures the **scope** of our loop is limited and that it doesn't get carried away, operating on things it shouldn't do.

One last point about arrays - it is often quite cumbersome to define them by hand. Imagine if you wanted to make an array for hundreds of files? Luckily you can also *declare* them from for loops too:

```
ARRAY2=($(for i in *.txt
do
echo $i
done))
```

This doesn't look very neat though - you can actually write a for loop like this on a single line - i.e.:

```
ARRAY2=($(for i in *.txt ;do echo $i; done))
```
Where `;` indicates a separate line (as seen in the above example).

The convenience of arrays, like much of this tutorial will become much more apparent as you become more experienced in using Unix for bioinformatics.

### Writing a bash script

So far we have learned a lot about bash as a programming language - but can we use it to write a program? Well actually... yes! This is extremely easy and it is exactly what we set out to do when we write a bash script.

Let's start with a really basic example. Type `nano` into the command line in order to open the `nano` text editor.

Then we can write a simple bash script, like so:

```
#!/bin/sh

# a simple bash script
echo "Hello world"

exit
```

Save it as `my_first_script.sh`. You can take another look at the output with `cat` or `less` if you want to check you saved it properly.

Let's breakdown some of the script. First of all there is this `#!/bin/sh` line. You don't need to worry too much about that - it's just good practice to ensure the script is run in the bash language. We also have another line starting with `#` - this is just a comment. Here it explains something about the script. Comments are really important actually and you should fill your script with them - they are a good way of letting yourself know what you have done. Again, they can be invaluable when you come back to your scripts after some time away...

Now we can actually run the script. We do that like so:

```
sh my_first_script.sh
```

You just wrote your first program! [What it said is also quite relevant](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program).

You can also write a script that is interactive. Let's write another script to take an input from the command line.

Open `nano` and create a script with the following:

```
#!/bin/sh

# a simple bash script with input
echo "My name is ${1} and my best friend is ${2}"

exit
```

Save it as `name_script.sh`. In this script, the `${1}` and `${2}` variables are just specifying that this script will take the first and second arguments to the script from the command line. Let's see it in action.

```
sh name_script.sh Mark Milo
```

Feel free to add whatever combination you want in here. You can actually try running this without the arguments too and see that it still works, just the output doesn't make much sense.

### A more serious scripting example

Now we have learned a little about how to script, let's write one that will do something for us. We'll create five files (again using a `for` loop) and then convert them all from `.txt` to `.jpg`.

Firstly, let's make those files:

```
for i in {1..5}
do
touch file_${i}.txt
done
```

Now, we can open up `nano` and write out script. We would do this like so:

```
#!/bin/sh

# a script to rename files

# declare an array
ARRAY=($(for i in file*.txt; do echo $i; done))

# loop over array
for FILE in ${ARRAY[@]}
do
echo "Creating ${FILE%.*}.jpg"
mv $FILE ${FILE%.*}.jpg
done
```
Now write this script out as a `file_renamer.sh`.

If you run this as `sh file_renamer.sh` - it will print the name of each file it converts to the screen. You can then use `ls` to see that it has indeed converted all the `.txt` files to `.jpg`.

### A script writing challenge

With all the skills we have learned in this tutorial, it is now time for you to put them to the test. Return to the `unix_exercises` directory you created when you used `git clone` and write a short script to do the following:

* make an array of the three files
* loop through the array and count the number of lines in the file
* print the number of lines and the name of the file to the output

<details><summary>Click here to see a possible solution.</summary>
<p>

```
#!/bin/sh

# a possible solution script

# declare an array
ARRAY=($(for i in *.t*; do echo $i; done))

# loop over array
for FILE in ${ARRAY[@]}
do
echo "${FILE}"
wc -l ${FILE}
done
```

</p>
</details>
