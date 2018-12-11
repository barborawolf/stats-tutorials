---
title: "Variants of linear models"
author: "LT"
output:
  html_document:
    toc: TRUE
    toc_float: TRUE
    df_print: "paged"
    keep_md: TRUE
---

# Linear models

Let's briefly remind ourselves how to get a linear regression model in R. We can use the birth weights data set. We also produce an accompanying plot.


```r
library(ggplot2)

bw = read.csv('birth_weights.csv')

fig1 = ggplot(bw, aes(y=Birth_weight, x=Weight)) +
  geom_point() +
  labs(y='Birth weight (kg)', x='Weight (kg)', caption='Data: Baystate Medical Center, 1986')

print(fig1 + geom_smooth(method=lm))
```

![](Linear-models_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

We can ask for a linear model using `lm()`, and inputting a formula of the type 'outcome ~ predictor(s)'. We will get a model that predicts the value of the outcome variable as the sum of an intercept plus some multiple of each predictor.


```r
model = lm(Birth_weight ~ Weight, bw)
summary(model)
```

```
## 
## Call:
## lm(formula = Birth_weight ~ Weight, data = bw)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -2.19212 -0.49797 -0.00384  0.50832  2.07560 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 2.369624   0.228493  10.371   <2e-16 ***
## Weight      0.009765   0.003778   2.585   0.0105 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7184 on 187 degrees of freedom
## Multiple R-squared:  0.0345,	Adjusted R-squared:  0.02933 
## F-statistic: 6.681 on 1 and 187 DF,  p-value: 0.0105
```

We will now look at a couple of variants of this basic linear model. These adapt the basic structure of the linear model in order to model non-linear relationships.

## Polynomials

Let's look at an example of a possible non-linear relationship between two variables. For this, we load some new data.

These data come from [diving penguins](http://jeb.biologists.org/content/211/8/1169). For each dive, we have the duration of the dive, the depth of the dive, and the penguin's heart rate.


```r
peng = read.csv('penguins.csv')

peng
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["HeartRate"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["Depth"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Duration"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"88.8","2":"5.00","3":"1.050000"},{"1":"103.4","2":"9.00","3":"1.183333"},{"1":"97.4","2":"22.00","3":"1.916667"},{"1":"85.3","2":"25.50","3":"3.466667"},{"1":"60.6","2":"30.50","3":"7.083333"},{"1":"77.6","2":"32.50","3":"4.766667"},{"1":"44.3","2":"38.00","3":"9.133333"},{"1":"32.8","2":"32.00","3":"11.000000"},{"1":"94.2","2":"6.00","3":"1.316667"},{"1":"99.8","2":"10.50","3":"1.483333"},{"1":"104.5","2":"6.00","3":"1.166667"},{"1":"78.0","2":"19.50","3":"2.716667"},{"1":"54.2","2":"27.50","3":"7.250000"},{"1":"79.0","2":"33.50","3":"4.783333"},{"1":"42.9","2":"40.50","3":"11.866667"},{"1":"134.0","2":"12.00","3":"1.083333"},{"1":"54.1","2":"43.00","3":"8.016667"},{"1":"31.8","2":"45.50","3":"11.283333"},{"1":"49.4","2":"35.00","3":"8.133333"},{"1":"57.1","2":"33.50","3":"6.083333"},{"1":"50.2","2":"147.00","3":"9.016667"},{"1":"97.3","2":"66.50","3":"2.316667"},{"1":"32.3","2":"116.00","3":"10.866667"},{"1":"42.1","2":"40.00","3":"6.050000"},{"1":"40.2","2":"46.50","3":"9.833333"},{"1":"34.6","2":"37.00","3":"8.766667"},{"1":"81.0","2":"11.00","3":"2.000000"},{"1":"44.5","2":"20.50","3":"6.366667"},{"1":"106.3","2":"30.50","3":"2.066667"},{"1":"36.3","2":"45.00","3":"9.933333"},{"1":"87.7","2":"26.00","3":"2.116667"},{"1":"24.1","2":"66.00","3":"18.150000"},{"1":"47.8","2":"170.00","3":"10.033333"},{"1":"44.9","2":"160.00","3":"9.983333"},{"1":"45.5","2":"140.00","3":"10.500000"},{"1":"47.7","2":"40.00","3":"5.283333"},{"1":"49.1","2":"25.00","3":"5.133333"},{"1":"43.6","2":"49.00","3":"7.300000"},{"1":"68.1","2":"100.00","3":"3.350000"},{"1":"51.7","2":"52.00","3":"5.933333"},{"1":"91.1","2":"39.00","3":"2.833333"},{"1":"34.0","2":"47.00","3":"9.033333"},{"1":"52.0","2":"39.00","3":"4.733333"},{"1":"103.8","2":"26.00","3":"1.916667"},{"1":"34.8","2":"42.00","3":"7.016667"},{"1":"36.9","2":"90.00","3":"9.216667"},{"1":"48.6","2":"160.00","3":"7.466667"},{"1":"43.8","2":"160.00","3":"8.000000"},{"1":"52.5","2":"130.00","3":"6.933333"},{"1":"67.2","2":"32.32","3":"3.733333"},{"1":"48.2","2":"29.78","3":"5.750000"},{"1":"52.3","2":"66.94","3":"8.100000"},{"1":"40.1","2":"34.37","3":"10.133317"},{"1":"83.6","2":"35.11","3":"2.583317"},{"1":"55.4","2":"49.22","3":"6.249967"},{"1":"47.1","2":"36.42","3":"8.633300"},{"1":"48.3","2":"51.41","3":"10.849950"},{"1":"104.5","2":"19.24","3":"1.100000"},{"1":"54.9","2":"37.40","3":"8.833283"},{"1":"41.0","2":"30.56","3":"11.749933"},{"1":"71.5","2":"35.94","3":"4.849983"},{"1":"74.7","2":"33.35","3":"3.683317"},{"1":"37.7","2":"46.48","3":"14.483250"},{"1":"67.8","2":"31.69","3":"4.733300"},{"1":"41.1","2":"56.10","3":"12.616600"},{"1":"29.6","2":"53.56","3":"15.449917"},{"1":"70.5","2":"12.30","3":"1.050000"},{"1":"47.1","2":"25.88","3":"5.366650"},{"1":"34.1","2":"36.33","3":"8.966617"},{"1":"43.3","2":"34.71","3":"8.499950"},{"1":"35.8","2":"66.45","3":"9.799933"},{"1":"32.7","2":"43.80","3":"10.933283"},{"1":"40.3","2":"50.49","3":"10.516600"},{"1":"36.2","2":"44.43","3":"10.483267"},{"1":"84.4","2":"31.00","3":"2.250000"},{"1":"31.3","2":"37.99","3":"11.816600"},{"1":"31.3","2":"57.32","3":"12.249933"},{"1":"78.5","2":"29.88","3":"1.464367"},{"1":"31.5","2":"44.63","3":"9.208700"},{"1":"57.5","2":"20.41","3":"2.416650"},{"1":"67.8","2":"22.41","3":"1.933317"},{"1":"48.5","2":"35.40","3":"3.299983"},{"1":"33.7","2":"74.51","3":"10.799933"},{"1":"27.5","2":"57.76","3":"13.516600"},{"1":"29.9","2":"53.90","3":"11.949933"},{"1":"39.2","2":"75.97","3":"9.499933"},{"1":"32.1","2":"55.03","3":"10.833267"},{"1":"30.3","2":"45.26","3":"14.149933"},{"1":"81.3","2":"47.90","3":"1.966650"},{"1":"113.6","2":"10.89","3":"1.100000"},{"1":"80.9","2":"16.94","3":"1.433317"},{"1":"76.6","2":"22.17","3":"2.533317"},{"1":"39.5","2":"47.12","3":"7.116633"},{"1":"38.8","2":"63.82","3":"8.499967"},{"1":"22.8","2":"53.61","3":"12.583267"},{"1":"34.3","2":"59.91","3":"10.683267"},{"1":"121.7","2":"22.61","3":"1.149983"},{"1":"35.5","2":"43.99","3":"9.116617"},{"1":"36.3","2":"49.51","3":"9.816600"},{"1":"25.5","2":"46.19","3":"11.983267"},{"1":"33.0","2":"40.87","3":"8.999950"},{"1":"111.2","2":"31.64","3":"1.816667"},{"1":"30.6","2":"52.49","3":"11.149933"},{"1":"119.5","2":"38.69","3":"1.849983"},{"1":"28.1","2":"56.15","3":"14.683250"},{"1":"73.3","2":"22.75","3":"2.183333"},{"1":"39.0","2":"34.37","3":"5.816633"},{"1":"28.5","2":"45.65","3":"9.899950"},{"1":"24.2","2":"46.43","3":"10.366600"},{"1":"23.5","2":"54.49","3":"12.399933"},{"1":"25.3","2":"67.38","3":"11.566600"},{"1":"46.6","2":"37.00","3":"8.333333"},{"1":"77.1","2":"135.00","3":"7.066667"},{"1":"77.5","2":"225.00","3":"7.466667"},{"1":"71.6","2":"225.00","3":"8.616667"},{"1":"101.8","2":"28.00","3":"2.866667"},{"1":"46.8","2":"145.00","3":"11.816667"},{"1":"50.6","2":"175.00","3":"10.783333"},{"1":"127.8","2":"8.60","3":"1.533333"},{"1":"42.1","2":"165.00","3":"13.533333"},{"1":"48.4","2":"170.00","3":"11.533333"},{"1":"50.8","2":"37.00","3":"8.216667"},{"1":"49.6","2":"160.00","3":"11.300000"},{"1":"56.4","2":"180.00","3":"10.283333"},{"1":"55.2","2":"170.00","3":"10.366667"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Let's investigate the relationship between dive duration and heart rate. A linear model perhaps fails to capture an important feature of the data.


```r
peng_plot = ggplot(peng, aes(y=HeartRate, x=Duration)) +
  geom_point() +
  labs(y='Heart rate (bpm)', x='Dive duration (mins)', caption='Data: Meir & Ponganis, 2009')

print(peng_plot + geom_smooth(method=lm))
```

![](Linear-models_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

Sometimes we can see this even more clearly by plotting the residuals from a linear model. Since the residuals contain the variance in the outcome that remains after subtracting the model's predictions, any pattern left over in the residuals is one that the model did not account for.

(Since we have changed the data set by adding a column for the residuals, we need to feed this altered data set into our plot with `%+%` in order for the change to be taken into account.)


```r
model1 = lm(HeartRate ~ Duration, peng)
peng$Residual = residuals(model1)

residual_plot = peng_plot +
  aes(y=Residual) +
  geom_hline(yintercept=0, lty='dashed')

print(residual_plot %+% peng)
```

![](Linear-models_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

There appears to be a 'bend' in the relationship. We can easily adapt the linear model to a 'bent' relationship by including as an additional predictor some curved function of our basic predictor. For example, the square of dive duration.

One way of adapting the R formula for the model so as to include the square of a predictor is to just write the mathematical expression for the square `^2`. Unfortunately, the arithmetic operators `+ - * / ^` and so on have particular meanings within formulae. For example, `+` adds in a new predictor, and `*` indicates an interaction. So in order to indicate to R that we want instead the literal arithmetic meaning of the `^` operator and not some special formula meaning, we have to wrap the expression in `I()`, a function whose purpose is to do exactly this: indicate a literal arithmetic meaning within a formula.


```r
model2 = lm(HeartRate ~ Duration + I(Duration^2), peng)
summary(model2)
```

```
## 
## Call:
## lm(formula = HeartRate ~ Duration + I(Duration^2), data = peng)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -30.115  -8.289  -1.567   8.016  34.187 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   111.60991    3.32024  33.615  < 2e-16 ***
## Duration      -11.32555    0.99734 -11.356  < 2e-16 ***
## I(Duration^2)   0.40212    0.06585   6.107 1.25e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.4 on 122 degrees of freedom
## Multiple R-squared:  0.782,	Adjusted R-squared:  0.7784 
## F-statistic: 218.8 on 2 and 122 DF,  p-value: < 2.2e-16
```

A function that is the sum of various powers of the predictor variable is known as a polynomial function. There are many possible polynomial functions. One basic way of classifying them is in terms of their 'degree', the highest power of the predictor included in the function. The function we used in our model above has degree 2, because it includes duration¹ and duration². Each additional power gives the model the flexibility to 'bend' in one more place. So whereas the linear model may not bend at all, the degree-2 polynomial model we used above can have one bend.

Sometimes polynomial functions are given shorthand names according to their degree:

0. constant
1. linear
2. quadratic
3. cubic
4. quartic
5. quintic
6. sextic
7. septic

If we want polynomial functions of higher degree, it will become tedious to put all the powers of the predictor into the formula. The R function `poly()` generates a polynomial, allowing us to specify the desired degree in a single input.


```r
model2 = lm(HeartRate ~ poly(Duration,degree=2), peng)
summary(model2)
```

```
## 
## Call:
## lm(formula = HeartRate ~ poly(Duration, degree = 2), data = peng)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -30.115  -8.289  -1.567   8.016  34.187 
## 
## Coefficients:
##                             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                   56.924      1.109  51.320  < 2e-16 ***
## poly(Duration, degree = 2)1 -248.097     12.401 -20.006  < 2e-16 ***
## poly(Duration, degree = 2)2   75.734     12.401   6.107 1.25e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.4 on 122 degrees of freedom
## Multiple R-squared:  0.782,	Adjusted R-squared:  0.7784 
## F-statistic: 218.8 on 2 and 122 DF,  p-value: < 2.2e-16
```

```r
model3 = lm(HeartRate ~ poly(Duration,degree=3), peng)
summary(model3)
```

```
## 
## Call:
## lm(formula = HeartRate ~ poly(Duration, degree = 3), data = peng)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -33.458  -7.882  -1.752   7.109  30.710 
## 
## Coefficients:
##                             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                   56.924      1.088  52.343  < 2e-16 ***
## poly(Duration, degree = 3)1 -248.097     12.159 -20.405  < 2e-16 ***
## poly(Duration, degree = 3)2   75.734     12.159   6.229 7.07e-09 ***
## poly(Duration, degree = 3)3  -29.571     12.159  -2.432   0.0165 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.16 on 121 degrees of freedom
## Multiple R-squared:  0.7921,	Adjusted R-squared:  0.787 
## F-statistic: 153.7 on 3 and 121 DF,  p-value: < 2.2e-16
```

There is another advantage to using the `poly()` function here. Take a look at what happens if we go up to a quintic model (polynomial of degree 5) for these data, using the literal powers of duration as our predictors.


```r
model5b = lm(HeartRate ~ Duration+I(Duration^2)+I(Duration^3)+I(Duration^4)+I(Duration^5), peng)

summary(model5b)
```

```
## 
## Call:
## lm(formula = HeartRate ~ Duration + I(Duration^2) + I(Duration^3) + 
##     I(Duration^4) + I(Duration^5), data = peng)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -35.612  -7.347  -0.847   7.236  30.307 
## 
## Coefficients:
##                 Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    1.294e+02  1.277e+01  10.133   <2e-16 ***
## Duration      -2.552e+01  1.269e+01  -2.011   0.0466 *  
## I(Duration^2)  3.486e+00  3.974e+00   0.877   0.3821    
## I(Duration^3) -2.794e-01  5.328e-01  -0.524   0.6009    
## I(Duration^4)  1.162e-02  3.184e-02   0.365   0.7157    
## I(Duration^5) -1.932e-04  6.941e-04  -0.278   0.7813    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.21 on 119 degrees of freedom
## Multiple R-squared:  0.7938,	Adjusted R-squared:  0.7851 
## F-statistic:  91.6 on 5 and 119 DF,  p-value: < 2.2e-16
```

We can see from the original plot of the heart rate data above that there is a fairly clear single bend in the overall trend. And this was reflected in the low *p*-value for the `I(Duration^2)` coefficient in the hypothesis tests for the model. However, now that we have included some higher powers of duration, the apparent usefulness of the square of duration has drastically diminished; its *p*-value is now quite large.

This is an old problem in a slightly new guise. Recall that in multiple regression the hypothesis tests for the coefficients can be thought of as testing the amount of variance in the outcome that each predictor accounts for, after having already considered all the other predictors. So if predictors are correlated with one another, and consequently explain some of the same variance in the outcome, they will obscure each other's effects in the individual hypothesis tests. The phenomenon of correlated predictors is often termed 'collinearity' (or 'multicollinearity' where more than just two predictors are involved).

Powers of a number tend to be correlated with one another over certain regions of the number line. Take *x*¹ and *x*² for example. In the interval 0 to 2, they are somewhat collinear.


```r
polyplot = ggplot(data.frame(x=seq(from=0, to=2, by=0.01))) +
  geom_line(aes(x=x, y=x)) +
  geom_line(aes(x=x, y=x^2), lty='dashed') +
  labs(y='y') +
  annotate('text', x=2, y=c(2,4), label=c('y = x¹','y = x²'), hjust=1, vjust=0)

print(polyplot)
```

![](Linear-models_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

So in a model that includes several powers of the predictor, these separate powers may obscure one another's effects in the individual hypothesis tests because they are correlated over the region of the scale that the data occupy.

If this property is undesirable for our use of the model, we can avoid it by transforming the data so that they occupy a region of the scale where the powers of the predictor are not correlated with one another. For example, *x*¹ and *x*² are not correlated over the interval -2 to 2.


```r
print(polyplot %+% data.frame(x=seq(from=-2, to=2, by=0.01)))
```

![](Linear-models_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

In fact, this is what the `poly()` function already does behind the scenes. It generates so-called 'orthogonal polynomials' of our predictor. This means that we will not have a collinearity problem if we want to test the explanatory power of individual powers of our predictor.

For example, for the quintic model generated using `poly()`, we see that the *p*-value for the quadratic component remains low despite the inclusion of the higher powers, according somewhat better with the visual intuition that the quadratic model is a good description of the data.


```r
model5 = lm(HeartRate ~ poly(Duration, degree=5), peng)
summary(model5)
```

```
## 
## Call:
## lm(formula = HeartRate ~ poly(Duration, degree = 5), data = peng)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -35.612  -7.347  -0.847   7.236  30.307 
## 
## Coefficients:
##                             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                   56.924      1.092  52.115  < 2e-16 ***
## poly(Duration, degree = 5)1 -248.097     12.212 -20.316  < 2e-16 ***
## poly(Duration, degree = 5)2   75.734     12.212   6.202 8.37e-09 ***
## poly(Duration, degree = 5)3  -29.571     12.212  -2.421    0.017 *  
## poly(Duration, degree = 5)4   11.374     12.212   0.931    0.354    
## poly(Duration, degree = 5)5   -3.399     12.212  -0.278    0.781    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.21 on 119 degrees of freedom
## Multiple R-squared:  0.7938,	Adjusted R-squared:  0.7851 
## F-statistic:  91.6 on 5 and 119 DF,  p-value: < 2.2e-16
```

However, we pay a certain price for this property. Because the orthogonal polynomials are based on a transformation of our predictor, the units of the coefficients in the model are no longer meaningful. They are on the transformed scale, not the original one.

It is also important to note that this orthogonality property is only important where we wish to consider the individual powers within a polynomial model. If we simply wish to assess the overall model, it will not matter whether the predictors are orthogonal polynomials or literal powers. The sum of the components of the model will be the same, and the model will make the same predictions and have the same overall fit to the data.

We can see this if we compare the `summary()` outputs of the two different quintic models we created above. They have the same *R*², for example, and the *p*-value for the overall *F*-test is also the same.

We should visualise models when we can. Polynomial models can be easily visualised as curves overlaid on the scatterplot. To add more than one model to the same plot, we can add multiple `geom_smooth()`s, each time with a different formula. The formulae are the same as those we use to create the linear models with `lm()`, only with the plot dimensions `y` and `x` substituted for our outcome and predictor variables.

When we have more than one smooth line on a plot, it is often clearer if we omit the standard error region around the line. Otherwise we will have too much shading obscuring the plot. We can also colour the lines differently.

(The default formula is the simple linear model, so the formula can be omitted for the linear one.)


```r
peng_plot_models = peng_plot +
  geom_smooth(method=lm, se=FALSE) +
  geom_smooth(method=lm, formula=y~poly(x,degree=2), se=FALSE, color='red') +
  geom_smooth(method=lm, formula=y~poly(x,degree=3), se=FALSE, color='green')

print(peng_plot_models)
```

![](Linear-models_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

As well as polynomials, other functions of the predictors can be included in linear models. For example, a logarithmic function seems to describe these data particularly well.


```r
print(peng_plot + geom_smooth(method=lm, formula=y~log(x)))
```

![](Linear-models_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

## Logistic regression

Let's now look at a rather different way of adapting the basic linear model to a non-linear relationship.

In the birth weights data, the `Visits` variable records how many times a doctor visited the mother while she was in hospital.


```r
table(bw$Visits)
```

```
## 
##   0   1   2   3   4   6 
## 100  47  30   7   4   1
```

We can see that only a very few mothers received multiple visits. The information available to us about larger numbers of visits is therefore rather sparse, so we will treat this variable as dichotomous, and compare the large number of mothers who did not receive any visits with those who received at least one visit.

To create a new dichotomous variable recording whether or not a mother received any visits, we will first create a new column in the data set containing only zeros, and then we will enter '1' wherever a mother had at least one visit.


```r
bw$Any_visits = 0
bw$Any_visits[bw$Visits>0] = 1

head(bw)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["Age"],"name":[1],"type":["int"],"align":["right"]},{"label":["Weight"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Race"],"name":[3],"type":["fctr"],"align":["left"]},{"label":["Smoker"],"name":[4],"type":["fctr"],"align":["left"]},{"label":["Visits"],"name":[5],"type":["int"],"align":["right"]},{"label":["Birth_weight"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["Any_visits"],"name":[7],"type":["dbl"],"align":["right"]}],"data":[{"1":"19","2":"82.55374","3":"black","4":"no","5":"0","6":"2.523","7":"0","_rn_":"1"},{"1":"33","2":"70.30676","3":"other","4":"no","5":"3","6":"2.551","7":"1","_rn_":"2"},{"1":"20","2":"47.62716","3":"white","4":"yes","5":"1","6":"2.557","7":"1","_rn_":"3"},{"1":"21","2":"48.98794","3":"white","4":"yes","5":"2","6":"2.594","7":"1","_rn_":"4"},{"1":"18","2":"48.53434","3":"white","4":"yes","5":"0","6":"2.600","7":"0","_rn_":"5"},{"1":"21","2":"56.24541","3":"other","4":"no","5":"0","6":"2.622","7":"0","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We will now explore the relationship between a mother's age and whether she received any doctor's visits.

Let's first create a plot. We put age on the *x* axis and our new binary 'any visits?' variable on the *y* axis. Because the visits variable has only two categories, we add a bit of vertical jitter to separate out the individual observations.

Because we have coded our 'any visits?' variable as 0 for 'no' and 1 for 'yes', its scale can be thought of as representing the probability of receiving any visits. So we add a label indicating this. To simplify the numerical labelling of the scale, we add a custom scale for the *y* axis. This can be done with the ggplot function `scale_y_continuous()`. The `breaks=` input specifies the points on the scale that we want to display. For probabilities, 0, 0.5, and 1 are sufficient for good visual orientation.


```r
visits_plot = ggplot(bw, aes(y=Any_visits, x=Age)) +
  geom_point(position=position_jitter(width=0, height=0.1)) +
  labs(y='P(visit)', caption='Data: Baystate Medical Center, 1986') +
  scale_y_continuous(breaks=c(0,0.5,1))

print(visits_plot)
```

![](Linear-models_files/figure-html/unnamed-chunk-16-1.png)<!-- -->

It would not be so wrong to just model this probability as a linear function of age. This is known as a 'linear probability model'.


```r
print(visits_plot + geom_smooth(method=lm))
```

![](Linear-models_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

```r
lpm = lm(Any_visits ~ Age, bw)
print(lpm)
```

```
## 
## Call:
## lm(formula = Any_visits ~ Age, data = bw)
## 
## Coefficients:
## (Intercept)          Age  
##     -0.0962       0.0244
```

The slope for age is an estimate of the constant change in the probability of receiving a visit with each additional year of age.

But the linear probability model is perhaps not so satisfying. Because it posits a constant increase in probability, the model will in some places predict a probability less than 0 or greater than 1. This isn't such a problem if such impossible predictions fall well outside the region of the scale that we are interested in. But in this case there are some realistic ages for which the model predcts greater than certain probability of receiving a doctor's visit.

We can check a model's prediction for a new data point using the `predict()` function. The first input is a model and the second input is a new data set containing the values of the predictor(s) for one or more new observations.


```r
predict(lpm, data.frame(Age=49))
```

```
##        1 
## 1.099584
```

It would be mathematically neater if we could ensure that the predicted probabilities are bounded by 0 and 1 and just get arbitrarily close to these values but do not ever reach them. This requires modeling a non-linear relationship of the predictors to the probability outcome.

We can still adapt the basic structure of linear models to this non-linear relationship if we transform the bounded 0 to 1 scale of the probabilities into an unbounded scale. This can be achieved by expressing the probabilities in a different form.

The 'odds' of an event is a numerical quantity that expresses its probability using a number that may go above 1. Odds express probabilities as the ratio of the probability of an event occuring to the probability of it not occurring. This is a notation most familiar from the world of betting, in phrases such as 'fifty-fifty', '10 : 1' or 'a million to one'.

So for example the odds of an event with probability 0.8 are `0.8 / 0.2`, so 4. This expresses the fact that the event is 4 times more likely to occur than it is to not occur.

To quickly experiment with the concept of odds, we will take a brief diversion and learn how to create our own functions in R. We will create a function that takes a probability as its input and then outputs the odds.

We define a function in R using `function()`. Inside the parentheses we give a name to the input of the function. Then we open the curly braces `{}` to describe the workings of the function. In here we do something with the input, using R commands just as normal. At the end of the function we use `return()` to say what the function should 'return' (i.e. what it should output). Like anything else, a function can be stored using `=`. The name under which we store the function is the name that we must then later type in order to use the function on a given new input.


```r
odds = function(p){
  o = p / (1 - p)
  return(o)
}

odds(0.8)
```

```
## [1] 4
```

Once created, our function can be used again and again on many new inputs.


```r
odds(0.95)
```

```
## [1] 19
```

```r
odds(0.5)
```

```
## [1] 1
```

```r
odds(0.1)
```

```
## [1] 0.1111111
```

We can now see that the limit of the odds as a probability approaches 1 is infinity.


```r
odds(1)
```

```
## [1] Inf
```

However, the odds are still bounded by zero at the lower end of the probability scale.


```r
odds(0)
```

```
## [1] 0
```

How can we extend the scale indefinitely in the negative direction? You may recall that logarithms turn positive numbers smaller than 1 into negative numbers.

(The `log()` function in R gives the natural logarithm.)


```r
log(0.1)
```

```
## [1] -2.302585
```

```r
log(0)
```

```
## [1] -Inf
```

So if we convert probabilities into the logarithm of their odds (or 'log odds' for short), we transform them onto an unbounded scale. To quickly check this and to see what the log odds of various probabilities are, let's create another function, this time for converting a probability into log odds.


```r
logodds = function(p){
  o = p / (1 - p)
  return(log(o))
}

logodds(0.8)
```

```
## [1] 1.386294
```

```r
logodds(0.5)
```

```
## [1] 0
```

```r
logodds(0.001)
```

```
## [1] -6.906755
```

R also provides a function `plogis()` for converting log odds back into probabilities.


```r
plogis(0)
```

```
## [1] 0.5
```

```r
plogis(1.5)
```

```
## [1] 0.8175745
```

```r
plogis(-2.5)
```

```
## [1] 0.07585818
```

So a log odds of 0 indicates that the probability of the outcome is 0.5. Positive log odds indicate the outcome is more likely to occur than not, and negative log odds the opposite.

If we model the log odds of an outcome instead of its probability, then we can still use the structure of a linear model to describe the non-linear relationship of predictors to outcome. This is what a 'logistic regression' does.

Since the algorithmic details of fitting such a model to data are rather different from those of a standard linear model, this is handled in R by a different function, `glm()`, which stands for *g*eneralized *l*inear *m*odel. This function handles many different kinds of functions relating predictors to outcome, so we must specify what `family` of models we want. For a logistic regression we want the 'binomial' family ('*bi*nomial' because of the two possible outcomes).

Otherwise, the model formula is the same as for a linear regression.


```r
log_model = glm(Any_visits ~ Age, bw, family=binomial)

summary(log_model)
```

```
## 
## Call:
## glm(formula = Any_visits ~ Age, family = binomial, data = bw)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -1.7200  -1.0747  -0.8347   1.1931   1.6589  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept) -2.55194    0.72333  -3.528 0.000419 ***
## Age          0.10480    0.03052   3.434 0.000595 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 261.37  on 188  degrees of freedom
## Residual deviance: 248.34  on 187  degrees of freedom
## AIC: 252.34
## 
## Number of Fisher Scoring iterations: 4
```

The coefficients can be interpreted much as for a linear model, but bearing in mind that the scale of the outcome is log odds and not probability. So for example the fact that the intercept on th elog odds scale is negative is telling us that the intercept on the probability scale is below 0.5 (recall that the log odds of 0.5 are 0).

A visualization can help a lot. We can do this with `geom_smooth()` just as for linear models, only with `glm` as the method instead of `lm`. We also need some way of including the `family=` input that we gave to `glm()` when we created the model. `geom_smooth()` allows for an input called `method.args=` which specifies a `list()` of inputs for the method.


```r
visits_plot = visits_plot +
  geom_smooth(method=glm, method.args=list(family=binomial))

print(visits_plot)
```

![](Linear-models_files/figure-html/unnamed-chunk-27-1.png)<!-- -->

We can get an additional check of the behaviour of the model by seeing what its predictions are for specific values of the predictor.


```r
predict(log_model, data.frame(Age=20))
```

```
##          1 
## -0.4560273
```

By default, the predictions are given on the log odds scale, which can be difficult to interpret. To get the probability instead, we can specify what `type=` of prediction we want. `'response'` will get us a prediction on the probability scale.

We can check these against the plot above.


```r
predict(log_model, data.frame(Age=20), type='response')
```

```
##         1 
## 0.3879287
```

```r
predict(log_model, data.frame(Age=30), type='response')
```

```
##         1 
## 0.6438071
```

Just as with other linear models, we can include multiple predictors and their interactions in a logistic regression model. For example, here we could explore the association of an interaction of age and smoking with doctor's visits.


```r
print(visits_plot + aes(color=Smoker))
```

![](Linear-models_files/figure-html/unnamed-chunk-30-1.png)<!-- -->

```r
log_model2 = glm(Any_visits ~ Age*Smoker, bw, family=binomial)
summary(log_model2)
```

```
## 
## Call:
## glm(formula = Any_visits ~ Age * Smoker, family = binomial, data = bw)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -1.7371  -1.1089  -0.6572   1.1197   1.8103  
## 
## Coefficients:
##               Estimate Std. Error z value Pr(>|z|)  
## (Intercept)   -1.66693    0.88099  -1.892   0.0585 .
## Age            0.07518    0.03707   2.028   0.0426 *
## Smokeryes     -2.43731    1.55046  -1.572   0.1160  
## Age:Smokeryes  0.08256    0.06505   1.269   0.2044  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 261.37  on 188  degrees of freedom
## Residual deviance: 243.95  on 185  degrees of freedom
## AIC: 251.95
## 
## Number of Fisher Scoring iterations: 4
```
