# Ack Tutorial

Ack is a very flexible utility which can be used to perform searches inside
source code, plenty of programmers are tempted of using grep for doing this, but
as we will see, ack provides a much more flexible alternative to the classical grep in coding
scenarios.
This tutorial collects typical usage scenarios of ack, notice that ack is
developed in Perl, and a faster although not so flexible version written in C is
'ag', if something faster is needed, just check it out.


```sh
ack 'pattern'
# searches for that pattern in all files starting from current directory and all
# its subdirectories except .svn and .git
```

```sh
ack 'pattern' path/to/dir
# searches for that pattern in all files starting from specified  directory and all
# its subdirectories except .svn and .git by default
```

```sh
ack 'pattern' -w 
# uses word boundaries, in order to not match pattern within another string
```

```sh
ack 'pattern' --ignore-dir env/ --ignore-dir dataset/
# searches for pattern but ignores everything inside the specified directories
```

```sh
ack 'pattern' --ignore-file=ext:pod,t,py,csv,tsc
# searches for pattern but ignores all the files who have the specified
# extensions
```

```sh
ack 'pattern' --ignore-file=match:/^#.+#$/
# searches for pattern but ignores all the files who match the specified regular
# expression
```

```sh
ack 'pattern' -l 
# only prints the file names containing the pattern
```


```sh
ack 'pattern' -L 
# only prints the file names NOT containing the pattern
```


```sh
ack 'pattern' -g 
# only prints file names matching either base name or absolute path 
# with pattern
```

```sh
ack -g log --ruby
# prints all the file which have in their path/name the specified pattern
# and or of type ruby
```

```sh
ack -g log --noruby
# prints all the file which have in their path/name the specified pattern
# but do not belong to the ruby type, remember that an ack --dump
# also helps us inspecting various types
```

```sh
ack pattern -n
# only searches in current directory without going into subdirectories
```

```sh
ack file1 --match foo
# the --match option allows us to defer the specification of the match
# so that it is not the first given parameter
```

```sh
ack pattern --pager=less
# allows the view of the results through a pager without suppressing colors and
# syntax highlighting
```

```sh
ack pattern -i 
# search for pattern ignoring case
```

```sh
ack -Q pattern 
# search for pattern doing a literal search, like fgrep, so we do not take into
# account regexes
```

```sh
ack -Q aa.bb.cc.dd /path/to/access.log
# search for pattern doing a literal search, like fgrep, so we do not take into
# account regexes metacharacters
```


```sh
ack --help=types
# shows help in order to do search with respect to file types,
```

Now if from type we see that in order to search for .sh and .ksh files
we have to specify `--bash` or `--nobash` we can include that in our searches for
example:

```sh
ack pattern --nobash
```

let's see a more complicated example:
```sh
ack pattern --nopython  --ignore-dir env/ --pager=less
```

```sh
ack -f --python
# prints all the filenames of type 'python'
# this can be useful for debugging purposes especially
# when we define our own types
```


```sh
ack pattern -v 
# works like in grep, will make the negation of the pattern and search for lines
# not containing that pattern
```



```sh
ack --match 'clone.*deep|deep.*clone'  --nopython  --ignore-dir env/ --pager=less
# searches all the files which are not python and not in the env/ directory and
# who have on the same line either the sequence clone <sommething> deep or 
# deep <something> clone
```


```sh
ack -l pattern2 $(ack -l pattern1)
# files that contains pattern1 and also contain pattern2, even if they are on
# different lines.
```


```sh
ack pattern -C 4
# gives 4 lines of context around matching lines
```

```sh
ack pattern -B 4
# gives 4 lines of context around matching lines
# taking lines before the matching line
```
```sh
ack pattern -A 4
# gives 4 lines of context around matching lines
# taking lines after the matching line
```

```sh
ack pattern -c 
# prints out for each file the number of matches for the specified pattern
# if also -l is active it will only show the number of matchine lines for
# files which are matching
```


```sh
ack pattern -ch 
# prints out for each file the number of matches for the specified pattern
# with the -h option active, we suppress the filenames and only get a total
# sum of matches
```

```sh
ack pattern -ch -w --python 
# print the number of matches for the word `pattern` in all python files
```

```sh
ack pattern --files-from=filelist.txt
# searches the pattern only in the files specified in the mentioned file
```


```sh
ack --dump
# prints all the current configuration with which ack is running,
# this is very useful for debugging purposes but also to understand
# how certain options have to be set or how a type of file is defined
```

## Modifying output after search

We can specify output options for our searches, for example, let's say we want
to wrap all the words containing the letter 'e' around underscores, we can do
something like:

```sh
ack2.pl "\w*e\w*" quick.txt --output="$`_$&_$'"
```

with the `--output` option we have special expansion strings which are:

* `$&` The whole string matched by PATTERN.
* `$1, $2, ... ` The contents of the 1st, 2nd ... bracketed groups in PATTERN
* `$\`` The string before the match
* `$'` The string after the match



## Using ack like grep

We generally can substitute grep with ack in many cases, let's see some
examples:

```sh
ack 'type \w+' README.md -oh
# outputs only the text matching the specified pattern, this is very
# useful especially when we want to extract text or remove text from a file
# or more files
```

```sh
ack 'classifier \w+' ML_NOTES -c
# counts the occurrences of the specified pattern
```

```sh
ack 'classifier \w+' ML_NOTES -i
# performs a case-insensitive search of the pattern
```

Notice that the position of the flags is not important, we can also put those
just after 'ack'.


## Define a new Type

```sh
ack pattern --type-add=cpp:ext:cpp,cc,cxx,m,hpp,hh,h,hxx --cpp
# here we are defining a file type called cpp with the supported extensions
# being cpp,cc,cxx,hpp,hh,h,hxx
# this is just used as a demo example, since this type already exists by default
# in ack then we search for these files using the mentioned pattern
```


We can also define types by inspecting the first line of the file, this is
useful for example for types of files who use shebang notation `#!` and so on,
let's see an example:

```sh
ack pattern --type-add=sctv:firstlinematch:/^sctv/ --sctv
# here we are defining a file type called sctv which is characterized by files
# having as a first line starting with the string sctv
# then we search for these files using the mentioned pattern
```

## Changing Ack configuration

By default Ack tries to read `~/.ackrc` and `.ackrc` in the current directory 
as the default configuration files, if these do not exist it still has a
default configuration.  We can change Ack configuration by first printing 
its current default configuration on the standard output and saving it 
to a file.

We can do this by issuing:
```sh
ack --create-ackrc > .ackrc
# saves to a file current ackrc configuration
```

and then by editing the `.ackrc` as we wish. 
So for example by defining our own types or telling ack which directories 
to ignore and so on.


We can manually specify a configuration file to ack by using:
```sh
ack pattern --ackrc .myconf.ackrc
# loads the mentioned configuration file as last, so it will have a 
# higher priority
```

If we changed our Ack configuration file `.ackrc` we can test the changes by
executing a `ack --dump` and checking if our change is listed in the current
configuration.


## Using Ack with other programs

We can take advantage of ack capabilities to also replace strings with the help
of other programs, e.g., perl or sed.


```sh
perl -p -i -e's/this/that/g' $(ack -f --perl)
```

or with gnu sed:

```sh
sed -i 's/oldstring/newstr/gI' $(ack -f --perl)
```

notice that the BSD version of sed may not support the option `I` for
case-insensitivity.
