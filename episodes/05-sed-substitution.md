---
title: "Find... and replace! With sed."
teaching: 30
exercises: 90
---
  
:::::::::::::::::::::::::::::::::::::: questions 

- How do we find and replace content using regex patterns?

::::::::::::::::::::::::::::::::::::::::::::::::
  
::::::::::::::::::::::::::::::::::::: objectives

- Learn regex substitutions using sed.

::::::::::::::::::::::::::::::::::::::::::::::::
  
  
## Regex substitution syntax

Regex substitutions allow you to use the pattern syntax we've learned so far to describe
complex 'find & replace' operations. For the following exercises we'll be using the program
'sed', but the substitution syntax is common to other implementations too.  It's this:  

~~~bash
's/pattern/replacement/'
~~~


's' for substitution. The 'pattern' component is as we've been using with 'grep -E' so far; 
the same concept and syntax, describing a pattern to be matched. 
The 'replacement' component is what will replace a match of the pattern.  

E.g. Substitute "hello" for "goodbye" would be:  

~~~bash
's/hello/goodbye/'
~~~

  
## Sed for regex substitutions

'sed' is a Unix command line utility, name short for "stream editor". It will parse text
passed to it, from a file or piped input stream, and can perform various transformations
to the text. It has many different capabilities, as can be found in the sed manual. 
Today we'll just be playing with the regular expression substitution capability. 
We'll be using 'sed -E', which enables the same "Extended Regular Expression" syntax as
we'd been using with 'grep -E'.  

There are a few ways to use it:  

~~~bash  
# Modify stream from another program, print result
other-command | sed -E 's/pat/replace/'

# Read input file, print contents with changes
sed -E 's/pat/replace/' inFile

# Read input file, write contents with changes to another file
sed -E 's/pat/replace/' inFile > outFile

# Read input file, write with changes to the same file (caution!)
sed -E -i 's/pat/replace/' file
~~~


Example:
  
~~~bash  
echo "hello Andrew" | sed -E 's/Andrew/Bob/'
~~~

~~~output  
hello Bob
~~~


All of our regex pattern capabilities still work:  

~~~bash
echo "hello Andrew" | sed -E 's/A\w+/Bob/'
~~~

~~~output
hello Bob
~~~

  
We replace the entirety of what we match:  

~~~bash
echo "hello Andrew" | sed -E 's/.+/A whole new line/'
~~~

~~~output
A whole new line
~~~ 
  
Only the first possible extended match is replaced:  

~~~bash
echo "hello Andrew" | sed -E 's/\w+/blah/'
~~~

~~~output
blah Andrew
~~~


However, a greedy mode may be enabled which keeps looking for subsequent matches to replace, 
by adding a 'g' to the end of your substitution string:  

~~~bash
echo "hello Andrew" | sed -E 's/\w+/blah/g'
~~~

~~~output
blah blah
~~~


The replacement section is allowed to be empty too:  

~~~bash
echo "hello    Andrew" | sed -E 's/\s+//g'
~~~

~~~output
helloAndrew
~~~



::::::::::::::::::::::::::::::::::::: challenge  

## Try it 1

Using ```sed -E 's/ / /g' wordplay1.txt```...
  
1. Replace all vowels (a,e,i,o,u) with 'oo'
2. Replace all words longer than 4 characters with 'blah'
3. Replace an *entire line* if it has any words longer than 4 characters with "had long words"
(as a literal phrase)
4. Insert "Some words: " (as a literal phrase) at the start of every line.


:::::::::::::::::::::::: solution  

## Solution
  
~~~bash
sed -E 's/[aeiou]/oo/g' wordplay1.txt
sed -E 's/\w{5,}/blah/g' wordplay1.txt
sed -E 's/.*\w{5,}.*/had long words/' wordplay1.txt
sed -E 's/^/Some words: /' wordplay1.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  


## Making use of back-references

Recall with grep we could "capture" or remember a part of a match by encasing it in round 
brackets '\( \)' and then refer back to an exact copy of what was matched, using '\\1' for
the first bracketed group, '\\2' for second bracketed group, etc.. Such back-references may
also be used within the replacement section of a substitution. This is how we start doing
more interesting find and replaces.

For example, we could add an exclamation mark after every word:

~~~bash
echo "hello Andrew" | sed -E 's/(\w+)/\1!/g'
~~~

