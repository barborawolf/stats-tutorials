# Exercises

For each exercise, write a short R script that carries out the tasks described. Make sure to test your script and check that it runs as a coherent whole, printing out the required output.

## A

1. Load the **fat.csv** data set from the class website. You can find the file at: [https://raw.githubusercontent.com/luketudge/stats-tutorials/master/tutorials/data/fat.csv]

See the tutorial on [files](Files.html) for a reminder of how to load a .csv file from the internet.

These data show the waist size, weight, and proportion body fat of a group of people.

2. Print out the first six rows of the data.

The variables have clearly been recorded in metric units (*cm* for the waist sizes, and *kg* for the weights). We would like to have them in imperial units as well (*inches* and *pounds*, respectively).

3. Create a new column in the dataframe called `Waist_In` giving the waist size in inches instead. Likewise, create a column giving the weight in pounds.

You can use the following approximate conversion formulas:

$$
inches = cm \over 2.54
$$

$$
pounds = kg \times 2.205
$$

4. Print out a summary of the altered data set so you can compare the two new variables to the original ones.

## B

1. Load the **wines.csv** data set from the class website: [https://raw.githubusercontent.com/luketudge/stats-tutorials/master/tutorials/data/wine.csv]

These data give ratings for many features of several wines tasted at a wine tasting event.

2. Print out the names of all the columns in the data.

3. What were the minimum and maximum `Fruity` ratings?

4. What 'labels' of wine were tasted and how many of each label?

