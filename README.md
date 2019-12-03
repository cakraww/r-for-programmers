# Manipulate data & making plots with R for programmers

## Introduction
R is a good tool in your toolbox to manipulate and visualize local data. Even if you can only make bar charts and line charts, it can be very useful compared to only display the data in text. It also has a nice IDE (RStudio).

But R is a weird programming language. It can be hard to get started with.

When you look for answer in stackoverflow about R, you might have many wtf moment. There are many absurd-hacky answers that people gladly offer. This is often frustating. I think this paragraph resembles a lot with R situation:
> Just for reference, 80% of awful Perl "code" in my $work falls under this - it was written by financial analysts who are smart enough to pick up a Perl book and some earlier scripts, clone off a script that does what business need is, and don't have CS/programming background to worry about how readable/maintainable their code was. - [from stackoverflow](https://stackoverflow.com/a/1491004/2649611)

This short guide shows a minimal way to get something done in R.

## Installation
Install [R](https://cran.rstudio.com/) and [RStudio](https://www.rstudio.com/products/rstudio/download/#download).

> At the time of writing, their versions are R 3.6.1 and RStudio 1.2

> RStudio also has built-in vim keybinding support. It is not perfect, but works good enough. The setting is on `Tools > Global Options > Code > Keybindings > Vim`.

Change RStudio working directory: `RStudio > Preferences > General > Default working directory`.

### Spark
To use Spark:
1. Install Java. Download JDK 8 from https://aws.amazon.com/corretto/. Using Java 8 because last time I tried, Spark only works with Java 8.
2. Install Spark from RStudio:
```R
install.packages("sparklyr")
library(sparklyr)

spark_available_versions()  # see available version
spark_install(version = "2.4")  
```

## Working with RStudio
### RStudio UI
Create a new file, then the top left pane is where you write the script. The console (R REPL) is on the bottom left pane. Plots and documentation is on the bottom right pane.

### Navigating RStudio
On mac, `Ctrl` is `Cmd`, and `Alt` is `Option`. These shortcut can help you to reach mouse/trackpad less often.
- `Ctrl+1` -> move cursor to top-left pane (script area).
- `Ctrl+2` -> move cursor to console (REPL).
- `Ctrl+Enter` on the script area -> run *the statement/expression* where the current cursor is at.
```R
x <-
    42  # when your cursor is here and you press ctrl+enter,
        # the entire `x <- 42` statement is run, not 42 only.
```
- `Alt+Enter` -> same as `Ctrl+Enter`, but your cursor will stay on the same line (not moved down).
- Block text then `Ctrl+enter` -> run selected code.

> You can just put your cursor at the end of function declaration's bracket to run the function declaration.


### Quick comment/uncomment
Block lines you want to comment/uncomment, press `Ctrl+Shift+C` (`Cmd+Shift+C` for Mac). Anything after `#` is commented.

### Quick Documentation
Run `?<function name>` to bring up the documentation for that particular function.
```R
?as.factor
```

### Installing packages/libraries
Use `install.packages('packageName')` to install package/library. Run it once on the Console and it will be installed permanently (until explicitly removed).
```R
install.packages('tidyverse')   # contains dplyr & ggplot
install.packages('funModeling') # used only for `freq`
```

## Learn by doing
Quick hands-on. R basics will come after this section.

- Download 2014 flights data in New York here: https://raw.githubusercontent.com/Rdatatable/data.table/master/vignettes/flights14.csv .
- Put the csv file into the RStudio working directory. You can find out where your working directory is by running `getwd()` in RStudio Console.
- Make sure you have installed package `tidyverse` and `funModeling`.
- Load the csv:
```R
flights <- read.csv('flights14.csv')
```
- Copy these to the script then run one by one by `Ctrl+Enter`-ing them.
```R
library(tidyverse)
# columns stats
flights %>% summary

# number of rows
flights %>% count

# show data in gui
flights %>% view

# distinct airlines
flights %>% .$carrier %>% levels

# select particular columns
flights %>% select(day, origin, dest, distance)

# show data only for UA & DL airlines
flights %>%
    filter(carrier %in% c('UA', 'DL')) %>%    # like WHERE in SQL. or `filter(carrier == 'UA' || carrier == 'DL')`
    view

# count by month
flights %>% 
  group_by(month) %>%   # like GROUP BY in SQL
  summarise(n=n())      # `n` on the left hand side is column name, `n()` is aggregated function to count number of row.

# count by carrier, take top 5, sort descending
flights %>%
  group_by(carrier) %>% 
  summarise(n=n()) %>%
  top_n(5, n) %>%       # top 5 by column `n`
  arrange(-n)           # sort by column `n` descending

library(funModeling)
# plot the frequency of the carrier
flights %>% select(carrier) %>% freq
# or
flights$carrier %>% freq

# plot top 10 dest airport on horizontal barplot
flights %>%
  group_by(dest) %>%
  summarise(n=n()) %>%
  top_n(10, n) %>%  
  # if you put `arrange` here, it won't affect the order on the plot
  ggplot(aes(x=dest, y=n)) + # which columns for x and y axis
    geom_col() +    # bar plot
    coord_flip()    # make it horizontal

# to make it sorted by n
flights %>%
  group_by(dest) %>%
  summarise(n=n()) %>%
  top_n(10, n) %>%  
  mutate(dest=fct_reorder(dest, n)) %>% # change the factor order
  ggplot(aes(x=dest, y=n)) + 
    geom_col() + 
    coord_flip()

# total airtime per carrier over month
flights %>%
  mutate(month=as.factor(month)) %>%  # treat month as category, not number
  group_by(month, carrier) %>%
  summarise(tot_airtime=sum(air_time)) %>%
  ggplot(aes(x=month, y=tot_airtime, group=carrier)) +
    geom_line(aes(color=carrier))
```

## R Basics
### Variable Assigment
Use `<-` for assigning variables. 
```R
x <- 42
```

> Variable name quirks: `.` is valid character for variable name, it is **not** a method call/accessing field like in OOP. So `as.factor` is just one function name, there is no `as` object.

> [Google's R style guide](https://google.github.io/styleguide/Rguide.xml) suggests `camelCase` for variables, `PascalCase` for functions; yet [Advanced R](http://adv-r.had.co.nz/Style.html) suggest `snake_case` for both variables and functions.

### String
Can use single or double quotes.
```R
'foo' == "foo"
```
Concat multiple strings using `paste`.
```R
paste('foo', 'bar') == 'foo bar'        # default separator is space
paste('foo', 'baz', sep='') == 'foobaz'
```
If you want to print to console, the python-like way is using `cat`.
```R
x <- 42
name <- 'foo'
cat(name, 'says the answer is', x, '\n')  # doesn't include newline automatically

# to remove spaces in between:
cat('this', 'has', 'no', 'space', 'in', 'between', sep='')
```

### Boolean
- The values are `TRUE` and `FALSE`.
- AND operator is `&&`; OR operator is `||`.
- Equal operator is `==`; not equal is `!=`.

### Breaking to multiple lines
If one statement is too long you can break it to multiple line. Note that you need to leave an operator hanging at the end to let R know you still have something below.
```R
40 +        
    2   # again, if you ctrl+enter on this line,
        # the entire 40+2 statement will be executed.
```

### Function
Declaring function example:
```R
identity <- function(x) {
    x   # last line in function is implicitly returned
}   # if you ctrl+enter here, the entire function declaration will be run
identity(3)     # return 3
# or, with dplyr syntax
library(dplyr)
3 %>% identity  # return 3
```
For explicit return inside a function, use `return()` function  e.g: `return(x)`.

### Vector, list, data frame, factor
Vector is this stuff: `c(1, 2, 3)` like array in C/Java or list in Python. 
```R
flights$carrier    # this returns a vector
# dplyr version
flights %>% .$carrier
```

List is this stuff: `list(a=c(1, 2), b='foo')`. It is like struct/record/dictionary. Useful to pass the parameter dynamically (like `kwargs` in Python).

Data frame is like a table in SQL. It has columns and rows. When you read a csv file, it will be read as a data frame.
```R
# create a new data frame
dummy_df <- data.frame(num=c(1, 2, 3), bool=c(TRUE, TRUE, FALSE))
dummy_df %>% colnames
# output: "num" "bool"
```

Factor is finite set of elements or categories (like colors: `red`, `blue`, `green`). The distinct elements are called `levels`. By default the levels are sorted alphabetically.
```R
flights$month %>% as.factor %>% levels
```

## Function Composition with Dplyr
### Dplyr (and maggritr operator)
With dplyr, you can code with much less parentheses `()` and your code will look functional programming-y. It is like SQL (dplyr also has joins) but with a syntax that can be composed (chainable).

You can use `%>%` to pass previous value to next function's argument.

```R
# these are all equivalent
summary(flights)
# or 
flights %>% summary
# or
flights %>% summary()
# or
flights %>% summary(.)
```
