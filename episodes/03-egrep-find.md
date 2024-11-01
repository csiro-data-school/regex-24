---
title: "Pattern matching with grep -E, part 1"
teaching: 45
exercises: 45
---
  
:::::::::::::::::::::::::::::::::::::: questions 

- How do we write regexs to match complex patterns in files or output streams?

::::::::::::::::::::::::::::::::::::::::::::::::
  
::::::::::::::::::::::::::::::::::::: objectives

- Learn the basic syntax of Extended Regular Expressions, using grep -E  

::::::::::::::::::::::::::::::::::::::::::::::::
  

You may have used the command 'grep' before, which allows you to search for a matching string.
It may be given a file name, to make it search within a file:  

~~~bash
grep 'react' wordplay1.txt
~~~
  
~~~output
class   react   exercise        impartial       heavenly
trees   shelf   amused  reactive        sheet
relation        reacting        handsome        ashamed
~~~
  
Or it can search piped output from another command:  

~~~bash
echo "Hello Andrew, you are Andrew?" | grep -o "Andrew"
~~~
  
~~~output
Andrew
Andrew
~~~
  
In this example, the '-o' flag was used to make grep print only the matches it finds, 
rather than whole matching lines. This will be a handy option when we start testing 
more ambiguous search patterns later.  


For the following lessons, we'll be making use of grep with a '-E' flag, 
which enables Extended Regular Expression (ERE) mode for searching regex patterns.
On some systems, an aliased command 'egrep' exists. The regular expression syntax
of 'grep -E' is the same as that of 'sed -E', which we'll be using for "find & replace"
commands later, and basically the same as most other implementations of regular expressions.  

  
## '\|' for matching this or that

Like in the 'if' statements of most programming languages, the pipe character acts as an OR,
allowing matching to any of the listed options.  

~~~bash
grep -E 'boat|bolt|bell' wordplay1.txt 
~~~
  
~~~output
bolt    glow    shaky   lumpy   hypnotic
bell    spiritual       materialistic   rule    sponge
exclusive       color   harbor  boat    bedroom
~~~
  

Round brackets may be used to group terms together. This grouping has multiple uses, as you'll
see as we continue, but one is to group OR options together.  E.g.:  

~~~bash
grep -E 'b(oat|olt|ell)' wordplay1.txt 
~~~
  
~~~output
bolt    glow    shaky   lumpy   hypnotic
bell    spiritual       materialistic   rule    sponge
exclusive       color   harbor  boat    bedroom
~~~
  

Consider the following:  

~~~bash
echo "ABC" | grep -E -o '(A|B)C'
~~~
  
~~~output
BC
~~~
  
Using '-o' to print matching part only, why wasn't the 'A' included in result?
The search pattern asked for either a A or a B which was followed by a C.
Only the B met this criteria.  


::::::::::::::::::::::::::::::::::::: challenge  

## Try it 1

1. Write a grep -E command to search wordplay1.txt for either 'corn' or 'cow'
2. Try the above with -w option enabled (match as whole words only)
3. Write a grep -E command to search wordplay1.txt for either 'reaction' or 'reactive'
4. Write an alternative working answer to 3.

:::::::::::::::::::::::: solution  

## Solution
  
~~~bash
grep -E 'corn|cow' wordplay1.txt
grep -E -w 'corn|cow' wordplay1.txt
grep -E 'reaction|reactive' wordplay1.txt
grep -E 'reacti(on|ve)' wordplay1.txt
~~~
  
:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  


## '[ ]' to match any single character in a list or range

This may sound familiar...

Use of square brackets `[ *list* ]` allows matching to a single character, 
where that character has to match any of the options listed between the brackets.
The brackets may contain a list, or a range, or a mix of both. 
For example, to match any one digit from 0 to 9, the following are equivalent:  

~~~bash
[0123456789]
[0-9]
[123-789]
~~~
  

Ranges work for alphabet characters too. The following are equivalent:  

~~~bash
[ABCDEFG]
[A-G]
~~~
  

Important: as always on a Unix shell, case matters!  A list of all alphanumeric characters is:  

~~~bash
[a-zA-Z0-9]
~~~
  
Example:  

~~~bash
echo "bog bag cog cot tog log lag" | grep -E -o '[a-z]og'
~~~
  