~~~output
hello! Andrew!
~~~

The back-reference is necessary, so that we can define the replacement as being "the same as
what we matched, plus an exclamation mark."

Another example, we could swap every second word:  

~~~bash
echo "one two three four" | sed -E 's/(\w+) (\w+)/\2 \1/g'
~~~

~~~output
two one four three
~~~

We separately capture two adjacent words, then in the replacement, refer back to them in reverse
order.

Remember that the entirety of a matched pattern is replaced.  So if we do something like this,
where a '.+' matches the whole rest of the line, then the whole rest of the line is replaced.

~~~bash
echo "one two three four" | sed -E 's/(\w+) (\w+).+/\2 \1/g'
~~~

~~~output
two one
~~~


::::::::::::::::::::::::::::::::::::: challenge  

## Try it 2

Using ```sed -E 's/ / /g' wordplay1.txt```...

5. Replace all vowels (a,e,i,o,u) with two copies of that same vowel
6. Switch the places of the first and second word on each line
7. Switch the places of the first and last word on each line, while retaining everything between


:::::::::::::::::::::::: solution  

## Solution
  
~~~bash
sed -E 's/([aeiou])/\1\1/g' wordplay1.txt
sed -E 's/^(\w+)\t(\w+)/\2\t\1/' wordplay1.txt
sed -E 's/^(\w+)(.+)\b(\w+)/\3\2\1/' wordplay1.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  



::::::::::::::::::::::::::::::::::::: callout

## Forward slashes

Note that, as a substitution pattern is bookended by forward slashes, there will be a conflict
if you actually want to include a literal forward slash. There are two ways of handling this.  
1. Escape your literal forward slash with a back slash, ensuring it acts as a literal character.  
``` \/ ```  
2. Alternatively, you can actually use any other character to bookend the substitution! E.g.:  
``` 's;pattern;replacement;' ```  
or  
``` 's#pattern#replacement#' ```    
If doing this, choose a character that won't otherwise appear in your substitution string and
that doesn't have any other special meaning.

:::::::::::::::::::::::::::::::::::::::::::
  


::::::::::::::::::::::::::::::::::::: challenge  

## Try it - date format conversion

Complete the following, to convert dates in namesndates.txt from dd/mm/yyyy format, into
yymmdd format.  E.g. change 23/08/2012 into 120823.  
``` sed -E 's; ; ;' namesndates.txt ```


:::::::::::::::::::::::: solution  

## Solution
>
~~~bash
sed -E 's;([0-9]{2})/([0-9]{2})/[0-9]{2}([0-9]{2});\3\2\1;' namesndates.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  


::::::::::::::::::::::::::::::::::::: challenge  

## Try it - make a FASTA file

DNA and RNA sequences are often represented in the "FASTA" format, which looks like:

~~~
>seqID
ATCGTACGTAGCTACGT
~~~


The file 'dnaSequences.txt' contains DNA sequences in a tab-separated format:

~~~
seqID    ATCGTACGTAGCTACGT
~~~


1. Use sed to convert dnaSequences.txt sequences into a FASTA format representation.
Hint: '\\n' may be used in the replacement pattern to insert a newline.
  
2. Some sequences may be very long. Can you make it so that there's never more than 20 
characters per line, with longer sequences split over multiple lines?


:::::::::::::::::::::::: solution  

## Solution

~~~bash
sed -E 's/(\S+)\s+(\w+)/>\1\n\2/' dnaSequences.txt
sed -E 's/(\S+)\s+(\w+)/>\1\n\2/' dnaSequences.txt | sed -E 's/(\w{20})/\1\n/g'
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  


::::::::::::::::::::::::::::::::::::: challenge  

## Try it - Fixing a file

Have another look at 'namesndates.txt' (```cat namesndates.txt```)
  
A number of typos and inconsistencies were made when adding rows to 'namesndates.txt':  
1. One row has a name written as "surname, firstname", instead of "firstname surname".  
2. One row has a date written in "dd-mm-yyyy" format instead of "dd/mm/yyyy" format.  
3. One row is comma-separated, while the rest are tab-separated.  
4. Some rows have multiple spaces, or a mix of tabs and spaces, in place of just single tabs.  
(```cat -T namesndates.txt``` can help distinguish tabs from spaces)  

