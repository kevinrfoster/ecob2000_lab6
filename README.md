Lab 6
================

### Econ B2000, MA Econometrics

### Kevin R Foster, the Colin Powell School at the City College of New York, CUNY

### Fall 2024

This lab will introduce a dataset that we’ll use in a few subsequent
labs.

## Background

I’ve created a version of the ACS data that has extensive data on people
who reported they were spouses/partners, along with other data (prefixed
h\_) about the person who reported as householder (ie, who is listed
first among the household members). So we can look at things like
relative ages and education of spouses.

``` r
library(ggplot2)
library(tidyverse)
library(haven)

setwd("..//ACS_2021_PUMS//") # your directory structure will be different
load("ACS_2021_couples.RData")
```

Now before you run to estimate a model, in general it is a good idea to
check summary stats before doing fancier models. Just like you ~~did~~
should have done in exam.

This data includes people who report they are a spouse or partner
(latter category includes “partner, friend, or visitor”). That can be
seen with \`summary(acs2021_couples\$RELATE)’. You might choose to
restrict your analysis to one or the other.

I’ve created a variable with the difference in age, which is the age of
the spouse minus the age of the head:

``` r
acs2021_couples$age_diff <- acs2021_couples$AGE - acs2021_couples$h_age
```

Check your understanding – if that’s negative, then which spouse is
older?

It’s worth noting that Census does not give any particular direction to
respondents about choosing a reference person for the household
([details](https://www.census.gov/programs-surveys/cps/technical-documentation/subject-definitions.html#householder)),
but does it look like people tend to make assumptions about that? You
should run this code and discuss about what you can infer from the
result:

``` r
summary(acs2021_couples$age_diff[(acs2021_couples$SEX == "Female")&(acs2021_couples$h_sex == "Male")])
summary(acs2021_couples$age_diff[(acs2021_couples$SEX == "Male")&(acs2021_couples$h_sex == "Female")])
summary(acs2021_couples$age_diff[(acs2021_couples$SEX == "Male")&(acs2021_couples$h_sex == "Male")])
summary(acs2021_couples$age_diff[(acs2021_couples$SEX == "Female")&(acs2021_couples$h_sex == "Female")])
```

Compare with this summary (for the first group)

``` r
summary(acs2021_couples$AGE[(acs2021_couples$SEX == "Female")&(acs2021_couples$h_sex == "Male")])
summary(acs2021_couples$h_age[(acs2021_couples$SEX == "Female")&(acs2021_couples$h_sex == "Male")])
```

(Maybe also check if you see some outliers…)

While it’s easy enough to work with age difference, it might also be
interesting to look at educational differences. Create a simple (but
perhaps effective) measure of years of education, then create
`educ_diff` in an analogous way to `age_diff`.

``` r
acs2021_couples$educ_numeric <- fct_recode(acs2021_couples$EDUC,
                                           "0" = "N/A or no schooling",
                                           "2" = "Nursery school to grade 4",
                                           "6.5" = "Grade 5, 6, 7, or 8",
                                           "9" = "Grade 9",
                                           "10" = "Grade 10",
                                           "11" = "Grade 11",
                                           "12" = "Grade 12",
                                           "13" = "1 year of college",
                                           "14" = "2 years of college",
                                           "15" = "3 years of college",
                                           "16" = "4 years of college",
                                           "17" = "5+ years of college")

acs2021_couples$educ_numeric <- as.numeric(levels(acs2021_couples$educ_numeric))[acs2021_couples$educ_numeric]

acs2021_couples$h_educ_numeric <- fct_recode(acs2021_couples$h_educ,
                                           "0" = "N/A or no schooling",
                                           "2" = "Nursery school to grade 4",
                                           "6.5" = "Grade 5, 6, 7, or 8",
                                           "9" = "Grade 9",
                                           "10" = "Grade 10",
                                           "11" = "Grade 11",
                                           "12" = "Grade 12",
                                           "13" = "1 year of college",
                                           "14" = "2 years of college",
                                           "15" = "3 years of college",
                                           "16" = "4 years of college",
                                           "17" = "5+ years of college")

acs2021_couples$h_educ_numeric <- as.numeric(levels(acs2021_couples$h_educ_numeric))[acs2021_couples$h_educ_numeric]

acs2021_couples$educ_diff <- acs2021_couples$educ_numeric - acs2021_couples$h_educ_numeric
```

Like Lab 5,you might want a smaller subsample. Maybe something like
this,

``` r
acs_subgroup <- acs2021_couples %>% filter((AGE >= 25) & (AGE <= 55) & 
                                     (LABFORCE == 2) & (WKSWORK2 > 4) & (UHRSWORK >= 35) )
```

Please run a regression where the dependent variable is the age
difference and the independent variables include education (of both
partners) and their ages (including polynomial terms). Show some good
complex hypothesis tests.
