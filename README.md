# Exploratory Data Analysis on Red Wine Quality

The dataset `wineQualityReds.csv` is available in this directory.

---
title: "Exploratory Data Analysis on Red Wine Quality"
output:
  pdf_document: default
  html_document: default
---

```{r setup, echo=FALSE,include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r, echo=FALSE,message=FALSE, results='hide', warning=FALSE}
#Initial imports
library(ggplot2)
library(dplyr)
library(GGally)
library(gridExtra)
```

# Getting to Know the Database

Viewing the first 6 rows in the dataset to see how the data looks like.

```{r, echo=FALSE,message=FALSE, warning=FALSE}
redwine <- read.csv('../Project/wineQualityReds.csv', sep = ',')
head(redwine)
```

There are 11 chemicals in the table that can influence the quality of red wine. For instance acidity level, sugar, PH level, alcohol, etc. The last column in the table specifies the quality (score between 0 to 10) of the wine based on all the chemicals.

Now let's get some info on different types of data in the dataset. According to the table below, `row number` and `quality` are of type integer, while the rest of the variables are numeric.

```{r, echo=FALSE,message=FALSE, warning=FALSE}
str(redwine)
```

I would like to remove the `X` vector from the dataset as it only holds row numbers in the table which is no use for me during my investigation.

```{r, echo=FALSE,message=FALSE, warning=FALSE}
redwine$X <- NULL
```

***

# Univariate Plots Section

I create some basic histograms for each chemical property in the table. 

First I like to see how the `quality` property is distributed in the table. Quality of red wine is one of the main point of interest for me in this dataset, as I'd like to see what properties in red wine can influence the quality. 

```{r, echo=FALSE,message=FALSE, warning=FALSE}
quality <- ggplot(data = redwine,
                  aes(x=quality)) +
  geom_histogram(colour = I('#741B3E'), fill = I('#741B3E'), binwidth = 0.5) +
  scale_x_continuous(limits = c(0,10), breaks = seq(0,10,1)) +
  xlab('Quality of Red Wine - Worst(0) to Best(10)') +
  ylab('Number of Red Wines')

ggsave('quality_hist.png', height = 4, width = 5, dpi = 600)
```

![Distribution of Red Wines Based on Quality](quality_hist.png)

Based on the plot the majority of the red wines -about more than 1200- have either 5 or 6 ranking for their quality. About 200 red wines have quality ranking as 7, but the number is not as significant as 5 and 6. The rest are either 3,4 or 8.

```{r, echo=FALSE,message=FALSE, warning=FALSE}
summary(redwine$quality)
```

The average quality of red wines in the database is 5.63, with the maximum ranking set to 8 and the minimum set to 3. Therefor, there is no red wine in the database with a quality of 10 or 0.

## Adding a New Column 'Rating' to My DataFrame
Since the quality rating is from 0-10, I like to create a rating system, from 1-10 (as there are no wines with rating zero in the data), and categories these numbers into ratings like this:

1-2 : Poor
3-4 : Below Average
5-6 : Average
7-8 : Above Average
9-10 : Excellent

```{r, echo=FALSE,message=FALSE, warning=FALSE}
redwine$rating <- NA

for (i in 1:nrow(redwine)) {
  if(redwine[i, 'quality'] %in% c(1,2)) {
   redwine[i, 'rating'] <- 'Poor'
  } 
  else if(redwine[i, 'quality'] %in% c(3,4)) {
      redwine[i, 'rating'] <- 'Below Average'
  } 
  else if(redwine[i, 'quality'] %in% c(5,6)) {
      redwine[i, 'rating'] <- 'Average'
  } 
  else if(redwine[i, 'quality'] %in% c(7,8)) {
      redwine[i, 'rating'] <- 'Above Average'
  } 
  else if(redwine[i, 'quality'] %in% c(9,10)) {
      redwine[i, 'rating'] <- 'Excellent'
  }
}
```

The first 20 rows of my dataset for the `rating` column now looks like this:

```{r, echo=FALSE,message=FALSE, warning=FALSE}
head(redwine$rating, 20)
```


After looking at the quality distribution, I'd like to get a histogram of all the chemical properties - as well as quality - in the table. These histograms can also help me in finding out if any of values in the plots need to be changed to e.g. log10 for a smoother distribution.

