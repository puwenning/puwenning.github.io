---
title: An Introduction to xfun
subtitle: A Collection of Miscellaneous Functions
author: "Yihui Xie"
date: "2018-07-02"
slug: xfun
githubEditURL: https://github.com/yihui/xfun/edit/master/vignettes/xfun.Rmd
output:
  knitr:::html_vignette:
    toc: yes
vignette: >
  %\VignetteIndexEntry{An Introduction to xfun}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---



After writing about 20 R packages, I found I had accumulated several utility functions that I used across different packages, so I decided to extract them into a separate package. Previously I had been using the evil triple-colon `:::` to access these internal utility functions. Now with **xfun**, these functions have been exported, and more importantly, documented. It should be better to use them under the sun instead of in the dark.

This page shows examples of a subset of functions in this package. For a full list of functions, see the help page `help(package = 'xfun')`. The source package is available on Github: https://github.com/yihui/xfun.

## No more partial matching for lists!

I have been bitten many times by partial matching in lists, e.g., when I want `x$a` but the element `a` does not exist in the list `x`, it returns the value `x$abc` if `abc` exists in `x`. This is [very annoying to me](https://twitter.com/xieyihui/status/782462926862954496). The function `xfun::strict_list()` makes a list "strict" by disabling the partial matching of the `$` operator, e.g.,


```r
library(xfun)
(z = strict_list(aaa = "I am aaa", b = 1:5))
```

```
## $aaa
## [1] "I am aaa"
## 
## $b
## [1] 1 2 3 4 5
```

```r
z$a  # NULL (strict matching)
```

```
## NULL
```

```r
z$aaa  # I am aaa
```

```
## [1] "I am aaa"
```

```r
z$b
```

```
## [1] 1 2 3 4 5
```

```r
z$c = "you can create a new element"

z2 = unclass(z)  # a normal list
z2$a  # partial matching
```

```
## Warning in z2$a: partial match of 'a' to 'aaa'
```

```
## [1] "I am aaa"
```

Similarly, the default partial matching in `attr()` can be annoying, too. The function `xfun::attr()` is simply a shorthand of `attr(..., exact = TRUE)`.

I want it, or I do not want. There is no "I probably want".

## Output character vectors for human eyes

When R prints a character vector, your eyes may be distracted by the indices like `[1]`, double quotes, and escape sequences. To see a character vector in its "raw" form, you can use `cat(..., sep = '\n')`. The function `raw_string()` marks a character vector as "raw", and the corresponding printing function will call `cat(sep = '\n')` to print the character vector to the console.


```r
library(xfun)
raw_string(head(LETTERS))
```

```
A
B
C
D
E
F
```

```r
(x = c("a \"b\"", "hello\tworld!"))
```

```
[1] "a \"b\""       "hello\tworld!"
```

```r
raw_string(x)  # this is more likely to be what you want to see
```

```
a "b"
hello	world!
```

## Print the content of a text file

I have used `paste(readLines('foo'), collapse = '\n')` many times before I decided to write a simple wrapper function `xfun::file_string()`. This function also makes use of `raw_string()`, so you can see the content of a file in the console as a side-effect, e.g.,


```r
f = system.file("LICENSE", package = "xfun")
xfun::file_string(f)
```

```
YEAR: 2018
COPYRIGHT HOLDER: Yihui Xie
```

```r
as.character(xfun::file_string(f))  # essentially a character string
```

```
[1] "YEAR: 2018\nCOPYRIGHT HOLDER: Yihui Xie"
```

## Search and replace strings in files

I can never remember how to properly use `grep` or `sed` to search and replace strings in multiple files. My favorite IDE, RStudio, has not provided this feature yet (you can only search and replace in the currently opened file). Therefore I did a quick and dirty implementation in R, including functions `gsub_files()`, `gsub_dir()`, and `gsub_ext()`, to search and replace strings in multiple files under a directory. Note that the files are assumed to be encoded in UTF-8. If you do not use UTF-8, we cannot be friends. Seriously.

All functions are based on `gsub_file()`, which performs searching and replacing in a single file, e.g.,


```r
library(xfun)
f = tempfile()
writeLines(c("hello", "world"), f)
gsub_file(f, "world", "woRld", fixed = TRUE)
file_string(f)
```

```
hello
woRld
```

The function `gsub_dir()` is very flexible: you can limit the list of files by MIME types, or extensions. For example, if you want to do substitution in text files, you may use `gsub_dir(..., mimetype = '^text/')`.

**WARNING**: Before using these functions, make sure that you have backed up your files, or version control your files. The files will be modified in-place. If you do not back up or use version control, there is no chance to regret.

## Manipulate filename extensions

Functions `file_ext()` and `sans_ext()` are based on functins in **tools**. The function `with_ext()` adds or replaces extensions of filenames, and it is vectorized.


```r
library(xfun)
p = c("abc.doc", "def123.tex", "path/to/foo.Rmd")
file_ext(p)
```

```
## [1] "doc" "tex" "Rmd"
```

```r
sans_ext(p)
```

```
## [1] "abc"         "def123"      "path/to/foo"
```

```r
with_ext(p, ".txt")
```

```
## [1] "abc.txt"         "def123.txt"      "path/to/foo.txt"
```

```r
with_ext(p, c(".ppt", ".sty", ".Rnw"))
```

```
## [1] "abc.ppt"         "def123.sty"      "path/to/foo.Rnw"
```

```r
with_ext(p, "html")
```

```
## [1] "abc.html"         "def123.html"      "path/to/foo.html"
```

## Types of operating systems

The series of functions `is_linux()`, `is_macos()`, `is_unix()`, and `is_windows()` test the types of the OS, using the information from `.Platform` and `Sys.info()`, e.g.,


```r
xfun::is_macos()
```

```
## [1] TRUE
```

```r
xfun::is_unix()
```

```
## [1] TRUE
```

```r
xfun::is_linux()
```

```
## [1] FALSE
```

```r
xfun::is_windows()
```

```
## [1] FALSE
```

## Loading and attaching packages

Often times I see users attach a series of packages in the beginning of their scripts by repeating `library()` multiple times. This could be easily vectorized, and the function `xfun::pkg_attach()` does this job. For example,


```r
library(testit)
library(parallel)
library(tinytex)
library(mime)
```

is equivalent to


```r
xfun::pkg_attach(c("testit", "parallel", "tinytex", "mime"))
```

I also see scripts that contain code to install a package if it is not available, e.g.,


```r
if (!requireNamespace("tinytex")) install.packages("tinytex")
library(tinytex)
```

This could be done via


```r
xfun::pkg_attach2("tinytex")
```

The function `pkg_attach2()` is a shorthand of `pkg_attach(..., install = TRUE)`, which means if a package is not available, install it. This function can also deal with multiple packages.

The function `loadable()` tests if a package is loadable.

## Read/write files in UTF-8

Functions `read_utf8()` and `write_utf8()` can be used to read/write files in UTF-8. They are simple wrappers of `readLines()` and `writeLines()`.

## Convert numbers to English words

The function `numbers_to_words()` (or `n2w()` for short) converts numbers to English words.


```r
n2w(0, cap = TRUE)
```

```
## [1] "Zero"
```

```r
n2w(seq(0, 121, 11), and = TRUE)
```

```
##  [1] "zero"                      
##  [2] "eleven"                    
##  [3] "twenty-two"                
##  [4] "thirty-three"              
##  [5] "forty-four"                
##  [6] "fifty-five"                
##  [7] "sixty-six"                 
##  [8] "seventy-seven"             
##  [9] "eighty-eight"              
## [10] "ninety-nine"               
## [11] "one hundred and ten"       
## [12] "one hundred and twenty-one"
```

```r
n2w(1e+06)
```

```
## [1] "one million"
```

```r
n2w(1e+11 + 12345678)
```

```
## [1] "one hundred billion, twelve million, three hundred forty-five thousand, six hundred seventy-eight"
```

```r
n2w(-987654321)
```

```
## [1] "minus nine hundred eighty-seven million, six hundred fifty-four thousand, three hundred twenty-one"
```

```r
n2w(1e+15 - 1)
```

```
## [1] "nine hundred ninety-nine trillion, nine hundred ninety-nine billion, nine hundred ninety-nine million, nine hundred ninety-nine thousand, nine hundred ninety-nine"
```

## Check reverse dependencies of a package

Running `R CMD check` on the reverse dependencies of **knitr** and **rmarkdown** is my least favorite thing in developing R packages, because the numbers of their reverse dependencies are huge. The function `rev_check()` reflects some of my past experience in this process. I think I have automated it as much as possible, and made it as easy as possible to discover possible new problems introduced by the current version of the package (compared to the CRAN version). Finally I can just sit back and let it run.

## Input a character vector into the RStudio source editor

The function `rstudio_type()` inputs characters in the RStudio source editor as if they were typed by a human. I came up with the idea when preparing my talk for rstudio::conf 2018 ([see this post](https://yihui.name/en/2018/03/blogdown-video-rstudio-conf/) for more details).

## Print session information

Since I have never been fully satisfied by the output of `sessionInfo()`, I tweaked it to make it more useful in my use cases. For example, it is rarely useful to print out the names of base R packages, or information about the matrix products / BLAS / LAPACK. Often times I want additional information in the session information, such as the Pandoc version when **rmarkdown** is used. The function `session_info()` tweaks the output of `sessionInfo()`, and makes it possible for other packages to append information in the output of `session_info()`.

You can choose to print out the versions of only the packages you specify, e.g.,


```r
xfun::session_info(c("xfun", "rmarkdown", "knitr", "tinytex"), 
  dependencies = FALSE)
```

```
## R version 3.5.0 (2018-04-23)
## Platform: x86_64-apple-darwin15.6.0 (64-bit)
## Running under: macOS High Sierra 10.13.5
## 
## Locale: en_US.UTF-8 / en_US.UTF-8 / en_US.UTF-8 / C / en_US.UTF-8 / en_US.UTF-8
## 
## Package version:
##   knitr_1.20.5     rmarkdown_1.10.3 tinytex_0.5.10  
##   xfun_0.2.9      
## 
## Pandoc version: 2.2.1
```

