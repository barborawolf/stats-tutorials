---
title: "Model fitting"
author: "LT"
output:
  html_document:
    toc: TRUE
    toc_float: TRUE
    df_print: "paged"
    keep_md: TRUE
---


```r
library(ggplot2)
library(GGally) # for a scatterplot matrix
```

# User-defined functions

We have seen functions at work in R already. A function takes one or more inputs in parentheses, and returns to us some output.

For example, the natural logarithm function.


```r
log(9000)
```

```
## [1] 9.10498
```

We may define our own functions in R. To define a function, we use `function()`. In the parentheses we give names to the expected input(s) of the function. We then open the curly braces `{}`. Inside the curly braces we write some R commands that carry out the operations we want our function to perform. In carrying out these operations, we refer to the inputs by the names we gave them. When our operations are complete, we 'return' the result. Whatever we place inside the parentheses of `return()` will be the output of the function. We can store a function using `=`, just as we store anything else.

We have seen a brief example of this already in a previous class, when we defined a function to calculate the square root of the mean.

As a further example, here is a function for calculating the estimated standard error of a mean, given the standard deviation `sd` and sample size `n` as inputs.


```r
se = function(sd, n){
  return(sd / sqrt(n))
}
```

Once we have stored a function, we may use it just like any other function in R. We refer to it by the name under which we stored it, and place the inputs in the parentheses, in the same order that we used when defining the function.

So to use the function above to calculate the standard error from an observed standard deviation of 2.5 and a sample size of 20:


```r
se(2.5, 20)
```

```
## [1] 0.559017
```

One common use of functions is to define a model that relates some predictor variable to an outcome variable. In this case, the inputs to the function are the values of the predictor variable, plus some parameters of the model, and the output of the function is the predicted value of the outcome variable.

For example, we can define a simple linear regression model. The function takes the value of the predictor variable *x* as its input, as well as the two parameters of the regression line, its intercept and its slope. For the sake of clarity, it is often a good idea to give the parameters of the model meaningful names.


```r
linear_mod = function(x, intercept, slope){
  return(intercept + slope*x)
}
```

When using a function as a model, we can make our R commands a little clearer by naming the parameter inputs, using `=`. This is optional, but it makes it easier to see what parameters are going into the model, and also makes it more obvious to us if we have entered them incorrectly.


```r
linear_mod(70, intercept=2, slope=0.01)
```

```
## [1] 2.7
```

Naming the inputs also ensures that the correct values are allocated to the correct parameters, even if we have entered them in a different order from the one we defined when creating the function.


```r
linear_mod(70, slope=0.01, intercept=2)
```

```
## [1] 2.7
```

# Model fitting

Once we have defined a model as a function relating predictor(s) to outcome, we can fit this model to a given set of data. Fitting the model to the data involves finding values for the parameters that produce predicted values of the outcome that in some way 'best fit' the actual observed values.

## Least squares

What defines a 'best fit'? There are several different ways of defining it. One is to consider the differences between the values that the model predicted and the observed values, and then to summarize these differences in some way. A common choice is to square the differences and calculate their mean (and then optionally take the square root of this number to bring the value back onto the original scale of measurement of the outcome). This measure of the fit of a model is termed the *R*oot *M*ean *S*quared *E*rror (RMSE), and we encountered it before when we used it as a measure of predictive accuracy in cross-validation.

The best fitting values of the model's parameters are those values that make this (or some other) measure of the error the smallest, meaning that this model's predictions are as close to the real observations as possible.

Since we have started working with functions, and since we will be calculating the RMSE a few times later on, let's define a function for calculating it. The function's inputs will be the predicted and observed values.


```r
rmse = function(predicted, observed){
  errors = predicted - observed
  sqerrors = errors^2
  return(sqrt(mean(sqerrors)))
}
```

You might be wondering why we bother to square the errors. One reason is that it ensures that all errors are positive, so that negative and positive errors do not cancel out. But if this were the only reason, we could just achieve the same thing by using the absolute values of the errors.

Using squared rather than absolute errors has another advantage. Unlike a function using absolute values, the RMSE penalizes large errors disproportionately more than it does small errors. To see why this may sometimes be desirable, consider the data below.

(We create some made-up data within R for the purposes of demonstration.)

No straight line provides a really great fit to these data, but intuitively the solid line that passes through the center of the group of observations is the best compromise. This line is indeed the one that minimizes the squared differences between its predictions for the outcome and the observed values of the outcome. It also minimizes the absolute differences.


```r
d = data.frame(Outcome=c(1,2,2,3), Predictor=c(1,1,3,3))

fig1 = ggplot(d, aes(y=Outcome, x=Predictor)) +
  geom_point() +
  geom_abline(intercept=1, slope=0.5) +
  coord_equal() +
  lims(y=c(0,4), x=c(0,4))

print(fig1 +
        geom_rect(xmin=0.5, xmax=1, ymin=1.5, ymax=2) +
        geom_rect(xmin=2.5, xmax=3, ymin=2.5, ymax=3) +
        geom_rect(xmin=0.5, xmax=1, ymin=1, ymax=1.5) +
        geom_rect(xmin=2.5, xmax=3, ymin=2, ymax=2.5))
```

