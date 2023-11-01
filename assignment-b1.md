assignment-b1
================
Lauren Puumala
2023-11-03

# STAT 545B Assignment B1: Making a Function

## Introduction

This is the first assignment of STAT 545B. In this .Rmd file, I will
define and document a new function called `count_obs` (Exercises 1 & 2).
I will then show various examples of using this function (Exercise 3),
followed by some formal tests of the function using the `testthat`
package (Exercise 4).

**First, I will load the relevant libraries.**

``` r
suppressPackageStartupMessages(library(tidyverse))
suppressPackageStartupMessages(library(testthat))
suppressPackageStartupMessages(library(datateachr))
suppressPackageStartupMessages(library(gapminder))
suppressPackageStartupMessages(library(palmerpenguins))
```

**Then, I will take a glimpse at the datasets that will be used in this
assignment for testing the function for a reminder of what they look
like.**

``` r
#REMINDER OF WHAT THE DATASETS USED IN THIS ASSIGNMENT LOOK LIKE
glimpse(vancouver_trees)
```

    ## Rows: 146,611
    ## Columns: 20
    ## $ tree_id            <dbl> 149556, 149563, 149579, 149590, 149604, 149616, 149…
    ## $ civic_number       <dbl> 494, 450, 4994, 858, 5032, 585, 4909, 4925, 4969, 7…
    ## $ std_street         <chr> "W 58TH AV", "W 58TH AV", "WINDSOR ST", "E 39TH AV"…
    ## $ genus_name         <chr> "ULMUS", "ZELKOVA", "STYRAX", "FRAXINUS", "ACER", "…
    ## $ species_name       <chr> "AMERICANA", "SERRATA", "JAPONICA", "AMERICANA", "C…
    ## $ cultivar_name      <chr> "BRANDON", NA, NA, "AUTUMN APPLAUSE", NA, "CHANTICL…
    ## $ common_name        <chr> "BRANDON ELM", "JAPANESE ZELKOVA", "JAPANESE SNOWBE…
    ## $ assigned           <chr> "N", "N", "N", "Y", "N", "N", "N", "N", "N", "N", "…
    ## $ root_barrier       <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "…
    ## $ plant_area         <chr> "N", "N", "4", "4", "4", "B", "6", "6", "3", "3", "…
    ## $ on_street_block    <dbl> 400, 400, 4900, 800, 5000, 500, 4900, 4900, 4900, 7…
    ## $ on_street          <chr> "W 58TH AV", "W 58TH AV", "WINDSOR ST", "E 39TH AV"…
    ## $ neighbourhood_name <chr> "MARPOLE", "MARPOLE", "KENSINGTON-CEDAR COTTAGE", "…
    ## $ street_side_name   <chr> "EVEN", "EVEN", "EVEN", "EVEN", "EVEN", "ODD", "ODD…
    ## $ height_range_id    <dbl> 2, 4, 3, 4, 2, 2, 3, 3, 2, 2, 2, 5, 3, 2, 2, 2, 2, …
    ## $ diameter           <dbl> 10.00, 10.00, 4.00, 18.00, 9.00, 5.00, 15.00, 14.00…
    ## $ curb               <chr> "N", "N", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "…
    ## $ date_planted       <date> 1999-01-13, 1996-05-31, 1993-11-22, 1996-04-29, 19…
    ## $ longitude          <dbl> -123.1161, -123.1147, -123.0846, -123.0870, -123.08…
    ## $ latitude           <dbl> 49.21776, 49.21776, 49.23938, 49.23469, 49.23894, 4…

``` r
glimpse(penguins)
```

    ## Rows: 344
    ## Columns: 8
    ## $ species           <fct> Adelie, Adelie, Adelie, Adelie, Adelie, Adelie, Adel…
    ## $ island            <fct> Torgersen, Torgersen, Torgersen, Torgersen, Torgerse…
    ## $ bill_length_mm    <dbl> 39.1, 39.5, 40.3, NA, 36.7, 39.3, 38.9, 39.2, 34.1, …
    ## $ bill_depth_mm     <dbl> 18.7, 17.4, 18.0, NA, 19.3, 20.6, 17.8, 19.6, 18.1, …
    ## $ flipper_length_mm <int> 181, 186, 195, NA, 193, 190, 181, 195, 193, 190, 186…
    ## $ body_mass_g       <int> 3750, 3800, 3250, NA, 3450, 3650, 3625, 4675, 3475, …
    ## $ sex               <fct> male, female, female, NA, female, male, female, male…
    ## $ year              <int> 2007, 2007, 2007, 2007, 2007, 2007, 2007, 2007, 2007…

