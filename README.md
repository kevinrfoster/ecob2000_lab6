Lab 6
================

### Econ B2000, MA Econometrics

### Kevin R Foster, the Colin Powell School at the City College of New York, CUNY

### Fall 2021

Form a group.

*First* a note to advance your programming. The following bits of code
do the same thing:

``` r
attach(acs2017_ny)
model_v1 <- lm(INCWAGE ~ AGE)
detach()

model_v2 <- lm(acs2017_ny$INCWAGE ~ acs2017_ny$AGE)

model_v3 <- lm(INCWAGE ~ AGE, data = acs2017_ny)
```

I prefer the last one, v3. I think v2 is too verbose (especially once
you have dozens of variables in your model) while v1 is liable to errors
since, if the `attach` command gets too far separated from the `lm()`
within your code chunks, can have unintended consequences. Later you
will be doing robustness checks, where you do the same regression on
slightly different subsets. (For example, compare a model fit on all
people 18-65 vs people 25-55 vs other age ranges.) In that case v3
becomes less verbose as well as less liable to have mistakes.

However specifying the dataset, as with v3, means that any preliminary
data transformations should modify the original dataset. So if you
create `newvarb <- f(oldvarb)`, you have to carefully set
`dataset$newvarb <- f(dataset$oldvarb)` and think about which dataset
gets that transformation.

## Lab 6

Anyway, on to the lab. This week we move to logit and probit models.
These are suited for when the dependent y variable takes values of just
0 or 1.

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
about `family = binomial`.

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
statistics like you ~~did~~ should have done in Q1 of exam.

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
order\!, but then don’t come crying to me when skynet targets you for a
nuke.) Of course you wouldn’t do that, but you might have created a
variable, ’notvaxxed \<-\` that would have opposite ordering.

Seriously, though, R can be weirdly tolerant. For example you could run
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
pick_use1 <- (Household_Pulse_data$REGION == "Northeast") # just for example!
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

Note that they give data on age as “TBIRTH\_YEAR” so if you put that
into a regression, the coefficient sign swaps from what we usually think
of, if we had AGE in the regression. Could define `AGE <- 2021 -
TBIRTH_YEAR`. A regression, `Y ~ AGE` might have a positive coefficient
on Age, that as person gets older Y tends to increase. It would then
have a *negative* coefficient on TBIRTH\_YEAR, that as person’s birth
year goes up (therefore they are younger) then Y tends to decrease.

What other X variables might you add? Maybe some interactions? LOTS OF
INTERACTIONS? What other subsets? What changes about results with
different subsets?

For homework, I will ask for predicted values so you can start to figure
out how to get those.

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