```{r, echo=FALSE,message=FALSE, warning=FALSE}
get_histogram <- function(var, xlabel) {
  return (qplot(x = var, data = redwine, xlab = xlabel, 
                color= I('#741B3E'), fill = I('#741B3E')))
}

png(height=1300, width=1500, res=300,
    filename='all_properties_hist.png')
grid.arrange(get_histogram(redwine$fixed.acidity, 'Fixed Acidity'),
get_histogram(redwine$volatile.acidity, 'Volatile Acidity'),
get_histogram(redwine$citric.acid, 'Citric Acid'),
get_histogram(redwine$residual.sugar, 'Residual Sugar'),
get_histogram(redwine$chlorides, 'Chlorides'),
get_histogram(redwine$free.sulfur.dioxide, 'Free Sulfur Dioxide'),
get_histogram(redwine$total.sulfur.dioxide, 'Total Sulfur Dioxide'),
get_histogram(redwine$density, 'Density'),
get_histogram(redwine$pH, 'PH'),
get_histogram(redwine$sulphates, 'Sulphates'),
get_histogram(redwine$alcohol, 'Alcohol'),
get_histogram(redwine$quality, 'Quality'),
ncol = 3)
```

![Distribution of Quality and All Chemical Properties](all_properties_hist.png)

## Fixed Acidity, Volatile Acidity and Citric Acid
`Fixed Acidity` distribution has most of its values from 5-12, with a few more than 12, and some outliers around 16. I apply the `log10` function to its x-axis to make the distribution look normal.

```{r,echo=FALSE,message=FALSE, warning=FALSE}
log10_fixed <- ggplot(data = redwine,
                      aes(x=log10(fixed.acidity))) +
  geom_histogram(color = I('#985871'), fill = I('#985871'), binwidth = 0.01) +
  xlab('Fixed Acidity (Log10)') +
  ylab('Count')

normal_fixed <- ggplot(data = redwine,
                      aes(x=(fixed.acidity))) +
  geom_histogram(binwidth = 0.1) +
  xlab('Fixed Acidity (Original)') +
  ylab('Count')

png(height=800, width=1200, res=200,
    filename='fixed_acidity.png')
grid.arrange(log10_fixed, normal_fixed, ncol=1)
```

![Fixed Acidity Level - Comparing Log10 to the Original Distribution](fixed_acidity.png)

`Volatile Acidity` distribution has most of its values between 0 and 0.8, with a few going to 1, and then outliers after 1 to 1.6.

By getting the square root of the values, I can change the distribution towards a normal one. However, the outliers still exist after 1.00

```{r,echo=FALSE,message=FALSE, warning=FALSE}
sqrt_volatile <- ggplot(data = redwine,
                        aes(x=sqrt(volatile.acidity))) +
  geom_histogram(color = I('#985871'), fill = I('#985871'), binwidth = 0.01) +
  xlab('Volatile Acidity (Sqrt)') +
  ylab('Count')

volatile_normal <- ggplot(data = redwine,
                          aes(x=(volatile.acidity))) +
  geom_histogram(binwidth = 0.01) +
  xlab('Volatile Acidity') +
  ylab('Count')

png(height=800, width=1200, res=200,
    filename='volatile_acidity.png')
grid.arrange(sqrt_volatile, volatile_normal, ncol=1)
```

![Volatile Acidity Level - Comparing Square Root to the Original Distribution](volatile_acidity.png)

`Citric Acid` values are distributed along the plot on its x-axis, starting from zero (with its most count) to 0.75. Seems there are outliers for citric acid level 1.00

The plot has a its rise and fall throughout the x-axis. The most count contains citric acid level of zero. There are also two more visible peaks in count around 0.25 and 0.5 values. 

```{r,echo=FALSE,message=FALSE, warning=FALSE}
png(height=600, width=1200, res=200,
    filename='citric_acidity.png')
ggplot(data = redwine,
       aes(x=(citric.acid))) +
  geom_histogram(binwidth = 0.01) +
  xlab('Citric Acid') +
  ylab('Count')
```

![Distribution of Citric Acid Level](citric_acidity.png)

## Residual Sugar
`Residual Sugar` distribution has the majority of the count around 1.5-2.5. There are many outliers around 7 and onwards. 

