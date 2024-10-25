---
title: "Python regular expressions"
teaching: 10
exercises: 0
---
  
:::::::::::::::::::::::::::::::::::::: questions 

- How can we invoke regular expressions using Python?

::::::::::::::::::::::::::::::::::::::::::::::::
  
::::::::::::::::::::::::::::::::::::: objectives

- Introduce regex capabilities in Python.

::::::::::::::::::::::::::::::::::::::::::::::::
  
  
Regular expressions, with search, replace and other capabilities, are available in through
Python through the 're' library.  E.g. ```import re```.  

Patterns follow all the same syntax you've learnt using grep and sed.
However, you'll usually find that the following patterns work that didn't work with Grep:  
  
Written as | Equivalent to
----|----
\\d | [0-9] A digit
\\D | [^0-9] Not a digit
\\t | A tab character
\\n | A newline
  
  
Python regexs, using the 're' library, are implemented through functions, where search patterns
and strings are given as function arguments.  
  

## Searching

The grep equivalent is the function re.search().  
```match = re.search( pattern, string )```  
The returned match object looks "true" if there was a match and has functions for interrogating 
aspects of the match, for example individual bracketed groups matched and span range of matched 
parts.

~~~python
import re
myString = 'word1 1234 word2'
match = re.search(r'\b(\d+) (\w+)', myString)
if match:
    print(match.group(0))
    print(match.span(0))
    print(match.group(1))
    print(match.group(2))
else:
    print("No Match")
~~~

~~~output
1234 word2
(6, 16)
1234
word2
~~~


Note the 'r' in front of the search pattern, ```r'\b(\d+) (\w+)'```.  This enables the pattern 
string to be passed to the re function in its literal form, which prevents backslashes being
being interpreted as escapes too early, *before* the function looks at it.  Without the preceeding
'r', double-backslashes would be necessary.  
E.g. ```'\\b(\\d+) (\\w+)'```
  
  
## Splitting

A neat 're' feature is splitting of a string into an array of substrings, using a regex as a 
delimiter, instead of just a literal comma or tab or space, etc.. With this functionality,
you can, for example, split up a line into individual words, using a regex "anything that's 
not a word" as the delimiter for the split.

~~~python
listOfWords = re.split(r'\W+', 'word_1 $%^ 1234,word2-word3')
print(listOfWords)
~~~

~~~output
['word_1', '1234', 'word2', 'word3']
~~~
  
  
## Substituting

Substitution commands in Python 're' take the form:  
```newstring = re.sub( pattern, replacement, string )```  
Again, both the pattern and replacement/back-reference syntax is as we've learnt already.

~~~python
import re
oldstring = 'Four 123 Five'
newstring = re.sub( r'(\w+)\s+(\d+)\s+(\w+)', r'\2-\1-\3', oldstring )
print(newstring)
~~~

~~~output
'123-Four-Five'
~~~
  
::::::::::::::::::::::::::::::::::::: callout

## RE Manual
  
More reference for the Python 're' library may be found at: 
https://docs.python.org/3/library/re.html

:::::::::::::::::::::::::::::::::::::::::::  
  

::::::::::::::::::::::::::::::::::::: challenge  

## Try it - Reformatting a file using Python

Write a Python script that reads through the file 'namesndates_v2.txt' and, for each line,
rearranges it (using re.sub) to the following format, and prints the result:  

**`month-year: surname,firstname @ place`**  
  
For example, the line...  

`Neve Erindale   23/08/2012      20:57   Coombs`  

...should be printed as:  

`08-2012: Erindale,Neve @ Coombs`  

  
Hints:  
1. `\t` may be used to match the tab characters used between the fields in the input file.  
2. Basic Python file reading:  
  
~~~python
with open(filename) as inFile:
  for line in inFile:
~~~
  
:::::::::::::::::::::::: solution  

## Solution

~~~python
import re
with open('namesndates_v2.txt') as inFile:
  for line in inFile:
    rearranged = re.sub( r'^(\w+) (\w+)\t\d+/(\d+)/(\d+)\t\S+\t(.+)', r'\3-\4: \2,\1 @ \5', line )
    print(rearranged, end="")
~~~

:::::::::::::::::::::::::::::::::  
:::::::::::::::::::::::::::::::::::::::::::::::  
  
  
::::::::::::::::::::::::::::::::::::: keypoints 

- Regular expressions through Python: 
- ```import re```
- ```match = re.search(r'pattern', 'string')```
- OR
- ```list = re.split(r'pattern', 'string')```
- OR
- ```re.sub( r'pattern', r'replacement', 'string' )```
- Reference: https://docs.python.org/3/library/re.html

::::::::::::::::::::::::::::::::::::::::::::::::
  

[r-markdown]: https://rmarkdown.rstudio.com/