~~~output
bog
cog
tog
log
~~~
  
Example:  

~~~bash
echo "1952 1986 2003 1995 2018" | grep -E -o '20[0-9][0-9]'
~~~
  
~~~output
2003
2018
~~~
  

::::::::::::::::::::::::::::::::::::: challenge  

## Try it 2

1. Write a grep -E command to search wordplay1.txt for 4-letter words ending in 'oat'
2. What if you didn't know if the words started with lower case or capitol letters?
3. What if *any* of the letters could be upper case or lower case?
4. Write an alternative working answer to 3.

:::::::::::::::::::::::: solution  

## Solution
  
~~~bash
grep -E -w -o '[a-z]oat' wordplay1.txt
grep -E -w -o '[a-zA-Z]oat' wordplay1.txt
grep -E -w -o '[a-zA-Z][oO][aA][tT]' wordplay1.txt
# OR
grep -E -w -o -i '[a-z]oat' wordplay1.txt
# grep -i ignores case (see 'man grep')
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  

When used within square brackets, a '^' means "NOT". 
For example '[^ABC]' would match any one character that *wasn't* A or B or C.

Example:  

~~~bash
echo "bog bag cog cot tog log lag" | grep -E -o '[^b][a-z]g'
~~~
  
~~~output
cog
tog
log
lag
~~~
  

::::::::::::::::::::::::::::::::::::: challenge  

## Try it 3

1. Modify your 'oat' word finder to find any 4-letter 'oat' words other than 'boat' and 'goat'

:::::::::::::::::::::::: solution  

## Solution

~~~bash
grep -E -w -o '[^bg]oat' wordplay1.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  

## Match start of line ^ or end of line $

The '^' symbol has another use.  When placed at the start of a regex pattern, it anchors the
pattern such that it must match from the start of a line.  
Similarly, a '$', when placed at the end of a regex pattern, anchors the pattern such that it
must match up to the end of a line.  
Consider the following examples demonstrating the effect:  

~~~bash
grep -E 'repeat' wordplay1.txt
~~~
  
~~~output
repeat  drum    quilt   superficial     uncovered
copper  bad     unpack  repeat  old-fashioned
sheet   accessible      word    enthusiastic    repeat
~~~
  
~~~bash
grep -E '^repeat' wordplay1.txt
~~~
  
~~~output
repeat  drum    quilt   superficial     uncovered
~~~
  
~~~bash
grep -E 'repeat$' wordplay1.txt
~~~
  
~~~output
sheet   accessible      word    enthusiastic    repeat
~~~
  
There will be more use for these anchors when we get into more complex patterns with wildcards.

  
## '.' is the match-anything regex wildcard

Speaking of, the match-anything wildcard (any single character) for regular expressions 
is a period, '.'

Consider:  

~~~bash
grep -E -o '.oat' wordplay1.txt
~~~
  
~~~output
boat
        oat
loat
coat
goat
~~~
  
For the short word "oat", it was the preceeding tab character that the wildcard matched.


One more concept to introduce before we get into some better examples of using this wildcard.



## Matching multiples of things

What if you wanted to match something that repeated multiple times?  
Any single character, bracket expression or grouping, can be modified to specify number
of occurances, with one of the following:

Symbol(s) | Effect
----|----
? | Present either one time or *no* times (0-1)
* | Present *any* number of times, including zero (0+)
+ | Present at least once, but any number of times (1+)
{n} | Repeated exactly 'n' times
{n,m} | Repeated between 'n' and 'm' times
{n,} | Repeated at least 'n' times
{,m} | Repeated at most 'm' times


For example, what if we wanted to search for 'colour' but maybe it had US spelling?  

~~~bash
grep -E 'colou?r' wordplay1.txt
~~~
  
~~~output
exclusive       color   harbor  boat    bedroom
~~~
  
Here the '?' modifier let the preceeding 'u' match either one time OR zero times.


Lets find all words from our words file that are at least 12 letters long:  

~~~bash
grep -E -o '[a-z]{12,}' wordplay1.txt 
~~~
  
~~~output
materialistic
distribution
enthusiastic
sophisticated
~~~
  
  
::::::::::::::::::::::::::::::::::::: callout

## Pattern repetition

