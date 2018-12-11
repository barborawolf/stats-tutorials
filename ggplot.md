---
title: "Data visualization with ggplot"
author: "LT"
output:
  html_document:
    toc: TRUE
    toc_float: TRUE
    df_print: "paged"
    keep_md: TRUE
---

# Packages

In the introductory tutorial we saw the basic use of R, loading data and making a few checks and calculations. For some of the more specialized tasks in data analysis, basic R won't be enough. We will need to call upon some additional 'packages' that must first be downloaded and installed.

You can download and install a package from within RStudio. You don't need to go to a website or search for an installation file online. Go to the **Packages** tab in RStudio, and there you will see a list of all the extra packages you currently have installed, along with a short description and version number for each. To search for a package and then install it, click on **Install** and then type the name of the package you want. RStudio will autocomplete a package name once you start typing.

If you followed the setup instructions for the course, you will already have installed the package that we will use now, ggplot2, so you don't need to install it again.

However, when we want to use an installed package in an R script, we need to load the package, and this is something that we need to do for each new script that uses the package. Fortunately, loading packages is easy. The `library()` function loads a package, and the input is the name of the package.


```r
library(ggplot2)
```

If successful, the `library()` function outputs nothing, so you won't see anything printed in the console. If you see an error message telling you there is no such package, either you have spelled the name of the package wrong, or it is not installed. Go and install it first.

We should put any `library()` commands at or near the very top of our R script, so that other people can see easily what packages they will need to have if they want to run our data analysis.

# ggplot

The ggplot package brings with it lots of functions for visualizing data. The main function is `ggplot()`. Note that the name of the *package* is 'ggplot2', but the *function* that the package brings us is called just 'ggplot'.

