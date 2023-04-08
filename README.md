
<!-- README.md is generated from README.Rmd. Please edit that file -->

# plot2

<!-- badges: start -->

[![R-universe status
badge](https://grantmcdermott.r-universe.dev/badges/plot2)](https://grantmcdermott.r-universe.dev)
[![R-CMD-check](https://github.com/grantmcdermott/plot2/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/grantmcdermott/plot2/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

A lightweight extension of the base R `plot` function, with support for
automatic grouping and legend handling, and several other enhancements.

## Installation

**plot2** is not yet on CRAN, but can be installed from R-universe.

``` r
install.packages("plot2", repos = "https://grantmcdermott.r-universe.dev")
```

## Motivation

R users are spoiled for choice when it comes to visualization
frameworks. The options of course include **ggplot2** (in my opinion,
the most important graphics system of the last decade) and **lattice**,
not to mention a bewildering array of extensions built around, on top
of, and in between these amazing packages.

It is perhaps not surprising then that the base R graphics system
sometimes gets short shrift. This is unfortunate, because base R offers
very powerful and flexible plotting facilities. Just type in
`demo(graphics)` or `demo(persp)` into your R console to get an idea, or
see these two excellent
[introductory](https://github.com/karoliskoncevicius/tutorial_r_introduction/blob/main/baseplotting.md)
[tutorials](https://quizzical-engelbart-d15a44.netlify.app/2021-2022_m2-data-2_visu-2_practice#1).
The downside of this power and flexibility is that plotting in base R
can require a fair bit of manual tinkering. A case in point is plotting
grouped data and then generating an appropriate legend. Doing this with
the generic `plot()` function can require several function calls (or a
loop), fiddling with your plot regions, and writing the legend out
manually.

The goal of `plot2` is to remove this annoyance by providing a
lightweight extension of the base (2D) `plot` function, particularly as
it applies to scatter and line plots with grouped data. For example,
`plot2` makes it easy to plot different categories of a dataset in a
single function call and highlight these categories (groups) using
modern colour palettes. Coincident with this grouping support, `plot2`
also produces automatic legends with scope for further customization.
While the package offers several other minor enhancements, it tries as
far as possible to be a drop-in replacement for the equivalent base plot
function. Users should be able to swap a valid `plot` call with `plot2`
without any changes to the expected output.

In summary, consider trying the **plot2** package if you are looking for
a more convenient version of the base R `plot` functionality. You can
use pretty much the same syntax and all of your theming elements should
carry over too. It has no other dependencies than base R itself, which
might make it more suitable for situations where dependency management
is expensive (e.g., an R application running in a browser via
[WebAssembly](https://docs.r-wasm.org/webr/latest/).)

## Examples

Let’s load the package then walk through some examples.

``` r
library(plot2)
```

As far as possible, `plot2` tries to be a drop-in replacement for
regular `plot` calls.

``` r
op = par(mfrow = c(1, 2))

plot(0:10, main = "plot")
plot2(0:10, main = "plot2")
```

<img src="man/figures/README-unnamed-chunk-3-1.png" width="100%" />

Similarly, we can plot elements from a data frame using either the
atomic or formula methods.

``` r
op = par(mfrow = c(2, 2))

plot(airquality$Day, airquality$Temp, main = "plot")
plot(Temp ~ Day, data = airquality, main = "plot (formula)")
plot2(airquality$Day, airquality$Temp, main = "plot2")
plot2(Temp ~ Day, data = airquality, main = "plot2 (formula)")
```

<img src="man/figures/README-unnamed-chunk-4-1.png" width="100%" />

``` r

par(op) # reset to a unique (single) plot frame
```

So far, so good. But where `plot2` starts to diverge from its base
counterpart is with respect to grouped data. In particular, `plot2`
allows you to characterize groups using the `by` argument.[^1] for a
longer discussion.\]

``` r
plot2(airquality$Day, airquality$Temp, by = airquality$Month)
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="100%" />

Notice that we also get an automatic legend. We’ll return to legend
customization shortly.

An even more convenient approach is to use the equivalent formula
syntax. Just place the grouping variable after a vertical bar (i.e.,
`|`).

``` r
plot2(Temp ~ Day | Month, airquality)
```

<img src="man/figures/README-unnamed-chunk-6-1.png" width="100%" />

Converting to a grouped line plot is a simple matter of adjusting the
`type` argument. I’ll also use this as an opportunity to highlight that
the legend responds to these plot element changes accordingly and can be
further customized. Note the use of the “!” in the `legend.position`
argument to place the legend *outside* the plot area.

``` r
plot2( 
  Temp ~ Day | Month, airquality,
  type = "l",
  legend.position = "right!"
)
```

<img src="man/figures/README-unnamed-chunk-7-1.png" width="100%" />

Let’s discuss colour palettes briefly. The default group colours in
`plot2` are either “Okabe-Ito” or “Viridis”, depending on the number of
groups. But this is easily changed via the `palette` argument. Note that
all palettes supported by `palette.pals()` and `hcl.pals()` are
supported out-of-the-box.[^2] Just pass on an appropriate palette name
as a string.

``` r
plot2(
  Temp ~ Day | Month, airquality,
  type = "l",
  legend.position = "right!",
  palette = "Tableau 10"
)
```

<img src="man/figures/README-unnamed-chunk-8-1.png" width="100%" />

Its possible to add a lot more customization to your plots. For example,
the accompanying `legend.args` and `palette.args` arguments allow you to
turn off the legend title and bounding box, switch the direction of the
text, add transparency to your colour palette, etc. etc. Here’s a quick
penultimate example, where we also show off `plot2`’s enhanced `grid`
argument.

``` r
par(family = "Hershey") # Use R's built-in Hershey fonts instead of Arial default

plot2(
  Temp ~ Day | Month, airquality,
  type = "b", pch = 16,
  grid = grid(), frame.plot = FALSE,
  legend.position = "right!", legend.args = list(bty = "n"),
  palette = "Tableau 10", palette.args = list(alpha = 0.5)
)
```

<img src="man/figures/README-unnamed-chunk-9-1.png" width="100%" />

The use of `par` (to set the font family) in the above example again
underscores the correspondence with the base graphics system. Because
`plot2` is effectively a convenience wrapper around base `plot`, any
global elements that you have set for the latter should carry over to
the former. For nice out-of-the-box themes, I recommend the
**basetheme** package.

``` r
library(basetheme)
basetheme("royal") # or "clean", "dark", "ink", "brutal", etc.

plot2(
  Temp ~ Day | Month, airquality,
  type = "b", pch = 15:19,
  legend.position = "right!", legend.args = list(bty = "n"),
  palette = "Tropic"
)
```

<img src="man/figures/README-unnamed-chunk-10-1.png" width="100%" />

``` r

# basetheme(NULL) # back to default
```

[^1]: At this point, experienced base plot users might protest that you
    *can* colour by groups using the `col` argument, e.g.
    `plot(airquality$Day, airquality$Temp, col = airquality$Month)`.
    This is true, but there are several points to push back on. First,
    you don’t get an automatic legend. Second, the base `plot` formula
    method doesn’t specify the grouping within the formula itself (not a
    deal-breaker, but not particularly consistent in my view). Third,
    and perhaps most importantly, this grouping doesn’t carry over to
    line plots (i.e., type=“l”). Instead, you have to transpose your
    data and use `mplot`. See
    [this](https://stackoverflow.com/questions/10519873/how-to-create-a-line-plot-with-groups-in-base-r-without-loops)
    old StackOverflow thread

[^2]: See the accompanying help pages of those functions for more
    details, or read the [article](https://arxiv.org/pdf/2303.04918.pdf)
    by Achim Zeileis and Paul Murrell.