```{r, echo=FALSE,message=FALSE, warning=FALSE}
png(height=600, width=1200, res=200,
    filename='residual_sugar.png')
ggplot(data = redwine,
       aes(x=residual.sugar)) +
  geom_histogram(binwidth = 0.1) +
  scale_x_continuous(limits = c(0,16), breaks = seq(0,16,1)) +
  xlab('Residual Sugar') +
  ylab('Count')
```

![Distribution of Residual Sugar](residual_sugar.png)

The plot looks like a normal distribution that is skewed to the left.

The outliers can have a direct influence on change the average value for residual sugar. Let's take a look at the summary for residual sugar to find out how the average is influenced.

```{r, echo=FALSE,message=FALSE, warning=FALSE}
summary(redwine$residual.sugar)
```

According to the plot for residual sugar, with most of the values concentrated around 1.5-2.5, I was expecting the mean to fall around this range as well. However, the existance of the outliers was an interesting factor for me to check to see how much they can influence the mean. According to the summary, the mean still falls around 2.5 inspite of the outliers. This can make sense as the density of the plot is more significant in 1.5-2.5 range, than around the outliers.

## Chlorides
`Chlorides` distribution has most of the values concentrated around 0.03-0.15. There are a few outliers especially after 0.35. I would like to check the summary to see how the mean value is influenced by the outliers.

```{r, echo=FALSE,message=FALSE, warning=FALSE}
png(height=600, width=1200, res=200,
    filename='chlorides.png')
ggplot(data = redwine,
       aes(x=chlorides)) +
  geom_histogram(binwidth = 0.002) +
  scale_x_continuous(limits = c(0.0,0.7), breaks = seq(0.0,0.7,0.05)) +
  xlab('Chlorides') +
  ylab('Count')
```

![Distribution of Chlorides](chlorides.png)

The plot shows that the distribution is almost normal and a bit skewed to the left. 

```{r, echo=FALSE,message=FALSE, warning=FALSE}
summary(redwine$chlorides)
```

I expected that the mean value would fall somewhere between 0.05-0.10, and according to the summary, it's correct. The mean value of 0.08 is in my expected range.

## Free Sulfur Diaxide and Total Sulfur Dioxide
`Total Sulfur Dioxide` distribution is skewed to left. The rise and fall of the distribution is not that much; however, there are a few noticable peaks in count around 30-40, 60-80, and 80-90.

I can use `log10` on the x-axis to change the distribution to normal. Let's see how this will look like.

```{r, echo=FALSE,message=FALSE, warning=FALSE}
log10 <- ggplot(data = redwine,
                aes(x=log10(total.sulfur.dioxide))) +
  geom_histogram(color = I('#985871'), fill = I('#985871'), binwidth = 0.04) +
  scale_x_continuous(limits = c(0.0,2.5), breaks = seq(0.0,2.5,0.2)) +
  xlab('Total Sulfur Dioxide (Log10)') +
  ylab('Count')

normal <- ggplot(data = redwine,
                 aes(x=total.sulfur.dioxide)) +
  geom_histogram(binwidth = 1) +
  xlab('Total Sulfur Dioxide (Normal)') +
  scale_x_continuous(limits = c(0, 300), breaks = seq(0,300,20)) +
  ylab('Count')

png(height=800, width=1200, res=200,
    filename='total_sulfur.png')
grid.arrange(log10, normal, ncol=1)
```

![Distribution of Total Sulfur Dioxide - Comparing Original to Log10](total_sulfur.png)

Based on the plots for `Total Sulfur Dioxide`, using `log10` on the x-axis helped the distribution to look normal-ish. There are still a few peaks in counts, but the overall shape of the distribution looks normal.

`Free Sulfur Dioxide` distribution is also skewed to the left, with most of the values concentrated around 5-15. 

Applying `log10` on the x-axis does not work for `Free Sulfur Dioxide`, though. It changes the distribution, but seems the distribution goes more skewed to the right, rather than getting normally distributed.

