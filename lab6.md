Lab 6
================

### Econ B2000, MA Econometrics

### Kevin R Foster, the Colin Powell School at the City College of New York, CUNY

### Fall 2023

We will start the first of a 3-part sequence.

Upon Claudia Goldin being named for Nobel Prize this month, I was
talking with a colleague, Prof Norma Fuentes-Mayorga. She described her
hypothesis about female labor force participation and especially choices
to work in the public sector, in jobs that are more stable. She has
heard this rationale in numerous interviews, particularly among
minoritized women, but we’d like to see if there is more evidence for
this. I have created an indicator (public_work) in the ACS2021 data,
based on the industry that the person works in.

In the first part, we will use basic OLS to estimate some models of the
choice to work in public sector. In second part (next week) we’ll
estimate with logit and probit models. In third part (week after) we’ll
use some additional machine-learning techniques.

We’ll start by discussing what is appropriate specification of
interaction terms, to find evidence of this effect. In your group you
can discuss about what subset is most relevant and how exactly you’d
implement the estimation. Then you will work on some results, come back
and share.

You’ll have to download a couple csv definition files, `IND_levels.csv`
and `publicwork_recode.csv`. Then you should first run these:

``` r
require(plyr)
require(dplyr)
require(tidyverse)
require(haven)

levels_n <- read.csv("IND_levels.csv")
names(levels_n) <- c("New_Level","levels_orig")
acs2021$IND <- as.factor(acs2021$IND)
levels_orig <- levels(acs2021$IND) 
levels_new <- join(data.frame(levels_orig),data.frame(levels_n))

acs2021$public_work <- acs2021$IND 
levels_public <- read.csv("publicwork_recode.csv")
names(levels_public) <- c("levels_orig","New_Level")
levels_new_pub <- join(data.frame(levels_orig),data.frame(levels_public))


levels(acs2021$IND) <- levels_new$New_Level
levels(acs2021$public_work) <- levels_new_pub$New_Level
```

Now before you run to estimate a model, in general it is a good idea to
check summary stats before doing fancier models. For example look at the
fractions by education, maybe do some statistics like you ~~did~~ should
have done in exam.

R doesn’t want a factor as dependent variable in lm() call, so we create
a numeric version,

``` r
acs2021$public_work_num <- as.numeric(acs2021$public_work == "work for public, stable")
```

Although other functions will take a factor. That can be trouble so be
careful! All the math underlying is just concerned with which of the
x-variables make the y-variable more likely to be a higher number. In
this case it’s ok, I’ve set it up but in general you want to confirm
which factor answer is one and which is zero.

For instance,

``` r
table(acs2021$public_work,acs2021$public_work_num)
```

shows that a one corresponds to ‘yes the person does work for public
and/or in a generally stable job’. But a different person could estimate
a model where the dependent variable is ‘yes the person works in private
sector’ and that would have different signs for the estimated
coefficients! Either model could be sensible, as long as you’re clear
about which one the computer is estimating. Be paranoid and check.

You can estimate models something like this (once you figure out what
subset of data you’ll use)

``` r
ols_out1 <- lm(public_work_num ~ female + educ_hs + educ_somecoll + educ_college + educ_advdeg + AGE, data = dat_use)
summary(ols_out1)
```
