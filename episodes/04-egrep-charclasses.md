---
title: "Pattern matching with grep -E, part 2"
teaching: 30
exercises: 30
---
  
:::::::::::::::::::::::::::::::::::::: questions 

- How do we use predefined character classes for more complex search patterns?

::::::::::::::::::::::::::::::::::::::::::::::::
  
::::::::::::::::::::::::::::::::::::: objectives

- Learn to incorporate predefined character classes into regular expressions.

::::::::::::::::::::::::::::::::::::::::::::::::
  
  
## Predefined character classes

The Extended Regular Expression syntax has a number of predefined character groupings that may
be written as a word, rather than a collection or range of characters.  
For example, you may write `[[:digit:]]` instead of writing `[0-9]`.  
  
Written as | Equivalent to
----|----
[[:digit:]] | [0-9]
[[:alpha:]] | [a-zA-Z]
[[:alnum:]] | [a-zA-Z0-9]
[[:upper:]] | [A-Z]
[[:lower:]] | [a-z]
[[:space:]] | Spaces, tabs, in some contexts new-lines
[[:graph:]] | Any printable character other than space
[[:punct:]] | Any printable character other than space or [a-zA-Z0-9]

::::::::::::::::::::::::::::::::::::: callout
  
## Why?
 
Why would you write `[[:digit:]]` instead of writing `[0-9]`? Or `[[:upper:]]` instead 
of `[A-Z]`?  In general, for our use, there's little difference other than readability/style. 
The difference comes if you need to make things more universal, as Arabic-numerals are not the
only number system in use around the world and the 26-letter English alphabet is not
the only writing system. So, for example, while `[0-9]` will always just be those
specific 10 characters, `[[:digit:]]` may have multiple alternate numeric systems encoded.  

::::::::::::::::::::::::::::::::::::::::::::::::  
  

## Regex shorthand escape symbols

The other way you may refer to predefined character classes, and the way you will likely
most commonly do so from here on, is using the following shorthands, formed by "escaping"
certain characters with a backslash.  For example '\\w' can be used to match to any 
"word" character, which means any letter, number, or (counterintuitively) an underscore.  
The shorthand symbols available are:

Written as | Equivalent to
----|----
\\w | "Word" character [a-zA-Z0-9] OR a _ (underscore)
\\W | [^\w] Inverse of \\w, any non-"word" character
\\s | Spaces, tabs, in some contexts new-lines
\\S | [^\s] Inverse of \\s, any non-space character
\\b | Boundary between "words" and "spaces" (0-length)
\\B | [^\b] In the middle of a "word" or multiple "spaces" (0-length)
\\< | Boundary at *start* of "word" between "words" and "spaces" (0-length)
\\> | Boundary at *end* of "word" between "words" and "spaces" (0-length)

Those last four are considered "anchors". They don't actually match characters, but they can
give a regex pattern more context, helping to orientate components. For example, a letter
being specifically at the start of a word.

The following are also commonly used within regex syntax (e.g. will work in Python or R),
but **are not understood by grep or sed**:

Written as | Equivalent to
----|----
\\d | [0-9] A digit
\\D | [^0-9] Not a digit
\\t | A tab character (does work in some version of sed, test yours)
\\n | A newline, if program supports multi-line matching



Here are some examples.  
The first two words of a line (start of line, word, space(s), word):  

~~~bash
echo 'word1 word_2 thirdWord' | grep -E -o '^\w+\s+\w+'
~~~

~~~output
word1 word_2
~~~


A word with spaces at both ends:  

~~~bash
echo 'word1 word_2 thirdWord!?' | grep -E -o '\s\w+\s'
~~~

~~~output
 word_2 
~~~


Every set of consecutive non-space characters:  
  
~~~bash
echo 'word1 word_2 thirdWord!?' | grep -E -o '\S+'
~~~

~~~output
word1
word_2
thirdWord!?
~~~


Everything up to the boundary of the last word:  

~~~bash
echo 'word1 word_2 thirdWord!?' | grep -E -o '.+\<'
~~~

~~~output
word1 word_2  
~~~


The middle characters of each word (bounded by not-a-word-boundary):  

~~~bash
echo 'word1 word_2 thirdWord!?' | grep -E -o '\B\w+\B'
~~~

~~~output
ord
ord_
hirdWor
~~~
  
  
::::::::::::::::::::::::::::::::::::: challenge  

## Try it 1

1. Use 'grep -E -o' on wordplay1.txt to print the first 2 words of any line using  
  a. [[:alpha:]] and [[:space:]]  
  b. \\w and \\s  
  c. \\S and \\s  
2. Use 'grep -E -o', with \\w and \\b, on wordplay1.txt to print a word that 
starts with 'p' and ends with 'g'.  
3. Use 'grep -E', with \\w and \\<, on wordplay1.txt to highlight the first 
letter of every word.  


:::::::::::::::::::::::: solution  

## Solution
  
