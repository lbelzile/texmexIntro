---
knit: "bookdown::render_book"
title: "Threshold exceedances and multivariate extremes"
author: "Léo Belzile"
site: bookdown::bookdown_site
bibliography: ["book.bib"]
biblio-style: "apalike"
link-citations: true
description: "This booklet is a web complement to the R tutorial for EVA2021 presentation of the `texmex` package by Southworth, Heffernan and Metcalfe (2020)."
---


# Preliminary remarks {-}


These notes are licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/) and were last compiled on 2021-06-20.

:::codingex

Download and install [**R** (current version 4.1.0, nicknamed "Camp Pontanezen")](https://cran.r-project.org/) and an integrated development environment such as [**RStudio**](https://www.rstudio.com/products/rstudio/download/#download).

You can install the **R** tutorial using


```r
# install.packages("remotes")
remotes::install_github("lbelzile/texmexTutorial")
```
It will install `texmex` and other packages alongside with the `learnr` tutorial.

:::