![](Model-fitting_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

Now consider the dashed line below. This line does not accord with our intuition that the values of the outcome are on average increasing with the values of the predictor. Nonetheless, it still minimizes the absolute differences. However, it does not minimize the squared differences, because it produces some differences that are larger than any of those that the solid line produces.


```r
fig1 = fig1 + geom_abline(intercept=2, slope=0, lty='dashed')

print(fig1 +
        geom_rect(xmin=0, xmax=1, ymin=1, ymax=2) +
        geom_rect(xmin=2, xmax=3, ymin=2, ymax=3))
```

![](Model-fitting_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

There can be more than one line that minimizes the absolute differences, such as those shown below (and many more in between). But there is only ever one line that minimizes the squared differences.


```r
print(fig1 +
        geom_abline(intercept=0.5, slope=0.5, lty='dashed') +
        geom_abline(intercept=1.5, slope=0.5, lty='dashed'))
```

![](Model-fitting_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

So using the squared differences as our measure of fit will give us a unique best fitting line, and also one that intuitively 'goes through the middle' of the overall trend in the data, at least to the extent that the form of the model permits.

The method of 'least squares' consists in finding the values for a linear model that minimize the squared differences between predicted and observed values of the outcome.

Let's now see an example using the birth weights data. We will use the data to estimate a model of birth weight as a linear function of mother's weight. We do this first of all just using R's own `lm()` function, which applies least squares.


```r
bw = read.csv('birth_weights.csv')
model = lm(Birth_weight ~ Weight, bw)
print(model)
```

```
## 
## Call:
## lm(formula = Birth_weight ~ Weight, data = bw)
## 
## Coefficients:
## (Intercept)       Weight  
##    2.369624     0.009765
```

We can now calculate the RMSE for the estimated model, using the `rmse()` function that we defined ourselves.

For the linear model of birth weight that we just estimated, we can get the predicted values with the `predict()` function, and the observed values are just the observed birth weights in the data.


```r
rmse_fit = rmse(predict(model), bw$Birth_weight)
print(rmse_fit)
```

```
## [1] 0.714628
```

If `lm()` has done its work and has found the parameter values for the linear model that best fit the data, then this RMSE should be the smallest RMSE possible for this model family.

To see what sort of values of RMSE are possible for a linear model of these data, we can turn to the `linear_mod()` function that we defined for ourselves above. We can try inputting many different values of the intercept and slope parameters, along with the observed mothers' weights, and see what predicted birth weights these combinations of values give us. We can then input these predictions into our `rmse()` function along with the observed birth weights, to get an RMSE value for each combination of intercept and slope.

We begin by defining a range of values that we want to try out for each parameter. We can get a range of values using the R function `seq()` (short for *seq*uence). The inputs say where the sequence of values should begin, where it should end, and how many values to create in between (`length.out`).


```r
n_values = 100

intercepts = seq(2, 3, length.out=n_values)

slopes = seq(0, 0.02, length.out=n_values)
```

We would like to try out every combination of these values. The R function `expand.grid()` generates a data frame that contains all the possible combinations of two or more variables with multiple values.


```r
models = expand.grid(intercept=intercepts, slope=slopes)
head(models)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["intercept"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["slope"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"2.000000","2":"0","_rn_":"1"},{"1":"2.010101","2":"0","_rn_":"2"},{"1":"2.020202","2":"0","_rn_":"3"},{"1":"2.030303","2":"0","_rn_":"4"},{"1":"2.040404","2":"0","_rn_":"5"},{"1":"2.050505","2":"0","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Now we can go through each of these combinations of intercept and slope, and ask how well the resulting regression model fits the observed birth weights data. For this, we use both of our functions from above: `linear_mod()` to generate the model's predictions, and `rmse()` to compare these to the observed birth weights.

We first add a new column to the data frame of parameter combinations, then fill it in a loop.


```r
models$RMSE = 0

for(m in 1:nrow(models)){
  predicted = linear_mod(bw$Weight, intercept=models$intercept[m], slope=models$slope[m])
  models$RMSE[m] = rmse(predicted, bw$Birth_weight)
}

head(models)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["intercept"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["slope"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["RMSE"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"2.000000","2":"0","3":"1.192135","_rn_":"1"},{"1":"2.010101","2":"0","3":"1.184147","_rn_":"2"},{"1":"2.020202","2":"0","3":"1.176192","_rn_":"3"},{"1":"2.030303","2":"0","3":"1.168270","_rn_":"4"},{"1":"2.040404","2":"0","3":"1.160382","_rn_":"5"},{"1":"2.050505","2":"0","3":"1.152529","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We have now tried out many different combinations of values for intercept and slope. We can visualize how well these fit the data by mapping the range of slopes and intercepts to the *x* and *y* axes, and mapping their RMSE values to a color scale.

The `geom_raster()` function from ggplot can be used to fill the area of a plot with colored tiles. For a bit of extra clarity, we can also add to the plot a custom color scale, defining which colors we want to use for the low and high ends of the scale. Color gradients can be a little difficult to perceive in detail, so we also add contour lines to show where RMSE values are getting lower. Countour lines use the `z` aesthetic mapping to determine the countours. Finally, we add dashed lines representing the best fit values, which we get from the model that we fit with `lm()` above.


```r
fig2 = ggplot(models, aes(x=slope, y=intercept, z=RMSE, fill=RMSE)) +
  geom_raster() +
  geom_contour(binwidth=0.01, color='black') +
  scale_fill_gradient(low='red', high='yellow') +
  geom_hline(yintercept=coefficients(model)[1], lty='dashed') +
  geom_vline(xintercept=coefficients(model)[2], lty='dashed')

print(fig2)
```

![](Model-fitting_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

We see that there are various combinations of intercept and slope that fit the data almost as well as the best fit values, and the fit gets worse the further away from these values we go.

We can also check that among the models we tried out, the one that best fits the data has an RMSE that is still not quite as low as that of the best possible model for these data as fit by `lm()`.


```r
min(models$RMSE)
```

```
## [1] 0.7146286
```

```r
min(models$RMSE) < rmse_fit
```

```
## [1] FALSE
```

In our process of searching through lot of possible models, we get quite close to the best fitting values of intercept and slope.

(We can pick out the row of our data frame where RMSE is at its minimum, and compare the slope and intercept values here to those obtained by `lm()`.)


```r
print(models[models$RMSE == min(models$RMSE),])
```

```
##      intercept      slope      RMSE
## 4838  2.373737 0.00969697 0.7146286
```

```r
print(model)
```

```
## 
## Call:
## lm(formula = Birth_weight ~ Weight, data = bw)
## 
## Coefficients:
## (Intercept)       Weight  
##    2.369624     0.009765
```

## Search

How did `lm()` find the best fitting values that our search just missed? It turns out that for linear models there is just a proven formula for calculating the best fitting values of the model parameters. We can just plug the data and the model form into this formula and get a guaranteed correct answer.

This is not always the case. If the function that we wish to fit to the data is not a linear model, then we don't necessarily have a guaranteed formula for finding the best fitting parameter values.

Can we do better than just searching among the possible parameter values for the best fit we can find? Essentially, no. We have to just search. But we can search a bit more systematically than just trying lots and lots of combinations. Instead, we can start by trying just a few combinations that are fairly close to one another, see which is best, and then 'move' in the direction of the best one, so that next we try out combinations that are close to this one, and so on. In so doing, we move through the space of possible parameter values, always moving in the direction of better and better fits to the data, until we find that fits no longer get any better.

In the simple case where we only need to fit two parameters to the data, we can think of the space of possible parameters values as being a 3-dimensional landscape. Two of the dimensions are the two model parameters, and the third dimension is the fit to the data (RMSE or some other chosen measure), such that 'higher ground' indicates a worse fit, and 'lower ground' indicates a better fit. As we search through this landscape, we are looking for the lowest ground, so we move wherever the landscape leads downwards.

There are a few important subtleties to this procedure, and there are many, many variations on it, but this method is essentially how a lot of model fitting algorithms work.

Before we turn to applying this principle to a non-linear model, let's apply it to the linear regression model we defined above. This is of course unnecessary, because a linear model has a proven formula for its solution, but seeing the search in action on a very simple example will give us a better understanding of how it works before we apply it to a more complex case.

The R function `nls()` (which stands for *n*onlinear *l*east *s*quares) applies an algorithm like this to estimate the parameters of a nonlinear model from a set of data. Using `nls()` works a lot like `lm()`. We input a formula describing the model. This formula includes on the predictor side the function that we have created to represent our model. We put the predictor variable in the function input, along with the names of the model parameters, but without assigning them any values. We also input the data.

One important difference from `lm()` is that we need to tell the algorithm where to begin its search. For this, we need to input a list of starting values for the parameters. If we have plotted our data and have explored the behavior of our function, then we may have an approximate idea of where to search. If not, we can just take a guess or pick some default value. Since the algorithm will move towards better fitting values, it often doesn't matter if we pick an arbitrary starting point. To illustrate this here, we will pick very bad initial values for the intercept and slope.

As noted above, there are lots of different specific algorithms for searching the space of possible parameter values. `nls()` allows us to specify a particular algorithm. To illustrate this, we ask here for the `'port'` algorithm (the precise details of which are a bit much to go into here).

Finally, the `trace=TRUE` option can track for us what values of the model's parameters the search procedure goes through before finding the final fitted values.


```r
starting_values = list(intercept=90, slope=90)

fit = nls(Birth_weight ~ linear_mod(Weight, intercept, slope), bw,
          starting_values,
          algorithm='port',
          trace=TRUE)
```

```
##   0: 2.8883374e+09:  90.0000  90.0000
##   1: 2.8462903e+09:  58.1045  89.8460
##   2: 3.3994136e+08: -2806.77  74.7243
##   3:     5660175.2: -1046.01  17.6934
##   4:     48.260509:  2.36963 0.00976458
##   5:     48.260509:  2.36962 0.00976452
##   6:     48.260509:  2.36962 0.00976452
```

We can see from the `trace` output that the algorithm initially had to search around a bit. Each row is one step in the search, and the last two columns give the values of the intercept and slope at that step. The algorithm began by searching in a direction that was only approximately right. Because this was nonetheless a direction in which the fit quickly got a lot better, the algorithm 'sped up' its search by making larger steps. This led to an 'overshoot' for the intercept, going into negative values, but then the fact that the fit began getting worse in this direction steered the algorithm towards the region of the best fitting values. It then searched more finely for a couple more steps.

We can plot its path through the space of parameter values. Since the process of saving the search trace into a data frame is a bit tedious, I have pre-prepared this step so that we can load the values from a file.

(The 'RSS' column is the *R*esidual *S*um of *S*quares, a measure of model fit equivalent to the RMSE, just without the Root and Mean part.)


```r
trace = read.csv('model_fitting.csv')

trace
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Iteration"],"name":[1],"type":["int"],"align":["right"]},{"label":["RSS"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["intercept"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["slope"],"name":[4],"type":["dbl"],"align":["right"]}],"data":[{"1":"0","2":"2.888337e+09","3":"90.00000","4":"90.00000000"},{"1":"1","2":"2.846290e+09","3":"58.10450","4":"89.84600000"},{"1":"2","2":"3.399414e+08","3":"-2806.77000","4":"74.72430000"},{"1":"3","2":"5.660175e+06","3":"-1046.01000","4":"17.69340000"},{"1":"4","2":"4.826051e+01","3":"2.36963","4":"0.00976458"},{"1":"5","2":"4.826051e+01","3":"2.36962","4":"0.00976452"},{"1":"6","2":"4.826051e+01","3":"2.36962","4":"0.00976452"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

To give the plot an informative background we first generate a space of RMSE values, as we did for the plot above.


```r
models = expand.grid(intercept=seq(-3000, 100, length.out=n_values),
                     slope=seq(0, 100, length.out=n_values))

models$RMSE = 0
for(m in 1:nrow(models)){
  predicted = linear_mod(bw$Weight, intercept=models$intercept[m], slope=models$slope[m])
  models$RMSE[m] = rmse(predicted, bw$Birth_weight)
}

fig3 = ggplot(models, aes(x=slope, y=intercept)) +
  geom_raster(aes(fill=RMSE)) +
  geom_contour(aes(z=RMSE), binwidth=100, color='black') +
  geom_line(data=trace, lwd=1) +
  geom_point(data=trace, shape='circle filled', fill='grey', size=3) +
  scale_fill_gradient(low='red', high='yellow')

print(fig3)
```

![](Model-fitting_files/figure-html/unnamed-chunk-22-1.png)<!-- -->

In the printed output for the model we see the final fitted values of the intercept and slope, as we do with `lm()`.


```r
print(fit)
```

```
## Nonlinear regression model
##   model: Birth_weight ~ linear_mod(Weight, intercept, slope)
##    data: bw
## intercept     slope 
##  2.369624  0.009765 
##  residual sum-of-squares: 96.52
## 
## Algorithm "port", convergence message: both X-convergence and relative convergence (5)
```

Are these exactly the same as the absolute best values obtained from the formula that `lm()` uses?


```r
coefficients(fit)
```

```
##   intercept       slope 
## 2.369623516 0.009764519
```

```r
coefficients(model)
```

```
## (Intercept)      Weight 
## 2.369623518 0.009764519
```

```r
coefficients(fit) == coefficients(model)
```

```
## intercept     slope 
##     FALSE     FALSE
```

They are not exactly the same, though the difference is so small that it does not show in the printed output. This is because the search algorithm only ever gets steadily closer to the best-fitting values. At some point it needs to decide when to stop. It does this by checking how much the fit improves each time it searches further, and it stops when the improvement is below some very small value (often termed the 'tolerance'). If we really need extra fine precision in our estimates of the best-fitting parameters, we can adjust the tolerance, but this is not usually necessary.

# A non-linear model

Let's look at an example of fitting a non-linear function to some data. We will use data from a series of experiments on categorization. Participants in the experiments were shown images of crabs and lobsters. These images were morphed so that some of them were mostly lobster-like, some were mostly crab-like, and some were intermediate in shape. The participants then had to rate how lobster-like they thought each image was.

There was a bit more going on in the experiments, but this is the aspect that we will focus on. You can read more about the data [here](https://www.frontiersin.org/articles/10.3389/fpsyg.2018.01905/full).

The 'Image' variable tracks what image was being shown, on a standardized scale from 0 to 1, where 0 is maximally crab-like, and 1 is maximally lobster-like. The 'Rating' variable tracks the participants' ratings, on the same scale.


```r
crabsters = read.csv('crabsters.csv')
head(crabsters)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["Experiment"],"name":[1],"type":["fctr"],"align":["left"]},{"label":["ID"],"name":[2],"type":["int"],"align":["right"]},{"label":["Block"],"name":[3],"type":["int"],"align":["right"]},{"label":["Trial"],"name":[4],"type":["int"],"align":["right"]},{"label":["Adaptor"],"name":[5],"type":["fctr"],"align":["left"]},{"label":["Image"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["RT"],"name":[7],"type":["int"],"align":["right"]},{"label":["Rating"],"name":[8],"type":["dbl"],"align":["right"]}],"data":[{"1":"A","2":"1","3":"1","4":"1","5":"lobster","6":"0.2","7":"743","8":"0.00","_rn_":"1"},{"1":"A","2":"1","3":"1","4":"2","5":"lobster","6":"0.7","7":"2722","8":"0.25","_rn_":"2"},{"1":"A","2":"1","3":"1","4":"3","5":"crab","6":"0.4","7":"1372","8":"0.00","_rn_":"3"},{"1":"A","2":"1","3":"1","4":"4","5":"crab","6":"0.7","7":"1151","8":"0.75","_rn_":"4"},{"1":"A","2":"1","3":"1","4":"5","5":"crab","6":"0.2","7":"1085","8":"0.00","_rn_":"5"},{"1":"A","2":"1","3":"1","4":"6","5":"lobster","6":"0.7","7":"1867","8":"0.75","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We can see that each participant rated each image multiple times, over many trials of the experiment.


```r
table(crabsters$ID, crabsters$Image)
```

```
##     
##      0.2 0.3 0.4 0.5 0.6 0.7 0.8
##   1   32  32  32  32  32  32  32
##   2   32  32  32  32  32  32  32
##   3   32  32  32  32  32  32  32
##   4   32  32  32  32  32  30  32
##   5   32  32  32  32  32  32  32
##   6   31  32  31  32  32  31  31
##   7   32  32  31  31  32  31  32
##   8   32  32  32  32  32  32  32
##   9   22  27  29  23  30  32  30
##   11  32  32  32  32  32  32  32
##   12  32  31  30  32  31  31  31
##   13  31  31  31  31  32  32  30
##   14  32  32  32  32  32  32  32
##   15  32  32  32  32  32  32  32
##   16  31  32  32  32  32  32  32
##   17  32  32  32  32  32  32  32
##   18  32  32  32  32  32  32  32
##   19  32  32  32  32  32  32  32
##   20  32  31  32  32  32  32  32
##   21  32  32  31  32  32  32  32
##   22  32  32  31  32  32  32  32
##   23  29  32  30  29  31  32  29
##   24  32  32  32  32  32  32  32
##   25  32  32  32  32  32  32  32
##   26  32  32  32  32  32  32  32
##   27  32  32  32  32  32  32  32
##   28  32  32  32  32  32  32  32
##   30  32  32  32  32  32  32  32
##   31  32  31  32  32  32  32  32
##   32  32  32  32  32  32  32  32
##   33  32  32  32  32  32  32  32
##   34  32  32  32  32  31  32  31
##   35  32  32  32  32  32  32  32
##   36  31  32  31  32  32  32  32
##   37  30  32  32  32  32  32  32
##   38  32  32  31  32  32  32  32
##   39  32  32  32  32  32  32  31
##   40  32  32  32  32  32  32  32
##   41  32  32  32  32  32  32  32
##   42  32  32  32  32  32  32  32
##   43  32  32  32  32  31  32  32
##   44  32  32  32  32  32  32  32
##   45  32  32  32  32  32  32  32
##   46  32  32  32  32  32  32  32
##   47  31  32  32  32  32  32  32
##   48  28  25  28  24  28  27  26
##   49  32  32  32  32  32  32  32
##   50  32  32  32  32  32  32  32
##   51  32  32  32  32  32  32  32
##   52  32  32  32  32  32  31  32
##   53  32  32  32  31  32  32  32
##   54  32  32  32  32  32  32  32
##   55  32  32  32  32  32  31  30
##   56  32  32  32  32  32  32  32
##   57  32  32  31  32  32  32  32
##   58  32  32  32  32  32  32  31
##   59  32  32  32  32  32  32  32
##   60  31  32  32  32  31  30  30
##   61  32  32  32  32  32  32  32
##   62  32  32  32  32  32  32  32
##   63  32  32  32  32  32  32  32
##   64  32  31  32  32  31  32  32
##   65  32  32  32  31  31  32  31
##   66  32  32  32  32  32  32  32
##   67  32  32  31  32  32  32  31
##   68  28  29  29  29  28  30  29
##   69  32  32  32  32  32  32  32
##   70  32  32  32  32  32  32  32
##   71  31  31  31  30  31  32  30
##   72  32  32  32  32  32  32  32
##   73  32  32  32  32  32  32  32
##   74  32  32  32  32  32  32  32
##   75  31  32  32  32  32  32  32
##   76  19  19  20  19  21  21  16
##   77  32  32  31  32  31  32  32
##   78  32  32  32  32  32  32  32
##   79  32  32  32  32  32  32  32
##   80  32  32  32  31  32  32  32
```

To get an idea of each participant's general reaction to the images, averaging out some trial-to-trial variability, we will calculate the mean ratings per participant per image.


```r
mean_ratings = aggregate(Rating ~ Image*ID, crabsters, mean)
mean_ratings
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Image"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["ID"],"name":[2],"type":["int"],"align":["right"]},{"label":["Rating"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"0.2","2":"1","3":"0.07031250"},{"1":"0.3","2":"1","3":"0.17968750"},{"1":"0.4","2":"1","3":"0.24218750"},{"1":"0.5","2":"1","3":"0.28125000"},{"1":"0.6","2":"1","3":"0.45312500"},{"1":"0.7","2":"1","3":"0.55468750"},{"1":"0.8","2":"1","3":"0.70312500"},{"1":"0.2","2":"2","3":"0.11718750"},{"1":"0.3","2":"2","3":"0.20312500"},{"1":"0.4","2":"2","3":"0.30468750"},{"1":"0.5","2":"2","3":"0.40625000"},{"1":"0.6","2":"2","3":"0.60156250"},{"1":"0.7","2":"2","3":"0.64062500"},{"1":"0.8","2":"2","3":"0.77343750"},{"1":"0.2","2":"3","3":"0.17187500"},{"1":"0.3","2":"3","3":"0.19531250"},{"1":"0.4","2":"3","3":"0.19531250"},{"1":"0.5","2":"3","3":"0.32031250"},{"1":"0.6","2":"3","3":"0.61718750"},{"1":"0.7","2":"3","3":"0.71875000"},{"1":"0.8","2":"3","3":"0.73437500"},{"1":"0.2","2":"4","3":"0.21875000"},{"1":"0.3","2":"4","3":"0.32031250"},{"1":"0.4","2":"4","3":"0.36718750"},{"1":"0.5","2":"4","3":"0.42968750"},{"1":"0.6","2":"4","3":"0.53125000"},{"1":"0.7","2":"4","3":"0.57500000"},{"1":"0.8","2":"4","3":"0.66406250"},{"1":"0.2","2":"5","3":"0.10937500"},{"1":"0.3","2":"5","3":"0.14843750"},{"1":"0.4","2":"5","3":"0.18750000"},{"1":"0.5","2":"5","3":"0.40625000"},{"1":"0.6","2":"5","3":"0.63281250"},{"1":"0.7","2":"5","3":"0.85156250"},{"1":"0.8","2":"5","3":"0.91406250"},{"1":"0.2","2":"6","3":"0.22580645"},{"1":"0.3","2":"6","3":"0.25781250"},{"1":"0.4","2":"6","3":"0.31451613"},{"1":"0.5","2":"6","3":"0.36718750"},{"1":"0.6","2":"6","3":"0.43750000"},{"1":"0.7","2":"6","3":"0.52419355"},{"1":"0.8","2":"6","3":"0.63709677"},{"1":"0.2","2":"7","3":"0.10937500"},{"1":"0.3","2":"7","3":"0.17187500"},{"1":"0.4","2":"7","3":"0.26612903"},{"1":"0.5","2":"7","3":"0.40322581"},{"1":"0.6","2":"7","3":"0.57031250"},{"1":"0.7","2":"7","3":"0.63709677"},{"1":"0.8","2":"7","3":"0.71093750"},{"1":"0.2","2":"8","3":"0.07812500"},{"1":"0.3","2":"8","3":"0.05468750"},{"1":"0.4","2":"8","3":"0.11718750"},{"1":"0.5","2":"8","3":"0.24218750"},{"1":"0.6","2":"8","3":"0.57812500"},{"1":"0.7","2":"8","3":"0.68750000"},{"1":"0.8","2":"8","3":"0.71093750"},{"1":"0.2","2":"9","3":"0.01136364"},{"1":"0.3","2":"9","3":"0.00000000"},{"1":"0.4","2":"9","3":"0.00862069"},{"1":"0.5","2":"9","3":"0.01086957"},{"1":"0.6","2":"9","3":"0.06666667"},{"1":"0.7","2":"9","3":"0.55468750"},{"1":"0.8","2":"9","3":"0.90000000"},{"1":"0.2","2":"11","3":"0.22656250"},{"1":"0.3","2":"11","3":"0.34375000"},{"1":"0.4","2":"11","3":"0.46875000"},{"1":"0.5","2":"11","3":"0.50781250"},{"1":"0.6","2":"11","3":"0.60937500"},{"1":"0.7","2":"11","3":"0.72656250"},{"1":"0.8","2":"11","3":"0.75781250"},{"1":"0.2","2":"12","3":"0.08593750"},{"1":"0.3","2":"12","3":"0.15322581"},{"1":"0.4","2":"12","3":"0.20000000"},{"1":"0.5","2":"12","3":"0.39843750"},{"1":"0.6","2":"12","3":"0.58064516"},{"1":"0.7","2":"12","3":"0.65322581"},{"1":"0.8","2":"12","3":"0.66935484"},{"1":"0.2","2":"13","3":"0.12903226"},{"1":"0.3","2":"13","3":"0.29838710"},{"1":"0.4","2":"13","3":"0.31451613"},{"1":"0.5","2":"13","3":"0.34677419"},{"1":"0.6","2":"13","3":"0.39843750"},{"1":"0.7","2":"13","3":"0.43750000"},{"1":"0.8","2":"13","3":"0.54166667"},{"1":"0.2","2":"14","3":"0.04687500"},{"1":"0.3","2":"14","3":"0.03125000"},{"1":"0.4","2":"14","3":"0.11718750"},{"1":"0.5","2":"14","3":"0.23437500"},{"1":"0.6","2":"14","3":"0.43750000"},{"1":"0.7","2":"14","3":"0.66406250"},{"1":"0.8","2":"14","3":"0.73437500"},{"1":"0.2","2":"15","3":"0.02343750"},{"1":"0.3","2":"15","3":"0.10156250"},{"1":"0.4","2":"15","3":"0.28906250"},{"1":"0.5","2":"15","3":"0.39843750"},{"1":"0.6","2":"15","3":"0.53125000"},{"1":"0.7","2":"15","3":"0.64843750"},{"1":"0.8","2":"15","3":"0.70312500"},{"1":"0.2","2":"16","3":"0.25806452"},{"1":"0.3","2":"16","3":"0.27343750"},{"1":"0.4","2":"16","3":"0.28125000"},{"1":"0.5","2":"16","3":"0.37500000"},{"1":"0.6","2":"16","3":"0.42187500"},{"1":"0.7","2":"16","3":"0.57031250"},{"1":"0.8","2":"16","3":"0.60937500"},{"1":"0.2","2":"17","3":"0.02343750"},{"1":"0.3","2":"17","3":"0.00000000"},{"1":"0.4","2":"17","3":"0.03125000"},{"1":"0.5","2":"17","3":"0.06250000"},{"1":"0.6","2":"17","3":"0.25781250"},{"1":"0.7","2":"17","3":"0.65625000"},{"1":"0.8","2":"17","3":"0.85156250"},{"1":"0.2","2":"18","3":"0.03125000"},{"1":"0.3","2":"18","3":"0.18750000"},{"1":"0.4","2":"18","3":"0.35156250"},{"1":"0.5","2":"18","3":"0.41406250"},{"1":"0.6","2":"18","3":"0.62500000"},{"1":"0.7","2":"18","3":"0.84375000"},{"1":"0.8","2":"18","3":"0.99218750"},{"1":"0.2","2":"19","3":"0.03125000"},{"1":"0.3","2":"19","3":"0.04687500"},{"1":"0.4","2":"19","3":"0.11718750"},{"1":"0.5","2":"19","3":"0.21093750"},{"1":"0.6","2":"19","3":"0.33593750"},{"1":"0.7","2":"19","3":"0.42187500"},{"1":"0.8","2":"19","3":"0.50781250"},{"1":"0.2","2":"20","3":"0.22656250"},{"1":"0.3","2":"20","3":"0.30645161"},{"1":"0.4","2":"20","3":"0.44531250"},{"1":"0.5","2":"20","3":"0.45312500"},{"1":"0.6","2":"20","3":"0.56250000"},{"1":"0.7","2":"20","3":"0.70312500"},{"1":"0.8","2":"20","3":"0.64843750"},{"1":"0.2","2":"21","3":"0.02343750"},{"1":"0.3","2":"21","3":"0.00781250"},{"1":"0.4","2":"21","3":"0.00000000"},{"1":"0.5","2":"21","3":"0.10156250"},{"1":"0.6","2":"21","3":"0.43750000"},{"1":"0.7","2":"21","3":"0.69531250"},{"1":"0.8","2":"21","3":"0.74218750"},{"1":"0.2","2":"22","3":"0.08593750"},{"1":"0.3","2":"22","3":"0.10156250"},{"1":"0.4","2":"22","3":"0.18548387"},{"1":"0.5","2":"22","3":"0.24218750"},{"1":"0.6","2":"22","3":"0.31250000"},{"1":"0.7","2":"22","3":"0.38281250"},{"1":"0.8","2":"22","3":"0.42968750"},{"1":"0.2","2":"23","3":"0.10344828"},{"1":"0.3","2":"23","3":"0.23437500"},{"1":"0.4","2":"23","3":"0.38333333"},{"1":"0.5","2":"23","3":"0.40517241"},{"1":"0.6","2":"23","3":"0.49193548"},{"1":"0.7","2":"23","3":"0.57812500"},{"1":"0.8","2":"23","3":"0.69827586"},{"1":"0.2","2":"24","3":"0.06250000"},{"1":"0.3","2":"24","3":"0.07031250"},{"1":"0.4","2":"24","3":"0.09375000"},{"1":"0.5","2":"24","3":"0.14843750"},{"1":"0.6","2":"24","3":"0.25000000"},{"1":"0.7","2":"24","3":"0.33593750"},{"1":"0.8","2":"24","3":"0.46093750"},{"1":"0.2","2":"25","3":"0.00000000"},{"1":"0.3","2":"25","3":"0.00000000"},{"1":"0.4","2":"25","3":"0.00000000"},{"1":"0.5","2":"25","3":"0.07812500"},{"1":"0.6","2":"25","3":"0.32031250"},{"1":"0.7","2":"25","3":"0.57812500"},{"1":"0.8","2":"25","3":"0.75781250"},{"1":"0.2","2":"26","3":"0.02343750"},{"1":"0.3","2":"26","3":"0.01562500"},{"1":"0.4","2":"26","3":"0.03906250"},{"1":"0.5","2":"26","3":"0.09375000"},{"1":"0.6","2":"26","3":"0.24218750"},{"1":"0.7","2":"26","3":"0.46093750"},{"1":"0.8","2":"26","3":"0.68750000"},{"1":"0.2","2":"27","3":"0.00000000"},{"1":"0.3","2":"27","3":"0.00000000"},{"1":"0.4","2":"27","3":"0.01562500"},{"1":"0.5","2":"27","3":"0.08593750"},{"1":"0.6","2":"27","3":"0.21875000"},{"1":"0.7","2":"27","3":"0.32031250"},{"1":"0.8","2":"27","3":"0.43750000"},{"1":"0.2","2":"28","3":"0.21093750"},{"1":"0.3","2":"28","3":"0.28906250"},{"1":"0.4","2":"28","3":"0.32812500"},{"1":"0.5","2":"28","3":"0.42968750"},{"1":"0.6","2":"28","3":"0.53125000"},{"1":"0.7","2":"28","3":"0.67187500"},{"1":"0.8","2":"28","3":"0.83593750"},{"1":"0.2","2":"30","3":"0.25000000"},{"1":"0.3","2":"30","3":"0.35937500"},{"1":"0.4","2":"30","3":"0.47656250"},{"1":"0.5","2":"30","3":"0.50000000"},{"1":"0.6","2":"30","3":"0.52343750"},{"1":"0.7","2":"30","3":"0.62500000"},{"1":"0.8","2":"30","3":"0.72656250"},{"1":"0.2","2":"31","3":"0.00781250"},{"1":"0.3","2":"31","3":"0.05645161"},{"1":"0.4","2":"31","3":"0.10937500"},{"1":"0.5","2":"31","3":"0.18750000"},{"1":"0.6","2":"31","3":"0.41406250"},{"1":"0.7","2":"31","3":"0.53125000"},{"1":"0.8","2":"31","3":"0.65625000"},{"1":"0.2","2":"32","3":"0.00000000"},{"1":"0.3","2":"32","3":"0.00781250"},{"1":"0.4","2":"32","3":"0.00781250"},{"1":"0.5","2":"32","3":"0.19531250"},{"1":"0.6","2":"32","3":"0.39843750"},{"1":"0.7","2":"32","3":"0.49218750"},{"1":"0.8","2":"32","3":"0.51562500"},{"1":"0.2","2":"33","3":"0.00781250"},{"1":"0.3","2":"33","3":"0.00000000"},{"1":"0.4","2":"33","3":"0.07031250"},{"1":"0.5","2":"33","3":"0.21093750"},{"1":"0.6","2":"33","3":"0.53125000"},{"1":"0.7","2":"33","3":"0.75781250"},{"1":"0.8","2":"33","3":"0.85937500"},{"1":"0.2","2":"34","3":"0.00000000"},{"1":"0.3","2":"34","3":"0.00000000"},{"1":"0.4","2":"34","3":"0.00781250"},{"1":"0.5","2":"34","3":"0.06250000"},{"1":"0.6","2":"34","3":"0.29838710"},{"1":"0.7","2":"34","3":"0.52343750"},{"1":"0.8","2":"34","3":"0.78225806"},{"1":"0.2","2":"35","3":"0.03906250"},{"1":"0.3","2":"35","3":"0.04687500"},{"1":"0.4","2":"35","3":"0.10156250"},{"1":"0.5","2":"35","3":"0.14843750"},{"1":"0.6","2":"35","3":"0.44531250"},{"1":"0.7","2":"35","3":"0.61718750"},{"1":"0.8","2":"35","3":"0.71875000"},{"1":"0.2","2":"36","3":"0.17741935"},{"1":"0.3","2":"36","3":"0.20312500"},{"1":"0.4","2":"36","3":"0.22580645"},{"1":"0.5","2":"36","3":"0.35156250"},{"1":"0.6","2":"36","3":"0.43750000"},{"1":"0.7","2":"36","3":"0.58593750"},{"1":"0.8","2":"36","3":"0.67187500"},{"1":"0.2","2":"37","3":"0.03333333"},{"1":"0.3","2":"37","3":"0.07031250"},{"1":"0.4","2":"37","3":"0.16406250"},{"1":"0.5","2":"37","3":"0.28906250"},{"1":"0.6","2":"37","3":"0.51562500"},{"1":"0.7","2":"37","3":"0.75781250"},{"1":"0.8","2":"37","3":"0.85156250"},{"1":"0.2","2":"38","3":"0.07031250"},{"1":"0.3","2":"38","3":"0.16406250"},{"1":"0.4","2":"38","3":"0.23387097"},{"1":"0.5","2":"38","3":"0.28125000"},{"1":"0.6","2":"38","3":"0.50000000"},{"1":"0.7","2":"38","3":"0.70312500"},{"1":"0.8","2":"38","3":"0.82031250"},{"1":"0.2","2":"39","3":"0.15625000"},{"1":"0.3","2":"39","3":"0.21875000"},{"1":"0.4","2":"39","3":"0.28906250"},{"1":"0.5","2":"39","3":"0.36718750"},{"1":"0.6","2":"39","3":"0.45312500"},{"1":"0.7","2":"39","3":"0.52343750"},{"1":"0.8","2":"39","3":"0.56451613"},{"1":"0.2","2":"40","3":"0.00000000"},{"1":"0.3","2":"40","3":"0.03906250"},{"1":"0.4","2":"40","3":"0.00000000"},{"1":"0.5","2":"40","3":"0.15625000"},{"1":"0.6","2":"40","3":"0.54687500"},{"1":"0.7","2":"40","3":"0.80468750"},{"1":"0.8","2":"40","3":"0.93750000"},{"1":"0.2","2":"41","3":"0.00000000"},{"1":"0.3","2":"41","3":"0.00000000"},{"1":"0.4","2":"41","3":"0.03125000"},{"1":"0.5","2":"41","3":"0.03906250"},{"1":"0.6","2":"41","3":"0.34375000"},{"1":"0.7","2":"41","3":"0.83593750"},{"1":"0.8","2":"41","3":"0.91406250"},{"1":"0.2","2":"42","3":"0.20312500"},{"1":"0.3","2":"42","3":"0.24218750"},{"1":"0.4","2":"42","3":"0.31250000"},{"1":"0.5","2":"42","3":"0.38281250"},{"1":"0.6","2":"42","3":"0.43750000"},{"1":"0.7","2":"42","3":"0.59375000"},{"1":"0.8","2":"42","3":"0.69531250"},{"1":"0.2","2":"43","3":"0.07812500"},{"1":"0.3","2":"43","3":"0.21093750"},{"1":"0.4","2":"43","3":"0.25000000"},{"1":"0.5","2":"43","3":"0.29687500"},{"1":"0.6","2":"43","3":"0.50000000"},{"1":"0.7","2":"43","3":"0.55468750"},{"1":"0.8","2":"43","3":"0.69531250"},{"1":"0.2","2":"44","3":"0.20312500"},{"1":"0.3","2":"44","3":"0.24218750"},{"1":"0.4","2":"44","3":"0.24218750"},{"1":"0.5","2":"44","3":"0.27343750"},{"1":"0.6","2":"44","3":"0.46875000"},{"1":"0.7","2":"44","3":"0.59375000"},{"1":"0.8","2":"44","3":"0.65625000"},{"1":"0.2","2":"45","3":"0.25000000"},{"1":"0.3","2":"45","3":"0.21875000"},{"1":"0.4","2":"45","3":"0.23437500"},{"1":"0.5","2":"45","3":"0.32031250"},{"1":"0.6","2":"45","3":"0.47656250"},{"1":"0.7","2":"45","3":"0.53906250"},{"1":"0.8","2":"45","3":"0.60156250"},{"1":"0.2","2":"46","3":"0.03125000"},{"1":"0.3","2":"46","3":"0.07812500"},{"1":"0.4","2":"46","3":"0.10156250"},{"1":"0.5","2":"46","3":"0.36718750"},{"1":"0.6","2":"46","3":"0.60156250"},{"1":"0.7","2":"46","3":"0.73437500"},{"1":"0.8","2":"46","3":"0.82031250"},{"1":"0.2","2":"47","3":"0.01612903"},{"1":"0.3","2":"47","3":"0.00781250"},{"1":"0.4","2":"47","3":"0.02343750"},{"1":"0.5","2":"47","3":"0.07812500"},{"1":"0.6","2":"47","3":"0.24218750"},{"1":"0.7","2":"47","3":"0.60156250"},{"1":"0.8","2":"47","3":"0.70312500"},{"1":"0.2","2":"48","3":"0.11607143"},{"1":"0.3","2":"48","3":"0.15000000"},{"1":"0.4","2":"48","3":"0.17857143"},{"1":"0.5","2":"48","3":"0.21875000"},{"1":"0.6","2":"48","3":"0.30357143"},{"1":"0.7","2":"48","3":"0.41666667"},{"1":"0.8","2":"48","3":"0.55769231"},{"1":"0.2","2":"49","3":"0.00000000"},{"1":"0.3","2":"49","3":"0.09375000"},{"1":"0.4","2":"49","3":"0.15625000"},{"1":"0.5","2":"49","3":"0.29687500"},{"1":"0.6","2":"49","3":"0.75000000"},{"1":"0.7","2":"49","3":"0.98437500"},{"1":"0.8","2":"49","3":"0.99218750"},{"1":"0.2","2":"50","3":"0.00000000"},{"1":"0.3","2":"50","3":"0.03125000"},{"1":"0.4","2":"50","3":"0.15625000"},{"1":"0.5","2":"50","3":"0.26562500"},{"1":"0.6","2":"50","3":"0.46875000"},{"1":"0.7","2":"50","3":"0.59375000"},{"1":"0.8","2":"50","3":"0.81250000"},{"1":"0.2","2":"51","3":"0.13281250"},{"1":"0.3","2":"51","3":"0.14843750"},{"1":"0.4","2":"51","3":"0.18750000"},{"1":"0.5","2":"51","3":"0.28125000"},{"1":"0.6","2":"51","3":"0.46093750"},{"1":"0.7","2":"51","3":"0.68750000"},{"1":"0.8","2":"51","3":"0.86718750"},{"1":"0.2","2":"52","3":"0.06250000"},{"1":"0.3","2":"52","3":"0.19531250"},{"1":"0.4","2":"52","3":"0.24218750"},{"1":"0.5","2":"52","3":"0.31250000"},{"1":"0.6","2":"52","3":"0.49218750"},{"1":"0.7","2":"52","3":"0.55645161"},{"1":"0.8","2":"52","3":"0.72656250"},{"1":"0.2","2":"53","3":"0.07812500"},{"1":"0.3","2":"53","3":"0.25781250"},{"1":"0.4","2":"53","3":"0.45312500"},{"1":"0.5","2":"53","3":"0.54032258"},{"1":"0.6","2":"53","3":"0.66406250"},{"1":"0.7","2":"53","3":"0.81250000"},{"1":"0.8","2":"53","3":"0.90625000"},{"1":"0.2","2":"54","3":"0.08593750"},{"1":"0.3","2":"54","3":"0.17968750"},{"1":"0.4","2":"54","3":"0.32812500"},{"1":"0.5","2":"54","3":"0.57812500"},{"1":"0.6","2":"54","3":"0.67968750"},{"1":"0.7","2":"54","3":"0.71093750"},{"1":"0.8","2":"54","3":"0.81250000"},{"1":"0.2","2":"55","3":"0.00781250"},{"1":"0.3","2":"55","3":"0.03125000"},{"1":"0.4","2":"55","3":"0.03125000"},{"1":"0.5","2":"55","3":"0.03906250"},{"1":"0.6","2":"55","3":"0.17968750"},{"1":"0.7","2":"55","3":"0.23387097"},{"1":"0.8","2":"55","3":"0.25000000"},{"1":"0.2","2":"56","3":"0.00781250"},{"1":"0.3","2":"56","3":"0.00781250"},{"1":"0.4","2":"56","3":"0.03906250"},{"1":"0.5","2":"56","3":"0.14843750"},{"1":"0.6","2":"56","3":"0.34375000"},{"1":"0.7","2":"56","3":"0.51562500"},{"1":"0.8","2":"56","3":"0.69531250"},{"1":"0.2","2":"57","3":"0.00781250"},{"1":"0.3","2":"57","3":"0.02343750"},{"1":"0.4","2":"57","3":"0.04838710"},{"1":"0.5","2":"57","3":"0.10937500"},{"1":"0.6","2":"57","3":"0.25781250"},{"1":"0.7","2":"57","3":"0.42968750"},{"1":"0.8","2":"57","3":"0.56250000"},{"1":"0.2","2":"58","3":"0.07031250"},{"1":"0.3","2":"58","3":"0.12500000"},{"1":"0.4","2":"58","3":"0.20312500"},{"1":"0.5","2":"58","3":"0.29687500"},{"1":"0.6","2":"58","3":"0.50000000"},{"1":"0.7","2":"58","3":"0.70312500"},{"1":"0.8","2":"58","3":"0.81451613"},{"1":"0.2","2":"59","3":"0.05468750"},{"1":"0.3","2":"59","3":"0.09375000"},{"1":"0.4","2":"59","3":"0.21093750"},{"1":"0.5","2":"59","3":"0.32812500"},{"1":"0.6","2":"59","3":"0.46875000"},{"1":"0.7","2":"59","3":"0.55468750"},{"1":"0.8","2":"59","3":"0.67187500"},{"1":"0.2","2":"60","3":"0.01612903"},{"1":"0.3","2":"60","3":"0.24218750"},{"1":"0.4","2":"60","3":"0.45312500"},{"1":"0.5","2":"60","3":"0.42968750"},{"1":"0.6","2":"60","3":"0.65322581"},{"1":"0.7","2":"60","3":"0.82500000"},{"1":"0.8","2":"60","3":"0.93333333"},{"1":"0.2","2":"61","3":"0.11718750"},{"1":"0.3","2":"61","3":"0.16406250"},{"1":"0.4","2":"61","3":"0.23437500"},{"1":"0.5","2":"61","3":"0.35937500"},{"1":"0.6","2":"61","3":"0.53906250"},{"1":"0.7","2":"61","3":"0.64843750"},{"1":"0.8","2":"61","3":"0.71093750"},{"1":"0.2","2":"62","3":"0.20312500"},{"1":"0.3","2":"62","3":"0.30468750"},{"1":"0.4","2":"62","3":"0.46875000"},{"1":"0.5","2":"62","3":"0.46093750"},{"1":"0.6","2":"62","3":"0.50000000"},{"1":"0.7","2":"62","3":"0.71093750"},{"1":"0.8","2":"62","3":"0.75781250"},{"1":"0.2","2":"63","3":"0.08593750"},{"1":"0.3","2":"63","3":"0.21093750"},{"1":"0.4","2":"63","3":"0.28906250"},{"1":"0.5","2":"63","3":"0.38281250"},{"1":"0.6","2":"63","3":"0.53906250"},{"1":"0.7","2":"63","3":"0.81250000"},{"1":"0.8","2":"63","3":"0.93750000"},{"1":"0.2","2":"64","3":"0.09375000"},{"1":"0.3","2":"64","3":"0.10483871"},{"1":"0.4","2":"64","3":"0.25000000"},{"1":"0.5","2":"64","3":"0.25000000"},{"1":"0.6","2":"64","3":"0.30645161"},{"1":"0.7","2":"64","3":"0.49218750"},{"1":"0.8","2":"64","3":"0.50781250"},{"1":"0.2","2":"65","3":"0.06250000"},{"1":"0.3","2":"65","3":"0.05468750"},{"1":"0.4","2":"65","3":"0.11718750"},{"1":"0.5","2":"65","3":"0.19354839"},{"1":"0.6","2":"65","3":"0.23387097"},{"1":"0.7","2":"65","3":"0.34375000"},{"1":"0.8","2":"65","3":"0.43548387"},{"1":"0.2","2":"66","3":"0.09375000"},{"1":"0.3","2":"66","3":"0.06250000"},{"1":"0.4","2":"66","3":"0.12500000"},{"1":"0.5","2":"66","3":"0.39062500"},{"1":"0.6","2":"66","3":"0.53125000"},{"1":"0.7","2":"66","3":"0.57812500"},{"1":"0.8","2":"66","3":"0.71875000"},{"1":"0.2","2":"67","3":"0.05468750"},{"1":"0.3","2":"67","3":"0.15625000"},{"1":"0.4","2":"67","3":"0.30645161"},{"1":"0.5","2":"67","3":"0.29687500"},{"1":"0.6","2":"67","3":"0.42968750"},{"1":"0.7","2":"67","3":"0.64062500"},{"1":"0.8","2":"67","3":"0.85483871"},{"1":"0.2","2":"68","3":"0.01785714"},{"1":"0.3","2":"68","3":"0.15517241"},{"1":"0.4","2":"68","3":"0.43103448"},{"1":"0.5","2":"68","3":"0.46551724"},{"1":"0.6","2":"68","3":"0.62500000"},{"1":"0.7","2":"68","3":"0.81666667"},{"1":"0.8","2":"68","3":"0.91379310"},{"1":"0.2","2":"69","3":"0.07812500"},{"1":"0.3","2":"69","3":"0.28125000"},{"1":"0.4","2":"69","3":"0.44531250"},{"1":"0.5","2":"69","3":"0.46093750"},{"1":"0.6","2":"69","3":"0.60937500"},{"1":"0.7","2":"69","3":"0.74218750"},{"1":"0.8","2":"69","3":"0.74218750"},{"1":"0.2","2":"70","3":"0.00000000"},{"1":"0.3","2":"70","3":"0.01562500"},{"1":"0.4","2":"70","3":"0.05468750"},{"1":"0.5","2":"70","3":"0.12500000"},{"1":"0.6","2":"70","3":"0.47656250"},{"1":"0.7","2":"70","3":"0.71093750"},{"1":"0.8","2":"70","3":"0.89843750"},{"1":"0.2","2":"71","3":"0.11290323"},{"1":"0.3","2":"71","3":"0.09677419"},{"1":"0.4","2":"71","3":"0.07258065"},{"1":"0.5","2":"71","3":"0.11666667"},{"1":"0.6","2":"71","3":"0.33064516"},{"1":"0.7","2":"71","3":"0.58593750"},{"1":"0.8","2":"71","3":"0.64166667"},{"1":"0.2","2":"72","3":"0.00000000"},{"1":"0.3","2":"72","3":"0.03906250"},{"1":"0.4","2":"72","3":"0.16406250"},{"1":"0.5","2":"72","3":"0.23437500"},{"1":"0.6","2":"72","3":"0.39062500"},{"1":"0.7","2":"72","3":"0.56250000"},{"1":"0.8","2":"72","3":"0.66406250"},{"1":"0.2","2":"73","3":"0.00000000"},{"1":"0.3","2":"73","3":"0.02343750"},{"1":"0.4","2":"73","3":"0.06250000"},{"1":"0.5","2":"73","3":"0.17187500"},{"1":"0.6","2":"73","3":"0.46875000"},{"1":"0.7","2":"73","3":"0.82031250"},{"1":"0.8","2":"73","3":"0.97656250"},{"1":"0.2","2":"74","3":"0.26562500"},{"1":"0.3","2":"74","3":"0.24218750"},{"1":"0.4","2":"74","3":"0.25781250"},{"1":"0.5","2":"74","3":"0.38281250"},{"1":"0.6","2":"74","3":"0.64843750"},{"1":"0.7","2":"74","3":"0.76562500"},{"1":"0.8","2":"74","3":"0.78906250"},{"1":"0.2","2":"75","3":"0.00000000"},{"1":"0.3","2":"75","3":"0.00781250"},{"1":"0.4","2":"75","3":"0.00000000"},{"1":"0.5","2":"75","3":"0.03125000"},{"1":"0.6","2":"75","3":"0.21875000"},{"1":"0.7","2":"75","3":"0.29687500"},{"1":"0.8","2":"75","3":"0.34375000"},{"1":"0.2","2":"76","3":"0.05263158"},{"1":"0.3","2":"76","3":"0.05263158"},{"1":"0.4","2":"76","3":"0.08750000"},{"1":"0.5","2":"76","3":"0.10526316"},{"1":"0.6","2":"76","3":"0.17857143"},{"1":"0.7","2":"76","3":"0.50000000"},{"1":"0.8","2":"76","3":"0.64062500"},{"1":"0.2","2":"77","3":"0.01562500"},{"1":"0.3","2":"77","3":"0.25000000"},{"1":"0.4","2":"77","3":"0.47580645"},{"1":"0.5","2":"77","3":"0.49218750"},{"1":"0.6","2":"77","3":"0.64516129"},{"1":"0.7","2":"77","3":"0.80468750"},{"1":"0.8","2":"77","3":"0.88281250"},{"1":"0.2","2":"78","3":"0.03906250"},{"1":"0.3","2":"78","3":"0.24218750"},{"1":"0.4","2":"78","3":"0.42187500"},{"1":"0.5","2":"78","3":"0.48437500"},{"1":"0.6","2":"78","3":"0.71875000"},{"1":"0.7","2":"78","3":"0.79687500"},{"1":"0.8","2":"78","3":"0.85937500"},{"1":"0.2","2":"79","3":"0.01562500"},{"1":"0.3","2":"79","3":"0.24218750"},{"1":"0.4","2":"79","3":"0.44531250"},{"1":"0.5","2":"79","3":"0.52343750"},{"1":"0.6","2":"79","3":"0.67968750"},{"1":"0.7","2":"79","3":"0.74218750"},{"1":"0.8","2":"79","3":"0.89843750"},{"1":"0.2","2":"80","3":"0.00781250"},{"1":"0.3","2":"80","3":"0.26562500"},{"1":"0.4","2":"80","3":"0.45312500"},{"1":"0.5","2":"80","3":"0.46774194"},{"1":"0.6","2":"80","3":"0.65625000"},{"1":"0.7","2":"80","3":"0.76562500"},{"1":"0.8","2":"80","3":"0.88281250"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Let's now plot these ratings. For a bit of extra plotting practice, we add some new features to the plot:

* overall means as a line, using `stat_summary()`, with `mean` as the summary function for the *y* values
* fixed limits for the *x* axis, using `lims()`
* custom labels for the *y* axis showing the meaning of the rating values


```r
fig_crabsters = ggplot(mean_ratings, aes(y=Rating, x=Image)) +
  geom_point(shape='circle filled', fill='grey', position=position_jitter(height=0, width=0.02)) +
  stat_summary(geom='line', fun.y=mean) +
  lims(x=c(0,1)) +
  scale_y_continuous(breaks=c(0,0.5,1), labels=c('0: Crab',0.5,'1: Lobster')) +
  labs(caption='Data: Reindl et al. (2018)')

print(fig_crabsters)
```

![](Model-fitting_files/figure-html/unnamed-chunk-28-1.png)<!-- -->

Two features of the data are important for us here. First, we appear to have an approximately S-shaped function relating image type to rating. So the participants do not change their opinion of the image by some constant amount as the image changes. Second, the participants do not seem to want to use the full range of the rating scale all the way up to full lobster. (This makes some sense when you look at the 'lobster' images used in the experiments. They don't look so lobster-like.)

One of the secondary objectives of this line of research was to quantify how much more 'suddenly' different kinds of people (novices and experts) change their minds about the categorization of an object when its shape changes. In terms of the trend we see on the plot above, a more 'sudden' switch from one category to another at a certain crucial point in the range of images would be apparent as a steeper, more 'S-like' curve.

We can check how the different participants behaved by faceting our plot by the variable that tracks participant ID.


```r
fig_crabsters_p = fig_crabsters + facet_wrap(~ID)

print(fig_crabsters_p)
```

![](Model-fitting_files/figure-html/unnamed-chunk-29-1.png)<!-- -->

To quantify the steepness of the individual participant curves, we can create a model that represents this S-shaped curve, with one of its parameters controlling the steepness of the curve, and then estimate the parameters of the model for each participant. Additionally, such a model will need to take into account the fact that ratings often 'level-off' before reaching the top of the lobsterness scale.

Here is such a model, adapted from an earlier study by the same authors (which you can read about [here](https://www.sciencedirect.com/science/article/pii/S0044523115000224)). As with the simpler linear model example above, we have expressed the model as an R function that takes the predictor value (here the image type) as its first input, followed by the values of the model's parameters, and outputs the predicted value of the outcome (here the rating).

We have given meaningful names to the parameters, that express what aspect of the relationship between predictor and outcome they control:

* asymptote: how far up the rating scale the ratings go
* location: where on the image scale the curve is located
* steepness: how 'steeply' the ratings change


```r
f = function(x, asymptote, location ,steepness){
  return(asymptote*(1-2^-((1-x)/(1-location))^-steepness))
}
```

First we should check how the function behaves. We can generate a sequence of *x* values, use our function to calculate the *y* values for certain combinations of parameters, then plot the results.

To keep the plot from getting too cluttered, we try just a few values for each parameter.

(Putting together the data frame of predicted values for each combination of parameter values looks a bit complicated, but uses techniques we have already seen above. We first generate the possible combinations of parameter values, using `expand.grid()` as we did above, but including also the possible *x* values among the combinations, then we input the columns of the resulting data frame into our function to generate the predicted *y* values.)


```r
f_data = expand.grid(x=seq(0,1,length.out=100),
                     asymptote=c(0.5,0.8,1),
                     location=c(0.2,0.5,0.8),
                     steepness=c(2,10))

f_data$y = f(f_data$x,
             asymptote=f_data$asymptote,
             location=f_data$location,
             steepness=f_data$steepness)

f_data$steepness = as.factor(f_data$steepness) # (only because we want steepness as discrete line types)

fig_fun = ggplot(f_data, aes(x=x, y=y, lty=steepness)) +
  geom_line() +
  facet_grid(asymptote ~ location, as.table=FALSE, labeller=label_both)

print(fig_fun)
```

![](Model-fitting_files/figure-html/unnamed-chunk-31-1.png)<!-- -->

As a first step, let's fit the model to the overall mean ratings instead of separately for each participant.

Our model is not linear, so it doesn't have a nice proven formula for estimation like the one that `lm()` uses to find the least squares solution for a linear regression. But there is still a solution somewhere among the possible combinations of parameter values. To get a first idea of where it might be, we can do as we did above and try out some values, checking the fit of each one to our data.

Because we now have three parameters to estimate, we will reduce the number of values we try out in order to keep things manageable.


```r
n_values = 16

models = expand.grid(asymptote=seq(0.4, 1, length.out=n_values),
                     location=seq(0.3, 0.9, length.out=n_values),
                     steepness=seq(0, 6, length.out=n_values))

models$RMSE = 0

for(m in 1:nrow(models)){
  predicted = f(mean_ratings$Image,
                asymptote=models$asymptote[m],
                location=models$location[m],
                steepness=models$steepness[m])
  models$RMSE[m] = rmse(predicted, mean_ratings$Rating)
}

head(models)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["asymptote"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["location"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["steepness"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["RMSE"],"name":[4],"type":["dbl"],"align":["right"]}],"data":[{"1":"0.40","2":"0.3","3":"0","4":"0.3109498","_rn_":"1"},{"1":"0.44","2":"0.3","3":"0","4":"0.3010549","_rn_":"2"},{"1":"0.48","2":"0.3","3":"0","4":"0.2921957","_rn_":"3"},{"1":"0.52","2":"0.3","3":"0","4":"0.2844689","_rn_":"4"},{"1":"0.56","2":"0.3","3":"0","4":"0.2779691","_rn_":"5"},{"1":"0.60","2":"0.3","3":"0","4":"0.2727840","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
print(models[models$RMSE == min(models$RMSE),])
```

```
##      asymptote location steepness      RMSE
## 1897      0.72     0.54       2.8 0.1339815
```

```r
fig_fit = ggplot(models, aes(x=location, y=asymptote, z=RMSE, fill=RMSE)) +
  geom_raster() +
  geom_contour(color='black') +
  scale_fill_gradient(low='red', high='yellow') +
  facet_wrap(~ steepness, labeller=label_both)

print(fig_fit)
```

![](Model-fitting_files/figure-html/unnamed-chunk-32-1.png)<!-- -->

Let's now try out `nls()` for these data. We use the default algorithm to begin with.


```r
starting_values = list(asymptote=0.7, location=0.5, steepness=3)

fit = nls(Rating ~ f(Image, asymptote, location, steepness), mean_ratings,
          starting_values,
          trace=TRUE)
```

```
## 10.59716 :  0.7 0.5 3.0
## 9.789216 :  0.7112306 0.5416603 2.7272050
## 9.776616 :  0.7216142 0.5469388 2.7423638
## 9.776598 :  0.7219079 0.5469838 2.7387921
## 9.776598 :  0.7219244 0.5469955 2.7387068
```

```r
print(fit)
```

```
## Nonlinear regression model
##   model: Rating ~ f(Image, asymptote, location, steepness)
##    data: mean_ratings
## asymptote  location steepness 
##    0.7219    0.5470    2.7387 
##  residual sum-of-squares: 9.777
## 
## Number of iterations to convergence: 4 
## Achieved convergence tolerance: 4.253e-06
```

## Controlling and checking the fit

In most cases, unless we have a fairly exotic function or data, the choice of starting point won't matter for the final result. The algorithm searches in the direction of better and better fits, so it will eventually find the best combination.

We can check this for our example.


```r
starting_values = list(asymptote=0.8, location=0.4, steepness=2)

fit = nls(Rating ~ f(Image, asymptote, location, steepness), mean_ratings,
          starting_values,
          trace=TRUE)
```

```
## 25.08039 :  0.8 0.4 2.0
## 10.39858 :  0.7266614 0.5778595 2.3349020
## 9.797548 :  0.7089020 0.5400335 2.7155596
## 9.776633 :  0.7215624 0.5469991 2.7430143
## 9.776598 :  0.7219094 0.5469843 2.7387469
## 9.776598 :  0.7219246 0.5469956 2.7387058
```

```r
print(fit)
```

```
## Nonlinear regression model
##   model: Rating ~ f(Image, asymptote, location, steepness)
##    data: mean_ratings
## asymptote  location steepness 
##    0.7219    0.5470    2.7387 
##  residual sum-of-squares: 9.777
## 
## Number of iterations to convergence: 5 
## Achieved convergence tolerance: 3.692e-06
```

But the starting values can sometimes have a big impact on whether the search succeeds at all. For some functions, certain combinations of parameter values may produce invalid or infinite predicted values, or predicted values that are so far off the observed values that it is unclear in which direction the fit might get better.


```r
starting_values = list(asymptote=0.9, location=0.5, steepness=0.1)

nls(Rating ~ f(Image, asymptote, location, steepness), mean_ratings,
    starting_values)
```

```
## Error in nls(Rating ~ f(Image, asymptote, location, steepness), mean_ratings, : singular gradient
```

If we find that the fitting process is returning errors of this kind, we may be able to help things along by setting boundaries on where the search algorithm may go during its search. `nls()` allows us to input lower and upper bounds for the search. If we want to leave any boundaries open, we can input `Inf` (or `-Inf` if we want to leave a lower boundary open). The default fitting algorithm in `nls()` does not allow for bounds, so we must also switch to another algorithm that does, the 'port' algorithm.


```r
starting_values = list(asymptote=0.9, location=0.5, steepness=0.1)

fit = nls(Rating ~ f(Image, asymptote, location, steepness), mean_ratings, starting_values,
          algorithm='port',
          lower=list(asymptote=0, location=0, steepness=0),
          upper=list(asymptote=1, location=1, steepness=Inf),
          trace=TRUE)
```

```
##   0:     19.856734: 0.900000 0.500000 0.100000
##   1:     15.191131: 0.572088  0.00000  1.46289
##   2:     8.7008452: 0.542966 0.328418  1.69649
##   3:     7.3770382: 0.597509 0.405771  1.74374
##   4:     7.1631684: 0.686970 0.619989  2.60313
##   5:     4.9304257: 0.710137 0.543391  2.54181
##   6:     4.8883594: 0.721956 0.547465  2.73713
##   7:     4.8882991: 0.721961 0.547018  2.73826
##   8:     4.8882990: 0.721933 0.546999  2.73867
##   9:     4.8882990: 0.721924 0.546995  2.73873
##  10:     4.8882989: 0.721925 0.546996  2.73870
##  11:     4.8882989: 0.721926 0.546996  2.73869
```

```r
print(fit)
```

```
## Nonlinear regression model
##   model: Rating ~ f(Image, asymptote, location, steepness)
##    data: mean_ratings
## asymptote  location steepness 
##    0.7219    0.5470    2.7387 
##  residual sum-of-squares: 9.777
## 
## Algorithm "port", convergence message: relative convergence (4)
```

Now we can successfully fit the model, and get the same estimates for the parameters as we did above, when we set no bounds but used a better guess for the starting values.

We are ready to check how well the estimated model fits the observed mean ratings. The models that `nls()` creates can also be input into the `predict()` function. So we can use `predict()` together with some made-up image values to generate the model's predicted values, which we can then add to our plot as a line.


```r
f_data = data.frame(Image=seq(0, 1, length.out=100))
f_data$Rating = predict(fit, f_data)

fig_crabsters = fig_crabsters +
  geom_line(data=f_data, lty='dashed')

print(fig_crabsters)
```

![](Model-fitting_files/figure-html/unnamed-chunk-37-1.png)<!-- -->

The function seems to approximate the trend in the averaged data reasonably well.

# Models as summaries

In the example we just looked at, with the crab and lobster images, the authors were not especially interested in comparing the chosen model to other models, nor with fitting it to the averaged ratings as we just did. Rather, they wanted to use the model as a way of summarizing different features of a given participant's behavior.

If participants have given lots of responses to lots of different kinds of stimuli, it can be difficult to compare participants using just their raw data. For example, here it would mean comparing mean ratings over several different images. An alternative is to fit a model to each participant's data, and use the estimated parameters of that model as a summary of the participant's behavior. This can be especially useful if we use a model whose parameters each characterize a specific aspect of behavior, as is the case with the model above. In particular the 'steepness' parameter here tells us to what extent a participant sharply changes their categorization of the image at some point along the crab-lobster continuum (as opposed to gradually changing it as the image gradually changes).

As we saw in the individual plots that we created above, the pattern of ratings varies quite a lot from participant to participant in this data set. We will now fit the model separately to each participant's data, and store the estimated parameters.

Let's see first how we would do this for just one participant. We can begin by selecting participant 1, whose ID we store in a variable.

The `subset()` function takes as its input a data frame, and gives us only those rows of the data frame where a certain condition holds. In this case, the condition is that the ID column should be equal to (`==`) 1.


```r
p = 1
p_ratings = subset(mean_ratings, ID==p)

p_ratings
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["Image"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["ID"],"name":[2],"type":["int"],"align":["right"]},{"label":["Rating"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"0.2","2":"1","3":"0.0703125","_rn_":"1"},{"1":"0.3","2":"1","3":"0.1796875","_rn_":"2"},{"1":"0.4","2":"1","3":"0.2421875","_rn_":"3"},{"1":"0.5","2":"1","3":"0.2812500","_rn_":"4"},{"1":"0.6","2":"1","3":"0.4531250","_rn_":"5"},{"1":"0.7","2":"1","3":"0.5546875","_rn_":"6"},{"1":"0.8","2":"1","3":"0.7031250","_rn_":"7"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We can then re-fit the model to just this subset of data. The `update()` function re-fits a model. We have to tell `update()` what aspect of the original fitting procedure we want to change. In this case it is the data.

While we are at it, we can change one other aspect of the fitting procedure in order to make it a little more robust to error. We can change the starting values. Now that we have estimated the best fitting values for the averaged ratings, we can use these as the starting point to search for each participant's best fitting values.

(We can extract fitted parameter values with `coefficients()`.)


```r
p_fit = update(fit, data=p_ratings, start=coefficients(fit))
```

```
##   0:  0.0059481202: 0.721926 0.546996  2.73869
##   1:  0.0029718126: 0.695829 0.543110  2.25117
##   2:  0.0029386310: 0.702513 0.546806  2.26852
##   3:  0.0029385234: 0.702843 0.546922  2.26478
##   4:  0.0029385229: 0.702868 0.546943  2.26479
##   5:  0.0029385229: 0.702870 0.546945  2.26477
##   6:  0.0029385229: 0.702870 0.546945  2.26477
```

```r
print(p_fit)
```

```
## Nonlinear regression model
##   model: Rating ~ f(Image, asymptote, location, steepness)
##    data: p_ratings
## asymptote  location steepness 
##    0.7029    0.5469    2.2648 
##  residual sum-of-squares: 0.005877
## 
## Algorithm "port", convergence message: relative convergence (4)
```

We can see that this participant has best fitting values for the three parameters that are slightly different from those that best fit the overall average ratings.

Now we want to do this for every participant in turn. We will need somewhere to store the results. For this, we create a new data frame with a column for participant ID (which we get by asking what the unique values of ID are in the original data frame), and columns for the three model parameters, which we initially fill with the value 0. In addition, we create a column for the RMSE of each participant's fitted model. This will allow us to identify any participants for whom the model fits worse than for others.


```r
p_models = data.frame(ID=unique(mean_ratings$ID),
                      asymptote=0,
                      location=0,
                      steepness=0,
                      RMSE=0)
```

Now we do as above, but with a loop in which the value of `p` changes on each iteration. Storing the estimated parameters is slightly tricky. For this, we need to pick out the right row of our new data frame, and the right columns, and replace them with the estimated values from the participant's fitted model.

(We also turn off the `trace` option, so we aren't flooded with output on all the attempted parameters values for every participant.)


```r
for(p in unique(p_models$ID)){
  
  p_ratings = subset(mean_ratings, ID==p)
  p_fit = update(fit, data=p_ratings, start=coefficients(fit), trace=FALSE)
  
  p_row = p_models$ID==p
  
  p_models[p_row,c('asymptote','location','steepness')] = coefficients(p_fit)
  p_models$RMSE[p_row] = rmse(predict(p_fit), p_ratings$Rating)
  
}

p_models
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["ID"],"name":[1],"type":["int"],"align":["right"]},{"label":["asymptote"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["location"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["steepness"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["RMSE"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"0.7028702","3":"0.5469450","4":"2.264766","5":"0.028975472"},{"1":"2","2":"0.7307807","3":"0.4670679","4":"2.759846","5":"0.034216726"},{"1":"3","2":"0.7432786","3":"0.4970058","4":"3.156417","5":"0.043574235"},{"1":"4","2":"0.6554821","3":"0.3530990","4":"1.685009","5":"0.020130810"},{"1":"5","2":"0.9036399","3":"0.5322073","4":"3.450335","5":"0.020750818"},{"1":"6","2":"0.6965973","3":"0.4761125","4":"1.301576","5":"0.004805142"},{"1":"7","2":"0.6797671","3":"0.4618194","4":"3.229399","5":"0.018501167"},{"1":"8","2":"0.7020010","3":"0.5327525","4":"5.464928","5":"0.023718712"},{"1":"9","2":"0.9000201","3":"0.6886698","4":"8.719277","5":"0.004934345"},{"1":"11","2":"0.7524964","3":"0.3553678","4":"2.037639","5":"0.028262657"},{"1":"12","2":"0.6617596","3":"0.4701362","4":"3.985183","5":"0.014173249"},{"1":"13","2":"0.5381050","3":"0.3650892","4":"1.467097","5":"0.041305831"},{"1":"14","2":"0.7228796","3":"0.5726644","4":"4.008811","5":"0.016423566"},{"1":"15","2":"0.6696860","3":"0.4771504","4":"3.783374","5":"0.042814687"},{"1":"16","2":"0.6530602","3":"0.4248393","4":"1.388060","5":"0.023011782"},{"1":"17","2":"0.8497901","3":"0.6520510","4":"5.241905","5":"0.013309420"},{"1":"18","2":"0.9779452","3":"0.5417326","4":"2.755982","5":"0.050544938"},{"1":"19","2":"0.4843360","3":"0.5428767","4":"3.421359","5":"0.021613156"},{"1":"20","2":"0.6729675","3":"0.3376597","4":"2.154094","5":"0.033804620"},{"1":"21","2":"0.7186730","3":"0.5853479","4":"8.520744","5":"0.017180421"},{"1":"22","2":"0.4211553","3":"0.4716956","4":"2.478668","5":"0.013203529"},{"1":"23","2":"0.6768325","3":"0.4453798","4":"2.054568","5":"0.048521279"},{"1":"24","2":"0.4725756","3":"0.6067573","4":"2.404209","5":"0.010422459"},{"1":"25","2":"0.7477601","3":"0.6382994","4":"4.675698","5":"0.033513492"},{"1":"26","2":"0.6885932","3":"0.6577148","4":"3.835019","5":"0.013019691"},{"1":"27","2":"0.4291185","3":"0.6258965","4":"3.899172","5":"0.024265087"},{"1":"28","2":"0.8880732","3":"0.5262931","4":"1.614104","5":"0.010632110"},{"1":"30","2":"0.7289583","3":"0.3197278","4":"1.386698","5":"0.038523235"},{"1":"31","2":"0.6288328","3":"0.5730816","4":"3.727977","5":"0.030762132"},{"1":"32","2":"0.5009815","3":"0.5419611","4":"6.798548","5":"0.023912329"},{"1":"33","2":"0.8101450","3":"0.5698889","4":"5.973611","5":"0.029364824"},{"1":"34","2":"0.7774449","3":"0.6577487","4":"4.289448","5":"0.035180122"},{"1":"35","2":"0.6899133","3":"0.5776603","4":"4.443510","5":"0.030632082"},{"1":"36","2":"0.6854061","3":"0.5072827","4":"1.985695","5":"0.015341197"},{"1":"37","2":"0.8378061","3":"0.5677225","4":"3.701485","5":"0.017038681"},{"1":"38","2":"0.8208986","3":"0.5614093","4":"2.767771","5":"0.022198525"},{"1":"39","2":"0.5558892","3":"0.3981828","4":"2.230352","5":"0.009053593"},{"1":"40","2":"0.8710366","3":"0.5807586","4":"7.668538","5":"0.039949257"},{"1":"41","2":"0.8984959","3":"0.6260408","4":"6.761758","5":"0.021882479"},{"1":"42","2":"0.7349589","3":"0.4971963","4":"1.579228","5":"0.013070407"},{"1":"43","2":"0.6834595","3":"0.5159050","4":"2.297120","5":"0.037717980"},{"1":"44","2":"0.6820355","3":"0.4999870","4":"1.874071","5":"0.034830357"},{"1":"45","2":"0.6182254","3":"0.4437388","4":"1.733838","5":"0.034481765"},{"1":"46","2":"0.7752159","3":"0.5259314","4":"4.977350","5":"0.035162179"},{"1":"47","2":"0.7019031","3":"0.6364362","4":"5.439218","5":"0.005675772"},{"1":"48","2":"0.6445394","3":"0.6205943","4":"1.666383","5":"0.004843093"},{"1":"49","2":"0.9902430","3":"0.5475044","4":"5.543805","5":"0.022297442"},{"1":"50","2":"0.7928636","3":"0.5955640","4":"3.100476","5":"0.049249103"},{"1":"51","2":"0.8836546","3":"0.5937370","4":"2.510289","5":"0.014075182"},{"1":"52","2":"0.7161804","3":"0.5361845","4":"2.280899","5":"0.038388546"},{"1":"53","2":"0.8655629","3":"0.4493640","4":"2.811546","5":"0.059829039"},{"1":"54","2":"0.7406182","3":"0.4198779","4":"4.922205","5":"0.035819813"},{"1":"55","2":"0.2426468","3":"0.5641504","4":"7.065649","5":"0.013763484"},{"1":"56","2":"0.6841254","3":"0.6236839","4":"3.783187","5":"0.032473555"},{"1":"57","2":"0.5579314","3":"0.6262159","4":"3.799692","5":"0.014289254"},{"1":"58","2":"0.8091704","3":"0.5608710","4":"2.997997","5":"0.011283170"},{"1":"59","2":"0.6332674","3":"0.5114215","4":"3.165154","5":"0.032863531"},{"1":"60","2":"0.9108503","3":"0.4973483","4":"2.702998","5":"0.077961956"},{"1":"61","2":"0.6975974","3":"0.4922832","4":"2.938243","5":"0.011442921"},{"1":"62","2":"0.7717868","3":"0.4147160","4":"1.695181","5":"0.049472433"},{"1":"63","2":"0.9446430","3":"0.5555694","4":"2.546077","5":"0.025859989"},{"1":"64","2":"0.5191412","3":"0.5078064","4":"2.326600","5":"0.034786501"},{"1":"65","2":"0.4396034","3":"0.5759408","4":"2.394356","5":"0.015891086"},{"1":"66","2":"0.6433493","3":"0.4940446","4":"4.622317","5":"0.052166385"},{"1":"67","2":"0.8930762","3":"0.6025423","4":"2.176786","5":"0.042742027"},{"1":"68","2":"0.8817591","3":"0.4965286","4":"2.997428","5":"0.074377077"},{"1":"69","2":"0.7366419","3":"0.4146221","4":"2.863492","5":"0.057028616"},{"1":"70","2":"0.8714191","3":"0.6131276","4":"4.420528","5":"0.047260970"},{"1":"71","2":"0.6508605","3":"0.5998475","4":"3.812461","5":"0.041244748"},{"1":"72","2":"0.6495570","3":"0.5707975","4":"3.431537","5":"0.030996517"},{"1":"73","2":"0.9611361","3":"0.6141720","4":"4.537568","5":"0.025658173"},{"1":"74","2":"0.8059934","3":"0.4736722","4":"2.431841","5":"0.050034863"},{"1":"75","2":"0.3203045","3":"0.5810609","4":"10.929912","5":"0.012912218"},{"1":"76","2":"0.6475286","3":"0.6461523","4":"3.955132","5":"0.028467291"},{"1":"77","2":"0.8463044","3":"0.4574201","4":"2.933093","5":"0.080824011"},{"1":"78","2":"0.8248680","3":"0.4462928","4":"3.512456","5":"0.056144239"},{"1":"79","2":"0.8070959","3":"0.4349994","4":"3.557441","5":"0.078068192"},{"1":"80","2":"0.8317118","3":"0.4584422","4":"2.946882","5":"0.080911855"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

What spread of values did we get for the fitted parameters? We can look at histograms of each one.

To avoid repeating the plotting commands three times, we can create one plot, and then add to this the `aes()` function to tell the plot what variable to show along the *x* axis.


```r
parameter_hist = ggplot(p_models) +
  geom_histogram(bins=20) +
  labs(caption='Data: Reindl et al. (2018)')

print(parameter_hist + aes(x=asymptote))
```

![](Model-fitting_files/figure-html/unnamed-chunk-42-1.png)<!-- -->

```r
print(parameter_hist + aes(x=location))
```

![](Model-fitting_files/figure-html/unnamed-chunk-42-2.png)<!-- -->

```r
print(parameter_hist + aes(x=steepness))
```

![](Model-fitting_files/figure-html/unnamed-chunk-42-3.png)<!-- -->

We might also sometimes be interested in whether the fitted values are related to one another. This is not particularly of interest in this example, but we will check anyway for the sake of completeness.

The 'GGally' package gives us some additional plotting functions for specific applications, such as `ggpairs()`, which plots a matrix of scatterplots, each of which plots the relationship between two of a set of variables within a data set. We tell `ggpairs()` which columns from the data to include in the plot.

We see that the location and steepness parameters are positively related to each other, for example.


```r
ggpairs(p_models, columns=c('asymptote','location','steepness'))
```

![](Model-fitting_files/figure-html/unnamed-chunk-43-1.png)<!-- -->

We have now obtained and explored fitted parameter values for each participant. These summarize a participant's performance in a way that the mean ratings per image could not easily capture.

Because the fitting process can be computationally somewhat time-consuming for larger data sets or complex models, it is often a good idea to save the data frame of fitted parameter values to a new file, in case we want to pick up analysis of it later.

Saving to csv format can be done with `write.csv()`.


```r
write.csv(p_models, 'crabsters_parameters.csv')
```

With this accomplished, we can go on to compare the fitted values across different experimental conditions, or across types of participants, as was one of the objectives of the original study. This would put us back in familiar statistical territory, comparing values across groups and so on. For this reason, we won't go on to do it here. Instead, we will look at one more important aspect of model fitting.

## Checking model fit

Because we applied boundaries to the search for best fitting values, we should first check that no participants actually got fitted values on one of the boundaries. This would indicate either that the boundaries were not appropriate and have artificially constrained the possible estimates we can get, or that something just went wrong with the fitting algorithm.

(The `range()` function gives us the minimum and maximum.)

Here we see that there are no fitted values on any of the boundaries.


```r
print(range(p_models$asymptote))
```

```
## [1] 0.2426468 0.9902430
```

```r
print(range(p_models$location))
```

```
## [1] 0.3197278 0.6886698
```

```r
print(range(p_models$steepness))
```

```
## [1]  1.301576 10.929912
```

We should also check whether the model did not drastically fail to fit any of the participants' data. For this, we can look first at the general spread of RMSE values using the same histogram from above.


```r
print(parameter_hist + aes(x=RMSE))
```

![](Model-fitting_files/figure-html/unnamed-chunk-46-1.png)<!-- -->

The worst RMSE values are at around 0.08. Given that the values of RMSE are on the scale of the ratings, which range from 0 to 1, this is not a really huge error.

We can locate the worst fits by ranking the RMSE values as bars, and displaying the participant ID numbers on the bars.

This requires some more plotting tricks:

* the `reorder()` function sorts the display order for one variable, based on the values of a second varable (here we sort the ID numbers by RMSE)
* the `label` plotting dimension in `aes()` assigns the specific values of a variable as labels on the plot
* `geom_text()` then applies the `label` from the main plot definition as plain text
* we remove the labelling of the *x* axis entirely, since we already have the IDs as labels


```r
rmse_ranks = ggplot(p_models, aes(y=RMSE, x=reorder(ID,RMSE), label=ID)) +
  geom_col() +
  geom_text(vjust=0) +
  scale_x_discrete(breaks=NULL, name=NULL)

print(rmse_ranks)
```

![](Model-fitting_files/figure-html/unnamed-chunk-47-1.png)<!-- -->

We can pick out the ID numbers of the worst fits by applying a subset condition.


```r
bad_fits = p_models$ID[p_models$RMSE>0.07]

print(bad_fits)
```

```
## [1] 60 68 77 79 80
```

If you look back at our individual participant plots, you will see that unlike most of the others, these participants have a combination of two features that the model does not account for.

First, they have a steady and fairly steep increase in ratings as the image changes. So they don't seem to classify the approximately crab-like images together with the more clearly crab-like ones. And nor do their ratings reach a maximum of lobsterness.

Additionally, these participants have a small 'plateau' in their ratings in the middle of the image morphing scale, as if they were perhaps classifying the most ambiguous images as belonging to a third, intermediate category, 'crabster'. Such a break in the transition from crab to lobster is not a feature of the model.


```r
fig_crabsters_p = fig_crabsters_p +
  aes(color=ID%in%bad_fits) +
  scale_color_manual(values=c('black','red')) +
  theme(legend.position='none')

print(fig_crabsters_p)
```

![](Model-fitting_files/figure-html/unnamed-chunk-49-1.png)<!-- -->