Using multiple piped sed substitutions, can you correct all of these mistakes?  
  

:::::::::::::::::::::::: solution  

## Solution
  
~~~bash
sed -E 's/^(\w+), (\w+)/\2 \1/' namesndates.txt | \
  sed -E 's;([0-9]{2})-([0-9]{2})-([0-9]{4});\1/\2/\3;' | \
  sed -E 's/,/\t/g' | \
  sed -E 's/\s{2,}/\t/g' > namesndates-FIXED.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  


::::::::::::::::::::::::::::::::::::: callout

## Prototyping solutions

Tips for turning longer complex strings into regular expression substitutions:  
1. Start by copying a real example of a whole string into your pattern section.  
2. Add escape back slashes to any forward slashes, literal brackets, etc., as necessary.  
3. "Circle" the parts of the string you'd like to separately retain, with round brackets.  
4. Write out your replacement pattern, using back-reference to what you circled.  
5. At this stage, the substitution should work, but only for the specific real example string
that you've started with.  
6. Finally, start abstracting your search pattern, replacing parts of your example string with 
wild-cards or character-classes as needed, to strike the balance between specificity and
ambiguity required to match all that you want and not all that you don't want.  
  
:::::::::::::::::::::::::::::::::::::::::::::::

  
::::::::::::::::::::::::::::::::::::: challenge  

## Try it - information from file names

'fileExample1.txt' contains the input text from the first regex example in the intro of this
course.  Can you recreate that transformation?  
Convert this list of files (fileExample1.txt)...
  
~~~
/path/to/my/data/folder1/s1-R1_6h-shade.fasta
/path/to/my/data/folder1/s2-R1_6h-sun.fasta
/path/to/my/data/folder1/s3-R1_24h-shade.fasta
/path/to/my/data/folder1/s4-R1_24h-sun.fasta
/path/to/my/data/folder2/s1-R2_6h-shade.fasta
/path/to/my/data/folder2/s2-R2_6h-sun.fasta
/path/to/my/data/folder2/s3-R2_24h-shade.fasta
/path/to/my/data/folder2/s4-R2_24h-sun.fasta
~~~

  
...into this table of sample information:
  
~~~output
Sample1 Rep1    6hours  shade   s1-R1_6h-shade.fasta    /path/to/my/data/folder1/
Sample2 Rep1    6hours  sun     s2-R1_6h-sun.fasta      /path/to/my/data/folder1/
Sample3 Rep1    24hours shade   s3-R1_24h-shade.fasta   /path/to/my/data/folder1/
Sample4 Rep1    24hours sun     s4-R1_24h-sun.fasta     /path/to/my/data/folder1/
Sample1 Rep2    6hours  shade   s1-R2_6h-shade.fasta    /path/to/my/data/folder2/
Sample2 Rep2    6hours  sun     s2-R2_6h-sun.fasta      /path/to/my/data/folder2/
Sample3 Rep2    24hours shade   s3-R2_24h-shade.fasta   /path/to/my/data/folder2/
Sample4 Rep2    24hours sun     s4-R2_24h-sun.fasta     /path/to/my/data/folder2/
~~~


:::::::::::::::::::::::: solution  

## Solution

~~~bash
sed -E \
's/^(.+\/)(s([0-9]+)-R([0-9]+)_([0-9]+)h-(\w+)\.fasta)/Sample\3\tRep\4\t\5hours\t\6\t\2\t\1/' \
    fileExample1.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  

::::::::::::::::::::::::::::::::::::: keypoints 

-  ``` sed -E 's/pattern/replacement/' ``` 
-  ``` 's/pattern/replacement/g' ``` - enables Greedy, replace-all mode.
- Use grouping () in pattern and back-reference \\1 in replacement...
- ... to rearrange or recontextualise parts of the matched input.
- Tips for writing complex substitutions:
- 1- Start with a complete real example pasted as your pattern.
- 2- Escape '&#92;' any forward slashes, literal brackets, etc., as necessary.
- 3- Circle the parts to retain, with round brackets.
- 4- Write your replacement rules, using back-references.
- 5- Substitution should now work for your specific real example.
- 6- Abstract pattern with wildcards, etc., to make ambiguous enough for all required cases.

::::::::::::::::::::::::::::::::::::::::::::::::
  

[r-markdown]: https://rmarkdown.rstudio.com/