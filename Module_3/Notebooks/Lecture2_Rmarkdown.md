Module 3: Statistics - Lecture 2
================

Install and load NHANES along with the pacakges 'tidyr', 'dplyr':

(To famiiarize yourself with the pacakges you can take a look at the cheatsheets RStudio has provided to you: <https://www.rstudio.com/resources/cheatsheets/>)

``` r
#install.packages('NAHNES')
#install.packages("tidyr")
#install.packages("dplyr")
library(NHANES)
```

    ## Warning: package 'NHANES' was built under R version 3.3.3

``` r
library(tidyr)
```

    ## Warning: package 'tidyr' was built under R version 3.3.3

``` r
library(dplyr)
```

    ## Warning: package 'dplyr' was built under R version 3.3.3

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
data('NHANES')
```

Add a new chunk by clicking the *Insert Chunk* button on the toolbar or by pressing *Ctrl+Alt+I*.

Pick a categorical variable and a numerical varaibel and reduce the data table to just those two. (Education and BMI)

``` r
mydata <- select(NHANES, one_of(c('Education', 'BMI')))
View(mydata)
```

Remove the NAs in the data frame

``` r
mydata <- filter(mydata, !is.na(Education) & !is.na(BMI))
```

    ## Warning: package 'bindrcpp' was built under R version 3.3.3

``` r
View(mydata)
unique(mydata$Education)
```

    ## [1] High School    Some College   College Grad   9 - 11th Grade
    ## [5] 8th Grade     
    ## 5 Levels: 8th Grade 9 - 11th Grade High School ... College Grad

Only focus on two of the education parameters

``` r
unique(mydata$Education)
```

    ## [1] High School    Some College   College Grad   9 - 11th Grade
    ## [5] 8th Grade     
    ## 5 Levels: 8th Grade 9 - 11th Grade High School ... College Grad

``` r
mydata <- filter(mydata, Education %in% c('Some College', 'High School'))
View(mydata)
```

Calcualte the mean for each group

``` r
mydata <- group_by(mydata, Education)
groupMeans <- summarize(mydata, statistic = mean(BMI))
groupMeans
```

    ## # A tibble: 2 x 2
    ##   Education    statistic
    ##   <fctr>           <dbl>
    ## 1 High School       29.4
    ## 2 Some College      29.2

Now calculate the difference between our statistics

``` r
# groupDiffs <- summarize(groupMeans, dif = diff(mn)) 
#groupDiffs <- summarize(groupMeans, dif = diff(mn))
#groupDiffs
#real.diff <- groupDiffs$dif

# above is what we did in class but below is a more flexible option:
#real.diff <- groupMeans$statistic[2] - groupMeans$statistic[1]

# to be really explicit about what difference we're interested in we can spcifiy what variable we want to subtract from what
real.diff <- groupMeans$statistic[which(groupMeans$Education == 'Some College')] -   groupMeans$statistic[which(groupMeans$Education == 'High School')]

real.diff
```

    ## [1] -0.2404631

Want to sample rows. Pay attention to the 'replace' parameter. If 'repalce = TRUE' the same row can be picked again. If 'repalce = FALSE' the same row can only be picked once. Sampling is a random function

``` r
sample(11:15, 5, replace = F)
```

    ## [1] 12 11 13 14 15

``` r
sample(11:15, 5, replace = F)
```

    ## [1] 15 12 14 13 11

To scramble the data, create a copy of the data, then scramble the Education column. This breaks any potential association.

``` r
scrambled.data <- mydata
scrambled.data$Education <- sample(scrambled.data$Education, length(scrambled.data$Education), replace = F)
View(scrambled.data)
```

Now calculate the (null) statistic. Because we want to do this repeately we will write this in a loop which repeats. This will take about 30sec. If you want, 1000 scrambles will also show the same pattern.

``` r
nr.scrambles <- 10000 # perform 10000 scrambling operations
null.diff <- vector('numeric', length = nr.scrambles)
for(i in 1:nr.scrambles){
  scrambled.data <- mydata
  scrambled.data$Education <- sample(scrambled.data$Education, length(scrambled.data$Education), replace = F)
  my.scrambled.data <- group_by(scrambled.data, Education)
  scrambled.groupMeans <- summarize(my.scrambled.data, statistic = mean(BMI))
  null.diff[i] <- scrambled.groupMeans$statistic[which(scrambled.groupMeans$Education == 'Some College')] - scrambled.groupMeans$statistic[which(scrambled.groupMeans$Education == 'High School')]

}
```

Note: if you prefer to do the above in a function (note this code was not actually run in the notebook, varaibels from above were used for plotting)

``` r
# assumption: the two columns have to be 'Education' and 'BMI'
scramble_function <- function(input.data){

  scrambled.data <- input.data
  scrambled.data$Education <- sample(scrambled.data$Education, length(scrambled.data$Education), replace = F)
  my.scrambled.data <- group_by(scrambled.data, Education)
  scrambled.groupMeans <- summarize(my.scrambled.data, statistic = mean(BMI))
  null.diff <- scrambled.groupMeans$statistic[[which(scrambled.groupMeans$Education == 'Some College')]] - scrambled.groupMeans$statistic[[which(scrambled.groupMeans$Education == 'High School')]]
  
  return(null.diff)
}

# I did not actually run the function, this is just as and example
# as demonstared in class, the function 'replicate' can be used
null.distribution <- replicate(1000, scramble_function(mydata))
```

Take a quick look at the null distribution:

``` r
head(null.distribution)
```

    ## [1]  0.5026557 -0.3727448 -0.4197144  0.5600530 -0.1531165 -0.5140425

Now take a look at what the null distribution of our statistic looks like

``` r
hist(null.diff, breaks = 50, main = 'Null distribution')
```

![](Lecture2_Rmarkdown_files/figure-markdown_github/unnamed-chunk-12-1.png)

This looks like a normal distribution and we can now compare where our actual value is compared to this

``` r
hist(null.diff, breaks = 50, main = 'Null distribution')
abline(v = real.diff, col = 'red') # creates the vertical line in red
```

![](Lecture2_Rmarkdown_files/figure-markdown_github/unnamed-chunk-13-1.png)

Where did we expect the statisitc to be? Is this value likely? We didn't expect the statistic to be either positive or negative. This means we can look to either side and ask what the probability is to see a value as exteme as real.diff or more extreme.

``` r
range.to.null <- abs(mean(null.diff) - real.diff)
more.extreme <- (null.diff < (mean(null.diff) - range.to.null) | null.diff > (mean(null.diff) + range.to.null))
head(more.extreme)
```

    ## [1]  TRUE FALSE  TRUE FALSE FALSE FALSE

To get the fraction of TRUEs we can take advantage of the fact that R encodes TRUE as 1 and FALSE as 0. This will give us our p-vaue. Note: if you run the scrambling process again, this p-value will change a bit. This is due to the random process of the scrambling.

``` r
# sum(more.extreme == TRUE) / length(more.extreme), or you can be explicit wiht the matching to TRUE
sum(more.extreme) / length(more.extreme)
```

    ## [1] 0.311

Estimation / confidence intervals This time we don't want to break the association so instead we pick entire rows Becasue we want to be able to select the same row multiple times we will pick with replacement

``` r
conf_int <- function(input.data){
  #sudo.pop <- input.data[sample(c(1:nrow(input.data)), 10000, replace = T),]
  # same column headers
  boot.data <- sample_n(input.data, nrow(input.data), replace = T)
  
  # calculate our statistic:
  my.boot.data <- group_by(boot.data, Education)
  boot.groupMeans <- summarize(my.boot.data, statistic = mean(BMI))
  boot.diff <- boot.groupMeans$statistic[which(boot.groupMeans$Education == 'Some College')] - boot.groupMeans$statistic[which(boot.groupMeans$Education == 'High School')]

  return(boot.diff)
}

boot.dist <- replicate(1000, conf_int(mydata))
```

Now let us take a qucik look at it

``` r
hist(boot.dist, breaks = 50, main = 'Bootstrap distribution')
```

![](Lecture2_Rmarkdown_files/figure-markdown_github/unnamed-chunk-17-1.png)

We can now use Rs 'quantile' function to see from where to where exactly a 95% confidence interval ranges:

``` r
conf.int95 <- quantile(boot.dist, c(0.025, 0.975))
conf.int95
```

    ##        2.5%       97.5% 
    ## -0.54799726  0.09194516

``` r
hist(boot.dist, breaks = 50, main = 'Bootstrap distribution')
abline(v = conf.int95[1], col = 'red')
abline(v = conf.int95[2], col = 'red')
```

![](Lecture2_Rmarkdown_files/figure-markdown_github/unnamed-chunk-18-1.png)
