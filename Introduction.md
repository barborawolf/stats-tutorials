---
title: "Introduction to R"
author: "LT"
output:
  html_document:
    toc: TRUE
    toc_float: TRUE
    df_print: "paged"
    keep_md: TRUE
---

# Commands

R is a programming language. To get R to do things, we must type in commands. We will get R to run commands in two main ways.

If we type a command into the **Console**, it is executed immediately and we will see the results. So we use the console when we want to test something out, see whether it works, check something, and so on.

When we are building up a complete analysis program, we don't want to have to type all the commands in the program into the console. So any commands that form part of our desired final analysis we put into a text file. This file is an R **script**. It is our 'finished product', and if we have constructed it properly it can be run as a whole unit and will carry out all of the steps in our analysis.

Generally, most of the basic commands we will use in R are of four broad types:

* mathematical expressions such as `2 + 2`
* functions, for example the square root function `sqrt(2)`
* assignments of contents into objects, for example `age = 35`
* comments, which serve to annotate our analysis `# (comments look like this)`

## Comments

Comments do nothing at all, but are very useful for letting others know (and for reminding ourselves) what each part of our program is doing. We must write a `#` character before a comment. This lets R know that it is a comment, and ensures that R will not try to execute it. Often, we will first write a comment describing what our program is doing, then below it the R command. For example:


```r
# Add 2 and 2.
2 + 2
```

```
## [1] 4
```

Note that the two blocks of text shown above give first the R commands, then their output. Try copying the command from the first block (i.e. the `2 + 2`) into your own console, then see the result printed out there. Do this for some of the other examples below, but try varying the commands, and check that the results match what you expected.

## Mathematical expressions

Basic mathematical expressions include:


```r
# Subtraction.
3 - 1
```

```
## [1] 2
```

```r
# Multiplication.
2 * 2
```

```
## [1] 4
```

```r
# Division.
1 / 3
```

```
## [1] 0.3333333
```

```r
# Exponentiation.
3 ^ 2
```

```
## [1] 9
```

Lots of other features of basic math are written as you would reasonably expect. For example:


```r
# Non-whole number with a decimal point.
2.718282
```

```
## [1] 2.718282
```

```r
# Negative number with minus sign.
-1
```

```
## [1] -1
```

More complex mathematical expressions are possible, and the usual rules for operator precedence apply. Parentheses play the same role as in standard algebra.


```r
# For example, exponentiation before division.
100 ^ 1 / 2
```

```
## [1] 50
```

```r
# Using parentheses to force the order of operations.
100 ^ (1 / 2)
```

```
## [1] 10
```

## Assignments

Just as in algebra, in R we can assign numbers (or indeed other things) into arbitrary variables, and then write expressions with those variables. We assign using `=`. Whatever is on the right hand side of the `=` is stored for later use, under the name that we give it on the left hand side. We are free to make up a name for our variable as we choose, under some constraints:

* must begin with a letter (otherwise R thinks we are beginning a numerical calculation)
* must contain only letters, numbers, or the symbols `.` and `_`
* no spaces (otherwise R thinks we mean two separate variables)

A basic example:


```r
# Create the variable x.
x = 36

# Write an expression containing the variable.
x + 1
```

```
## [1] 37
```

It is best to choose meaningful names for our variables. This makes our program more intuitive to read.


```r
my_age = 36
my_age + 1
```

```
## [1] 37
```

Assignment can also be done using the two characters `<-`. Some people prefer this, because the arrow shape that this combination makes represents more intuitively what happens for an assignment: some contents are 'going into' the variable.


```r
# Functionally exactly the same as above:
my_age <- 36
my_age + 1
```

```
## [1] 37
```

Use whichever you prefer, but be consistent. I will use `=` because it is one character instead of two, and because `=` is also used for assignment in other programming languages.

## Functions

R has many functions available. Most of them have intuitive names. To apply a function, we type its name followed by parentheses `()` and inside the parentheses we place the input to the function. Some examples:


```r
# Square root.
sqrt(2)
```

```
## [1] 1.414214
```

```r
# Natural logarithm.
log(9000)
```

```
## [1] 9.10498
```

```r
# Exponential function.
exp(1)
```

```
## [1] 2.718282
```

```r
# Absolute value.
abs(-1)
```

```
## [1] 1
```

# Organizing our analysis

## The working directory

Some functions are non-mathematical, and do instead practical things helping us with the organization of our analysis.

The first such useful function is `getwd()`. This function tells us what folder on our computer we are currently working in (*wd* stands for *w*orking *d*irectory). This function needs no input. Nonetheless, it must still be followd by parentheses. The parentheses are R's way of recognizing that what we have typed is a function that we want to execute (and not, for example, the name of a variable).


```r
# This example has printed my working directory.
# Type it into your console to see yours.
getwd()
```

```
## [1] "/home/lt/Dropbox/Teach/Shared/Tutorials"
```