``` r
glimpse(gapminder)
```

    ## Rows: 1,704
    ## Columns: 6
    ## $ country   <fct> "Afghanistan", "Afghanistan", "Afghanistan", "Afghanistan", …
    ## $ continent <fct> Asia, Asia, Asia, Asia, Asia, Asia, Asia, Asia, Asia, Asia, …
    ## $ year      <int> 1952, 1957, 1962, 1967, 1972, 1977, 1982, 1987, 1992, 1997, …
    ## $ lifeExp   <dbl> 28.801, 30.332, 31.997, 34.020, 36.088, 38.438, 39.854, 40.8…
    ## $ pop       <int> 8425333, 9240934, 10267083, 11537966, 13079460, 14880372, 12…
    ## $ gdpPercap <dbl> 779.4453, 820.8530, 853.1007, 836.1971, 739.9811, 786.1134, …

## Exercises 1 & 2: Make a function and document it

The code chunk below defines the function `count_obs` and documents it
with roxygen tags. The purpose of this function is to count the number
of observations of a categorical variable or combinations of categorical
variables and returns the result as a summary tibble. The input
parameters to the function are the data frame of interest (`data`) and a
character vector containing the name(s) of the grouping variable(s)
(`group_vars`). For example, if *vancouver_trees* was used for `data`
and **neighbourhood_name** was used for `group_vars`, the function would
return a tibble listing the number of trees in the dataset that are in
each neighbourhood. If, instead, **neighbourhood_name** *and* **species
name** were used for `group_vars`, the function would return a tibble
listing the number of trees of each species in each neighbourhood.

The function checks the class of the grouping variables specified in
`group_vars` to ensure they belong to one of the following classes:
character, factor, or date. If the grouping variables do not belong to
the acceptable classes, an error message saying “Incorrect grouping
variable class. Ensure all group_vars are of class chr, fct, or date” is
returned.

``` r
#' @title
#' Count the Number of Observations of Categorical Variables
#'
#' @description
#' `count_obs` returns a summary tibble listing the number of observations for each value of a categorical 
#' variable or combination of categorical variables in a specified data frame. The grouping variables must
#' belong to one of the following classes: chr, fct, or date.
#'  
#'  @param data A data frame;  this parameter was named `data` to make it obvious that this should be the
#'  raw data frame from which the user wants to extract information.
#'  @param group_vars Character vector containing the name(s) of the grouping variable(s). One or more
#'  grouping variables may be used. The grouping variables specify which categorical variable(s) in `data`
#'  the function should count observations in. The grouping variable must be of class chr, fct, or date.
#'  This parameter was named `group_vars` to make it obvious that this variable should be used to specify
#'  the names of the variables that should be used for grouping the data when the function is called. 
#'  
#'  @return A tibble listing the number of observations of each value of a categorical variable or
#'  combination of categorical variables specified in `group_vars` from the data frame `data`.
#'  
#' @examples
#' count_obs(df, "categorical_variable_1")

count_obs <- function(data, group_vars, .groups = 'drop') {
  # Make sure `data` is a data frame
  if (is.data.frame(data) == FALSE) {
    stop('`data` must be a data frame.')
  }

  # Only allow grouping by variables that belong to character, date, or factor classes
  # If any grouping variable is an incorrect class, an error message will be thrown
  i <- 1 # initialize indexing variable
  
  for (i in 1:length(group_vars)) { # loop to check variable classes in group_vars
    if (is.character(data[[group_vars[[i]]]]) == FALSE && is.factor(data[[group_vars[[i]]]]) == FALSE && 
        is.Date(data[[group_vars[[i]]]]) == FALSE) {
      stop('Incorrect grouping variable class. Ensure all group_vars are of class chr, fct, or date.')
    } else {
      i <- i + 1 # if variable is correct type, move onto next variable in group_vars
    }
  }

  # Find the number observations for each group and return the summary table
  data %>%
    group_by(pick({{ group_vars }})) %>%
    summarize(num_obs = n(), .groups = .groups)
}
```

## Exercise 3: Examples

### Examples using vancouver_trees dataset

First, I will test the function using the *vancouver_trees* dataset for
the `data` parameter. This dataset includes both categorical and
numerical variables.  
- I will show how the function works when only one categorical variable
is used for the parameter `group_vars` (first two function calls). I
will demonstrate this with character and date grouping variables
(**neighbourhood_name** and **date_planted**, respectively).  
- I will then demonstrate that the function works when `group_vars` is a
character vector containing the names of two categorical grouping
variables (third function call; using **neighbourhood_name** and
**species_name**).  
- I will then demonstrate that the function returns an error when
`group_vars` contains a grouping variable of type double (fourth
function call; using **longitude** and **neighbourhood_name**).  
- In the fifth function call, I will pass a number to the `data`
parameter instead of a data frame to ensure that the function throws the
correct error.  
- In the sixth and final function call, I will supply `group_vars` using
incorrect syntax (not as a character vector) to ensure that the function
also throws an error for this.