```{r, echo=FALSE,message=FALSE, warning=FALSE}
log10_free <- ggplot(data = redwine,
                aes(x=log10(free.sulfur.dioxide))) +
  geom_histogram(color = I('#985871'), fill = I('#985871'), binwidth = 0.05) +
  xlab('Free Sulfur Dioxide (Log10)') +
  ylab('Count')

normal_free <- ggplot(data = redwine,
                 aes(x=free.sulfur.dioxide)) +
  geom_histogram(binwidth = 1) +
  scale_x_continuous(limits = c(0, 80), breaks = seq(0,80,10)) +
  xlab('Free Sulfur Dioxide (Normal)') +
  ylab('Count')

png(height=800, width=1200, res=200,
    filename='free_sulfur.png')
grid.arrange(log10_free, normal_free, ncol=1)
```

![Distribution of Free Sulfur Dioxide - Comparing Original to Log10](free_sulfur.png)

## Density
The `Density` distribution looks like a normal distribution. 

```{r,echo=FALSE,message=FALSE, warning=FALSE}
png(height=600, width=1200, res=200,
    filename='density.png')
ggplot(data = redwine,
       aes(x=density)) +
  geom_histogram(binwidth = 0.0001) +
  xlab('Density') +
  ylab('Count')
```

![Density Distribution](density.png)

Getting the summary of the distribution, I have:

```{r,echo=FALSE,message=FALSE, warning=FALSE}
summary(redwine$density)
```

The mean of the distribution is 0.9967 which looks like it's almost in the middle of the distribution. The median is different than mean by 0.0011, which is also almost close to the middle of the distribution.

## PH
The `PH` distribution looks normal, with a few outliers. 

```{r,echo=FALSE,message=FALSE, warning=FALSE}
png(height=600, width=1200, res=200,
    filename='ph.png')
ggplot(data = redwine,
       aes(x=pH)) +
  geom_histogram(binwidth = 0.01) +
  scale_x_continuous(limits = c(2.5, 4.5), breaks = seq(2.5,4.5,0.2)) +
  xlab('PH') +
  ylab('Count')
```

![PH Level Distribution](ph.png)

```{r,echo=FALSE,message=FALSE, warning=FALSE}
summary(redwine$pH)
```

The summary of the distribution shows the mean value falls at 3.31, very close to the median value.

## Sulphates
The `Sulphates` distribution has most of its values concentrated around 0.5-0.7, and a few outliers around 1.6 and 2.0. I also check the summary of the table to see where the mean and median are placed.

```{r,echo=FALSE,message=FALSE, warning=FALSE}
png(height=600, width=1200, res=200,
    filename='sulphates.png')
ggplot(data = redwine,
       aes(x=sulphates)) +
  geom_histogram(binwidth = 0.01) +
  scale_x_continuous(breaks = seq(0.0,2.5,0.2)) +
  xlab('Sulphates') +
  ylab('Count')
```

![Distribution of Sulphate](sulphates.png)

The distribution looks normal-ish and is a bit skewed to the left.

```{r,echo=FALSE,message=FALSE, warning=FALSE}
summary(redwine$sulphates)
```

The mean at 0.65 is within the range where most of the values are. The median at 0.62 looks also close to the mean.

## Alcohol
The `Alcohol` distribution has its falls and rises, with its most values around 9.5% of alcohol. 

```{r,echo=FALSE,message=FALSE, warning=FALSE}
png(height=600, width=1200, res=200,
    filename='alcohol.png')
ggplot(data = redwine,
       aes(x=alcohol)) +
  geom_histogram(binwidth = 0.1) +
  scale_x_continuous(limits = c(7.0,16.0), breaks = seq(7.0,16.0,0.5)) +
  xlab('Alcohol') +
  ylab('Count')
```

![Alcohol Distribution](alcohol.png)

```{r,echo=FALSE,message=FALSE, warning=FALSE}
summary(redwine$alcohol)
```

The summary of the alcohol table shows the mean alcohol percentage is at 10.42, and the median is 10.20. 

## Univariate Analysis
Now, I'll answer a few questions regarding the database and my initial analysis.

### What is the structure of the dataset?
The dataset has 1599 observations and 13 variables. Since I removed column `X`, and added column `rating`, the number of variables stay the same.

From the 13 variables, 11 of them are about chemical properties of red wines, 1 is about how the quality of red wine changes based on these chemical properties, and another one is where the red wines fall in the rating system based  on their quality.

### What is/are the main feature(s) of interest in the dataset?
For me, the two important factors of interest are quality and rating. Further in the analysis I would like to see how the quality changes based on each chemical property.

Also, I would like to see if the alcohol level have any correlations with the quality of red wines.

