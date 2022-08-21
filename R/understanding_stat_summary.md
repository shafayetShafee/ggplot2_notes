stat summary of Ggplot2
================

-   <a href="#demystifying-stat_-layers-in-ggplot2"
    id="toc-demystifying-stat_-layers-in-ggplot2">Demystifying stat_ layers
    in {ggplot2}</a>

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

-   **`stat_summary` summerizes one dimension of the data.**