``` r
# TESTING WITH vancouver_trees DATASET
count_obs(vancouver_trees, 'neighbourhood_name') # group_vars is type chr
```

    ## # A tibble: 22 × 2
    ##    neighbourhood_name       num_obs
    ##    <chr>                      <int>
    ##  1 ARBUTUS-RIDGE               5169
    ##  2 DOWNTOWN                    5159
    ##  3 DUNBAR-SOUTHLANDS           9415
    ##  4 FAIRVIEW                    4002
    ##  5 GRANDVIEW-WOODLAND          6703
    ##  6 HASTINGS-SUNRISE           10547
    ##  7 KENSINGTON-CEDAR COTTAGE   11042
    ##  8 KERRISDALE                  6936
    ##  9 KILLARNEY                   6148
    ## 10 KITSILANO                   8115
    ## # ℹ 12 more rows

``` r
count_obs(vancouver_trees, 'date_planted') # group_vars is type date
```

    ## # A tibble: 3,995 × 2
    ##    date_planted num_obs
    ##    <date>         <int>
    ##  1 1989-10-27         2
    ##  2 1989-10-30         5
    ##  3 1989-10-31        16
    ##  4 1989-11-02        25
    ##  5 1989-11-03         4
    ##  6 1989-11-06         2
    ##  7 1989-11-07        37
    ##  8 1989-11-08        44
    ##  9 1989-11-09         5
    ## 10 1989-11-10        31
    ## # ℹ 3,985 more rows

``` r
count_obs(vancouver_trees, c('neighbourhood_name', 'species_name')) # group_vars are both type chr
```

    ## # A tibble: 3,056 × 3
    ##    neighbourhood_name species_name   num_obs
    ##    <chr>              <chr>            <int>
    ##  1 ARBUTUS-RIDGE      ABIES                2
    ##  2 ARBUTUS-RIDGE      ACERIFOLIA   X     103
    ##  3 ARBUTUS-RIDGE      ACUTISSIMA           5
    ##  4 ARBUTUS-RIDGE      ALNIFOLIA           16
    ##  5 ARBUTUS-RIDGE      AMERICANA          225
    ##  6 ARBUTUS-RIDGE      AQUIFOLIUM           3
    ##  7 ARBUTUS-RIDGE      ARIA                19
    ##  8 ARBUTUS-RIDGE      ARNOLDIANA X         4
    ##  9 ARBUTUS-RIDGE      AUCUPARIA           16
    ## 10 ARBUTUS-RIDGE      AVIUM               15
    ## # ℹ 3,046 more rows

``` r
count_obs(vancouver_trees, c('longitude', 'neighbourhood_name')) # should return an error b/c longitude is of type dbl
```

    ## Error in count_obs(vancouver_trees, c("longitude", "neighbourhood_name")): Incorrect grouping variable class. Ensure all group_vars are of class chr, fct, or date.

``` r
count_obs(100, 'neighbourhood_name') # should return an error b/c data is not a data frame
```

    ## Error in count_obs(100, "neighbourhood_name"): `data` must be a data frame.

``` r
count_obs(vancouver_trees, neighbourhood_name) # should return an error b/c group_vars is supplied with incorrect syntax
```

    ## Error in eval(expr, envir, enclos): object 'neighbourhood_name' not found

The results of the above code chunk show that the function operates as
desired when chr and date grouping variables are used for `group_vars`
and fails when `group_vars` contains a grouping variable of class dbl,
when `data` is not a data frame, and when `group_vars` is not supplied
as a character vector.

### Examples using penguins dataset

Next, I will test the function using the *penguins* dataset for the
`data` parameter.  
- I will show how the function works when only one categorical variable
is used for the parameter `group_vars` (first function call). I will
demonstrate this with a fct grouping variable (**species**).  
- I will then demonstrate that the function works when `group_vars` is a
character vector containing the names of two categorical grouping
variables of class fct (second function call; using **island** and
**species**).  
- Lastly, I will demonstrate that the function returns an error when
`group_vars` is a grouping variable of class int (third function call;
using **body_mass_g**).

``` r
# TESTING WITH penguins DATASET
count_obs(penguins, 'species') # group_vars is class fct
```

    ## # A tibble: 3 × 2
    ##   species   num_obs
    ##   <fct>       <int>
    ## 1 Adelie        152
    ## 2 Chinstrap      68
    ## 3 Gentoo        124

