---
output: hugodown::hugo_document

slug: tidymodels-2021-q1
title: Catch up with tidymodels
date: 2021-03-10
author: Julia Silge
description: >
    Releases of tidymodels packages in Q1 of 2021 offer new functions for 
    easier model building and resampling, along with a new package for 
    resampling spatial data.

photo:
  url: https://unsplash.com/photos/4Zk45jNyQS4
  author: Timo Wielink

# one of: "deep-dive", "learn", "package", "programming", or "other"
categories: [roundup] 
tags: [tidymodels, parsnip, rsample]
---




The [tidymodels](https://www.tidymodels.org/) framework is a collection of R packages for modeling and machine learning using tidyverse principles. There have been quite a number of updates and new developments in the tidymodels ecosystem since our [last blog post in December](https://www.tidyverse.org/blog/2020/12/finetune-0-0-1/)! Since that post, tidymodels maintainers have published eight CRAN releases of existing packages. You can install these updates from CRAN with:


```r
install.packages(c("broom", "butcher", "embed", "parsnip",
                   "rsample", "rules", "tune", "workflows"))
```

We purposefully write code in small, modular packages to make them easier to maintain (for us!) and use in production systems (for you!) but this does mean that sometimes any given package release can feel a bit minor. Some of the changes in these releases are small bug fixes or updates for changes in CRAN standards. However, there are also some substantively helpful new functions for modeling and resampling, and we want to make sure that folks can stay up-to-date with the changes and new features available. 

We plan to begin **regular updates** every three or four months here on the tidyverse blog summarizing what's happening lately in the tidymodels ecosystem overall. We'll still continue the focused blog posts on more major new features that we've always written; look for one soon on a new package for creating and handling a collection of multiple modeling workflows all together. The `NEWS` files are linked here for each package, but read below for more details on some highlights that may interest you!

- [broom](https://broom.tidymodels.org/news/#broom-0-7-5-2021-02-19)
- [butcher](https://butcher.tidymodels.org/news/#butcher-0-1-3-2021-03-04)
- [embed](https://embed.tidymodels.org/news/#embed-0-1-4-2021-01-16)
- [parsnip](https://parsnip.tidymodels.org/news/#parsnip-0-1-5-2021-01-19)
- [rsample](https://rsample.tidymodels.org/news/index.html#rsample-0-0-9-2021-02-17)
- [rules](https://rules.tidymodels.org/news/#rules-0-1-1-2021-01-16)
- [tune](https://tune.tidymodels.org/news/index.html#tune-0-1-3-2021-02-28)
- [workflows](https://workflows.tidymodels.org/news/index.html#workflows-0-2-2-2021-03-10)

## Choose parsnip models with an RStudio addin

The parsnip package provides support for a plethora of models. You can explore these models online at [tidymodels.org](https://www.tidymodels.org/find/parsnip/), but the recent release of parsnip also contains an RStudio addin for choosing parsnip models and generating code to specify them.

![addin gif](parsnip_addin.gif)

You can choose by classification or regression models, and even match by a regular expression.

There is now also [an `augment()` function for parsnip models](https://parsnip.tidymodels.org/reference/augment.html), in addition to the `augment()` functions [for tuning results](https://tune.tidymodels.org/reference/augment.html) and [for workflows](https://workflows.tidymodels.org/reference/augment.workflow.html). [This recent screencast demonstrates](https://juliasilge.com/blog/student-debt/) how to use parsnip's `augment()` function.

## New functions in rsample

Most of the changes in the recent release for [rsample](https://rsample.tidymodels.org/) are internal and developer-facing, made to support rsample-adjacent packages like our new package for resampling spatial data (see below! 👀) but the new `reg_intervals()` function allows you to find bootstrap confidence intervals for simple models fluently. You have always been able to use rsample functions for [flexible bootstrap resampling](https://www.tidymodels.org/learn/statistics/bootstrap/) but this new convenience function reduces the steps to get confidence intervals for models like `lm()` and `glm()`.


```r
library(rsample)
data(ad_data, package = "modeldata")

set.seed(123)
reg_intervals(
  Class ~ tau + VEGF,
  model_fn = "glm", 
  data = ad_data, 
  family = "binomial"
)
```

```
## # A tibble: 2 x 6
##   term  .lower .estimate .upper .alpha .method  
##   <chr>  <dbl>     <dbl>  <dbl>  <dbl> <chr>    
## 1 tau   -4.92     -4.11   -3.08   0.05 student-t
## 2 VEGF   0.651     0.959   1.22   0.05 student-t
```

Check out [my recent screencast](https://juliasilge.com/blog/superbowl-conf-int/) for more details on using `reg_intervals()`.

Also take a look at the [new `permutations()` function](https://rsample.tidymodels.org/reference/permutations.html) for permuting variables!

## Resampling for spatial data

We are pleased to announce the first release of the [spatialsample](https://spatialsample.tidymodels.org/) package.

You can install it from CRAN with:


```r
install.packages("spatialsample")
```

The goal of spatialsample is to provide functions and classes for spatial resampling to use with [rsample](https://rsample.tidymodels.org/). We intend to grow the number of spatial resampling approaches included in the package; the initial release includes `spatial_clustering_cv()`, a straightforward spatial resampling strategy with light dependencies based on k-means clustering.


```r
library(spatialsample)
data("ames", package = "modeldata")

set.seed(234)
folds <- spatial_clustering_cv(ames, coords = c("Latitude", "Longitude"), v = 5)
folds
```

```
## #  5-fold spatial cross-validation 
## # A tibble: 5 x 2
##   splits             id   
##   <list>             <chr>
## 1 <split [2277/653]> Fold1
## 2 <split [2767/163]> Fold2
## 3 <split [2040/890]> Fold3
## 4 <split [2567/363]> Fold4
## 5 <split [2069/861]> Fold5
```

In this example, the `ames` data on houses in Ames, IA is resampled with `v = 5`; notice that the resulting partitions do not contain an equal number of observations.

We can create a helper plotting function to visualize the five folds.


```r
library(ggplot2)
library(purrr)
library(dplyr)

plot_splits <- function(split) {
    p <- analysis(split) %>%
        mutate(analysis = "Analysis") %>%
        bind_rows(assessment(split) %>%
                      mutate(analysis = "Assessment")) %>%
        ggplot(aes(Longitude, Latitude, color = analysis)) + 
        geom_point(alpha = 0.5) +
        labs(color = NULL)
    print(p)
}

walk(folds$splits, plot_splits)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-.gif)

Check out the [main vignette](https://spatialsample.tidymodels.org/articles/spatialsample.html) to see how this resampling strategy can be used for modeling, and [submit an issue](https://github.com/tidymodels/spatialsample/issues) if there is a particular spatial resampling approach that you are interested in us prioritizing for future releases.

## Acknowledgements

A big thanks to all of the contributors who helped make these releases possible! For some of these packages (like rsample, butcher, and embed), we have never said thank you before so we'll take this opportunity to express our appreciation.

- broom: [&#x0040;AdroMine](https://github.com/AdroMine), [&#x0040;alexpghayes](https://github.com/alexpghayes), [&#x0040;Amogh-Joshi](https://github.com/Amogh-Joshi), [&#x0040;anddis](https://github.com/anddis), [&#x0040;andrjohns](https://github.com/andrjohns), [&#x0040;AntoniosBarotsis](https://github.com/AntoniosBarotsis), [&#x0040;arthur-e](https://github.com/arthur-e), [&#x0040;asreece](https://github.com/asreece), [&#x0040;asshah4](https://github.com/asshah4), [&#x0040;briatte](https://github.com/briatte), [&#x0040;bwiernik](https://github.com/bwiernik), [&#x0040;cbhurley](https://github.com/cbhurley), [&#x0040;clausherther](https://github.com/clausherther), [&#x0040;clauswilke](https://github.com/clauswilke), [&#x0040;crsh](https://github.com/crsh), [&#x0040;DarwinAwardWinner](https://github.com/DarwinAwardWinner), [&#x0040;deblnia](https://github.com/deblnia), [&#x0040;deschen1](https://github.com/deschen1), [&#x0040;eheinzen](https://github.com/eheinzen), [&#x0040;friendly](https://github.com/friendly), [&#x0040;grantmcdermott](https://github.com/grantmcdermott), [&#x0040;hasandiwan](https://github.com/hasandiwan), [&#x0040;hd-barros](https://github.com/hd-barros), [&#x0040;hfrick](https://github.com/hfrick), [&#x0040;hughjonesd](https://github.com/hughjonesd), [&#x0040;IndrajeetPatil](https://github.com/IndrajeetPatil), [&#x0040;irkaal](https://github.com/irkaal), [&#x0040;jiho](https://github.com/jiho), [&#x0040;jmbarbone](https://github.com/jmbarbone), [&#x0040;joshyam-k](https://github.com/joshyam-k), [&#x0040;JReising09](https://github.com/JReising09), [&#x0040;julian-urbano](https://github.com/julian-urbano), [&#x0040;juliasilge](https://github.com/juliasilge), [&#x0040;kjfarley](https://github.com/kjfarley), [&#x0040;kristyrobledo](https://github.com/kristyrobledo), [&#x0040;leeweizhe1993](https://github.com/leeweizhe1993), [&#x0040;leungi](https://github.com/leungi), [&#x0040;LukasWallrich](https://github.com/LukasWallrich), [&#x0040;matthieu-faron](https://github.com/matthieu-faron), [&#x0040;MatthieuStigler](https://github.com/MatthieuStigler), [&#x0040;milanwiedemann](https://github.com/milanwiedemann), [&#x0040;mk9y](https://github.com/mk9y), [&#x0040;mlatif71](https://github.com/mlatif71), [&#x0040;mlaviolet](https://github.com/mlaviolet), [&#x0040;Nateme16](https://github.com/Nateme16), [&#x0040;nlubock](https://github.com/nlubock), [&#x0040;pachamaltese](https://github.com/pachamaltese), [&#x0040;rudeboybert](https://github.com/rudeboybert), [&#x0040;saadaslam](https://github.com/saadaslam), [&#x0040;simonpcouch](https://github.com/simonpcouch), [&#x0040;tavareshugo](https://github.com/tavareshugo), [&#x0040;uqzwang](https://github.com/uqzwang), [&#x0040;vincentarelbundock](https://github.com/vincentarelbundock), [&#x0040;WillemVervoort](https://github.com/WillemVervoort), and [&#x0040;zief0002](https://github.com/zief0002)
- butcher: [&#x0040;abichat](https://github.com/abichat), [&#x0040;adtserapio](https://github.com/adtserapio), [&#x0040;DavisVaughan](https://github.com/DavisVaughan), [&#x0040;edwinschut](https://github.com/edwinschut), [&#x0040;hfrick](https://github.com/hfrick), [&#x0040;irkaal](https://github.com/irkaal), [&#x0040;jarauh](https://github.com/jarauh), [&#x0040;juliasilge](https://github.com/juliasilge), [&#x0040;jyuu](https://github.com/jyuu), [&#x0040;kevinykuo](https://github.com/kevinykuo), [&#x0040;klin333](https://github.com/klin333), [&#x0040;mkearney](https://github.com/mkearney), [&#x0040;natejessee](https://github.com/natejessee), [&#x0040;topepo](https://github.com/topepo), and [&#x0040;UnclAlDeveloper](https://github.com/UnclAlDeveloper)
- embed: [&#x0040;agilebean](https://github.com/agilebean), [&#x0040;ajing](https://github.com/ajing), [&#x0040;Athospd](https://github.com/Athospd), [&#x0040;Cardosaum](https://github.com/Cardosaum), [&#x0040;ciberger](https://github.com/ciberger), [&#x0040;data-datum](https://github.com/data-datum), [&#x0040;dfalbel](https://github.com/dfalbel), [&#x0040;EmilHvitfeldt](https://github.com/EmilHvitfeldt), [&#x0040;goleng](https://github.com/goleng), [&#x0040;hfrick](https://github.com/hfrick), [&#x0040;ismailmuller](https://github.com/ismailmuller), [&#x0040;juliasilge](https://github.com/juliasilge), [&#x0040;konradsemsch](https://github.com/konradsemsch), [&#x0040;kylegilde](https://github.com/kylegilde), [&#x0040;lorenzwalthert](https://github.com/lorenzwalthert), [&#x0040;mlduarte](https://github.com/mlduarte), [&#x0040;nhward](https://github.com/nhward), [&#x0040;niszet](https://github.com/niszet), [&#x0040;quantumlinguist](https://github.com/quantumlinguist), [&#x0040;smingerson](https://github.com/smingerson), [&#x0040;tmastny](https://github.com/tmastny), [&#x0040;tonigril](https://github.com/tonigril), and [&#x0040;topepo](https://github.com/topepo)
- parsnip: [&#x0040;awunderground](https://github.com/awunderground), [&#x0040;Bijaelo](https://github.com/Bijaelo), [&#x0040;DavisVaughan](https://github.com/DavisVaughan), [&#x0040;derek-corcoran-barrios](https://github.com/derek-corcoran-barrios), [&#x0040;eamoncaddigan](https://github.com/eamoncaddigan), [&#x0040;EmilHvitfeldt](https://github.com/EmilHvitfeldt), [&#x0040;ericpgreen](https://github.com/ericpgreen), [&#x0040;hfrick](https://github.com/hfrick), [&#x0040;irkaal](https://github.com/irkaal), [&#x0040;jjcurtin](https://github.com/jjcurtin), [&#x0040;joeycouse](https://github.com/joeycouse), [&#x0040;juliasilge](https://github.com/juliasilge), [&#x0040;kwiscion](https://github.com/kwiscion), [&#x0040;kylegilde](https://github.com/kylegilde), [&#x0040;lorenzwalthert](https://github.com/lorenzwalthert), [&#x0040;markfairbanks](https://github.com/markfairbanks), [&#x0040;mdancho84](https://github.com/mdancho84), [&#x0040;mlane3](https://github.com/mlane3), [&#x0040;mrepetto94](https://github.com/mrepetto94), [&#x0040;ndiquattro](https://github.com/ndiquattro), [&#x0040;rorynolan](https://github.com/rorynolan), [&#x0040;shosaco](https://github.com/shosaco), [&#x0040;smingerson](https://github.com/smingerson), [&#x0040;tanho63](https://github.com/tanho63), and [&#x0040;topepo](https://github.com/topepo)
- rsample: [&#x0040;alexpghayes](https://github.com/alexpghayes), [&#x0040;apreshill](https://github.com/apreshill), [&#x0040;Athospd](https://github.com/Athospd), [&#x0040;brunocarlin](https://github.com/brunocarlin), [&#x0040;ColinConwell](https://github.com/ColinConwell), [&#x0040;cportner](https://github.com/cportner), [&#x0040;danilinares](https://github.com/danilinares), [&#x0040;DanOvando](https://github.com/DanOvando), [&#x0040;DavisVaughan](https://github.com/DavisVaughan), [&#x0040;dchiu911](https://github.com/dchiu911), [&#x0040;Dpananos](https://github.com/Dpananos), [&#x0040;dpastling](https://github.com/dpastling), [&#x0040;EmilHvitfeldt](https://github.com/EmilHvitfeldt), [&#x0040;fbchow](https://github.com/fbchow), [&#x0040;fusaroli](https://github.com/fusaroli), [&#x0040;gcameron89777](https://github.com/gcameron89777), [&#x0040;gregrs-uk](https://github.com/gregrs-uk), [&#x0040;gtalckmin](https://github.com/gtalckmin), [&#x0040;hfrick](https://github.com/hfrick), [&#x0040;hlynurhallgrims](https://github.com/hlynurhallgrims), [&#x0040;irkaal](https://github.com/irkaal), [&#x0040;issactoast](https://github.com/issactoast), [&#x0040;JamesM131](https://github.com/JamesM131), [&#x0040;johnaeanderson](https://github.com/johnaeanderson), [&#x0040;jonkeane](https://github.com/jonkeane), [&#x0040;juliasilge](https://github.com/juliasilge), [&#x0040;jyuu](https://github.com/jyuu), [&#x0040;krlmlr](https://github.com/krlmlr), [&#x0040;kylegilde](https://github.com/kylegilde), [&#x0040;mattwarkentin](https://github.com/mattwarkentin), [&#x0040;mdancho84](https://github.com/mdancho84), [&#x0040;msmith01](https://github.com/msmith01), [&#x0040;MxNl](https://github.com/MxNl), [&#x0040;NikolaiVogl](https://github.com/NikolaiVogl), [&#x0040;oude-gao](https://github.com/oude-gao), [&#x0040;PathosEthosLogos](https://github.com/PathosEthosLogos), [&#x0040;RMHogervorst](https://github.com/RMHogervorst), [&#x0040;sccmckenzie](https://github.com/sccmckenzie), [&#x0040;Shu-Wan](https://github.com/Shu-Wan), [&#x0040;skeller88](https://github.com/skeller88), [&#x0040;skinnider](https://github.com/skinnider), [&#x0040;sschooler](https://github.com/sschooler), [&#x0040;swt30](https://github.com/swt30), [&#x0040;tjmahr](https://github.com/tjmahr), [&#x0040;tmastny](https://github.com/tmastny), [&#x0040;topepo](https://github.com/topepo), and [&#x0040;UnclAlDeveloper](https://github.com/UnclAlDeveloper)
- rules: [&#x0040;frequena](https://github.com/frequena), [&#x0040;hfrick](https://github.com/hfrick), [&#x0040;irkaal](https://github.com/irkaal), [&#x0040;jaredlander](https://github.com/jaredlander), [&#x0040;juliasilge](https://github.com/juliasilge), [&#x0040;topepo](https://github.com/topepo), and [&#x0040;vidarsumo](https://github.com/vidarsumo)
- tune: [&#x0040;DavisVaughan](https://github.com/DavisVaughan), [&#x0040;HenrikBengtsson](https://github.com/HenrikBengtsson), [&#x0040;hfrick](https://github.com/hfrick), [&#x0040;juliasilge](https://github.com/juliasilge), [&#x0040;kevin-m-kent](https://github.com/kevin-m-kent), [&#x0040;kylegilde](https://github.com/kylegilde), [&#x0040;mine-cetinkaya-rundel](https://github.com/mine-cetinkaya-rundel), [&#x0040;rorynolan](https://github.com/rorynolan), [&#x0040;siegfried](https://github.com/siegfried), [&#x0040;stevenpawley](https://github.com/stevenpawley), and [&#x0040;topepo](https://github.com/topepo)
