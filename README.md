Lab 6
================

### Econ B2000, MA Econometrics

### Kevin R Foster, the Colin Powell School at the City College of New York, CUNY

### Fall 2024

We will start the first of a 3-part sequence.

In the first part (Lab 6), we will use basic OLS to estimate some models
of a 0/1 y-variable. In second part (Lab 7) we’ll estimate with logit
and probit models. In third part (Lab 8) we’ll use some additional
machine-learning techniques.

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

## With 0/1 y-variable

I’ll look at what factors relate to a partner being older and I’ll
choose to consider traditional pairs, where a man and woman are married
and he is placed as householder (in the olden days, would be called
‘head of household’). I’ll create a dummy variable for if the man is
more than 3 years older than the woman.

``` r
trad_data <- acs2021_couples %>% filter( (SEX == "Female") & (h_sex == "Male") )

trad_data$he_more_than_3yrs_than_her <- as.numeric(trad_data$age_diff < -3)
```

Note the variable name.

All the math underlying is just concerned with which of the x-variables
make the y-variable more likely to be a higher number. In this case it’s
ok, I’ve set it up for you, but in general you want to confirm which
factor answer is one and which is zero.

For instance,

``` r
table(trad_data$he_more_than_3yrs_than_her,cut(trad_data$age_diff,c(-100,-10, -3, 0, 3, 10, 100)))
```

shows that a one corresponds to ‘he is older by 3 or more years’ and
zero corresponds to ‘not’. But a different person could estimate a model
where the dependent variable is ‘he is *not* older by 3 or more years’
and that would have opposite signs for the estimated coefficients!
Either model could be sensible, as long as you’re clear about which one
the computer is estimating. Be paranoid and check.

You can estimate models something like this (once you figure out what
subset of data you’ll use)

``` r
ols_out1 <- lm(he_more_than_3yrs_than_her ~ educ_hs + educ_somecoll + educ_college + educ_advdeg + AGE, data = trad_data)
summary(ols_out1)
```

Note that interpreting AGE coefficient is a bit complicated – that’s not
age at marriage but the woman’s current age.

And here I’ve used a couple of the Education dummies, you could also use
a factor with more levels or just the years of education that we
created. You should understand the differences.

``` r
ols_out2 <- lm(he_more_than_3yrs_than_her ~ educ_numeric + h_educ_numeric + AGE, data = trad_data)
summary(ols_out2)
ols_out3 <- lm(he_more_than_3yrs_than_her ~ EDUC + h_educ + AGE, data = trad_data)
summary(ols_out3)
```

Try some more complicated factors, too, such as state (but first make a
guess of what states you’d expect to have positive or negative values,
so that you can see what you learn from this).

``` r
ols_out4 <- lm(he_more_than_3yrs_than_her ~ educ_numeric + h_educ_numeric + AGE + STATEFIP, data = trad_data)
summary(ols_out4)
```

You should be able to demonstrate some complicated hypothesis tests
(such as whether all all of the coefficients on a factor are jointly
zero).

Can you make some graphs to show some of the results from your
regressions?