### What other features in the dataset will help support the investigation into the feature(s) of interest?
For quality and rating, all the chemical properties can play an important role in supprting my investigation.

### Did you create any new variables from existing variables in the dataset?
Yes, I created a new column `rating` which holds a rating system from `Poor` to `Excellent` based on the quality of red wines.

### Of the features you investigated, were there any unusual distributions? Did you perform any operations on the data to tidy, adjust, or change the form of the data? If so, why did you do this?
1. I removed the `X` column since it was only holding the row numbers and was not playing any role in my investigation
2. I added the `rating` column to later categories the red wines in quality based on their ratings
3. Regarding the distributions, for Fixed Acidity, Volatile Acidity, Free Sulfur Diaxide and Total Sulfur Dioxide I applied the log-transformation on them to make the distribution more normal-ish.

***

# Bivariate Plots Section
With the plots in this section, I would like to dig more into seeing how the chemical properties can affect each other, or the quality of the wine.

## An Overview of the Correlations between Variables
For an overview, using `corrplot` function, I want to create a plot that can show me the correlation of chemical properties to each other.

```{r,echo=FALSE, message=FALSE, warning=FALSE}
library(corrplot)
png(height=1500, width=1500, pointsize=30, file="correlations_prperties.png")
M <- cor(redwine[c(1:11, 12)])
corrplot(M, method = 'circle', order ="hclust", 
         tl.col="black", tl.cex = 0.8, tl.offset = 1)
```

![Correlation Plot for Each Chemical Property and Quality of Red Wine](correlations_prperties.png)

Putting aside the correlation of each property with itself(i.e. 1), let's look at the rest of the patterns.

The circle color for correlation between Total Sulfur Dioxide and Free Sulfur Dioxide is towards the darket blue shade which shows a positive correlation.

```{r,echo=FALSE, message=FALSE, warning=FALSE}
cor(redwine$total.sulfur.dioxide, redwine$free.sulfur.dioxide)
```

Citric Acid and Fixed Acidity also have a positive correlation. However, both have a negative correlation with the Volatile Acidity.

The statistical calculations show the same thing.

Correlation between Citric Acid and Fixed Acidity:
```{r,echo=FALSE, message=FALSE, warning=FALSE}
cor(redwine$fixed.acidity, redwine$citric.acid)
```

Correlation between Citric Acid and Volatile Acidity:
```{r,echo=FALSE, message=FALSE, warning=FALSE}
cor(redwine$volatile.acidity, redwine$citric.acid)
```

Correlation between Fixed Acidity and Volatile Acidity:
```{r,echo=FALSE, message=FALSE, warning=FALSE}
cor(redwine$volatile.acidity, redwine$fixed.acidity)
```

Other correlations of interest are :

PH level and Fixed Acidity, as well as PH level and Citric Acid, both having a negative correlations.

Density and Alcohol level also have a negative correlation with each other.

Quality seems to have a positive correlation with Alcohol level.

## Plots

### Fixed Acidity, Volatile Acidity and Citric Acid
Using three plots I display the relationship between each of the acidity levels.

```{r,echo=FALSE, message=FALSE, warning=FALSE}
f_c_a <- ggplot(data = redwine,
                aes(x=log10(fixed.acidity), y=citric.acid)) +
  geom_jitter(alpha=1/3, color=I('#636262')) +
  geom_smooth(method = 'loess', color=I('#CB3C33')) +
  geom_smooth(method = 'lm', color=I('#D8A525'))+
  xlab('Fixed Acidity') +
  ylab('Citric Acid')
ggsave('f_c_a.png', height = 4, width = 5, dpi = 600)

f_v_a_ <- ggplot(data = redwine,
                 aes(x=fixed.acidity, y=volatile.acidity)) +
  geom_jitter(alpha=1/3, color=I('#636262')) +
  geom_smooth(method = 'loess', color=I('#CB3C33')) +
  geom_smooth(method = 'lm', color=I('#D8A525'))+
  xlab('Fixed Acidity') +
  ylab('Volatile Acidity')
ggsave('f_v_a.png', height = 4, width = 5, dpi = 600)

c_v_a <- ggplot(data = redwine,
                aes(x=citric.acid, y=volatile.acidity)) +
  geom_jitter(alpha=1/3, color=I('#636262')) +
  geom_smooth(method = 'loess', color=I('#CB3C33')) +
  geom_smooth(method = 'lm', color=I('#D8A525'))+
  xlab('Citric Acid') +
  ylab('Volatile Acidity')
ggsave('c_v_a.png', height = 4, width = 5, dpi = 600)
```