``` r
count_obs(penguins, c('island', 'species')) # group_vars are both class fct
```

    ## # A tibble: 5 × 3
    ##   island    species   num_obs
    ##   <fct>     <fct>       <int>
    ## 1 Biscoe    Adelie         44
    ## 2 Biscoe    Gentoo        124
    ## 3 Dream     Adelie         56
    ## 4 Dream     Chinstrap      68
    ## 5 Torgersen Adelie         52

``` r
count_obs(penguins, "body_mass_g") # should return an error because body_mass_g is of class int
```

    ## Error in count_obs(penguins, "body_mass_g"): Incorrect grouping variable class. Ensure all group_vars are of class chr, fct, or date.

The results of the above code chunk show that the function operates as
desired when grouping variables of class fct are used for `group_vars`
and fails when `group_vars` contains a grouping variable of class int.

### Examples using gapminder dataset

Next, I will test the function using the *gapminder* dataset for the
`data` parameter.  
- I will show how the function works when only one categorical variable
is used for the parameter `group_vars` (first function call). I will
demonstrate this with a fct grouping variable (**country**).  
- I will then demonstrate that the function works when `group_vars` is a
character vector containing the names of two categorical grouping
variables of class fct (second function call; using **continent** and
**country**).  
- Lastly, I will demonstrate that the function returns an error when
`group_vars` contains three grouping variables, one of which is of class
dbl (third function call; using **continent**, **lifeExp**, and
**country**).

``` r
# TESTING WITH gapminder DATASET
count_obs(gapminder, 'country') # group_vars is class fct
```

    ## # A tibble: 142 × 2
    ##    country     num_obs
    ##    <fct>         <int>
    ##  1 Afghanistan      12
    ##  2 Albania          12
    ##  3 Algeria          12
    ##  4 Angola           12
    ##  5 Argentina        12
    ##  6 Australia        12
    ##  7 Austria          12
    ##  8 Bahrain          12
    ##  9 Bangladesh       12
    ## 10 Belgium          12
    ## # ℹ 132 more rows

``` r
count_obs(gapminder, c('continent', 'country')) # group_vars are both class fct
```

    ## # A tibble: 142 × 3
    ##    continent country                  num_obs
    ##    <fct>     <fct>                      <int>
    ##  1 Africa    Algeria                       12
    ##  2 Africa    Angola                        12
    ##  3 Africa    Benin                         12
    ##  4 Africa    Botswana                      12
    ##  5 Africa    Burkina Faso                  12
    ##  6 Africa    Burundi                       12
    ##  7 Africa    Cameroon                      12
    ##  8 Africa    Central African Republic      12
    ##  9 Africa    Chad                          12
    ## 10 Africa    Comoros                       12
    ## # ℹ 132 more rows

``` r
count_obs(gapminder, c('continent', 'lifeExp', 'country')) # should return an error because lifeExp is of class dbl
```

    ## Error in count_obs(gapminder, c("continent", "lifeExp", "country")): Incorrect grouping variable class. Ensure all group_vars are of class chr, fct, or date.

The results of the above code chunk show that the function operates as
desired when grouping variables of class fct are used for `group_vars`
and fails when `group_vars` contains a grouping variable of class dbl.

## Exercise 4: Test the function

I will use the *vancouver_trees* dataset for all of the following tests.

### Error due to incorrect grouping variable class

First, I will perform a test to check that `count_obs()` gives an error
saying “Incorrect grouping variable class. Ensure all group_vars are of
class chr, fct, or date” when the grouping variables identified in the
parameter group_vars do not belong to the following classes: chr, fct,
or date. Within a single `test_that()` call, I will perform three calls
to `expect_error()` to test three different types of function calls that
should generate this error:  
1. In the first, I will try to group by one dbl variable (**diameter**)
2. In the second, I will try to group by a chr variable
(**neighbourhood_name**) and a dbl variable (**diameter**)  
3. In the third, I will try to group by three dbl variables
(**latitude**, **longitude**, and **diameter**)

``` r
# TEST CASES THAT SHOULD GIVE AN ERROR DUE TO INCORRECT VARIABLE CLASS
error_msg <- 'Incorrect grouping variable class. Ensure all group_vars are of class chr, fct, or date'

test_that('Error for non-categorical grouping variable(s)', {
  expect_error(count_obs(vancouver_trees, 'diameter'), error_msg)
  expect_error(count_obs(vancouver_trees, c('neighbourhood_name', 'diameter')), error_msg)
  expect_error(count_obs(vancouver_trees, c('latitude', 'longitude', 'diameter')), error_msg)
})
```

    ## Test passed 🌈

