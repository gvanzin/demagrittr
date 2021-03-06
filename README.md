# demagrittr

## What is this package?
demagrittr() converts magrittr's syntax to eager evaluation syntax for
the purpose of: 

+ understanding quite complicated and nested piped sentences
+ debugging when an error occurs
+ run-time reduction (if `%>%` is heavily used inside a long loop)

This is experimental and not fully tested, so I would be glad
if you could inform me of any misunderstandings or mistakes.  

## Installation
``` r
# install.packages("devtools")
devtools::install_github("tobcap/demagrittr")
library("demagrittr")
```

## Usage
``` r
# direct passing 
demagrittr(x %>% f %>% g %>% h)

# call object can be treated
expr <- quote(x %>% f %>% g %>% h)
demagrittr(expr)

# compile and evaluate
demagrittr(1:10 %>% sum %>% log %>% sin, eval_ = TRUE)
# [1] -0.7615754
```

# benchmarking
``` r
library("microbenchmark")
library("magrittr")
library("pipeR")
library("demagrittr")

expr1 <- quote(1:10 %>% sum %>% log %>% sin)
expr2 <- demagrittr(expr1)
expr3 <- quote(1:10 %>>% sum %>>% log %>>% sin)

microbenchmark(
    "%>%" = eval(expr1)
  , demagrittr = eval(expr2)
  , "%>>%" = eval(expr3)
  , times = 1e3)
# Unit: microseconds
#        expr     min       lq      mean   median       uq      max neval
#         %>% 320.071 334.7810 372.66629 347.2635 364.2025 2766.955  1000
#  demagrittr  12.036  14.2660  17.21454  16.0490  17.3860  453.804  1000
#        %>>%  79.349  85.1445  98.48822  90.9395  96.2890 2281.055  1000

identical(eval(expr1), eval(expr2))
# [1] TRUE


## from http://renkun.me/blog/2014/08/08/difference-between-magrittr-and-pipeR.html#performance
expr4 <- quote(
  lapply(1:100000, function(i) {
    sample(letters,6,replace = T) %>%
      paste(collapse = "") %>%
      "=="("rstats")
  })
)
expr5 <- demagrittr(expr4)
expr6 <- quote(
  lapply(1:100000, function(i) {
    sample(letters,6,replace = T) %>>%
      paste(collapse = "") %>>%
      "=="("rstats")
  })
)

# My poor laptop takes huge time. The unit is 'seconds'.
microbenchmark(
    "%>%" = eval(expr4)
  , demagrittr = eval(expr5)
  , "%>>%" = eval(expr6)
  , times = 1) # may produce an unstable result
# Unit: seconds
#        expr       min        lq      mean    median        uq       max neval
#         %>% 50.781067 50.781067 50.781067 50.781067 50.781067 50.781067     1
#  demagrittr  3.615714  3.615714  3.615714  3.615714  3.615714  3.615714     1
#        %>>% 10.414592 10.414592 10.414592 10.414592 10.414592 10.414592     1

identical(eval(expr4), eval(expr5))
# [1] TRUE

```

## Not supported 
* control visibility of a result of the last function
 (printing the result or not)

## ToDo
* make a wrapper function just like source() or sys.source()