![Relation between Fixed Acidity and Citric Acid](f_c_a.png)

Fixed Acidity and Citric Acid level seem to have a positive correlation with each other; meaning as the fixed acidity pf red wine increases, the citric acid level also increases.

![Relation between Fixed Acidity and Volatile Acidity](f_v_a.png)

Fixed acidity and volatile acidity have a negative correlation with each other. Higher levels of volatile acidity has lower levels of fixed acidity.

![Relation between Citric Acid and Volatile Acidity](c_v_a.png)

Citric acid and volatile acidity have a negative correlation with each other as well.

### Comparing Acidity and PH Levels

```{r,echo=FALSE, message=FALSE, warning=FALSE}
fixed_ph <- ggplot(data = redwine,
                aes(x=pH, y=log10(fixed.acidity))) +
  geom_jitter(alpha=1/3, color=I('#636262')) +
  geom_smooth(method = 'loess', color=I('#CB3C33')) +
  geom_smooth(method = 'lm', color=I('#D8A525'))+
  xlab('PH') +
  ylab('Fixed Acidity')
ggsave('fixed_acidity_ph.png', height = 6, width = 7, dpi = 700)

volatile_ph <- ggplot(data = redwine,
                aes(x=pH, y=volatile.acidity)) +
  geom_jitter(alpha=1/3, color=I('#636262')) +
  geom_smooth(method = 'loess', color=I('#CB3C33')) +
  geom_smooth(method = 'lm', color=I('#D8A525'))+
  xlab('PH') +
  ylab('Volatile Acidity')
ggsave('volatile_acidity_ph.png', height = 6, width = 7, dpi = 700)

citric_ph <- ggplot(data = redwine,
                aes(x=pH, y=citric.acid)) +
  geom_jitter(alpha=1/3, color=I('#636262')) +
  geom_smooth(method = 'loess', color=I('#CB3C33')) +
  geom_smooth(method = 'lm', color=I('#D8A525'))+
  xlab('PH') +
  ylab('Citric Acid')
ggsave('citric_acid_ph.png', height = 6, width = 7, dpi = 700)

png(height=800, width=1400, res=200,
    pointsize = 10,file="acidity_ph_level.png")
grid.arrange(fixed_ph, volatile_ph, citric_ph, ncol=3)

```

![Comparing Acidity with PH Levels](acidity_ph_level.png)

PH level has a positive correlation with volatile acidity and negative correlations with citric acid and fixed acidity which makes sense. 

The lower the PH level, the higher is its acidic property. High amount of volatile acidity also causes an unpleasant and vinegar-like taste in red wines. Therefor, it makes sense to see that higher level of acidity (i.e. lower PH level) in PH correlates with higher level of volatile acidity.

In contrast, lower level of acidity in PH (i.e. higher PH level) has a negative correlation with citric acid and fixed acidity levels.

### Residual Sugar and Density
Residual sugar and density seem to have a positive correlation with each other. As the density increases, the redisual sugar increases as well.

```{r,echo=FALSE, message=FALSE, warning=FALSE}
sugar_dens <- ggplot(data = redwine,
                aes(x=density, y=residual.sugar)) +
  geom_jitter(alpha=1/2,color=I('#636262')) +
  geom_smooth(method = 'loess', color=I('#CB3C33')) +
  geom_smooth(method = 'lm', color=I('#D8A525'))+
  xlab('Density') +
  ylab('Residual Sugar')
ggsave('sugar_density.png', height = 4, width = 6, dpi = 600)
```

![Relation between Residual Sugar and Density](sugar_density.png)

If I also calculate the numeric value of the correlation, it's 0.355. This correlation falls within the range that is not very significant.

### Alcohol Level and Density
Density has a negative correlation with the amount of alcohol in red wine, meaning as the density of red wine increases, its amount of alcohol decreases.