This folder is where R will look by default when searching for data to load, and it is also where R will place any graphs or output files we create.

Use `getwd()` in the console to check what folder you are working in, so that you make sure it is the one you want. If you want to change this folder, the simplest way is to do it via the RStudio **Tools** menu. This was described in the setup guide for the course.

You will rarely need to use `getwd()` in a finished R script. Instead, write your script under the assumption that all the relevant data files are located in whatever folder you are currently working in.

## Printing results

This guide helpfully displays the results of each command in a separate box, so you can see the expected console printout. When we run a completed R script, R will not automatically print results into the console. Instead, when we run a script all the steps of the analysis still take place in the background, but nothing is printed out unless we explicitly ask for it.

We ask for something to be printed using the `print()` function.

This will not be printed out when we run a complete script:


```r
2 + 2
```

But this will:


```r
print(2 + 2)
```

```
## [1] 4
```

# Data

## Loading

We will usually want to load data from an external source. There are many ways to do this. For simplicity, we will mostly stick to reading data from text files. A simple popular format is the .csv file. csv stands for *c*omma *s*eparated *v*alues. It has a simple table-like structure:

* The first row of the file defines the names of the columns. These are the names of the variables in our data set (things like age, sex, reaction time, and so on). These names are separated by commas.
* Subsequent rows each give one observation. For example one participant, or one trial of an experiment.
* The values of the variables in each row are also separated by commas.

We load data from a file like this using the `read.csv()` function. The input is the name of the file. The file should be located in our current working directory. Note that the names of files are enclosed in quotation marks `''`, to distinguish them from the names of variables within R.


```r
read.csv('birth_weights.csv')
```

Here again there is a slight optional variation. The double quotation marks `""` also work. As before, use what you like, but be consistent.


```r
read.csv("birth_weights.csv")
```

On its own, `read.csv()` just reads the file and will simply print out its contents if we run the command in the console. In order to be able to work with the data, we need to assign them into a variable, and give that variable a name. We already know how to do this.

It is fairly common to choose a very abbreviated but still relevant name for the variable in which we store our data. We will need to type it often in our analyses, so we want to avoid something really long.


```r
bw = read.csv('birth_weights.csv')
```

Now we have the data and can access them repeatedly using this variable name.

In our example data set we have the birth weights in kilos of babies at a US hospital, along with data about each baby's mother, such as her weight before the pregnancy, whether she smoked during the pregnancy, etc.