~~~bash
grep -E -o '^[[:alpha:]]+[[:space:]]+[[:alpha:]]+' wordplay1.txt 
grep -E -o '^\w+\s+\w+' wordplay1.txt  
grep -E -o '^\S+\s+\S+' wordplay1.txt
grep -E -o '\bp\w+g\b' wordplay1.txt
grep -E '\<\w' wordplay1.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  
  
::::::::::::::::::::::::::::::::::::: callout

## Tab characters
 
There are a few options for getting a tab character to work with grep or sed on a bash
command line (if a space '\\s' or word boundary anchor will not be specific enough):  
1. On some systems, pressing ctrl+v followed by tab, will insert a literal tab character.  
2. On some systems, a literal tab could be copy and pasted in from a text editor.  
3. A dollar sign in front of pattern can enable escape character interpretation in bash,
based on ANSI-C rules, where '\\t' is a tab.  
E.g. ``` echo $'|\t|' ```  OR  ``` grep -E $'\t' ```  
This last option is neatest, but beware other conflicting escape character interpretations
in this mode, meaning that to use '\\w' or '\\b' etc., you will need to double-escape
them, with two slashes.  
E.g. for a word with tabs either side: ``` grep -E $'\t\w+\t' ```  

:::::::::::::::::::::::::::::::::::::::::::
  

## Capturing groups and back-references

We'd mentioned earlier that round brackets ( ) have multiple uses. One use is to "capture"
a match seen within the round brackets, remembering the contents of what matched
within, such that a copy of the contents may be referred to again. We'll make much use of this 
feature in the next lesson, for find-n-replace substitutions! 
Within grep though, a "back-reference" to a captured group can be used to identify something
that repeats identically within a line. The reference to the previously seen item is used in the
form '\\number'. It works like this:  

~~~bash
echo 'blah1 blah2 blah2 blah4?' | grep -E -o '(\w+)\s+\1'
~~~

~~~output
blah2 blah2
~~~


Here a word '\\w+' is "captured" by the round brackets, is followed by space, then is referenced 
again by '\\1', which represents a stored copy of what was originally matched by the '\\w+'. 
Hence this grep only matches a case where a word is followed by another copy of that same word.  

If we'd like to make multiple back-references, we use multiple pairs of round brackets and 
increment our back-reference number by one for each open bracket from left-to-right.  E.g.:  

~~~bash
echo 'blah1 blah2 blah2 blah4?' | grep -E -o '([a-z]+)([0-9])\s+\1\2'
~~~

~~~output
blah2 blah2
~~~


We had the same outcome, but stored letters part "blah" separate to the digits part "2", then
referred to the two captured parts as '\\1' and '\\2' respectively.  

Consider the following:  
  
~~~bash
echo 'ABCDEFGGFEDCBA' | grep -E -o '(\w)(\w)(\w)\3\2\1'
~~~

~~~output
EFGGFE
~~~

Our pattern matched to three letters, then to those same three letters repeated again, but 
*in reverse order*!  



::::::::::::::::::::::::::::::::::::: challenge  

## Try it 2

4. Grep 'wordplay1.txt' to print the only line that contains the same word repeated twice.
Hint: You may need to use the '\\b' word-boundary anchor.
5. Grep 'namesndates.txt' to print the name of a person whose firstname and surname start
with the same letter.
6. Grep 'namesndates.txt' to print a date where the month and the day are the same number.


:::::::::::::::::::::::: solution  

## Solution
  
~~~bash
grep -E '\b(\w+)\b.+\b\1\b' wordplay1.txt
grep -E -o '^(\w)\w+\s\1\w+' namesndates.txt
grep -E -o '([0-9]{2})/\1/[0-9]{4}' namesndates.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  
  
::::::::::::::::::::::::::::::::::::: keypoints 

- grep in Extended Regex mode has a number of predefined character classes:
-  ``` [:alpha:]  [:alnum:]  [:digit:]  [:upper:]  [:lower:]  [:punct:]  [:space:] ``` 
- and escape-character enabled shorthand character classes and anchors:
-  ``` \w ``` : Word character [a-zA-Z0-9] OR a _ (underscore)
-  ``` \W ``` : ```[^\w]``` Inverse of \\w, any non-word character
-  ``` \s ``` : Spaces, tabs, in some contexts new-lines
-  ``` \S ``` : ```[^\s]``` Inverse of \\s, any non-space character
-  ``` \b ``` : Boundary between adjacent word and space, 0-length anchor
-  ``` \B ``` : ```[^\b]``` In the middle of a word or multiple spaces, 0-length anchor
-  ``` \< ``` : Boundary at *start* of word between word and space, 0-length anchor
-  ``` \> ``` : Boundary at *end* of word between word and space, 0-length anchor
- You can refer back to an exact copy of a matched (group) using \\1, \\2, etc..

::::::::::::::::::::::::::::::::::::::::::::::::
  

[r-markdown]: https://rmarkdown.rstudio.com/