Note, it's probably clear from the previous example that, when specifying repetition of a 
pattern in this way, e.g. "match any letter from a to z, 12 or more times", 
it's the general ambiguous pattern concept which is repeated, not a specific matching case.
Hence we found 12 lots of "any letter" in a row, rather than "any letter" and then 12
repeats of that letter.

:::::::::::::::::::::::::::::::::::::::::::::
  
  
::::::::::::::::::::::::::::::::::::: callout

## Greediness

Regular expression search patterns are considered "greedy". That is, when you use
'+' or '\*' or '{n,}', they will always match the *longest* possible string that 
still fits the pattern.  
E.g. '.\*', which means "any character, zero or more times", will always match an entire line
without further specific context around it. 
  
::::::::::::::::::::::::::::::::::::::::::::::::  
  
  
::::::::::::::::::::::::::::::::::::: challenge  

## Try it 4

1. Use 'grep -E -o' on wordplay1.txt to match *all* of any line that starts with a 'c'  
2. Use 'grep -E -o' on wordplay1.txt to match the *first word* of lines starting with a 'c'  
3. Modify previous answer to keep words 5 or 6 letters long only.
4. Use 'grep -E -o' on wordplay1.txt to match both 'stand' and 'outstanding'

:::::::::::::::::::::::: solution  

## Solution
  
~~~bash
grep -E -o '^c.+' wordplay1.txt
grep -E -o '^c[a-z]+' wordplay1.txt
grep -E -o -w '^c[a-z]{4,5}' wordplay1.txt
grep -E -o '(out)?stand(ing)?' wordplay1.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  


What if we wanted to match a date, and didn't know if the year would be 2 digits or 4?
Our pattern for a date is "a digit, from 0-9, either one or two of them, then a forward slash,
then a digit, either one or two of them, then either 2 digits OR 4 digits."  

~~~bash
echo "11/06/91  not/a/date  5/9/2018" | \
  grep -E -o '[0-9]{1,2}/[0-9]{1,2}/([0-9]{2}|[0-9]{4})'
~~~
  
~~~output
11/06/91
5/9/2018
~~~
  

Alternatively, if we were sure about the sensibleness of the contents we were searching, 
we may get away with just:  

~~~bash
echo "11/06/91 5/9/2018" | grep -E -o '[0-9]+/[0-9]+/[0-9]+'
~~~
  
~~~output
11/06/91
5/9/2018
~~~
  

::::::::::::::::::::::::::::::::::::: challenge  

## Try it 5

Have a look at 'namesndates.txt' (```cat namesndates.txt```)  

1. Use 'grep -E -o' on namesndates.txt to list times (e.g. 20:57).  
2. Use 'grep -E -o' on namesndates.txt to list all calendar dates.  
3. Consider extra information we know about dates: days will be at most 31, months at most 12.  
    Could we use aspects of this information within our regex for more sensibleness checking?  
  
:::::::::::::::::::::::: solution  

## Solution
>
~~~bash
grep -E -o '[0-9]{1,2}:[0-9]{1,2}' namesndates.txt
grep -E -o '[0-9]{1,2}/[0-9]{1,2}/([0-9]{2}|[0-9]{4})' namesndates.txt
grep -E -o '[0-3]?[0-9]/[0-1]?[0-9]/([0-9]{2}|[0-9]{4})' namesndates.txt
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  

::::::::::::::::::::::::::::::::::::: keypoints 
  
- grep in Extended Regex mode (or egrep) allows complex pattern matching in files/streams.
- ```|``` acts as an OR between options
- ```( )``` allows grouping, e.g. for OR modifier, with quantifiers, etc..
- ```[ ]``` matches a character from a list or range of contained options
- ```[^ ]``` matches a character NOT in a list or range of contained options
- ```^``` at the start of a regex means match at start of line
- ```$``` at the end of a regex means match at end of line
- ```.``` is the match-all (any single character) wildcard
- ```?``` quantifies previous character or group as occurring zero or one time
- ```*``` quantifies previous character or group as occurring zero or more times
- ```+``` quantifies previous character or group as occurring one or more times
- ```{n,m}``` quantifies previous character or group as occurring between n and m times
- Quantifiers are greedy- will always match longest possible fit.  

::::::::::::::::::::::::::::::::::::::::::::::::
  

[r-markdown]: https://rmarkdown.rstudio.com/