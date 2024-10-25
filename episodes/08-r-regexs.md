---
title: "R regular expressions"
teaching: 10
exercises: 0
---
  
:::::::::::::::::::::::::::::::::::::: questions 

- How can we invoke regular expressions using R?

::::::::::::::::::::::::::::::::::::::::::::::::
  
::::::::::::::::::::::::::::::::::::: objectives

- Introduce regex capabilities in R.

::::::::::::::::::::::::::::::::::::::::::::::::
  
  
Regular expressions, with search, replace capabilities, are also available in R.
Again, patterns follow all the same syntax you've learnt using grep and sed.
And again, you'll find that the following patterns work that didn't work with Grep:  
  
Written as | Equivalent to
----|----
\\d | [0-9] A digit
\\D | [^0-9] Not a digit
\\t | A tab character

  
There are multiple ways of utilising regular expressions through R, but we're going to focus on
a set of functions provided by the 'stringr' package:  

~~~r
library(stringr)
# We'll also load tidyverse for later
library(tidyverse)
~~~
  
The grep equivalent in 'stringr' is `str_detect`. It's intended for use in searching vectors of strings.  

```str_detect( string.vector, 'pattern' )```  

Basic use returns a list of TRUE/FALSE for which vector entries matched the search pattern.  
  
**Note that you'll need to use double-backslashes in place of any single backslash in R.**  
  
For example, find words where the same letter is repeated consecutively:  

~~~r
words <- c("apple", "banana", "carrot")
str_detect( words, '(\\w)\\1' )
~~~

~~~output
[1]  TRUE FALSE  TRUE
~~~


Applying the output of str_detect back to the original string vector, as indices, allows access to the actual matched entries:  

~~~r
words[ str_detect( words, '(\\w)\\1' ) ]
~~~

~~~output
[1] "apple"  "carrot"
~~~
  
  
'str_detect' can also be used in conjunction with data frames. Within a data frame, each column is stored as a separate vector
and so we can use 'str_detect' within any function that accesses a single data frame column (such as mutate()).  
  
For the next example, we're going to make use of the 'stringr' package coming pre-loaded with a longer list of words, provided 
exactly for demonstrations like this, named 'fruit'.  Check it out:  

~~~r
fruit
length(fruit)
~~~
  
First we'll convert the fruit vector into a data frame:  
  
~~~r
framed_fruit <- tibble(fruit_name = fruit)
framed_fruit
~~~
  
We can then add a new column to the data frame, which is the result of using 'str_detect' to find our repeated letter pattern:  
  
~~~r
framed_fruit %>% 
  mutate(has_double = str_detect(fruit_name, '(\\w)\\1'))
~~~
  
~~~output
# A tibble: 80 × 2
   fruit_name   has_double
   <chr>        <lgl>     
 1 apple        TRUE      
 2 apricot      FALSE     
 3 avocado      FALSE     
 4 banana       FALSE     
 5 bell pepper  TRUE      
 6 bilberry     TRUE      
 7 blackberry   TRUE      
 8 blackcurrant TRUE      
 9 blood orange TRUE      
10 blueberry    TRUE      
# ... 70 more rows
~~~
  
Another approach is to use the 'filter' function to reduce the data frame to only matching rows:  
  
~~~r
framed_fruit %>% 
  filter( str_detect(fruit_name, '(\\w)\\1') )
~~~
  
~~~output
# A tibble: 29 × 1
   fruit_name  
   <chr>       
 1 apple       
 2 bell pepper 
 3 bilberry    
 4 blackberry  
 5 blackcurrant
 6 blood orange
 7 blueberry   
 8 boysenberry 
 9 cherry      
10 chili pepper
# ... 19 more rows
~~~  
  
  
---
  

Substitution in R is enabled through the 'str_replace' or 'str_replace_all' functions.  

```str_replace( string.vector, 'pattern', 'replacement' )```  
  
These functions will complete substitutions in all matching entries in a string vector. 
The difference is that 'str_replace_all' will also perform greedy matching and replacing within
individual vector entries, where 'str_replace' will only replace the first match in each vector
entry.  The functions return the modified string vector.
  
~~~r
str_replace(words, '(\\w)\\1', '\\1a\\1a\\1')
~~~
  
~~~output
[1] "apapaple"  "banana"    "carararot"
~~~
  
And again, these functions may be combined with data frame use too.  For example:  
  
~~~r
framed_fruit %>% 
  mutate( long_vowels = str_replace_all(fruit_name, '([aeiou])', '\\1\\1\\1\\1\\1') )
~~~  
  
~~~output
# A tibble: 80 × 2
   fruit_name   long_vowels                     
   <chr>        <chr>                           
 1 apple        aaaaappleeeee                   
 2 apricot      aaaaapriiiiicooooot             
 3 avocado      aaaaavooooocaaaaadooooo         
 4 banana       baaaaanaaaaanaaaaa              
 5 bell pepper  beeeeell peeeeeppeeeeer         
 6 bilberry     biiiiilbeeeeerry                
 7 blackberry   blaaaaackbeeeeerry              
 8 blackcurrant blaaaaackcuuuuurraaaaant        
 9 blood orange blooooooooood oooooraaaaangeeeee
10 blueberry    bluuuuueeeeebeeeeerry           
# ... 70 more rows
~~~
    
  
::::::::::::::::::::::::::::::::::::: callout

## Documentation  

More reference for the R stringr functions may be found here:    
https://www.rdocumentation.org/packages/stringr/versions/0.3/topics/str_detect  
https://www.rdocumentation.org/packages/stringr/versions/0.3/topics/str_replace  
https://www.rdocumentation.org/packages/stringr/versions/0.3/topics/str_replace_all  
  
Older alternate functions that achieve the same result:  
https://www.rdocumentation.org/packages/base/versions/3.5.1/topics/grep  
https://www.rdocumentation.org/packages/base/versions/3.5.1/topics/regex  
  
:::::::::::::::::::::::::::::::::::::::::::  
  
  
::::::::::::::::::::::::::::::::::::: keypoints 

- Regular expressions through R:
- ```str_detect( string.vector, 'pattern' )```
- ```str_replace( string.vector, 'pattern', 'replacement )```
- ```str_replace_all( string.vector, 'pattern', 'replacement )``` for 'greedy' match & replace.
- Need to double escape ```\\``` any back slashes in patterns.

::::::::::::::::::::::::::::::::::::::::::::::::
  

[r-markdown]: https://rmarkdown.rstudio.com/
