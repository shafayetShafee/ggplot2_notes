stat summary of Ggplot2
================

-   <a href="#demystifying-stat_-layers-in-ggplot2"
    id="toc-demystifying-stat_-layers-in-ggplot2">Demystifying stat_ layers
    in {ggplot2}</a>
    -   <a href="#some-usecases-of-stat_summary"
        id="toc-some-usecases-of-stat_summary">Some Usecases of stat_summary</a>

> **DISCLAIMER:** This note is based on (mostly copy pasted from) these
> sources \[1\]
> [demystifying-stat-layers-ggplot2](https://yjunechoe.github.io/posts/2020-09-26-demystifying-stat-layers-ggplot2/)

## Demystifying stat\_ layers in {ggplot2}

`stat_summary` works in the following order:

1.  The data that is passed into `ggplot()` is inherited to
    `stat_summary` if one is not provided

2.  The function passed into the `fun.data` argument applies
    transformations to a part of data (that was inherited/provided).
    `fun.data` defaults to `mean_se` function.

3.  The result is then passed into `geom` provided in the `geom`
    argument of the `stat_summary` (`geom` defaults to `pointrange` if
    not specified).

4.  If the transformed data contains all the required mappings for the
    geom, then geom will be printed.

-   **`stat_summary` summarizes one dimension of the data.**

``` r
library(dplyr)
library(ggplot2)
library(tibble)
```

``` r
height_df <- tibble(group = "A",
                    height = rnorm(30, 170, 10))

height_df %>% 
  ggplot(aes(x = group, y = height)) +
  stat_summary()
```

    ## No summary function supplied, defaulting to `mean_se()`

![](understanding_stat_summary_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

So as the points mentioned above, `geom = pointrange` and
`fun.data = mean_se` are used here.

To verify this, we can actually look into the transformed data by
`mean_se` and by `stat_summary`.

The `mean_se` function internally looks like,

``` r
mean_se
```

    ## function (x, mult = 1) 
    ## {
    ##     x <- stats::na.omit(x)
    ##     se <- mult * sqrt(stats::var(x)/length(x))
    ##     mean <- mean(x)
    ##     new_data_frame(list(y = mean, ymin = mean - se, ymax = mean + 
    ##         se), n = 1)
    ## }
    ## <bytecode: 0x00000290149b0168>
    ## <environment: namespace:ggplot2>

``` r
mean_se(height_df$height)
```

    ##          y     ymin     ymax
    ## 1 170.5571 169.0382 172.0759

``` r
point_range_plot <- height_df %>% 
  ggplot(aes(group, height)) + 
  stat_summary()

layer_data(point_range_plot, 1) # here 1 means 1st layer which is pointrange here,
```

    ## No summary function supplied, defaulting to `mean_se()`

    ##   x group        y     ymin     ymax PANEL flipped_aes colour size linetype
    ## 1 1     1 170.5571 169.0382 172.0759     1       FALSE  black  0.5        1
    ##   shape fill alpha stroke
    ## 1    19   NA    NA      1

``` r
# but since here only one geom is used, so if we omit 1, consequence will be same.
```

which is similar as above, so it is proved that `stat_summary` using
`mean_se` as default.

### Some Usecases of stat_summary

#### Error bar showing 95% confidence Interval

``` r
data(penguins, package = "palmerpenguins")

peng <- na.omit(penguins)
```

``` r
peng %>% 
  ggplot(aes(sex, body_mass_g)) +
  stat_summary(
    fun.data = ~mean_se(.x, mult = 1.96),
    geom = "errorbar"
  )
```

![](understanding_stat_summary_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

#### A color coded bar plot of median

Here we want to color a bar of medians if median for a specific bar is
less than a threshold (say 40).

``` r
# custom function for fun.data

calc_median_and_color <- function(x, threshold = 40) {
  tibble(y = median(x)) %>% 
    mutate(
      fill = if_else(y < threshold, "pink",  "gray35")
    )
}

peng %>% 
  ggplot(aes(species, bill_length_mm)) +
  stat_summary(
    fun.data = calc_median_and_color,
    geom = "bar"
  )
```

![](understanding_stat_summary_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

Here we have deliberately used the colname `fill` so that it used as an
argument of geom.

#### pointrange plot with changing size

``` r
peng %>% 
  ggplot(aes(species, bill_length_mm)) +
  stat_summary(
    fun.data = \(x) {
      scaled_size <- length(x) / nrow(peng)
      mean_se(x) %>% mutate(size = scaled_size)
    }
  )
```

![](understanding_stat_summary_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->
