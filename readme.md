# generativeart

Create Generative Art with R.

![](img/generativeart.png)

[More on Instagram](https://www.instagram.com/cutterkom/)

## Description

> One overly simple but useful definition is that generative art is art programmed using a computer that intentionally introduces randomness as part of its creation process.
-- [Why Love Generative Art? - Artnome](https://www.artnome.com/news/2018/8/8/why-love-generative-art)

The `R` package `generativeart` let's you create images based on many thousand points.
The position of every single point is calculated by a formula, which has random parameters.
Because of the random numbers, every image looks different.

In order to make an image reproducible, `generative art` implements a log file that saves the `file_name`, the `seed` and the `formula`.

## Install

You can install the package with the `devtools` package directly from Github:

```r
devtools::install_github("cutterkom/generativeart")
```

`generativeart` uses the packages `ggplot2`, `magrittr`, `purrr` and `dplyr`.

## Usage

The package works with a specific directory structure that fits my needs best.
The first step is to create it with `setup_directories()`.
All images are saved by default in `img/everything/`. I use `img/handpicked/` to choose the best ones.
In `logfile/` you will find a `csv` file that saves the `file_name`, the `seed` and the used `formula`.

```r
library(generativeart)

# set the paths
IMG_DIR <- "img/"
IMG_SUBDIR <- "everything/"
IMG_SUBDIR2 <- "handpicked/"
IMG_PATH <- paste0(IMG_DIR, IMG_SUBDIR)

LOGFILE_DIR <- "logfile/"
LOGFILE <- "logfile.csv"
LOGFILE_PATH <- paste0(LOGFILE_DIR, LOGFILE)

# create the directory structure
generativeart::setup_directories(IMG_DIR, IMG_SUBDIR, IMG_SUBDIR2, LOGFILE_DIR)

# include a specific formula, for example:
my_formula <- list(
  x = quote(runif(1, -1, 1) * x_i^2 - sin(y_i^2)),
  y = quote(runif(1, -1, 1) * y_i^3 - cos(x_i^2))
)

# call the main function to create five images with a polar coordinate system
generativeart::generate_img(formula = my_formula, nr_of_img = 5, polar = TRUE, filetype = "png", color = "black", background_color = "white")

```

* You can create as many images as you want by setting `nr_of_img`.
* For every image a seed is drawn from a number between 1 and 10000.
* This seed determines the random numbers in the formula.
* You can choose between cartesian and polar coordinate systems by setting `polar = TRUE` or `polar = FALSE`
* You can choose the colors with `color = 'black'` and `background_color = 'hotpink'`
* You can save the output image in various formats.
Default is `png`, the alternatives are defined by the `device` options of [`ggplot::ggsave()`](https://ggplot2.tidyverse.org/reference/ggsave.html).
* the formula is a `list()`

## Examples

It is a good idea to use the sine and cosine in the formula, since it guarantees nice shapes, especially when combined with a polar coordinate system. One simple example:

```r
my_formula <- list(
  x = quote(runif(1, -1, 1) * x_i^2 - sin(y_i^2)),
  y = quote(runif(1, -1, 1) * y_i^3 - cos(x_i^2))
)

generativeart::generate_img(formula = my_formula, nr_of_img = 5, polar = TRUE, color = "black", background_color = "white")

```

Two possible images:

`seed = 1821`, `polar = TRUE`:
![](img/2018-11-16-17-13_seed_1821.png)

`seed = 5451`, `polar = FALSE`:
![](img/2018-11-16-17-12_seed_5451.png)

The corresponding log file looks like that:

| file_name                      | seed | formula_x                            | formula_y                            | 
|--------------------------------|------|--------------------------------------|--------------------------------------| 
| 2018-11-16-17-13_seed_1821.png | 1821 | runif(1, -1, 1) * x_i^2 - sin(y_i^2) | runif(1, -1, 1) * y_i^3 - cos(x_i^2) | 
| 2018-11-16-17-12_seed_5451.png | 5451 | runif(1, -1, 1) * x_i^2 - sin(y_i^2) | runif(1, -1, 1) * y_i^3 - cos(x_i^2) | 


## Inspiration

The basic concept is heavily inspired by [Fronkonstin's great blog](https://fronkonstin.com/).

## My note
### Main function: generative_img.R

The main job of generative_img

1. Call generate_data(formula) and get a returned dataframe storing x-y-points-pairs evaluated in terms of the formula

2. Call geneate_plot (to create plot with ggplot2) with the dataframe obtained from step 1 passed as argument

### Data generation: generate_data.R

Prerequsites:
  - %>% is called the forward pipe operator
    - The LHS of %>% will be put into the first argument of the RHS function
  - length(seq(from = -pi, to = pi, by = 0.01)) is a vector of length 629

Code:
```
seq(from = -pi, to = pi, by = 0.01) %>% expand.grid(x_i = ., y_i = .)
```
  - expand.grid will return a (629^2 by 2) dataframe consists of rows in the form (x_i, y_i). i.e. all possible combinations of elements of the vectors passed in

    - [See usage of expand.grid](https://vimsky.com/zh-tw/examples/usage/create-a-data-frame-of-all-the-combinations-of-vectors-passed-as-argument-in-r-programming-expand-grid-function.html)


```
seq(from = -pi, to = pi, by = 0.01) %>%
    expand.grid(x_i = ., y_i = .) %>%
    dplyr::mutate(!!!formula)
```
- [Triple exclamation marks on R](https://stackoverflow.com/questions/61180201/triple-exclamation-marks-on-r)

- The mutate function in dplyr packages will create variables and column bind the new variable into the dataframe passed in. 
- So after the line of code executed, the dataframe becomes dimension of (629^2 by 4)
- The (x, y) pair is added in col3 and col4 respectively, with their value evaluated by the formula provided by the user

### Plotting: generate_plot.R

[Learn ggplot2 with R-for-data-science](https://r4ds.had.co.nz/data-visualisation.html)

[Learn ggplot2 with this comprehensive guide](https://ggplot2-book.org/coord.html)

- We only plot the last two columns of the dataframe returned by generate_data.R, which is (x, y).

- Note that by default the angle is mapped to the x variable, but you can set theta = "y" to map the angle to the y variable.