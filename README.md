# SimSEMWebsite
---
title: "Using simsem to conduct Power Analysis for Structural Equational Modeling"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, warning = FALSE)
```
Here is an example of conducting a power analysis for Structural Equation Modeling (SEM) using the simsem program in R.  In this example, we estimate the amount of data needed to conduct a SEM with three factors with three indicators, and two path coefficients measuring the direct effect that factor three has on factor one and two.

First, we need to library the simsem program.  Then we need to create a matrix for our indicators.  Here we are creating a matrix with three unique indicators for three factors meaning that we need a total of nine indicators for the three factors.  

For the loadings that we want to assign values to later, we assign them to the value NA for now.

Then we create the starting or initial values in the loading.start matrix.  We assign the loading values of .7 to each of the indicators for each of the factors.  These starting values can vary depending upon the context of your research.

Next, we need to assign the cross loading among loadings with different factors.  We could assign them to zero and assume that there is no correlation between loadings of different factors; however, below, we have chosen to model the correlation between the loading as essentially random with a uniform distribution.

Finally, we combine the loading matrix, the starting values, and the cross-loading correlations matrices into one loading matrix titled LY

```{r, message=FALSE, warning=FALSE}
library(simsem)
loading <- matrix(0, 9, 3)
loading[1:3, 1] <- NA
loading[4:6, 2] <- NA
loading[7:9, 3] <- NA; loading
loading.start <- matrix("", 9, 3)
loading.start[1:3, 1] <- 0.7
loading.start[4:6, 2] <- 0.7
loading.start[7:9, 3] <- 0.7
#Cross loading or loadings among indicators of different factors are random instead of zero
loading.trivial <- matrix("runif(1, -0.2, 0.2)", 9, 3)
loading.trivial[is.na(loading)] <- 0
LY <- bind(loading, loading.start, misspec=loading.trivial); LY
```
Next, we have to create the error variables for each of the indicators.  We simply assign a random error using and random normal distribution for each of the error indicators and assign them a variance of one.  We then combine the error variables for the indicator variables with the variances for the error variable into an error indicator matrix called RTE.
```{r, message=FALSE, warning=FALSE}
error.cor.trivial <- matrix("rnorm(1, 0, 0.1)", 9, 9)
diag(error.cor.trivial) <- 1
RTE <- binds(diag(9), misspec=error.cor.trivial); RTE
```
Next we need to assign the correlation between the factors.  For simplicity, we are assuming that all the correlation between the factors are zero.
```{r, message=FALSE, warning=FALSE}
factor.cor <- diag(3)
RPS <- binds(factor.cor, 0.0); RPS
```
Next, we need to create the path coefficients matrix.  Below we have two paths that are not zero.  These are paths from factor three to factor and two, which we initially set to NA. We have assigned these paths to be from a normal and uniform distribution with initial parameters shown below.  Then we assign those initial values to the path coefficient matrix by combining them into one path coefficient matrix called BE.  
```{r, message=FALSE, warning=FALSE}
path <- matrix(0, 3, 3)
path[3, 1:2] <- NA
path.start <- matrix(0, 3, 3)
path.start[3, 1] <- "rnorm(1, 0.6, 0.05)"
path.start[3, 2] <- "runif(1, 0.3, 0.5)"
BE <- bind(path, path.start); BE

```
Finally, we can create the sem model that we will simulate to find the appropriate power.  We combine can each of the pieces that we developed earlier in the example, into one model titled SEM.model.

Then we simulate this model with 50 to 500 participants.  We then plot the output.

We can evaluate the simulation by reviewing graphs.  As we can see as n gets higher, the values for the sample size sensitive statistics increase.  For example, the chi-square, the statistic becomes larger with larger samples sizes.  We can see for statistics like the RMSEA, that this value hovers around .1, which relatively low.  

Finally, we can review what we want, which is the sample size requirements to conduct this SEM project.  We can look at the values for the two path coefficients regressed upon factor 1 and factor 2.  We can see that a sample size of 93 is required for this model, because that is the largest sample requirement. 

```{r, message=FALSE, warning=FALSE}
SEM.model <- model(BE=BE, LY=LY, RPS=RPS, RTE=RTE, modelType="SEM")

Output <- sim(NULL, n=50:500, SEM.model) 
plotCutoff(Output, 0.05)
Cpow <- getPower(Output)
findPower(Cpow, "N", 0.80)
```

