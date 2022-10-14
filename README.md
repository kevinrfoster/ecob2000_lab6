Lab 6
================

### Econ B2000, MA Econometrics

### Kevin R Foster, the Colin Powell School at the City College of New York, CUNY

### Fall 2022

Form a group.

This week we move to logit and probit models. These are suited for when
the dependent y variable takes values of just 0 or 1. This is polar
opposite of previous cases where we assume y variable can take any real
value. There is a middle ground where y can take a limited number of
values (more than 2 but less than infinite) but we look at the extreme
cases for now.

We will use the Household Pulse data, from midterm, and consider
people’s choice to get vaxx.

The generic format for a logit model is

``` r
model_logit1 <- glm(vaxx ~ EEDUC,
            family = binomial, data = Household_Pulse_data)
```

where obviously you can expand the set of independent X variables.

The main differences from lm() are that the call now has a g in it,
`glm()`, for Generalized Linear Model; and that it includes the bit
about `family = binomial`. Doing it is easy; understanding the results
is a bit more difficult.

*Before you rush off to estimate that* just note that there is some
preliminary work to be done (such as creating the data to use).

``` r
Household_Pulse_data$vaxx <- (Household_Pulse_data$RECVDVACC == "yes got vaxx")
is.na(Household_Pulse_data$vaxx) <- which(Household_Pulse_data$RECVDVACC == "NA") 
```

What is the difference between “NA” as label of a factor and actual NA
that R understands? Mostly note that
`sum(is.na(Household_Pulse_data$RECVDVACC))` evaluates as zero. There
are valid reasons to do one or the other, just make it a conscious
choice.

In general it is a good idea to check summary stats before doing fancier
models. For example look at the fractions by education, maybe do some
statistics like you ~~did~~ should have done in exam.

``` r
table(Household_Pulse_data$vaxx,Household_Pulse_data$EEDUC)
```

(although kable package could do nicer, if you want to play)

Note that R is quite tolerant of using a factor as dependent variable,
although you should be paranoid about how it chooses the ordering. In
this case, the ordering works out, but in general you should check
things like `summary(Household_Pulse_data$vaxx)` vs
`summary(as.numeric(Household_Pulse_data$vaxx))`. You could also create
`vaxx_factor <- as.factor(Household_Pulse_data$vaxx)` – check how it
assigns `levels(vaxx_factor)`. You could set those, for example,
`levels(vaxx_factor) <- c("no","yes")` or whatever else. (You could
even, if you were diabolical, set levels to be ~~c(1,0)~~ – yes in that
order!, but then don’t come crying to me when skynet targets you for a
nuke.) Of course you wouldn’t do that, but you might have created a
variable, ’notvaxxed \<-\` that would have opposite ordering.

The problem created is that the model gives estimates for what X
variables tend to make y bigger – and you want to ensure that you
understand what “bigger” means. More likely to be vaccinated is
different than more likely to be unvaccinated. Either model could be
sensible, as long as you’re clear about which one the computer is
estimating.

R can be weirdly tolerant. For example you could run
`glm(RECVDVACC ~ EEDUC,family = binomial)` and it would happily chug
along without throwing any errors – even though RECVDVACC has **THREE**
levels and doesn’t really fit with **bi**nomial. I know, I know, there
are a thousand other things where R fetches an error for the tiniest
thing but with these big problems it just trots along, whatever you say
boss. So that’s one rationale to call `as.numeric` to make sure that
what you think is 0 and 1 is, actually, what R thinks is 0 and 1.

As usual when you do your analysis you might want to subset, for
example,

``` r
pick_use1 <- (Household_Pulse_data$TBIRTH_YEAR < 2000) 
dat_use1 <- subset(Household_Pulse_data, pick_use1)

# and to be finicky, might want to use this for factors after subsetting in case some get lost
dat_use1$RECVDVACC <- droplevels(dat_use1$RECVDVACC) 
```

So start from this baseline model and launch ahead,

``` r
model_logit1 <- glm(vaxx ~ TBIRTH_YEAR + EEDUC + MS + RRACE + RHISPANIC + GENID_DESCRIBE,
            family = binomial, data = dat_use1)
summary(model_logit1)
```

Note that HHPulse gives data on age as “TBIRTH_YEAR” so if you put that
into a regression, the coefficient sign swaps from what we usually think
of, if we had AGE in the regression. Could define
`AGE <- 2022 - TBIRTH_YEAR`. A regression, `Y ~ AGE` might have a
positive coefficient on Age, that as person gets older Y tends to
increase. It would then have a *negative* coefficient on TBIRTH_YEAR,
that as person’s birth year goes up (therefore they are younger) then Y
tends to decrease. Again the math don’t care, it’s just the humans who
have to work to follow along.

What other X variables might you add? Maybe some interactions? LOTS OF
INTERACTIONS? What other subsets? What changes about results with
different subsets?

For homework, I will ask for predicted values so you can start to figure
out how to get those. Something like this, although you’ll want to look
at various different sets:

``` r
new_data_to_be_predicted <- data.frame(TBIRTH_YEAR = 1990,
                                       EEDUC = factor("bach deg", levels = levels(dat_use1$EEDUC)),
                                       MS = factor("never",levels = levels(dat_use1$MS)),
                                       RRACE = factor("Black",levels = levels(dat_use1$RRACE)),
                                       RHISPANIC = factor("Hispanic",levels = levels(dat_use1$RHISPANIC)),
                                       GENID_DESCRIBE = factor("male", levels = levels(dat_use1$GENID_DESCRIBE))
)
predict(model_logit1,new_data_to_be_predicted)
```

Do the X variables have the expected signs and patterns of significance?
Explain if there is a plausible causal link from X variables to Y and
not the reverse. Explain your results, giving details about the
estimation, some predicted values, and providing any relevant graphics.
Impress.

Also estimate a probit model (details in Lecture Notes) and OLS, with
the same X and Y variables. Compare the results, such as coefficients
and predicted values. If you’re eager, try to split the sample into
training and test data, then compare which model predicts better in the
sample that it hasn’t seen yet.
