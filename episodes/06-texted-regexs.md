---
title: "Regexs within text editors"
teaching: 5
exercises: 0
---
  
:::::::::::::::::::::::::::::::::::::: questions 

- How can we invoke regular expressions within certain text editors?

::::::::::::::::::::::::::::::::::::::::::::::::
  
::::::::::::::::::::::::::::::::::::: objectives

- Introduce regex substitutions implemented in text editors, e.g. Nano, VSCode, Notepad++, etc...

::::::::::::::::::::::::::::::::::::::::::::::::
  


Regular expressions, for both searching and substituting, with all the capabilities we've
looked at today, are fully implemented in many modern text editors.  
  
For example, on Windows: Visual Studio Code, Notepad++, Sublime Text...
  
Patterns follow all the same syntax you've learnt using grep and sed.
However, you'll usually find that the following patterns work that didn't work with Grep:  
  
Written as | Equivalent to
----|----
\\d | [0-9] A digit
\\D | [^0-9] Not a digit
\\t | A tab character
\\n | A newline, if program supports multi-line matching
  
In some text editors, for example VSCode, back-reference variables are referred to with a '$' sign 
instead of a backslash. E.g. instead of \1 and \2, you use $1 and $2. Experiment with your favourite
text editor to see which it expects.  
  
In Nano, find and replace options are listed at the bottom of the screen as ```^W Where Is``` and 
```^\ Replace``` respectively, which in Windows corresponds to ```Ctrl+W``` for Find or ```Ctrl+\```
for Replace.  
Once either is enabled a new option for toggling Regex mode on or off is listed at the 
bottom: ```M-R Reg.exp.```. This command (Meta+R) corresponds to ```Alt+R``` on Windows.  
Toggling Regex mode on will change the prompt from ```Search:``` to ```Search [Regexp]:```.  
With Regex mode on, all the same syntax from grep and sed should work, including using \1, \2, etc. to 
reference groups captured by round brackets when doing a replace.  


::::::::::::::::::::::::::::::::::::: keypoints 

- Regular expression capabilities are incorporated in most modern text editors for find and replace.

::::::::::::::::::::::::::::::::::::::::::::::::
  

[r-markdown]: https://rmarkdown.rstudio.com/