As we can see, the test passes, meaning that the function returns the
appropriate error when the user tries to group by dbl variables.

### Error due to nonsense inputs

In this next call to `test_that()`, I will check if `count_obs()` fails
when its inputs do not make sense. Namely, I will perform four calls to
`expect_error()`:  
1. In the first, the variable name specified for group_vars
(**country**) will not be a variable that is actually in the dataset
passed to the function (*neighbourhood_name*)  
2. In the second, incorrect syntax will be used for the variable name
specified for group_vars. In particular, it will not be inputted as a
character vector.  
3. In the third, I will try using numbers as the two inputs to the
function, which does not make sense.  
4. In the fourth, I will only supply one parameter input (the data
frame).

``` r
# OTHER TEST CASES THAT SHOULD GIVE AN ERROR
test_that('Test other cases that should give an error', {
  expect_error(count_obs(vancouver_trees, 'country')) # 'country' is not a variable in vancouver_trees dataset
  expect_error(count_obs(vancouver_trees, neighbourhood_name)) # group_vars is not a character vector
  expect_error(count_obs(1, 2)) # nonsense inputs to both parameters
  expect_error(count_obs(vancouver_trees)) # missing group_vars
})
```

    ## Test passed 😀

As we can see, the test passes, meaning that the function returns
errors, as desired, for all of these cases.

### Check that the function output belongs to the correct class

In this call to `test_that()`, I will perform three calls to
`expect_s3_class()` to check that the output of `count_obs()` is a
tibble when suitable inputs are passed to the function. In these three
calls to the expect function, I will specify different numbers of
categorical grouping variables to make sure that the function works when
the user wants to group by one or more variables. I will test this with
chr and date grouping variables. Specifically,  
1. In the first test, I will group by the variable **genus_name**  
2. In the second test, I will group by the variables **species_name**
and **genus_name**  
3. In the third test, I will group by the variables
**neighbourhood_name**, **genus_name**, **species_name**, and
**date_planted**

``` r
# ENSURE FUNCTION EXECUTES AND RETURNS A TIBBLE WHEN APPROPRIATE VALUES ARE PASSED TO data AND group_vars
return_class <- c("tbl", "tbl_df", "data.frame")

test_that('Test cases that should work and return a tibble', {
  expect_s3_class(count_obs(vancouver_trees, 'genus_name'), return_class)
  expect_s3_class(count_obs(vancouver_trees, c('species_name', 'genus_name')), return_class)
  expect_s3_class(count_obs(vancouver_trees, c('neighbourhood_name', 'genus_name', 'species_name', 
                                               'date_planted')), return_class)
})
```

    ## Test passed 🎉

As we can see, the test passed, showing that the function executed and
returned tibbles for all three test cases.

### Compare function output to manual operations

In this final call to `test_that()`, I will check if `count_obs()`
returns a tibble that is identical to that obtained when I manually
create a summary table listing the number of observations of a
categorical variable or variables in the chosen dataset (i.e. when I try
doing the same thing as `count_obs()` manually). To ensure that the
function is robust, I will include three calls to `expect_identical()`
using different numbers of categorical grouping variables:  
1. In the first, I will use **neighbourhood_name** as the grouping
variable  
2. In the second, I will use **neighbourhood_name** and **species_name**
as the grouping variables  
3. In the third, I will use **neighbourhood_name**, **species_name**,
and **date_planted** as the grouping variables

``` r
# ENSURE FUNCTION RETURNS IDENTICAL TIBBLE TO THAT OBTAINED MANUALLY
test_1var <- vancouver_trees %>%
  group_by(neighbourhood_name) %>%
  summarise(num_obs = n(), .groups = 'drop')

test_2var <- vancouver_trees %>%
  group_by(neighbourhood_name, species_name) %>%
  summarise(num_obs = n(), .groups = 'drop')

test_3var <- vancouver_trees %>%
  group_by(neighbourhood_name, species_name, date_planted) %>%
  summarise(num_obs = n(), .groups = 'drop')

test_that('Compare to manual computations', {
  expect_identical(test_1var,count_obs(vancouver_trees, 'neighbourhood_name'))
  expect_identical(test_2var,count_obs(vancouver_trees, c('neighbourhood_name', 'species_name')))
  expect_identical(test_3var,count_obs(vancouver_trees, c('neighbourhood_name', 'species_name', 'date_planted')))
})
```

    ## Test passed 🎉

Again, we see that the test passed, indicating that the function returns
the same tibbles as manual operations doing the same thing.