```r
bw
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Age"],"name":[1],"type":["int"],"align":["right"]},{"label":["Weight"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Race"],"name":[3],"type":["fctr"],"align":["left"]},{"label":["Smoker"],"name":[4],"type":["fctr"],"align":["left"]},{"label":["Visits"],"name":[5],"type":["int"],"align":["right"]},{"label":["Birth_weight"],"name":[6],"type":["dbl"],"align":["right"]}],"data":[{"1":"19","2":"82.55374","3":"black","4":"no","5":"0","6":"2.523"},{"1":"33","2":"70.30676","3":"other","4":"no","5":"3","6":"2.551"},{"1":"20","2":"47.62716","3":"white","4":"yes","5":"1","6":"2.557"},{"1":"21","2":"48.98794","3":"white","4":"yes","5":"2","6":"2.594"},{"1":"18","2":"48.53434","3":"white","4":"yes","5":"0","6":"2.600"},{"1":"21","2":"56.24541","3":"other","4":"no","5":"0","6":"2.622"},{"1":"22","2":"53.52386","3":"white","4":"no","5":"1","6":"2.637"},{"1":"17","2":"46.71998","3":"other","4":"no","5":"1","6":"2.637"},{"1":"29","2":"55.79182","3":"white","4":"yes","5":"1","6":"2.663"},{"1":"26","2":"51.25590","3":"white","4":"yes","5":"0","6":"2.665"},{"1":"19","2":"43.09124","3":"other","4":"no","5":"0","6":"2.722"},{"1":"19","2":"68.03880","3":"other","4":"no","5":"1","6":"2.733"},{"1":"22","2":"43.09124","3":"other","4":"no","5":"0","6":"2.751"},{"1":"30","2":"48.53434","3":"other","4":"no","5":"2","6":"2.750"},{"1":"18","2":"45.35920","3":"white","4":"yes","5":"0","6":"2.769"},{"1":"18","2":"45.35920","3":"white","4":"yes","5":"0","6":"2.769"},{"1":"15","2":"44.45202","3":"black","4":"no","5":"0","6":"2.778"},{"1":"25","2":"53.52386","3":"white","4":"yes","5":"3","6":"2.782"},{"1":"20","2":"54.43104","3":"other","4":"no","5":"0","6":"2.807"},{"1":"28","2":"54.43104","3":"white","4":"yes","5":"1","6":"2.821"},{"1":"32","2":"54.88463","3":"other","4":"no","5":"2","6":"2.835"},{"1":"31","2":"45.35920","3":"white","4":"no","5":"3","6":"2.835"},{"1":"36","2":"91.62558","3":"white","4":"no","5":"1","6":"2.836"},{"1":"28","2":"54.43104","3":"other","4":"no","5":"0","6":"2.863"},{"1":"25","2":"54.43104","3":"other","4":"no","5":"2","6":"2.877"},{"1":"28","2":"75.74986","3":"white","4":"no","5":"0","6":"2.877"},{"1":"17","2":"55.33822","3":"white","4":"yes","5":"0","6":"2.906"},{"1":"29","2":"68.03880","3":"white","4":"no","5":"2","6":"2.920"},{"1":"26","2":"76.20346","3":"black","4":"yes","5":"0","6":"2.920"},{"1":"17","2":"51.25590","3":"black","4":"no","5":"1","6":"2.920"},{"1":"17","2":"51.25590","3":"black","4":"no","5":"1","6":"2.920"},{"1":"24","2":"40.82328","3":"white","4":"yes","5":"1","6":"2.948"},{"1":"35","2":"54.88463","3":"black","4":"yes","5":"1","6":"2.948"},{"1":"25","2":"70.30676","3":"white","4":"no","5":"1","6":"2.977"},{"1":"25","2":"56.69900","3":"black","4":"no","5":"0","6":"2.977"},{"1":"29","2":"63.50288","3":"white","4":"yes","5":"2","6":"2.977"},{"1":"19","2":"62.59570","3":"white","4":"yes","5":"2","6":"2.977"},{"1":"27","2":"56.24541","3":"white","4":"yes","5":"0","6":"2.922"},{"1":"31","2":"97.52228","3":"white","4":"yes","5":"2","6":"3.005"},{"1":"33","2":"49.44153","3":"white","4":"yes","5":"1","6":"3.033"},{"1":"21","2":"83.91452","3":"black","4":"yes","5":"2","6":"3.042"},{"1":"19","2":"85.72889","3":"white","4":"no","5":"2","6":"3.062"},{"1":"23","2":"58.96696","3":"black","4":"no","5":"1","6":"3.062"},{"1":"21","2":"72.57472","3":"white","4":"no","5":"0","6":"3.062"},{"1":"18","2":"40.82328","3":"white","4":"yes","5":"0","6":"3.062"},{"1":"18","2":"40.82328","3":"white","4":"yes","5":"0","6":"3.062"},{"1":"32","2":"59.87414","3":"white","4":"no","5":"4","6":"3.080"},{"1":"19","2":"59.87414","3":"other","4":"no","5":"0","6":"3.090"},{"1":"24","2":"52.16308","3":"white","4":"no","5":"2","6":"3.090"},{"1":"22","2":"38.55532","3":"other","4":"yes","5":"0","6":"3.090"},{"1":"22","2":"54.43104","3":"white","4":"no","5":"1","6":"3.100"},{"1":"23","2":"58.05978","3":"other","4":"no","5":"0","6":"3.104"},{"1":"22","2":"58.96696","3":"white","4":"yes","5":"0","6":"3.132"},{"1":"30","2":"43.09124","3":"white","4":"yes","5":"2","6":"3.147"},{"1":"19","2":"52.16308","3":"other","4":"no","5":"0","6":"3.175"},{"1":"16","2":"49.89512","3":"other","4":"no","5":"0","6":"3.175"},{"1":"21","2":"49.89512","3":"other","4":"yes","5":"0","6":"3.203"},{"1":"30","2":"69.39958","3":"other","4":"no","5":"0","6":"3.203"},{"1":"20","2":"46.71998","3":"other","4":"no","5":"0","6":"3.203"},{"1":"17","2":"53.97745","3":"other","4":"no","5":"0","6":"3.225"},{"1":"17","2":"53.97745","3":"other","4":"no","5":"0","6":"3.225"},{"1":"23","2":"53.97745","3":"other","4":"no","5":"2","6":"3.232"},{"1":"24","2":"49.89512","3":"other","4":"no","5":"0","6":"3.232"},{"1":"28","2":"63.50288","3":"white","4":"no","5":"0","6":"3.234"},{"1":"26","2":"60.32774","3":"other","4":"yes","5":"0","6":"3.260"},{"1":"20","2":"76.65705","3":"other","4":"no","5":"1","6":"3.274"},{"1":"24","2":"52.16308","3":"other","4":"no","5":"2","6":"3.274"},{"1":"28","2":"113.39800","3":"other","4":"yes","5":"6","6":"3.303"},{"1":"20","2":"63.95647","3":"white","4":"no","5":"1","6":"3.317"},{"1":"22","2":"71.66754","3":"black","4":"no","5":"2","6":"3.317"},{"1":"22","2":"50.80230","3":"white","4":"yes","5":"0","6":"3.317"},{"1":"31","2":"68.03880","3":"other","4":"yes","5":"2","6":"3.321"},{"1":"23","2":"52.16308","3":"other","4":"yes","5":"1","6":"3.331"},{"1":"16","2":"50.80230","3":"black","4":"no","5":"0","6":"3.374"},{"1":"16","2":"61.23492","3":"white","4":"yes","5":"0","6":"3.374"},{"1":"18","2":"103.87257","3":"black","4":"no","5":"0","6":"3.402"},{"1":"25","2":"63.50288","3":"white","4":"no","5":"1","6":"3.416"},{"1":"32","2":"60.78133","3":"white","4":"yes","5":"4","6":"3.430"},{"1":"20","2":"54.88463","3":"black","4":"yes","5":"0","6":"3.444"},{"1":"23","2":"86.18248","3":"white","4":"no","5":"0","6":"3.459"},{"1":"22","2":"59.42055","3":"white","4":"no","5":"1","6":"3.460"},{"1":"32","2":"77.11064","3":"white","4":"no","5":"0","6":"3.473"},{"1":"30","2":"49.89512","3":"other","4":"no","5":"0","6":"3.544"},{"1":"20","2":"57.60618","3":"other","4":"no","5":"0","6":"3.487"},{"1":"23","2":"55.79182","3":"other","4":"no","5":"0","6":"3.544"},{"1":"17","2":"54.43104","3":"other","4":"yes","5":"0","6":"3.572"},{"1":"19","2":"47.62716","3":"other","4":"no","5":"0","6":"3.572"},{"1":"23","2":"58.96696","3":"white","4":"no","5":"0","6":"3.586"},{"1":"36","2":"79.37860","3":"white","4":"no","5":"0","6":"3.600"},{"1":"22","2":"56.69900","3":"white","4":"no","5":"1","6":"3.614"},{"1":"24","2":"60.32774","3":"white","4":"no","5":"0","6":"3.614"},{"1":"21","2":"60.78133","3":"other","4":"no","5":"2","6":"3.629"},{"1":"19","2":"106.59412","3":"white","4":"yes","5":"0","6":"3.629"},{"1":"25","2":"43.09124","3":"white","4":"yes","5":"0","6":"3.637"},{"1":"16","2":"61.23492","3":"white","4":"yes","5":"0","6":"3.643"},{"1":"29","2":"61.23492","3":"white","4":"no","5":"1","6":"3.651"},{"1":"29","2":"69.85317","3":"white","4":"no","5":"1","6":"3.651"},{"1":"19","2":"66.67802","3":"white","4":"yes","5":"0","6":"3.651"},{"1":"19","2":"66.67802","3":"white","4":"yes","5":"0","6":"3.651"},{"1":"30","2":"62.14210","3":"white","4":"no","5":"1","6":"3.699"},{"1":"24","2":"49.89512","3":"white","4":"no","5":"1","6":"3.728"},{"1":"19","2":"83.46093","3":"white","4":"yes","5":"0","6":"3.756"},{"1":"24","2":"49.89512","3":"other","4":"no","5":"0","6":"3.770"},{"1":"23","2":"49.89512","3":"white","4":"no","5":"1","6":"3.770"},{"1":"20","2":"54.43104","3":"other","4":"no","5":"0","6":"3.770"},{"1":"25","2":"109.31567","3":"black","4":"no","5":"0","6":"3.790"},{"1":"30","2":"50.80230","3":"white","4":"no","5":"1","6":"3.799"},{"1":"22","2":"76.65705","3":"white","4":"no","5":"0","6":"3.827"},{"1":"18","2":"54.43104","3":"white","4":"yes","5":"2","6":"3.856"},{"1":"16","2":"77.11064","3":"black","4":"no","5":"4","6":"3.860"},{"1":"32","2":"84.36811","3":"white","4":"no","5":"2","6":"3.860"},{"1":"18","2":"54.43104","3":"other","4":"no","5":"1","6":"3.884"},{"1":"29","2":"58.96696","3":"white","4":"yes","5":"2","6":"3.884"},{"1":"33","2":"53.07026","3":"white","4":"no","5":"1","6":"3.912"},{"1":"20","2":"77.11064","3":"white","4":"yes","5":"0","6":"3.940"},{"1":"28","2":"60.78133","3":"other","4":"no","5":"1","6":"3.941"},{"1":"14","2":"61.23492","3":"white","4":"no","5":"0","6":"3.941"},{"1":"28","2":"58.96696","3":"other","4":"no","5":"0","6":"3.969"},{"1":"25","2":"54.43104","3":"white","4":"no","5":"2","6":"3.983"},{"1":"16","2":"43.09124","3":"other","4":"no","5":"1","6":"3.997"},{"1":"20","2":"71.66754","3":"white","4":"no","5":"1","6":"3.997"},{"1":"26","2":"72.57472","3":"other","4":"no","5":"0","6":"4.054"},{"1":"21","2":"52.16308","3":"white","4":"no","5":"1","6":"4.054"},{"1":"22","2":"58.51337","3":"white","4":"no","5":"0","6":"4.111"},{"1":"25","2":"58.96696","3":"white","4":"no","5":"2","6":"4.153"},{"1":"31","2":"54.43104","3":"white","4":"no","5":"2","6":"4.167"},{"1":"35","2":"77.11064","3":"white","4":"no","5":"1","6":"4.174"},{"1":"19","2":"54.43104","3":"white","4":"yes","5":"0","6":"4.238"},{"1":"24","2":"52.61667","3":"white","4":"no","5":"1","6":"4.593"},{"1":"45","2":"55.79182","3":"white","4":"no","5":"1","6":"4.990"},{"1":"28","2":"54.43104","3":"other","4":"yes","5":"0","6":"0.709"},{"1":"29","2":"58.96696","3":"white","4":"no","5":"2","6":"1.021"},{"1":"34","2":"84.82170","3":"black","4":"yes","5":"0","6":"1.135"},{"1":"25","2":"47.62716","3":"other","4":"no","5":"0","6":"1.330"},{"1":"25","2":"38.55532","3":"other","4":"no","5":"0","6":"1.474"},{"1":"27","2":"68.03880","3":"other","4":"no","5":"0","6":"1.588"},{"1":"23","2":"43.99842","3":"other","4":"no","5":"1","6":"1.588"},{"1":"24","2":"58.05978","3":"black","4":"no","5":"1","6":"1.701"},{"1":"24","2":"59.87414","3":"other","4":"no","5":"0","6":"1.729"},{"1":"21","2":"74.84268","3":"white","4":"yes","5":"1","6":"1.790"},{"1":"32","2":"47.62716","3":"white","4":"yes","5":"0","6":"1.818"},{"1":"19","2":"41.27687","3":"white","4":"yes","5":"0","6":"1.885"},{"1":"25","2":"52.16308","3":"other","4":"no","5":"0","6":"1.893"},{"1":"16","2":"58.96696","3":"other","4":"no","5":"1","6":"1.899"},{"1":"25","2":"41.73046","3":"white","4":"yes","5":"0","6":"1.928"},{"1":"20","2":"68.03880","3":"white","4":"yes","5":"2","6":"1.928"},{"1":"21","2":"90.71840","3":"black","4":"no","5":"2","6":"1.928"},{"1":"24","2":"70.30676","3":"white","4":"yes","5":"0","6":"1.936"},{"1":"21","2":"46.71998","3":"other","4":"no","5":"0","6":"1.970"},{"1":"20","2":"56.69900","3":"other","4":"no","5":"0","6":"2.055"},{"1":"25","2":"40.36969","3":"other","4":"no","5":"1","6":"2.055"},{"1":"19","2":"46.26638","3":"white","4":"no","5":"2","6":"2.082"},{"1":"19","2":"50.80230","3":"white","4":"yes","5":"0","6":"2.084"},{"1":"26","2":"53.07026","3":"white","4":"yes","5":"0","6":"2.084"},{"1":"24","2":"62.59570","3":"white","4":"no","5":"0","6":"2.100"},{"1":"17","2":"58.96696","3":"other","4":"yes","5":"0","6":"2.125"},{"1":"20","2":"54.43104","3":"black","4":"yes","5":"3","6":"2.126"},{"1":"22","2":"58.96696","3":"white","4":"yes","5":"1","6":"2.187"},{"1":"27","2":"58.96696","3":"black","4":"no","5":"0","6":"2.187"},{"1":"20","2":"36.28736","3":"other","4":"yes","5":"0","6":"2.211"},{"1":"17","2":"49.89512","3":"white","4":"yes","5":"0","6":"2.225"},{"1":"25","2":"47.62716","3":"other","4":"no","5":"1","6":"2.240"},{"1":"20","2":"49.44153","3":"other","4":"no","5":"0","6":"2.240"},{"1":"18","2":"67.13162","3":"other","4":"no","5":"0","6":"2.282"},{"1":"18","2":"49.89512","3":"black","4":"yes","5":"0","6":"2.296"},{"1":"20","2":"54.88463","3":"white","4":"yes","5":"0","6":"2.296"},{"1":"21","2":"45.35920","3":"other","4":"no","5":"4","6":"2.301"},{"1":"26","2":"43.54483","3":"other","4":"no","5":"0","6":"2.325"},{"1":"31","2":"46.26638","3":"white","4":"yes","5":"1","6":"2.353"},{"1":"15","2":"49.89512","3":"white","4":"no","5":"0","6":"2.353"},{"1":"23","2":"84.82170","3":"black","4":"yes","5":"1","6":"2.367"},{"1":"20","2":"55.33822","3":"black","4":"yes","5":"0","6":"2.381"},{"1":"24","2":"47.62716","3":"black","4":"yes","5":"0","6":"2.381"},{"1":"15","2":"52.16308","3":"other","4":"no","5":"0","6":"2.381"},{"1":"23","2":"54.43104","3":"other","4":"no","5":"0","6":"2.410"},{"1":"30","2":"64.41006","3":"white","4":"yes","5":"0","6":"2.410"},{"1":"22","2":"58.96696","3":"white","4":"yes","5":"1","6":"2.410"},{"1":"17","2":"54.43104","3":"white","4":"yes","5":"3","6":"2.414"},{"1":"23","2":"49.89512","3":"white","4":"yes","5":"0","6":"2.424"},{"1":"17","2":"54.43104","3":"black","4":"no","5":"2","6":"2.438"},{"1":"26","2":"69.85317","3":"other","4":"no","5":"1","6":"2.442"},{"1":"20","2":"47.62716","3":"other","4":"no","5":"3","6":"2.450"},{"1":"26","2":"86.18248","3":"white","4":"yes","5":"0","6":"2.466"},{"1":"14","2":"45.81279","3":"other","4":"yes","5":"0","6":"2.466"},{"1":"28","2":"43.09124","3":"white","4":"yes","5":"2","6":"2.466"},{"1":"14","2":"45.35920","3":"other","4":"no","5":"2","6":"2.495"},{"1":"23","2":"42.63765","3":"other","4":"yes","5":"0","6":"2.495"},{"1":"17","2":"64.41006","3":"black","4":"no","5":"0","6":"2.495"},{"1":"21","2":"58.96696","3":"white","4":"yes","5":"3","6":"2.495"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Checking

Once we have our data loaded, it is important to check whether they have the structure we expect before we begin any analysis. There are various helpful functions for checking our data.


```r
# See the names of the columns.
names(bw)
```

```
## [1] "Age"          "Weight"       "Race"         "Smoker"      
## [5] "Visits"       "Birth_weight"
```

```r
# See the first few rows.
head(bw)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["Age"],"name":[1],"type":["int"],"align":["right"]},{"label":["Weight"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Race"],"name":[3],"type":["fctr"],"align":["left"]},{"label":["Smoker"],"name":[4],"type":["fctr"],"align":["left"]},{"label":["Visits"],"name":[5],"type":["int"],"align":["right"]},{"label":["Birth_weight"],"name":[6],"type":["dbl"],"align":["right"]}],"data":[{"1":"19","2":"82.55374","3":"black","4":"no","5":"0","6":"2.523","_rn_":"1"},{"1":"33","2":"70.30676","3":"other","4":"no","5":"3","6":"2.551","_rn_":"2"},{"1":"20","2":"47.62716","3":"white","4":"yes","5":"1","6":"2.557","_rn_":"3"},{"1":"21","2":"48.98794","3":"white","4":"yes","5":"2","6":"2.594","_rn_":"4"},{"1":"18","2":"48.53434","3":"white","4":"yes","5":"0","6":"2.600","_rn_":"5"},{"1":"21","2":"56.24541","3":"other","4":"no","5":"0","6":"2.622","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# See summaries of the data in each column.
summary(bw)
```

```
##       Age            Weight          Race    Smoker        Visits      
##  Min.   :14.00   Min.   : 36.29   black:26   no :115   Min.   :0.0000  
##  1st Qu.:19.00   1st Qu.: 49.90   other:67   yes: 74   1st Qu.:0.0000  
##  Median :23.00   Median : 54.88   white:96             Median :0.0000  
##  Mean   :23.24   Mean   : 58.88                        Mean   :0.7937  
##  3rd Qu.:26.00   3rd Qu.: 63.50                        3rd Qu.:1.0000  
##  Max.   :45.00   Max.   :113.40                        Max.   :6.0000  
##   Birth_weight  
##  Min.   :0.709  
##  1st Qu.:2.414  
##  Median :2.977  
##  Mean   :2.945  
##  3rd Qu.:3.487  
##  Max.   :4.990
```

```r
# Check how many rows of data we have.
nrow(bw)
```

```
## [1] 189
```

We can pick out just one of the columns of our data using the `$` symbol, followed by the name of the column.


```r
# See a summary of just the Weight column.
summary(bw$Weight)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   36.29   49.90   54.88   58.88   63.50  113.40
```

```r
# The mean birth weight.
mean(bw$Birth_weight)
```

```
## [1] 2.944587
```

```r
# The standard deviation of age.
sd(bw$Age)
```

```
## [1] 5.298678
```

Some columns give categorical information, not numeric. R calls categorical variables 'factors'. The summary of a factor just tells us how many observations we have in each category.


```r
summary(bw$Smoker)
```

```
##  no yes 
## 115  74
```

We can also get the same information using the `table()` function. But the more useful feature of `table()` is to create cross-categorized tables using two factors. To achieve this, we must input both factors to `table()`.

Functions that can have multiple inputs require the inputs to be separated by commas. The order of the inputs matters. In the case of `table()`, the first input is the factor whose categories will be given as rows, and the second input is the factor whose categories will be given as columns. (This is a common pattern in R: rows first, columns second.)


```r
# Smoking status.
table(bw$Smoker)
```

```
## 
##  no yes 
## 115  74
```

```r
# Smoking by race.
table(bw$Smoker,bw$Race)
```

```
##      
##       black other white
##   no     16    55    44
##   yes    10    12    52
```

```r
# The other way around.
table(bw$Race,bw$Smoker)
```

```
##        
##         no yes
##   black 16  10
##   other 55  12
##   white 44  52
```

This use of cross-categorized tables is good for checking whether we have the expected number of combinations of conditions in our data. For example, whether each participant did each type of trial as many times as we expect.

# Analysis

We have already seen above how to calculate some basic descriptive statistics. Sometimes we will need to calculate something a bit more elaborate, and R will not have a function ready to help us.

For example, R does not have a function for calculating the (parametric) standard error. To achieve something like this, we have to put together a few of the ingredients we have learned above.


```r
# SE of weights.
# (SD divided by square root of sample size,
# sample size given by number of rows in data).
sd(bw$Weight) / sqrt(nrow(bw))
```

```
## [1] 1.008935
```

Even for a fairly short calculation like this, we can make our R script a bit more intuitively readable by breaking down a multi-step calculation into separate lines, and storing the results of each intermediate step in an intuitively-named variable.


```r
# SE of weights.
sd_Weight = sd(bw$Weight)
n = nrow(bw)
se_Weight = sd_Weight / sqrt(n)
print(se_Weight)
```

```
## [1] 1.008935
```

## Formulae

Statistical procedures in R are usually carried out using a 'formula'. This formula describes a statistical model of the relationships among the variables in our data. Just as in many mathematical formulaic descriptions of models, the outcome variable goes at the left end of the formula, and the predictor variables go on the right. R uses the `~` character to separate outcome from predictors.

We input the formula, using the names of the columns from our data, and then as a second input we give the data themselves (so R knows where to look to find the variables mentioned in the formula).

For example, the humble two-sample *t*-test is based on a very simple model, with a numerical outcome variable and a single categorical predictor. We will calculate this test for the model that explains birth weight by smoking status. (The *t*-test tests the null model that denies that this relationship exists at all in the sampled population).

In this case, the output gives us a *p*-value, a confidence interval for the population difference in mean birth weight, and the observed sample means, among other things.


```r
t.test(Birth_weight ~ Smoker, bw)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  Birth_weight by Smoker
## t = 2.7299, df = 170.1, p-value = 0.007003
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  0.07857486 0.48897860
## sample estimates:
##  mean in group no mean in group yes 
##          3.055696          2.771919
```

Many of the functions that execute statistical procedures can be tuned by specifying additional options. You may have noticed that the `t.test()` function above gave us the Welch test by default (with the degrees of freedom corrected for a difference in variances between the two groups). We can alter this behaviour by giving `t.test()` an additional input. These additional inputs must usually be named, so that R knows what aspect of the procedure we want to change. Here the required name is `var.equal` (short for *var*iances *equal*). We set these named options using the same `=` that we use to assign things. In this case, since the option is something that can either be turned on or turned off (we can either assume equal variances or not), we can give it the value `TRUE` or `FALSE`.


```r
t.test(Birth_weight ~ Smoker, bw, var.equal=TRUE)
```

```
## 
## 	Two Sample t-test
## 
## data:  Birth_weight by Smoker
## t = 2.6529, df = 187, p-value = 0.008667
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  0.07275612 0.49479735
## sample estimates:
##  mean in group no mean in group yes 
##          3.055696          2.771919
```

## Aggregation

Often the same formula that was used to apply a statistical procedure can also be used to calculate the summary statistics that should accompany the procedure.

The `aggregate()` function takes a formula as its input. Just as for a statistical model, the formula specifies what outcome variable we want to summarize, and what other variable(s) we want to split the summary by. We then input the data, and the name of the function we want to use in summarizing the data (often this will be `mean`).

For example, to get separate mean birth weights for the two smoking groups we can copy and paste the same formula we used above.


```r
aggregate(Birth_weight ~ Smoker, bw, mean)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Smoker"],"name":[1],"type":["fctr"],"align":["left"]},{"label":["Birth_weight"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"no","2":"3.055696"},{"1":"yes","2":"2.771919"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

To get other summary statistics, we just change the function that we input to `aggregate()`. For example for the Standard Deviation:


```r
aggregate(Birth_weight ~ Smoker, bw, sd)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Smoker"],"name":[1],"type":["fctr"],"align":["left"]},{"label":["Birth_weight"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"no","2":"0.7526566"},{"1":"yes","2":"0.6596349"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Models

Formulas are also used to specify the form of a model that we wish to fit to our data. For example, we can estimate the parameters of a linear regression model. The general function for linear models is `lm()`.

For example, here we estimate a linear regression with birth weight as the outcome and mother's weight as predictor.

We get the estimated coefficients of the model: intercept and slope.


```r
lm(Birth_weight ~ Weight, bw)
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

To get a more detailed examination of our model, including hypothesis tests of the model coefficients, and measures of model fit such as R-squared, we can apply the `summary()` function.


```r
summary(lm(Birth_weight ~ Weight, bw))
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

Models can also be stored for later use with `=`.


```r
model1 = lm(Birth_weight ~ Weight, bw)

summary(model1)
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

Many procedures allow for multiple predictors. To include more than one predictor variable in a linear regression, we join them in the formula with `+`.


```r
model2 = lm(Birth_weight ~ Weight + Smoker, bw)

summary(model2)
```

```
## 
## Call:
## lm(formula = Birth_weight ~ Weight + Smoker, data = bw)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -2.03090 -0.44569  0.02916  0.52176  1.96776 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.501125   0.230836  10.835   <2e-16 ***
## Weight       0.009340   0.003726   2.507   0.0130 *  
## Smokeryes   -0.272081   0.105591  -2.577   0.0107 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7078 on 186 degrees of freedom
## Multiple R-squared:  0.06777,	Adjusted R-squared:  0.05775 
## F-statistic: 6.761 on 2 and 186 DF,  p-value: 0.001464
```

Interactions between predictors can be added using the `*` symbol. Including an interaction like this also by default includes the simple effects of the individual predictors contained in the interaction.


```r
model3 = lm(Birth_weight ~ Weight * Smoker, bw)

summary(model3)
```

```
## 
## Call:
## lm(formula = Birth_weight ~ Weight * Smoker, data = bw)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -2.03880 -0.45476  0.02836  0.53084  1.97684 
## 
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)       2.350578   0.312733   7.516 2.35e-12 ***
## Weight            0.011876   0.005148   2.307   0.0222 *  
## Smokeryes         0.041384   0.451187   0.092   0.9270    
## Weight:Smokeryes -0.005339   0.007470  -0.715   0.4757    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7088 on 185 degrees of freedom
## Multiple R-squared:  0.07034,	Adjusted R-squared:  0.05527 
## F-statistic: 4.666 on 3 and 185 DF,  p-value: 0.003621
```

A useful general function for comparing models is the `anova()` function. If we input to it more than one different model (of the same data), it will compare the models in some way. What comparison it performs will depend on the type of model. For our linear regression models we get the *F*-test.


```r
anova(model1, model2, model3)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["Res.Df"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["RSS"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Df"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Sum of Sq"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["F"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["Pr(>F)"],"name":[6],"type":["dbl"],"align":["right"]}],"data":[{"1":"187","2":"96.52102","3":"NA","4":"NA","5":"NA","6":"NA","_rn_":"1"},{"1":"186","2":"93.19430","3":"1","4":"3.3267196","5":"6.6221025","6":"0.01085545","_rn_":"2"},{"1":"185","2":"92.93772","3":"1","4":"0.2565761","5":"0.5107353","6":"0.47572156","_rn_":"3"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Of course, before going ahead with any analyses, we should first plot our data. Plotting is the topic of the next class.

# Troubleshooting

## Errors

Sometimes we get things wrong. A misplaced comma or parenthesis will stop R commands from working as desired. If we type in something that just 'doesn't work', R will stop and print out an error message.

Sometimes the text of this error message will be fairly informative and helpful:


```r
# R will here correctly identify the unexpected extra parenthesis here:
lm(Birth_weight ~ Weight, bw))
```

```
## Error: <text>:2:30: unexpected ')'
## 1: # R will here correctly identify the unexpected extra parenthesis here:
## 2: lm(Birth_weight ~ Weight, bw))
##                                 ^
```

At other times R will not guess correctly what we were aiming for and what exactly went wrong:


```r
# R will not spot here that the problem is the $ and not the ):
lm(Birth_weight ~ Weight, bw$)
```

```
## Error: <text>:2:30: unexpected ')'
## 1: # R will not spot here that the problem is the $ and not the ):
## 2: lm(Birth_weight ~ Weight, bw$)
##                                 ^
```

A common error is to get the names of variables very slightly wrong. For example, all names in R are case sensitive, so we need to be careful about this:


```r
lm(birth_weight ~ Weight, bw)
```

```
## Error in eval(predvars, data, env): object 'birth_weight' not found
```

## Help

If we know what function we want to use, but we are not sure how to use it, the `?` can call up documentation for a function, for example `? t.test`.

(You will need to type the `?` into your own RStudio console to see the documentation.)

Under the section **Usage** we see a short template of the inputs that the function expects. Under **Arguments** we get more details about the nature of each input. But often the **Examples** section is the most informative.

For many things, there is clearer and more detailed help available online. A clearly-worded Google search will almost always get you to an example or explanation on one of the main R and programming community sites, such as [StackOverflow](https://stackoverflow.com/) or [R-bloggers](https://www.r-bloggers.com/).
