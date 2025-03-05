Lab 9 - HPC
================

# Learning goals

In this lab, you are expected to practice the following skills:

- Evaluate whether a problem can be parallelized or not.
- Practice with the parallel package.
- Use Rscript to submit jobs.

## Problem 1

Give yourself a few minutes to think about what you learned about
parallelization. List three examples of problems that you believe may be
solved using parallel computing, and check for packages on the HPC CRAN
task view that may be related to it.

*Tensorflow can do numerical computation using data flow graphs in a
parallel way. We can also parallelize cross-validation in machine
learning across folds using caret. We can do parallel model training
using mlr foreach, and doParallel and can do bootstrapping with parallel
or boot.*

## Problem 2: Pre-parallelization

The following functions can be written to be more efficient without
using `parallel`:

1.  This function generates a `n x k` dataset with all its entries
    having a Poisson distribution with mean `lambda`.

``` r
fun1 <- function(n = 100, k = 4, lambda = 4) {
  x <- NULL
  
  for (i in 1:n) # slow
    x <- rbind(x, rpois(k, lambda)) # needs to reallocate memory each time
  
  return(x)
}

fun1alt <- function(n = 100, k = 4, lambda = 4) {
  # YOUR CODE HERE
  matrix(rpois(n*k, lambda = lambda), ncol = k)
  # allows pre-allocating memory
  # rpois generates random numbers all at once
}

# Benchmarking
microbenchmark::microbenchmark(
  fun1(),
  fun1alt()
  # unit = "ns" for nanoseconds instead of microseconds
)
```

How much faster?

*The median time the new function takes seemts to be around 820-880
microseconds faster than the original function each time.*

2.  Find the column max (hint: Checkout the function `max.col()`).

``` r
# Data Generating Process (10 x 10,000 matrix)
set.seed(1234)
x <- matrix(rnorm(1e4), nrow=10)

# Find each column's max value
fun2 <- function(x) {
  apply(x, 2, max)
}

fun2alt <- function(x) {
  # YOUR CODE HERE
  x[cbind(max.col(t(x)), 1:ncol(x))]
  # avoids function calls inside loops
  # directly extracting the max values, without any loop
}

# Benchmarking
bench <- microbenchmark::microbenchmark(
  fun2(x),
  fun2alt(x)
  # unit = "us"
)
```

``` r
bench
```

    ## Unit: microseconds
    ##        expr    min      lq     mean median     uq     max neval
    ##     fun2(x) 2139.7 2270.10 2913.326 2391.9 2901.3 27949.7   100
    ##  fun2alt(x)  195.7  216.05  340.393  267.0  333.0  5825.6   100

``` r
plot(bench)
ggplot2::autoplot(bench) + ggplot2::theme_minimal()
```

![](lab09-hpc_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->![](lab09-hpc_files/figure-gfm/unnamed-chunk-2-2.png)<!-- -->

*As seen in the plots, the new function was faster for most cases, as
compared to the original function. The new function takes around a tenth
of the time the old function does. However, there seems to be at least
one case where the original function was faster.*

## Problem 3: Parallelize everything

We will now turn our attention to non-parametric
[bootstrapping](https://en.wikipedia.org/wiki/Bootstrapping_(statistics)).
Among its many uses, non-parametric bootstrapping allow us to obtain
confidence intervals for parameter estimates without relying on
parametric assumptions.

The main assumption is that we can approximate many experiments by
resampling observations from our original dataset, which reflects the
population.

This function implements the non-parametric bootstrap:

``` r
library(parallel)

my_boot <- function(dat, stat, R, ncpus = 1L) {
  # dat is the dataset
  # stat is the statistical function that we use to compute the estimates
  
  # Getting the random indices
  n <- nrow(dat)
  idx <- matrix(sample.int(n, n*R, TRUE), nrow=n, ncol=R) # resampling indices for bootstrapping
 
  # Making the cluster using `ncpus`
  # STEP 1: GOES HERE
  # create worker nodes
  # create cluster for parallel computing
  # ncpus specifies using multiple cpu cores
  # PSOCK - parallel socket cluster
  c1 <- makePSOCKcluster(ncpus)
  # STEP 2: GOES HERE
  # prevents memory leak by shutting down cluster, free up system resources
  # on.exit(stopCluster(c1)) # automatically stop
  
  # export the variables to the cluster
  # sending the variables to all worker nodes, each run in an isolated environment,
  # don't have access to global variables
  clusterExport(c1, varlist = c("idx", "dat", "stat"), envir = environment())
  
  # STEP 3: THIS FUNCTION NEEDS TO BE REPLACED WITH parLapply
  # ans <- lapply(seq_len(R), function(i) {
  #   stat(dat[idx[,i], , drop=FALSE])
  # })
  
  # replace sequential apply to parallelized apply
  ans <- parLapply(c1, seq_len(R), function(i) {
  stat(dat[idx[,i], , drop=FALSE])
  })
  
  # Coercing the list into a matrix
  ans <- do.call(rbind, ans)
  
  # STEP 4: GOES HERE
  stopCluster(c1) # manually stop
  
  
  ans
  
}
```

1.  Use the previous pseudocode, and make it work with `parallel`. Here
    is just an example for you to try:

``` r
# Bootstrap of a linear regression model
my_stat <- function(d) coef(lm(y~x, data = d))

# DATA SIM
set.seed(1)
n <- 500 
R <- 1e4
x <- cbind(rnorm(n)) 
y <- x*5 + rnorm(n)

# Check if we get something similar as lm
ans0 <- confint(lm(y~x))
cat("OLD CI \n")
print(ans0)

# use 4 CPU cores
ans1 <- my_boot(dat = data.frame(x, y), my_stat, R = R, ncpus = 4) 
qs <- c(0.025, 0.975)
cat("\nBootstrap CI \n")
print(t(apply(ans1, 2, quantile, probs = qs)))
```

*The resulting CIs are very close to the same, slight differences.*

2.  Check whether your version actually goes faster than the
    non-parallel version:

``` r
# your code here
parallel::detectCores()

# non parallel
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 1L))

# parallel 8 core
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 8L))
```

*Overall, the 8-core version runs much faster than the non-parallel
version (elapsed times of 5.25 seconds vs. 7.09 seconds). The user and
system time is higher for the parallel version, suggesting that it uses
more CPU resources.*

## Problem 4: Compile this markdown document using Rscript

Once you have saved this Rmd file, try running the following command in
your terminal:

``` bash
Rscript --vanilla -e 'rmarkdown::render("[full-path-to-your-Rmd-file.Rmd]")' &
```

Where `[full-path-to-your-Rmd-file.Rmd]` should be replace with the full
path to your Rmd file… :).