The *gg* in ggplot stands for *g*rammar of *g*raphics, a particular conceptual approach to describing graphs. You can read about it in more detail [here](https://www.tandfonline.com/doi/abs/10.1198/jcgs.2009.07098), but the core idea is to describe graphs in terms of three components:

* data
* aesthetic mapping, a mapping of variables from the data onto dimensions of the graph, such as:
  + *x* dimension
  + *y* dimension
  + color
  + size
* geometric objects, the shapes used to represent the data, such as:
  + points
  + lines
  + bars
  + or more complex shapes like boxplots

Let's load the example data set and create a plot. We will make a classic scatterplot, showing the mothers' weights on the *x* axis and their babies' weights on the *y* axis.

We use the main `ggplot()` function to say what data we want to use and what aesthetic mapping we want. The first input is our data set. The second input is itself a function, the `aes()` function. *aes* is short for *aesthetic*, and this function organizes the aesthetic mapping as described above. Inside the `aes()` function, we assign variables from the data set to dimensions of the plot. We do this using the same `=` that we use for assignment in general.


```r
bw = read.csv('birth_weights.csv')

ggplot(bw, aes(y=Birth_weight, x=Weight))
```

![](ggplot_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

In this first extremely minimal example, we didn't yet add on a geometric object. So the plot is empty. But notice that ggplot already does some useful automatic jobs. It has created the *x* and *y* scales with the necessary range, and has added gridlines to the plot background.

Now we will do it again with points as our geometric object, to produce the finished scatterplot. Geometric objects are added on to the core plot definition using `+`. Each type of geometric object has its own function. The one that we want is called `geom_point()`. Because we have already defined the organization of the plot with `aes()`, `geom_point()` doesn't need any input telling it what to plot or where.


```r
ggplot(bw, aes(y=Birth_weight, x=Weight)) + geom_point()
```

![](ggplot_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

If we use any other dimensions when defining the plot, such as color, size, or shape, any geometric object that is able to represent that dimension will take this into account. For example, points can be shown in different colors if a factor variable is mapped to the color dimension.


```r
ggplot(bw, aes(y=Birth_weight, x=Weight, color=Smoker)) + geom_point()
```

![](ggplot_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

## Adding to plots

We will often want to have a few different versions of our plot, for example a basic version showing the raw data, perhaps a version showing the overall means, a version that splits the data into two subgroups, and so on. It would be tedious to have to repeat the basic underlying plot definition for each version. We should avoid this unnecessary repetition. The first step in doing so is to store the most basic form of our plot, so that we can use it repeatedly. We can store a plot using `=` just as we do for storing loaded data or other information. Here we store it under the name 'fig1'.


```r
fig1 = ggplot(bw, aes(y=Birth_weight, x=Weight, color=Smoker)) + geom_point()
```

Now the plot is stored and we can re-use it under this name. One thing we can do with a stored plot is display it again. This is done with the same `print()` function we saw earlier.


```r
print(fig1)
```

![](ggplot_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

We can now add a new geometric object to the stored plot, and store the result in a new object, so we have our two versions. Let's add a smooth line representing the relationship between our *x* and *y* variables.


```r
fig2 = fig1 + geom_smooth()
print(fig2)
```

```
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
```

![](ggplot_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

Notice that we also got a short message printed out when we ran `geom_smooth()`. This is not an error message but simply a warning. Some functions when we run them will print out a reminder of what the function is doing, so that we can check it is really what we wanted. This occurs most often in cases where the function's default behavior might not be what some users typically want.

The content of the warning message tells us something about a 'method' called 'loess', which `geom_smooth()` has used. *loess* stands for *lo*cally *es*timated *s*mooth, and calculates a regression of *y* onto *x* locally for each region of the *x* scale. This is more complex than we will usually need. We can change it to a linear regression.

To change the behaviors of plotting functions, we must give them some input specifying what aspect of their behavior we want to change and what we want to change it to. We will change the `method` to `lm` for a linear model. Note that the `lm` refers to the same function that we used to calculate a linear regression in the previous class. It is being used here to produce the parameters for the line, which `geom_smooth()` then draws.


```r
fig2 = fig1 + geom_smooth(method=lm)
print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

Most plotting functions have lots and lots of options that we can change in order to fine-tune our plot. For example, we can turn off the standard error region that `geom_smooth()` draws by default. Options that are just turned on or off are controlled by the logical values `TRUE` and `FALSE`. These must be written in uppercase.


```r
fig2 = fig1 + geom_smooth(method=lm, se=FALSE)
print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

The order in which we add geometric objects to the plot matters. They will be drawn in that order, so later objects that overlap with earlier objects will obscure them. For example, if we had created our scatterplot first with the smoothing line and then with the points, it would look slightly different.

(The difference in this case is fairly subtle, but if you look carefully you notice that some of the points are now drawn over the line.)


```r
ggplot(bw, aes(y=Birth_weight, x=Weight, color=Smoker)) + geom_smooth() + geom_point()
```

```
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
```

![](ggplot_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

## Plot extras

There are a few extra things that we can add to a plot that are not central to its structure, but that can help to prettify it a bit.

Fairly often we will want to modify the labels on the axes. By default, they are given the names of the variables in our data set. So where feasible, we can make plotting easier by just naming the variables appropriately in the original table of data. But if we want labels with spaces in them, or if we want to state the units of the variable, that can get unwieldy. An alternative is to specify labels manually using the `labs()` function.

As with `aes()`, the inputs to the `labs()` function are assignments to plot dimensions. Because the labels are just literal text, and do not refer to a function or object in R, they must be given in quotation marks.


```r
fig2 = fig2 + labs(y='Birth weight (kg)', x='Weight (kg)')
print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

We can also use `labs()` to assign a caption. We don't always need this, but one good use of the caption is to provide a brief reference for where the data come from. This can be important for plots that might be shown without their original context, because it ensures that a record of the source is built in to the plot image.


```r
fig2 = fig2 + labs(caption='Data: Baystate Medical Center, 1986')
print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

`labs()` can also assign a title at the top of the plot. We won't always need this, but one use of a title is to assign a label to a plot that we will later use as one of several plots presented at once.


```r
fig2 = fig2 + labs(title='A')
print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

We can modify the overall style of the plot by adding on a 'theme'. There are various theme functions, and you can see the look of them at the [ggplot reference website](https://ggplot2.tidyverse.org/reference/ggtheme.html). For example, the 'classic' theme does not have guiding gridlines or a shaded background.


```r
print(fig2 + theme_classic())
```

![](ggplot_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

## Scales

In the example plots above, ggplot decided for us what range of values to show on each axis scale. And it tends to make fairly good decision about this. But sometimes we will want to take control of this aspect of our plots ourselves. For example, if we are showing data that come from a discrete rating scale we might want to show only particular values.

Various `scale_` functions handle this. There are lots of such functions, and it can sometimes take a bit of work to find out which one we want. But generally the function will have the name of the plot dimension, plus some indication of what kind of scale we want to apply. For example, if we want to change the values shown for 'Birth weight' in our plot above, we need the function `scale_y_continuous()`, because we are dealing with the *y* dimension and a continous scale.

The `breaks=` input tells the `scale_` function at what points on the scale we would like to show values. We can input several numbers, using the `c()` function to group the numbers together.


```r
print(fig2 + scale_y_continuous(breaks=c(1,3,5)))
```

![](ggplot_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

The same principle applies to color scales. To change the colors assigned to categories, use `scale_color_manual()`. The required input is again a set of values grouped together using `c()`. Since these are not 'break points' along a scale, but just categorical values, the input is called `values=`. The names of colors are given in `''`.


```r
print(fig2 + scale_color_manual(values=c('skyblue','brown')))
```

![](ggplot_files/figure-html/unnamed-chunk-16-1.png)<!-- -->

If you are wondering what named colors are available in R, you can see a full list of them by typing the `colors()` function into the console.


```r
colors()
```

```
##   [1] "white"                "aliceblue"            "antiquewhite"        
##   [4] "antiquewhite1"        "antiquewhite2"        "antiquewhite3"       
##   [7] "antiquewhite4"        "aquamarine"           "aquamarine1"         
##  [10] "aquamarine2"          "aquamarine3"          "aquamarine4"         
##  [13] "azure"                "azure1"               "azure2"              
##  [16] "azure3"               "azure4"               "beige"               
##  [19] "bisque"               "bisque1"              "bisque2"             
##  [22] "bisque3"              "bisque4"              "black"               
##  [25] "blanchedalmond"       "blue"                 "blue1"               
##  [28] "blue2"                "blue3"                "blue4"               
##  [31] "blueviolet"           "brown"                "brown1"              
##  [34] "brown2"               "brown3"               "brown4"              
##  [37] "burlywood"            "burlywood1"           "burlywood2"          
##  [40] "burlywood3"           "burlywood4"           "cadetblue"           
##  [43] "cadetblue1"           "cadetblue2"           "cadetblue3"          
##  [46] "cadetblue4"           "chartreuse"           "chartreuse1"         
##  [49] "chartreuse2"          "chartreuse3"          "chartreuse4"         
##  [52] "chocolate"            "chocolate1"           "chocolate2"          
##  [55] "chocolate3"           "chocolate4"           "coral"               
##  [58] "coral1"               "coral2"               "coral3"              
##  [61] "coral4"               "cornflowerblue"       "cornsilk"            
##  [64] "cornsilk1"            "cornsilk2"            "cornsilk3"           
##  [67] "cornsilk4"            "cyan"                 "cyan1"               
##  [70] "cyan2"                "cyan3"                "cyan4"               
##  [73] "darkblue"             "darkcyan"             "darkgoldenrod"       
##  [76] "darkgoldenrod1"       "darkgoldenrod2"       "darkgoldenrod3"      
##  [79] "darkgoldenrod4"       "darkgray"             "darkgreen"           
##  [82] "darkgrey"             "darkkhaki"            "darkmagenta"         
##  [85] "darkolivegreen"       "darkolivegreen1"      "darkolivegreen2"     
##  [88] "darkolivegreen3"      "darkolivegreen4"      "darkorange"          
##  [91] "darkorange1"          "darkorange2"          "darkorange3"         
##  [94] "darkorange4"          "darkorchid"           "darkorchid1"         
##  [97] "darkorchid2"          "darkorchid3"          "darkorchid4"         
## [100] "darkred"              "darksalmon"           "darkseagreen"        
## [103] "darkseagreen1"        "darkseagreen2"        "darkseagreen3"       
## [106] "darkseagreen4"        "darkslateblue"        "darkslategray"       
## [109] "darkslategray1"       "darkslategray2"       "darkslategray3"      
## [112] "darkslategray4"       "darkslategrey"        "darkturquoise"       
## [115] "darkviolet"           "deeppink"             "deeppink1"           
## [118] "deeppink2"            "deeppink3"            "deeppink4"           
## [121] "deepskyblue"          "deepskyblue1"         "deepskyblue2"        
## [124] "deepskyblue3"         "deepskyblue4"         "dimgray"             
## [127] "dimgrey"              "dodgerblue"           "dodgerblue1"         
## [130] "dodgerblue2"          "dodgerblue3"          "dodgerblue4"         
## [133] "firebrick"            "firebrick1"           "firebrick2"          
## [136] "firebrick3"           "firebrick4"           "floralwhite"         
## [139] "forestgreen"          "gainsboro"            "ghostwhite"          
## [142] "gold"                 "gold1"                "gold2"               
## [145] "gold3"                "gold4"                "goldenrod"           
## [148] "goldenrod1"           "goldenrod2"           "goldenrod3"          
## [151] "goldenrod4"           "gray"                 "gray0"               
## [154] "gray1"                "gray2"                "gray3"               
## [157] "gray4"                "gray5"                "gray6"               
## [160] "gray7"                "gray8"                "gray9"               
## [163] "gray10"               "gray11"               "gray12"              
## [166] "gray13"               "gray14"               "gray15"              
## [169] "gray16"               "gray17"               "gray18"              
## [172] "gray19"               "gray20"               "gray21"              
## [175] "gray22"               "gray23"               "gray24"              
## [178] "gray25"               "gray26"               "gray27"              
## [181] "gray28"               "gray29"               "gray30"              
## [184] "gray31"               "gray32"               "gray33"              
## [187] "gray34"               "gray35"               "gray36"              
## [190] "gray37"               "gray38"               "gray39"              
## [193] "gray40"               "gray41"               "gray42"              
## [196] "gray43"               "gray44"               "gray45"              
## [199] "gray46"               "gray47"               "gray48"              
## [202] "gray49"               "gray50"               "gray51"              
## [205] "gray52"               "gray53"               "gray54"              
## [208] "gray55"               "gray56"               "gray57"              
## [211] "gray58"               "gray59"               "gray60"              
## [214] "gray61"               "gray62"               "gray63"              
## [217] "gray64"               "gray65"               "gray66"              
## [220] "gray67"               "gray68"               "gray69"              
## [223] "gray70"               "gray71"               "gray72"              
## [226] "gray73"               "gray74"               "gray75"              
## [229] "gray76"               "gray77"               "gray78"              
## [232] "gray79"               "gray80"               "gray81"              
## [235] "gray82"               "gray83"               "gray84"              
## [238] "gray85"               "gray86"               "gray87"              
## [241] "gray88"               "gray89"               "gray90"              
## [244] "gray91"               "gray92"               "gray93"              
## [247] "gray94"               "gray95"               "gray96"              
## [250] "gray97"               "gray98"               "gray99"              
## [253] "gray100"              "green"                "green1"              
## [256] "green2"               "green3"               "green4"              
## [259] "greenyellow"          "grey"                 "grey0"               
## [262] "grey1"                "grey2"                "grey3"               
## [265] "grey4"                "grey5"                "grey6"               
## [268] "grey7"                "grey8"                "grey9"               
## [271] "grey10"               "grey11"               "grey12"              
## [274] "grey13"               "grey14"               "grey15"              
## [277] "grey16"               "grey17"               "grey18"              
## [280] "grey19"               "grey20"               "grey21"              
## [283] "grey22"               "grey23"               "grey24"              
## [286] "grey25"               "grey26"               "grey27"              
## [289] "grey28"               "grey29"               "grey30"              
## [292] "grey31"               "grey32"               "grey33"              
## [295] "grey34"               "grey35"               "grey36"              
## [298] "grey37"               "grey38"               "grey39"              
## [301] "grey40"               "grey41"               "grey42"              
## [304] "grey43"               "grey44"               "grey45"              
## [307] "grey46"               "grey47"               "grey48"              
## [310] "grey49"               "grey50"               "grey51"              
## [313] "grey52"               "grey53"               "grey54"              
## [316] "grey55"               "grey56"               "grey57"              
## [319] "grey58"               "grey59"               "grey60"              
## [322] "grey61"               "grey62"               "grey63"              
## [325] "grey64"               "grey65"               "grey66"              
## [328] "grey67"               "grey68"               "grey69"              
## [331] "grey70"               "grey71"               "grey72"              
## [334] "grey73"               "grey74"               "grey75"              
## [337] "grey76"               "grey77"               "grey78"              
## [340] "grey79"               "grey80"               "grey81"              
## [343] "grey82"               "grey83"               "grey84"              
## [346] "grey85"               "grey86"               "grey87"              
## [349] "grey88"               "grey89"               "grey90"              
## [352] "grey91"               "grey92"               "grey93"              
## [355] "grey94"               "grey95"               "grey96"              
## [358] "grey97"               "grey98"               "grey99"              
## [361] "grey100"              "honeydew"             "honeydew1"           
## [364] "honeydew2"            "honeydew3"            "honeydew4"           
## [367] "hotpink"              "hotpink1"             "hotpink2"            
## [370] "hotpink3"             "hotpink4"             "indianred"           
## [373] "indianred1"           "indianred2"           "indianred3"          
## [376] "indianred4"           "ivory"                "ivory1"              
## [379] "ivory2"               "ivory3"               "ivory4"              
## [382] "khaki"                "khaki1"               "khaki2"              
## [385] "khaki3"               "khaki4"               "lavender"            
## [388] "lavenderblush"        "lavenderblush1"       "lavenderblush2"      
## [391] "lavenderblush3"       "lavenderblush4"       "lawngreen"           
## [394] "lemonchiffon"         "lemonchiffon1"        "lemonchiffon2"       
## [397] "lemonchiffon3"        "lemonchiffon4"        "lightblue"           
## [400] "lightblue1"           "lightblue2"           "lightblue3"          
## [403] "lightblue4"           "lightcoral"           "lightcyan"           
## [406] "lightcyan1"           "lightcyan2"           "lightcyan3"          
## [409] "lightcyan4"           "lightgoldenrod"       "lightgoldenrod1"     
## [412] "lightgoldenrod2"      "lightgoldenrod3"      "lightgoldenrod4"     
## [415] "lightgoldenrodyellow" "lightgray"            "lightgreen"          
## [418] "lightgrey"            "lightpink"            "lightpink1"          
## [421] "lightpink2"           "lightpink3"           "lightpink4"          
## [424] "lightsalmon"          "lightsalmon1"         "lightsalmon2"        
## [427] "lightsalmon3"         "lightsalmon4"         "lightseagreen"       
## [430] "lightskyblue"         "lightskyblue1"        "lightskyblue2"       
## [433] "lightskyblue3"        "lightskyblue4"        "lightslateblue"      
## [436] "lightslategray"       "lightslategrey"       "lightsteelblue"      
## [439] "lightsteelblue1"      "lightsteelblue2"      "lightsteelblue3"     
## [442] "lightsteelblue4"      "lightyellow"          "lightyellow1"        
## [445] "lightyellow2"         "lightyellow3"         "lightyellow4"        
## [448] "limegreen"            "linen"                "magenta"             
## [451] "magenta1"             "magenta2"             "magenta3"            
## [454] "magenta4"             "maroon"               "maroon1"             
## [457] "maroon2"              "maroon3"              "maroon4"             
## [460] "mediumaquamarine"     "mediumblue"           "mediumorchid"        
## [463] "mediumorchid1"        "mediumorchid2"        "mediumorchid3"       
## [466] "mediumorchid4"        "mediumpurple"         "mediumpurple1"       
## [469] "mediumpurple2"        "mediumpurple3"        "mediumpurple4"       
## [472] "mediumseagreen"       "mediumslateblue"      "mediumspringgreen"   
## [475] "mediumturquoise"      "mediumvioletred"      "midnightblue"        
## [478] "mintcream"            "mistyrose"            "mistyrose1"          
## [481] "mistyrose2"           "mistyrose3"           "mistyrose4"          
## [484] "moccasin"             "navajowhite"          "navajowhite1"        
## [487] "navajowhite2"         "navajowhite3"         "navajowhite4"        
## [490] "navy"                 "navyblue"             "oldlace"             
## [493] "olivedrab"            "olivedrab1"           "olivedrab2"          
## [496] "olivedrab3"           "olivedrab4"           "orange"              
## [499] "orange1"              "orange2"              "orange3"             
## [502] "orange4"              "orangered"            "orangered1"          
## [505] "orangered2"           "orangered3"           "orangered4"          
## [508] "orchid"               "orchid1"              "orchid2"             
## [511] "orchid3"              "orchid4"              "palegoldenrod"       
## [514] "palegreen"            "palegreen1"           "palegreen2"          
## [517] "palegreen3"           "palegreen4"           "paleturquoise"       
## [520] "paleturquoise1"       "paleturquoise2"       "paleturquoise3"      
## [523] "paleturquoise4"       "palevioletred"        "palevioletred1"      
## [526] "palevioletred2"       "palevioletred3"       "palevioletred4"      
## [529] "papayawhip"           "peachpuff"            "peachpuff1"          
## [532] "peachpuff2"           "peachpuff3"           "peachpuff4"          
## [535] "peru"                 "pink"                 "pink1"               
## [538] "pink2"                "pink3"                "pink4"               
## [541] "plum"                 "plum1"                "plum2"               
## [544] "plum3"                "plum4"                "powderblue"          
## [547] "purple"               "purple1"              "purple2"             
## [550] "purple3"              "purple4"              "red"                 
## [553] "red1"                 "red2"                 "red3"                
## [556] "red4"                 "rosybrown"            "rosybrown1"          
## [559] "rosybrown2"           "rosybrown3"           "rosybrown4"          
## [562] "royalblue"            "royalblue1"           "royalblue2"          
## [565] "royalblue3"           "royalblue4"           "saddlebrown"         
## [568] "salmon"               "salmon1"              "salmon2"             
## [571] "salmon3"              "salmon4"              "sandybrown"          
## [574] "seagreen"             "seagreen1"            "seagreen2"           
## [577] "seagreen3"            "seagreen4"            "seashell"            
## [580] "seashell1"            "seashell2"            "seashell3"           
## [583] "seashell4"            "sienna"               "sienna1"             
## [586] "sienna2"              "sienna3"              "sienna4"             
## [589] "skyblue"              "skyblue1"             "skyblue2"            
## [592] "skyblue3"             "skyblue4"             "slateblue"           
## [595] "slateblue1"           "slateblue2"           "slateblue3"          
## [598] "slateblue4"           "slategray"            "slategray1"          
## [601] "slategray2"           "slategray3"           "slategray4"          
## [604] "slategrey"            "snow"                 "snow1"               
## [607] "snow2"                "snow3"                "snow4"               
## [610] "springgreen"          "springgreen1"         "springgreen2"        
## [613] "springgreen3"         "springgreen4"         "steelblue"           
## [616] "steelblue1"           "steelblue2"           "steelblue3"          
## [619] "steelblue4"           "tan"                  "tan1"                
## [622] "tan2"                 "tan3"                 "tan4"                
## [625] "thistle"              "thistle1"             "thistle2"            
## [628] "thistle3"             "thistle4"             "tomato"              
## [631] "tomato1"              "tomato2"              "tomato3"             
## [634] "tomato4"              "turquoise"            "turquoise1"          
## [637] "turquoise2"           "turquoise3"           "turquoise4"          
## [640] "violet"               "violetred"            "violetred1"          
## [643] "violetred2"           "violetred3"           "violetred4"          
## [646] "wheat"                "wheat1"               "wheat2"              
## [649] "wheat3"               "wheat4"               "whitesmoke"          
## [652] "yellow"               "yellow1"              "yellow2"             
## [655] "yellow3"              "yellow4"              "yellowgreen"
```

## Saving plots to file

Of course we also want to save our plots as images so we can put them in articles, websites, presentations, and posters. One way of saving a plot is via the **Export** button in RStudio's **Plots** tab. This is good for saving a 'one-off' plot that we aren't likely to want to come back to and modify. But if we want to make the creation of the image reproducible, then we should include it as a command in our R script. This allows us to come back and just run our entire analysis again, maybe with new data, and get the new plot image automatically.

The `ggsave()` function saves a plot to an image file. The first input is the name we want to give the file, and the second input is the plot object. We use the suffix of the filename to specify what image format we want.

Here we save one of the plots already created above, as a png file:


```r
ggsave('figure_2.png', fig2)
```

```
## Saving 7 x 5 in image
```

Again, a warning message informs us of the function's default behavior. In this case it refers to the dimensions of the image. If we want to change the width and height of the image, we have to specify the `width`and `height` options in the input to `ggsave()`.

The units of the image dimensions are by default inches (abbreviated to 'in' in the warning message above). Small values in the range of 2 or 3 will give a small, chunky image in which the text and objects are large relative to the overall plot size. Values much larger than 10 will give a more sparse-looking plot, in which text and points are relatively small.


```r
ggsave('figure_2.png', fig2, width=3, height=2)
```

If we want a very large-format image, for example to go on a big poster, then a scalable format such as svg is best. Instead of being stored as pixels, which will get fuzzy when the image is zoomed up to a large size, an svg image is stored as a description of lines and shapes, so it stays crisp at whatever size it is zoomed up to.

Unfortunately, not every operating system can create these files from within R. If you type the command `capabilities('cairo')` into your console and you get the answer `TRUE`, then you are probably good to go. Try saving an svg image and see whether it works.


```r
ggsave('figure_2.svg', fig2)
```

```
## Saving 7 x 5 in image
```

If R prints out an error message saying that some packages were not found (possibly the `svglite` package), you could try installing those packages and it may then work.

## Faceting

If we want to show the same plot for different subsets of our data, we don't need to chop up our data and then create the plot separately for each part. ggplot's faceting functions add a 'split' to a plot, creating it separately for different groups.

The input to these functions is a formula describing the split. We have seen formulae already when we used them to specify subsets of data for `aggregate()`. We will stick here to the basic case in which we want to split by just one grouping variable. In this case, we just need the `~` that appears in all formulae, followed by the name of the grouping variable we want to split by.

The `facet_wrap()` function handles this simplest case.

For example, if we wanted to show the two smoking groups side by side instead of as different colors on the same plot, the full plot command would look like this:


```r
ggplot(bw, aes(y=Birth_weight,x=Weight)) + geom_point() + facet_wrap(~Smoker)
```

![](ggplot_files/figure-html/unnamed-chunk-21-1.png)<!-- -->

As with all other plot features, a facet can be added to an existing plot. For example, we could take the plot we created earlier, and show it separately for each of the three categories of the `Race` variable.


```r
print(fig2 + facet_wrap(~Race))
```

![](ggplot_files/figure-html/unnamed-chunk-22-1.png)<!-- -->

Faceting can be particularly useful if we have data from an experiment in which each participant completed many trials of the experiment. We can define a plot that visualizes a participant's performance (for example a speed-accuracy trade-off), and then facet by the participant ID variable. The separate plots will show us whether all participants behaved according to more or less the same pattern, or whether there is a lot of variability among participants or some anomalous cases.

## Distributions

An important first exploration of our data is to check how a variable is distributed. We have different options for this. A histogram is a good start. The histogram divides a scale up into 'bins', and counts how many observations we have in each bin.

Let's see an example with the distribution of birth weights. We need the birth weights on the *x* axis, and then the histogram as our geometric object,


```r
fig0 = ggplot(bw, aes(x=Birth_weight)) + labs(x='Birth weight (kg)')

fig1 = fig0 + geom_histogram()

print(fig1)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](ggplot_files/figure-html/unnamed-chunk-23-1.png)<!-- -->

Again, we see a short warning message. This time, we are being told about a default choice regarding the width of the bins for the histogram. We can change this by specifying `binwidth` in the input to `geom_histogram()`. The value refers to the scale of measurement. For example to group together birth weights in intervals of half a kilo, we would use 0.5.


```r
fig1 = fig0 + geom_histogram(binwidth=0.5)

print(fig1)
```

![](ggplot_files/figure-html/unnamed-chunk-24-1.png)<!-- -->

If we want instead to specify the total number of bins rather than the width of each one, we can use the `bins` input instead.


```r
fig1 = fig0 + geom_histogram(bins=20)

print(fig1)
```

![](ggplot_files/figure-html/unnamed-chunk-25-1.png)<!-- -->

## Summary statistics

If we want to plot summary statistics, such as a mean value, there are ggplot functions for this too. `stat_summary()` is the most general one. Let's show the mean birth weights side by side for the two smoking groups.


```r
fig0 = ggplot(bw, aes(y=Birth_weight, x=Smoker)) + labs(y='Birth weight (kg)')

fig1 = fig0 + stat_summary()

print(fig1)
```

```
## No summary function supplied, defaulting to `mean_se()
```

![](ggplot_files/figure-html/unnamed-chunk-26-1.png)<!-- -->

Again a warning message tells us about a default option that has been applied. By default, `stat_summary()` shows the mean value and its Standard Error (this is the meaning of the `mean_se()` function mentioned in the warning message).

We can specify our own summary functions, but this becomes a little trickier and is beyond the scope of this tutorial.

To make sure our plot can be interpreted correctly, we should say what statistics the points and ranges of the summary show.


```r
fig1 = fig1 + labs(caption='mean ± SE')

print(fig1)
```

```
## No summary function supplied, defaulting to `mean_se()
```

![](ggplot_files/figure-html/unnamed-chunk-27-1.png)<!-- -->

## A more complex example

Let's try a different kind of plot, and see some slightly trickier tweaks that we may sometimes need to make.

We will produce side-by-side boxplots showing the distribution of birth weights in each of the two smoking categories. In addition, we will overlay the individual observations as points. A plot like this combines two objectives of data visualization. On the one hand we want to give a somewhat compressed summary of the data so we can see an overall pattern (this is what the boxplots will achieve), but we also don't want to lose too much information (this is why we will still show the individual observations).

(When writing a longer command, we can make the text of our analysis program a bit more readable by splitting it over two or more lines. This has no functional effect on our program, it just makes it easier for a human to read, which is important if we want people to be able to see clearly what we did with the data.)


```r
# Just the core definition of the plot, and a better label for the y axis.
fig0 = ggplot(bw, aes(y=Birth_weight,x=Smoker)) +
  labs(y='Birth weight (kg)')

# Add the boxplots.
fig1 = fig0 + geom_boxplot()

print(fig1)
```

![](ggplot_files/figure-html/unnamed-chunk-28-1.png)<!-- -->

The boxplots show quite well the overall tendency for smoking mothers to have smaller babies. Now we will add the points.


```r
fig2 = fig1 + geom_point()

print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-29-1.png)<!-- -->

The points haven't turned out quite so nicely. The problem is that there are too many of them, so they overlap each other. One solution to this is to randomly 'jitter' the points horizontally, so that they are spaced apart from one another a bit.

To change the behavior of how a geom is positioned, we must specify a `position` function in the input to the geom. As is typical in ggplot, functions of the same type all begin with the same word, and positioning functions all begin with `position_`. The one that we want is `position_jitter()`. This function itself needs some input. We must set the `width` and `height`. We definitely don't want to jitter the points vertically, as this would be changing the birth weight values. We only want horizontal jitter. Since we have a categorical variable on the horizontal axis, the units for this axis are as if the distance between the two categories were 1. So a fairly small jitter width like 0.2 will ensure that the jitter does not jitter any points so far out that they move over into the neighboring category.


```r
fig2 = fig1 +
  geom_point(position=position_jitter(width=0.2,height=0))

print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-30-1.png)<!-- -->

Looks better. But we now have a very slight additional problem. Look carefully at the very lowest birth weight, in the smoking group. Our plot has duplicated this observation. This is because it was drawn once in the boxplot (because boxplots show outliers as individual points), and then once again as a point by `geom_point()`.

We need to go back and tell the `geom_boxplot()` function not to show outliers. `geom_boxplot` has an option called `outlier.shape`. If we set the outlier shape to just be an empty piece of text (`''`), it will not be shown.


```r
fig2 = fig0 +
  geom_boxplot(outlier.shape='') +
  geom_point(position=position_jitter(width=0.2,height=0))

print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-31-1.png)<!-- -->

Just to see a few more small subtleties, we will now try also coloring in the boxplots so that the two smoking groups are in different colors. This isn't really necessary, since they are already shown separately, but it can make the image a bit more visually appealing. For this, we need to go back to the definition of the plot and add in an aesthetic mapping to color.

(It is allowed for a variable to be mapped to more than one dimension.)


```r
fig0 = ggplot(bw, aes(y=Birth_weight,x=Smoker,color=Smoker)) +
  labs(y='Birth weight (kg)')

fig1 = fig0 +
  geom_boxplot()

print(fig1)
```

![](ggplot_files/figure-html/unnamed-chunk-32-1.png)<!-- -->

This hasn't worked out as desired. The color dimension in ggplot refers to the color of things that have no area, such as lines and points. So we have colored the outlines of the boxplots. For the filled in color of areas, we need the 'fill' dimension.


```r
fig0 = ggplot(bw, aes(y=Birth_weight,x=Smoker,fill=Smoker)) +
  labs(y='Birth weight (kg)')

fig1 = fig0 +
  geom_boxplot()

print(fig1)
```

![](ggplot_files/figure-html/unnamed-chunk-33-1.png)<!-- -->

That does what we wanted. Now let's try it with the points.


```r
fig2 = fig0 +
  geom_boxplot(outlier.shape='') +
  geom_point(position=position_jitter(width=0.2,height=0))

print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-34-1.png)<!-- -->

It might be nice if the points were filled in with color as well. As noted above, `fill` only applies to things with area. So we need to change the default points into filled circles. We can do this by specifying the `shape` input for `geom_point()`.


```r
fig2 = fig0  +
  geom_boxplot(outlier.shape='') +
  geom_point(shape='circle filled', position=position_jitter(width=0.2,height=0))

print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-35-1.png)<!-- -->

One last detail to take care of.

Since the two smoking groups are labeled on the *x* axis already, and the colors are just for visual pleasantness and clarity, the legend at the right of the plot is a bit redundant and is using up valuable space in the image. The `theme()` function handles overall details like this. We can remove legends by setting the `legend.position` option to be `'none'`.


```r
fig2 = fig2  +
  theme(legend.position='none')

print(fig2)
```

![](ggplot_files/figure-html/unnamed-chunk-36-1.png)<!-- -->