```{r,echo=FALSE, message=FALSE, warning=FALSE}
alcohol_dens <- ggplot(data = redwine,
                aes(x=density, y=alcohol)) +
  geom_jitter(alpha=1/2,color=I('#636262')) +
  geom_smooth(method = 'loess', color=I('#CB3C33')) +
  geom_smooth(method = 'lm', color=I('#D8A525'))+
  xlab('Density') +
  ylab('Alcohol')
ggsave('alcohol_density.png', height = 4, width = 6, dpi = 600)
```

![Relation between Alcohol Level and Density](alcohol_density.png)

### Comparing Quality with PH, Alcohol, Volatile Acidity Level

```{r,echo=FALSE, message=FALSE, warning=FALSE}
quality_ph <- ggplot(data = redwine,
                aes(x=as.factor(quality), y=pH)) +
  geom_boxplot(aes(fill=I('#D8A525'))) +
  stat_summary(fun.y = median) +
  coord_cartesian(ylim = c(3.1,3.6)) +
  xlab('Quality') +
  ylab('PH')
ggsave('quality_ph.png', height = 4, width = 6, dpi = 600)
```

![Quality of Red Wine Based on Its PH Level](quality_ph.png)

I expected to see a wider range for quality 5 and 6 in comparison to other categories, but according to the quality vs. ph plot, it looks like the range of PH level is more or less the same for each category in quality. 

The highest median is for quality 3, and the lowest is for quality 8. I expected to see higher PH level for wine quality 8, which means its less acidic, but seems that's not the case.

I calculate the correlation between quality and ph level to double-check my observation from the plot. Seems quality and ph levels are hardly correlated.
```{r,echo=FALSE, message=FALSE, warning=FALSE}
cor(redwine$pH, redwine$quality)
```

```{r,echo=FALSE, message=FALSE, warning=FALSE}
quality_alcohol <- ggplot(data = redwine,
                aes(x=as.factor(quality), y=alcohol)) +
  geom_boxplot(aes(fill=I('#D8A525'))) +
  stat_summary(fun.y = median) +
  coord_cartesian(ylim = c(9,13)) +
  xlab('Quality') +
  ylab('Alcohol')
ggsave('quality_volatile.png', height = 4, width = 6, dpi = 600)
```

![Quality of Red Wine Based on Its Alcohol Level](quality_alcohol.png)

The highest median for alcohol belongs to a range with quality 8. This supports the positive correlation between wine quality and the level of alcohol. 

Quality level 8 also seems to have the widest range among all other categories.

```{r,echo=FALSE, message=FALSE, warning=FALSE}
quality_volatile <- ggplot(data = redwine,
                aes(x=as.factor(quality), y=volatile.acidity)) +
  geom_boxplot(aes(fill=I('#D8A525'))) +
  stat_summary(fun.y = median) +
  coord_cartesian(ylim = c(0.2,1.1)) +
  xlab('Quality') +
  ylab('Volatile Acidity')
ggsave('quality_volatile.png', height = 4, width = 6, dpi = 600)
```

![Quality of Red Wine Based on Its Volatile Acidity Level](quality_volatile.png)

The lowest median for volatile acidity belongs to the highest quality of red wine, which makes sense since the lower the volatile acidity level, the higher will be the quality of wine.

The range of wines with quality 3 or 4 have the widest range of volatile acidity levels.

***

## Bivariate Analysis
I sum up the bivariate section by answering a few more questions about the dataset.

### Talk about some of the relationships observed in this part of the investigation. How did the feature(s) of interest vary with other features in the dataset?
Quality of red wine correlates strongly with the amount of alcohol, and correlates negatively with the volatile acidity level.

PH levels correlate strongly to the citric, fixed and volatile acidity levels. For the unpleasant acidity level (i.e. volatile acidity) the PH level is low, while for citric and fixed acidity level (i.e. plesant acidity properties) PH level stays higher.

I expected to see PH levels correlating with the quality of wine, but that was not the case. They were hardly correlated. 

### Did you observe any interesting relationships between the other features (not the main feature(s) of interest)?
I observed that the amount of density is correlated negatively with the amount of alcohol; the more the density, the less the alcohol level. Also, density has a positive correlation with residual sugar level.

### What was the strongest relationship you found?
From the variables analyzed, the strongest relationship was between Citric Acid and Fixed Acidity, which had a correlation coefficient of 0.67

Also, PH level and Fixed Acidity had a strong negative correlation of -0.68 with each other

