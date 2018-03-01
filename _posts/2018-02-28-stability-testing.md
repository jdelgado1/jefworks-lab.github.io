---
layout: post
comments: false
tags: [single cell, tutorial, work in progress, R]
---

<script src="https://code.highcharts.com/highcharts.js"></script>
<script src="https://code.highcharts.com/modules/heatmap.js"></script>
<script src="https://code.highcharts.com/highcharts-more.js"></script>

# Stability testing: How do you know whether your single-cell clusters are 'real'?

In single-cell RNA-seq analysis, we are often looking to identify transcriptional subpopulations that may be interpreted as distinct cell-types and subtypes or cell-states. For each cell, we measure its expression of thousands of genes, which we can use as features to cluster these cells into transcriptionally-similar clusters. But, one questions that keeps popping up in my mind:

> How do you know whether your clusters are 'real'? How do you know you haven't 'over-clustered' your data?

In this blog post, I will show a few examples with both simulated and real data to highlight why we should care about this issue of over-clustering and provide a strategy to address over-clustering available in the [`MUDAN` package](https://github.com/JEFworks/MUDAN) I am currently developing.

## Simulated data

First, we will explore the issue of over-clustering and how to address over-clustering using simulated data. So let's first simulate some data!

```r
#' Simulate gene expression counts counts
#' @param G number of groups
#' @param N number of cells per group
#' @param M number of genes
#' @param initmean default mean gene expression
#' @param initvar default gene expression variance
#' @param upreg gene expression increase for upregulated genes
#' @param upregvar gene expression variance for upregulated genes
#' @param ng number of upregulated genes per group
simulate.data <- function(G=5, N=30, M=100, initmean=0, initvar=10, upreg=10, upregvar=10, ng=20, seed=0) {
    set.seed(seed)
    mat <- matrix(rnorm(N*M*G, initmean, initvar), M, N*G)
    rownames(mat) <- paste0('gene', 1:M)
    colnames(mat) <- paste0('cell', 1:(N*G))
    group <- factor(sapply(1:G, function(x) rep(paste0('group', x), N)))
    names(group) <- colnames(mat)

    diff <- lapply(1:G, function(x) {
        diff <- rownames(mat)[(((x-1)*ng)+1):(((x-1)*ng)+ng)]
        mat[diff, group==paste0('group', x)] <<- mat[diff, group==paste0('group', x)] + rnorm(ng, upreg, upregvar)
        return(diff)
    })
    names(diff) <- paste0('group', 1:G)

    mat[mat<0] <- 0
    mat <- round(mat)

    return(list(cd=mat, group=group))
}
data <- simulate.data()
```

We can visualize our simulated data as a heatmap. Check out my [previous post on `highcharter`](http://jef.works/blog/2018/02/10/interactive-visualizations-with-highcharter/) in case you are interested in how this plot is generated and embedded in this blog post. (You can also view the page source). Obviously, we simulated for and can see 5 distinct transcriptional subpopulations or clusters.

```r
library(highcharter)
hc1 <- hchart(data$cd, "heatmap", hcaes(x = gene, y = data$group, value = value)) %>%
    hc_colorAxis(stops = color_stops(10, rev(RColorBrewer::brewer.pal(10, "RdBu")))) %>%
    hc_title(text = "Simulated Gene Expression") %>%
    hc_legend(layout = "vertical", verticalAlign = "bottom", align = "right", valueDecimals = 0) %>%
    hc_plotOptions(series = list(dataLabels = list(enabled = FALSE)))
```

<div id='hc1'></div><script>$(function () {
$('#hc1').highcharts(
{
  "title": {
    "text": "Simulated Gene Expression"
  },
  "yAxis": {
    "title": {
      "text": ""
    },
    "categories": ["gene1", "gene2", "gene3", "gene4", "gene5", "gene6", "gene7", "gene8", "gene9", "gene10", "gene11", "gene12", "gene13", "gene14", "gene15", "gene16", "gene17", "gene18", "gene19", "gene20", "gene21", "gene22", "gene23", "gene24", "gene25", "gene26", "gene27", "gene28", "gene29", "gene30", "gene31", "gene32", "gene33", "gene34", "gene35", "gene36", "gene37", "gene38", "gene39", "gene40", "gene41", "gene42", "gene43", "gene44", "gene45", "gene46", "gene47", "gene48", "gene49", "gene50", "gene51", "gene52", "gene53", "gene54", "gene55", "gene56", "gene57", "gene58", "gene59", "gene60", "gene61", "gene62", "gene63", "gene64", "gene65", "gene66", "gene67", "gene68", "gene69", "gene70", "gene71", "gene72", "gene73", "gene74", "gene75", "gene76", "gene77", "gene78", "gene79", "gene80", "gene81", "gene82", "gene83", "gene84", "gene85", "gene86", "gene87", "gene88", "gene89", "gene90", "gene91", "gene92", "gene93", "gene94", "gene95", "gene96", "gene97", "gene98", "gene99", "gene100"],
    "reversed": true
  },
  "credits": {
    "enabled": false
  },
  "exporting": {
    "enabled": false
  },
  "plotOptions": {
    "series": {
      "label": {
        "enabled": false
      },
      "turboThreshold": 0,
      "showInLegend": {
        "x": {},
        "y": {},
        "value": {}
      },
      "boderWidth": 0,
      "dataLabels": {
        "enabled": false
      }
    },
    "treemap": {
      "layoutAlgorithm": "squarified"
    }
  },
  "series": [
    {
      "data": [
        {
          "x": 0,
          "y": 0,
          "value": 19,
          "name": "cell1 ~ gene1"
        },
        {
          "x": 0,
          "y": 1,
          "value": 0,
          "name": "cell1 ~ gene2"
        },
        {
          "x": 0,
          "y": 2,
          "value": 32,
          "name": "cell1 ~ gene3"
        },
        {
          "x": 0,
          "y": 3,
          "value": 34,
          "name": "cell1 ~ gene4"
        },
        {
          "x": 0,
          "y": 4,
          "value": 23,
          "name": "cell1 ~ gene5"
        },
        {
          "x": 0,
          "y": 5,
          "value": 0,
          "name": "cell1 ~ gene6"
        },
        {
          "x": 0,
          "y": 6,
          "value": 17,
          "name": "cell1 ~ gene7"
        },
        {
          "x": 0,
          "y": 7,
          "value": 8,
          "name": "cell1 ~ gene8"
        },
        {
          "x": 0,
          "y": 8,
          "value": 4,
          "name": "cell1 ~ gene9"
        },
        {
          "x": 0,
          "y": 9,
          "value": 23,
          "name": "cell1 ~ gene10"
        },
        {
          "x": 0,
          "y": 10,
          "value": 22,
          "name": "cell1 ~ gene11"
        },
        {
          "x": 0,
          "y": 11,
          "value": 0,
          "name": "cell1 ~ gene12"
        },
        {
          "x": 0,
          "y": 12,
          "value": 6,
          "name": "cell1 ~ gene13"
        },
        {
          "x": 0,
          "y": 13,
          "value": 9,
          "name": "cell1 ~ gene14"
        },
        {
          "x": 0,
          "y": 14,
          "value": 22,
          "name": "cell1 ~ gene15"
        },
        {
          "x": 0,
          "y": 15,
          "value": 28,
          "name": "cell1 ~ gene16"
        },
        {
          "x": 0,
          "y": 16,
          "value": 16,
          "name": "cell1 ~ gene17"
        },
        {
          "x": 0,
          "y": 17,
          "value": 12,
          "name": "cell1 ~ gene18"
        },
        {
          "x": 0,
          "y": 18,
          "value": 27,
          "name": "cell1 ~ gene19"
        },
        {
          "x": 0,
          "y": 19,
          "value": 0,
          "name": "cell1 ~ gene20"
        },
        {
          "x": 0,
          "y": 20,
          "value": 0,
          "name": "cell1 ~ gene21"
        },
        {
          "x": 0,
          "y": 21,
          "value": 4,
          "name": "cell1 ~ gene22"
        },
        {
          "x": 0,
          "y": 22,
          "value": 1,
          "name": "cell1 ~ gene23"
        },
        {
          "x": 0,
          "y": 23,
          "value": 8,
          "name": "cell1 ~ gene24"
        },
        {
          "x": 0,
          "y": 24,
          "value": 0,
          "name": "cell1 ~ gene25"
        },
        {
          "x": 0,
          "y": 25,
          "value": 5,
          "name": "cell1 ~ gene26"
        },
        {
          "x": 0,
          "y": 26,
          "value": 11,
          "name": "cell1 ~ gene27"
        },
        {
          "x": 0,
          "y": 27,
          "value": 0,
          "name": "cell1 ~ gene28"
        },
        {
          "x": 0,
          "y": 28,
          "value": 0,
          "name": "cell1 ~ gene29"
        },
        {
          "x": 0,
          "y": 29,
          "value": 0,
          "name": "cell1 ~ gene30"
        },
        {
          "x": 0,
          "y": 30,
          "value": 0,
          "name": "cell1 ~ gene31"
        },
        {
          "x": 0,
          "y": 31,
          "value": 0,
          "name": "cell1 ~ gene32"
        },
        {
          "x": 0,
          "y": 32,
          "value": 0,
          "name": "cell1 ~ gene33"
        },
        {
          "x": 0,
          "y": 33,
          "value": 0,
          "name": "cell1 ~ gene34"
        },
        {
          "x": 0,
          "y": 34,
          "value": 7,
          "name": "cell1 ~ gene35"
        },
        {
          "x": 0,
          "y": 35,
          "value": 12,
          "name": "cell1 ~ gene36"
        },
        {
          "x": 0,
          "y": 36,
          "value": 10,
          "name": "cell1 ~ gene37"
        },
        {
          "x": 0,
          "y": 37,
          "value": 0,
          "name": "cell1 ~ gene38"
        },
        {
          "x": 0,
          "y": 38,
          "value": 12,
          "name": "cell1 ~ gene39"
        },
        {
          "x": 0,
          "y": 39,
          "value": 0,
          "name": "cell1 ~ gene40"
        },
        {
          "x": 0,
          "y": 40,
          "value": 18,
          "name": "cell1 ~ gene41"
        },
        {
          "x": 0,
          "y": 41,
          "value": 6,
          "name": "cell1 ~ gene42"
        },
        {
          "x": 0,
          "y": 42,
          "value": 0,
          "name": "cell1 ~ gene43"
        },
        {
          "x": 0,
          "y": 43,
          "value": 0,
          "name": "cell1 ~ gene44"
        },
        {
          "x": 0,
          "y": 44,
          "value": 0,
          "name": "cell1 ~ gene45"
        },
        {
          "x": 0,
          "y": 45,
          "value": 0,
          "name": "cell1 ~ gene46"
        },
        {
          "x": 0,
          "y": 46,
          "value": 0,
          "name": "cell1 ~ gene47"
        },
        {
          "x": 0,
          "y": 47,
          "value": 12,
          "name": "cell1 ~ gene48"
        },
        {
          "x": 0,
          "y": 48,
          "value": 8,
          "name": "cell1 ~ gene49"
        },
        {
          "x": 0,
          "y": 49,
          "value": 0,
          "name": "cell1 ~ gene50"
        },
        {
          "x": 0,
          "y": 50,
          "value": 3,
          "name": "cell1 ~ gene51"
        },
        {
          "x": 0,
          "y": 51,
          "value": 0,
          "name": "cell1 ~ gene52"
        },
        {
          "x": 0,
          "y": 52,
          "value": 24,
          "name": "cell1 ~ gene53"
        },
        {
          "x": 0,
          "y": 53,
          "value": 0,
          "name": "cell1 ~ gene54"
        },
        {
          "x": 0,
          "y": 54,
          "value": 0,
          "name": "cell1 ~ gene55"
        },
        {
          "x": 0,
          "y": 55,
          "value": 3,
          "name": "cell1 ~ gene56"
        },
        {
          "x": 0,
          "y": 56,
          "value": 6,
          "name": "cell1 ~ gene57"
        },
        {
          "x": 0,
          "y": 57,
          "value": 0,
          "name": "cell1 ~ gene58"
        },
        {
          "x": 0,
          "y": 58,
          "value": 0,
          "name": "cell1 ~ gene59"
        },
        {
          "x": 0,
          "y": 59,
          "value": 0,
          "name": "cell1 ~ gene60"
        },
        {
          "x": 0,
          "y": 60,
          "value": 4,
          "name": "cell1 ~ gene61"
        },
        {
          "x": 0,
          "y": 61,
          "value": 0,
          "name": "cell1 ~ gene62"
        },
        {
          "x": 0,
          "y": 62,
          "value": 0,
          "name": "cell1 ~ gene63"
        },
        {
          "x": 0,
          "y": 63,
          "value": 0,
          "name": "cell1 ~ gene64"
        },
        {
          "x": 0,
          "y": 64,
          "value": 0,
          "name": "cell1 ~ gene65"
        },
        {
          "x": 0,
          "y": 65,
          "value": 2,
          "name": "cell1 ~ gene66"
        },
        {
          "x": 0,
          "y": 66,
          "value": 0,
          "name": "cell1 ~ gene67"
        },
        {
          "x": 0,
          "y": 67,
          "value": 4,
          "name": "cell1 ~ gene68"
        },
        {
          "x": 0,
          "y": 68,
          "value": 2,
          "name": "cell1 ~ gene69"
        },
        {
          "x": 0,
          "y": 69,
          "value": 1,
          "name": "cell1 ~ gene70"
        },
        {
          "x": 0,
          "y": 70,
          "value": 0,
          "name": "cell1 ~ gene71"
        },
        {
          "x": 0,
          "y": 71,
          "value": 3,
          "name": "cell1 ~ gene72"
        },
        {
          "x": 0,
          "y": 72,
          "value": 0,
          "name": "cell1 ~ gene73"
        },
        {
          "x": 0,
          "y": 73,
          "value": 0,
          "name": "cell1 ~ gene74"
        },
        {
          "x": 0,
          "y": 74,
          "value": 7,
          "name": "cell1 ~ gene75"
        },
        {
          "x": 0,
          "y": 75,
          "value": 11,
          "name": "cell1 ~ gene76"
        },
        {
          "x": 0,
          "y": 76,
          "value": 1,
          "name": "cell1 ~ gene77"
        },
        {
          "x": 0,
          "y": 77,
          "value": 0,
          "name": "cell1 ~ gene78"
        },
        {
          "x": 0,
          "y": 78,
          "value": 0,
          "name": "cell1 ~ gene79"
        },
        {
          "x": 0,
          "y": 79,
          "value": 0,
          "name": "cell1 ~ gene80"
        },
        {
          "x": 0,
          "y": 80,
          "value": 0,
          "name": "cell1 ~ gene81"
        },
        {
          "x": 0,
          "y": 81,
          "value": 13,
          "name": "cell1 ~ gene82"
        },
        {
          "x": 0,
          "y": 82,
          "value": 8,
          "name": "cell1 ~ gene83"
        },
        {
          "x": 0,
          "y": 83,
          "value": 0,
          "name": "cell1 ~ gene84"
        },
        {
          "x": 0,
          "y": 84,
          "value": 0,
          "name": "cell1 ~ gene85"
        },
        {
          "x": 0,
          "y": 85,
          "value": 0,
          "name": "cell1 ~ gene86"
        },
        {
          "x": 0,
          "y": 86,
          "value": 10,
          "name": "cell1 ~ gene87"
        },
        {
          "x": 0,
          "y": 87,
          "value": 0,
          "name": "cell1 ~ gene88"
        },
        {
          "x": 0,
          "y": 88,
          "value": 13,
          "name": "cell1 ~ gene89"
        },
        {
          "x": 0,
          "y": 89,
          "value": 6,
          "name": "cell1 ~ gene90"
        },
        {
          "x": 0,
          "y": 90,
          "value": 13,
          "name": "cell1 ~ gene91"
        },
        {
          "x": 0,
          "y": 91,
          "value": 0,
          "name": "cell1 ~ gene92"
        },
        {
          "x": 0,
          "y": 92,
          "value": 0,
          "name": "cell1 ~ gene93"
        },
        {
          "x": 0,
          "y": 93,
          "value": 0,
          "name": "cell1 ~ gene94"
        },
        {
          "x": 0,
          "y": 94,
          "value": 6,
          "name": "cell1 ~ gene95"
        },
        {
          "x": 0,
          "y": 95,
          "value": 1,
          "name": "cell1 ~ gene96"
        },
        {
          "x": 0,
          "y": 96,
          "value": 0,
          "name": "cell1 ~ gene97"
        },
        {
          "x": 0,
          "y": 97,
          "value": 15,
          "name": "cell1 ~ gene98"
        },
        {
          "x": 0,
          "y": 98,
          "value": 2,
          "name": "cell1 ~ gene99"
        },
        {
          "x": 0,
          "y": 99,
          "value": 10,
          "name": "cell1 ~ gene100"
        },
        {
          "x": 1,
          "y": 0,
          "value": 14,
          "name": "cell2 ~ gene1"
        },
        {
          "x": 1,
          "y": 1,
          "value": 0,
          "name": "cell2 ~ gene2"
        },
        {
          "x": 1,
          "y": 2,
          "value": 12,
          "name": "cell2 ~ gene3"
        },
        {
          "x": 1,
          "y": 3,
          "value": 22,
          "name": "cell2 ~ gene4"
        },
        {
          "x": 1,
          "y": 4,
          "value": 8,
          "name": "cell2 ~ gene5"
        },
        {
          "x": 1,
          "y": 5,
          "value": 18,
          "name": "cell2 ~ gene6"
        },
        {
          "x": 1,
          "y": 6,
          "value": 13,
          "name": "cell2 ~ gene7"
        },
        {
          "x": 1,
          "y": 7,
          "value": 27,
          "name": "cell2 ~ gene8"
        },
        {
          "x": 1,
          "y": 8,
          "value": 0,
          "name": "cell2 ~ gene9"
        },
        {
          "x": 1,
          "y": 9,
          "value": 16,
          "name": "cell2 ~ gene10"
        },
        {
          "x": 1,
          "y": 10,
          "value": 10,
          "name": "cell2 ~ gene11"
        },
        {
          "x": 1,
          "y": 11,
          "value": 0,
          "name": "cell2 ~ gene12"
        },
        {
          "x": 1,
          "y": 12,
          "value": 18,
          "name": "cell2 ~ gene13"
        },
        {
          "x": 1,
          "y": 13,
          "value": 12,
          "name": "cell2 ~ gene14"
        },
        {
          "x": 1,
          "y": 14,
          "value": 8,
          "name": "cell2 ~ gene15"
        },
        {
          "x": 1,
          "y": 15,
          "value": 43,
          "name": "cell2 ~ gene16"
        },
        {
          "x": 1,
          "y": 16,
          "value": 2,
          "name": "cell2 ~ gene17"
        },
        {
          "x": 1,
          "y": 17,
          "value": 24,
          "name": "cell2 ~ gene18"
        },
        {
          "x": 1,
          "y": 18,
          "value": 27,
          "name": "cell2 ~ gene19"
        },
        {
          "x": 1,
          "y": 19,
          "value": 5,
          "name": "cell2 ~ gene20"
        },
        {
          "x": 1,
          "y": 20,
          "value": 0,
          "name": "cell2 ~ gene21"
        },
        {
          "x": 1,
          "y": 21,
          "value": 2,
          "name": "cell2 ~ gene22"
        },
        {
          "x": 1,
          "y": 22,
          "value": 0,
          "name": "cell2 ~ gene23"
        },
        {
          "x": 1,
          "y": 23,
          "value": 0,
          "name": "cell2 ~ gene24"
        },
        {
          "x": 1,
          "y": 24,
          "value": 0,
          "name": "cell2 ~ gene25"
        },
        {
          "x": 1,
          "y": 25,
          "value": 16,
          "name": "cell2 ~ gene26"
        },
        {
          "x": 1,
          "y": 26,
          "value": 15,
          "name": "cell2 ~ gene27"
        },
        {
          "x": 1,
          "y": 27,
          "value": 3,
          "name": "cell2 ~ gene28"
        },
        {
          "x": 1,
          "y": 28,
          "value": 0,
          "name": "cell2 ~ gene29"
        },
        {
          "x": 1,
          "y": 29,
          "value": 0,
          "name": "cell2 ~ gene30"
        },
        {
          "x": 1,
          "y": 30,
          "value": 15,
          "name": "cell2 ~ gene31"
        },
        {
          "x": 1,
          "y": 31,
          "value": 0,
          "name": "cell2 ~ gene32"
        },
        {
          "x": 1,
          "y": 32,
          "value": 8,
          "name": "cell2 ~ gene33"
        },
        {
          "x": 1,
          "y": 33,
          "value": 0,
          "name": "cell2 ~ gene34"
        },
        {
          "x": 1,
          "y": 34,
          "value": 7,
          "name": "cell2 ~ gene35"
        },
        {
          "x": 1,
          "y": 35,
          "value": 0,
          "name": "cell2 ~ gene36"
        },
        {
          "x": 1,
          "y": 36,
          "value": 0,
          "name": "cell2 ~ gene37"
        },
        {
          "x": 1,
          "y": 37,
          "value": 0,
          "name": "cell2 ~ gene38"
        },
        {
          "x": 1,
          "y": 38,
          "value": 0,
          "name": "cell2 ~ gene39"
        },
        {
          "x": 1,
          "y": 39,
          "value": 0,
          "name": "cell2 ~ gene40"
        },
        {
          "x": 1,
          "y": 40,
          "value": 7,
          "name": "cell2 ~ gene41"
        },
        {
          "x": 1,
          "y": 41,
          "value": 1,
          "name": "cell2 ~ gene42"
        },
        {
          "x": 1,
          "y": 42,
          "value": 0,
          "name": "cell2 ~ gene43"
        },
        {
          "x": 1,
          "y": 43,
          "value": 0,
          "name": "cell2 ~ gene44"
        },
        {
          "x": 1,
          "y": 44,
          "value": 0,
          "name": "cell2 ~ gene45"
        },
        {
          "x": 1,
          "y": 45,
          "value": 0,
          "name": "cell2 ~ gene46"
        },
        {
          "x": 1,
          "y": 46,
          "value": 0,
          "name": "cell2 ~ gene47"
        },
        {
          "x": 1,
          "y": 47,
          "value": 0,
          "name": "cell2 ~ gene48"
        },
        {
          "x": 1,
          "y": 48,
          "value": 5,
          "name": "cell2 ~ gene49"
        },
        {
          "x": 1,
          "y": 49,
          "value": 0,
          "name": "cell2 ~ gene50"
        },
        {
          "x": 1,
          "y": 50,
          "value": 15,
          "name": "cell2 ~ gene51"
        },
        {
          "x": 1,
          "y": 51,
          "value": 0,
          "name": "cell2 ~ gene52"
        },
        {
          "x": 1,
          "y": 52,
          "value": 0,
          "name": "cell2 ~ gene53"
        },
        {
          "x": 1,
          "y": 53,
          "value": 24,
          "name": "cell2 ~ gene54"
        },
        {
          "x": 1,
          "y": 54,
          "value": 9,
          "name": "cell2 ~ gene55"
        },
        {
          "x": 1,
          "y": 55,
          "value": 0,
          "name": "cell2 ~ gene56"
        },
        {
          "x": 1,
          "y": 56,
          "value": 0,
          "name": "cell2 ~ gene57"
        },
        {
          "x": 1,
          "y": 57,
          "value": 6,
          "name": "cell2 ~ gene58"
        },
        {
          "x": 1,
          "y": 58,
          "value": 0,
          "name": "cell2 ~ gene59"
        },
        {
          "x": 1,
          "y": 59,
          "value": 0,
          "name": "cell2 ~ gene60"
        },
        {
          "x": 1,
          "y": 60,
          "value": 3,
          "name": "cell2 ~ gene61"
        },
        {
          "x": 1,
          "y": 61,
          "value": 0,
          "name": "cell2 ~ gene62"
        },
        {
          "x": 1,
          "y": 62,
          "value": 27,
          "name": "cell2 ~ gene63"
        },
        {
          "x": 1,
          "y": 63,
          "value": 17,
          "name": "cell2 ~ gene64"
        },
        {
          "x": 1,
          "y": 64,
          "value": 8,
          "name": "cell2 ~ gene65"
        },
        {
          "x": 1,
          "y": 65,
          "value": 7,
          "name": "cell2 ~ gene66"
        },
        {
          "x": 1,
          "y": 66,
          "value": 0,
          "name": "cell2 ~ gene67"
        },
        {
          "x": 1,
          "y": 67,
          "value": 9,
          "name": "cell2 ~ gene68"
        },
        {
          "x": 1,
          "y": 68,
          "value": 0,
          "name": "cell2 ~ gene69"
        },
        {
          "x": 1,
          "y": 69,
          "value": 0,
          "name": "cell2 ~ gene70"
        },
        {
          "x": 1,
          "y": 70,
          "value": 19,
          "name": "cell2 ~ gene71"
        },
        {
          "x": 1,
          "y": 71,
          "value": 0,
          "name": "cell2 ~ gene72"
        },
        {
          "x": 1,
          "y": 72,
          "value": 0,
          "name": "cell2 ~ gene73"
        },
        {
          "x": 1,
          "y": 73,
          "value": 0,
          "name": "cell2 ~ gene74"
        },
        {
          "x": 1,
          "y": 74,
          "value": 0,
          "name": "cell2 ~ gene75"
        },
        {
          "x": 1,
          "y": 75,
          "value": 6,
          "name": "cell2 ~ gene76"
        },
        {
          "x": 1,
          "y": 76,
          "value": 0,
          "name": "cell2 ~ gene77"
        },
        {
          "x": 1,
          "y": 77,
          "value": 0,
          "name": "cell2 ~ gene78"
        },
        {
          "x": 1,
          "y": 78,
          "value": 0,
          "name": "cell2 ~ gene79"
        },
        {
          "x": 1,
          "y": 79,
          "value": 0,
          "name": "cell2 ~ gene80"
        },
        {
          "x": 1,
          "y": 80,
          "value": 0,
          "name": "cell2 ~ gene81"
        },
        {
          "x": 1,
          "y": 81,
          "value": 0,
          "name": "cell2 ~ gene82"
        },
        {
          "x": 1,
          "y": 82,
          "value": 0,
          "name": "cell2 ~ gene83"
        },
        {
          "x": 1,
          "y": 83,
          "value": 10,
          "name": "cell2 ~ gene84"
        },
        {
          "x": 1,
          "y": 84,
          "value": 5,
          "name": "cell2 ~ gene85"
        },
        {
          "x": 1,
          "y": 85,
          "value": 0,
          "name": "cell2 ~ gene86"
        },
        {
          "x": 1,
          "y": 86,
          "value": 2,
          "name": "cell2 ~ gene87"
        },
        {
          "x": 1,
          "y": 87,
          "value": 0,
          "name": "cell2 ~ gene88"
        },
        {
          "x": 1,
          "y": 88,
          "value": 0,
          "name": "cell2 ~ gene89"
        },
        {
          "x": 1,
          "y": 89,
          "value": 11,
          "name": "cell2 ~ gene90"
        },
        {
          "x": 1,
          "y": 90,
          "value": 0,
          "name": "cell2 ~ gene91"
        },
        {
          "x": 1,
          "y": 91,
          "value": 2,
          "name": "cell2 ~ gene92"
        },
        {
          "x": 1,
          "y": 92,
          "value": 20,
          "name": "cell2 ~ gene93"
        },
        {
          "x": 1,
          "y": 93,
          "value": 0,
          "name": "cell2 ~ gene94"
        },
        {
          "x": 1,
          "y": 94,
          "value": 10,
          "name": "cell2 ~ gene95"
        },
        {
          "x": 1,
          "y": 95,
          "value": 18,
          "name": "cell2 ~ gene96"
        },
        {
          "x": 1,
          "y": 96,
          "value": 0,
          "name": "cell2 ~ gene97"
        },
        {
          "x": 1,
          "y": 97,
          "value": 0,
          "name": "cell2 ~ gene98"
        },
        {
          "x": 1,
          "y": 98,
          "value": 0,
          "name": "cell2 ~ gene99"
        },
        {
          "x": 1,
          "y": 99,
          "value": 0,
          "name": "cell2 ~ gene100"
        },
        {
          "x": 2,
          "y": 0,
          "value": 0,
          "name": "cell3 ~ gene1"
        },
        {
          "x": 2,
          "y": 1,
          "value": 0,
          "name": "cell3 ~ gene2"
        },
        {
          "x": 2,
          "y": 2,
          "value": 31,
          "name": "cell3 ~ gene3"
        },
        {
          "x": 2,
          "y": 3,
          "value": 27,
          "name": "cell3 ~ gene4"
        },
        {
          "x": 2,
          "y": 4,
          "value": 27,
          "name": "cell3 ~ gene5"
        },
        {
          "x": 2,
          "y": 5,
          "value": 27,
          "name": "cell3 ~ gene6"
        },
        {
          "x": 2,
          "y": 6,
          "value": 23,
          "name": "cell3 ~ gene7"
        },
        {
          "x": 2,
          "y": 7,
          "value": 19,
          "name": "cell3 ~ gene8"
        },
        {
          "x": 2,
          "y": 8,
          "value": 4,
          "name": "cell3 ~ gene9"
        },
        {
          "x": 2,
          "y": 9,
          "value": 0,
          "name": "cell3 ~ gene10"
        },
        {
          "x": 2,
          "y": 10,
          "value": 24,
          "name": "cell3 ~ gene11"
        },
        {
          "x": 2,
          "y": 11,
          "value": 4,
          "name": "cell3 ~ gene12"
        },
        {
          "x": 2,
          "y": 12,
          "value": 7,
          "name": "cell3 ~ gene13"
        },
        {
          "x": 2,
          "y": 13,
          "value": 9,
          "name": "cell3 ~ gene14"
        },
        {
          "x": 2,
          "y": 14,
          "value": 40,
          "name": "cell3 ~ gene15"
        },
        {
          "x": 2,
          "y": 15,
          "value": 44,
          "name": "cell3 ~ gene16"
        },
        {
          "x": 2,
          "y": 16,
          "value": 0,
          "name": "cell3 ~ gene17"
        },
        {
          "x": 2,
          "y": 17,
          "value": 22,
          "name": "cell3 ~ gene18"
        },
        {
          "x": 2,
          "y": 18,
          "value": 31,
          "name": "cell3 ~ gene19"
        },
        {
          "x": 2,
          "y": 19,
          "value": 0,
          "name": "cell3 ~ gene20"
        },
        {
          "x": 2,
          "y": 20,
          "value": 14,
          "name": "cell3 ~ gene21"
        },
        {
          "x": 2,
          "y": 21,
          "value": 0,
          "name": "cell3 ~ gene22"
        },
        {
          "x": 2,
          "y": 22,
          "value": 3,
          "name": "cell3 ~ gene23"
        },
        {
          "x": 2,
          "y": 23,
          "value": 0,
          "name": "cell3 ~ gene24"
        },
        {
          "x": 2,
          "y": 24,
          "value": 16,
          "name": "cell3 ~ gene25"
        },
        {
          "x": 2,
          "y": 25,
          "value": 0,
          "name": "cell3 ~ gene26"
        },
        {
          "x": 2,
          "y": 26,
          "value": 0,
          "name": "cell3 ~ gene27"
        },
        {
          "x": 2,
          "y": 27,
          "value": 0,
          "name": "cell3 ~ gene28"
        },
        {
          "x": 2,
          "y": 28,
          "value": 4,
          "name": "cell3 ~ gene29"
        },
        {
          "x": 2,
          "y": 29,
          "value": 22,
          "name": "cell3 ~ gene30"
        },
        {
          "x": 2,
          "y": 30,
          "value": 0,
          "name": "cell3 ~ gene31"
        },
        {
          "x": 2,
          "y": 31,
          "value": 0,
          "name": "cell3 ~ gene32"
        },
        {
          "x": 2,
          "y": 32,
          "value": 0,
          "name": "cell3 ~ gene33"
        },
        {
          "x": 2,
          "y": 33,
          "value": 15,
          "name": "cell3 ~ gene34"
        },
        {
          "x": 2,
          "y": 34,
          "value": 0,
          "name": "cell3 ~ gene35"
        },
        {
          "x": 2,
          "y": 35,
          "value": 5,
          "name": "cell3 ~ gene36"
        },
        {
          "x": 2,
          "y": 36,
          "value": 17,
          "name": "cell3 ~ gene37"
        },
        {
          "x": 2,
          "y": 37,
          "value": 0,
          "name": "cell3 ~ gene38"
        },
        {
          "x": 2,
          "y": 38,
          "value": 0,
          "name": "cell3 ~ gene39"
        },
        {
          "x": 2,
          "y": 39,
          "value": 0,
          "name": "cell3 ~ gene40"
        },
        {
          "x": 2,
          "y": 40,
          "value": 5,
          "name": "cell3 ~ gene41"
        },
        {
          "x": 2,
          "y": 41,
          "value": 0,
          "name": "cell3 ~ gene42"
        },
        {
          "x": 2,
          "y": 42,
          "value": 16,
          "name": "cell3 ~ gene43"
        },
        {
          "x": 2,
          "y": 43,
          "value": 0,
          "name": "cell3 ~ gene44"
        },
        {
          "x": 2,
          "y": 44,
          "value": 0,
          "name": "cell3 ~ gene45"
        },
        {
          "x": 2,
          "y": 45,
          "value": 0,
          "name": "cell3 ~ gene46"
        },
        {
          "x": 2,
          "y": 46,
          "value": 15,
          "name": "cell3 ~ gene47"
        },
        {
          "x": 2,
          "y": 47,
          "value": 0,
          "name": "cell3 ~ gene48"
        },
        {
          "x": 2,
          "y": 48,
          "value": 0,
          "name": "cell3 ~ gene49"
        },
        {
          "x": 2,
          "y": 49,
          "value": 0,
          "name": "cell3 ~ gene50"
        },
        {
          "x": 2,
          "y": 50,
          "value": 0,
          "name": "cell3 ~ gene51"
        },
        {
          "x": 2,
          "y": 51,
          "value": 5,
          "name": "cell3 ~ gene52"
        },
        {
          "x": 2,
          "y": 52,
          "value": 4,
          "name": "cell3 ~ gene53"
        },
        {
          "x": 2,
          "y": 53,
          "value": 19,
          "name": "cell3 ~ gene54"
        },
        {
          "x": 2,
          "y": 54,
          "value": 10,
          "name": "cell3 ~ gene55"
        },
        {
          "x": 2,
          "y": 55,
          "value": 1,
          "name": "cell3 ~ gene56"
        },
        {
          "x": 2,
          "y": 56,
          "value": 0,
          "name": "cell3 ~ gene57"
        },
        {
          "x": 2,
          "y": 57,
          "value": 6,
          "name": "cell3 ~ gene58"
        },
        {
          "x": 2,
          "y": 58,
          "value": 0,
          "name": "cell3 ~ gene59"
        },
        {
          "x": 2,
          "y": 59,
          "value": 1,
          "name": "cell3 ~ gene60"
        },
        {
          "x": 2,
          "y": 60,
          "value": 4,
          "name": "cell3 ~ gene61"
        },
        {
          "x": 2,
          "y": 61,
          "value": 6,
          "name": "cell3 ~ gene62"
        },
        {
          "x": 2,
          "y": 62,
          "value": 0,
          "name": "cell3 ~ gene63"
        },
        {
          "x": 2,
          "y": 63,
          "value": 15,
          "name": "cell3 ~ gene64"
        },
        {
          "x": 2,
          "y": 64,
          "value": 7,
          "name": "cell3 ~ gene65"
        },
        {
          "x": 2,
          "y": 65,
          "value": 25,
          "name": "cell3 ~ gene66"
        },
        {
          "x": 2,
          "y": 66,
          "value": 0,
          "name": "cell3 ~ gene67"
        },
        {
          "x": 2,
          "y": 67,
          "value": 0,
          "name": "cell3 ~ gene68"
        },
        {
          "x": 2,
          "y": 68,
          "value": 0,
          "name": "cell3 ~ gene69"
        },
        {
          "x": 2,
          "y": 69,
          "value": 0,
          "name": "cell3 ~ gene70"
        },
        {
          "x": 2,
          "y": 70,
          "value": 3,
          "name": "cell3 ~ gene71"
        },
        {
          "x": 2,
          "y": 71,
          "value": 17,
          "name": "cell3 ~ gene72"
        },
        {
          "x": 2,
          "y": 72,
          "value": 0,
          "name": "cell3 ~ gene73"
        },
        {
          "x": 2,
          "y": 73,
          "value": 0,
          "name": "cell3 ~ gene74"
        },
        {
          "x": 2,
          "y": 74,
          "value": 0,
          "name": "cell3 ~ gene75"
        },
        {
          "x": 2,
          "y": 75,
          "value": 6,
          "name": "cell3 ~ gene76"
        },
        {
          "x": 2,
          "y": 76,
          "value": 2,
          "name": "cell3 ~ gene77"
        },
        {
          "x": 2,
          "y": 77,
          "value": 13,
          "name": "cell3 ~ gene78"
        },
        {
          "x": 2,
          "y": 78,
          "value": 0,
          "name": "cell3 ~ gene79"
        },
        {
          "x": 2,
          "y": 79,
          "value": 0,
          "name": "cell3 ~ gene80"
        },
        {
          "x": 2,
          "y": 80,
          "value": 10,
          "name": "cell3 ~ gene81"
        },
        {
          "x": 2,
          "y": 81,
          "value": 0,
          "name": "cell3 ~ gene82"
        },
        {
          "x": 2,
          "y": 82,
          "value": 6,
          "name": "cell3 ~ gene83"
        },
        {
          "x": 2,
          "y": 83,
          "value": 5,
          "name": "cell3 ~ gene84"
        },
        {
          "x": 2,
          "y": 84,
          "value": 4,
          "name": "cell3 ~ gene85"
        },
        {
          "x": 2,
          "y": 85,
          "value": 17,
          "name": "cell3 ~ gene86"
        },
        {
          "x": 2,
          "y": 86,
          "value": 0,
          "name": "cell3 ~ gene87"
        },
        {
          "x": 2,
          "y": 87,
          "value": 0,
          "name": "cell3 ~ gene88"
        },
        {
          "x": 2,
          "y": 88,
          "value": 1,
          "name": "cell3 ~ gene89"
        },
        {
          "x": 2,
          "y": 89,
          "value": 0,
          "name": "cell3 ~ gene90"
        },
        {
          "x": 2,
          "y": 90,
          "value": 6,
          "name": "cell3 ~ gene91"
        },
        {
          "x": 2,
          "y": 91,
          "value": 16,
          "name": "cell3 ~ gene92"
        },
        {
          "x": 2,
          "y": 92,
          "value": 0,
          "name": "cell3 ~ gene93"
        },
        {
          "x": 2,
          "y": 93,
          "value": 0,
          "name": "cell3 ~ gene94"
        },
        {
          "x": 2,
          "y": 94,
          "value": 1,
          "name": "cell3 ~ gene95"
        },
        {
          "x": 2,
          "y": 95,
          "value": 5,
          "name": "cell3 ~ gene96"
        },
        {
          "x": 2,
          "y": 96,
          "value": 0,
          "name": "cell3 ~ gene97"
        },
        {
          "x": 2,
          "y": 97,
          "value": 4,
          "name": "cell3 ~ gene98"
        },
        {
          "x": 2,
          "y": 98,
          "value": 4,
          "name": "cell3 ~ gene99"
        },
        {
          "x": 2,
          "y": 99,
          "value": 0,
          "name": "cell3 ~ gene100"
        },
        {
          "x": 3,
          "y": 0,
          "value": 5,
          "name": "cell4 ~ gene1"
        },
        {
          "x": 3,
          "y": 1,
          "value": 14,
          "name": "cell4 ~ gene2"
        },
        {
          "x": 3,
          "y": 2,
          "value": 25,
          "name": "cell4 ~ gene3"
        },
        {
          "x": 3,
          "y": 3,
          "value": 41,
          "name": "cell4 ~ gene4"
        },
        {
          "x": 3,
          "y": 4,
          "value": 17,
          "name": "cell4 ~ gene5"
        },
        {
          "x": 3,
          "y": 5,
          "value": 0,
          "name": "cell4 ~ gene6"
        },
        {
          "x": 3,
          "y": 6,
          "value": 22,
          "name": "cell4 ~ gene7"
        },
        {
          "x": 3,
          "y": 7,
          "value": 12,
          "name": "cell4 ~ gene8"
        },
        {
          "x": 3,
          "y": 8,
          "value": 2,
          "name": "cell4 ~ gene9"
        },
        {
          "x": 3,
          "y": 9,
          "value": 0,
          "name": "cell4 ~ gene10"
        },
        {
          "x": 3,
          "y": 10,
          "value": 0,
          "name": "cell4 ~ gene11"
        },
        {
          "x": 3,
          "y": 11,
          "value": 0,
          "name": "cell4 ~ gene12"
        },
        {
          "x": 3,
          "y": 12,
          "value": 7,
          "name": "cell4 ~ gene13"
        },
        {
          "x": 3,
          "y": 13,
          "value": 15,
          "name": "cell4 ~ gene14"
        },
        {
          "x": 3,
          "y": 14,
          "value": 28,
          "name": "cell4 ~ gene15"
        },
        {
          "x": 3,
          "y": 15,
          "value": 48,
          "name": "cell4 ~ gene16"
        },
        {
          "x": 3,
          "y": 16,
          "value": 10,
          "name": "cell4 ~ gene17"
        },
        {
          "x": 3,
          "y": 17,
          "value": 30,
          "name": "cell4 ~ gene18"
        },
        {
          "x": 3,
          "y": 18,
          "value": 18,
          "name": "cell4 ~ gene19"
        },
        {
          "x": 3,
          "y": 19,
          "value": 0,
          "name": "cell4 ~ gene20"
        },
        {
          "x": 3,
          "y": 20,
          "value": 4,
          "name": "cell4 ~ gene21"
        },
        {
          "x": 3,
          "y": 21,
          "value": 0,
          "name": "cell4 ~ gene22"
        },
        {
          "x": 3,
          "y": 22,
          "value": 0,
          "name": "cell4 ~ gene23"
        },
        {
          "x": 3,
          "y": 23,
          "value": 0,
          "name": "cell4 ~ gene24"
        },
        {
          "x": 3,
          "y": 24,
          "value": 11,
          "name": "cell4 ~ gene25"
        },
        {
          "x": 3,
          "y": 25,
          "value": 16,
          "name": "cell4 ~ gene26"
        },
        {
          "x": 3,
          "y": 26,
          "value": 10,
          "name": "cell4 ~ gene27"
        },
        {
          "x": 3,
          "y": 27,
          "value": 0,
          "name": "cell4 ~ gene28"
        },
        {
          "x": 3,
          "y": 28,
          "value": 0,
          "name": "cell4 ~ gene29"
        },
        {
          "x": 3,
          "y": 29,
          "value": 2,
          "name": "cell4 ~ gene30"
        },
        {
          "x": 3,
          "y": 30,
          "value": 11,
          "name": "cell4 ~ gene31"
        },
        {
          "x": 3,
          "y": 31,
          "value": 8,
          "name": "cell4 ~ gene32"
        },
        {
          "x": 3,
          "y": 32,
          "value": 3,
          "name": "cell4 ~ gene33"
        },
        {
          "x": 3,
          "y": 33,
          "value": 2,
          "name": "cell4 ~ gene34"
        },
        {
          "x": 3,
          "y": 34,
          "value": 0,
          "name": "cell4 ~ gene35"
        },
        {
          "x": 3,
          "y": 35,
          "value": 4,
          "name": "cell4 ~ gene36"
        },
        {
          "x": 3,
          "y": 36,
          "value": 1,
          "name": "cell4 ~ gene37"
        },
        {
          "x": 3,
          "y": 37,
          "value": 1,
          "name": "cell4 ~ gene38"
        },
        {
          "x": 3,
          "y": 38,
          "value": 0,
          "name": "cell4 ~ gene39"
        },
        {
          "x": 3,
          "y": 39,
          "value": 0,
          "name": "cell4 ~ gene40"
        },
        {
          "x": 3,
          "y": 40,
          "value": 0,
          "name": "cell4 ~ gene41"
        },
        {
          "x": 3,
          "y": 41,
          "value": 8,
          "name": "cell4 ~ gene42"
        },
        {
          "x": 3,
          "y": 42,
          "value": 0,
          "name": "cell4 ~ gene43"
        },
        {
          "x": 3,
          "y": 43,
          "value": 0,
          "name": "cell4 ~ gene44"
        },
        {
          "x": 3,
          "y": 44,
          "value": 0,
          "name": "cell4 ~ gene45"
        },
        {
          "x": 3,
          "y": 45,
          "value": 14,
          "name": "cell4 ~ gene46"
        },
        {
          "x": 3,
          "y": 46,
          "value": 0,
          "name": "cell4 ~ gene47"
        },
        {
          "x": 3,
          "y": 47,
          "value": 2,
          "name": "cell4 ~ gene48"
        },
        {
          "x": 3,
          "y": 48,
          "value": 1,
          "name": "cell4 ~ gene49"
        },
        {
          "x": 3,
          "y": 49,
          "value": 0,
          "name": "cell4 ~ gene50"
        },
        {
          "x": 3,
          "y": 50,
          "value": 0,
          "name": "cell4 ~ gene51"
        },
        {
          "x": 3,
          "y": 51,
          "value": 13,
          "name": "cell4 ~ gene52"
        },
        {
          "x": 3,
          "y": 52,
          "value": 5,
          "name": "cell4 ~ gene53"
        },
        {
          "x": 3,
          "y": 53,
          "value": 0,
          "name": "cell4 ~ gene54"
        },
        {
          "x": 3,
          "y": 54,
          "value": 1,
          "name": "cell4 ~ gene55"
        },
        {
          "x": 3,
          "y": 55,
          "value": 6,
          "name": "cell4 ~ gene56"
        },
        {
          "x": 3,
          "y": 56,
          "value": 0,
          "name": "cell4 ~ gene57"
        },
        {
          "x": 3,
          "y": 57,
          "value": 8,
          "name": "cell4 ~ gene58"
        },
        {
          "x": 3,
          "y": 58,
          "value": 0,
          "name": "cell4 ~ gene59"
        },
        {
          "x": 3,
          "y": 59,
          "value": 0,
          "name": "cell4 ~ gene60"
        },
        {
          "x": 3,
          "y": 60,
          "value": 7,
          "name": "cell4 ~ gene61"
        },
        {
          "x": 3,
          "y": 61,
          "value": 3,
          "name": "cell4 ~ gene62"
        },
        {
          "x": 3,
          "y": 62,
          "value": 5,
          "name": "cell4 ~ gene63"
        },
        {
          "x": 3,
          "y": 63,
          "value": 10,
          "name": "cell4 ~ gene64"
        },
        {
          "x": 3,
          "y": 64,
          "value": 2,
          "name": "cell4 ~ gene65"
        },
        {
          "x": 3,
          "y": 65,
          "value": 0,
          "name": "cell4 ~ gene66"
        },
        {
          "x": 3,
          "y": 66,
          "value": 1,
          "name": "cell4 ~ gene67"
        },
        {
          "x": 3,
          "y": 67,
          "value": 0,
          "name": "cell4 ~ gene68"
        },
        {
          "x": 3,
          "y": 68,
          "value": 0,
          "name": "cell4 ~ gene69"
        },
        {
          "x": 3,
          "y": 69,
          "value": 10,
          "name": "cell4 ~ gene70"
        },
        {
          "x": 3,
          "y": 70,
          "value": 0,
          "name": "cell4 ~ gene71"
        },
        {
          "x": 3,
          "y": 71,
          "value": 1,
          "name": "cell4 ~ gene72"
        },
        {
          "x": 3,
          "y": 72,
          "value": 0,
          "name": "cell4 ~ gene73"
        },
        {
          "x": 3,
          "y": 73,
          "value": 0,
          "name": "cell4 ~ gene74"
        },
        {
          "x": 3,
          "y": 74,
          "value": 10,
          "name": "cell4 ~ gene75"
        },
        {
          "x": 3,
          "y": 75,
          "value": 5,
          "name": "cell4 ~ gene76"
        },
        {
          "x": 3,
          "y": 76,
          "value": 13,
          "name": "cell4 ~ gene77"
        },
        {
          "x": 3,
          "y": 77,
          "value": 0,
          "name": "cell4 ~ gene78"
        },
        {
          "x": 3,
          "y": 78,
          "value": 0,
          "name": "cell4 ~ gene79"
        },
        {
          "x": 3,
          "y": 79,
          "value": 0,
          "name": "cell4 ~ gene80"
        },
        {
          "x": 3,
          "y": 80,
          "value": 0,
          "name": "cell4 ~ gene81"
        },
        {
          "x": 3,
          "y": 81,
          "value": 0,
          "name": "cell4 ~ gene82"
        },
        {
          "x": 3,
          "y": 82,
          "value": 0,
          "name": "cell4 ~ gene83"
        },
        {
          "x": 3,
          "y": 83,
          "value": 0,
          "name": "cell4 ~ gene84"
        },
        {
          "x": 3,
          "y": 84,
          "value": 6,
          "name": "cell4 ~ gene85"
        },
        {
          "x": 3,
          "y": 85,
          "value": 5,
          "name": "cell4 ~ gene86"
        },
        {
          "x": 3,
          "y": 86,
          "value": 9,
          "name": "cell4 ~ gene87"
        },
        {
          "x": 3,
          "y": 87,
          "value": 27,
          "name": "cell4 ~ gene88"
        },
        {
          "x": 3,
          "y": 88,
          "value": 0,
          "name": "cell4 ~ gene89"
        },
        {
          "x": 3,
          "y": 89,
          "value": 7,
          "name": "cell4 ~ gene90"
        },
        {
          "x": 3,
          "y": 90,
          "value": 33,
          "name": "cell4 ~ gene91"
        },
        {
          "x": 3,
          "y": 91,
          "value": 6,
          "name": "cell4 ~ gene92"
        },
        {
          "x": 3,
          "y": 92,
          "value": 0,
          "name": "cell4 ~ gene93"
        },
        {
          "x": 3,
          "y": 93,
          "value": 0,
          "name": "cell4 ~ gene94"
        },
        {
          "x": 3,
          "y": 94,
          "value": 0,
          "name": "cell4 ~ gene95"
        },
        {
          "x": 3,
          "y": 95,
          "value": 0,
          "name": "cell4 ~ gene96"
        },
        {
          "x": 3,
          "y": 96,
          "value": 0,
          "name": "cell4 ~ gene97"
        },
        {
          "x": 3,
          "y": 97,
          "value": 0,
          "name": "cell4 ~ gene98"
        },
        {
          "x": 3,
          "y": 98,
          "value": 3,
          "name": "cell4 ~ gene99"
        },
        {
          "x": 3,
          "y": 99,
          "value": 0,
          "name": "cell4 ~ gene100"
        },
        {
          "x": 4,
          "y": 0,
          "value": 2,
          "name": "cell5 ~ gene1"
        },
        {
          "x": 4,
          "y": 1,
          "value": 0,
          "name": "cell5 ~ gene2"
        },
        {
          "x": 4,
          "y": 2,
          "value": 25,
          "name": "cell5 ~ gene3"
        },
        {
          "x": 4,
          "y": 3,
          "value": 26,
          "name": "cell5 ~ gene4"
        },
        {
          "x": 4,
          "y": 4,
          "value": 35,
          "name": "cell5 ~ gene5"
        },
        {
          "x": 4,
          "y": 5,
          "value": 0,
          "name": "cell5 ~ gene6"
        },
        {
          "x": 4,
          "y": 6,
          "value": 23,
          "name": "cell5 ~ gene7"
        },
        {
          "x": 4,
          "y": 7,
          "value": 28,
          "name": "cell5 ~ gene8"
        },
        {
          "x": 4,
          "y": 8,
          "value": 6,
          "name": "cell5 ~ gene9"
        },
        {
          "x": 4,
          "y": 9,
          "value": 13,
          "name": "cell5 ~ gene10"
        },
        {
          "x": 4,
          "y": 10,
          "value": 20,
          "name": "cell5 ~ gene11"
        },
        {
          "x": 4,
          "y": 11,
          "value": 5,
          "name": "cell5 ~ gene12"
        },
        {
          "x": 4,
          "y": 12,
          "value": 27,
          "name": "cell5 ~ gene13"
        },
        {
          "x": 4,
          "y": 13,
          "value": 0,
          "name": "cell5 ~ gene14"
        },
        {
          "x": 4,
          "y": 14,
          "value": 21,
          "name": "cell5 ~ gene15"
        },
        {
          "x": 4,
          "y": 15,
          "value": 38,
          "name": "cell5 ~ gene16"
        },
        {
          "x": 4,
          "y": 16,
          "value": 26,
          "name": "cell5 ~ gene17"
        },
        {
          "x": 4,
          "y": 17,
          "value": 29,
          "name": "cell5 ~ gene18"
        },
        {
          "x": 4,
          "y": 18,
          "value": 24,
          "name": "cell5 ~ gene19"
        },
        {
          "x": 4,
          "y": 19,
          "value": 15,
          "name": "cell5 ~ gene20"
        },
        {
          "x": 4,
          "y": 20,
          "value": 0,
          "name": "cell5 ~ gene21"
        },
        {
          "x": 4,
          "y": 21,
          "value": 0,
          "name": "cell5 ~ gene22"
        },
        {
          "x": 4,
          "y": 22,
          "value": 0,
          "name": "cell5 ~ gene23"
        },
        {
          "x": 4,
          "y": 23,
          "value": 0,
          "name": "cell5 ~ gene24"
        },
        {
          "x": 4,
          "y": 24,
          "value": 4,
          "name": "cell5 ~ gene25"
        },
        {
          "x": 4,
          "y": 25,
          "value": 0,
          "name": "cell5 ~ gene26"
        },
        {
          "x": 4,
          "y": 26,
          "value": 7,
          "name": "cell5 ~ gene27"
        },
        {
          "x": 4,
          "y": 27,
          "value": 21,
          "name": "cell5 ~ gene28"
        },
        {
          "x": 4,
          "y": 28,
          "value": 0,
          "name": "cell5 ~ gene29"
        },
        {
          "x": 4,
          "y": 29,
          "value": 6,
          "name": "cell5 ~ gene30"
        },
        {
          "x": 4,
          "y": 30,
          "value": 0,
          "name": "cell5 ~ gene31"
        },
        {
          "x": 4,
          "y": 31,
          "value": 0,
          "name": "cell5 ~ gene32"
        },
        {
          "x": 4,
          "y": 32,
          "value": 0,
          "name": "cell5 ~ gene33"
        },
        {
          "x": 4,
          "y": 33,
          "value": 0,
          "name": "cell5 ~ gene34"
        },
        {
          "x": 4,
          "y": 34,
          "value": 3,
          "name": "cell5 ~ gene35"
        },
        {
          "x": 4,
          "y": 35,
          "value": 0,
          "name": "cell5 ~ gene36"
        },
        {
          "x": 4,
          "y": 36,
          "value": 19,
          "name": "cell5 ~ gene37"
        },
        {
          "x": 4,
          "y": 37,
          "value": 0,
          "name": "cell5 ~ gene38"
        },
        {
          "x": 4,
          "y": 38,
          "value": 0,
          "name": "cell5 ~ gene39"
        },
        {
          "x": 4,
          "y": 39,
          "value": 0,
          "name": "cell5 ~ gene40"
        },
        {
          "x": 4,
          "y": 40,
          "value": 24,
          "name": "cell5 ~ gene41"
        },
        {
          "x": 4,
          "y": 41,
          "value": 16,
          "name": "cell5 ~ gene42"
        },
        {
          "x": 4,
          "y": 42,
          "value": 13,
          "name": "cell5 ~ gene43"
        },
        {
          "x": 4,
          "y": 43,
          "value": 8,
          "name": "cell5 ~ gene44"
        },
        {
          "x": 4,
          "y": 44,
          "value": 5,
          "name": "cell5 ~ gene45"
        },
        {
          "x": 4,
          "y": 45,
          "value": 0,
          "name": "cell5 ~ gene46"
        },
        {
          "x": 4,
          "y": 46,
          "value": 0,
          "name": "cell5 ~ gene47"
        },
        {
          "x": 4,
          "y": 47,
          "value": 0,
          "name": "cell5 ~ gene48"
        },
        {
          "x": 4,
          "y": 48,
          "value": 0,
          "name": "cell5 ~ gene49"
        },
        {
          "x": 4,
          "y": 49,
          "value": 0,
          "name": "cell5 ~ gene50"
        },
        {
          "x": 4,
          "y": 50,
          "value": 0,
          "name": "cell5 ~ gene51"
        },
        {
          "x": 4,
          "y": 51,
          "value": 7,
          "name": "cell5 ~ gene52"
        },
        {
          "x": 4,
          "y": 52,
          "value": 0,
          "name": "cell5 ~ gene53"
        },
        {
          "x": 4,
          "y": 53,
          "value": 0,
          "name": "cell5 ~ gene54"
        },
        {
          "x": 4,
          "y": 54,
          "value": 0,
          "name": "cell5 ~ gene55"
        },
        {
          "x": 4,
          "y": 55,
          "value": 0,
          "name": "cell5 ~ gene56"
        },
        {
          "x": 4,
          "y": 56,
          "value": 8,
          "name": "cell5 ~ gene57"
        },
        {
          "x": 4,
          "y": 57,
          "value": 0,
          "name": "cell5 ~ gene58"
        },
        {
          "x": 4,
          "y": 58,
          "value": 0,
          "name": "cell5 ~ gene59"
        },
        {
          "x": 4,
          "y": 59,
          "value": 6,
          "name": "cell5 ~ gene60"
        },
        {
          "x": 4,
          "y": 60,
          "value": 12,
          "name": "cell5 ~ gene61"
        },
        {
          "x": 4,
          "y": 61,
          "value": 19,
          "name": "cell5 ~ gene62"
        },
        {
          "x": 4,
          "y": 62,
          "value": 0,
          "name": "cell5 ~ gene63"
        },
        {
          "x": 4,
          "y": 63,
          "value": 9,
          "name": "cell5 ~ gene64"
        },
        {
          "x": 4,
          "y": 64,
          "value": 0,
          "name": "cell5 ~ gene65"
        },
        {
          "x": 4,
          "y": 65,
          "value": 0,
          "name": "cell5 ~ gene66"
        },
        {
          "x": 4,
          "y": 66,
          "value": 14,
          "name": "cell5 ~ gene67"
        },
        {
          "x": 4,
          "y": 67,
          "value": 0,
          "name": "cell5 ~ gene68"
        },
        {
          "x": 4,
          "y": 68,
          "value": 7,
          "name": "cell5 ~ gene69"
        },
        {
          "x": 4,
          "y": 69,
          "value": 0,
          "name": "cell5 ~ gene70"
        },
        {
          "x": 4,
          "y": 70,
          "value": 0,
          "name": "cell5 ~ gene71"
        },
        {
          "x": 4,
          "y": 71,
          "value": 14,
          "name": "cell5 ~ gene72"
        },
        {
          "x": 4,
          "y": 72,
          "value": 0,
          "name": "cell5 ~ gene73"
        },
        {
          "x": 4,
          "y": 73,
          "value": 0,
          "name": "cell5 ~ gene74"
        },
        {
          "x": 4,
          "y": 74,
          "value": 7,
          "name": "cell5 ~ gene75"
        },
        {
          "x": 4,
          "y": 75,
          "value": 9,
          "name": "cell5 ~ gene76"
        },
        {
          "x": 4,
          "y": 76,
          "value": 0,
          "name": "cell5 ~ gene77"
        },
        {
          "x": 4,
          "y": 77,
          "value": 11,
          "name": "cell5 ~ gene78"
        },
        {
          "x": 4,
          "y": 78,
          "value": 0,
          "name": "cell5 ~ gene79"
        },
        {
          "x": 4,
          "y": 79,
          "value": 0,
          "name": "cell5 ~ gene80"
        },
        {
          "x": 4,
          "y": 80,
          "value": 0,
          "name": "cell5 ~ gene81"
        },
        {
          "x": 4,
          "y": 81,
          "value": 0,
          "name": "cell5 ~ gene82"
        },
        {
          "x": 4,
          "y": 82,
          "value": 3,
          "name": "cell5 ~ gene83"
        },
        {
          "x": 4,
          "y": 83,
          "value": 0,
          "name": "cell5 ~ gene84"
        },
        {
          "x": 4,
          "y": 84,
          "value": 12,
          "name": "cell5 ~ gene85"
        },
        {
          "x": 4,
          "y": 85,
          "value": 6,
          "name": "cell5 ~ gene86"
        },
        {
          "x": 4,
          "y": 86,
          "value": 1,
          "name": "cell5 ~ gene87"
        },
        {
          "x": 4,
          "y": 87,
          "value": 18,
          "name": "cell5 ~ gene88"
        },
        {
          "x": 4,
          "y": 88,
          "value": 0,
          "name": "cell5 ~ gene89"
        },
        {
          "x": 4,
          "y": 89,
          "value": 3,
          "name": "cell5 ~ gene90"
        },
        {
          "x": 4,
          "y": 90,
          "value": 8,
          "name": "cell5 ~ gene91"
        },
        {
          "x": 4,
          "y": 91,
          "value": 0,
          "name": "cell5 ~ gene92"
        },
        {
          "x": 4,
          "y": 92,
          "value": 0,
          "name": "cell5 ~ gene93"
        },
        {
          "x": 4,
          "y": 93,
          "value": 0,
          "name": "cell5 ~ gene94"
        },
        {
          "x": 4,
          "y": 94,
          "value": 0,
          "name": "cell5 ~ gene95"
        },
        {
          "x": 4,
          "y": 95,
          "value": 1,
          "name": "cell5 ~ gene96"
        },
        {
          "x": 4,
          "y": 96,
          "value": 0,
          "name": "cell5 ~ gene97"
        },
        {
          "x": 4,
          "y": 97,
          "value": 0,
          "name": "cell5 ~ gene98"
        },
        {
          "x": 4,
          "y": 98,
          "value": 8,
          "name": "cell5 ~ gene99"
        },
        {
          "x": 4,
          "y": 99,
          "value": 0,
          "name": "cell5 ~ gene100"
        },
        {
          "x": 5,
          "y": 0,
          "value": 0,
          "name": "cell6 ~ gene1"
        },
        {
          "x": 5,
          "y": 1,
          "value": 4,
          "name": "cell6 ~ gene2"
        },
        {
          "x": 5,
          "y": 2,
          "value": 35,
          "name": "cell6 ~ gene3"
        },
        {
          "x": 5,
          "y": 3,
          "value": 4,
          "name": "cell6 ~ gene4"
        },
        {
          "x": 5,
          "y": 4,
          "value": 21,
          "name": "cell6 ~ gene5"
        },
        {
          "x": 5,
          "y": 5,
          "value": 16,
          "name": "cell6 ~ gene6"
        },
        {
          "x": 5,
          "y": 6,
          "value": 26,
          "name": "cell6 ~ gene7"
        },
        {
          "x": 5,
          "y": 7,
          "value": 37,
          "name": "cell6 ~ gene8"
        },
        {
          "x": 5,
          "y": 8,
          "value": 17,
          "name": "cell6 ~ gene9"
        },
        {
          "x": 5,
          "y": 9,
          "value": 0,
          "name": "cell6 ~ gene10"
        },
        {
          "x": 5,
          "y": 10,
          "value": 8,
          "name": "cell6 ~ gene11"
        },
        {
          "x": 5,
          "y": 11,
          "value": 13,
          "name": "cell6 ~ gene12"
        },
        {
          "x": 5,
          "y": 12,
          "value": 28,
          "name": "cell6 ~ gene13"
        },
        {
          "x": 5,
          "y": 13,
          "value": 19,
          "name": "cell6 ~ gene14"
        },
        {
          "x": 5,
          "y": 14,
          "value": 19,
          "name": "cell6 ~ gene15"
        },
        {
          "x": 5,
          "y": 15,
          "value": 38,
          "name": "cell6 ~ gene16"
        },
        {
          "x": 5,
          "y": 16,
          "value": 12,
          "name": "cell6 ~ gene17"
        },
        {
          "x": 5,
          "y": 17,
          "value": 39,
          "name": "cell6 ~ gene18"
        },
        {
          "x": 5,
          "y": 18,
          "value": 14,
          "name": "cell6 ~ gene19"
        },
        {
          "x": 5,
          "y": 19,
          "value": 4,
          "name": "cell6 ~ gene20"
        },
        {
          "x": 5,
          "y": 20,
          "value": 0,
          "name": "cell6 ~ gene21"
        },
        {
          "x": 5,
          "y": 21,
          "value": 0,
          "name": "cell6 ~ gene22"
        },
        {
          "x": 5,
          "y": 22,
          "value": 0,
          "name": "cell6 ~ gene23"
        },
        {
          "x": 5,
          "y": 23,
          "value": 0,
          "name": "cell6 ~ gene24"
        },
        {
          "x": 5,
          "y": 24,
          "value": 0,
          "name": "cell6 ~ gene25"
        },
        {
          "x": 5,
          "y": 25,
          "value": 15,
          "name": "cell6 ~ gene26"
        },
        {
          "x": 5,
          "y": 26,
          "value": 1,
          "name": "cell6 ~ gene27"
        },
        {
          "x": 5,
          "y": 27,
          "value": 13,
          "name": "cell6 ~ gene28"
        },
        {
          "x": 5,
          "y": 28,
          "value": 0,
          "name": "cell6 ~ gene29"
        },
        {
          "x": 5,
          "y": 29,
          "value": 0,
          "name": "cell6 ~ gene30"
        },
        {
          "x": 5,
          "y": 30,
          "value": 0,
          "name": "cell6 ~ gene31"
        },
        {
          "x": 5,
          "y": 31,
          "value": 14,
          "name": "cell6 ~ gene32"
        },
        {
          "x": 5,
          "y": 32,
          "value": 0,
          "name": "cell6 ~ gene33"
        },
        {
          "x": 5,
          "y": 33,
          "value": 15,
          "name": "cell6 ~ gene34"
        },
        {
          "x": 5,
          "y": 34,
          "value": 0,
          "name": "cell6 ~ gene35"
        },
        {
          "x": 5,
          "y": 35,
          "value": 0,
          "name": "cell6 ~ gene36"
        },
        {
          "x": 5,
          "y": 36,
          "value": 0,
          "name": "cell6 ~ gene37"
        },
        {
          "x": 5,
          "y": 37,
          "value": 0,
          "name": "cell6 ~ gene38"
        },
        {
          "x": 5,
          "y": 38,
          "value": 0,
          "name": "cell6 ~ gene39"
        },
        {
          "x": 5,
          "y": 39,
          "value": 0,
          "name": "cell6 ~ gene40"
        },
        {
          "x": 5,
          "y": 40,
          "value": 0,
          "name": "cell6 ~ gene41"
        },
        {
          "x": 5,
          "y": 41,
          "value": 0,
          "name": "cell6 ~ gene42"
        },
        {
          "x": 5,
          "y": 42,
          "value": 0,
          "name": "cell6 ~ gene43"
        },
        {
          "x": 5,
          "y": 43,
          "value": 9,
          "name": "cell6 ~ gene44"
        },
        {
          "x": 5,
          "y": 44,
          "value": 0,
          "name": "cell6 ~ gene45"
        },
        {
          "x": 5,
          "y": 45,
          "value": 0,
          "name": "cell6 ~ gene46"
        },
        {
          "x": 5,
          "y": 46,
          "value": 6,
          "name": "cell6 ~ gene47"
        },
        {
          "x": 5,
          "y": 47,
          "value": 0,
          "name": "cell6 ~ gene48"
        },
        {
          "x": 5,
          "y": 48,
          "value": 7,
          "name": "cell6 ~ gene49"
        },
        {
          "x": 5,
          "y": 49,
          "value": 1,
          "name": "cell6 ~ gene50"
        },
        {
          "x": 5,
          "y": 50,
          "value": 15,
          "name": "cell6 ~ gene51"
        },
        {
          "x": 5,
          "y": 51,
          "value": 0,
          "name": "cell6 ~ gene52"
        },
        {
          "x": 5,
          "y": 52,
          "value": 0,
          "name": "cell6 ~ gene53"
        },
        {
          "x": 5,
          "y": 53,
          "value": 4,
          "name": "cell6 ~ gene54"
        },
        {
          "x": 5,
          "y": 54,
          "value": 0,
          "name": "cell6 ~ gene55"
        },
        {
          "x": 5,
          "y": 55,
          "value": 0,
          "name": "cell6 ~ gene56"
        },
        {
          "x": 5,
          "y": 56,
          "value": 5,
          "name": "cell6 ~ gene57"
        },
        {
          "x": 5,
          "y": 57,
          "value": 5,
          "name": "cell6 ~ gene58"
        },
        {
          "x": 5,
          "y": 58,
          "value": 0,
          "name": "cell6 ~ gene59"
        },
        {
          "x": 5,
          "y": 59,
          "value": 6,
          "name": "cell6 ~ gene60"
        },
        {
          "x": 5,
          "y": 60,
          "value": 8,
          "name": "cell6 ~ gene61"
        },
        {
          "x": 5,
          "y": 61,
          "value": 8,
          "name": "cell6 ~ gene62"
        },
        {
          "x": 5,
          "y": 62,
          "value": 10,
          "name": "cell6 ~ gene63"
        },
        {
          "x": 5,
          "y": 63,
          "value": 13,
          "name": "cell6 ~ gene64"
        },
        {
          "x": 5,
          "y": 64,
          "value": 0,
          "name": "cell6 ~ gene65"
        },
        {
          "x": 5,
          "y": 65,
          "value": 0,
          "name": "cell6 ~ gene66"
        },
        {
          "x": 5,
          "y": 66,
          "value": 0,
          "name": "cell6 ~ gene67"
        },
        {
          "x": 5,
          "y": 67,
          "value": 0,
          "name": "cell6 ~ gene68"
        },
        {
          "x": 5,
          "y": 68,
          "value": 0,
          "name": "cell6 ~ gene69"
        },
        {
          "x": 5,
          "y": 69,
          "value": 4,
          "name": "cell6 ~ gene70"
        },
        {
          "x": 5,
          "y": 70,
          "value": 0,
          "name": "cell6 ~ gene71"
        },
        {
          "x": 5,
          "y": 71,
          "value": 0,
          "name": "cell6 ~ gene72"
        },
        {
          "x": 5,
          "y": 72,
          "value": 0,
          "name": "cell6 ~ gene73"
        },
        {
          "x": 5,
          "y": 73,
          "value": 1,
          "name": "cell6 ~ gene74"
        },
        {
          "x": 5,
          "y": 74,
          "value": 0,
          "name": "cell6 ~ gene75"
        },
        {
          "x": 5,
          "y": 75,
          "value": 0,
          "name": "cell6 ~ gene76"
        },
        {
          "x": 5,
          "y": 76,
          "value": 0,
          "name": "cell6 ~ gene77"
        },
        {
          "x": 5,
          "y": 77,
          "value": 11,
          "name": "cell6 ~ gene78"
        },
        {
          "x": 5,
          "y": 78,
          "value": 0,
          "name": "cell6 ~ gene79"
        },
        {
          "x": 5,
          "y": 79,
          "value": 0,
          "name": "cell6 ~ gene80"
        },
        {
          "x": 5,
          "y": 80,
          "value": 0,
          "name": "cell6 ~ gene81"
        },
        {
          "x": 5,
          "y": 81,
          "value": 0,
          "name": "cell6 ~ gene82"
        },
        {
          "x": 5,
          "y": 82,
          "value": 0,
          "name": "cell6 ~ gene83"
        },
        {
          "x": 5,
          "y": 83,
          "value": 0,
          "name": "cell6 ~ gene84"
        },
        {
          "x": 5,
          "y": 84,
          "value": 0,
          "name": "cell6 ~ gene85"
        },
        {
          "x": 5,
          "y": 85,
          "value": 0,
          "name": "cell6 ~ gene86"
        },
        {
          "x": 5,
          "y": 86,
          "value": 7,
          "name": "cell6 ~ gene87"
        },
        {
          "x": 5,
          "y": 87,
          "value": 0,
          "name": "cell6 ~ gene88"
        },
        {
          "x": 5,
          "y": 88,
          "value": 0,
          "name": "cell6 ~ gene89"
        },
        {
          "x": 5,
          "y": 89,
          "value": 8,
          "name": "cell6 ~ gene90"
        },
        {
          "x": 5,
          "y": 90,
          "value": 0,
          "name": "cell6 ~ gene91"
        },
        {
          "x": 5,
          "y": 91,
          "value": 0,
          "name": "cell6 ~ gene92"
        },
        {
          "x": 5,
          "y": 92,
          "value": 0,
          "name": "cell6 ~ gene93"
        },
        {
          "x": 5,
          "y": 93,
          "value": 0,
          "name": "cell6 ~ gene94"
        },
        {
          "x": 5,
          "y": 94,
          "value": 3,
          "name": "cell6 ~ gene95"
        },
        {
          "x": 5,
          "y": 95,
          "value": 0,
          "name": "cell6 ~ gene96"
        },
        {
          "x": 5,
          "y": 96,
          "value": 15,
          "name": "cell6 ~ gene97"
        },
        {
          "x": 5,
          "y": 97,
          "value": 4,
          "name": "cell6 ~ gene98"
        },
        {
          "x": 5,
          "y": 98,
          "value": 17,
          "name": "cell6 ~ gene99"
        },
        {
          "x": 5,
          "y": 99,
          "value": 6,
          "name": "cell6 ~ gene100"
        },
        {
          "x": 6,
          "y": 0,
          "value": 0,
          "name": "cell7 ~ gene1"
        },
        {
          "x": 6,
          "y": 1,
          "value": 6,
          "name": "cell7 ~ gene2"
        },
        {
          "x": 6,
          "y": 2,
          "value": 23,
          "name": "cell7 ~ gene3"
        },
        {
          "x": 6,
          "y": 3,
          "value": 31,
          "name": "cell7 ~ gene4"
        },
        {
          "x": 6,
          "y": 4,
          "value": 30,
          "name": "cell7 ~ gene5"
        },
        {
          "x": 6,
          "y": 5,
          "value": 16,
          "name": "cell7 ~ gene6"
        },
        {
          "x": 6,
          "y": 6,
          "value": 32,
          "name": "cell7 ~ gene7"
        },
        {
          "x": 6,
          "y": 7,
          "value": 5,
          "name": "cell7 ~ gene8"
        },
        {
          "x": 6,
          "y": 8,
          "value": 20,
          "name": "cell7 ~ gene9"
        },
        {
          "x": 6,
          "y": 9,
          "value": 0,
          "name": "cell7 ~ gene10"
        },
        {
          "x": 6,
          "y": 10,
          "value": 0,
          "name": "cell7 ~ gene11"
        },
        {
          "x": 6,
          "y": 11,
          "value": 0,
          "name": "cell7 ~ gene12"
        },
        {
          "x": 6,
          "y": 12,
          "value": 27,
          "name": "cell7 ~ gene13"
        },
        {
          "x": 6,
          "y": 13,
          "value": 1,
          "name": "cell7 ~ gene14"
        },
        {
          "x": 6,
          "y": 14,
          "value": 21,
          "name": "cell7 ~ gene15"
        },
        {
          "x": 6,
          "y": 15,
          "value": 33,
          "name": "cell7 ~ gene16"
        },
        {
          "x": 6,
          "y": 16,
          "value": 8,
          "name": "cell7 ~ gene17"
        },
        {
          "x": 6,
          "y": 17,
          "value": 0,
          "name": "cell7 ~ gene18"
        },
        {
          "x": 6,
          "y": 18,
          "value": 34,
          "name": "cell7 ~ gene19"
        },
        {
          "x": 6,
          "y": 19,
          "value": 3,
          "name": "cell7 ~ gene20"
        },
        {
          "x": 6,
          "y": 20,
          "value": 5,
          "name": "cell7 ~ gene21"
        },
        {
          "x": 6,
          "y": 21,
          "value": 0,
          "name": "cell7 ~ gene22"
        },
        {
          "x": 6,
          "y": 22,
          "value": 0,
          "name": "cell7 ~ gene23"
        },
        {
          "x": 6,
          "y": 23,
          "value": 1,
          "name": "cell7 ~ gene24"
        },
        {
          "x": 6,
          "y": 24,
          "value": 8,
          "name": "cell7 ~ gene25"
        },
        {
          "x": 6,
          "y": 25,
          "value": 0,
          "name": "cell7 ~ gene26"
        },
        {
          "x": 6,
          "y": 26,
          "value": 0,
          "name": "cell7 ~ gene27"
        },
        {
          "x": 6,
          "y": 27,
          "value": 13,
          "name": "cell7 ~ gene28"
        },
        {
          "x": 6,
          "y": 28,
          "value": 0,
          "name": "cell7 ~ gene29"
        },
        {
          "x": 6,
          "y": 29,
          "value": 0,
          "name": "cell7 ~ gene30"
        },
        {
          "x": 6,
          "y": 30,
          "value": 0,
          "name": "cell7 ~ gene31"
        },
        {
          "x": 6,
          "y": 31,
          "value": 7,
          "name": "cell7 ~ gene32"
        },
        {
          "x": 6,
          "y": 32,
          "value": 6,
          "name": "cell7 ~ gene33"
        },
        {
          "x": 6,
          "y": 33,
          "value": 0,
          "name": "cell7 ~ gene34"
        },
        {
          "x": 6,
          "y": 34,
          "value": 16,
          "name": "cell7 ~ gene35"
        },
        {
          "x": 6,
          "y": 35,
          "value": 0,
          "name": "cell7 ~ gene36"
        },
        {
          "x": 6,
          "y": 36,
          "value": 0,
          "name": "cell7 ~ gene37"
        },
        {
          "x": 6,
          "y": 37,
          "value": 11,
          "name": "cell7 ~ gene38"
        },
        {
          "x": 6,
          "y": 38,
          "value": 5,
          "name": "cell7 ~ gene39"
        },
        {
          "x": 6,
          "y": 39,
          "value": 9,
          "name": "cell7 ~ gene40"
        },
        {
          "x": 6,
          "y": 40,
          "value": 15,
          "name": "cell7 ~ gene41"
        },
        {
          "x": 6,
          "y": 41,
          "value": 2,
          "name": "cell7 ~ gene42"
        },
        {
          "x": 6,
          "y": 42,
          "value": 6,
          "name": "cell7 ~ gene43"
        },
        {
          "x": 6,
          "y": 43,
          "value": 0,
          "name": "cell7 ~ gene44"
        },
        {
          "x": 6,
          "y": 44,
          "value": 5,
          "name": "cell7 ~ gene45"
        },
        {
          "x": 6,
          "y": 45,
          "value": 0,
          "name": "cell7 ~ gene46"
        },
        {
          "x": 6,
          "y": 46,
          "value": 13,
          "name": "cell7 ~ gene47"
        },
        {
          "x": 6,
          "y": 47,
          "value": 0,
          "name": "cell7 ~ gene48"
        },
        {
          "x": 6,
          "y": 48,
          "value": 0,
          "name": "cell7 ~ gene49"
        },
        {
          "x": 6,
          "y": 49,
          "value": 0,
          "name": "cell7 ~ gene50"
        },
        {
          "x": 6,
          "y": 50,
          "value": 0,
          "name": "cell7 ~ gene51"
        },
        {
          "x": 6,
          "y": 51,
          "value": 0,
          "name": "cell7 ~ gene52"
        },
        {
          "x": 6,
          "y": 52,
          "value": 20,
          "name": "cell7 ~ gene53"
        },
        {
          "x": 6,
          "y": 53,
          "value": 7,
          "name": "cell7 ~ gene54"
        },
        {
          "x": 6,
          "y": 54,
          "value": 0,
          "name": "cell7 ~ gene55"
        },
        {
          "x": 6,
          "y": 55,
          "value": 0,
          "name": "cell7 ~ gene56"
        },
        {
          "x": 6,
          "y": 56,
          "value": 0,
          "name": "cell7 ~ gene57"
        },
        {
          "x": 6,
          "y": 57,
          "value": 4,
          "name": "cell7 ~ gene58"
        },
        {
          "x": 6,
          "y": 58,
          "value": 8,
          "name": "cell7 ~ gene59"
        },
        {
          "x": 6,
          "y": 59,
          "value": 0,
          "name": "cell7 ~ gene60"
        },
        {
          "x": 6,
          "y": 60,
          "value": 0,
          "name": "cell7 ~ gene61"
        },
        {
          "x": 6,
          "y": 61,
          "value": 0,
          "name": "cell7 ~ gene62"
        },
        {
          "x": 6,
          "y": 62,
          "value": 0,
          "name": "cell7 ~ gene63"
        },
        {
          "x": 6,
          "y": 63,
          "value": 5,
          "name": "cell7 ~ gene64"
        },
        {
          "x": 6,
          "y": 64,
          "value": 0,
          "name": "cell7 ~ gene65"
        },
        {
          "x": 6,
          "y": 65,
          "value": 10,
          "name": "cell7 ~ gene66"
        },
        {
          "x": 6,
          "y": 66,
          "value": 12,
          "name": "cell7 ~ gene67"
        },
        {
          "x": 6,
          "y": 67,
          "value": 0,
          "name": "cell7 ~ gene68"
        },
        {
          "x": 6,
          "y": 68,
          "value": 3,
          "name": "cell7 ~ gene69"
        },
        {
          "x": 6,
          "y": 69,
          "value": 13,
          "name": "cell7 ~ gene70"
        },
        {
          "x": 6,
          "y": 70,
          "value": 0,
          "name": "cell7 ~ gene71"
        },
        {
          "x": 6,
          "y": 71,
          "value": 0,
          "name": "cell7 ~ gene72"
        },
        {
          "x": 6,
          "y": 72,
          "value": 0,
          "name": "cell7 ~ gene73"
        },
        {
          "x": 6,
          "y": 73,
          "value": 0,
          "name": "cell7 ~ gene74"
        },
        {
          "x": 6,
          "y": 74,
          "value": 0,
          "name": "cell7 ~ gene75"
        },
        {
          "x": 6,
          "y": 75,
          "value": 4,
          "name": "cell7 ~ gene76"
        },
        {
          "x": 6,
          "y": 76,
          "value": 0,
          "name": "cell7 ~ gene77"
        },
        {
          "x": 6,
          "y": 77,
          "value": 0,
          "name": "cell7 ~ gene78"
        },
        {
          "x": 6,
          "y": 78,
          "value": 2,
          "name": "cell7 ~ gene79"
        },
        {
          "x": 6,
          "y": 79,
          "value": 0,
          "name": "cell7 ~ gene80"
        },
        {
          "x": 6,
          "y": 80,
          "value": 14,
          "name": "cell7 ~ gene81"
        },
        {
          "x": 6,
          "y": 81,
          "value": 13,
          "name": "cell7 ~ gene82"
        },
        {
          "x": 6,
          "y": 82,
          "value": 0,
          "name": "cell7 ~ gene83"
        },
        {
          "x": 6,
          "y": 83,
          "value": 0,
          "name": "cell7 ~ gene84"
        },
        {
          "x": 6,
          "y": 84,
          "value": 0,
          "name": "cell7 ~ gene85"
        },
        {
          "x": 6,
          "y": 85,
          "value": 2,
          "name": "cell7 ~ gene86"
        },
        {
          "x": 6,
          "y": 86,
          "value": 0,
          "name": "cell7 ~ gene87"
        },
        {
          "x": 6,
          "y": 87,
          "value": 0,
          "name": "cell7 ~ gene88"
        },
        {
          "x": 6,
          "y": 88,
          "value": 8,
          "name": "cell7 ~ gene89"
        },
        {
          "x": 6,
          "y": 89,
          "value": 10,
          "name": "cell7 ~ gene90"
        },
        {
          "x": 6,
          "y": 90,
          "value": 2,
          "name": "cell7 ~ gene91"
        },
        {
          "x": 6,
          "y": 91,
          "value": 3,
          "name": "cell7 ~ gene92"
        },
        {
          "x": 6,
          "y": 92,
          "value": 0,
          "name": "cell7 ~ gene93"
        },
        {
          "x": 6,
          "y": 93,
          "value": 0,
          "name": "cell7 ~ gene94"
        },
        {
          "x": 6,
          "y": 94,
          "value": 12,
          "name": "cell7 ~ gene95"
        },
        {
          "x": 6,
          "y": 95,
          "value": 1,
          "name": "cell7 ~ gene96"
        },
        {
          "x": 6,
          "y": 96,
          "value": 0,
          "name": "cell7 ~ gene97"
        },
        {
          "x": 6,
          "y": 97,
          "value": 16,
          "name": "cell7 ~ gene98"
        },
        {
          "x": 6,
          "y": 98,
          "value": 0,
          "name": "cell7 ~ gene99"
        },
        {
          "x": 6,
          "y": 99,
          "value": 11,
          "name": "cell7 ~ gene100"
        },
        {
          "x": 7,
          "y": 0,
          "value": 12,
          "name": "cell8 ~ gene1"
        },
        {
          "x": 7,
          "y": 1,
          "value": 3,
          "name": "cell8 ~ gene2"
        },
        {
          "x": 7,
          "y": 2,
          "value": 15,
          "name": "cell8 ~ gene3"
        },
        {
          "x": 7,
          "y": 3,
          "value": 30,
          "name": "cell8 ~ gene4"
        },
        {
          "x": 7,
          "y": 4,
          "value": 3,
          "name": "cell8 ~ gene5"
        },
        {
          "x": 7,
          "y": 5,
          "value": 2,
          "name": "cell8 ~ gene6"
        },
        {
          "x": 7,
          "y": 6,
          "value": 24,
          "name": "cell8 ~ gene7"
        },
        {
          "x": 7,
          "y": 7,
          "value": 12,
          "name": "cell8 ~ gene8"
        },
        {
          "x": 7,
          "y": 8,
          "value": 5,
          "name": "cell8 ~ gene9"
        },
        {
          "x": 7,
          "y": 9,
          "value": 0,
          "name": "cell8 ~ gene10"
        },
        {
          "x": 7,
          "y": 10,
          "value": 12,
          "name": "cell8 ~ gene11"
        },
        {
          "x": 7,
          "y": 11,
          "value": 8,
          "name": "cell8 ~ gene12"
        },
        {
          "x": 7,
          "y": 12,
          "value": 11,
          "name": "cell8 ~ gene13"
        },
        {
          "x": 7,
          "y": 13,
          "value": 0,
          "name": "cell8 ~ gene14"
        },
        {
          "x": 7,
          "y": 14,
          "value": 25,
          "name": "cell8 ~ gene15"
        },
        {
          "x": 7,
          "y": 15,
          "value": 41,
          "name": "cell8 ~ gene16"
        },
        {
          "x": 7,
          "y": 16,
          "value": 23,
          "name": "cell8 ~ gene17"
        },
        {
          "x": 7,
          "y": 17,
          "value": 18,
          "name": "cell8 ~ gene18"
        },
        {
          "x": 7,
          "y": 18,
          "value": 22,
          "name": "cell8 ~ gene19"
        },
        {
          "x": 7,
          "y": 19,
          "value": 19,
          "name": "cell8 ~ gene20"
        },
        {
          "x": 7,
          "y": 20,
          "value": 0,
          "name": "cell8 ~ gene21"
        },
        {
          "x": 7,
          "y": 21,
          "value": 0,
          "name": "cell8 ~ gene22"
        },
        {
          "x": 7,
          "y": 22,
          "value": 5,
          "name": "cell8 ~ gene23"
        },
        {
          "x": 7,
          "y": 23,
          "value": 0,
          "name": "cell8 ~ gene24"
        },
        {
          "x": 7,
          "y": 24,
          "value": 0,
          "name": "cell8 ~ gene25"
        },
        {
          "x": 7,
          "y": 25,
          "value": 6,
          "name": "cell8 ~ gene26"
        },
        {
          "x": 7,
          "y": 26,
          "value": 0,
          "name": "cell8 ~ gene27"
        },
        {
          "x": 7,
          "y": 27,
          "value": 5,
          "name": "cell8 ~ gene28"
        },
        {
          "x": 7,
          "y": 28,
          "value": 0,
          "name": "cell8 ~ gene29"
        },
        {
          "x": 7,
          "y": 29,
          "value": 0,
          "name": "cell8 ~ gene30"
        },
        {
          "x": 7,
          "y": 30,
          "value": 6,
          "name": "cell8 ~ gene31"
        },
        {
          "x": 7,
          "y": 31,
          "value": 0,
          "name": "cell8 ~ gene32"
        },
        {
          "x": 7,
          "y": 32,
          "value": 0,
          "name": "cell8 ~ gene33"
        },
        {
          "x": 7,
          "y": 33,
          "value": 0,
          "name": "cell8 ~ gene34"
        },
        {
          "x": 7,
          "y": 34,
          "value": 10,
          "name": "cell8 ~ gene35"
        },
        {
          "x": 7,
          "y": 35,
          "value": 0,
          "name": "cell8 ~ gene36"
        },
        {
          "x": 7,
          "y": 36,
          "value": 10,
          "name": "cell8 ~ gene37"
        },
        {
          "x": 7,
          "y": 37,
          "value": 0,
          "name": "cell8 ~ gene38"
        },
        {
          "x": 7,
          "y": 38,
          "value": 0,
          "name": "cell8 ~ gene39"
        },
        {
          "x": 7,
          "y": 39,
          "value": 0,
          "name": "cell8 ~ gene40"
        },
        {
          "x": 7,
          "y": 40,
          "value": 6,
          "name": "cell8 ~ gene41"
        },
        {
          "x": 7,
          "y": 41,
          "value": 0,
          "name": "cell8 ~ gene42"
        },
        {
          "x": 7,
          "y": 42,
          "value": 0,
          "name": "cell8 ~ gene43"
        },
        {
          "x": 7,
          "y": 43,
          "value": 0,
          "name": "cell8 ~ gene44"
        },
        {
          "x": 7,
          "y": 44,
          "value": 6,
          "name": "cell8 ~ gene45"
        },
        {
          "x": 7,
          "y": 45,
          "value": 11,
          "name": "cell8 ~ gene46"
        },
        {
          "x": 7,
          "y": 46,
          "value": 0,
          "name": "cell8 ~ gene47"
        },
        {
          "x": 7,
          "y": 47,
          "value": 3,
          "name": "cell8 ~ gene48"
        },
        {
          "x": 7,
          "y": 48,
          "value": 4,
          "name": "cell8 ~ gene49"
        },
        {
          "x": 7,
          "y": 49,
          "value": 12,
          "name": "cell8 ~ gene50"
        },
        {
          "x": 7,
          "y": 50,
          "value": 0,
          "name": "cell8 ~ gene51"
        },
        {
          "x": 7,
          "y": 51,
          "value": 11,
          "name": "cell8 ~ gene52"
        },
        {
          "x": 7,
          "y": 52,
          "value": 1,
          "name": "cell8 ~ gene53"
        },
        {
          "x": 7,
          "y": 53,
          "value": 0,
          "name": "cell8 ~ gene54"
        },
        {
          "x": 7,
          "y": 54,
          "value": 5,
          "name": "cell8 ~ gene55"
        },
        {
          "x": 7,
          "y": 55,
          "value": 3,
          "name": "cell8 ~ gene56"
        },
        {
          "x": 7,
          "y": 56,
          "value": 4,
          "name": "cell8 ~ gene57"
        },
        {
          "x": 7,
          "y": 57,
          "value": 10,
          "name": "cell8 ~ gene58"
        },
        {
          "x": 7,
          "y": 58,
          "value": 0,
          "name": "cell8 ~ gene59"
        },
        {
          "x": 7,
          "y": 59,
          "value": 0,
          "name": "cell8 ~ gene60"
        },
        {
          "x": 7,
          "y": 60,
          "value": 0,
          "name": "cell8 ~ gene61"
        },
        {
          "x": 7,
          "y": 61,
          "value": 11,
          "name": "cell8 ~ gene62"
        },
        {
          "x": 7,
          "y": 62,
          "value": 20,
          "name": "cell8 ~ gene63"
        },
        {
          "x": 7,
          "y": 63,
          "value": 10,
          "name": "cell8 ~ gene64"
        },
        {
          "x": 7,
          "y": 64,
          "value": 10,
          "name": "cell8 ~ gene65"
        },
        {
          "x": 7,
          "y": 65,
          "value": 18,
          "name": "cell8 ~ gene66"
        },
        {
          "x": 7,
          "y": 66,
          "value": 0,
          "name": "cell8 ~ gene67"
        },
        {
          "x": 7,
          "y": 67,
          "value": 0,
          "name": "cell8 ~ gene68"
        },
        {
          "x": 7,
          "y": 68,
          "value": 0,
          "name": "cell8 ~ gene69"
        },
        {
          "x": 7,
          "y": 69,
          "value": 3,
          "name": "cell8 ~ gene70"
        },
        {
          "x": 7,
          "y": 70,
          "value": 0,
          "name": "cell8 ~ gene71"
        },
        {
          "x": 7,
          "y": 71,
          "value": 0,
          "name": "cell8 ~ gene72"
        },
        {
          "x": 7,
          "y": 72,
          "value": 12,
          "name": "cell8 ~ gene73"
        },
        {
          "x": 7,
          "y": 73,
          "value": 0,
          "name": "cell8 ~ gene74"
        },
        {
          "x": 7,
          "y": 74,
          "value": 0,
          "name": "cell8 ~ gene75"
        },
        {
          "x": 7,
          "y": 75,
          "value": 4,
          "name": "cell8 ~ gene76"
        },
        {
          "x": 7,
          "y": 76,
          "value": 0,
          "name": "cell8 ~ gene77"
        },
        {
          "x": 7,
          "y": 77,
          "value": 0,
          "name": "cell8 ~ gene78"
        },
        {
          "x": 7,
          "y": 78,
          "value": 10,
          "name": "cell8 ~ gene79"
        },
        {
          "x": 7,
          "y": 79,
          "value": 20,
          "name": "cell8 ~ gene80"
        },
        {
          "x": 7,
          "y": 80,
          "value": 7,
          "name": "cell8 ~ gene81"
        },
        {
          "x": 7,
          "y": 81,
          "value": 7,
          "name": "cell8 ~ gene82"
        },
        {
          "x": 7,
          "y": 82,
          "value": 0,
          "name": "cell8 ~ gene83"
        },
        {
          "x": 7,
          "y": 83,
          "value": 11,
          "name": "cell8 ~ gene84"
        },
        {
          "x": 7,
          "y": 84,
          "value": 11,
          "name": "cell8 ~ gene85"
        },
        {
          "x": 7,
          "y": 85,
          "value": 0,
          "name": "cell8 ~ gene86"
        },
        {
          "x": 7,
          "y": 86,
          "value": 2,
          "name": "cell8 ~ gene87"
        },
        {
          "x": 7,
          "y": 87,
          "value": 9,
          "name": "cell8 ~ gene88"
        },
        {
          "x": 7,
          "y": 88,
          "value": 22,
          "name": "cell8 ~ gene89"
        },
        {
          "x": 7,
          "y": 89,
          "value": 0,
          "name": "cell8 ~ gene90"
        },
        {
          "x": 7,
          "y": 90,
          "value": 10,
          "name": "cell8 ~ gene91"
        },
        {
          "x": 7,
          "y": 91,
          "value": 14,
          "name": "cell8 ~ gene92"
        },
        {
          "x": 7,
          "y": 92,
          "value": 9,
          "name": "cell8 ~ gene93"
        },
        {
          "x": 7,
          "y": 93,
          "value": 3,
          "name": "cell8 ~ gene94"
        },
        {
          "x": 7,
          "y": 94,
          "value": 0,
          "name": "cell8 ~ gene95"
        },
        {
          "x": 7,
          "y": 95,
          "value": 2,
          "name": "cell8 ~ gene96"
        },
        {
          "x": 7,
          "y": 96,
          "value": 9,
          "name": "cell8 ~ gene97"
        },
        {
          "x": 7,
          "y": 97,
          "value": 0,
          "name": "cell8 ~ gene98"
        },
        {
          "x": 7,
          "y": 98,
          "value": 0,
          "name": "cell8 ~ gene99"
        },
        {
          "x": 7,
          "y": 99,
          "value": 0,
          "name": "cell8 ~ gene100"
        },
        {
          "x": 8,
          "y": 0,
          "value": 0,
          "name": "cell9 ~ gene1"
        },
        {
          "x": 8,
          "y": 1,
          "value": 0,
          "name": "cell9 ~ gene2"
        },
        {
          "x": 8,
          "y": 2,
          "value": 32,
          "name": "cell9 ~ gene3"
        },
        {
          "x": 8,
          "y": 3,
          "value": 12,
          "name": "cell9 ~ gene4"
        },
        {
          "x": 8,
          "y": 4,
          "value": 14,
          "name": "cell9 ~ gene5"
        },
        {
          "x": 8,
          "y": 5,
          "value": 0,
          "name": "cell9 ~ gene6"
        },
        {
          "x": 8,
          "y": 6,
          "value": 31,
          "name": "cell9 ~ gene7"
        },
        {
          "x": 8,
          "y": 7,
          "value": 16,
          "name": "cell9 ~ gene8"
        },
        {
          "x": 8,
          "y": 8,
          "value": 16,
          "name": "cell9 ~ gene9"
        },
        {
          "x": 8,
          "y": 9,
          "value": 0,
          "name": "cell9 ~ gene10"
        },
        {
          "x": 8,
          "y": 10,
          "value": 27,
          "name": "cell9 ~ gene11"
        },
        {
          "x": 8,
          "y": 11,
          "value": 5,
          "name": "cell9 ~ gene12"
        },
        {
          "x": 8,
          "y": 12,
          "value": 5,
          "name": "cell9 ~ gene13"
        },
        {
          "x": 8,
          "y": 13,
          "value": 23,
          "name": "cell9 ~ gene14"
        },
        {
          "x": 8,
          "y": 14,
          "value": 40,
          "name": "cell9 ~ gene15"
        },
        {
          "x": 8,
          "y": 15,
          "value": 40,
          "name": "cell9 ~ gene16"
        },
        {
          "x": 8,
          "y": 16,
          "value": 9,
          "name": "cell9 ~ gene17"
        },
        {
          "x": 8,
          "y": 17,
          "value": 7,
          "name": "cell9 ~ gene18"
        },
        {
          "x": 8,
          "y": 18,
          "value": 23,
          "name": "cell9 ~ gene19"
        },
        {
          "x": 8,
          "y": 19,
          "value": 0,
          "name": "cell9 ~ gene20"
        },
        {
          "x": 8,
          "y": 20,
          "value": 0,
          "name": "cell9 ~ gene21"
        },
        {
          "x": 8,
          "y": 21,
          "value": 7,
          "name": "cell9 ~ gene22"
        },
        {
          "x": 8,
          "y": 22,
          "value": 0,
          "name": "cell9 ~ gene23"
        },
        {
          "x": 8,
          "y": 23,
          "value": 0,
          "name": "cell9 ~ gene24"
        },
        {
          "x": 8,
          "y": 24,
          "value": 0,
          "name": "cell9 ~ gene25"
        },
        {
          "x": 8,
          "y": 25,
          "value": 13,
          "name": "cell9 ~ gene26"
        },
        {
          "x": 8,
          "y": 26,
          "value": 0,
          "name": "cell9 ~ gene27"
        },
        {
          "x": 8,
          "y": 27,
          "value": 0,
          "name": "cell9 ~ gene28"
        },
        {
          "x": 8,
          "y": 28,
          "value": 6,
          "name": "cell9 ~ gene29"
        },
        {
          "x": 8,
          "y": 29,
          "value": 3,
          "name": "cell9 ~ gene30"
        },
        {
          "x": 8,
          "y": 30,
          "value": 0,
          "name": "cell9 ~ gene31"
        },
        {
          "x": 8,
          "y": 31,
          "value": 0,
          "name": "cell9 ~ gene32"
        },
        {
          "x": 8,
          "y": 32,
          "value": 0,
          "name": "cell9 ~ gene33"
        },
        {
          "x": 8,
          "y": 33,
          "value": 17,
          "name": "cell9 ~ gene34"
        },
        {
          "x": 8,
          "y": 34,
          "value": 1,
          "name": "cell9 ~ gene35"
        },
        {
          "x": 8,
          "y": 35,
          "value": 0,
          "name": "cell9 ~ gene36"
        },
        {
          "x": 8,
          "y": 36,
          "value": 12,
          "name": "cell9 ~ gene37"
        },
        {
          "x": 8,
          "y": 37,
          "value": 0,
          "name": "cell9 ~ gene38"
        },
        {
          "x": 8,
          "y": 38,
          "value": 5,
          "name": "cell9 ~ gene39"
        },
        {
          "x": 8,
          "y": 39,
          "value": 12,
          "name": "cell9 ~ gene40"
        },
        {
          "x": 8,
          "y": 40,
          "value": 0,
          "name": "cell9 ~ gene41"
        },
        {
          "x": 8,
          "y": 41,
          "value": 3,
          "name": "cell9 ~ gene42"
        },
        {
          "x": 8,
          "y": 42,
          "value": 10,
          "name": "cell9 ~ gene43"
        },
        {
          "x": 8,
          "y": 43,
          "value": 11,
          "name": "cell9 ~ gene44"
        },
        {
          "x": 8,
          "y": 44,
          "value": 15,
          "name": "cell9 ~ gene45"
        },
        {
          "x": 8,
          "y": 45,
          "value": 19,
          "name": "cell9 ~ gene46"
        },
        {
          "x": 8,
          "y": 46,
          "value": 2,
          "name": "cell9 ~ gene47"
        },
        {
          "x": 8,
          "y": 47,
          "value": 0,
          "name": "cell9 ~ gene48"
        },
        {
          "x": 8,
          "y": 48,
          "value": 0,
          "name": "cell9 ~ gene49"
        },
        {
          "x": 8,
          "y": 49,
          "value": 7,
          "name": "cell9 ~ gene50"
        },
        {
          "x": 8,
          "y": 50,
          "value": 0,
          "name": "cell9 ~ gene51"
        },
        {
          "x": 8,
          "y": 51,
          "value": 5,
          "name": "cell9 ~ gene52"
        },
        {
          "x": 8,
          "y": 52,
          "value": 0,
          "name": "cell9 ~ gene53"
        },
        {
          "x": 8,
          "y": 53,
          "value": 0,
          "name": "cell9 ~ gene54"
        },
        {
          "x": 8,
          "y": 54,
          "value": 3,
          "name": "cell9 ~ gene55"
        },
        {
          "x": 8,
          "y": 55,
          "value": 0,
          "name": "cell9 ~ gene56"
        },
        {
          "x": 8,
          "y": 56,
          "value": 0,
          "name": "cell9 ~ gene57"
        },
        {
          "x": 8,
          "y": 57,
          "value": 0,
          "name": "cell9 ~ gene58"
        },
        {
          "x": 8,
          "y": 58,
          "value": 14,
          "name": "cell9 ~ gene59"
        },
        {
          "x": 8,
          "y": 59,
          "value": 0,
          "name": "cell9 ~ gene60"
        },
        {
          "x": 8,
          "y": 60,
          "value": 1,
          "name": "cell9 ~ gene61"
        },
        {
          "x": 8,
          "y": 61,
          "value": 0,
          "name": "cell9 ~ gene62"
        },
        {
          "x": 8,
          "y": 62,
          "value": 30,
          "name": "cell9 ~ gene63"
        },
        {
          "x": 8,
          "y": 63,
          "value": 0,
          "name": "cell9 ~ gene64"
        },
        {
          "x": 8,
          "y": 64,
          "value": 18,
          "name": "cell9 ~ gene65"
        },
        {
          "x": 8,
          "y": 65,
          "value": 9,
          "name": "cell9 ~ gene66"
        },
        {
          "x": 8,
          "y": 66,
          "value": 0,
          "name": "cell9 ~ gene67"
        },
        {
          "x": 8,
          "y": 67,
          "value": 5,
          "name": "cell9 ~ gene68"
        },
        {
          "x": 8,
          "y": 68,
          "value": 7,
          "name": "cell9 ~ gene69"
        },
        {
          "x": 8,
          "y": 69,
          "value": 0,
          "name": "cell9 ~ gene70"
        },
        {
          "x": 8,
          "y": 70,
          "value": 0,
          "name": "cell9 ~ gene71"
        },
        {
          "x": 8,
          "y": 71,
          "value": 22,
          "name": "cell9 ~ gene72"
        },
        {
          "x": 8,
          "y": 72,
          "value": 0,
          "name": "cell9 ~ gene73"
        },
        {
          "x": 8,
          "y": 73,
          "value": 0,
          "name": "cell9 ~ gene74"
        },
        {
          "x": 8,
          "y": 74,
          "value": 12,
          "name": "cell9 ~ gene75"
        },
        {
          "x": 8,
          "y": 75,
          "value": 0,
          "name": "cell9 ~ gene76"
        },
        {
          "x": 8,
          "y": 76,
          "value": 2,
          "name": "cell9 ~ gene77"
        },
        {
          "x": 8,
          "y": 77,
          "value": 0,
          "name": "cell9 ~ gene78"
        },
        {
          "x": 8,
          "y": 78,
          "value": 0,
          "name": "cell9 ~ gene79"
        },
        {
          "x": 8,
          "y": 79,
          "value": 0,
          "name": "cell9 ~ gene80"
        },
        {
          "x": 8,
          "y": 80,
          "value": 8,
          "name": "cell9 ~ gene81"
        },
        {
          "x": 8,
          "y": 81,
          "value": 0,
          "name": "cell9 ~ gene82"
        },
        {
          "x": 8,
          "y": 82,
          "value": 0,
          "name": "cell9 ~ gene83"
        },
        {
          "x": 8,
          "y": 83,
          "value": 2,
          "name": "cell9 ~ gene84"
        },
        {
          "x": 8,
          "y": 84,
          "value": 4,
          "name": "cell9 ~ gene85"
        },
        {
          "x": 8,
          "y": 85,
          "value": 0,
          "name": "cell9 ~ gene86"
        },
        {
          "x": 8,
          "y": 86,
          "value": 0,
          "name": "cell9 ~ gene87"
        },
        {
          "x": 8,
          "y": 87,
          "value": 0,
          "name": "cell9 ~ gene88"
        },
        {
          "x": 8,
          "y": 88,
          "value": 0,
          "name": "cell9 ~ gene89"
        },
        {
          "x": 8,
          "y": 89,
          "value": 0,
          "name": "cell9 ~ gene90"
        },
        {
          "x": 8,
          "y": 90,
          "value": 0,
          "name": "cell9 ~ gene91"
        },
        {
          "x": 8,
          "y": 91,
          "value": 0,
          "name": "cell9 ~ gene92"
        },
        {
          "x": 8,
          "y": 92,
          "value": 0,
          "name": "cell9 ~ gene93"
        },
        {
          "x": 8,
          "y": 93,
          "value": 0,
          "name": "cell9 ~ gene94"
        },
        {
          "x": 8,
          "y": 94,
          "value": 0,
          "name": "cell9 ~ gene95"
        },
        {
          "x": 8,
          "y": 95,
          "value": 15,
          "name": "cell9 ~ gene96"
        },
        {
          "x": 8,
          "y": 96,
          "value": 4,
          "name": "cell9 ~ gene97"
        },
        {
          "x": 8,
          "y": 97,
          "value": 3,
          "name": "cell9 ~ gene98"
        },
        {
          "x": 8,
          "y": 98,
          "value": 0,
          "name": "cell9 ~ gene99"
        },
        {
          "x": 8,
          "y": 99,
          "value": 0,
          "name": "cell9 ~ gene100"
        },
        {
          "x": 9,
          "y": 0,
          "value": 0,
          "name": "cell10 ~ gene1"
        },
        {
          "x": 9,
          "y": 1,
          "value": 0,
          "name": "cell10 ~ gene2"
        },
        {
          "x": 9,
          "y": 2,
          "value": 19,
          "name": "cell10 ~ gene3"
        },
        {
          "x": 9,
          "y": 3,
          "value": 21,
          "name": "cell10 ~ gene4"
        },
        {
          "x": 9,
          "y": 4,
          "value": 37,
          "name": "cell10 ~ gene5"
        },
        {
          "x": 9,
          "y": 5,
          "value": 8,
          "name": "cell10 ~ gene6"
        },
        {
          "x": 9,
          "y": 6,
          "value": 19,
          "name": "cell10 ~ gene7"
        },
        {
          "x": 9,
          "y": 7,
          "value": 0,
          "name": "cell10 ~ gene8"
        },
        {
          "x": 9,
          "y": 8,
          "value": 0,
          "name": "cell10 ~ gene9"
        },
        {
          "x": 9,
          "y": 9,
          "value": 25,
          "name": "cell10 ~ gene10"
        },
        {
          "x": 9,
          "y": 10,
          "value": 18,
          "name": "cell10 ~ gene11"
        },
        {
          "x": 9,
          "y": 11,
          "value": 13,
          "name": "cell10 ~ gene12"
        },
        {
          "x": 9,
          "y": 12,
          "value": 20,
          "name": "cell10 ~ gene13"
        },
        {
          "x": 9,
          "y": 13,
          "value": 13,
          "name": "cell10 ~ gene14"
        },
        {
          "x": 9,
          "y": 14,
          "value": 23,
          "name": "cell10 ~ gene15"
        },
        {
          "x": 9,
          "y": 15,
          "value": 38,
          "name": "cell10 ~ gene16"
        },
        {
          "x": 9,
          "y": 16,
          "value": 27,
          "name": "cell10 ~ gene17"
        },
        {
          "x": 9,
          "y": 17,
          "value": 30,
          "name": "cell10 ~ gene18"
        },
        {
          "x": 9,
          "y": 18,
          "value": 20,
          "name": "cell10 ~ gene19"
        },
        {
          "x": 9,
          "y": 19,
          "value": 5,
          "name": "cell10 ~ gene20"
        },
        {
          "x": 9,
          "y": 20,
          "value": 0,
          "name": "cell10 ~ gene21"
        },
        {
          "x": 9,
          "y": 21,
          "value": 11,
          "name": "cell10 ~ gene22"
        },
        {
          "x": 9,
          "y": 22,
          "value": 18,
          "name": "cell10 ~ gene23"
        },
        {
          "x": 9,
          "y": 23,
          "value": 0,
          "name": "cell10 ~ gene24"
        },
        {
          "x": 9,
          "y": 24,
          "value": 0,
          "name": "cell10 ~ gene25"
        },
        {
          "x": 9,
          "y": 25,
          "value": 0,
          "name": "cell10 ~ gene26"
        },
        {
          "x": 9,
          "y": 26,
          "value": 0,
          "name": "cell10 ~ gene27"
        },
        {
          "x": 9,
          "y": 27,
          "value": 11,
          "name": "cell10 ~ gene28"
        },
        {
          "x": 9,
          "y": 28,
          "value": 12,
          "name": "cell10 ~ gene29"
        },
        {
          "x": 9,
          "y": 29,
          "value": 20,
          "name": "cell10 ~ gene30"
        },
        {
          "x": 9,
          "y": 30,
          "value": 5,
          "name": "cell10 ~ gene31"
        },
        {
          "x": 9,
          "y": 31,
          "value": 0,
          "name": "cell10 ~ gene32"
        },
        {
          "x": 9,
          "y": 32,
          "value": 5,
          "name": "cell10 ~ gene33"
        },
        {
          "x": 9,
          "y": 33,
          "value": 0,
          "name": "cell10 ~ gene34"
        },
        {
          "x": 9,
          "y": 34,
          "value": 0,
          "name": "cell10 ~ gene35"
        },
        {
          "x": 9,
          "y": 35,
          "value": 2,
          "name": "cell10 ~ gene36"
        },
        {
          "x": 9,
          "y": 36,
          "value": 1,
          "name": "cell10 ~ gene37"
        },
        {
          "x": 9,
          "y": 37,
          "value": 8,
          "name": "cell10 ~ gene38"
        },
        {
          "x": 9,
          "y": 38,
          "value": 0,
          "name": "cell10 ~ gene39"
        },
        {
          "x": 9,
          "y": 39,
          "value": 0,
          "name": "cell10 ~ gene40"
        },
        {
          "x": 9,
          "y": 40,
          "value": 2,
          "name": "cell10 ~ gene41"
        },
        {
          "x": 9,
          "y": 41,
          "value": 9,
          "name": "cell10 ~ gene42"
        },
        {
          "x": 9,
          "y": 42,
          "value": 0,
          "name": "cell10 ~ gene43"
        },
        {
          "x": 9,
          "y": 43,
          "value": 0,
          "name": "cell10 ~ gene44"
        },
        {
          "x": 9,
          "y": 44,
          "value": 0,
          "name": "cell10 ~ gene45"
        },
        {
          "x": 9,
          "y": 45,
          "value": 0,
          "name": "cell10 ~ gene46"
        },
        {
          "x": 9,
          "y": 46,
          "value": 0,
          "name": "cell10 ~ gene47"
        },
        {
          "x": 9,
          "y": 47,
          "value": 0,
          "name": "cell10 ~ gene48"
        },
        {
          "x": 9,
          "y": 48,
          "value": 7,
          "name": "cell10 ~ gene49"
        },
        {
          "x": 9,
          "y": 49,
          "value": 0,
          "name": "cell10 ~ gene50"
        },
        {
          "x": 9,
          "y": 50,
          "value": 0,
          "name": "cell10 ~ gene51"
        },
        {
          "x": 9,
          "y": 51,
          "value": 0,
          "name": "cell10 ~ gene52"
        },
        {
          "x": 9,
          "y": 52,
          "value": 12,
          "name": "cell10 ~ gene53"
        },
        {
          "x": 9,
          "y": 53,
          "value": 0,
          "name": "cell10 ~ gene54"
        },
        {
          "x": 9,
          "y": 54,
          "value": 11,
          "name": "cell10 ~ gene55"
        },
        {
          "x": 9,
          "y": 55,
          "value": 0,
          "name": "cell10 ~ gene56"
        },
        {
          "x": 9,
          "y": 56,
          "value": 0,
          "name": "cell10 ~ gene57"
        },
        {
          "x": 9,
          "y": 57,
          "value": 0,
          "name": "cell10 ~ gene58"
        },
        {
          "x": 9,
          "y": 58,
          "value": 0,
          "name": "cell10 ~ gene59"
        },
        {
          "x": 9,
          "y": 59,
          "value": 16,
          "name": "cell10 ~ gene60"
        },
        {
          "x": 9,
          "y": 60,
          "value": 0,
          "name": "cell10 ~ gene61"
        },
        {
          "x": 9,
          "y": 61,
          "value": 26,
          "name": "cell10 ~ gene62"
        },
        {
          "x": 9,
          "y": 62,
          "value": 3,
          "name": "cell10 ~ gene63"
        },
        {
          "x": 9,
          "y": 63,
          "value": 0,
          "name": "cell10 ~ gene64"
        },
        {
          "x": 9,
          "y": 64,
          "value": 7,
          "name": "cell10 ~ gene65"
        },
        {
          "x": 9,
          "y": 65,
          "value": 9,
          "name": "cell10 ~ gene66"
        },
        {
          "x": 9,
          "y": 66,
          "value": 0,
          "name": "cell10 ~ gene67"
        },
        {
          "x": 9,
          "y": 67,
          "value": 1,
          "name": "cell10 ~ gene68"
        },
        {
          "x": 9,
          "y": 68,
          "value": 18,
          "name": "cell10 ~ gene69"
        },
        {
          "x": 9,
          "y": 69,
          "value": 0,
          "name": "cell10 ~ gene70"
        },
        {
          "x": 9,
          "y": 70,
          "value": 0,
          "name": "cell10 ~ gene71"
        },
        {
          "x": 9,
          "y": 71,
          "value": 8,
          "name": "cell10 ~ gene72"
        },
        {
          "x": 9,
          "y": 72,
          "value": 0,
          "name": "cell10 ~ gene73"
        },
        {
          "x": 9,
          "y": 73,
          "value": 0,
          "name": "cell10 ~ gene74"
        },
        {
          "x": 9,
          "y": 74,
          "value": 14,
          "name": "cell10 ~ gene75"
        },
        {
          "x": 9,
          "y": 75,
          "value": 0,
          "name": "cell10 ~ gene76"
        },
        {
          "x": 9,
          "y": 76,
          "value": 0,
          "name": "cell10 ~ gene77"
        },
        {
          "x": 9,
          "y": 77,
          "value": 0,
          "name": "cell10 ~ gene78"
        },
        {
          "x": 9,
          "y": 78,
          "value": 0,
          "name": "cell10 ~ gene79"
        },
        {
          "x": 9,
          "y": 79,
          "value": 0,
          "name": "cell10 ~ gene80"
        },
        {
          "x": 9,
          "y": 80,
          "value": 0,
          "name": "cell10 ~ gene81"
        },
        {
          "x": 9,
          "y": 81,
          "value": 0,
          "name": "cell10 ~ gene82"
        },
        {
          "x": 9,
          "y": 82,
          "value": 8,
          "name": "cell10 ~ gene83"
        },
        {
          "x": 9,
          "y": 83,
          "value": 5,
          "name": "cell10 ~ gene84"
        },
        {
          "x": 9,
          "y": 84,
          "value": 0,
          "name": "cell10 ~ gene85"
        },
        {
          "x": 9,
          "y": 85,
          "value": 24,
          "name": "cell10 ~ gene86"
        },
        {
          "x": 9,
          "y": 86,
          "value": 15,
          "name": "cell10 ~ gene87"
        },
        {
          "x": 9,
          "y": 87,
          "value": 0,
          "name": "cell10 ~ gene88"
        },
        {
          "x": 9,
          "y": 88,
          "value": 0,
          "name": "cell10 ~ gene89"
        },
        {
          "x": 9,
          "y": 89,
          "value": 0,
          "name": "cell10 ~ gene90"
        },
        {
          "x": 9,
          "y": 90,
          "value": 10,
          "name": "cell10 ~ gene91"
        },
        {
          "x": 9,
          "y": 91,
          "value": 6,
          "name": "cell10 ~ gene92"
        },
        {
          "x": 9,
          "y": 92,
          "value": 0,
          "name": "cell10 ~ gene93"
        },
        {
          "x": 9,
          "y": 93,
          "value": 0,
          "name": "cell10 ~ gene94"
        },
        {
          "x": 9,
          "y": 94,
          "value": 1,
          "name": "cell10 ~ gene95"
        },
        {
          "x": 9,
          "y": 95,
          "value": 10,
          "name": "cell10 ~ gene96"
        },
        {
          "x": 9,
          "y": 96,
          "value": 0,
          "name": "cell10 ~ gene97"
        },
        {
          "x": 9,
          "y": 97,
          "value": 11,
          "name": "cell10 ~ gene98"
        },
        {
          "x": 9,
          "y": 98,
          "value": 8,
          "name": "cell10 ~ gene99"
        },
        {
          "x": 9,
          "y": 99,
          "value": 0,
          "name": "cell10 ~ gene100"
        },
        {
          "x": 10,
          "y": 0,
          "value": 3,
          "name": "cell11 ~ gene1"
        },
        {
          "x": 10,
          "y": 1,
          "value": 18,
          "name": "cell11 ~ gene2"
        },
        {
          "x": 10,
          "y": 2,
          "value": 17,
          "name": "cell11 ~ gene3"
        },
        {
          "x": 10,
          "y": 3,
          "value": 7,
          "name": "cell11 ~ gene4"
        },
        {
          "x": 10,
          "y": 4,
          "value": 4,
          "name": "cell11 ~ gene5"
        },
        {
          "x": 10,
          "y": 5,
          "value": 11,
          "name": "cell11 ~ gene6"
        },
        {
          "x": 10,
          "y": 6,
          "value": 29,
          "name": "cell11 ~ gene7"
        },
        {
          "x": 10,
          "y": 7,
          "value": 13,
          "name": "cell11 ~ gene8"
        },
        {
          "x": 10,
          "y": 8,
          "value": 2,
          "name": "cell11 ~ gene9"
        },
        {
          "x": 10,
          "y": 9,
          "value": 0,
          "name": "cell11 ~ gene10"
        },
        {
          "x": 10,
          "y": 10,
          "value": 10,
          "name": "cell11 ~ gene11"
        },
        {
          "x": 10,
          "y": 11,
          "value": 13,
          "name": "cell11 ~ gene12"
        },
        {
          "x": 10,
          "y": 12,
          "value": 0,
          "name": "cell11 ~ gene13"
        },
        {
          "x": 10,
          "y": 13,
          "value": 18,
          "name": "cell11 ~ gene14"
        },
        {
          "x": 10,
          "y": 14,
          "value": 10,
          "name": "cell11 ~ gene15"
        },
        {
          "x": 10,
          "y": 15,
          "value": 30,
          "name": "cell11 ~ gene16"
        },
        {
          "x": 10,
          "y": 16,
          "value": 0,
          "name": "cell11 ~ gene17"
        },
        {
          "x": 10,
          "y": 17,
          "value": 29,
          "name": "cell11 ~ gene18"
        },
        {
          "x": 10,
          "y": 18,
          "value": 17,
          "name": "cell11 ~ gene19"
        },
        {
          "x": 10,
          "y": 19,
          "value": 8,
          "name": "cell11 ~ gene20"
        },
        {
          "x": 10,
          "y": 20,
          "value": 0,
          "name": "cell11 ~ gene21"
        },
        {
          "x": 10,
          "y": 21,
          "value": 0,
          "name": "cell11 ~ gene22"
        },
        {
          "x": 10,
          "y": 22,
          "value": 0,
          "name": "cell11 ~ gene23"
        },
        {
          "x": 10,
          "y": 23,
          "value": 0,
          "name": "cell11 ~ gene24"
        },
        {
          "x": 10,
          "y": 24,
          "value": 0,
          "name": "cell11 ~ gene25"
        },
        {
          "x": 10,
          "y": 25,
          "value": 14,
          "name": "cell11 ~ gene26"
        },
        {
          "x": 10,
          "y": 26,
          "value": 6,
          "name": "cell11 ~ gene27"
        },
        {
          "x": 10,
          "y": 27,
          "value": 0,
          "name": "cell11 ~ gene28"
        },
        {
          "x": 10,
          "y": 28,
          "value": 2,
          "name": "cell11 ~ gene29"
        },
        {
          "x": 10,
          "y": 29,
          "value": 0,
          "name": "cell11 ~ gene30"
        },
        {
          "x": 10,
          "y": 30,
          "value": 0,
          "name": "cell11 ~ gene31"
        },
        {
          "x": 10,
          "y": 31,
          "value": 2,
          "name": "cell11 ~ gene32"
        },
        {
          "x": 10,
          "y": 32,
          "value": 27,
          "name": "cell11 ~ gene33"
        },
        {
          "x": 10,
          "y": 33,
          "value": 2,
          "name": "cell11 ~ gene34"
        },
        {
          "x": 10,
          "y": 34,
          "value": 1,
          "name": "cell11 ~ gene35"
        },
        {
          "x": 10,
          "y": 35,
          "value": 0,
          "name": "cell11 ~ gene36"
        },
        {
          "x": 10,
          "y": 36,
          "value": 12,
          "name": "cell11 ~ gene37"
        },
        {
          "x": 10,
          "y": 37,
          "value": 12,
          "name": "cell11 ~ gene38"
        },
        {
          "x": 10,
          "y": 38,
          "value": 0,
          "name": "cell11 ~ gene39"
        },
        {
          "x": 10,
          "y": 39,
          "value": 0,
          "name": "cell11 ~ gene40"
        },
        {
          "x": 10,
          "y": 40,
          "value": 0,
          "name": "cell11 ~ gene41"
        },
        {
          "x": 10,
          "y": 41,
          "value": 1,
          "name": "cell11 ~ gene42"
        },
        {
          "x": 10,
          "y": 42,
          "value": 0,
          "name": "cell11 ~ gene43"
        },
        {
          "x": 10,
          "y": 43,
          "value": 10,
          "name": "cell11 ~ gene44"
        },
        {
          "x": 10,
          "y": 44,
          "value": 2,
          "name": "cell11 ~ gene45"
        },
        {
          "x": 10,
          "y": 45,
          "value": 1,
          "name": "cell11 ~ gene46"
        },
        {
          "x": 10,
          "y": 46,
          "value": 4,
          "name": "cell11 ~ gene47"
        },
        {
          "x": 10,
          "y": 47,
          "value": 0,
          "name": "cell11 ~ gene48"
        },
        {
          "x": 10,
          "y": 48,
          "value": 0,
          "name": "cell11 ~ gene49"
        },
        {
          "x": 10,
          "y": 49,
          "value": 14,
          "name": "cell11 ~ gene50"
        },
        {
          "x": 10,
          "y": 50,
          "value": 2,
          "name": "cell11 ~ gene51"
        },
        {
          "x": 10,
          "y": 51,
          "value": 4,
          "name": "cell11 ~ gene52"
        },
        {
          "x": 10,
          "y": 52,
          "value": 0,
          "name": "cell11 ~ gene53"
        },
        {
          "x": 10,
          "y": 53,
          "value": 22,
          "name": "cell11 ~ gene54"
        },
        {
          "x": 10,
          "y": 54,
          "value": 0,
          "name": "cell11 ~ gene55"
        },
        {
          "x": 10,
          "y": 55,
          "value": 5,
          "name": "cell11 ~ gene56"
        },
        {
          "x": 10,
          "y": 56,
          "value": 0,
          "name": "cell11 ~ gene57"
        },
        {
          "x": 10,
          "y": 57,
          "value": 9,
          "name": "cell11 ~ gene58"
        },
        {
          "x": 10,
          "y": 58,
          "value": 0,
          "name": "cell11 ~ gene59"
        },
        {
          "x": 10,
          "y": 59,
          "value": 10,
          "name": "cell11 ~ gene60"
        },
        {
          "x": 10,
          "y": 60,
          "value": 10,
          "name": "cell11 ~ gene61"
        },
        {
          "x": 10,
          "y": 61,
          "value": 0,
          "name": "cell11 ~ gene62"
        },
        {
          "x": 10,
          "y": 62,
          "value": 0,
          "name": "cell11 ~ gene63"
        },
        {
          "x": 10,
          "y": 63,
          "value": 4,
          "name": "cell11 ~ gene64"
        },
        {
          "x": 10,
          "y": 64,
          "value": 0,
          "name": "cell11 ~ gene65"
        },
        {
          "x": 10,
          "y": 65,
          "value": 7,
          "name": "cell11 ~ gene66"
        },
        {
          "x": 10,
          "y": 66,
          "value": 8,
          "name": "cell11 ~ gene67"
        },
        {
          "x": 10,
          "y": 67,
          "value": 8,
          "name": "cell11 ~ gene68"
        },
        {
          "x": 10,
          "y": 68,
          "value": 0,
          "name": "cell11 ~ gene69"
        },
        {
          "x": 10,
          "y": 69,
          "value": 18,
          "name": "cell11 ~ gene70"
        },
        {
          "x": 10,
          "y": 70,
          "value": 0,
          "name": "cell11 ~ gene71"
        },
        {
          "x": 10,
          "y": 71,
          "value": 0,
          "name": "cell11 ~ gene72"
        },
        {
          "x": 10,
          "y": 72,
          "value": 0,
          "name": "cell11 ~ gene73"
        },
        {
          "x": 10,
          "y": 73,
          "value": 0,
          "name": "cell11 ~ gene74"
        },
        {
          "x": 10,
          "y": 74,
          "value": 0,
          "name": "cell11 ~ gene75"
        },
        {
          "x": 10,
          "y": 75,
          "value": 2,
          "name": "cell11 ~ gene76"
        },
        {
          "x": 10,
          "y": 76,
          "value": 0,
          "name": "cell11 ~ gene77"
        },
        {
          "x": 10,
          "y": 77,
          "value": 0,
          "name": "cell11 ~ gene78"
        },
        {
          "x": 10,
          "y": 78,
          "value": 0,
          "name": "cell11 ~ gene79"
        },
        {
          "x": 10,
          "y": 79,
          "value": 0,
          "name": "cell11 ~ gene80"
        },
        {
          "x": 10,
          "y": 80,
          "value": 0,
          "name": "cell11 ~ gene81"
        },
        {
          "x": 10,
          "y": 81,
          "value": 18,
          "name": "cell11 ~ gene82"
        },
        {
          "x": 10,
          "y": 82,
          "value": 0,
          "name": "cell11 ~ gene83"
        },
        {
          "x": 10,
          "y": 83,
          "value": 7,
          "name": "cell11 ~ gene84"
        },
        {
          "x": 10,
          "y": 84,
          "value": 0,
          "name": "cell11 ~ gene85"
        },
        {
          "x": 10,
          "y": 85,
          "value": 0,
          "name": "cell11 ~ gene86"
        },
        {
          "x": 10,
          "y": 86,
          "value": 0,
          "name": "cell11 ~ gene87"
        },
        {
          "x": 10,
          "y": 87,
          "value": 2,
          "name": "cell11 ~ gene88"
        },
        {
          "x": 10,
          "y": 88,
          "value": 0,
          "name": "cell11 ~ gene89"
        },
        {
          "x": 10,
          "y": 89,
          "value": 2,
          "name": "cell11 ~ gene90"
        },
        {
          "x": 10,
          "y": 90,
          "value": 3,
          "name": "cell11 ~ gene91"
        },
        {
          "x": 10,
          "y": 91,
          "value": 7,
          "name": "cell11 ~ gene92"
        },
        {
          "x": 10,
          "y": 92,
          "value": 0,
          "name": "cell11 ~ gene93"
        },
        {
          "x": 10,
          "y": 93,
          "value": 14,
          "name": "cell11 ~ gene94"
        },
        {
          "x": 10,
          "y": 94,
          "value": 0,
          "name": "cell11 ~ gene95"
        },
        {
          "x": 10,
          "y": 95,
          "value": 0,
          "name": "cell11 ~ gene96"
        },
        {
          "x": 10,
          "y": 96,
          "value": 11,
          "name": "cell11 ~ gene97"
        },
        {
          "x": 10,
          "y": 97,
          "value": 12,
          "name": "cell11 ~ gene98"
        },
        {
          "x": 10,
          "y": 98,
          "value": 0,
          "name": "cell11 ~ gene99"
        },
        {
          "x": 10,
          "y": 99,
          "value": 5,
          "name": "cell11 ~ gene100"
        },
        {
          "x": 11,
          "y": 0,
          "value": 0,
          "name": "cell12 ~ gene1"
        },
        {
          "x": 11,
          "y": 1,
          "value": 15,
          "name": "cell12 ~ gene2"
        },
        {
          "x": 11,
          "y": 2,
          "value": 19,
          "name": "cell12 ~ gene3"
        },
        {
          "x": 11,
          "y": 3,
          "value": 20,
          "name": "cell12 ~ gene4"
        },
        {
          "x": 11,
          "y": 4,
          "value": 16,
          "name": "cell12 ~ gene5"
        },
        {
          "x": 11,
          "y": 5,
          "value": 0,
          "name": "cell12 ~ gene6"
        },
        {
          "x": 11,
          "y": 6,
          "value": 35,
          "name": "cell12 ~ gene7"
        },
        {
          "x": 11,
          "y": 7,
          "value": 11,
          "name": "cell12 ~ gene8"
        },
        {
          "x": 11,
          "y": 8,
          "value": 9,
          "name": "cell12 ~ gene9"
        },
        {
          "x": 11,
          "y": 9,
          "value": 0,
          "name": "cell12 ~ gene10"
        },
        {
          "x": 11,
          "y": 10,
          "value": 10,
          "name": "cell12 ~ gene11"
        },
        {
          "x": 11,
          "y": 11,
          "value": 6,
          "name": "cell12 ~ gene12"
        },
        {
          "x": 11,
          "y": 12,
          "value": 0,
          "name": "cell12 ~ gene13"
        },
        {
          "x": 11,
          "y": 13,
          "value": 2,
          "name": "cell12 ~ gene14"
        },
        {
          "x": 11,
          "y": 14,
          "value": 19,
          "name": "cell12 ~ gene15"
        },
        {
          "x": 11,
          "y": 15,
          "value": 38,
          "name": "cell12 ~ gene16"
        },
        {
          "x": 11,
          "y": 16,
          "value": 3,
          "name": "cell12 ~ gene17"
        },
        {
          "x": 11,
          "y": 17,
          "value": 10,
          "name": "cell12 ~ gene18"
        },
        {
          "x": 11,
          "y": 18,
          "value": 27,
          "name": "cell12 ~ gene19"
        },
        {
          "x": 11,
          "y": 19,
          "value": 0,
          "name": "cell12 ~ gene20"
        },
        {
          "x": 11,
          "y": 20,
          "value": 10,
          "name": "cell12 ~ gene21"
        },
        {
          "x": 11,
          "y": 21,
          "value": 0,
          "name": "cell12 ~ gene22"
        },
        {
          "x": 11,
          "y": 22,
          "value": 0,
          "name": "cell12 ~ gene23"
        },
        {
          "x": 11,
          "y": 23,
          "value": 0,
          "name": "cell12 ~ gene24"
        },
        {
          "x": 11,
          "y": 24,
          "value": 0,
          "name": "cell12 ~ gene25"
        },
        {
          "x": 11,
          "y": 25,
          "value": 7,
          "name": "cell12 ~ gene26"
        },
        {
          "x": 11,
          "y": 26,
          "value": 7,
          "name": "cell12 ~ gene27"
        },
        {
          "x": 11,
          "y": 27,
          "value": 0,
          "name": "cell12 ~ gene28"
        },
        {
          "x": 11,
          "y": 28,
          "value": 0,
          "name": "cell12 ~ gene29"
        },
        {
          "x": 11,
          "y": 29,
          "value": 0,
          "name": "cell12 ~ gene30"
        },
        {
          "x": 11,
          "y": 30,
          "value": 13,
          "name": "cell12 ~ gene31"
        },
        {
          "x": 11,
          "y": 31,
          "value": 0,
          "name": "cell12 ~ gene32"
        },
        {
          "x": 11,
          "y": 32,
          "value": 0,
          "name": "cell12 ~ gene33"
        },
        {
          "x": 11,
          "y": 33,
          "value": 0,
          "name": "cell12 ~ gene34"
        },
        {
          "x": 11,
          "y": 34,
          "value": 8,
          "name": "cell12 ~ gene35"
        },
        {
          "x": 11,
          "y": 35,
          "value": 18,
          "name": "cell12 ~ gene36"
        },
        {
          "x": 11,
          "y": 36,
          "value": 5,
          "name": "cell12 ~ gene37"
        },
        {
          "x": 11,
          "y": 37,
          "value": 0,
          "name": "cell12 ~ gene38"
        },
        {
          "x": 11,
          "y": 38,
          "value": 23,
          "name": "cell12 ~ gene39"
        },
        {
          "x": 11,
          "y": 39,
          "value": 1,
          "name": "cell12 ~ gene40"
        },
        {
          "x": 11,
          "y": 40,
          "value": 14,
          "name": "cell12 ~ gene41"
        },
        {
          "x": 11,
          "y": 41,
          "value": 13,
          "name": "cell12 ~ gene42"
        },
        {
          "x": 11,
          "y": 42,
          "value": 0,
          "name": "cell12 ~ gene43"
        },
        {
          "x": 11,
          "y": 43,
          "value": 16,
          "name": "cell12 ~ gene44"
        },
        {
          "x": 11,
          "y": 44,
          "value": 0,
          "name": "cell12 ~ gene45"
        },
        {
          "x": 11,
          "y": 45,
          "value": 0,
          "name": "cell12 ~ gene46"
        },
        {
          "x": 11,
          "y": 46,
          "value": 1,
          "name": "cell12 ~ gene47"
        },
        {
          "x": 11,
          "y": 47,
          "value": 1,
          "name": "cell12 ~ gene48"
        },
        {
          "x": 11,
          "y": 48,
          "value": 0,
          "name": "cell12 ~ gene49"
        },
        {
          "x": 11,
          "y": 49,
          "value": 0,
          "name": "cell12 ~ gene50"
        },
        {
          "x": 11,
          "y": 50,
          "value": 15,
          "name": "cell12 ~ gene51"
        },
        {
          "x": 11,
          "y": 51,
          "value": 10,
          "name": "cell12 ~ gene52"
        },
        {
          "x": 11,
          "y": 52,
          "value": 3,
          "name": "cell12 ~ gene53"
        },
        {
          "x": 11,
          "y": 53,
          "value": 0,
          "name": "cell12 ~ gene54"
        },
        {
          "x": 11,
          "y": 54,
          "value": 12,
          "name": "cell12 ~ gene55"
        },
        {
          "x": 11,
          "y": 55,
          "value": 7,
          "name": "cell12 ~ gene56"
        },
        {
          "x": 11,
          "y": 56,
          "value": 0,
          "name": "cell12 ~ gene57"
        },
        {
          "x": 11,
          "y": 57,
          "value": 0,
          "name": "cell12 ~ gene58"
        },
        {
          "x": 11,
          "y": 58,
          "value": 0,
          "name": "cell12 ~ gene59"
        },
        {
          "x": 11,
          "y": 59,
          "value": 0,
          "name": "cell12 ~ gene60"
        },
        {
          "x": 11,
          "y": 60,
          "value": 4,
          "name": "cell12 ~ gene61"
        },
        {
          "x": 11,
          "y": 61,
          "value": 9,
          "name": "cell12 ~ gene62"
        },
        {
          "x": 11,
          "y": 62,
          "value": 2,
          "name": "cell12 ~ gene63"
        },
        {
          "x": 11,
          "y": 63,
          "value": 0,
          "name": "cell12 ~ gene64"
        },
        {
          "x": 11,
          "y": 64,
          "value": 4,
          "name": "cell12 ~ gene65"
        },
        {
          "x": 11,
          "y": 65,
          "value": 11,
          "name": "cell12 ~ gene66"
        },
        {
          "x": 11,
          "y": 66,
          "value": 0,
          "name": "cell12 ~ gene67"
        },
        {
          "x": 11,
          "y": 67,
          "value": 0,
          "name": "cell12 ~ gene68"
        },
        {
          "x": 11,
          "y": 68,
          "value": 8,
          "name": "cell12 ~ gene69"
        },
        {
          "x": 11,
          "y": 69,
          "value": 0,
          "name": "cell12 ~ gene70"
        },
        {
          "x": 11,
          "y": 70,
          "value": 5,
          "name": "cell12 ~ gene71"
        },
        {
          "x": 11,
          "y": 71,
          "value": 0,
          "name": "cell12 ~ gene72"
        },
        {
          "x": 11,
          "y": 72,
          "value": 17,
          "name": "cell12 ~ gene73"
        },
        {
          "x": 11,
          "y": 73,
          "value": 12,
          "name": "cell12 ~ gene74"
        },
        {
          "x": 11,
          "y": 74,
          "value": 7,
          "name": "cell12 ~ gene75"
        },
        {
          "x": 11,
          "y": 75,
          "value": 8,
          "name": "cell12 ~ gene76"
        },
        {
          "x": 11,
          "y": 76,
          "value": 10,
          "name": "cell12 ~ gene77"
        },
        {
          "x": 11,
          "y": 77,
          "value": 11,
          "name": "cell12 ~ gene78"
        },
        {
          "x": 11,
          "y": 78,
          "value": 0,
          "name": "cell12 ~ gene79"
        },
        {
          "x": 11,
          "y": 79,
          "value": 16,
          "name": "cell12 ~ gene80"
        },
        {
          "x": 11,
          "y": 80,
          "value": 0,
          "name": "cell12 ~ gene81"
        },
        {
          "x": 11,
          "y": 81,
          "value": 0,
          "name": "cell12 ~ gene82"
        },
        {
          "x": 11,
          "y": 82,
          "value": 0,
          "name": "cell12 ~ gene83"
        },
        {
          "x": 11,
          "y": 83,
          "value": 6,
          "name": "cell12 ~ gene84"
        },
        {
          "x": 11,
          "y": 84,
          "value": 4,
          "name": "cell12 ~ gene85"
        },
        {
          "x": 11,
          "y": 85,
          "value": 6,
          "name": "cell12 ~ gene86"
        },
        {
          "x": 11,
          "y": 86,
          "value": 1,
          "name": "cell12 ~ gene87"
        },
        {
          "x": 11,
          "y": 87,
          "value": 0,
          "name": "cell12 ~ gene88"
        },
        {
          "x": 11,
          "y": 88,
          "value": 1,
          "name": "cell12 ~ gene89"
        },
        {
          "x": 11,
          "y": 89,
          "value": 18,
          "name": "cell12 ~ gene90"
        },
        {
          "x": 11,
          "y": 90,
          "value": 0,
          "name": "cell12 ~ gene91"
        },
        {
          "x": 11,
          "y": 91,
          "value": 5,
          "name": "cell12 ~ gene92"
        },
        {
          "x": 11,
          "y": 92,
          "value": 5,
          "name": "cell12 ~ gene93"
        },
        {
          "x": 11,
          "y": 93,
          "value": 12,
          "name": "cell12 ~ gene94"
        },
        {
          "x": 11,
          "y": 94,
          "value": 0,
          "name": "cell12 ~ gene95"
        },
        {
          "x": 11,
          "y": 95,
          "value": 0,
          "name": "cell12 ~ gene96"
        },
        {
          "x": 11,
          "y": 96,
          "value": 3,
          "name": "cell12 ~ gene97"
        },
        {
          "x": 11,
          "y": 97,
          "value": 0,
          "name": "cell12 ~ gene98"
        },
        {
          "x": 11,
          "y": 98,
          "value": 2,
          "name": "cell12 ~ gene99"
        },
        {
          "x": 11,
          "y": 99,
          "value": 0,
          "name": "cell12 ~ gene100"
        },
        {
          "x": 12,
          "y": 0,
          "value": 0,
          "name": "cell13 ~ gene1"
        },
        {
          "x": 12,
          "y": 1,
          "value": 0,
          "name": "cell13 ~ gene2"
        },
        {
          "x": 12,
          "y": 2,
          "value": 27,
          "name": "cell13 ~ gene3"
        },
        {
          "x": 12,
          "y": 3,
          "value": 22,
          "name": "cell13 ~ gene4"
        },
        {
          "x": 12,
          "y": 4,
          "value": 27,
          "name": "cell13 ~ gene5"
        },
        {
          "x": 12,
          "y": 5,
          "value": 0,
          "name": "cell13 ~ gene6"
        },
        {
          "x": 12,
          "y": 6,
          "value": 37,
          "name": "cell13 ~ gene7"
        },
        {
          "x": 12,
          "y": 7,
          "value": 14,
          "name": "cell13 ~ gene8"
        },
        {
          "x": 12,
          "y": 8,
          "value": 0,
          "name": "cell13 ~ gene9"
        },
        {
          "x": 12,
          "y": 9,
          "value": 4,
          "name": "cell13 ~ gene10"
        },
        {
          "x": 12,
          "y": 10,
          "value": 17,
          "name": "cell13 ~ gene11"
        },
        {
          "x": 12,
          "y": 11,
          "value": 10,
          "name": "cell13 ~ gene12"
        },
        {
          "x": 12,
          "y": 12,
          "value": 23,
          "name": "cell13 ~ gene13"
        },
        {
          "x": 12,
          "y": 13,
          "value": 13,
          "name": "cell13 ~ gene14"
        },
        {
          "x": 12,
          "y": 14,
          "value": 26,
          "name": "cell13 ~ gene15"
        },
        {
          "x": 12,
          "y": 15,
          "value": 26,
          "name": "cell13 ~ gene16"
        },
        {
          "x": 12,
          "y": 16,
          "value": 18,
          "name": "cell13 ~ gene17"
        },
        {
          "x": 12,
          "y": 17,
          "value": 16,
          "name": "cell13 ~ gene18"
        },
        {
          "x": 12,
          "y": 18,
          "value": 36,
          "name": "cell13 ~ gene19"
        },
        {
          "x": 12,
          "y": 19,
          "value": 5,
          "name": "cell13 ~ gene20"
        },
        {
          "x": 12,
          "y": 20,
          "value": 14,
          "name": "cell13 ~ gene21"
        },
        {
          "x": 12,
          "y": 21,
          "value": 0,
          "name": "cell13 ~ gene22"
        },
        {
          "x": 12,
          "y": 22,
          "value": 0,
          "name": "cell13 ~ gene23"
        },
        {
          "x": 12,
          "y": 23,
          "value": 8,
          "name": "cell13 ~ gene24"
        },
        {
          "x": 12,
          "y": 24,
          "value": 11,
          "name": "cell13 ~ gene25"
        },
        {
          "x": 12,
          "y": 25,
          "value": 0,
          "name": "cell13 ~ gene26"
        },
        {
          "x": 12,
          "y": 26,
          "value": 8,
          "name": "cell13 ~ gene27"
        },
        {
          "x": 12,
          "y": 27,
          "value": 11,
          "name": "cell13 ~ gene28"
        },
        {
          "x": 12,
          "y": 28,
          "value": 0,
          "name": "cell13 ~ gene29"
        },
        {
          "x": 12,
          "y": 29,
          "value": 7,
          "name": "cell13 ~ gene30"
        },
        {
          "x": 12,
          "y": 30,
          "value": 0,
          "name": "cell13 ~ gene31"
        },
        {
          "x": 12,
          "y": 31,
          "value": 0,
          "name": "cell13 ~ gene32"
        },
        {
          "x": 12,
          "y": 32,
          "value": 0,
          "name": "cell13 ~ gene33"
        },
        {
          "x": 12,
          "y": 33,
          "value": 0,
          "name": "cell13 ~ gene34"
        },
        {
          "x": 12,
          "y": 34,
          "value": 15,
          "name": "cell13 ~ gene35"
        },
        {
          "x": 12,
          "y": 35,
          "value": 0,
          "name": "cell13 ~ gene36"
        },
        {
          "x": 12,
          "y": 36,
          "value": 0,
          "name": "cell13 ~ gene37"
        },
        {
          "x": 12,
          "y": 37,
          "value": 0,
          "name": "cell13 ~ gene38"
        },
        {
          "x": 12,
          "y": 38,
          "value": 1,
          "name": "cell13 ~ gene39"
        },
        {
          "x": 12,
          "y": 39,
          "value": 0,
          "name": "cell13 ~ gene40"
        },
        {
          "x": 12,
          "y": 40,
          "value": 0,
          "name": "cell13 ~ gene41"
        },
        {
          "x": 12,
          "y": 41,
          "value": 0,
          "name": "cell13 ~ gene42"
        },
        {
          "x": 12,
          "y": 42,
          "value": 0,
          "name": "cell13 ~ gene43"
        },
        {
          "x": 12,
          "y": 43,
          "value": 5,
          "name": "cell13 ~ gene44"
        },
        {
          "x": 12,
          "y": 44,
          "value": 2,
          "name": "cell13 ~ gene45"
        },
        {
          "x": 12,
          "y": 45,
          "value": 4,
          "name": "cell13 ~ gene46"
        },
        {
          "x": 12,
          "y": 46,
          "value": 0,
          "name": "cell13 ~ gene47"
        },
        {
          "x": 12,
          "y": 47,
          "value": 0,
          "name": "cell13 ~ gene48"
        },
        {
          "x": 12,
          "y": 48,
          "value": 19,
          "name": "cell13 ~ gene49"
        },
        {
          "x": 12,
          "y": 49,
          "value": 14,
          "name": "cell13 ~ gene50"
        },
        {
          "x": 12,
          "y": 50,
          "value": 0,
          "name": "cell13 ~ gene51"
        },
        {
          "x": 12,
          "y": 51,
          "value": 0,
          "name": "cell13 ~ gene52"
        },
        {
          "x": 12,
          "y": 52,
          "value": 0,
          "name": "cell13 ~ gene53"
        },
        {
          "x": 12,
          "y": 53,
          "value": 0,
          "name": "cell13 ~ gene54"
        },
        {
          "x": 12,
          "y": 54,
          "value": 0,
          "name": "cell13 ~ gene55"
        },
        {
          "x": 12,
          "y": 55,
          "value": 0,
          "name": "cell13 ~ gene56"
        },
        {
          "x": 12,
          "y": 56,
          "value": 1,
          "name": "cell13 ~ gene57"
        },
        {
          "x": 12,
          "y": 57,
          "value": 17,
          "name": "cell13 ~ gene58"
        },
        {
          "x": 12,
          "y": 58,
          "value": 0,
          "name": "cell13 ~ gene59"
        },
        {
          "x": 12,
          "y": 59,
          "value": 24,
          "name": "cell13 ~ gene60"
        },
        {
          "x": 12,
          "y": 60,
          "value": 13,
          "name": "cell13 ~ gene61"
        },
        {
          "x": 12,
          "y": 61,
          "value": 0,
          "name": "cell13 ~ gene62"
        },
        {
          "x": 12,
          "y": 62,
          "value": 0,
          "name": "cell13 ~ gene63"
        },
        {
          "x": 12,
          "y": 63,
          "value": 4,
          "name": "cell13 ~ gene64"
        },
        {
          "x": 12,
          "y": 64,
          "value": 13,
          "name": "cell13 ~ gene65"
        },
        {
          "x": 12,
          "y": 65,
          "value": 14,
          "name": "cell13 ~ gene66"
        },
        {
          "x": 12,
          "y": 66,
          "value": 2,
          "name": "cell13 ~ gene67"
        },
        {
          "x": 12,
          "y": 67,
          "value": 0,
          "name": "cell13 ~ gene68"
        },
        {
          "x": 12,
          "y": 68,
          "value": 11,
          "name": "cell13 ~ gene69"
        },
        {
          "x": 12,
          "y": 69,
          "value": 10,
          "name": "cell13 ~ gene70"
        },
        {
          "x": 12,
          "y": 70,
          "value": 0,
          "name": "cell13 ~ gene71"
        },
        {
          "x": 12,
          "y": 71,
          "value": 0,
          "name": "cell13 ~ gene72"
        },
        {
          "x": 12,
          "y": 72,
          "value": 0,
          "name": "cell13 ~ gene73"
        },
        {
          "x": 12,
          "y": 73,
          "value": 3,
          "name": "cell13 ~ gene74"
        },
        {
          "x": 12,
          "y": 74,
          "value": 0,
          "name": "cell13 ~ gene75"
        },
        {
          "x": 12,
          "y": 75,
          "value": 0,
          "name": "cell13 ~ gene76"
        },
        {
          "x": 12,
          "y": 76,
          "value": 0,
          "name": "cell13 ~ gene77"
        },
        {
          "x": 12,
          "y": 77,
          "value": 8,
          "name": "cell13 ~ gene78"
        },
        {
          "x": 12,
          "y": 78,
          "value": 7,
          "name": "cell13 ~ gene79"
        },
        {
          "x": 12,
          "y": 79,
          "value": 3,
          "name": "cell13 ~ gene80"
        },
        {
          "x": 12,
          "y": 80,
          "value": 0,
          "name": "cell13 ~ gene81"
        },
        {
          "x": 12,
          "y": 81,
          "value": 0,
          "name": "cell13 ~ gene82"
        },
        {
          "x": 12,
          "y": 82,
          "value": 0,
          "name": "cell13 ~ gene83"
        },
        {
          "x": 12,
          "y": 83,
          "value": 9,
          "name": "cell13 ~ gene84"
        },
        {
          "x": 12,
          "y": 84,
          "value": 0,
          "name": "cell13 ~ gene85"
        },
        {
          "x": 12,
          "y": 85,
          "value": 0,
          "name": "cell13 ~ gene86"
        },
        {
          "x": 12,
          "y": 86,
          "value": 0,
          "name": "cell13 ~ gene87"
        },
        {
          "x": 12,
          "y": 87,
          "value": 0,
          "name": "cell13 ~ gene88"
        },
        {
          "x": 12,
          "y": 88,
          "value": 25,
          "name": "cell13 ~ gene89"
        },
        {
          "x": 12,
          "y": 89,
          "value": 22,
          "name": "cell13 ~ gene90"
        },
        {
          "x": 12,
          "y": 90,
          "value": 0,
          "name": "cell13 ~ gene91"
        },
        {
          "x": 12,
          "y": 91,
          "value": 4,
          "name": "cell13 ~ gene92"
        },
        {
          "x": 12,
          "y": 92,
          "value": 3,
          "name": "cell13 ~ gene93"
        },
        {
          "x": 12,
          "y": 93,
          "value": 4,
          "name": "cell13 ~ gene94"
        },
        {
          "x": 12,
          "y": 94,
          "value": 5,
          "name": "cell13 ~ gene95"
        },
        {
          "x": 12,
          "y": 95,
          "value": 0,
          "name": "cell13 ~ gene96"
        },
        {
          "x": 12,
          "y": 96,
          "value": 0,
          "name": "cell13 ~ gene97"
        },
        {
          "x": 12,
          "y": 97,
          "value": 0,
          "name": "cell13 ~ gene98"
        },
        {
          "x": 12,
          "y": 98,
          "value": 7,
          "name": "cell13 ~ gene99"
        },
        {
          "x": 12,
          "y": 99,
          "value": 0,
          "name": "cell13 ~ gene100"
        },
        {
          "x": 13,
          "y": 0,
          "value": 9,
          "name": "cell14 ~ gene1"
        },
        {
          "x": 13,
          "y": 1,
          "value": 0,
          "name": "cell14 ~ gene2"
        },
        {
          "x": 13,
          "y": 2,
          "value": 21,
          "name": "cell14 ~ gene3"
        },
        {
          "x": 13,
          "y": 3,
          "value": 26,
          "name": "cell14 ~ gene4"
        },
        {
          "x": 13,
          "y": 4,
          "value": 20,
          "name": "cell14 ~ gene5"
        },
        {
          "x": 13,
          "y": 5,
          "value": 1,
          "name": "cell14 ~ gene6"
        },
        {
          "x": 13,
          "y": 6,
          "value": 38,
          "name": "cell14 ~ gene7"
        },
        {
          "x": 13,
          "y": 7,
          "value": 10,
          "name": "cell14 ~ gene8"
        },
        {
          "x": 13,
          "y": 8,
          "value": 0,
          "name": "cell14 ~ gene9"
        },
        {
          "x": 13,
          "y": 9,
          "value": 0,
          "name": "cell14 ~ gene10"
        },
        {
          "x": 13,
          "y": 10,
          "value": 15,
          "name": "cell14 ~ gene11"
        },
        {
          "x": 13,
          "y": 11,
          "value": 0,
          "name": "cell14 ~ gene12"
        },
        {
          "x": 13,
          "y": 12,
          "value": 0,
          "name": "cell14 ~ gene13"
        },
        {
          "x": 13,
          "y": 13,
          "value": 1,
          "name": "cell14 ~ gene14"
        },
        {
          "x": 13,
          "y": 14,
          "value": 24,
          "name": "cell14 ~ gene15"
        },
        {
          "x": 13,
          "y": 15,
          "value": 33,
          "name": "cell14 ~ gene16"
        },
        {
          "x": 13,
          "y": 16,
          "value": 28,
          "name": "cell14 ~ gene17"
        },
        {
          "x": 13,
          "y": 17,
          "value": 14,
          "name": "cell14 ~ gene18"
        },
        {
          "x": 13,
          "y": 18,
          "value": 13,
          "name": "cell14 ~ gene19"
        },
        {
          "x": 13,
          "y": 19,
          "value": 0,
          "name": "cell14 ~ gene20"
        },
        {
          "x": 13,
          "y": 20,
          "value": 10,
          "name": "cell14 ~ gene21"
        },
        {
          "x": 13,
          "y": 21,
          "value": 0,
          "name": "cell14 ~ gene22"
        },
        {
          "x": 13,
          "y": 22,
          "value": 0,
          "name": "cell14 ~ gene23"
        },
        {
          "x": 13,
          "y": 23,
          "value": 21,
          "name": "cell14 ~ gene24"
        },
        {
          "x": 13,
          "y": 24,
          "value": 0,
          "name": "cell14 ~ gene25"
        },
        {
          "x": 13,
          "y": 25,
          "value": 0,
          "name": "cell14 ~ gene26"
        },
        {
          "x": 13,
          "y": 26,
          "value": 0,
          "name": "cell14 ~ gene27"
        },
        {
          "x": 13,
          "y": 27,
          "value": 0,
          "name": "cell14 ~ gene28"
        },
        {
          "x": 13,
          "y": 28,
          "value": 2,
          "name": "cell14 ~ gene29"
        },
        {
          "x": 13,
          "y": 29,
          "value": 0,
          "name": "cell14 ~ gene30"
        },
        {
          "x": 13,
          "y": 30,
          "value": 0,
          "name": "cell14 ~ gene31"
        },
        {
          "x": 13,
          "y": 31,
          "value": 0,
          "name": "cell14 ~ gene32"
        },
        {
          "x": 13,
          "y": 32,
          "value": 0,
          "name": "cell14 ~ gene33"
        },
        {
          "x": 13,
          "y": 33,
          "value": 11,
          "name": "cell14 ~ gene34"
        },
        {
          "x": 13,
          "y": 34,
          "value": 7,
          "name": "cell14 ~ gene35"
        },
        {
          "x": 13,
          "y": 35,
          "value": 18,
          "name": "cell14 ~ gene36"
        },
        {
          "x": 13,
          "y": 36,
          "value": 10,
          "name": "cell14 ~ gene37"
        },
        {
          "x": 13,
          "y": 37,
          "value": 1,
          "name": "cell14 ~ gene38"
        },
        {
          "x": 13,
          "y": 38,
          "value": 0,
          "name": "cell14 ~ gene39"
        },
        {
          "x": 13,
          "y": 39,
          "value": 2,
          "name": "cell14 ~ gene40"
        },
        {
          "x": 13,
          "y": 40,
          "value": 3,
          "name": "cell14 ~ gene41"
        },
        {
          "x": 13,
          "y": 41,
          "value": 0,
          "name": "cell14 ~ gene42"
        },
        {
          "x": 13,
          "y": 42,
          "value": 0,
          "name": "cell14 ~ gene43"
        },
        {
          "x": 13,
          "y": 43,
          "value": 7,
          "name": "cell14 ~ gene44"
        },
        {
          "x": 13,
          "y": 44,
          "value": 0,
          "name": "cell14 ~ gene45"
        },
        {
          "x": 13,
          "y": 45,
          "value": 0,
          "name": "cell14 ~ gene46"
        },
        {
          "x": 13,
          "y": 46,
          "value": 0,
          "name": "cell14 ~ gene47"
        },
        {
          "x": 13,
          "y": 47,
          "value": 0,
          "name": "cell14 ~ gene48"
        },
        {
          "x": 13,
          "y": 48,
          "value": 9,
          "name": "cell14 ~ gene49"
        },
        {
          "x": 13,
          "y": 49,
          "value": 0,
          "name": "cell14 ~ gene50"
        },
        {
          "x": 13,
          "y": 50,
          "value": 11,
          "name": "cell14 ~ gene51"
        },
        {
          "x": 13,
          "y": 51,
          "value": 3,
          "name": "cell14 ~ gene52"
        },
        {
          "x": 13,
          "y": 52,
          "value": 2,
          "name": "cell14 ~ gene53"
        },
        {
          "x": 13,
          "y": 53,
          "value": 0,
          "name": "cell14 ~ gene54"
        },
        {
          "x": 13,
          "y": 54,
          "value": 12,
          "name": "cell14 ~ gene55"
        },
        {
          "x": 13,
          "y": 55,
          "value": 0,
          "name": "cell14 ~ gene56"
        },
        {
          "x": 13,
          "y": 56,
          "value": 0,
          "name": "cell14 ~ gene57"
        },
        {
          "x": 13,
          "y": 57,
          "value": 2,
          "name": "cell14 ~ gene58"
        },
        {
          "x": 13,
          "y": 58,
          "value": 2,
          "name": "cell14 ~ gene59"
        },
        {
          "x": 13,
          "y": 59,
          "value": 20,
          "name": "cell14 ~ gene60"
        },
        {
          "x": 13,
          "y": 60,
          "value": 0,
          "name": "cell14 ~ gene61"
        },
        {
          "x": 13,
          "y": 61,
          "value": 8,
          "name": "cell14 ~ gene62"
        },
        {
          "x": 13,
          "y": 62,
          "value": 0,
          "name": "cell14 ~ gene63"
        },
        {
          "x": 13,
          "y": 63,
          "value": 15,
          "name": "cell14 ~ gene64"
        },
        {
          "x": 13,
          "y": 64,
          "value": 6,
          "name": "cell14 ~ gene65"
        },
        {
          "x": 13,
          "y": 65,
          "value": 6,
          "name": "cell14 ~ gene66"
        },
        {
          "x": 13,
          "y": 66,
          "value": 3,
          "name": "cell14 ~ gene67"
        },
        {
          "x": 13,
          "y": 67,
          "value": 0,
          "name": "cell14 ~ gene68"
        },
        {
          "x": 13,
          "y": 68,
          "value": 0,
          "name": "cell14 ~ gene69"
        },
        {
          "x": 13,
          "y": 69,
          "value": 6,
          "name": "cell14 ~ gene70"
        },
        {
          "x": 13,
          "y": 70,
          "value": 0,
          "name": "cell14 ~ gene71"
        },
        {
          "x": 13,
          "y": 71,
          "value": 12,
          "name": "cell14 ~ gene72"
        },
        {
          "x": 13,
          "y": 72,
          "value": 0,
          "name": "cell14 ~ gene73"
        },
        {
          "x": 13,
          "y": 73,
          "value": 0,
          "name": "cell14 ~ gene74"
        },
        {
          "x": 13,
          "y": 74,
          "value": 0,
          "name": "cell14 ~ gene75"
        },
        {
          "x": 13,
          "y": 75,
          "value": 1,
          "name": "cell14 ~ gene76"
        },
        {
          "x": 13,
          "y": 76,
          "value": 11,
          "name": "cell14 ~ gene77"
        },
        {
          "x": 13,
          "y": 77,
          "value": 0,
          "name": "cell14 ~ gene78"
        },
        {
          "x": 13,
          "y": 78,
          "value": 0,
          "name": "cell14 ~ gene79"
        },
        {
          "x": 13,
          "y": 79,
          "value": 1,
          "name": "cell14 ~ gene80"
        },
        {
          "x": 13,
          "y": 80,
          "value": 0,
          "name": "cell14 ~ gene81"
        },
        {
          "x": 13,
          "y": 81,
          "value": 0,
          "name": "cell14 ~ gene82"
        },
        {
          "x": 13,
          "y": 82,
          "value": 0,
          "name": "cell14 ~ gene83"
        },
        {
          "x": 13,
          "y": 83,
          "value": 10,
          "name": "cell14 ~ gene84"
        },
        {
          "x": 13,
          "y": 84,
          "value": 7,
          "name": "cell14 ~ gene85"
        },
        {
          "x": 13,
          "y": 85,
          "value": 0,
          "name": "cell14 ~ gene86"
        },
        {
          "x": 13,
          "y": 86,
          "value": 0,
          "name": "cell14 ~ gene87"
        },
        {
          "x": 13,
          "y": 87,
          "value": 0,
          "name": "cell14 ~ gene88"
        },
        {
          "x": 13,
          "y": 88,
          "value": 0,
          "name": "cell14 ~ gene89"
        },
        {
          "x": 13,
          "y": 89,
          "value": 0,
          "name": "cell14 ~ gene90"
        },
        {
          "x": 13,
          "y": 90,
          "value": 8,
          "name": "cell14 ~ gene91"
        },
        {
          "x": 13,
          "y": 91,
          "value": 0,
          "name": "cell14 ~ gene92"
        },
        {
          "x": 13,
          "y": 92,
          "value": 8,
          "name": "cell14 ~ gene93"
        },
        {
          "x": 13,
          "y": 93,
          "value": 8,
          "name": "cell14 ~ gene94"
        },
        {
          "x": 13,
          "y": 94,
          "value": 0,
          "name": "cell14 ~ gene95"
        },
        {
          "x": 13,
          "y": 95,
          "value": 0,
          "name": "cell14 ~ gene96"
        },
        {
          "x": 13,
          "y": 96,
          "value": 19,
          "name": "cell14 ~ gene97"
        },
        {
          "x": 13,
          "y": 97,
          "value": 0,
          "name": "cell14 ~ gene98"
        },
        {
          "x": 13,
          "y": 98,
          "value": 2,
          "name": "cell14 ~ gene99"
        },
        {
          "x": 13,
          "y": 99,
          "value": 11,
          "name": "cell14 ~ gene100"
        },
        {
          "x": 14,
          "y": 0,
          "value": 5,
          "name": "cell15 ~ gene1"
        },
        {
          "x": 14,
          "y": 1,
          "value": 1,
          "name": "cell15 ~ gene2"
        },
        {
          "x": 14,
          "y": 2,
          "value": 6,
          "name": "cell15 ~ gene3"
        },
        {
          "x": 14,
          "y": 3,
          "value": 41,
          "name": "cell15 ~ gene4"
        },
        {
          "x": 14,
          "y": 4,
          "value": 11,
          "name": "cell15 ~ gene5"
        },
        {
          "x": 14,
          "y": 5,
          "value": 0,
          "name": "cell15 ~ gene6"
        },
        {
          "x": 14,
          "y": 6,
          "value": 20,
          "name": "cell15 ~ gene7"
        },
        {
          "x": 14,
          "y": 7,
          "value": 28,
          "name": "cell15 ~ gene8"
        },
        {
          "x": 14,
          "y": 8,
          "value": 0,
          "name": "cell15 ~ gene9"
        },
        {
          "x": 14,
          "y": 9,
          "value": 0,
          "name": "cell15 ~ gene10"
        },
        {
          "x": 14,
          "y": 10,
          "value": 2,
          "name": "cell15 ~ gene11"
        },
        {
          "x": 14,
          "y": 11,
          "value": 0,
          "name": "cell15 ~ gene12"
        },
        {
          "x": 14,
          "y": 12,
          "value": 18,
          "name": "cell15 ~ gene13"
        },
        {
          "x": 14,
          "y": 13,
          "value": 16,
          "name": "cell15 ~ gene14"
        },
        {
          "x": 14,
          "y": 14,
          "value": 42,
          "name": "cell15 ~ gene15"
        },
        {
          "x": 14,
          "y": 15,
          "value": 28,
          "name": "cell15 ~ gene16"
        },
        {
          "x": 14,
          "y": 16,
          "value": 17,
          "name": "cell15 ~ gene17"
        },
        {
          "x": 14,
          "y": 17,
          "value": 0,
          "name": "cell15 ~ gene18"
        },
        {
          "x": 14,
          "y": 18,
          "value": 24,
          "name": "cell15 ~ gene19"
        },
        {
          "x": 14,
          "y": 19,
          "value": 20,
          "name": "cell15 ~ gene20"
        },
        {
          "x": 14,
          "y": 20,
          "value": 5,
          "name": "cell15 ~ gene21"
        },
        {
          "x": 14,
          "y": 21,
          "value": 0,
          "name": "cell15 ~ gene22"
        },
        {
          "x": 14,
          "y": 22,
          "value": 0,
          "name": "cell15 ~ gene23"
        },
        {
          "x": 14,
          "y": 23,
          "value": 0,
          "name": "cell15 ~ gene24"
        },
        {
          "x": 14,
          "y": 24,
          "value": 5,
          "name": "cell15 ~ gene25"
        },
        {
          "x": 14,
          "y": 25,
          "value": 4,
          "name": "cell15 ~ gene26"
        },
        {
          "x": 14,
          "y": 26,
          "value": 15,
          "name": "cell15 ~ gene27"
        },
        {
          "x": 14,
          "y": 27,
          "value": 0,
          "name": "cell15 ~ gene28"
        },
        {
          "x": 14,
          "y": 28,
          "value": 0,
          "name": "cell15 ~ gene29"
        },
        {
          "x": 14,
          "y": 29,
          "value": 0,
          "name": "cell15 ~ gene30"
        },
        {
          "x": 14,
          "y": 30,
          "value": 0,
          "name": "cell15 ~ gene31"
        },
        {
          "x": 14,
          "y": 31,
          "value": 0,
          "name": "cell15 ~ gene32"
        },
        {
          "x": 14,
          "y": 32,
          "value": 0,
          "name": "cell15 ~ gene33"
        },
        {
          "x": 14,
          "y": 33,
          "value": 3,
          "name": "cell15 ~ gene34"
        },
        {
          "x": 14,
          "y": 34,
          "value": 0,
          "name": "cell15 ~ gene35"
        },
        {
          "x": 14,
          "y": 35,
          "value": 0,
          "name": "cell15 ~ gene36"
        },
        {
          "x": 14,
          "y": 36,
          "value": 11,
          "name": "cell15 ~ gene37"
        },
        {
          "x": 14,
          "y": 37,
          "value": 0,
          "name": "cell15 ~ gene38"
        },
        {
          "x": 14,
          "y": 38,
          "value": 0,
          "name": "cell15 ~ gene39"
        },
        {
          "x": 14,
          "y": 39,
          "value": 0,
          "name": "cell15 ~ gene40"
        },
        {
          "x": 14,
          "y": 40,
          "value": 0,
          "name": "cell15 ~ gene41"
        },
        {
          "x": 14,
          "y": 41,
          "value": 6,
          "name": "cell15 ~ gene42"
        },
        {
          "x": 14,
          "y": 42,
          "value": 3,
          "name": "cell15 ~ gene43"
        },
        {
          "x": 14,
          "y": 43,
          "value": 0,
          "name": "cell15 ~ gene44"
        },
        {
          "x": 14,
          "y": 44,
          "value": 3,
          "name": "cell15 ~ gene45"
        },
        {
          "x": 14,
          "y": 45,
          "value": 0,
          "name": "cell15 ~ gene46"
        },
        {
          "x": 14,
          "y": 46,
          "value": 8,
          "name": "cell15 ~ gene47"
        },
        {
          "x": 14,
          "y": 47,
          "value": 0,
          "name": "cell15 ~ gene48"
        },
        {
          "x": 14,
          "y": 48,
          "value": 3,
          "name": "cell15 ~ gene49"
        },
        {
          "x": 14,
          "y": 49,
          "value": 13,
          "name": "cell15 ~ gene50"
        },
        {
          "x": 14,
          "y": 50,
          "value": 18,
          "name": "cell15 ~ gene51"
        },
        {
          "x": 14,
          "y": 51,
          "value": 0,
          "name": "cell15 ~ gene52"
        },
        {
          "x": 14,
          "y": 52,
          "value": 0,
          "name": "cell15 ~ gene53"
        },
        {
          "x": 14,
          "y": 53,
          "value": 0,
          "name": "cell15 ~ gene54"
        },
        {
          "x": 14,
          "y": 54,
          "value": 5,
          "name": "cell15 ~ gene55"
        },
        {
          "x": 14,
          "y": 55,
          "value": 3,
          "name": "cell15 ~ gene56"
        },
        {
          "x": 14,
          "y": 56,
          "value": 0,
          "name": "cell15 ~ gene57"
        },
        {
          "x": 14,
          "y": 57,
          "value": 0,
          "name": "cell15 ~ gene58"
        },
        {
          "x": 14,
          "y": 58,
          "value": 24,
          "name": "cell15 ~ gene59"
        },
        {
          "x": 14,
          "y": 59,
          "value": 0,
          "name": "cell15 ~ gene60"
        },
        {
          "x": 14,
          "y": 60,
          "value": 3,
          "name": "cell15 ~ gene61"
        },
        {
          "x": 14,
          "y": 61,
          "value": 0,
          "name": "cell15 ~ gene62"
        },
        {
          "x": 14,
          "y": 62,
          "value": 15,
          "name": "cell15 ~ gene63"
        },
        {
          "x": 14,
          "y": 63,
          "value": 12,
          "name": "cell15 ~ gene64"
        },
        {
          "x": 14,
          "y": 64,
          "value": 1,
          "name": "cell15 ~ gene65"
        },
        {
          "x": 14,
          "y": 65,
          "value": 1,
          "name": "cell15 ~ gene66"
        },
        {
          "x": 14,
          "y": 66,
          "value": 9,
          "name": "cell15 ~ gene67"
        },
        {
          "x": 14,
          "y": 67,
          "value": 0,
          "name": "cell15 ~ gene68"
        },
        {
          "x": 14,
          "y": 68,
          "value": 0,
          "name": "cell15 ~ gene69"
        },
        {
          "x": 14,
          "y": 69,
          "value": 0,
          "name": "cell15 ~ gene70"
        },
        {
          "x": 14,
          "y": 70,
          "value": 5,
          "name": "cell15 ~ gene71"
        },
        {
          "x": 14,
          "y": 71,
          "value": 28,
          "name": "cell15 ~ gene72"
        },
        {
          "x": 14,
          "y": 72,
          "value": 3,
          "name": "cell15 ~ gene73"
        },
        {
          "x": 14,
          "y": 73,
          "value": 0,
          "name": "cell15 ~ gene74"
        },
        {
          "x": 14,
          "y": 74,
          "value": 8,
          "name": "cell15 ~ gene75"
        },
        {
          "x": 14,
          "y": 75,
          "value": 6,
          "name": "cell15 ~ gene76"
        },
        {
          "x": 14,
          "y": 76,
          "value": 4,
          "name": "cell15 ~ gene77"
        },
        {
          "x": 14,
          "y": 77,
          "value": 1,
          "name": "cell15 ~ gene78"
        },
        {
          "x": 14,
          "y": 78,
          "value": 6,
          "name": "cell15 ~ gene79"
        },
        {
          "x": 14,
          "y": 79,
          "value": 0,
          "name": "cell15 ~ gene80"
        },
        {
          "x": 14,
          "y": 80,
          "value": 0,
          "name": "cell15 ~ gene81"
        },
        {
          "x": 14,
          "y": 81,
          "value": 6,
          "name": "cell15 ~ gene82"
        },
        {
          "x": 14,
          "y": 82,
          "value": 3,
          "name": "cell15 ~ gene83"
        },
        {
          "x": 14,
          "y": 83,
          "value": 11,
          "name": "cell15 ~ gene84"
        },
        {
          "x": 14,
          "y": 84,
          "value": 0,
          "name": "cell15 ~ gene85"
        },
        {
          "x": 14,
          "y": 85,
          "value": 0,
          "name": "cell15 ~ gene86"
        },
        {
          "x": 14,
          "y": 86,
          "value": 0,
          "name": "cell15 ~ gene87"
        },
        {
          "x": 14,
          "y": 87,
          "value": 8,
          "name": "cell15 ~ gene88"
        },
        {
          "x": 14,
          "y": 88,
          "value": 11,
          "name": "cell15 ~ gene89"
        },
        {
          "x": 14,
          "y": 89,
          "value": 0,
          "name": "cell15 ~ gene90"
        },
        {
          "x": 14,
          "y": 90,
          "value": 0,
          "name": "cell15 ~ gene91"
        },
        {
          "x": 14,
          "y": 91,
          "value": 0,
          "name": "cell15 ~ gene92"
        },
        {
          "x": 14,
          "y": 92,
          "value": 9,
          "name": "cell15 ~ gene93"
        },
        {
          "x": 14,
          "y": 93,
          "value": 0,
          "name": "cell15 ~ gene94"
        },
        {
          "x": 14,
          "y": 94,
          "value": 1,
          "name": "cell15 ~ gene95"
        },
        {
          "x": 14,
          "y": 95,
          "value": 7,
          "name": "cell15 ~ gene96"
        },
        {
          "x": 14,
          "y": 96,
          "value": 0,
          "name": "cell15 ~ gene97"
        },
        {
          "x": 14,
          "y": 97,
          "value": 2,
          "name": "cell15 ~ gene98"
        },
        {
          "x": 14,
          "y": 98,
          "value": 0,
          "name": "cell15 ~ gene99"
        },
        {
          "x": 14,
          "y": 99,
          "value": 0,
          "name": "cell15 ~ gene100"
        },
        {
          "x": 15,
          "y": 0,
          "value": 7,
          "name": "cell16 ~ gene1"
        },
        {
          "x": 15,
          "y": 1,
          "value": 0,
          "name": "cell16 ~ gene2"
        },
        {
          "x": 15,
          "y": 2,
          "value": 16,
          "name": "cell16 ~ gene3"
        },
        {
          "x": 15,
          "y": 3,
          "value": 37,
          "name": "cell16 ~ gene4"
        },
        {
          "x": 15,
          "y": 4,
          "value": 15,
          "name": "cell16 ~ gene5"
        },
        {
          "x": 15,
          "y": 5,
          "value": 29,
          "name": "cell16 ~ gene6"
        },
        {
          "x": 15,
          "y": 6,
          "value": 15,
          "name": "cell16 ~ gene7"
        },
        {
          "x": 15,
          "y": 7,
          "value": 20,
          "name": "cell16 ~ gene8"
        },
        {
          "x": 15,
          "y": 8,
          "value": 10,
          "name": "cell16 ~ gene9"
        },
        {
          "x": 15,
          "y": 9,
          "value": 0,
          "name": "cell16 ~ gene10"
        },
        {
          "x": 15,
          "y": 10,
          "value": 29,
          "name": "cell16 ~ gene11"
        },
        {
          "x": 15,
          "y": 11,
          "value": 0,
          "name": "cell16 ~ gene12"
        },
        {
          "x": 15,
          "y": 12,
          "value": 7,
          "name": "cell16 ~ gene13"
        },
        {
          "x": 15,
          "y": 13,
          "value": 6,
          "name": "cell16 ~ gene14"
        },
        {
          "x": 15,
          "y": 14,
          "value": 14,
          "name": "cell16 ~ gene15"
        },
        {
          "x": 15,
          "y": 15,
          "value": 29,
          "name": "cell16 ~ gene16"
        },
        {
          "x": 15,
          "y": 16,
          "value": 7,
          "name": "cell16 ~ gene17"
        },
        {
          "x": 15,
          "y": 17,
          "value": 20,
          "name": "cell16 ~ gene18"
        },
        {
          "x": 15,
          "y": 18,
          "value": 19,
          "name": "cell16 ~ gene19"
        },
        {
          "x": 15,
          "y": 19,
          "value": 0,
          "name": "cell16 ~ gene20"
        },
        {
          "x": 15,
          "y": 20,
          "value": 0,
          "name": "cell16 ~ gene21"
        },
        {
          "x": 15,
          "y": 21,
          "value": 0,
          "name": "cell16 ~ gene22"
        },
        {
          "x": 15,
          "y": 22,
          "value": 0,
          "name": "cell16 ~ gene23"
        },
        {
          "x": 15,
          "y": 23,
          "value": 0,
          "name": "cell16 ~ gene24"
        },
        {
          "x": 15,
          "y": 24,
          "value": 0,
          "name": "cell16 ~ gene25"
        },
        {
          "x": 15,
          "y": 25,
          "value": 16,
          "name": "cell16 ~ gene26"
        },
        {
          "x": 15,
          "y": 26,
          "value": 0,
          "name": "cell16 ~ gene27"
        },
        {
          "x": 15,
          "y": 27,
          "value": 0,
          "name": "cell16 ~ gene28"
        },
        {
          "x": 15,
          "y": 28,
          "value": 0,
          "name": "cell16 ~ gene29"
        },
        {
          "x": 15,
          "y": 29,
          "value": 0,
          "name": "cell16 ~ gene30"
        },
        {
          "x": 15,
          "y": 30,
          "value": 0,
          "name": "cell16 ~ gene31"
        },
        {
          "x": 15,
          "y": 31,
          "value": 6,
          "name": "cell16 ~ gene32"
        },
        {
          "x": 15,
          "y": 32,
          "value": 3,
          "name": "cell16 ~ gene33"
        },
        {
          "x": 15,
          "y": 33,
          "value": 0,
          "name": "cell16 ~ gene34"
        },
        {
          "x": 15,
          "y": 34,
          "value": 0,
          "name": "cell16 ~ gene35"
        },
        {
          "x": 15,
          "y": 35,
          "value": 22,
          "name": "cell16 ~ gene36"
        },
        {
          "x": 15,
          "y": 36,
          "value": 0,
          "name": "cell16 ~ gene37"
        },
        {
          "x": 15,
          "y": 37,
          "value": 8,
          "name": "cell16 ~ gene38"
        },
        {
          "x": 15,
          "y": 38,
          "value": 6,
          "name": "cell16 ~ gene39"
        },
        {
          "x": 15,
          "y": 39,
          "value": 6,
          "name": "cell16 ~ gene40"
        },
        {
          "x": 15,
          "y": 40,
          "value": 9,
          "name": "cell16 ~ gene41"
        },
        {
          "x": 15,
          "y": 41,
          "value": 0,
          "name": "cell16 ~ gene42"
        },
        {
          "x": 15,
          "y": 42,
          "value": 10,
          "name": "cell16 ~ gene43"
        },
        {
          "x": 15,
          "y": 43,
          "value": 5,
          "name": "cell16 ~ gene44"
        },
        {
          "x": 15,
          "y": 44,
          "value": 10,
          "name": "cell16 ~ gene45"
        },
        {
          "x": 15,
          "y": 45,
          "value": 0,
          "name": "cell16 ~ gene46"
        },
        {
          "x": 15,
          "y": 46,
          "value": 5,
          "name": "cell16 ~ gene47"
        },
        {
          "x": 15,
          "y": 47,
          "value": 13,
          "name": "cell16 ~ gene48"
        },
        {
          "x": 15,
          "y": 48,
          "value": 1,
          "name": "cell16 ~ gene49"
        },
        {
          "x": 15,
          "y": 49,
          "value": 0,
          "name": "cell16 ~ gene50"
        },
        {
          "x": 15,
          "y": 50,
          "value": 12,
          "name": "cell16 ~ gene51"
        },
        {
          "x": 15,
          "y": 51,
          "value": 0,
          "name": "cell16 ~ gene52"
        },
        {
          "x": 15,
          "y": 52,
          "value": 0,
          "name": "cell16 ~ gene53"
        },
        {
          "x": 15,
          "y": 53,
          "value": 0,
          "name": "cell16 ~ gene54"
        },
        {
          "x": 15,
          "y": 54,
          "value": 0,
          "name": "cell16 ~ gene55"
        },
        {
          "x": 15,
          "y": 55,
          "value": 17,
          "name": "cell16 ~ gene56"
        },
        {
          "x": 15,
          "y": 56,
          "value": 0,
          "name": "cell16 ~ gene57"
        },
        {
          "x": 15,
          "y": 57,
          "value": 0,
          "name": "cell16 ~ gene58"
        },
        {
          "x": 15,
          "y": 58,
          "value": 0,
          "name": "cell16 ~ gene59"
        },
        {
          "x": 15,
          "y": 59,
          "value": 7,
          "name": "cell16 ~ gene60"
        },
        {
          "x": 15,
          "y": 60,
          "value": 3,
          "name": "cell16 ~ gene61"
        },
        {
          "x": 15,
          "y": 61,
          "value": 0,
          "name": "cell16 ~ gene62"
        },
        {
          "x": 15,
          "y": 62,
          "value": 0,
          "name": "cell16 ~ gene63"
        },
        {
          "x": 15,
          "y": 63,
          "value": 0,
          "name": "cell16 ~ gene64"
        },
        {
          "x": 15,
          "y": 64,
          "value": 17,
          "name": "cell16 ~ gene65"
        },
        {
          "x": 15,
          "y": 65,
          "value": 9,
          "name": "cell16 ~ gene66"
        },
        {
          "x": 15,
          "y": 66,
          "value": 16,
          "name": "cell16 ~ gene67"
        },
        {
          "x": 15,
          "y": 67,
          "value": 9,
          "name": "cell16 ~ gene68"
        },
        {
          "x": 15,
          "y": 68,
          "value": 0,
          "name": "cell16 ~ gene69"
        },
        {
          "x": 15,
          "y": 69,
          "value": 10,
          "name": "cell16 ~ gene70"
        },
        {
          "x": 15,
          "y": 70,
          "value": 11,
          "name": "cell16 ~ gene71"
        },
        {
          "x": 15,
          "y": 71,
          "value": 2,
          "name": "cell16 ~ gene72"
        },
        {
          "x": 15,
          "y": 72,
          "value": 13,
          "name": "cell16 ~ gene73"
        },
        {
          "x": 15,
          "y": 73,
          "value": 1,
          "name": "cell16 ~ gene74"
        },
        {
          "x": 15,
          "y": 74,
          "value": 16,
          "name": "cell16 ~ gene75"
        },
        {
          "x": 15,
          "y": 75,
          "value": 0,
          "name": "cell16 ~ gene76"
        },
        {
          "x": 15,
          "y": 76,
          "value": 0,
          "name": "cell16 ~ gene77"
        },
        {
          "x": 15,
          "y": 77,
          "value": 8,
          "name": "cell16 ~ gene78"
        },
        {
          "x": 15,
          "y": 78,
          "value": 3,
          "name": "cell16 ~ gene79"
        },
        {
          "x": 15,
          "y": 79,
          "value": 0,
          "name": "cell16 ~ gene80"
        },
        {
          "x": 15,
          "y": 80,
          "value": 0,
          "name": "cell16 ~ gene81"
        },
        {
          "x": 15,
          "y": 81,
          "value": 0,
          "name": "cell16 ~ gene82"
        },
        {
          "x": 15,
          "y": 82,
          "value": 4,
          "name": "cell16 ~ gene83"
        },
        {
          "x": 15,
          "y": 83,
          "value": 0,
          "name": "cell16 ~ gene84"
        },
        {
          "x": 15,
          "y": 84,
          "value": 0,
          "name": "cell16 ~ gene85"
        },
        {
          "x": 15,
          "y": 85,
          "value": 10,
          "name": "cell16 ~ gene86"
        },
        {
          "x": 15,
          "y": 86,
          "value": 0,
          "name": "cell16 ~ gene87"
        },
        {
          "x": 15,
          "y": 87,
          "value": 0,
          "name": "cell16 ~ gene88"
        },
        {
          "x": 15,
          "y": 88,
          "value": 8,
          "name": "cell16 ~ gene89"
        },
        {
          "x": 15,
          "y": 89,
          "value": 0,
          "name": "cell16 ~ gene90"
        },
        {
          "x": 15,
          "y": 90,
          "value": 7,
          "name": "cell16 ~ gene91"
        },
        {
          "x": 15,
          "y": 91,
          "value": 7,
          "name": "cell16 ~ gene92"
        },
        {
          "x": 15,
          "y": 92,
          "value": 0,
          "name": "cell16 ~ gene93"
        },
        {
          "x": 15,
          "y": 93,
          "value": 0,
          "name": "cell16 ~ gene94"
        },
        {
          "x": 15,
          "y": 94,
          "value": 2,
          "name": "cell16 ~ gene95"
        },
        {
          "x": 15,
          "y": 95,
          "value": 0,
          "name": "cell16 ~ gene96"
        },
        {
          "x": 15,
          "y": 96,
          "value": 11,
          "name": "cell16 ~ gene97"
        },
        {
          "x": 15,
          "y": 97,
          "value": 13,
          "name": "cell16 ~ gene98"
        },
        {
          "x": 15,
          "y": 98,
          "value": 8,
          "name": "cell16 ~ gene99"
        },
        {
          "x": 15,
          "y": 99,
          "value": 5,
          "name": "cell16 ~ gene100"
        },
        {
          "x": 16,
          "y": 0,
          "value": 0,
          "name": "cell17 ~ gene1"
        },
        {
          "x": 16,
          "y": 1,
          "value": 28,
          "name": "cell17 ~ gene2"
        },
        {
          "x": 16,
          "y": 2,
          "value": 34,
          "name": "cell17 ~ gene3"
        },
        {
          "x": 16,
          "y": 3,
          "value": 21,
          "name": "cell17 ~ gene4"
        },
        {
          "x": 16,
          "y": 4,
          "value": 11,
          "name": "cell17 ~ gene5"
        },
        {
          "x": 16,
          "y": 5,
          "value": 16,
          "name": "cell17 ~ gene6"
        },
        {
          "x": 16,
          "y": 6,
          "value": 23,
          "name": "cell17 ~ gene7"
        },
        {
          "x": 16,
          "y": 7,
          "value": 7,
          "name": "cell17 ~ gene8"
        },
        {
          "x": 16,
          "y": 8,
          "value": 6,
          "name": "cell17 ~ gene9"
        },
        {
          "x": 16,
          "y": 9,
          "value": 0,
          "name": "cell17 ~ gene10"
        },
        {
          "x": 16,
          "y": 10,
          "value": 19,
          "name": "cell17 ~ gene11"
        },
        {
          "x": 16,
          "y": 11,
          "value": 11,
          "name": "cell17 ~ gene12"
        },
        {
          "x": 16,
          "y": 12,
          "value": 10,
          "name": "cell17 ~ gene13"
        },
        {
          "x": 16,
          "y": 13,
          "value": 25,
          "name": "cell17 ~ gene14"
        },
        {
          "x": 16,
          "y": 14,
          "value": 29,
          "name": "cell17 ~ gene15"
        },
        {
          "x": 16,
          "y": 15,
          "value": 47,
          "name": "cell17 ~ gene16"
        },
        {
          "x": 16,
          "y": 16,
          "value": 0,
          "name": "cell17 ~ gene17"
        },
        {
          "x": 16,
          "y": 17,
          "value": 32,
          "name": "cell17 ~ gene18"
        },
        {
          "x": 16,
          "y": 18,
          "value": 22,
          "name": "cell17 ~ gene19"
        },
        {
          "x": 16,
          "y": 19,
          "value": 5,
          "name": "cell17 ~ gene20"
        },
        {
          "x": 16,
          "y": 20,
          "value": 1,
          "name": "cell17 ~ gene21"
        },
        {
          "x": 16,
          "y": 21,
          "value": 4,
          "name": "cell17 ~ gene22"
        },
        {
          "x": 16,
          "y": 22,
          "value": 0,
          "name": "cell17 ~ gene23"
        },
        {
          "x": 16,
          "y": 23,
          "value": 0,
          "name": "cell17 ~ gene24"
        },
        {
          "x": 16,
          "y": 24,
          "value": 1,
          "name": "cell17 ~ gene25"
        },
        {
          "x": 16,
          "y": 25,
          "value": 8,
          "name": "cell17 ~ gene26"
        },
        {
          "x": 16,
          "y": 26,
          "value": 0,
          "name": "cell17 ~ gene27"
        },
        {
          "x": 16,
          "y": 27,
          "value": 0,
          "name": "cell17 ~ gene28"
        },
        {
          "x": 16,
          "y": 28,
          "value": 0,
          "name": "cell17 ~ gene29"
        },
        {
          "x": 16,
          "y": 29,
          "value": 2,
          "name": "cell17 ~ gene30"
        },
        {
          "x": 16,
          "y": 30,
          "value": 0,
          "name": "cell17 ~ gene31"
        },
        {
          "x": 16,
          "y": 31,
          "value": 0,
          "name": "cell17 ~ gene32"
        },
        {
          "x": 16,
          "y": 32,
          "value": 0,
          "name": "cell17 ~ gene33"
        },
        {
          "x": 16,
          "y": 33,
          "value": 0,
          "name": "cell17 ~ gene34"
        },
        {
          "x": 16,
          "y": 34,
          "value": 15,
          "name": "cell17 ~ gene35"
        },
        {
          "x": 16,
          "y": 35,
          "value": 16,
          "name": "cell17 ~ gene36"
        },
        {
          "x": 16,
          "y": 36,
          "value": 0,
          "name": "cell17 ~ gene37"
        },
        {
          "x": 16,
          "y": 37,
          "value": 0,
          "name": "cell17 ~ gene38"
        },
        {
          "x": 16,
          "y": 38,
          "value": 11,
          "name": "cell17 ~ gene39"
        },
        {
          "x": 16,
          "y": 39,
          "value": 0,
          "name": "cell17 ~ gene40"
        },
        {
          "x": 16,
          "y": 40,
          "value": 0,
          "name": "cell17 ~ gene41"
        },
        {
          "x": 16,
          "y": 41,
          "value": 0,
          "name": "cell17 ~ gene42"
        },
        {
          "x": 16,
          "y": 42,
          "value": 3,
          "name": "cell17 ~ gene43"
        },
        {
          "x": 16,
          "y": 43,
          "value": 2,
          "name": "cell17 ~ gene44"
        },
        {
          "x": 16,
          "y": 44,
          "value": 5,
          "name": "cell17 ~ gene45"
        },
        {
          "x": 16,
          "y": 45,
          "value": 20,
          "name": "cell17 ~ gene46"
        },
        {
          "x": 16,
          "y": 46,
          "value": 11,
          "name": "cell17 ~ gene47"
        },
        {
          "x": 16,
          "y": 47,
          "value": 0,
          "name": "cell17 ~ gene48"
        },
        {
          "x": 16,
          "y": 48,
          "value": 7,
          "name": "cell17 ~ gene49"
        },
        {
          "x": 16,
          "y": 49,
          "value": 11,
          "name": "cell17 ~ gene50"
        },
        {
          "x": 16,
          "y": 50,
          "value": 9,
          "name": "cell17 ~ gene51"
        },
        {
          "x": 16,
          "y": 51,
          "value": 0,
          "name": "cell17 ~ gene52"
        },
        {
          "x": 16,
          "y": 52,
          "value": 0,
          "name": "cell17 ~ gene53"
        },
        {
          "x": 16,
          "y": 53,
          "value": 21,
          "name": "cell17 ~ gene54"
        },
        {
          "x": 16,
          "y": 54,
          "value": 6,
          "name": "cell17 ~ gene55"
        },
        {
          "x": 16,
          "y": 55,
          "value": 18,
          "name": "cell17 ~ gene56"
        },
        {
          "x": 16,
          "y": 56,
          "value": 13,
          "name": "cell17 ~ gene57"
        },
        {
          "x": 16,
          "y": 57,
          "value": 3,
          "name": "cell17 ~ gene58"
        },
        {
          "x": 16,
          "y": 58,
          "value": 10,
          "name": "cell17 ~ gene59"
        },
        {
          "x": 16,
          "y": 59,
          "value": 7,
          "name": "cell17 ~ gene60"
        },
        {
          "x": 16,
          "y": 60,
          "value": 7,
          "name": "cell17 ~ gene61"
        },
        {
          "x": 16,
          "y": 61,
          "value": 9,
          "name": "cell17 ~ gene62"
        },
        {
          "x": 16,
          "y": 62,
          "value": 15,
          "name": "cell17 ~ gene63"
        },
        {
          "x": 16,
          "y": 63,
          "value": 0,
          "name": "cell17 ~ gene64"
        },
        {
          "x": 16,
          "y": 64,
          "value": 0,
          "name": "cell17 ~ gene65"
        },
        {
          "x": 16,
          "y": 65,
          "value": 0,
          "name": "cell17 ~ gene66"
        },
        {
          "x": 16,
          "y": 66,
          "value": 5,
          "name": "cell17 ~ gene67"
        },
        {
          "x": 16,
          "y": 67,
          "value": 0,
          "name": "cell17 ~ gene68"
        },
        {
          "x": 16,
          "y": 68,
          "value": 0,
          "name": "cell17 ~ gene69"
        },
        {
          "x": 16,
          "y": 69,
          "value": 3,
          "name": "cell17 ~ gene70"
        },
        {
          "x": 16,
          "y": 70,
          "value": 0,
          "name": "cell17 ~ gene71"
        },
        {
          "x": 16,
          "y": 71,
          "value": 0,
          "name": "cell17 ~ gene72"
        },
        {
          "x": 16,
          "y": 72,
          "value": 0,
          "name": "cell17 ~ gene73"
        },
        {
          "x": 16,
          "y": 73,
          "value": 0,
          "name": "cell17 ~ gene74"
        },
        {
          "x": 16,
          "y": 74,
          "value": 20,
          "name": "cell17 ~ gene75"
        },
        {
          "x": 16,
          "y": 75,
          "value": 0,
          "name": "cell17 ~ gene76"
        },
        {
          "x": 16,
          "y": 76,
          "value": 9,
          "name": "cell17 ~ gene77"
        },
        {
          "x": 16,
          "y": 77,
          "value": 0,
          "name": "cell17 ~ gene78"
        },
        {
          "x": 16,
          "y": 78,
          "value": 0,
          "name": "cell17 ~ gene79"
        },
        {
          "x": 16,
          "y": 79,
          "value": 12,
          "name": "cell17 ~ gene80"
        },
        {
          "x": 16,
          "y": 80,
          "value": 1,
          "name": "cell17 ~ gene81"
        },
        {
          "x": 16,
          "y": 81,
          "value": 7,
          "name": "cell17 ~ gene82"
        },
        {
          "x": 16,
          "y": 82,
          "value": 5,
          "name": "cell17 ~ gene83"
        },
        {
          "x": 16,
          "y": 83,
          "value": 4,
          "name": "cell17 ~ gene84"
        },
        {
          "x": 16,
          "y": 84,
          "value": 0,
          "name": "cell17 ~ gene85"
        },
        {
          "x": 16,
          "y": 85,
          "value": 3,
          "name": "cell17 ~ gene86"
        },
        {
          "x": 16,
          "y": 86,
          "value": 0,
          "name": "cell17 ~ gene87"
        },
        {
          "x": 16,
          "y": 87,
          "value": 0,
          "name": "cell17 ~ gene88"
        },
        {
          "x": 16,
          "y": 88,
          "value": 0,
          "name": "cell17 ~ gene89"
        },
        {
          "x": 16,
          "y": 89,
          "value": 2,
          "name": "cell17 ~ gene90"
        },
        {
          "x": 16,
          "y": 90,
          "value": 2,
          "name": "cell17 ~ gene91"
        },
        {
          "x": 16,
          "y": 91,
          "value": 0,
          "name": "cell17 ~ gene92"
        },
        {
          "x": 16,
          "y": 92,
          "value": 0,
          "name": "cell17 ~ gene93"
        },
        {
          "x": 16,
          "y": 93,
          "value": 11,
          "name": "cell17 ~ gene94"
        },
        {
          "x": 16,
          "y": 94,
          "value": 0,
          "name": "cell17 ~ gene95"
        },
        {
          "x": 16,
          "y": 95,
          "value": 0,
          "name": "cell17 ~ gene96"
        },
        {
          "x": 16,
          "y": 96,
          "value": 8,
          "name": "cell17 ~ gene97"
        },
        {
          "x": 16,
          "y": 97,
          "value": 0,
          "name": "cell17 ~ gene98"
        },
        {
          "x": 16,
          "y": 98,
          "value": 2,
          "name": "cell17 ~ gene99"
        },
        {
          "x": 16,
          "y": 99,
          "value": 1,
          "name": "cell17 ~ gene100"
        },
        {
          "x": 17,
          "y": 0,
          "value": 14,
          "name": "cell18 ~ gene1"
        },
        {
          "x": 17,
          "y": 1,
          "value": 0,
          "name": "cell18 ~ gene2"
        },
        {
          "x": 17,
          "y": 2,
          "value": 23,
          "name": "cell18 ~ gene3"
        },
        {
          "x": 17,
          "y": 3,
          "value": 28,
          "name": "cell18 ~ gene4"
        },
        {
          "x": 17,
          "y": 4,
          "value": 13,
          "name": "cell18 ~ gene5"
        },
        {
          "x": 17,
          "y": 5,
          "value": 32,
          "name": "cell18 ~ gene6"
        },
        {
          "x": 17,
          "y": 6,
          "value": 28,
          "name": "cell18 ~ gene7"
        },
        {
          "x": 17,
          "y": 7,
          "value": 29,
          "name": "cell18 ~ gene8"
        },
        {
          "x": 17,
          "y": 8,
          "value": 1,
          "name": "cell18 ~ gene9"
        },
        {
          "x": 17,
          "y": 9,
          "value": 3,
          "name": "cell18 ~ gene10"
        },
        {
          "x": 17,
          "y": 10,
          "value": 6,
          "name": "cell18 ~ gene11"
        },
        {
          "x": 17,
          "y": 11,
          "value": 21,
          "name": "cell18 ~ gene12"
        },
        {
          "x": 17,
          "y": 12,
          "value": 15,
          "name": "cell18 ~ gene13"
        },
        {
          "x": 17,
          "y": 13,
          "value": 23,
          "name": "cell18 ~ gene14"
        },
        {
          "x": 17,
          "y": 14,
          "value": 28,
          "name": "cell18 ~ gene15"
        },
        {
          "x": 17,
          "y": 15,
          "value": 33,
          "name": "cell18 ~ gene16"
        },
        {
          "x": 17,
          "y": 16,
          "value": 29,
          "name": "cell18 ~ gene17"
        },
        {
          "x": 17,
          "y": 17,
          "value": 24,
          "name": "cell18 ~ gene18"
        },
        {
          "x": 17,
          "y": 18,
          "value": 10,
          "name": "cell18 ~ gene19"
        },
        {
          "x": 17,
          "y": 19,
          "value": 0,
          "name": "cell18 ~ gene20"
        },
        {
          "x": 17,
          "y": 20,
          "value": 0,
          "name": "cell18 ~ gene21"
        },
        {
          "x": 17,
          "y": 21,
          "value": 19,
          "name": "cell18 ~ gene22"
        },
        {
          "x": 17,
          "y": 22,
          "value": 0,
          "name": "cell18 ~ gene23"
        },
        {
          "x": 17,
          "y": 23,
          "value": 0,
          "name": "cell18 ~ gene24"
        },
        {
          "x": 17,
          "y": 24,
          "value": 22,
          "name": "cell18 ~ gene25"
        },
        {
          "x": 17,
          "y": 25,
          "value": 4,
          "name": "cell18 ~ gene26"
        },
        {
          "x": 17,
          "y": 26,
          "value": 12,
          "name": "cell18 ~ gene27"
        },
        {
          "x": 17,
          "y": 27,
          "value": 17,
          "name": "cell18 ~ gene28"
        },
        {
          "x": 17,
          "y": 28,
          "value": 9,
          "name": "cell18 ~ gene29"
        },
        {
          "x": 17,
          "y": 29,
          "value": 6,
          "name": "cell18 ~ gene30"
        },
        {
          "x": 17,
          "y": 30,
          "value": 0,
          "name": "cell18 ~ gene31"
        },
        {
          "x": 17,
          "y": 31,
          "value": 0,
          "name": "cell18 ~ gene32"
        },
        {
          "x": 17,
          "y": 32,
          "value": 0,
          "name": "cell18 ~ gene33"
        },
        {
          "x": 17,
          "y": 33,
          "value": 0,
          "name": "cell18 ~ gene34"
        },
        {
          "x": 17,
          "y": 34,
          "value": 0,
          "name": "cell18 ~ gene35"
        },
        {
          "x": 17,
          "y": 35,
          "value": 13,
          "name": "cell18 ~ gene36"
        },
        {
          "x": 17,
          "y": 36,
          "value": 1,
          "name": "cell18 ~ gene37"
        },
        {
          "x": 17,
          "y": 37,
          "value": 0,
          "name": "cell18 ~ gene38"
        },
        {
          "x": 17,
          "y": 38,
          "value": 0,
          "name": "cell18 ~ gene39"
        },
        {
          "x": 17,
          "y": 39,
          "value": 0,
          "name": "cell18 ~ gene40"
        },
        {
          "x": 17,
          "y": 40,
          "value": 15,
          "name": "cell18 ~ gene41"
        },
        {
          "x": 17,
          "y": 41,
          "value": 0,
          "name": "cell18 ~ gene42"
        },
        {
          "x": 17,
          "y": 42,
          "value": 10,
          "name": "cell18 ~ gene43"
        },
        {
          "x": 17,
          "y": 43,
          "value": 12,
          "name": "cell18 ~ gene44"
        },
        {
          "x": 17,
          "y": 44,
          "value": 13,
          "name": "cell18 ~ gene45"
        },
        {
          "x": 17,
          "y": 45,
          "value": 2,
          "name": "cell18 ~ gene46"
        },
        {
          "x": 17,
          "y": 46,
          "value": 0,
          "name": "cell18 ~ gene47"
        },
        {
          "x": 17,
          "y": 47,
          "value": 10,
          "name": "cell18 ~ gene48"
        },
        {
          "x": 17,
          "y": 48,
          "value": 2,
          "name": "cell18 ~ gene49"
        },
        {
          "x": 17,
          "y": 49,
          "value": 0,
          "name": "cell18 ~ gene50"
        },
        {
          "x": 17,
          "y": 50,
          "value": 2,
          "name": "cell18 ~ gene51"
        },
        {
          "x": 17,
          "y": 51,
          "value": 8,
          "name": "cell18 ~ gene52"
        },
        {
          "x": 17,
          "y": 52,
          "value": 1,
          "name": "cell18 ~ gene53"
        },
        {
          "x": 17,
          "y": 53,
          "value": 8,
          "name": "cell18 ~ gene54"
        },
        {
          "x": 17,
          "y": 54,
          "value": 0,
          "name": "cell18 ~ gene55"
        },
        {
          "x": 17,
          "y": 55,
          "value": 7,
          "name": "cell18 ~ gene56"
        },
        {
          "x": 17,
          "y": 56,
          "value": 0,
          "name": "cell18 ~ gene57"
        },
        {
          "x": 17,
          "y": 57,
          "value": 0,
          "name": "cell18 ~ gene58"
        },
        {
          "x": 17,
          "y": 58,
          "value": 0,
          "name": "cell18 ~ gene59"
        },
        {
          "x": 17,
          "y": 59,
          "value": 14,
          "name": "cell18 ~ gene60"
        },
        {
          "x": 17,
          "y": 60,
          "value": 0,
          "name": "cell18 ~ gene61"
        },
        {
          "x": 17,
          "y": 61,
          "value": 0,
          "name": "cell18 ~ gene62"
        },
        {
          "x": 17,
          "y": 62,
          "value": 8,
          "name": "cell18 ~ gene63"
        },
        {
          "x": 17,
          "y": 63,
          "value": 0,
          "name": "cell18 ~ gene64"
        },
        {
          "x": 17,
          "y": 64,
          "value": 25,
          "name": "cell18 ~ gene65"
        },
        {
          "x": 17,
          "y": 65,
          "value": 6,
          "name": "cell18 ~ gene66"
        },
        {
          "x": 17,
          "y": 66,
          "value": 17,
          "name": "cell18 ~ gene67"
        },
        {
          "x": 17,
          "y": 67,
          "value": 5,
          "name": "cell18 ~ gene68"
        },
        {
          "x": 17,
          "y": 68,
          "value": 0,
          "name": "cell18 ~ gene69"
        },
        {
          "x": 17,
          "y": 69,
          "value": 13,
          "name": "cell18 ~ gene70"
        },
        {
          "x": 17,
          "y": 70,
          "value": 6,
          "name": "cell18 ~ gene71"
        },
        {
          "x": 17,
          "y": 71,
          "value": 0,
          "name": "cell18 ~ gene72"
        },
        {
          "x": 17,
          "y": 72,
          "value": 6,
          "name": "cell18 ~ gene73"
        },
        {
          "x": 17,
          "y": 73,
          "value": 10,
          "name": "cell18 ~ gene74"
        },
        {
          "x": 17,
          "y": 74,
          "value": 0,
          "name": "cell18 ~ gene75"
        },
        {
          "x": 17,
          "y": 75,
          "value": 0,
          "name": "cell18 ~ gene76"
        },
        {
          "x": 17,
          "y": 76,
          "value": 8,
          "name": "cell18 ~ gene77"
        },
        {
          "x": 17,
          "y": 77,
          "value": 0,
          "name": "cell18 ~ gene78"
        },
        {
          "x": 17,
          "y": 78,
          "value": 8,
          "name": "cell18 ~ gene79"
        },
        {
          "x": 17,
          "y": 79,
          "value": 10,
          "name": "cell18 ~ gene80"
        },
        {
          "x": 17,
          "y": 80,
          "value": 8,
          "name": "cell18 ~ gene81"
        },
        {
          "x": 17,
          "y": 81,
          "value": 2,
          "name": "cell18 ~ gene82"
        },
        {
          "x": 17,
          "y": 82,
          "value": 0,
          "name": "cell18 ~ gene83"
        },
        {
          "x": 17,
          "y": 83,
          "value": 4,
          "name": "cell18 ~ gene84"
        },
        {
          "x": 17,
          "y": 84,
          "value": 1,
          "name": "cell18 ~ gene85"
        },
        {
          "x": 17,
          "y": 85,
          "value": 0,
          "name": "cell18 ~ gene86"
        },
        {
          "x": 17,
          "y": 86,
          "value": 0,
          "name": "cell18 ~ gene87"
        },
        {
          "x": 17,
          "y": 87,
          "value": 0,
          "name": "cell18 ~ gene88"
        },
        {
          "x": 17,
          "y": 88,
          "value": 2,
          "name": "cell18 ~ gene89"
        },
        {
          "x": 17,
          "y": 89,
          "value": 0,
          "name": "cell18 ~ gene90"
        },
        {
          "x": 17,
          "y": 90,
          "value": 0,
          "name": "cell18 ~ gene91"
        },
        {
          "x": 17,
          "y": 91,
          "value": 5,
          "name": "cell18 ~ gene92"
        },
        {
          "x": 17,
          "y": 92,
          "value": 0,
          "name": "cell18 ~ gene93"
        },
        {
          "x": 17,
          "y": 93,
          "value": 0,
          "name": "cell18 ~ gene94"
        },
        {
          "x": 17,
          "y": 94,
          "value": 0,
          "name": "cell18 ~ gene95"
        },
        {
          "x": 17,
          "y": 95,
          "value": 11,
          "name": "cell18 ~ gene96"
        },
        {
          "x": 17,
          "y": 96,
          "value": 21,
          "name": "cell18 ~ gene97"
        },
        {
          "x": 17,
          "y": 97,
          "value": 0,
          "name": "cell18 ~ gene98"
        },
        {
          "x": 17,
          "y": 98,
          "value": 0,
          "name": "cell18 ~ gene99"
        },
        {
          "x": 17,
          "y": 99,
          "value": 1,
          "name": "cell18 ~ gene100"
        },
        {
          "x": 18,
          "y": 0,
          "value": 4,
          "name": "cell19 ~ gene1"
        },
        {
          "x": 18,
          "y": 1,
          "value": 2,
          "name": "cell19 ~ gene2"
        },
        {
          "x": 18,
          "y": 2,
          "value": 13,
          "name": "cell19 ~ gene3"
        },
        {
          "x": 18,
          "y": 3,
          "value": 28,
          "name": "cell19 ~ gene4"
        },
        {
          "x": 18,
          "y": 4,
          "value": 5,
          "name": "cell19 ~ gene5"
        },
        {
          "x": 18,
          "y": 5,
          "value": 0,
          "name": "cell19 ~ gene6"
        },
        {
          "x": 18,
          "y": 6,
          "value": 16,
          "name": "cell19 ~ gene7"
        },
        {
          "x": 18,
          "y": 7,
          "value": 19,
          "name": "cell19 ~ gene8"
        },
        {
          "x": 18,
          "y": 8,
          "value": 7,
          "name": "cell19 ~ gene9"
        },
        {
          "x": 18,
          "y": 9,
          "value": 0,
          "name": "cell19 ~ gene10"
        },
        {
          "x": 18,
          "y": 10,
          "value": 9,
          "name": "cell19 ~ gene11"
        },
        {
          "x": 18,
          "y": 11,
          "value": 18,
          "name": "cell19 ~ gene12"
        },
        {
          "x": 18,
          "y": 12,
          "value": 8,
          "name": "cell19 ~ gene13"
        },
        {
          "x": 18,
          "y": 13,
          "value": 23,
          "name": "cell19 ~ gene14"
        },
        {
          "x": 18,
          "y": 14,
          "value": 41,
          "name": "cell19 ~ gene15"
        },
        {
          "x": 18,
          "y": 15,
          "value": 20,
          "name": "cell19 ~ gene16"
        },
        {
          "x": 18,
          "y": 16,
          "value": 29,
          "name": "cell19 ~ gene17"
        },
        {
          "x": 18,
          "y": 17,
          "value": 19,
          "name": "cell19 ~ gene18"
        },
        {
          "x": 18,
          "y": 18,
          "value": 38,
          "name": "cell19 ~ gene19"
        },
        {
          "x": 18,
          "y": 19,
          "value": 3,
          "name": "cell19 ~ gene20"
        },
        {
          "x": 18,
          "y": 20,
          "value": 0,
          "name": "cell19 ~ gene21"
        },
        {
          "x": 18,
          "y": 21,
          "value": 0,
          "name": "cell19 ~ gene22"
        },
        {
          "x": 18,
          "y": 22,
          "value": 0,
          "name": "cell19 ~ gene23"
        },
        {
          "x": 18,
          "y": 23,
          "value": 6,
          "name": "cell19 ~ gene24"
        },
        {
          "x": 18,
          "y": 24,
          "value": 0,
          "name": "cell19 ~ gene25"
        },
        {
          "x": 18,
          "y": 25,
          "value": 0,
          "name": "cell19 ~ gene26"
        },
        {
          "x": 18,
          "y": 26,
          "value": 10,
          "name": "cell19 ~ gene27"
        },
        {
          "x": 18,
          "y": 27,
          "value": 2,
          "name": "cell19 ~ gene28"
        },
        {
          "x": 18,
          "y": 28,
          "value": 0,
          "name": "cell19 ~ gene29"
        },
        {
          "x": 18,
          "y": 29,
          "value": 10,
          "name": "cell19 ~ gene30"
        },
        {
          "x": 18,
          "y": 30,
          "value": 15,
          "name": "cell19 ~ gene31"
        },
        {
          "x": 18,
          "y": 31,
          "value": 0,
          "name": "cell19 ~ gene32"
        },
        {
          "x": 18,
          "y": 32,
          "value": 0,
          "name": "cell19 ~ gene33"
        },
        {
          "x": 18,
          "y": 33,
          "value": 0,
          "name": "cell19 ~ gene34"
        },
        {
          "x": 18,
          "y": 34,
          "value": 6,
          "name": "cell19 ~ gene35"
        },
        {
          "x": 18,
          "y": 35,
          "value": 0,
          "name": "cell19 ~ gene36"
        },
        {
          "x": 18,
          "y": 36,
          "value": 7,
          "name": "cell19 ~ gene37"
        },
        {
          "x": 18,
          "y": 37,
          "value": 20,
          "name": "cell19 ~ gene38"
        },
        {
          "x": 18,
          "y": 38,
          "value": 12,
          "name": "cell19 ~ gene39"
        },
        {
          "x": 18,
          "y": 39,
          "value": 0,
          "name": "cell19 ~ gene40"
        },
        {
          "x": 18,
          "y": 40,
          "value": 0,
          "name": "cell19 ~ gene41"
        },
        {
          "x": 18,
          "y": 41,
          "value": 18,
          "name": "cell19 ~ gene42"
        },
        {
          "x": 18,
          "y": 42,
          "value": 17,
          "name": "cell19 ~ gene43"
        },
        {
          "x": 18,
          "y": 43,
          "value": 0,
          "name": "cell19 ~ gene44"
        },
        {
          "x": 18,
          "y": 44,
          "value": 15,
          "name": "cell19 ~ gene45"
        },
        {
          "x": 18,
          "y": 45,
          "value": 10,
          "name": "cell19 ~ gene46"
        },
        {
          "x": 18,
          "y": 46,
          "value": 8,
          "name": "cell19 ~ gene47"
        },
        {
          "x": 18,
          "y": 47,
          "value": 0,
          "name": "cell19 ~ gene48"
        },
        {
          "x": 18,
          "y": 48,
          "value": 1,
          "name": "cell19 ~ gene49"
        },
        {
          "x": 18,
          "y": 49,
          "value": 0,
          "name": "cell19 ~ gene50"
        },
        {
          "x": 18,
          "y": 50,
          "value": 9,
          "name": "cell19 ~ gene51"
        },
        {
          "x": 18,
          "y": 51,
          "value": 0,
          "name": "cell19 ~ gene52"
        },
        {
          "x": 18,
          "y": 52,
          "value": 11,
          "name": "cell19 ~ gene53"
        },
        {
          "x": 18,
          "y": 53,
          "value": 0,
          "name": "cell19 ~ gene54"
        },
        {
          "x": 18,
          "y": 54,
          "value": 0,
          "name": "cell19 ~ gene55"
        },
        {
          "x": 18,
          "y": 55,
          "value": 11,
          "name": "cell19 ~ gene56"
        },
        {
          "x": 18,
          "y": 56,
          "value": 3,
          "name": "cell19 ~ gene57"
        },
        {
          "x": 18,
          "y": 57,
          "value": 0,
          "name": "cell19 ~ gene58"
        },
        {
          "x": 18,
          "y": 58,
          "value": 0,
          "name": "cell19 ~ gene59"
        },
        {
          "x": 18,
          "y": 59,
          "value": 0,
          "name": "cell19 ~ gene60"
        },
        {
          "x": 18,
          "y": 60,
          "value": 0,
          "name": "cell19 ~ gene61"
        },
        {
          "x": 18,
          "y": 61,
          "value": 12,
          "name": "cell19 ~ gene62"
        },
        {
          "x": 18,
          "y": 62,
          "value": 7,
          "name": "cell19 ~ gene63"
        },
        {
          "x": 18,
          "y": 63,
          "value": 0,
          "name": "cell19 ~ gene64"
        },
        {
          "x": 18,
          "y": 64,
          "value": 8,
          "name": "cell19 ~ gene65"
        },
        {
          "x": 18,
          "y": 65,
          "value": 0,
          "name": "cell19 ~ gene66"
        },
        {
          "x": 18,
          "y": 66,
          "value": 7,
          "name": "cell19 ~ gene67"
        },
        {
          "x": 18,
          "y": 67,
          "value": 14,
          "name": "cell19 ~ gene68"
        },
        {
          "x": 18,
          "y": 68,
          "value": 10,
          "name": "cell19 ~ gene69"
        },
        {
          "x": 18,
          "y": 69,
          "value": 0,
          "name": "cell19 ~ gene70"
        },
        {
          "x": 18,
          "y": 70,
          "value": 0,
          "name": "cell19 ~ gene71"
        },
        {
          "x": 18,
          "y": 71,
          "value": 0,
          "name": "cell19 ~ gene72"
        },
        {
          "x": 18,
          "y": 72,
          "value": 0,
          "name": "cell19 ~ gene73"
        },
        {
          "x": 18,
          "y": 73,
          "value": 13,
          "name": "cell19 ~ gene74"
        },
        {
          "x": 18,
          "y": 74,
          "value": 0,
          "name": "cell19 ~ gene75"
        },
        {
          "x": 18,
          "y": 75,
          "value": 0,
          "name": "cell19 ~ gene76"
        },
        {
          "x": 18,
          "y": 76,
          "value": 0,
          "name": "cell19 ~ gene77"
        },
        {
          "x": 18,
          "y": 77,
          "value": 0,
          "name": "cell19 ~ gene78"
        },
        {
          "x": 18,
          "y": 78,
          "value": 0,
          "name": "cell19 ~ gene79"
        },
        {
          "x": 18,
          "y": 79,
          "value": 12,
          "name": "cell19 ~ gene80"
        },
        {
          "x": 18,
          "y": 80,
          "value": 0,
          "name": "cell19 ~ gene81"
        },
        {
          "x": 18,
          "y": 81,
          "value": 20,
          "name": "cell19 ~ gene82"
        },
        {
          "x": 18,
          "y": 82,
          "value": 9,
          "name": "cell19 ~ gene83"
        },
        {
          "x": 18,
          "y": 83,
          "value": 11,
          "name": "cell19 ~ gene84"
        },
        {
          "x": 18,
          "y": 84,
          "value": 0,
          "name": "cell19 ~ gene85"
        },
        {
          "x": 18,
          "y": 85,
          "value": 0,
          "name": "cell19 ~ gene86"
        },
        {
          "x": 18,
          "y": 86,
          "value": 1,
          "name": "cell19 ~ gene87"
        },
        {
          "x": 18,
          "y": 87,
          "value": 0,
          "name": "cell19 ~ gene88"
        },
        {
          "x": 18,
          "y": 88,
          "value": 0,
          "name": "cell19 ~ gene89"
        },
        {
          "x": 18,
          "y": 89,
          "value": 14,
          "name": "cell19 ~ gene90"
        },
        {
          "x": 18,
          "y": 90,
          "value": 0,
          "name": "cell19 ~ gene91"
        },
        {
          "x": 18,
          "y": 91,
          "value": 0,
          "name": "cell19 ~ gene92"
        },
        {
          "x": 18,
          "y": 92,
          "value": 6,
          "name": "cell19 ~ gene93"
        },
        {
          "x": 18,
          "y": 93,
          "value": 0,
          "name": "cell19 ~ gene94"
        },
        {
          "x": 18,
          "y": 94,
          "value": 0,
          "name": "cell19 ~ gene95"
        },
        {
          "x": 18,
          "y": 95,
          "value": 17,
          "name": "cell19 ~ gene96"
        },
        {
          "x": 18,
          "y": 96,
          "value": 3,
          "name": "cell19 ~ gene97"
        },
        {
          "x": 18,
          "y": 97,
          "value": 0,
          "name": "cell19 ~ gene98"
        },
        {
          "x": 18,
          "y": 98,
          "value": 0,
          "name": "cell19 ~ gene99"
        },
        {
          "x": 18,
          "y": 99,
          "value": 16,
          "name": "cell19 ~ gene100"
        },
        {
          "x": 19,
          "y": 0,
          "value": 26,
          "name": "cell20 ~ gene1"
        },
        {
          "x": 19,
          "y": 1,
          "value": 6,
          "name": "cell20 ~ gene2"
        },
        {
          "x": 19,
          "y": 2,
          "value": 19,
          "name": "cell20 ~ gene3"
        },
        {
          "x": 19,
          "y": 3,
          "value": 0,
          "name": "cell20 ~ gene4"
        },
        {
          "x": 19,
          "y": 4,
          "value": 34,
          "name": "cell20 ~ gene5"
        },
        {
          "x": 19,
          "y": 5,
          "value": 4,
          "name": "cell20 ~ gene6"
        },
        {
          "x": 19,
          "y": 6,
          "value": 31,
          "name": "cell20 ~ gene7"
        },
        {
          "x": 19,
          "y": 7,
          "value": 0,
          "name": "cell20 ~ gene8"
        },
        {
          "x": 19,
          "y": 8,
          "value": 10,
          "name": "cell20 ~ gene9"
        },
        {
          "x": 19,
          "y": 9,
          "value": 0,
          "name": "cell20 ~ gene10"
        },
        {
          "x": 19,
          "y": 10,
          "value": 37,
          "name": "cell20 ~ gene11"
        },
        {
          "x": 19,
          "y": 11,
          "value": 0,
          "name": "cell20 ~ gene12"
        },
        {
          "x": 19,
          "y": 12,
          "value": 24,
          "name": "cell20 ~ gene13"
        },
        {
          "x": 19,
          "y": 13,
          "value": 0,
          "name": "cell20 ~ gene14"
        },
        {
          "x": 19,
          "y": 14,
          "value": 27,
          "name": "cell20 ~ gene15"
        },
        {
          "x": 19,
          "y": 15,
          "value": 53,
          "name": "cell20 ~ gene16"
        },
        {
          "x": 19,
          "y": 16,
          "value": 24,
          "name": "cell20 ~ gene17"
        },
        {
          "x": 19,
          "y": 17,
          "value": 25,
          "name": "cell20 ~ gene18"
        },
        {
          "x": 19,
          "y": 18,
          "value": 18,
          "name": "cell20 ~ gene19"
        },
        {
          "x": 19,
          "y": 19,
          "value": 2,
          "name": "cell20 ~ gene20"
        },
        {
          "x": 19,
          "y": 20,
          "value": 30,
          "name": "cell20 ~ gene21"
        },
        {
          "x": 19,
          "y": 21,
          "value": 0,
          "name": "cell20 ~ gene22"
        },
        {
          "x": 19,
          "y": 22,
          "value": 0,
          "name": "cell20 ~ gene23"
        },
        {
          "x": 19,
          "y": 23,
          "value": 0,
          "name": "cell20 ~ gene24"
        },
        {
          "x": 19,
          "y": 24,
          "value": 0,
          "name": "cell20 ~ gene25"
        },
        {
          "x": 19,
          "y": 25,
          "value": 4,
          "name": "cell20 ~ gene26"
        },
        {
          "x": 19,
          "y": 26,
          "value": 2,
          "name": "cell20 ~ gene27"
        },
        {
          "x": 19,
          "y": 27,
          "value": 12,
          "name": "cell20 ~ gene28"
        },
        {
          "x": 19,
          "y": 28,
          "value": 4,
          "name": "cell20 ~ gene29"
        },
        {
          "x": 19,
          "y": 29,
          "value": 0,
          "name": "cell20 ~ gene30"
        },
        {
          "x": 19,
          "y": 30,
          "value": 0,
          "name": "cell20 ~ gene31"
        },
        {
          "x": 19,
          "y": 31,
          "value": 1,
          "name": "cell20 ~ gene32"
        },
        {
          "x": 19,
          "y": 32,
          "value": 23,
          "name": "cell20 ~ gene33"
        },
        {
          "x": 19,
          "y": 33,
          "value": 0,
          "name": "cell20 ~ gene34"
        },
        {
          "x": 19,
          "y": 34,
          "value": 0,
          "name": "cell20 ~ gene35"
        },
        {
          "x": 19,
          "y": 35,
          "value": 8,
          "name": "cell20 ~ gene36"
        },
        {
          "x": 19,
          "y": 36,
          "value": 0,
          "name": "cell20 ~ gene37"
        },
        {
          "x": 19,
          "y": 37,
          "value": 0,
          "name": "cell20 ~ gene38"
        },
        {
          "x": 19,
          "y": 38,
          "value": 0,
          "name": "cell20 ~ gene39"
        },
        {
          "x": 19,
          "y": 39,
          "value": 0,
          "name": "cell20 ~ gene40"
        },
        {
          "x": 19,
          "y": 40,
          "value": 0,
          "name": "cell20 ~ gene41"
        },
        {
          "x": 19,
          "y": 41,
          "value": 0,
          "name": "cell20 ~ gene42"
        },
        {
          "x": 19,
          "y": 42,
          "value": 0,
          "name": "cell20 ~ gene43"
        },
        {
          "x": 19,
          "y": 43,
          "value": 2,
          "name": "cell20 ~ gene44"
        },
        {
          "x": 19,
          "y": 44,
          "value": 8,
          "name": "cell20 ~ gene45"
        },
        {
          "x": 19,
          "y": 45,
          "value": 0,
          "name": "cell20 ~ gene46"
        },
        {
          "x": 19,
          "y": 46,
          "value": 0,
          "name": "cell20 ~ gene47"
        },
        {
          "x": 19,
          "y": 47,
          "value": 0,
          "name": "cell20 ~ gene48"
        },
        {
          "x": 19,
          "y": 48,
          "value": 0,
          "name": "cell20 ~ gene49"
        },
        {
          "x": 19,
          "y": 49,
          "value": 0,
          "name": "cell20 ~ gene50"
        },
        {
          "x": 19,
          "y": 50,
          "value": 0,
          "name": "cell20 ~ gene51"
        },
        {
          "x": 19,
          "y": 51,
          "value": 8,
          "name": "cell20 ~ gene52"
        },
        {
          "x": 19,
          "y": 52,
          "value": 0,
          "name": "cell20 ~ gene53"
        },
        {
          "x": 19,
          "y": 53,
          "value": 9,
          "name": "cell20 ~ gene54"
        },
        {
          "x": 19,
          "y": 54,
          "value": 2,
          "name": "cell20 ~ gene55"
        },
        {
          "x": 19,
          "y": 55,
          "value": 0,
          "name": "cell20 ~ gene56"
        },
        {
          "x": 19,
          "y": 56,
          "value": 0,
          "name": "cell20 ~ gene57"
        },
        {
          "x": 19,
          "y": 57,
          "value": 3,
          "name": "cell20 ~ gene58"
        },
        {
          "x": 19,
          "y": 58,
          "value": 0,
          "name": "cell20 ~ gene59"
        },
        {
          "x": 19,
          "y": 59,
          "value": 12,
          "name": "cell20 ~ gene60"
        },
        {
          "x": 19,
          "y": 60,
          "value": 0,
          "name": "cell20 ~ gene61"
        },
        {
          "x": 19,
          "y": 61,
          "value": 0,
          "name": "cell20 ~ gene62"
        },
        {
          "x": 19,
          "y": 62,
          "value": 0,
          "name": "cell20 ~ gene63"
        },
        {
          "x": 19,
          "y": 63,
          "value": 2,
          "name": "cell20 ~ gene64"
        },
        {
          "x": 19,
          "y": 64,
          "value": 2,
          "name": "cell20 ~ gene65"
        },
        {
          "x": 19,
          "y": 65,
          "value": 0,
          "name": "cell20 ~ gene66"
        },
        {
          "x": 19,
          "y": 66,
          "value": 16,
          "name": "cell20 ~ gene67"
        },
        {
          "x": 19,
          "y": 67,
          "value": 0,
          "name": "cell20 ~ gene68"
        },
        {
          "x": 19,
          "y": 68,
          "value": 0,
          "name": "cell20 ~ gene69"
        },
        {
          "x": 19,
          "y": 69,
          "value": 11,
          "name": "cell20 ~ gene70"
        },
        {
          "x": 19,
          "y": 70,
          "value": 0,
          "name": "cell20 ~ gene71"
        },
        {
          "x": 19,
          "y": 71,
          "value": 0,
          "name": "cell20 ~ gene72"
        },
        {
          "x": 19,
          "y": 72,
          "value": 0,
          "name": "cell20 ~ gene73"
        },
        {
          "x": 19,
          "y": 73,
          "value": 0,
          "name": "cell20 ~ gene74"
        },
        {
          "x": 19,
          "y": 74,
          "value": 10,
          "name": "cell20 ~ gene75"
        },
        {
          "x": 19,
          "y": 75,
          "value": 8,
          "name": "cell20 ~ gene76"
        },
        {
          "x": 19,
          "y": 76,
          "value": 0,
          "name": "cell20 ~ gene77"
        },
        {
          "x": 19,
          "y": 77,
          "value": 0,
          "name": "cell20 ~ gene78"
        },
        {
          "x": 19,
          "y": 78,
          "value": 0,
          "name": "cell20 ~ gene79"
        },
        {
          "x": 19,
          "y": 79,
          "value": 14,
          "name": "cell20 ~ gene80"
        },
        {
          "x": 19,
          "y": 80,
          "value": 8,
          "name": "cell20 ~ gene81"
        },
        {
          "x": 19,
          "y": 81,
          "value": 0,
          "name": "cell20 ~ gene82"
        },
        {
          "x": 19,
          "y": 82,
          "value": 0,
          "name": "cell20 ~ gene83"
        },
        {
          "x": 19,
          "y": 83,
          "value": 25,
          "name": "cell20 ~ gene84"
        },
        {
          "x": 19,
          "y": 84,
          "value": 0,
          "name": "cell20 ~ gene85"
        },
        {
          "x": 19,
          "y": 85,
          "value": 0,
          "name": "cell20 ~ gene86"
        },
        {
          "x": 19,
          "y": 86,
          "value": 0,
          "name": "cell20 ~ gene87"
        },
        {
          "x": 19,
          "y": 87,
          "value": 0,
          "name": "cell20 ~ gene88"
        },
        {
          "x": 19,
          "y": 88,
          "value": 0,
          "name": "cell20 ~ gene89"
        },
        {
          "x": 19,
          "y": 89,
          "value": 9,
          "name": "cell20 ~ gene90"
        },
        {
          "x": 19,
          "y": 90,
          "value": 0,
          "name": "cell20 ~ gene91"
        },
        {
          "x": 19,
          "y": 91,
          "value": 0,
          "name": "cell20 ~ gene92"
        },
        {
          "x": 19,
          "y": 92,
          "value": 10,
          "name": "cell20 ~ gene93"
        },
        {
          "x": 19,
          "y": 93,
          "value": 0,
          "name": "cell20 ~ gene94"
        },
        {
          "x": 19,
          "y": 94,
          "value": 0,
          "name": "cell20 ~ gene95"
        },
        {
          "x": 19,
          "y": 95,
          "value": 0,
          "name": "cell20 ~ gene96"
        },
        {
          "x": 19,
          "y": 96,
          "value": 0,
          "name": "cell20 ~ gene97"
        },
        {
          "x": 19,
          "y": 97,
          "value": 0,
          "name": "cell20 ~ gene98"
        },
        {
          "x": 19,
          "y": 98,
          "value": 2,
          "name": "cell20 ~ gene99"
        },
        {
          "x": 19,
          "y": 99,
          "value": 0,
          "name": "cell20 ~ gene100"
        },
        {
          "x": 20,
          "y": 0,
          "value": 10,
          "name": "cell21 ~ gene1"
        },
        {
          "x": 20,
          "y": 1,
          "value": 0,
          "name": "cell21 ~ gene2"
        },
        {
          "x": 20,
          "y": 2,
          "value": 18,
          "name": "cell21 ~ gene3"
        },
        {
          "x": 20,
          "y": 3,
          "value": 18,
          "name": "cell21 ~ gene4"
        },
        {
          "x": 20,
          "y": 4,
          "value": 24,
          "name": "cell21 ~ gene5"
        },
        {
          "x": 20,
          "y": 5,
          "value": 6,
          "name": "cell21 ~ gene6"
        },
        {
          "x": 20,
          "y": 6,
          "value": 19,
          "name": "cell21 ~ gene7"
        },
        {
          "x": 20,
          "y": 7,
          "value": 14,
          "name": "cell21 ~ gene8"
        },
        {
          "x": 20,
          "y": 8,
          "value": 6,
          "name": "cell21 ~ gene9"
        },
        {
          "x": 20,
          "y": 9,
          "value": 0,
          "name": "cell21 ~ gene10"
        },
        {
          "x": 20,
          "y": 10,
          "value": 6,
          "name": "cell21 ~ gene11"
        },
        {
          "x": 20,
          "y": 11,
          "value": 0,
          "name": "cell21 ~ gene12"
        },
        {
          "x": 20,
          "y": 12,
          "value": 19,
          "name": "cell21 ~ gene13"
        },
        {
          "x": 20,
          "y": 13,
          "value": 18,
          "name": "cell21 ~ gene14"
        },
        {
          "x": 20,
          "y": 14,
          "value": 36,
          "name": "cell21 ~ gene15"
        },
        {
          "x": 20,
          "y": 15,
          "value": 41,
          "name": "cell21 ~ gene16"
        },
        {
          "x": 20,
          "y": 16,
          "value": 34,
          "name": "cell21 ~ gene17"
        },
        {
          "x": 20,
          "y": 17,
          "value": 29,
          "name": "cell21 ~ gene18"
        },
        {
          "x": 20,
          "y": 18,
          "value": 0,
          "name": "cell21 ~ gene19"
        },
        {
          "x": 20,
          "y": 19,
          "value": 2,
          "name": "cell21 ~ gene20"
        },
        {
          "x": 20,
          "y": 20,
          "value": 1,
          "name": "cell21 ~ gene21"
        },
        {
          "x": 20,
          "y": 21,
          "value": 2,
          "name": "cell21 ~ gene22"
        },
        {
          "x": 20,
          "y": 22,
          "value": 0,
          "name": "cell21 ~ gene23"
        },
        {
          "x": 20,
          "y": 23,
          "value": 22,
          "name": "cell21 ~ gene24"
        },
        {
          "x": 20,
          "y": 24,
          "value": 8,
          "name": "cell21 ~ gene25"
        },
        {
          "x": 20,
          "y": 25,
          "value": 9,
          "name": "cell21 ~ gene26"
        },
        {
          "x": 20,
          "y": 26,
          "value": 0,
          "name": "cell21 ~ gene27"
        },
        {
          "x": 20,
          "y": 27,
          "value": 0,
          "name": "cell21 ~ gene28"
        },
        {
          "x": 20,
          "y": 28,
          "value": 8,
          "name": "cell21 ~ gene29"
        },
        {
          "x": 20,
          "y": 29,
          "value": 0,
          "name": "cell21 ~ gene30"
        },
        {
          "x": 20,
          "y": 30,
          "value": 0,
          "name": "cell21 ~ gene31"
        },
        {
          "x": 20,
          "y": 31,
          "value": 10,
          "name": "cell21 ~ gene32"
        },
        {
          "x": 20,
          "y": 32,
          "value": 7,
          "name": "cell21 ~ gene33"
        },
        {
          "x": 20,
          "y": 33,
          "value": 0,
          "name": "cell21 ~ gene34"
        },
        {
          "x": 20,
          "y": 34,
          "value": 0,
          "name": "cell21 ~ gene35"
        },
        {
          "x": 20,
          "y": 35,
          "value": 0,
          "name": "cell21 ~ gene36"
        },
        {
          "x": 20,
          "y": 36,
          "value": 18,
          "name": "cell21 ~ gene37"
        },
        {
          "x": 20,
          "y": 37,
          "value": 0,
          "name": "cell21 ~ gene38"
        },
        {
          "x": 20,
          "y": 38,
          "value": 0,
          "name": "cell21 ~ gene39"
        },
        {
          "x": 20,
          "y": 39,
          "value": 6,
          "name": "cell21 ~ gene40"
        },
        {
          "x": 20,
          "y": 40,
          "value": 0,
          "name": "cell21 ~ gene41"
        },
        {
          "x": 20,
          "y": 41,
          "value": 0,
          "name": "cell21 ~ gene42"
        },
        {
          "x": 20,
          "y": 42,
          "value": 0,
          "name": "cell21 ~ gene43"
        },
        {
          "x": 20,
          "y": 43,
          "value": 21,
          "name": "cell21 ~ gene44"
        },
        {
          "x": 20,
          "y": 44,
          "value": 0,
          "name": "cell21 ~ gene45"
        },
        {
          "x": 20,
          "y": 45,
          "value": 0,
          "name": "cell21 ~ gene46"
        },
        {
          "x": 20,
          "y": 46,
          "value": 10,
          "name": "cell21 ~ gene47"
        },
        {
          "x": 20,
          "y": 47,
          "value": 0,
          "name": "cell21 ~ gene48"
        },
        {
          "x": 20,
          "y": 48,
          "value": 17,
          "name": "cell21 ~ gene49"
        },
        {
          "x": 20,
          "y": 49,
          "value": 4,
          "name": "cell21 ~ gene50"
        },
        {
          "x": 20,
          "y": 50,
          "value": 6,
          "name": "cell21 ~ gene51"
        },
        {
          "x": 20,
          "y": 51,
          "value": 21,
          "name": "cell21 ~ gene52"
        },
        {
          "x": 20,
          "y": 52,
          "value": 0,
          "name": "cell21 ~ gene53"
        },
        {
          "x": 20,
          "y": 53,
          "value": 13,
          "name": "cell21 ~ gene54"
        },
        {
          "x": 20,
          "y": 54,
          "value": 0,
          "name": "cell21 ~ gene55"
        },
        {
          "x": 20,
          "y": 55,
          "value": 0,
          "name": "cell21 ~ gene56"
        },
        {
          "x": 20,
          "y": 56,
          "value": 0,
          "name": "cell21 ~ gene57"
        },
        {
          "x": 20,
          "y": 57,
          "value": 3,
          "name": "cell21 ~ gene58"
        },
        {
          "x": 20,
          "y": 58,
          "value": 4,
          "name": "cell21 ~ gene59"
        },
        {
          "x": 20,
          "y": 59,
          "value": 0,
          "name": "cell21 ~ gene60"
        },
        {
          "x": 20,
          "y": 60,
          "value": 0,
          "name": "cell21 ~ gene61"
        },
        {
          "x": 20,
          "y": 61,
          "value": 6,
          "name": "cell21 ~ gene62"
        },
        {
          "x": 20,
          "y": 62,
          "value": 0,
          "name": "cell21 ~ gene63"
        },
        {
          "x": 20,
          "y": 63,
          "value": 0,
          "name": "cell21 ~ gene64"
        },
        {
          "x": 20,
          "y": 64,
          "value": 0,
          "name": "cell21 ~ gene65"
        },
        {
          "x": 20,
          "y": 65,
          "value": 4,
          "name": "cell21 ~ gene66"
        },
        {
          "x": 20,
          "y": 66,
          "value": 0,
          "name": "cell21 ~ gene67"
        },
        {
          "x": 20,
          "y": 67,
          "value": 0,
          "name": "cell21 ~ gene68"
        },
        {
          "x": 20,
          "y": 68,
          "value": 0,
          "name": "cell21 ~ gene69"
        },
        {
          "x": 20,
          "y": 69,
          "value": 14,
          "name": "cell21 ~ gene70"
        },
        {
          "x": 20,
          "y": 70,
          "value": 2,
          "name": "cell21 ~ gene71"
        },
        {
          "x": 20,
          "y": 71,
          "value": 6,
          "name": "cell21 ~ gene72"
        },
        {
          "x": 20,
          "y": 72,
          "value": 30,
          "name": "cell21 ~ gene73"
        },
        {
          "x": 20,
          "y": 73,
          "value": 11,
          "name": "cell21 ~ gene74"
        },
        {
          "x": 20,
          "y": 74,
          "value": 0,
          "name": "cell21 ~ gene75"
        },
        {
          "x": 20,
          "y": 75,
          "value": 0,
          "name": "cell21 ~ gene76"
        },
        {
          "x": 20,
          "y": 76,
          "value": 4,
          "name": "cell21 ~ gene77"
        },
        {
          "x": 20,
          "y": 77,
          "value": 0,
          "name": "cell21 ~ gene78"
        },
        {
          "x": 20,
          "y": 78,
          "value": 12,
          "name": "cell21 ~ gene79"
        },
        {
          "x": 20,
          "y": 79,
          "value": 8,
          "name": "cell21 ~ gene80"
        },
        {
          "x": 20,
          "y": 80,
          "value": 4,
          "name": "cell21 ~ gene81"
        },
        {
          "x": 20,
          "y": 81,
          "value": 3,
          "name": "cell21 ~ gene82"
        },
        {
          "x": 20,
          "y": 82,
          "value": 5,
          "name": "cell21 ~ gene83"
        },
        {
          "x": 20,
          "y": 83,
          "value": 0,
          "name": "cell21 ~ gene84"
        },
        {
          "x": 20,
          "y": 84,
          "value": 3,
          "name": "cell21 ~ gene85"
        },
        {
          "x": 20,
          "y": 85,
          "value": 2,
          "name": "cell21 ~ gene86"
        },
        {
          "x": 20,
          "y": 86,
          "value": 0,
          "name": "cell21 ~ gene87"
        },
        {
          "x": 20,
          "y": 87,
          "value": 0,
          "name": "cell21 ~ gene88"
        },
        {
          "x": 20,
          "y": 88,
          "value": 0,
          "name": "cell21 ~ gene89"
        },
        {
          "x": 20,
          "y": 89,
          "value": 2,
          "name": "cell21 ~ gene90"
        },
        {
          "x": 20,
          "y": 90,
          "value": 7,
          "name": "cell21 ~ gene91"
        },
        {
          "x": 20,
          "y": 91,
          "value": 3,
          "name": "cell21 ~ gene92"
        },
        {
          "x": 20,
          "y": 92,
          "value": 10,
          "name": "cell21 ~ gene93"
        },
        {
          "x": 20,
          "y": 93,
          "value": 0,
          "name": "cell21 ~ gene94"
        },
        {
          "x": 20,
          "y": 94,
          "value": 0,
          "name": "cell21 ~ gene95"
        },
        {
          "x": 20,
          "y": 95,
          "value": 0,
          "name": "cell21 ~ gene96"
        },
        {
          "x": 20,
          "y": 96,
          "value": 11,
          "name": "cell21 ~ gene97"
        },
        {
          "x": 20,
          "y": 97,
          "value": 0,
          "name": "cell21 ~ gene98"
        },
        {
          "x": 20,
          "y": 98,
          "value": 0,
          "name": "cell21 ~ gene99"
        },
        {
          "x": 20,
          "y": 99,
          "value": 18,
          "name": "cell21 ~ gene100"
        },
        {
          "x": 21,
          "y": 0,
          "value": 13,
          "name": "cell22 ~ gene1"
        },
        {
          "x": 21,
          "y": 1,
          "value": 0,
          "name": "cell22 ~ gene2"
        },
        {
          "x": 21,
          "y": 2,
          "value": 23,
          "name": "cell22 ~ gene3"
        },
        {
          "x": 21,
          "y": 3,
          "value": 10,
          "name": "cell22 ~ gene4"
        },
        {
          "x": 21,
          "y": 4,
          "value": 21,
          "name": "cell22 ~ gene5"
        },
        {
          "x": 21,
          "y": 5,
          "value": 24,
          "name": "cell22 ~ gene6"
        },
        {
          "x": 21,
          "y": 6,
          "value": 44,
          "name": "cell22 ~ gene7"
        },
        {
          "x": 21,
          "y": 7,
          "value": 14,
          "name": "cell22 ~ gene8"
        },
        {
          "x": 21,
          "y": 8,
          "value": 12,
          "name": "cell22 ~ gene9"
        },
        {
          "x": 21,
          "y": 9,
          "value": 0,
          "name": "cell22 ~ gene10"
        },
        {
          "x": 21,
          "y": 10,
          "value": 7,
          "name": "cell22 ~ gene11"
        },
        {
          "x": 21,
          "y": 11,
          "value": 0,
          "name": "cell22 ~ gene12"
        },
        {
          "x": 21,
          "y": 12,
          "value": 30,
          "name": "cell22 ~ gene13"
        },
        {
          "x": 21,
          "y": 13,
          "value": 0,
          "name": "cell22 ~ gene14"
        },
        {
          "x": 21,
          "y": 14,
          "value": 17,
          "name": "cell22 ~ gene15"
        },
        {
          "x": 21,
          "y": 15,
          "value": 47,
          "name": "cell22 ~ gene16"
        },
        {
          "x": 21,
          "y": 16,
          "value": 18,
          "name": "cell22 ~ gene17"
        },
        {
          "x": 21,
          "y": 17,
          "value": 5,
          "name": "cell22 ~ gene18"
        },
        {
          "x": 21,
          "y": 18,
          "value": 25,
          "name": "cell22 ~ gene19"
        },
        {
          "x": 21,
          "y": 19,
          "value": 0,
          "name": "cell22 ~ gene20"
        },
        {
          "x": 21,
          "y": 20,
          "value": 8,
          "name": "cell22 ~ gene21"
        },
        {
          "x": 21,
          "y": 21,
          "value": 0,
          "name": "cell22 ~ gene22"
        },
        {
          "x": 21,
          "y": 22,
          "value": 14,
          "name": "cell22 ~ gene23"
        },
        {
          "x": 21,
          "y": 23,
          "value": 2,
          "name": "cell22 ~ gene24"
        },
        {
          "x": 21,
          "y": 24,
          "value": 18,
          "name": "cell22 ~ gene25"
        },
        {
          "x": 21,
          "y": 25,
          "value": 0,
          "name": "cell22 ~ gene26"
        },
        {
          "x": 21,
          "y": 26,
          "value": 1,
          "name": "cell22 ~ gene27"
        },
        {
          "x": 21,
          "y": 27,
          "value": 0,
          "name": "cell22 ~ gene28"
        },
        {
          "x": 21,
          "y": 28,
          "value": 2,
          "name": "cell22 ~ gene29"
        },
        {
          "x": 21,
          "y": 29,
          "value": 0,
          "name": "cell22 ~ gene30"
        },
        {
          "x": 21,
          "y": 30,
          "value": 0,
          "name": "cell22 ~ gene31"
        },
        {
          "x": 21,
          "y": 31,
          "value": 6,
          "name": "cell22 ~ gene32"
        },
        {
          "x": 21,
          "y": 32,
          "value": 6,
          "name": "cell22 ~ gene33"
        },
        {
          "x": 21,
          "y": 33,
          "value": 0,
          "name": "cell22 ~ gene34"
        },
        {
          "x": 21,
          "y": 34,
          "value": 0,
          "name": "cell22 ~ gene35"
        },
        {
          "x": 21,
          "y": 35,
          "value": 6,
          "name": "cell22 ~ gene36"
        },
        {
          "x": 21,
          "y": 36,
          "value": 0,
          "name": "cell22 ~ gene37"
        },
        {
          "x": 21,
          "y": 37,
          "value": 10,
          "name": "cell22 ~ gene38"
        },
        {
          "x": 21,
          "y": 38,
          "value": 1,
          "name": "cell22 ~ gene39"
        },
        {
          "x": 21,
          "y": 39,
          "value": 0,
          "name": "cell22 ~ gene40"
        },
        {
          "x": 21,
          "y": 40,
          "value": 0,
          "name": "cell22 ~ gene41"
        },
        {
          "x": 21,
          "y": 41,
          "value": 0,
          "name": "cell22 ~ gene42"
        },
        {
          "x": 21,
          "y": 42,
          "value": 8,
          "name": "cell22 ~ gene43"
        },
        {
          "x": 21,
          "y": 43,
          "value": 11,
          "name": "cell22 ~ gene44"
        },
        {
          "x": 21,
          "y": 44,
          "value": 0,
          "name": "cell22 ~ gene45"
        },
        {
          "x": 21,
          "y": 45,
          "value": 9,
          "name": "cell22 ~ gene46"
        },
        {
          "x": 21,
          "y": 46,
          "value": 11,
          "name": "cell22 ~ gene47"
        },
        {
          "x": 21,
          "y": 47,
          "value": 0,
          "name": "cell22 ~ gene48"
        },
        {
          "x": 21,
          "y": 48,
          "value": 0,
          "name": "cell22 ~ gene49"
        },
        {
          "x": 21,
          "y": 49,
          "value": 1,
          "name": "cell22 ~ gene50"
        },
        {
          "x": 21,
          "y": 50,
          "value": 0,
          "name": "cell22 ~ gene51"
        },
        {
          "x": 21,
          "y": 51,
          "value": 0,
          "name": "cell22 ~ gene52"
        },
        {
          "x": 21,
          "y": 52,
          "value": 4,
          "name": "cell22 ~ gene53"
        },
        {
          "x": 21,
          "y": 53,
          "value": 3,
          "name": "cell22 ~ gene54"
        },
        {
          "x": 21,
          "y": 54,
          "value": 23,
          "name": "cell22 ~ gene55"
        },
        {
          "x": 21,
          "y": 55,
          "value": 0,
          "name": "cell22 ~ gene56"
        },
        {
          "x": 21,
          "y": 56,
          "value": 1,
          "name": "cell22 ~ gene57"
        },
        {
          "x": 21,
          "y": 57,
          "value": 0,
          "name": "cell22 ~ gene58"
        },
        {
          "x": 21,
          "y": 58,
          "value": 0,
          "name": "cell22 ~ gene59"
        },
        {
          "x": 21,
          "y": 59,
          "value": 0,
          "name": "cell22 ~ gene60"
        },
        {
          "x": 21,
          "y": 60,
          "value": 7,
          "name": "cell22 ~ gene61"
        },
        {
          "x": 21,
          "y": 61,
          "value": 14,
          "name": "cell22 ~ gene62"
        },
        {
          "x": 21,
          "y": 62,
          "value": 0,
          "name": "cell22 ~ gene63"
        },
        {
          "x": 21,
          "y": 63,
          "value": 0,
          "name": "cell22 ~ gene64"
        },
        {
          "x": 21,
          "y": 64,
          "value": 3,
          "name": "cell22 ~ gene65"
        },
        {
          "x": 21,
          "y": 65,
          "value": 12,
          "name": "cell22 ~ gene66"
        },
        {
          "x": 21,
          "y": 66,
          "value": 8,
          "name": "cell22 ~ gene67"
        },
        {
          "x": 21,
          "y": 67,
          "value": 0,
          "name": "cell22 ~ gene68"
        },
        {
          "x": 21,
          "y": 68,
          "value": 0,
          "name": "cell22 ~ gene69"
        },
        {
          "x": 21,
          "y": 69,
          "value": 0,
          "name": "cell22 ~ gene70"
        },
        {
          "x": 21,
          "y": 70,
          "value": 9,
          "name": "cell22 ~ gene71"
        },
        {
          "x": 21,
          "y": 71,
          "value": 10,
          "name": "cell22 ~ gene72"
        },
        {
          "x": 21,
          "y": 72,
          "value": 0,
          "name": "cell22 ~ gene73"
        },
        {
          "x": 21,
          "y": 73,
          "value": 2,
          "name": "cell22 ~ gene74"
        },
        {
          "x": 21,
          "y": 74,
          "value": 0,
          "name": "cell22 ~ gene75"
        },
        {
          "x": 21,
          "y": 75,
          "value": 20,
          "name": "cell22 ~ gene76"
        },
        {
          "x": 21,
          "y": 76,
          "value": 2,
          "name": "cell22 ~ gene77"
        },
        {
          "x": 21,
          "y": 77,
          "value": 0,
          "name": "cell22 ~ gene78"
        },
        {
          "x": 21,
          "y": 78,
          "value": 5,
          "name": "cell22 ~ gene79"
        },
        {
          "x": 21,
          "y": 79,
          "value": 0,
          "name": "cell22 ~ gene80"
        },
        {
          "x": 21,
          "y": 80,
          "value": 0,
          "name": "cell22 ~ gene81"
        },
        {
          "x": 21,
          "y": 81,
          "value": 3,
          "name": "cell22 ~ gene82"
        },
        {
          "x": 21,
          "y": 82,
          "value": 11,
          "name": "cell22 ~ gene83"
        },
        {
          "x": 21,
          "y": 83,
          "value": 0,
          "name": "cell22 ~ gene84"
        },
        {
          "x": 21,
          "y": 84,
          "value": 25,
          "name": "cell22 ~ gene85"
        },
        {
          "x": 21,
          "y": 85,
          "value": 9,
          "name": "cell22 ~ gene86"
        },
        {
          "x": 21,
          "y": 86,
          "value": 0,
          "name": "cell22 ~ gene87"
        },
        {
          "x": 21,
          "y": 87,
          "value": 0,
          "name": "cell22 ~ gene88"
        },
        {
          "x": 21,
          "y": 88,
          "value": 0,
          "name": "cell22 ~ gene89"
        },
        {
          "x": 21,
          "y": 89,
          "value": 0,
          "name": "cell22 ~ gene90"
        },
        {
          "x": 21,
          "y": 90,
          "value": 4,
          "name": "cell22 ~ gene91"
        },
        {
          "x": 21,
          "y": 91,
          "value": 6,
          "name": "cell22 ~ gene92"
        },
        {
          "x": 21,
          "y": 92,
          "value": 0,
          "name": "cell22 ~ gene93"
        },
        {
          "x": 21,
          "y": 93,
          "value": 11,
          "name": "cell22 ~ gene94"
        },
        {
          "x": 21,
          "y": 94,
          "value": 0,
          "name": "cell22 ~ gene95"
        },
        {
          "x": 21,
          "y": 95,
          "value": 0,
          "name": "cell22 ~ gene96"
        },
        {
          "x": 21,
          "y": 96,
          "value": 26,
          "name": "cell22 ~ gene97"
        },
        {
          "x": 21,
          "y": 97,
          "value": 13,
          "name": "cell22 ~ gene98"
        },
        {
          "x": 21,
          "y": 98,
          "value": 0,
          "name": "cell22 ~ gene99"
        },
        {
          "x": 21,
          "y": 99,
          "value": 0,
          "name": "cell22 ~ gene100"
        },
        {
          "x": 22,
          "y": 0,
          "value": 0,
          "name": "cell23 ~ gene1"
        },
        {
          "x": 22,
          "y": 1,
          "value": 0,
          "name": "cell23 ~ gene2"
        },
        {
          "x": 22,
          "y": 2,
          "value": 32,
          "name": "cell23 ~ gene3"
        },
        {
          "x": 22,
          "y": 3,
          "value": 27,
          "name": "cell23 ~ gene4"
        },
        {
          "x": 22,
          "y": 4,
          "value": 26,
          "name": "cell23 ~ gene5"
        },
        {
          "x": 22,
          "y": 5,
          "value": 8,
          "name": "cell23 ~ gene6"
        },
        {
          "x": 22,
          "y": 6,
          "value": 38,
          "name": "cell23 ~ gene7"
        },
        {
          "x": 22,
          "y": 7,
          "value": 20,
          "name": "cell23 ~ gene8"
        },
        {
          "x": 22,
          "y": 8,
          "value": 7,
          "name": "cell23 ~ gene9"
        },
        {
          "x": 22,
          "y": 9,
          "value": 0,
          "name": "cell23 ~ gene10"
        },
        {
          "x": 22,
          "y": 10,
          "value": 6,
          "name": "cell23 ~ gene11"
        },
        {
          "x": 22,
          "y": 11,
          "value": 0,
          "name": "cell23 ~ gene12"
        },
        {
          "x": 22,
          "y": 12,
          "value": 4,
          "name": "cell23 ~ gene13"
        },
        {
          "x": 22,
          "y": 13,
          "value": 5,
          "name": "cell23 ~ gene14"
        },
        {
          "x": 22,
          "y": 14,
          "value": 30,
          "name": "cell23 ~ gene15"
        },
        {
          "x": 22,
          "y": 15,
          "value": 27,
          "name": "cell23 ~ gene16"
        },
        {
          "x": 22,
          "y": 16,
          "value": 20,
          "name": "cell23 ~ gene17"
        },
        {
          "x": 22,
          "y": 17,
          "value": 24,
          "name": "cell23 ~ gene18"
        },
        {
          "x": 22,
          "y": 18,
          "value": 15,
          "name": "cell23 ~ gene19"
        },
        {
          "x": 22,
          "y": 19,
          "value": 5,
          "name": "cell23 ~ gene20"
        },
        {
          "x": 22,
          "y": 20,
          "value": 3,
          "name": "cell23 ~ gene21"
        },
        {
          "x": 22,
          "y": 21,
          "value": 0,
          "name": "cell23 ~ gene22"
        },
        {
          "x": 22,
          "y": 22,
          "value": 18,
          "name": "cell23 ~ gene23"
        },
        {
          "x": 22,
          "y": 23,
          "value": 11,
          "name": "cell23 ~ gene24"
        },
        {
          "x": 22,
          "y": 24,
          "value": 0,
          "name": "cell23 ~ gene25"
        },
        {
          "x": 22,
          "y": 25,
          "value": 5,
          "name": "cell23 ~ gene26"
        },
        {
          "x": 22,
          "y": 26,
          "value": 15,
          "name": "cell23 ~ gene27"
        },
        {
          "x": 22,
          "y": 27,
          "value": 0,
          "name": "cell23 ~ gene28"
        },
        {
          "x": 22,
          "y": 28,
          "value": 0,
          "name": "cell23 ~ gene29"
        },
        {
          "x": 22,
          "y": 29,
          "value": 0,
          "name": "cell23 ~ gene30"
        },
        {
          "x": 22,
          "y": 30,
          "value": 0,
          "name": "cell23 ~ gene31"
        },
        {
          "x": 22,
          "y": 31,
          "value": 1,
          "name": "cell23 ~ gene32"
        },
        {
          "x": 22,
          "y": 32,
          "value": 0,
          "name": "cell23 ~ gene33"
        },
        {
          "x": 22,
          "y": 33,
          "value": 0,
          "name": "cell23 ~ gene34"
        },
        {
          "x": 22,
          "y": 34,
          "value": 0,
          "name": "cell23 ~ gene35"
        },
        {
          "x": 22,
          "y": 35,
          "value": 7,
          "name": "cell23 ~ gene36"
        },
        {
          "x": 22,
          "y": 36,
          "value": 0,
          "name": "cell23 ~ gene37"
        },
        {
          "x": 22,
          "y": 37,
          "value": 0,
          "name": "cell23 ~ gene38"
        },
        {
          "x": 22,
          "y": 38,
          "value": 4,
          "name": "cell23 ~ gene39"
        },
        {
          "x": 22,
          "y": 39,
          "value": 0,
          "name": "cell23 ~ gene40"
        },
        {
          "x": 22,
          "y": 40,
          "value": 0,
          "name": "cell23 ~ gene41"
        },
        {
          "x": 22,
          "y": 41,
          "value": 3,
          "name": "cell23 ~ gene42"
        },
        {
          "x": 22,
          "y": 42,
          "value": 0,
          "name": "cell23 ~ gene43"
        },
        {
          "x": 22,
          "y": 43,
          "value": 4,
          "name": "cell23 ~ gene44"
        },
        {
          "x": 22,
          "y": 44,
          "value": 0,
          "name": "cell23 ~ gene45"
        },
        {
          "x": 22,
          "y": 45,
          "value": 0,
          "name": "cell23 ~ gene46"
        },
        {
          "x": 22,
          "y": 46,
          "value": 6,
          "name": "cell23 ~ gene47"
        },
        {
          "x": 22,
          "y": 47,
          "value": 0,
          "name": "cell23 ~ gene48"
        },
        {
          "x": 22,
          "y": 48,
          "value": 6,
          "name": "cell23 ~ gene49"
        },
        {
          "x": 22,
          "y": 49,
          "value": 11,
          "name": "cell23 ~ gene50"
        },
        {
          "x": 22,
          "y": 50,
          "value": 0,
          "name": "cell23 ~ gene51"
        },
        {
          "x": 22,
          "y": 51,
          "value": 0,
          "name": "cell23 ~ gene52"
        },
        {
          "x": 22,
          "y": 52,
          "value": 0,
          "name": "cell23 ~ gene53"
        },
        {
          "x": 22,
          "y": 53,
          "value": 5,
          "name": "cell23 ~ gene54"
        },
        {
          "x": 22,
          "y": 54,
          "value": 0,
          "name": "cell23 ~ gene55"
        },
        {
          "x": 22,
          "y": 55,
          "value": 0,
          "name": "cell23 ~ gene56"
        },
        {
          "x": 22,
          "y": 56,
          "value": 17,
          "name": "cell23 ~ gene57"
        },
        {
          "x": 22,
          "y": 57,
          "value": 19,
          "name": "cell23 ~ gene58"
        },
        {
          "x": 22,
          "y": 58,
          "value": 0,
          "name": "cell23 ~ gene59"
        },
        {
          "x": 22,
          "y": 59,
          "value": 0,
          "name": "cell23 ~ gene60"
        },
        {
          "x": 22,
          "y": 60,
          "value": 0,
          "name": "cell23 ~ gene61"
        },
        {
          "x": 22,
          "y": 61,
          "value": 8,
          "name": "cell23 ~ gene62"
        },
        {
          "x": 22,
          "y": 62,
          "value": 6,
          "name": "cell23 ~ gene63"
        },
        {
          "x": 22,
          "y": 63,
          "value": 0,
          "name": "cell23 ~ gene64"
        },
        {
          "x": 22,
          "y": 64,
          "value": 0,
          "name": "cell23 ~ gene65"
        },
        {
          "x": 22,
          "y": 65,
          "value": 10,
          "name": "cell23 ~ gene66"
        },
        {
          "x": 22,
          "y": 66,
          "value": 2,
          "name": "cell23 ~ gene67"
        },
        {
          "x": 22,
          "y": 67,
          "value": 3,
          "name": "cell23 ~ gene68"
        },
        {
          "x": 22,
          "y": 68,
          "value": 2,
          "name": "cell23 ~ gene69"
        },
        {
          "x": 22,
          "y": 69,
          "value": 17,
          "name": "cell23 ~ gene70"
        },
        {
          "x": 22,
          "y": 70,
          "value": 0,
          "name": "cell23 ~ gene71"
        },
        {
          "x": 22,
          "y": 71,
          "value": 20,
          "name": "cell23 ~ gene72"
        },
        {
          "x": 22,
          "y": 72,
          "value": 0,
          "name": "cell23 ~ gene73"
        },
        {
          "x": 22,
          "y": 73,
          "value": 0,
          "name": "cell23 ~ gene74"
        },
        {
          "x": 22,
          "y": 74,
          "value": 0,
          "name": "cell23 ~ gene75"
        },
        {
          "x": 22,
          "y": 75,
          "value": 0,
          "name": "cell23 ~ gene76"
        },
        {
          "x": 22,
          "y": 76,
          "value": 0,
          "name": "cell23 ~ gene77"
        },
        {
          "x": 22,
          "y": 77,
          "value": 0,
          "name": "cell23 ~ gene78"
        },
        {
          "x": 22,
          "y": 78,
          "value": 0,
          "name": "cell23 ~ gene79"
        },
        {
          "x": 22,
          "y": 79,
          "value": 0,
          "name": "cell23 ~ gene80"
        },
        {
          "x": 22,
          "y": 80,
          "value": 26,
          "name": "cell23 ~ gene81"
        },
        {
          "x": 22,
          "y": 81,
          "value": 6,
          "name": "cell23 ~ gene82"
        },
        {
          "x": 22,
          "y": 82,
          "value": 0,
          "name": "cell23 ~ gene83"
        },
        {
          "x": 22,
          "y": 83,
          "value": 21,
          "name": "cell23 ~ gene84"
        },
        {
          "x": 22,
          "y": 84,
          "value": 10,
          "name": "cell23 ~ gene85"
        },
        {
          "x": 22,
          "y": 85,
          "value": 11,
          "name": "cell23 ~ gene86"
        },
        {
          "x": 22,
          "y": 86,
          "value": 0,
          "name": "cell23 ~ gene87"
        },
        {
          "x": 22,
          "y": 87,
          "value": 1,
          "name": "cell23 ~ gene88"
        },
        {
          "x": 22,
          "y": 88,
          "value": 11,
          "name": "cell23 ~ gene89"
        },
        {
          "x": 22,
          "y": 89,
          "value": 0,
          "name": "cell23 ~ gene90"
        },
        {
          "x": 22,
          "y": 90,
          "value": 0,
          "name": "cell23 ~ gene91"
        },
        {
          "x": 22,
          "y": 91,
          "value": 2,
          "name": "cell23 ~ gene92"
        },
        {
          "x": 22,
          "y": 92,
          "value": 0,
          "name": "cell23 ~ gene93"
        },
        {
          "x": 22,
          "y": 93,
          "value": 2,
          "name": "cell23 ~ gene94"
        },
        {
          "x": 22,
          "y": 94,
          "value": 16,
          "name": "cell23 ~ gene95"
        },
        {
          "x": 22,
          "y": 95,
          "value": 18,
          "name": "cell23 ~ gene96"
        },
        {
          "x": 22,
          "y": 96,
          "value": 0,
          "name": "cell23 ~ gene97"
        },
        {
          "x": 22,
          "y": 97,
          "value": 1,
          "name": "cell23 ~ gene98"
        },
        {
          "x": 22,
          "y": 98,
          "value": 0,
          "name": "cell23 ~ gene99"
        },
        {
          "x": 22,
          "y": 99,
          "value": 1,
          "name": "cell23 ~ gene100"
        },
        {
          "x": 23,
          "y": 0,
          "value": 4,
          "name": "cell24 ~ gene1"
        },
        {
          "x": 23,
          "y": 1,
          "value": 8,
          "name": "cell24 ~ gene2"
        },
        {
          "x": 23,
          "y": 2,
          "value": 20,
          "name": "cell24 ~ gene3"
        },
        {
          "x": 23,
          "y": 3,
          "value": 26,
          "name": "cell24 ~ gene4"
        },
        {
          "x": 23,
          "y": 4,
          "value": 23,
          "name": "cell24 ~ gene5"
        },
        {
          "x": 23,
          "y": 5,
          "value": 0,
          "name": "cell24 ~ gene6"
        },
        {
          "x": 23,
          "y": 6,
          "value": 32,
          "name": "cell24 ~ gene7"
        },
        {
          "x": 23,
          "y": 7,
          "value": 13,
          "name": "cell24 ~ gene8"
        },
        {
          "x": 23,
          "y": 8,
          "value": 0,
          "name": "cell24 ~ gene9"
        },
        {
          "x": 23,
          "y": 9,
          "value": 0,
          "name": "cell24 ~ gene10"
        },
        {
          "x": 23,
          "y": 10,
          "value": 6,
          "name": "cell24 ~ gene11"
        },
        {
          "x": 23,
          "y": 11,
          "value": 0,
          "name": "cell24 ~ gene12"
        },
        {
          "x": 23,
          "y": 12,
          "value": 30,
          "name": "cell24 ~ gene13"
        },
        {
          "x": 23,
          "y": 13,
          "value": 19,
          "name": "cell24 ~ gene14"
        },
        {
          "x": 23,
          "y": 14,
          "value": 33,
          "name": "cell24 ~ gene15"
        },
        {
          "x": 23,
          "y": 15,
          "value": 34,
          "name": "cell24 ~ gene16"
        },
        {
          "x": 23,
          "y": 16,
          "value": 10,
          "name": "cell24 ~ gene17"
        },
        {
          "x": 23,
          "y": 17,
          "value": 13,
          "name": "cell24 ~ gene18"
        },
        {
          "x": 23,
          "y": 18,
          "value": 20,
          "name": "cell24 ~ gene19"
        },
        {
          "x": 23,
          "y": 19,
          "value": 20,
          "name": "cell24 ~ gene20"
        },
        {
          "x": 23,
          "y": 20,
          "value": 4,
          "name": "cell24 ~ gene21"
        },
        {
          "x": 23,
          "y": 21,
          "value": 0,
          "name": "cell24 ~ gene22"
        },
        {
          "x": 23,
          "y": 22,
          "value": 0,
          "name": "cell24 ~ gene23"
        },
        {
          "x": 23,
          "y": 23,
          "value": 0,
          "name": "cell24 ~ gene24"
        },
        {
          "x": 23,
          "y": 24,
          "value": 13,
          "name": "cell24 ~ gene25"
        },
        {
          "x": 23,
          "y": 25,
          "value": 0,
          "name": "cell24 ~ gene26"
        },
        {
          "x": 23,
          "y": 26,
          "value": 0,
          "name": "cell24 ~ gene27"
        },
        {
          "x": 23,
          "y": 27,
          "value": 2,
          "name": "cell24 ~ gene28"
        },
        {
          "x": 23,
          "y": 28,
          "value": 4,
          "name": "cell24 ~ gene29"
        },
        {
          "x": 23,
          "y": 29,
          "value": 10,
          "name": "cell24 ~ gene30"
        },
        {
          "x": 23,
          "y": 30,
          "value": 0,
          "name": "cell24 ~ gene31"
        },
        {
          "x": 23,
          "y": 31,
          "value": 4,
          "name": "cell24 ~ gene32"
        },
        {
          "x": 23,
          "y": 32,
          "value": 0,
          "name": "cell24 ~ gene33"
        },
        {
          "x": 23,
          "y": 33,
          "value": 0,
          "name": "cell24 ~ gene34"
        },
        {
          "x": 23,
          "y": 34,
          "value": 0,
          "name": "cell24 ~ gene35"
        },
        {
          "x": 23,
          "y": 35,
          "value": 0,
          "name": "cell24 ~ gene36"
        },
        {
          "x": 23,
          "y": 36,
          "value": 0,
          "name": "cell24 ~ gene37"
        },
        {
          "x": 23,
          "y": 37,
          "value": 0,
          "name": "cell24 ~ gene38"
        },
        {
          "x": 23,
          "y": 38,
          "value": 0,
          "name": "cell24 ~ gene39"
        },
        {
          "x": 23,
          "y": 39,
          "value": 0,
          "name": "cell24 ~ gene40"
        },
        {
          "x": 23,
          "y": 40,
          "value": 21,
          "name": "cell24 ~ gene41"
        },
        {
          "x": 23,
          "y": 41,
          "value": 17,
          "name": "cell24 ~ gene42"
        },
        {
          "x": 23,
          "y": 42,
          "value": 0,
          "name": "cell24 ~ gene43"
        },
        {
          "x": 23,
          "y": 43,
          "value": 21,
          "name": "cell24 ~ gene44"
        },
        {
          "x": 23,
          "y": 44,
          "value": 0,
          "name": "cell24 ~ gene45"
        },
        {
          "x": 23,
          "y": 45,
          "value": 12,
          "name": "cell24 ~ gene46"
        },
        {
          "x": 23,
          "y": 46,
          "value": 3,
          "name": "cell24 ~ gene47"
        },
        {
          "x": 23,
          "y": 47,
          "value": 0,
          "name": "cell24 ~ gene48"
        },
        {
          "x": 23,
          "y": 48,
          "value": 0,
          "name": "cell24 ~ gene49"
        },
        {
          "x": 23,
          "y": 49,
          "value": 10,
          "name": "cell24 ~ gene50"
        },
        {
          "x": 23,
          "y": 50,
          "value": 10,
          "name": "cell24 ~ gene51"
        },
        {
          "x": 23,
          "y": 51,
          "value": 0,
          "name": "cell24 ~ gene52"
        },
        {
          "x": 23,
          "y": 52,
          "value": 8,
          "name": "cell24 ~ gene53"
        },
        {
          "x": 23,
          "y": 53,
          "value": 8,
          "name": "cell24 ~ gene54"
        },
        {
          "x": 23,
          "y": 54,
          "value": 0,
          "name": "cell24 ~ gene55"
        },
        {
          "x": 23,
          "y": 55,
          "value": 0,
          "name": "cell24 ~ gene56"
        },
        {
          "x": 23,
          "y": 56,
          "value": 0,
          "name": "cell24 ~ gene57"
        },
        {
          "x": 23,
          "y": 57,
          "value": 0,
          "name": "cell24 ~ gene58"
        },
        {
          "x": 23,
          "y": 58,
          "value": 0,
          "name": "cell24 ~ gene59"
        },
        {
          "x": 23,
          "y": 59,
          "value": 15,
          "name": "cell24 ~ gene60"
        },
        {
          "x": 23,
          "y": 60,
          "value": 0,
          "name": "cell24 ~ gene61"
        },
        {
          "x": 23,
          "y": 61,
          "value": 5,
          "name": "cell24 ~ gene62"
        },
        {
          "x": 23,
          "y": 62,
          "value": 1,
          "name": "cell24 ~ gene63"
        },
        {
          "x": 23,
          "y": 63,
          "value": 15,
          "name": "cell24 ~ gene64"
        },
        {
          "x": 23,
          "y": 64,
          "value": 7,
          "name": "cell24 ~ gene65"
        },
        {
          "x": 23,
          "y": 65,
          "value": 6,
          "name": "cell24 ~ gene66"
        },
        {
          "x": 23,
          "y": 66,
          "value": 0,
          "name": "cell24 ~ gene67"
        },
        {
          "x": 23,
          "y": 67,
          "value": 4,
          "name": "cell24 ~ gene68"
        },
        {
          "x": 23,
          "y": 68,
          "value": 8,
          "name": "cell24 ~ gene69"
        },
        {
          "x": 23,
          "y": 69,
          "value": 0,
          "name": "cell24 ~ gene70"
        },
        {
          "x": 23,
          "y": 70,
          "value": 23,
          "name": "cell24 ~ gene71"
        },
        {
          "x": 23,
          "y": 71,
          "value": 6,
          "name": "cell24 ~ gene72"
        },
        {
          "x": 23,
          "y": 72,
          "value": 0,
          "name": "cell24 ~ gene73"
        },
        {
          "x": 23,
          "y": 73,
          "value": 9,
          "name": "cell24 ~ gene74"
        },
        {
          "x": 23,
          "y": 74,
          "value": 2,
          "name": "cell24 ~ gene75"
        },
        {
          "x": 23,
          "y": 75,
          "value": 0,
          "name": "cell24 ~ gene76"
        },
        {
          "x": 23,
          "y": 76,
          "value": 1,
          "name": "cell24 ~ gene77"
        },
        {
          "x": 23,
          "y": 77,
          "value": 3,
          "name": "cell24 ~ gene78"
        },
        {
          "x": 23,
          "y": 78,
          "value": 2,
          "name": "cell24 ~ gene79"
        },
        {
          "x": 23,
          "y": 79,
          "value": 1,
          "name": "cell24 ~ gene80"
        },
        {
          "x": 23,
          "y": 80,
          "value": 0,
          "name": "cell24 ~ gene81"
        },
        {
          "x": 23,
          "y": 81,
          "value": 8,
          "name": "cell24 ~ gene82"
        },
        {
          "x": 23,
          "y": 82,
          "value": 17,
          "name": "cell24 ~ gene83"
        },
        {
          "x": 23,
          "y": 83,
          "value": 0,
          "name": "cell24 ~ gene84"
        },
        {
          "x": 23,
          "y": 84,
          "value": 1,
          "name": "cell24 ~ gene85"
        },
        {
          "x": 23,
          "y": 85,
          "value": 5,
          "name": "cell24 ~ gene86"
        },
        {
          "x": 23,
          "y": 86,
          "value": 1,
          "name": "cell24 ~ gene87"
        },
        {
          "x": 23,
          "y": 87,
          "value": 0,
          "name": "cell24 ~ gene88"
        },
        {
          "x": 23,
          "y": 88,
          "value": 8,
          "name": "cell24 ~ gene89"
        },
        {
          "x": 23,
          "y": 89,
          "value": 17,
          "name": "cell24 ~ gene90"
        },
        {
          "x": 23,
          "y": 90,
          "value": 9,
          "name": "cell24 ~ gene91"
        },
        {
          "x": 23,
          "y": 91,
          "value": 6,
          "name": "cell24 ~ gene92"
        },
        {
          "x": 23,
          "y": 92,
          "value": 0,
          "name": "cell24 ~ gene93"
        },
        {
          "x": 23,
          "y": 93,
          "value": 0,
          "name": "cell24 ~ gene94"
        },
        {
          "x": 23,
          "y": 94,
          "value": 0,
          "name": "cell24 ~ gene95"
        },
        {
          "x": 23,
          "y": 95,
          "value": 1,
          "name": "cell24 ~ gene96"
        },
        {
          "x": 23,
          "y": 96,
          "value": 0,
          "name": "cell24 ~ gene97"
        },
        {
          "x": 23,
          "y": 97,
          "value": 12,
          "name": "cell24 ~ gene98"
        },
        {
          "x": 23,
          "y": 98,
          "value": 4,
          "name": "cell24 ~ gene99"
        },
        {
          "x": 23,
          "y": 99,
          "value": 0,
          "name": "cell24 ~ gene100"
        },
        {
          "x": 24,
          "y": 0,
          "value": 19,
          "name": "cell25 ~ gene1"
        },
        {
          "x": 24,
          "y": 1,
          "value": 0,
          "name": "cell25 ~ gene2"
        },
        {
          "x": 24,
          "y": 2,
          "value": 7,
          "name": "cell25 ~ gene3"
        },
        {
          "x": 24,
          "y": 3,
          "value": 22,
          "name": "cell25 ~ gene4"
        },
        {
          "x": 24,
          "y": 4,
          "value": 27,
          "name": "cell25 ~ gene5"
        },
        {
          "x": 24,
          "y": 5,
          "value": 0,
          "name": "cell25 ~ gene6"
        },
        {
          "x": 24,
          "y": 6,
          "value": 18,
          "name": "cell25 ~ gene7"
        },
        {
          "x": 24,
          "y": 7,
          "value": 22,
          "name": "cell25 ~ gene8"
        },
        {
          "x": 24,
          "y": 8,
          "value": 0,
          "name": "cell25 ~ gene9"
        },
        {
          "x": 24,
          "y": 9,
          "value": 15,
          "name": "cell25 ~ gene10"
        },
        {
          "x": 24,
          "y": 10,
          "value": 24,
          "name": "cell25 ~ gene11"
        },
        {
          "x": 24,
          "y": 11,
          "value": 7,
          "name": "cell25 ~ gene12"
        },
        {
          "x": 24,
          "y": 12,
          "value": 14,
          "name": "cell25 ~ gene13"
        },
        {
          "x": 24,
          "y": 13,
          "value": 3,
          "name": "cell25 ~ gene14"
        },
        {
          "x": 24,
          "y": 14,
          "value": 26,
          "name": "cell25 ~ gene15"
        },
        {
          "x": 24,
          "y": 15,
          "value": 34,
          "name": "cell25 ~ gene16"
        },
        {
          "x": 24,
          "y": 16,
          "value": 18,
          "name": "cell25 ~ gene17"
        },
        {
          "x": 24,
          "y": 17,
          "value": 26,
          "name": "cell25 ~ gene18"
        },
        {
          "x": 24,
          "y": 18,
          "value": 33,
          "name": "cell25 ~ gene19"
        },
        {
          "x": 24,
          "y": 19,
          "value": 9,
          "name": "cell25 ~ gene20"
        },
        {
          "x": 24,
          "y": 20,
          "value": 0,
          "name": "cell25 ~ gene21"
        },
        {
          "x": 24,
          "y": 21,
          "value": 0,
          "name": "cell25 ~ gene22"
        },
        {
          "x": 24,
          "y": 22,
          "value": 11,
          "name": "cell25 ~ gene23"
        },
        {
          "x": 24,
          "y": 23,
          "value": 0,
          "name": "cell25 ~ gene24"
        },
        {
          "x": 24,
          "y": 24,
          "value": 7,
          "name": "cell25 ~ gene25"
        },
        {
          "x": 24,
          "y": 25,
          "value": 0,
          "name": "cell25 ~ gene26"
        },
        {
          "x": 24,
          "y": 26,
          "value": 13,
          "name": "cell25 ~ gene27"
        },
        {
          "x": 24,
          "y": 27,
          "value": 0,
          "name": "cell25 ~ gene28"
        },
        {
          "x": 24,
          "y": 28,
          "value": 0,
          "name": "cell25 ~ gene29"
        },
        {
          "x": 24,
          "y": 29,
          "value": 0,
          "name": "cell25 ~ gene30"
        },
        {
          "x": 24,
          "y": 30,
          "value": 0,
          "name": "cell25 ~ gene31"
        },
        {
          "x": 24,
          "y": 31,
          "value": 1,
          "name": "cell25 ~ gene32"
        },
        {
          "x": 24,
          "y": 32,
          "value": 1,
          "name": "cell25 ~ gene33"
        },
        {
          "x": 24,
          "y": 33,
          "value": 20,
          "name": "cell25 ~ gene34"
        },
        {
          "x": 24,
          "y": 34,
          "value": 0,
          "name": "cell25 ~ gene35"
        },
        {
          "x": 24,
          "y": 35,
          "value": 0,
          "name": "cell25 ~ gene36"
        },
        {
          "x": 24,
          "y": 36,
          "value": 0,
          "name": "cell25 ~ gene37"
        },
        {
          "x": 24,
          "y": 37,
          "value": 13,
          "name": "cell25 ~ gene38"
        },
        {
          "x": 24,
          "y": 38,
          "value": 4,
          "name": "cell25 ~ gene39"
        },
        {
          "x": 24,
          "y": 39,
          "value": 5,
          "name": "cell25 ~ gene40"
        },
        {
          "x": 24,
          "y": 40,
          "value": 16,
          "name": "cell25 ~ gene41"
        },
        {
          "x": 24,
          "y": 41,
          "value": 8,
          "name": "cell25 ~ gene42"
        },
        {
          "x": 24,
          "y": 42,
          "value": 8,
          "name": "cell25 ~ gene43"
        },
        {
          "x": 24,
          "y": 43,
          "value": 0,
          "name": "cell25 ~ gene44"
        },
        {
          "x": 24,
          "y": 44,
          "value": 10,
          "name": "cell25 ~ gene45"
        },
        {
          "x": 24,
          "y": 45,
          "value": 0,
          "name": "cell25 ~ gene46"
        },
        {
          "x": 24,
          "y": 46,
          "value": 1,
          "name": "cell25 ~ gene47"
        },
        {
          "x": 24,
          "y": 47,
          "value": 15,
          "name": "cell25 ~ gene48"
        },
        {
          "x": 24,
          "y": 48,
          "value": 11,
          "name": "cell25 ~ gene49"
        },
        {
          "x": 24,
          "y": 49,
          "value": 0,
          "name": "cell25 ~ gene50"
        },
        {
          "x": 24,
          "y": 50,
          "value": 0,
          "name": "cell25 ~ gene51"
        },
        {
          "x": 24,
          "y": 51,
          "value": 9,
          "name": "cell25 ~ gene52"
        },
        {
          "x": 24,
          "y": 52,
          "value": 0,
          "name": "cell25 ~ gene53"
        },
        {
          "x": 24,
          "y": 53,
          "value": 2,
          "name": "cell25 ~ gene54"
        },
        {
          "x": 24,
          "y": 54,
          "value": 6,
          "name": "cell25 ~ gene55"
        },
        {
          "x": 24,
          "y": 55,
          "value": 4,
          "name": "cell25 ~ gene56"
        },
        {
          "x": 24,
          "y": 56,
          "value": 0,
          "name": "cell25 ~ gene57"
        },
        {
          "x": 24,
          "y": 57,
          "value": 0,
          "name": "cell25 ~ gene58"
        },
        {
          "x": 24,
          "y": 58,
          "value": 3,
          "name": "cell25 ~ gene59"
        },
        {
          "x": 24,
          "y": 59,
          "value": 0,
          "name": "cell25 ~ gene60"
        },
        {
          "x": 24,
          "y": 60,
          "value": 0,
          "name": "cell25 ~ gene61"
        },
        {
          "x": 24,
          "y": 61,
          "value": 8,
          "name": "cell25 ~ gene62"
        },
        {
          "x": 24,
          "y": 62,
          "value": 8,
          "name": "cell25 ~ gene63"
        },
        {
          "x": 24,
          "y": 63,
          "value": 1,
          "name": "cell25 ~ gene64"
        },
        {
          "x": 24,
          "y": 64,
          "value": 7,
          "name": "cell25 ~ gene65"
        },
        {
          "x": 24,
          "y": 65,
          "value": 0,
          "name": "cell25 ~ gene66"
        },
        {
          "x": 24,
          "y": 66,
          "value": 0,
          "name": "cell25 ~ gene67"
        },
        {
          "x": 24,
          "y": 67,
          "value": 12,
          "name": "cell25 ~ gene68"
        },
        {
          "x": 24,
          "y": 68,
          "value": 0,
          "name": "cell25 ~ gene69"
        },
        {
          "x": 24,
          "y": 69,
          "value": 0,
          "name": "cell25 ~ gene70"
        },
        {
          "x": 24,
          "y": 70,
          "value": 0,
          "name": "cell25 ~ gene71"
        },
        {
          "x": 24,
          "y": 71,
          "value": 0,
          "name": "cell25 ~ gene72"
        },
        {
          "x": 24,
          "y": 72,
          "value": 3,
          "name": "cell25 ~ gene73"
        },
        {
          "x": 24,
          "y": 73,
          "value": 0,
          "name": "cell25 ~ gene74"
        },
        {
          "x": 24,
          "y": 74,
          "value": 15,
          "name": "cell25 ~ gene75"
        },
        {
          "x": 24,
          "y": 75,
          "value": 0,
          "name": "cell25 ~ gene76"
        },
        {
          "x": 24,
          "y": 76,
          "value": 0,
          "name": "cell25 ~ gene77"
        },
        {
          "x": 24,
          "y": 77,
          "value": 16,
          "name": "cell25 ~ gene78"
        },
        {
          "x": 24,
          "y": 78,
          "value": 0,
          "name": "cell25 ~ gene79"
        },
        {
          "x": 24,
          "y": 79,
          "value": 0,
          "name": "cell25 ~ gene80"
        },
        {
          "x": 24,
          "y": 80,
          "value": 18,
          "name": "cell25 ~ gene81"
        },
        {
          "x": 24,
          "y": 81,
          "value": 2,
          "name": "cell25 ~ gene82"
        },
        {
          "x": 24,
          "y": 82,
          "value": 0,
          "name": "cell25 ~ gene83"
        },
        {
          "x": 24,
          "y": 83,
          "value": 0,
          "name": "cell25 ~ gene84"
        },
        {
          "x": 24,
          "y": 84,
          "value": 0,
          "name": "cell25 ~ gene85"
        },
        {
          "x": 24,
          "y": 85,
          "value": 11,
          "name": "cell25 ~ gene86"
        },
        {
          "x": 24,
          "y": 86,
          "value": 0,
          "name": "cell25 ~ gene87"
        },
        {
          "x": 24,
          "y": 87,
          "value": 10,
          "name": "cell25 ~ gene88"
        },
        {
          "x": 24,
          "y": 88,
          "value": 0,
          "name": "cell25 ~ gene89"
        },
        {
          "x": 24,
          "y": 89,
          "value": 13,
          "name": "cell25 ~ gene90"
        },
        {
          "x": 24,
          "y": 90,
          "value": 0,
          "name": "cell25 ~ gene91"
        },
        {
          "x": 24,
          "y": 91,
          "value": 14,
          "name": "cell25 ~ gene92"
        },
        {
          "x": 24,
          "y": 92,
          "value": 0,
          "name": "cell25 ~ gene93"
        },
        {
          "x": 24,
          "y": 93,
          "value": 7,
          "name": "cell25 ~ gene94"
        },
        {
          "x": 24,
          "y": 94,
          "value": 6,
          "name": "cell25 ~ gene95"
        },
        {
          "x": 24,
          "y": 95,
          "value": 23,
          "name": "cell25 ~ gene96"
        },
        {
          "x": 24,
          "y": 96,
          "value": 8,
          "name": "cell25 ~ gene97"
        },
        {
          "x": 24,
          "y": 97,
          "value": 0,
          "name": "cell25 ~ gene98"
        },
        {
          "x": 24,
          "y": 98,
          "value": 0,
          "name": "cell25 ~ gene99"
        },
        {
          "x": 24,
          "y": 99,
          "value": 0,
          "name": "cell25 ~ gene100"
        },
        {
          "x": 25,
          "y": 0,
          "value": 1,
          "name": "cell26 ~ gene1"
        },
        {
          "x": 25,
          "y": 1,
          "value": 0,
          "name": "cell26 ~ gene2"
        },
        {
          "x": 25,
          "y": 2,
          "value": 30,
          "name": "cell26 ~ gene3"
        },
        {
          "x": 25,
          "y": 3,
          "value": 17,
          "name": "cell26 ~ gene4"
        },
        {
          "x": 25,
          "y": 4,
          "value": 16,
          "name": "cell26 ~ gene5"
        },
        {
          "x": 25,
          "y": 5,
          "value": 26,
          "name": "cell26 ~ gene6"
        },
        {
          "x": 25,
          "y": 6,
          "value": 22,
          "name": "cell26 ~ gene7"
        },
        {
          "x": 25,
          "y": 7,
          "value": 2,
          "name": "cell26 ~ gene8"
        },
        {
          "x": 25,
          "y": 8,
          "value": 4,
          "name": "cell26 ~ gene9"
        },
        {
          "x": 25,
          "y": 9,
          "value": 0,
          "name": "cell26 ~ gene10"
        },
        {
          "x": 25,
          "y": 10,
          "value": 29,
          "name": "cell26 ~ gene11"
        },
        {
          "x": 25,
          "y": 11,
          "value": 0,
          "name": "cell26 ~ gene12"
        },
        {
          "x": 25,
          "y": 12,
          "value": 18,
          "name": "cell26 ~ gene13"
        },
        {
          "x": 25,
          "y": 13,
          "value": 23,
          "name": "cell26 ~ gene14"
        },
        {
          "x": 25,
          "y": 14,
          "value": 22,
          "name": "cell26 ~ gene15"
        },
        {
          "x": 25,
          "y": 15,
          "value": 40,
          "name": "cell26 ~ gene16"
        },
        {
          "x": 25,
          "y": 16,
          "value": 8,
          "name": "cell26 ~ gene17"
        },
        {
          "x": 25,
          "y": 17,
          "value": 29,
          "name": "cell26 ~ gene18"
        },
        {
          "x": 25,
          "y": 18,
          "value": 14,
          "name": "cell26 ~ gene19"
        },
        {
          "x": 25,
          "y": 19,
          "value": 22,
          "name": "cell26 ~ gene20"
        },
        {
          "x": 25,
          "y": 20,
          "value": 3,
          "name": "cell26 ~ gene21"
        },
        {
          "x": 25,
          "y": 21,
          "value": 4,
          "name": "cell26 ~ gene22"
        },
        {
          "x": 25,
          "y": 22,
          "value": 0,
          "name": "cell26 ~ gene23"
        },
        {
          "x": 25,
          "y": 23,
          "value": 0,
          "name": "cell26 ~ gene24"
        },
        {
          "x": 25,
          "y": 24,
          "value": 12,
          "name": "cell26 ~ gene25"
        },
        {
          "x": 25,
          "y": 25,
          "value": 4,
          "name": "cell26 ~ gene26"
        },
        {
          "x": 25,
          "y": 26,
          "value": 0,
          "name": "cell26 ~ gene27"
        },
        {
          "x": 25,
          "y": 27,
          "value": 1,
          "name": "cell26 ~ gene28"
        },
        {
          "x": 25,
          "y": 28,
          "value": 6,
          "name": "cell26 ~ gene29"
        },
        {
          "x": 25,
          "y": 29,
          "value": 0,
          "name": "cell26 ~ gene30"
        },
        {
          "x": 25,
          "y": 30,
          "value": 0,
          "name": "cell26 ~ gene31"
        },
        {
          "x": 25,
          "y": 31,
          "value": 16,
          "name": "cell26 ~ gene32"
        },
        {
          "x": 25,
          "y": 32,
          "value": 0,
          "name": "cell26 ~ gene33"
        },
        {
          "x": 25,
          "y": 33,
          "value": 0,
          "name": "cell26 ~ gene34"
        },
        {
          "x": 25,
          "y": 34,
          "value": 0,
          "name": "cell26 ~ gene35"
        },
        {
          "x": 25,
          "y": 35,
          "value": 0,
          "name": "cell26 ~ gene36"
        },
        {
          "x": 25,
          "y": 36,
          "value": 0,
          "name": "cell26 ~ gene37"
        },
        {
          "x": 25,
          "y": 37,
          "value": 11,
          "name": "cell26 ~ gene38"
        },
        {
          "x": 25,
          "y": 38,
          "value": 0,
          "name": "cell26 ~ gene39"
        },
        {
          "x": 25,
          "y": 39,
          "value": 11,
          "name": "cell26 ~ gene40"
        },
        {
          "x": 25,
          "y": 40,
          "value": 27,
          "name": "cell26 ~ gene41"
        },
        {
          "x": 25,
          "y": 41,
          "value": 6,
          "name": "cell26 ~ gene42"
        },
        {
          "x": 25,
          "y": 42,
          "value": 0,
          "name": "cell26 ~ gene43"
        },
        {
          "x": 25,
          "y": 43,
          "value": 0,
          "name": "cell26 ~ gene44"
        },
        {
          "x": 25,
          "y": 44,
          "value": 8,
          "name": "cell26 ~ gene45"
        },
        {
          "x": 25,
          "y": 45,
          "value": 6,
          "name": "cell26 ~ gene46"
        },
        {
          "x": 25,
          "y": 46,
          "value": 6,
          "name": "cell26 ~ gene47"
        },
        {
          "x": 25,
          "y": 47,
          "value": 0,
          "name": "cell26 ~ gene48"
        },
        {
          "x": 25,
          "y": 48,
          "value": 0,
          "name": "cell26 ~ gene49"
        },
        {
          "x": 25,
          "y": 49,
          "value": 0,
          "name": "cell26 ~ gene50"
        },
        {
          "x": 25,
          "y": 50,
          "value": 2,
          "name": "cell26 ~ gene51"
        },
        {
          "x": 25,
          "y": 51,
          "value": 9,
          "name": "cell26 ~ gene52"
        },
        {
          "x": 25,
          "y": 52,
          "value": 3,
          "name": "cell26 ~ gene53"
        },
        {
          "x": 25,
          "y": 53,
          "value": 4,
          "name": "cell26 ~ gene54"
        },
        {
          "x": 25,
          "y": 54,
          "value": 0,
          "name": "cell26 ~ gene55"
        },
        {
          "x": 25,
          "y": 55,
          "value": 0,
          "name": "cell26 ~ gene56"
        },
        {
          "x": 25,
          "y": 56,
          "value": 3,
          "name": "cell26 ~ gene57"
        },
        {
          "x": 25,
          "y": 57,
          "value": 5,
          "name": "cell26 ~ gene58"
        },
        {
          "x": 25,
          "y": 58,
          "value": 0,
          "name": "cell26 ~ gene59"
        },
        {
          "x": 25,
          "y": 59,
          "value": 0,
          "name": "cell26 ~ gene60"
        },
        {
          "x": 25,
          "y": 60,
          "value": 0,
          "name": "cell26 ~ gene61"
        },
        {
          "x": 25,
          "y": 61,
          "value": 0,
          "name": "cell26 ~ gene62"
        },
        {
          "x": 25,
          "y": 62,
          "value": 9,
          "name": "cell26 ~ gene63"
        },
        {
          "x": 25,
          "y": 63,
          "value": 7,
          "name": "cell26 ~ gene64"
        },
        {
          "x": 25,
          "y": 64,
          "value": 8,
          "name": "cell26 ~ gene65"
        },
        {
          "x": 25,
          "y": 65,
          "value": 3,
          "name": "cell26 ~ gene66"
        },
        {
          "x": 25,
          "y": 66,
          "value": 0,
          "name": "cell26 ~ gene67"
        },
        {
          "x": 25,
          "y": 67,
          "value": 0,
          "name": "cell26 ~ gene68"
        },
        {
          "x": 25,
          "y": 68,
          "value": 0,
          "name": "cell26 ~ gene69"
        },
        {
          "x": 25,
          "y": 69,
          "value": 8,
          "name": "cell26 ~ gene70"
        },
        {
          "x": 25,
          "y": 70,
          "value": 0,
          "name": "cell26 ~ gene71"
        },
        {
          "x": 25,
          "y": 71,
          "value": 18,
          "name": "cell26 ~ gene72"
        },
        {
          "x": 25,
          "y": 72,
          "value": 0,
          "name": "cell26 ~ gene73"
        },
        {
          "x": 25,
          "y": 73,
          "value": 4,
          "name": "cell26 ~ gene74"
        },
        {
          "x": 25,
          "y": 74,
          "value": 0,
          "name": "cell26 ~ gene75"
        },
        {
          "x": 25,
          "y": 75,
          "value": 0,
          "name": "cell26 ~ gene76"
        },
        {
          "x": 25,
          "y": 76,
          "value": 0,
          "name": "cell26 ~ gene77"
        },
        {
          "x": 25,
          "y": 77,
          "value": 17,
          "name": "cell26 ~ gene78"
        },
        {
          "x": 25,
          "y": 78,
          "value": 17,
          "name": "cell26 ~ gene79"
        },
        {
          "x": 25,
          "y": 79,
          "value": 13,
          "name": "cell26 ~ gene80"
        },
        {
          "x": 25,
          "y": 80,
          "value": 14,
          "name": "cell26 ~ gene81"
        },
        {
          "x": 25,
          "y": 81,
          "value": 0,
          "name": "cell26 ~ gene82"
        },
        {
          "x": 25,
          "y": 82,
          "value": 3,
          "name": "cell26 ~ gene83"
        },
        {
          "x": 25,
          "y": 83,
          "value": 0,
          "name": "cell26 ~ gene84"
        },
        {
          "x": 25,
          "y": 84,
          "value": 3,
          "name": "cell26 ~ gene85"
        },
        {
          "x": 25,
          "y": 85,
          "value": 10,
          "name": "cell26 ~ gene86"
        },
        {
          "x": 25,
          "y": 86,
          "value": 11,
          "name": "cell26 ~ gene87"
        },
        {
          "x": 25,
          "y": 87,
          "value": 0,
          "name": "cell26 ~ gene88"
        },
        {
          "x": 25,
          "y": 88,
          "value": 0,
          "name": "cell26 ~ gene89"
        },
        {
          "x": 25,
          "y": 89,
          "value": 9,
          "name": "cell26 ~ gene90"
        },
        {
          "x": 25,
          "y": 90,
          "value": 13,
          "name": "cell26 ~ gene91"
        },
        {
          "x": 25,
          "y": 91,
          "value": 23,
          "name": "cell26 ~ gene92"
        },
        {
          "x": 25,
          "y": 92,
          "value": 0,
          "name": "cell26 ~ gene93"
        },
        {
          "x": 25,
          "y": 93,
          "value": 1,
          "name": "cell26 ~ gene94"
        },
        {
          "x": 25,
          "y": 94,
          "value": 11,
          "name": "cell26 ~ gene95"
        },
        {
          "x": 25,
          "y": 95,
          "value": 0,
          "name": "cell26 ~ gene96"
        },
        {
          "x": 25,
          "y": 96,
          "value": 18,
          "name": "cell26 ~ gene97"
        },
        {
          "x": 25,
          "y": 97,
          "value": 0,
          "name": "cell26 ~ gene98"
        },
        {
          "x": 25,
          "y": 98,
          "value": 0,
          "name": "cell26 ~ gene99"
        },
        {
          "x": 25,
          "y": 99,
          "value": 0,
          "name": "cell26 ~ gene100"
        },
        {
          "x": 26,
          "y": 0,
          "value": 7,
          "name": "cell27 ~ gene1"
        },
        {
          "x": 26,
          "y": 1,
          "value": 9,
          "name": "cell27 ~ gene2"
        },
        {
          "x": 26,
          "y": 2,
          "value": 23,
          "name": "cell27 ~ gene3"
        },
        {
          "x": 26,
          "y": 3,
          "value": 32,
          "name": "cell27 ~ gene4"
        },
        {
          "x": 26,
          "y": 4,
          "value": 12,
          "name": "cell27 ~ gene5"
        },
        {
          "x": 26,
          "y": 5,
          "value": 0,
          "name": "cell27 ~ gene6"
        },
        {
          "x": 26,
          "y": 6,
          "value": 30,
          "name": "cell27 ~ gene7"
        },
        {
          "x": 26,
          "y": 7,
          "value": 6,
          "name": "cell27 ~ gene8"
        },
        {
          "x": 26,
          "y": 8,
          "value": 0,
          "name": "cell27 ~ gene9"
        },
        {
          "x": 26,
          "y": 9,
          "value": 0,
          "name": "cell27 ~ gene10"
        },
        {
          "x": 26,
          "y": 10,
          "value": 21,
          "name": "cell27 ~ gene11"
        },
        {
          "x": 26,
          "y": 11,
          "value": 19,
          "name": "cell27 ~ gene12"
        },
        {
          "x": 26,
          "y": 12,
          "value": 21,
          "name": "cell27 ~ gene13"
        },
        {
          "x": 26,
          "y": 13,
          "value": 28,
          "name": "cell27 ~ gene14"
        },
        {
          "x": 26,
          "y": 14,
          "value": 26,
          "name": "cell27 ~ gene15"
        },
        {
          "x": 26,
          "y": 15,
          "value": 39,
          "name": "cell27 ~ gene16"
        },
        {
          "x": 26,
          "y": 16,
          "value": 19,
          "name": "cell27 ~ gene17"
        },
        {
          "x": 26,
          "y": 17,
          "value": 29,
          "name": "cell27 ~ gene18"
        },
        {
          "x": 26,
          "y": 18,
          "value": 19,
          "name": "cell27 ~ gene19"
        },
        {
          "x": 26,
          "y": 19,
          "value": 13,
          "name": "cell27 ~ gene20"
        },
        {
          "x": 26,
          "y": 20,
          "value": 10,
          "name": "cell27 ~ gene21"
        },
        {
          "x": 26,
          "y": 21,
          "value": 3,
          "name": "cell27 ~ gene22"
        },
        {
          "x": 26,
          "y": 22,
          "value": 0,
          "name": "cell27 ~ gene23"
        },
        {
          "x": 26,
          "y": 23,
          "value": 0,
          "name": "cell27 ~ gene24"
        },
        {
          "x": 26,
          "y": 24,
          "value": 0,
          "name": "cell27 ~ gene25"
        },
        {
          "x": 26,
          "y": 25,
          "value": 0,
          "name": "cell27 ~ gene26"
        },
        {
          "x": 26,
          "y": 26,
          "value": 0,
          "name": "cell27 ~ gene27"
        },
        {
          "x": 26,
          "y": 27,
          "value": 18,
          "name": "cell27 ~ gene28"
        },
        {
          "x": 26,
          "y": 28,
          "value": 0,
          "name": "cell27 ~ gene29"
        },
        {
          "x": 26,
          "y": 29,
          "value": 0,
          "name": "cell27 ~ gene30"
        },
        {
          "x": 26,
          "y": 30,
          "value": 6,
          "name": "cell27 ~ gene31"
        },
        {
          "x": 26,
          "y": 31,
          "value": 0,
          "name": "cell27 ~ gene32"
        },
        {
          "x": 26,
          "y": 32,
          "value": 0,
          "name": "cell27 ~ gene33"
        },
        {
          "x": 26,
          "y": 33,
          "value": 3,
          "name": "cell27 ~ gene34"
        },
        {
          "x": 26,
          "y": 34,
          "value": 9,
          "name": "cell27 ~ gene35"
        },
        {
          "x": 26,
          "y": 35,
          "value": 9,
          "name": "cell27 ~ gene36"
        },
        {
          "x": 26,
          "y": 36,
          "value": 0,
          "name": "cell27 ~ gene37"
        },
        {
          "x": 26,
          "y": 37,
          "value": 0,
          "name": "cell27 ~ gene38"
        },
        {
          "x": 26,
          "y": 38,
          "value": 0,
          "name": "cell27 ~ gene39"
        },
        {
          "x": 26,
          "y": 39,
          "value": 0,
          "name": "cell27 ~ gene40"
        },
        {
          "x": 26,
          "y": 40,
          "value": 3,
          "name": "cell27 ~ gene41"
        },
        {
          "x": 26,
          "y": 41,
          "value": 0,
          "name": "cell27 ~ gene42"
        },
        {
          "x": 26,
          "y": 42,
          "value": 2,
          "name": "cell27 ~ gene43"
        },
        {
          "x": 26,
          "y": 43,
          "value": 4,
          "name": "cell27 ~ gene44"
        },
        {
          "x": 26,
          "y": 44,
          "value": 0,
          "name": "cell27 ~ gene45"
        },
        {
          "x": 26,
          "y": 45,
          "value": 0,
          "name": "cell27 ~ gene46"
        },
        {
          "x": 26,
          "y": 46,
          "value": 1,
          "name": "cell27 ~ gene47"
        },
        {
          "x": 26,
          "y": 47,
          "value": 7,
          "name": "cell27 ~ gene48"
        },
        {
          "x": 26,
          "y": 48,
          "value": 8,
          "name": "cell27 ~ gene49"
        },
        {
          "x": 26,
          "y": 49,
          "value": 11,
          "name": "cell27 ~ gene50"
        },
        {
          "x": 26,
          "y": 50,
          "value": 6,
          "name": "cell27 ~ gene51"
        },
        {
          "x": 26,
          "y": 51,
          "value": 0,
          "name": "cell27 ~ gene52"
        },
        {
          "x": 26,
          "y": 52,
          "value": 1,
          "name": "cell27 ~ gene53"
        },
        {
          "x": 26,
          "y": 53,
          "value": 0,
          "name": "cell27 ~ gene54"
        },
        {
          "x": 26,
          "y": 54,
          "value": 0,
          "name": "cell27 ~ gene55"
        },
        {
          "x": 26,
          "y": 55,
          "value": 0,
          "name": "cell27 ~ gene56"
        },
        {
          "x": 26,
          "y": 56,
          "value": 2,
          "name": "cell27 ~ gene57"
        },
        {
          "x": 26,
          "y": 57,
          "value": 0,
          "name": "cell27 ~ gene58"
        },
        {
          "x": 26,
          "y": 58,
          "value": 5,
          "name": "cell27 ~ gene59"
        },
        {
          "x": 26,
          "y": 59,
          "value": 6,
          "name": "cell27 ~ gene60"
        },
        {
          "x": 26,
          "y": 60,
          "value": 22,
          "name": "cell27 ~ gene61"
        },
        {
          "x": 26,
          "y": 61,
          "value": 7,
          "name": "cell27 ~ gene62"
        },
        {
          "x": 26,
          "y": 62,
          "value": 1,
          "name": "cell27 ~ gene63"
        },
        {
          "x": 26,
          "y": 63,
          "value": 0,
          "name": "cell27 ~ gene64"
        },
        {
          "x": 26,
          "y": 64,
          "value": 0,
          "name": "cell27 ~ gene65"
        },
        {
          "x": 26,
          "y": 65,
          "value": 0,
          "name": "cell27 ~ gene66"
        },
        {
          "x": 26,
          "y": 66,
          "value": 0,
          "name": "cell27 ~ gene67"
        },
        {
          "x": 26,
          "y": 67,
          "value": 0,
          "name": "cell27 ~ gene68"
        },
        {
          "x": 26,
          "y": 68,
          "value": 0,
          "name": "cell27 ~ gene69"
        },
        {
          "x": 26,
          "y": 69,
          "value": 5,
          "name": "cell27 ~ gene70"
        },
        {
          "x": 26,
          "y": 70,
          "value": 3,
          "name": "cell27 ~ gene71"
        },
        {
          "x": 26,
          "y": 71,
          "value": 4,
          "name": "cell27 ~ gene72"
        },
        {
          "x": 26,
          "y": 72,
          "value": 0,
          "name": "cell27 ~ gene73"
        },
        {
          "x": 26,
          "y": 73,
          "value": 5,
          "name": "cell27 ~ gene74"
        },
        {
          "x": 26,
          "y": 74,
          "value": 0,
          "name": "cell27 ~ gene75"
        },
        {
          "x": 26,
          "y": 75,
          "value": 0,
          "name": "cell27 ~ gene76"
        },
        {
          "x": 26,
          "y": 76,
          "value": 0,
          "name": "cell27 ~ gene77"
        },
        {
          "x": 26,
          "y": 77,
          "value": 8,
          "name": "cell27 ~ gene78"
        },
        {
          "x": 26,
          "y": 78,
          "value": 0,
          "name": "cell27 ~ gene79"
        },
        {
          "x": 26,
          "y": 79,
          "value": 0,
          "name": "cell27 ~ gene80"
        },
        {
          "x": 26,
          "y": 80,
          "value": 0,
          "name": "cell27 ~ gene81"
        },
        {
          "x": 26,
          "y": 81,
          "value": 16,
          "name": "cell27 ~ gene82"
        },
        {
          "x": 26,
          "y": 82,
          "value": 1,
          "name": "cell27 ~ gene83"
        },
        {
          "x": 26,
          "y": 83,
          "value": 9,
          "name": "cell27 ~ gene84"
        },
        {
          "x": 26,
          "y": 84,
          "value": 3,
          "name": "cell27 ~ gene85"
        },
        {
          "x": 26,
          "y": 85,
          "value": 0,
          "name": "cell27 ~ gene86"
        },
        {
          "x": 26,
          "y": 86,
          "value": 7,
          "name": "cell27 ~ gene87"
        },
        {
          "x": 26,
          "y": 87,
          "value": 10,
          "name": "cell27 ~ gene88"
        },
        {
          "x": 26,
          "y": 88,
          "value": 13,
          "name": "cell27 ~ gene89"
        },
        {
          "x": 26,
          "y": 89,
          "value": 4,
          "name": "cell27 ~ gene90"
        },
        {
          "x": 26,
          "y": 90,
          "value": 0,
          "name": "cell27 ~ gene91"
        },
        {
          "x": 26,
          "y": 91,
          "value": 0,
          "name": "cell27 ~ gene92"
        },
        {
          "x": 26,
          "y": 92,
          "value": 2,
          "name": "cell27 ~ gene93"
        },
        {
          "x": 26,
          "y": 93,
          "value": 8,
          "name": "cell27 ~ gene94"
        },
        {
          "x": 26,
          "y": 94,
          "value": 13,
          "name": "cell27 ~ gene95"
        },
        {
          "x": 26,
          "y": 95,
          "value": 10,
          "name": "cell27 ~ gene96"
        },
        {
          "x": 26,
          "y": 96,
          "value": 0,
          "name": "cell27 ~ gene97"
        },
        {
          "x": 26,
          "y": 97,
          "value": 0,
          "name": "cell27 ~ gene98"
        },
        {
          "x": 26,
          "y": 98,
          "value": 7,
          "name": "cell27 ~ gene99"
        },
        {
          "x": 26,
          "y": 99,
          "value": 0,
          "name": "cell27 ~ gene100"
        },
        {
          "x": 27,
          "y": 0,
          "value": 0,
          "name": "cell28 ~ gene1"
        },
        {
          "x": 27,
          "y": 1,
          "value": 4,
          "name": "cell28 ~ gene2"
        },
        {
          "x": 27,
          "y": 2,
          "value": 22,
          "name": "cell28 ~ gene3"
        },
        {
          "x": 27,
          "y": 3,
          "value": 29,
          "name": "cell28 ~ gene4"
        },
        {
          "x": 27,
          "y": 4,
          "value": 18,
          "name": "cell28 ~ gene5"
        },
        {
          "x": 27,
          "y": 5,
          "value": 25,
          "name": "cell28 ~ gene6"
        },
        {
          "x": 27,
          "y": 6,
          "value": 31,
          "name": "cell28 ~ gene7"
        },
        {
          "x": 27,
          "y": 7,
          "value": 2,
          "name": "cell28 ~ gene8"
        },
        {
          "x": 27,
          "y": 8,
          "value": 0,
          "name": "cell28 ~ gene9"
        },
        {
          "x": 27,
          "y": 9,
          "value": 3,
          "name": "cell28 ~ gene10"
        },
        {
          "x": 27,
          "y": 10,
          "value": 19,
          "name": "cell28 ~ gene11"
        },
        {
          "x": 27,
          "y": 11,
          "value": 1,
          "name": "cell28 ~ gene12"
        },
        {
          "x": 27,
          "y": 12,
          "value": 22,
          "name": "cell28 ~ gene13"
        },
        {
          "x": 27,
          "y": 13,
          "value": 6,
          "name": "cell28 ~ gene14"
        },
        {
          "x": 27,
          "y": 14,
          "value": 49,
          "name": "cell28 ~ gene15"
        },
        {
          "x": 27,
          "y": 15,
          "value": 30,
          "name": "cell28 ~ gene16"
        },
        {
          "x": 27,
          "y": 16,
          "value": 4,
          "name": "cell28 ~ gene17"
        },
        {
          "x": 27,
          "y": 17,
          "value": 26,
          "name": "cell28 ~ gene18"
        },
        {
          "x": 27,
          "y": 18,
          "value": 24,
          "name": "cell28 ~ gene19"
        },
        {
          "x": 27,
          "y": 19,
          "value": 0,
          "name": "cell28 ~ gene20"
        },
        {
          "x": 27,
          "y": 20,
          "value": 0,
          "name": "cell28 ~ gene21"
        },
        {
          "x": 27,
          "y": 21,
          "value": 7,
          "name": "cell28 ~ gene22"
        },
        {
          "x": 27,
          "y": 22,
          "value": 0,
          "name": "cell28 ~ gene23"
        },
        {
          "x": 27,
          "y": 23,
          "value": 0,
          "name": "cell28 ~ gene24"
        },
        {
          "x": 27,
          "y": 24,
          "value": 17,
          "name": "cell28 ~ gene25"
        },
        {
          "x": 27,
          "y": 25,
          "value": 2,
          "name": "cell28 ~ gene26"
        },
        {
          "x": 27,
          "y": 26,
          "value": 13,
          "name": "cell28 ~ gene27"
        },
        {
          "x": 27,
          "y": 27,
          "value": 0,
          "name": "cell28 ~ gene28"
        },
        {
          "x": 27,
          "y": 28,
          "value": 3,
          "name": "cell28 ~ gene29"
        },
        {
          "x": 27,
          "y": 29,
          "value": 0,
          "name": "cell28 ~ gene30"
        },
        {
          "x": 27,
          "y": 30,
          "value": 6,
          "name": "cell28 ~ gene31"
        },
        {
          "x": 27,
          "y": 31,
          "value": 15,
          "name": "cell28 ~ gene32"
        },
        {
          "x": 27,
          "y": 32,
          "value": 0,
          "name": "cell28 ~ gene33"
        },
        {
          "x": 27,
          "y": 33,
          "value": 13,
          "name": "cell28 ~ gene34"
        },
        {
          "x": 27,
          "y": 34,
          "value": 0,
          "name": "cell28 ~ gene35"
        },
        {
          "x": 27,
          "y": 35,
          "value": 15,
          "name": "cell28 ~ gene36"
        },
        {
          "x": 27,
          "y": 36,
          "value": 13,
          "name": "cell28 ~ gene37"
        },
        {
          "x": 27,
          "y": 37,
          "value": 2,
          "name": "cell28 ~ gene38"
        },
        {
          "x": 27,
          "y": 38,
          "value": 7,
          "name": "cell28 ~ gene39"
        },
        {
          "x": 27,
          "y": 39,
          "value": 3,
          "name": "cell28 ~ gene40"
        },
        {
          "x": 27,
          "y": 40,
          "value": 3,
          "name": "cell28 ~ gene41"
        },
        {
          "x": 27,
          "y": 41,
          "value": 2,
          "name": "cell28 ~ gene42"
        },
        {
          "x": 27,
          "y": 42,
          "value": 8,
          "name": "cell28 ~ gene43"
        },
        {
          "x": 27,
          "y": 43,
          "value": 0,
          "name": "cell28 ~ gene44"
        },
        {
          "x": 27,
          "y": 44,
          "value": 0,
          "name": "cell28 ~ gene45"
        },
        {
          "x": 27,
          "y": 45,
          "value": 5,
          "name": "cell28 ~ gene46"
        },
        {
          "x": 27,
          "y": 46,
          "value": 0,
          "name": "cell28 ~ gene47"
        },
        {
          "x": 27,
          "y": 47,
          "value": 0,
          "name": "cell28 ~ gene48"
        },
        {
          "x": 27,
          "y": 48,
          "value": 8,
          "name": "cell28 ~ gene49"
        },
        {
          "x": 27,
          "y": 49,
          "value": 14,
          "name": "cell28 ~ gene50"
        },
        {
          "x": 27,
          "y": 50,
          "value": 12,
          "name": "cell28 ~ gene51"
        },
        {
          "x": 27,
          "y": 51,
          "value": 15,
          "name": "cell28 ~ gene52"
        },
        {
          "x": 27,
          "y": 52,
          "value": 3,
          "name": "cell28 ~ gene53"
        },
        {
          "x": 27,
          "y": 53,
          "value": 0,
          "name": "cell28 ~ gene54"
        },
        {
          "x": 27,
          "y": 54,
          "value": 0,
          "name": "cell28 ~ gene55"
        },
        {
          "x": 27,
          "y": 55,
          "value": 6,
          "name": "cell28 ~ gene56"
        },
        {
          "x": 27,
          "y": 56,
          "value": 0,
          "name": "cell28 ~ gene57"
        },
        {
          "x": 27,
          "y": 57,
          "value": 0,
          "name": "cell28 ~ gene58"
        },
        {
          "x": 27,
          "y": 58,
          "value": 0,
          "name": "cell28 ~ gene59"
        },
        {
          "x": 27,
          "y": 59,
          "value": 8,
          "name": "cell28 ~ gene60"
        },
        {
          "x": 27,
          "y": 60,
          "value": 1,
          "name": "cell28 ~ gene61"
        },
        {
          "x": 27,
          "y": 61,
          "value": 0,
          "name": "cell28 ~ gene62"
        },
        {
          "x": 27,
          "y": 62,
          "value": 17,
          "name": "cell28 ~ gene63"
        },
        {
          "x": 27,
          "y": 63,
          "value": 5,
          "name": "cell28 ~ gene64"
        },
        {
          "x": 27,
          "y": 64,
          "value": 0,
          "name": "cell28 ~ gene65"
        },
        {
          "x": 27,
          "y": 65,
          "value": 2,
          "name": "cell28 ~ gene66"
        },
        {
          "x": 27,
          "y": 66,
          "value": 10,
          "name": "cell28 ~ gene67"
        },
        {
          "x": 27,
          "y": 67,
          "value": 11,
          "name": "cell28 ~ gene68"
        },
        {
          "x": 27,
          "y": 68,
          "value": 4,
          "name": "cell28 ~ gene69"
        },
        {
          "x": 27,
          "y": 69,
          "value": 1,
          "name": "cell28 ~ gene70"
        },
        {
          "x": 27,
          "y": 70,
          "value": 10,
          "name": "cell28 ~ gene71"
        },
        {
          "x": 27,
          "y": 71,
          "value": 0,
          "name": "cell28 ~ gene72"
        },
        {
          "x": 27,
          "y": 72,
          "value": 9,
          "name": "cell28 ~ gene73"
        },
        {
          "x": 27,
          "y": 73,
          "value": 8,
          "name": "cell28 ~ gene74"
        },
        {
          "x": 27,
          "y": 74,
          "value": 0,
          "name": "cell28 ~ gene75"
        },
        {
          "x": 27,
          "y": 75,
          "value": 0,
          "name": "cell28 ~ gene76"
        },
        {
          "x": 27,
          "y": 76,
          "value": 7,
          "name": "cell28 ~ gene77"
        },
        {
          "x": 27,
          "y": 77,
          "value": 6,
          "name": "cell28 ~ gene78"
        },
        {
          "x": 27,
          "y": 78,
          "value": 0,
          "name": "cell28 ~ gene79"
        },
        {
          "x": 27,
          "y": 79,
          "value": 0,
          "name": "cell28 ~ gene80"
        },
        {
          "x": 27,
          "y": 80,
          "value": 4,
          "name": "cell28 ~ gene81"
        },
        {
          "x": 27,
          "y": 81,
          "value": 7,
          "name": "cell28 ~ gene82"
        },
        {
          "x": 27,
          "y": 82,
          "value": 9,
          "name": "cell28 ~ gene83"
        },
        {
          "x": 27,
          "y": 83,
          "value": 5,
          "name": "cell28 ~ gene84"
        },
        {
          "x": 27,
          "y": 84,
          "value": 3,
          "name": "cell28 ~ gene85"
        },
        {
          "x": 27,
          "y": 85,
          "value": 10,
          "name": "cell28 ~ gene86"
        },
        {
          "x": 27,
          "y": 86,
          "value": 7,
          "name": "cell28 ~ gene87"
        },
        {
          "x": 27,
          "y": 87,
          "value": 3,
          "name": "cell28 ~ gene88"
        },
        {
          "x": 27,
          "y": 88,
          "value": 2,
          "name": "cell28 ~ gene89"
        },
        {
          "x": 27,
          "y": 89,
          "value": 14,
          "name": "cell28 ~ gene90"
        },
        {
          "x": 27,
          "y": 90,
          "value": 29,
          "name": "cell28 ~ gene91"
        },
        {
          "x": 27,
          "y": 91,
          "value": 1,
          "name": "cell28 ~ gene92"
        },
        {
          "x": 27,
          "y": 92,
          "value": 0,
          "name": "cell28 ~ gene93"
        },
        {
          "x": 27,
          "y": 93,
          "value": 0,
          "name": "cell28 ~ gene94"
        },
        {
          "x": 27,
          "y": 94,
          "value": 0,
          "name": "cell28 ~ gene95"
        },
        {
          "x": 27,
          "y": 95,
          "value": 5,
          "name": "cell28 ~ gene96"
        },
        {
          "x": 27,
          "y": 96,
          "value": 0,
          "name": "cell28 ~ gene97"
        },
        {
          "x": 27,
          "y": 97,
          "value": 25,
          "name": "cell28 ~ gene98"
        },
        {
          "x": 27,
          "y": 98,
          "value": 0,
          "name": "cell28 ~ gene99"
        },
        {
          "x": 27,
          "y": 99,
          "value": 0,
          "name": "cell28 ~ gene100"
        },
        {
          "x": 28,
          "y": 0,
          "value": 3,
          "name": "cell29 ~ gene1"
        },
        {
          "x": 28,
          "y": 1,
          "value": 0,
          "name": "cell29 ~ gene2"
        },
        {
          "x": 28,
          "y": 2,
          "value": 33,
          "name": "cell29 ~ gene3"
        },
        {
          "x": 28,
          "y": 3,
          "value": 5,
          "name": "cell29 ~ gene4"
        },
        {
          "x": 28,
          "y": 4,
          "value": 23,
          "name": "cell29 ~ gene5"
        },
        {
          "x": 28,
          "y": 5,
          "value": 25,
          "name": "cell29 ~ gene6"
        },
        {
          "x": 28,
          "y": 6,
          "value": 12,
          "name": "cell29 ~ gene7"
        },
        {
          "x": 28,
          "y": 7,
          "value": 22,
          "name": "cell29 ~ gene8"
        },
        {
          "x": 28,
          "y": 8,
          "value": 0,
          "name": "cell29 ~ gene9"
        },
        {
          "x": 28,
          "y": 9,
          "value": 0,
          "name": "cell29 ~ gene10"
        },
        {
          "x": 28,
          "y": 10,
          "value": 12,
          "name": "cell29 ~ gene11"
        },
        {
          "x": 28,
          "y": 11,
          "value": 0,
          "name": "cell29 ~ gene12"
        },
        {
          "x": 28,
          "y": 12,
          "value": 15,
          "name": "cell29 ~ gene13"
        },
        {
          "x": 28,
          "y": 13,
          "value": 6,
          "name": "cell29 ~ gene14"
        },
        {
          "x": 28,
          "y": 14,
          "value": 20,
          "name": "cell29 ~ gene15"
        },
        {
          "x": 28,
          "y": 15,
          "value": 32,
          "name": "cell29 ~ gene16"
        },
        {
          "x": 28,
          "y": 16,
          "value": 27,
          "name": "cell29 ~ gene17"
        },
        {
          "x": 28,
          "y": 17,
          "value": 26,
          "name": "cell29 ~ gene18"
        },
        {
          "x": 28,
          "y": 18,
          "value": 42,
          "name": "cell29 ~ gene19"
        },
        {
          "x": 28,
          "y": 19,
          "value": 3,
          "name": "cell29 ~ gene20"
        },
        {
          "x": 28,
          "y": 20,
          "value": 0,
          "name": "cell29 ~ gene21"
        },
        {
          "x": 28,
          "y": 21,
          "value": 5,
          "name": "cell29 ~ gene22"
        },
        {
          "x": 28,
          "y": 22,
          "value": 0,
          "name": "cell29 ~ gene23"
        },
        {
          "x": 28,
          "y": 23,
          "value": 0,
          "name": "cell29 ~ gene24"
        },
        {
          "x": 28,
          "y": 24,
          "value": 0,
          "name": "cell29 ~ gene25"
        },
        {
          "x": 28,
          "y": 25,
          "value": 16,
          "name": "cell29 ~ gene26"
        },
        {
          "x": 28,
          "y": 26,
          "value": 14,
          "name": "cell29 ~ gene27"
        },
        {
          "x": 28,
          "y": 27,
          "value": 8,
          "name": "cell29 ~ gene28"
        },
        {
          "x": 28,
          "y": 28,
          "value": 7,
          "name": "cell29 ~ gene29"
        },
        {
          "x": 28,
          "y": 29,
          "value": 0,
          "name": "cell29 ~ gene30"
        },
        {
          "x": 28,
          "y": 30,
          "value": 13,
          "name": "cell29 ~ gene31"
        },
        {
          "x": 28,
          "y": 31,
          "value": 4,
          "name": "cell29 ~ gene32"
        },
        {
          "x": 28,
          "y": 32,
          "value": 0,
          "name": "cell29 ~ gene33"
        },
        {
          "x": 28,
          "y": 33,
          "value": 9,
          "name": "cell29 ~ gene34"
        },
        {
          "x": 28,
          "y": 34,
          "value": 0,
          "name": "cell29 ~ gene35"
        },
        {
          "x": 28,
          "y": 35,
          "value": 0,
          "name": "cell29 ~ gene36"
        },
        {
          "x": 28,
          "y": 36,
          "value": 0,
          "name": "cell29 ~ gene37"
        },
        {
          "x": 28,
          "y": 37,
          "value": 0,
          "name": "cell29 ~ gene38"
        },
        {
          "x": 28,
          "y": 38,
          "value": 0,
          "name": "cell29 ~ gene39"
        },
        {
          "x": 28,
          "y": 39,
          "value": 6,
          "name": "cell29 ~ gene40"
        },
        {
          "x": 28,
          "y": 40,
          "value": 5,
          "name": "cell29 ~ gene41"
        },
        {
          "x": 28,
          "y": 41,
          "value": 0,
          "name": "cell29 ~ gene42"
        },
        {
          "x": 28,
          "y": 42,
          "value": 13,
          "name": "cell29 ~ gene43"
        },
        {
          "x": 28,
          "y": 43,
          "value": 1,
          "name": "cell29 ~ gene44"
        },
        {
          "x": 28,
          "y": 44,
          "value": 2,
          "name": "cell29 ~ gene45"
        },
        {
          "x": 28,
          "y": 45,
          "value": 6,
          "name": "cell29 ~ gene46"
        },
        {
          "x": 28,
          "y": 46,
          "value": 3,
          "name": "cell29 ~ gene47"
        },
        {
          "x": 28,
          "y": 47,
          "value": 3,
          "name": "cell29 ~ gene48"
        },
        {
          "x": 28,
          "y": 48,
          "value": 0,
          "name": "cell29 ~ gene49"
        },
        {
          "x": 28,
          "y": 49,
          "value": 2,
          "name": "cell29 ~ gene50"
        },
        {
          "x": 28,
          "y": 50,
          "value": 7,
          "name": "cell29 ~ gene51"
        },
        {
          "x": 28,
          "y": 51,
          "value": 5,
          "name": "cell29 ~ gene52"
        },
        {
          "x": 28,
          "y": 52,
          "value": 0,
          "name": "cell29 ~ gene53"
        },
        {
          "x": 28,
          "y": 53,
          "value": 9,
          "name": "cell29 ~ gene54"
        },
        {
          "x": 28,
          "y": 54,
          "value": 0,
          "name": "cell29 ~ gene55"
        },
        {
          "x": 28,
          "y": 55,
          "value": 0,
          "name": "cell29 ~ gene56"
        },
        {
          "x": 28,
          "y": 56,
          "value": 0,
          "name": "cell29 ~ gene57"
        },
        {
          "x": 28,
          "y": 57,
          "value": 0,
          "name": "cell29 ~ gene58"
        },
        {
          "x": 28,
          "y": 58,
          "value": 5,
          "name": "cell29 ~ gene59"
        },
        {
          "x": 28,
          "y": 59,
          "value": 18,
          "name": "cell29 ~ gene60"
        },
        {
          "x": 28,
          "y": 60,
          "value": 2,
          "name": "cell29 ~ gene61"
        },
        {
          "x": 28,
          "y": 61,
          "value": 6,
          "name": "cell29 ~ gene62"
        },
        {
          "x": 28,
          "y": 62,
          "value": 0,
          "name": "cell29 ~ gene63"
        },
        {
          "x": 28,
          "y": 63,
          "value": 0,
          "name": "cell29 ~ gene64"
        },
        {
          "x": 28,
          "y": 64,
          "value": 0,
          "name": "cell29 ~ gene65"
        },
        {
          "x": 28,
          "y": 65,
          "value": 0,
          "name": "cell29 ~ gene66"
        },
        {
          "x": 28,
          "y": 66,
          "value": 5,
          "name": "cell29 ~ gene67"
        },
        {
          "x": 28,
          "y": 67,
          "value": 7,
          "name": "cell29 ~ gene68"
        },
        {
          "x": 28,
          "y": 68,
          "value": 1,
          "name": "cell29 ~ gene69"
        },
        {
          "x": 28,
          "y": 69,
          "value": 0,
          "name": "cell29 ~ gene70"
        },
        {
          "x": 28,
          "y": 70,
          "value": 5,
          "name": "cell29 ~ gene71"
        },
        {
          "x": 28,
          "y": 71,
          "value": 0,
          "name": "cell29 ~ gene72"
        },
        {
          "x": 28,
          "y": 72,
          "value": 1,
          "name": "cell29 ~ gene73"
        },
        {
          "x": 28,
          "y": 73,
          "value": 9,
          "name": "cell29 ~ gene74"
        },
        {
          "x": 28,
          "y": 74,
          "value": 0,
          "name": "cell29 ~ gene75"
        },
        {
          "x": 28,
          "y": 75,
          "value": 0,
          "name": "cell29 ~ gene76"
        },
        {
          "x": 28,
          "y": 76,
          "value": 0,
          "name": "cell29 ~ gene77"
        },
        {
          "x": 28,
          "y": 77,
          "value": 0,
          "name": "cell29 ~ gene78"
        },
        {
          "x": 28,
          "y": 78,
          "value": 0,
          "name": "cell29 ~ gene79"
        },
        {
          "x": 28,
          "y": 79,
          "value": 2,
          "name": "cell29 ~ gene80"
        },
        {
          "x": 28,
          "y": 80,
          "value": 13,
          "name": "cell29 ~ gene81"
        },
        {
          "x": 28,
          "y": 81,
          "value": 0,
          "name": "cell29 ~ gene82"
        },
        {
          "x": 28,
          "y": 82,
          "value": 0,
          "name": "cell29 ~ gene83"
        },
        {
          "x": 28,
          "y": 83,
          "value": 11,
          "name": "cell29 ~ gene84"
        },
        {
          "x": 28,
          "y": 84,
          "value": 5,
          "name": "cell29 ~ gene85"
        },
        {
          "x": 28,
          "y": 85,
          "value": 0,
          "name": "cell29 ~ gene86"
        },
        {
          "x": 28,
          "y": 86,
          "value": 1,
          "name": "cell29 ~ gene87"
        },
        {
          "x": 28,
          "y": 87,
          "value": 25,
          "name": "cell29 ~ gene88"
        },
        {
          "x": 28,
          "y": 88,
          "value": 4,
          "name": "cell29 ~ gene89"
        },
        {
          "x": 28,
          "y": 89,
          "value": 1,
          "name": "cell29 ~ gene90"
        },
        {
          "x": 28,
          "y": 90,
          "value": 4,
          "name": "cell29 ~ gene91"
        },
        {
          "x": 28,
          "y": 91,
          "value": 20,
          "name": "cell29 ~ gene92"
        },
        {
          "x": 28,
          "y": 92,
          "value": 0,
          "name": "cell29 ~ gene93"
        },
        {
          "x": 28,
          "y": 93,
          "value": 5,
          "name": "cell29 ~ gene94"
        },
        {
          "x": 28,
          "y": 94,
          "value": 0,
          "name": "cell29 ~ gene95"
        },
        {
          "x": 28,
          "y": 95,
          "value": 1,
          "name": "cell29 ~ gene96"
        },
        {
          "x": 28,
          "y": 96,
          "value": 0,
          "name": "cell29 ~ gene97"
        },
        {
          "x": 28,
          "y": 97,
          "value": 11,
          "name": "cell29 ~ gene98"
        },
        {
          "x": 28,
          "y": 98,
          "value": 7,
          "name": "cell29 ~ gene99"
        },
        {
          "x": 28,
          "y": 99,
          "value": 9,
          "name": "cell29 ~ gene100"
        },
        {
          "x": 29,
          "y": 0,
          "value": 19,
          "name": "cell30 ~ gene1"
        },
        {
          "x": 29,
          "y": 1,
          "value": 2,
          "name": "cell30 ~ gene2"
        },
        {
          "x": 29,
          "y": 2,
          "value": 20,
          "name": "cell30 ~ gene3"
        },
        {
          "x": 29,
          "y": 3,
          "value": 26,
          "name": "cell30 ~ gene4"
        },
        {
          "x": 29,
          "y": 4,
          "value": 10,
          "name": "cell30 ~ gene5"
        },
        {
          "x": 29,
          "y": 5,
          "value": 0,
          "name": "cell30 ~ gene6"
        },
        {
          "x": 29,
          "y": 6,
          "value": 18,
          "name": "cell30 ~ gene7"
        },
        {
          "x": 29,
          "y": 7,
          "value": 2,
          "name": "cell30 ~ gene8"
        },
        {
          "x": 29,
          "y": 8,
          "value": 8,
          "name": "cell30 ~ gene9"
        },
        {
          "x": 29,
          "y": 9,
          "value": 0,
          "name": "cell30 ~ gene10"
        },
        {
          "x": 29,
          "y": 10,
          "value": 1,
          "name": "cell30 ~ gene11"
        },
        {
          "x": 29,
          "y": 11,
          "value": 0,
          "name": "cell30 ~ gene12"
        },
        {
          "x": 29,
          "y": 12,
          "value": 26,
          "name": "cell30 ~ gene13"
        },
        {
          "x": 29,
          "y": 13,
          "value": 11,
          "name": "cell30 ~ gene14"
        },
        {
          "x": 29,
          "y": 14,
          "value": 22,
          "name": "cell30 ~ gene15"
        },
        {
          "x": 29,
          "y": 15,
          "value": 37,
          "name": "cell30 ~ gene16"
        },
        {
          "x": 29,
          "y": 16,
          "value": 17,
          "name": "cell30 ~ gene17"
        },
        {
          "x": 29,
          "y": 17,
          "value": 33,
          "name": "cell30 ~ gene18"
        },
        {
          "x": 29,
          "y": 18,
          "value": 14,
          "name": "cell30 ~ gene19"
        },
        {
          "x": 29,
          "y": 19,
          "value": 12,
          "name": "cell30 ~ gene20"
        },
        {
          "x": 29,
          "y": 20,
          "value": 0,
          "name": "cell30 ~ gene21"
        },
        {
          "x": 29,
          "y": 21,
          "value": 0,
          "name": "cell30 ~ gene22"
        },
        {
          "x": 29,
          "y": 22,
          "value": 0,
          "name": "cell30 ~ gene23"
        },
        {
          "x": 29,
          "y": 23,
          "value": 11,
          "name": "cell30 ~ gene24"
        },
        {
          "x": 29,
          "y": 24,
          "value": 0,
          "name": "cell30 ~ gene25"
        },
        {
          "x": 29,
          "y": 25,
          "value": 0,
          "name": "cell30 ~ gene26"
        },
        {
          "x": 29,
          "y": 26,
          "value": 0,
          "name": "cell30 ~ gene27"
        },
        {
          "x": 29,
          "y": 27,
          "value": 0,
          "name": "cell30 ~ gene28"
        },
        {
          "x": 29,
          "y": 28,
          "value": 3,
          "name": "cell30 ~ gene29"
        },
        {
          "x": 29,
          "y": 29,
          "value": 0,
          "name": "cell30 ~ gene30"
        },
        {
          "x": 29,
          "y": 30,
          "value": 0,
          "name": "cell30 ~ gene31"
        },
        {
          "x": 29,
          "y": 31,
          "value": 9,
          "name": "cell30 ~ gene32"
        },
        {
          "x": 29,
          "y": 32,
          "value": 0,
          "name": "cell30 ~ gene33"
        },
        {
          "x": 29,
          "y": 33,
          "value": 0,
          "name": "cell30 ~ gene34"
        },
        {
          "x": 29,
          "y": 34,
          "value": 6,
          "name": "cell30 ~ gene35"
        },
        {
          "x": 29,
          "y": 35,
          "value": 0,
          "name": "cell30 ~ gene36"
        },
        {
          "x": 29,
          "y": 36,
          "value": 0,
          "name": "cell30 ~ gene37"
        },
        {
          "x": 29,
          "y": 37,
          "value": 6,
          "name": "cell30 ~ gene38"
        },
        {
          "x": 29,
          "y": 38,
          "value": 0,
          "name": "cell30 ~ gene39"
        },
        {
          "x": 29,
          "y": 39,
          "value": 0,
          "name": "cell30 ~ gene40"
        },
        {
          "x": 29,
          "y": 40,
          "value": 3,
          "name": "cell30 ~ gene41"
        },
        {
          "x": 29,
          "y": 41,
          "value": 0,
          "name": "cell30 ~ gene42"
        },
        {
          "x": 29,
          "y": 42,
          "value": 0,
          "name": "cell30 ~ gene43"
        },
        {
          "x": 29,
          "y": 43,
          "value": 6,
          "name": "cell30 ~ gene44"
        },
        {
          "x": 29,
          "y": 44,
          "value": 2,
          "name": "cell30 ~ gene45"
        },
        {
          "x": 29,
          "y": 45,
          "value": 0,
          "name": "cell30 ~ gene46"
        },
        {
          "x": 29,
          "y": 46,
          "value": 0,
          "name": "cell30 ~ gene47"
        },
        {
          "x": 29,
          "y": 47,
          "value": 0,
          "name": "cell30 ~ gene48"
        },
        {
          "x": 29,
          "y": 48,
          "value": 0,
          "name": "cell30 ~ gene49"
        },
        {
          "x": 29,
          "y": 49,
          "value": 0,
          "name": "cell30 ~ gene50"
        },
        {
          "x": 29,
          "y": 50,
          "value": 10,
          "name": "cell30 ~ gene51"
        },
        {
          "x": 29,
          "y": 51,
          "value": 0,
          "name": "cell30 ~ gene52"
        },
        {
          "x": 29,
          "y": 52,
          "value": 0,
          "name": "cell30 ~ gene53"
        },
        {
          "x": 29,
          "y": 53,
          "value": 0,
          "name": "cell30 ~ gene54"
        },
        {
          "x": 29,
          "y": 54,
          "value": 0,
          "name": "cell30 ~ gene55"
        },
        {
          "x": 29,
          "y": 55,
          "value": 10,
          "name": "cell30 ~ gene56"
        },
        {
          "x": 29,
          "y": 56,
          "value": 0,
          "name": "cell30 ~ gene57"
        },
        {
          "x": 29,
          "y": 57,
          "value": 0,
          "name": "cell30 ~ gene58"
        },
        {
          "x": 29,
          "y": 58,
          "value": 13,
          "name": "cell30 ~ gene59"
        },
        {
          "x": 29,
          "y": 59,
          "value": 1,
          "name": "cell30 ~ gene60"
        },
        {
          "x": 29,
          "y": 60,
          "value": 2,
          "name": "cell30 ~ gene61"
        },
        {
          "x": 29,
          "y": 61,
          "value": 10,
          "name": "cell30 ~ gene62"
        },
        {
          "x": 29,
          "y": 62,
          "value": 0,
          "name": "cell30 ~ gene63"
        },
        {
          "x": 29,
          "y": 63,
          "value": 4,
          "name": "cell30 ~ gene64"
        },
        {
          "x": 29,
          "y": 64,
          "value": 5,
          "name": "cell30 ~ gene65"
        },
        {
          "x": 29,
          "y": 65,
          "value": 5,
          "name": "cell30 ~ gene66"
        },
        {
          "x": 29,
          "y": 66,
          "value": 12,
          "name": "cell30 ~ gene67"
        },
        {
          "x": 29,
          "y": 67,
          "value": 0,
          "name": "cell30 ~ gene68"
        },
        {
          "x": 29,
          "y": 68,
          "value": 8,
          "name": "cell30 ~ gene69"
        },
        {
          "x": 29,
          "y": 69,
          "value": 0,
          "name": "cell30 ~ gene70"
        },
        {
          "x": 29,
          "y": 70,
          "value": 0,
          "name": "cell30 ~ gene71"
        },
        {
          "x": 29,
          "y": 71,
          "value": 0,
          "name": "cell30 ~ gene72"
        },
        {
          "x": 29,
          "y": 72,
          "value": 0,
          "name": "cell30 ~ gene73"
        },
        {
          "x": 29,
          "y": 73,
          "value": 0,
          "name": "cell30 ~ gene74"
        },
        {
          "x": 29,
          "y": 74,
          "value": 3,
          "name": "cell30 ~ gene75"
        },
        {
          "x": 29,
          "y": 75,
          "value": 9,
          "name": "cell30 ~ gene76"
        },
        {
          "x": 29,
          "y": 76,
          "value": 0,
          "name": "cell30 ~ gene77"
        },
        {
          "x": 29,
          "y": 77,
          "value": 0,
          "name": "cell30 ~ gene78"
        },
        {
          "x": 29,
          "y": 78,
          "value": 0,
          "name": "cell30 ~ gene79"
        },
        {
          "x": 29,
          "y": 79,
          "value": 12,
          "name": "cell30 ~ gene80"
        },
        {
          "x": 29,
          "y": 80,
          "value": 0,
          "name": "cell30 ~ gene81"
        },
        {
          "x": 29,
          "y": 81,
          "value": 0,
          "name": "cell30 ~ gene82"
        },
        {
          "x": 29,
          "y": 82,
          "value": 0,
          "name": "cell30 ~ gene83"
        },
        {
          "x": 29,
          "y": 83,
          "value": 0,
          "name": "cell30 ~ gene84"
        },
        {
          "x": 29,
          "y": 84,
          "value": 9,
          "name": "cell30 ~ gene85"
        },
        {
          "x": 29,
          "y": 85,
          "value": 23,
          "name": "cell30 ~ gene86"
        },
        {
          "x": 29,
          "y": 86,
          "value": 12,
          "name": "cell30 ~ gene87"
        },
        {
          "x": 29,
          "y": 87,
          "value": 14,
          "name": "cell30 ~ gene88"
        },
        {
          "x": 29,
          "y": 88,
          "value": 1,
          "name": "cell30 ~ gene89"
        },
        {
          "x": 29,
          "y": 89,
          "value": 2,
          "name": "cell30 ~ gene90"
        },
        {
          "x": 29,
          "y": 90,
          "value": 4,
          "name": "cell30 ~ gene91"
        },
        {
          "x": 29,
          "y": 91,
          "value": 7,
          "name": "cell30 ~ gene92"
        },
        {
          "x": 29,
          "y": 92,
          "value": 0,
          "name": "cell30 ~ gene93"
        },
        {
          "x": 29,
          "y": 93,
          "value": 0,
          "name": "cell30 ~ gene94"
        },
        {
          "x": 29,
          "y": 94,
          "value": 0,
          "name": "cell30 ~ gene95"
        },
        {
          "x": 29,
          "y": 95,
          "value": 0,
          "name": "cell30 ~ gene96"
        },
        {
          "x": 29,
          "y": 96,
          "value": 7,
          "name": "cell30 ~ gene97"
        },
        {
          "x": 29,
          "y": 97,
          "value": 12,
          "name": "cell30 ~ gene98"
        },
        {
          "x": 29,
          "y": 98,
          "value": 0,
          "name": "cell30 ~ gene99"
        },
        {
          "x": 29,
          "y": 99,
          "value": 0,
          "name": "cell30 ~ gene100"
        },
        {
          "x": 30,
          "y": 0,
          "value": 0,
          "name": "cell31 ~ gene1"
        },
        {
          "x": 30,
          "y": 1,
          "value": 5,
          "name": "cell31 ~ gene2"
        },
        {
          "x": 30,
          "y": 2,
          "value": 0,
          "name": "cell31 ~ gene3"
        },
        {
          "x": 30,
          "y": 3,
          "value": 0,
          "name": "cell31 ~ gene4"
        },
        {
          "x": 30,
          "y": 4,
          "value": 0,
          "name": "cell31 ~ gene5"
        },
        {
          "x": 30,
          "y": 5,
          "value": 6,
          "name": "cell31 ~ gene6"
        },
        {
          "x": 30,
          "y": 6,
          "value": 0,
          "name": "cell31 ~ gene7"
        },
        {
          "x": 30,
          "y": 7,
          "value": 0,
          "name": "cell31 ~ gene8"
        },
        {
          "x": 30,
          "y": 8,
          "value": 19,
          "name": "cell31 ~ gene9"
        },
        {
          "x": 30,
          "y": 9,
          "value": 14,
          "name": "cell31 ~ gene10"
        },
        {
          "x": 30,
          "y": 10,
          "value": 4,
          "name": "cell31 ~ gene11"
        },
        {
          "x": 30,
          "y": 11,
          "value": 0,
          "name": "cell31 ~ gene12"
        },
        {
          "x": 30,
          "y": 12,
          "value": 0,
          "name": "cell31 ~ gene13"
        },
        {
          "x": 30,
          "y": 13,
          "value": 0,
          "name": "cell31 ~ gene14"
        },
        {
          "x": 30,
          "y": 14,
          "value": 0,
          "name": "cell31 ~ gene15"
        },
        {
          "x": 30,
          "y": 15,
          "value": 14,
          "name": "cell31 ~ gene16"
        },
        {
          "x": 30,
          "y": 16,
          "value": 0,
          "name": "cell31 ~ gene17"
        },
        {
          "x": 30,
          "y": 17,
          "value": 0,
          "name": "cell31 ~ gene18"
        },
        {
          "x": 30,
          "y": 18,
          "value": 4,
          "name": "cell31 ~ gene19"
        },
        {
          "x": 30,
          "y": 19,
          "value": 18,
          "name": "cell31 ~ gene20"
        },
        {
          "x": 30,
          "y": 20,
          "value": 0,
          "name": "cell31 ~ gene21"
        },
        {
          "x": 30,
          "y": 21,
          "value": 24,
          "name": "cell31 ~ gene22"
        },
        {
          "x": 30,
          "y": 22,
          "value": 10,
          "name": "cell31 ~ gene23"
        },
        {
          "x": 30,
          "y": 23,
          "value": 0,
          "name": "cell31 ~ gene24"
        },
        {
          "x": 30,
          "y": 24,
          "value": 21,
          "name": "cell31 ~ gene25"
        },
        {
          "x": 30,
          "y": 25,
          "value": 11,
          "name": "cell31 ~ gene26"
        },
        {
          "x": 30,
          "y": 26,
          "value": 12,
          "name": "cell31 ~ gene27"
        },
        {
          "x": 30,
          "y": 27,
          "value": 5,
          "name": "cell31 ~ gene28"
        },
        {
          "x": 30,
          "y": 28,
          "value": 15,
          "name": "cell31 ~ gene29"
        },
        {
          "x": 30,
          "y": 29,
          "value": 40,
          "name": "cell31 ~ gene30"
        },
        {
          "x": 30,
          "y": 30,
          "value": 30,
          "name": "cell31 ~ gene31"
        },
        {
          "x": 30,
          "y": 31,
          "value": 8,
          "name": "cell31 ~ gene32"
        },
        {
          "x": 30,
          "y": 32,
          "value": 0,
          "name": "cell31 ~ gene33"
        },
        {
          "x": 30,
          "y": 33,
          "value": 8,
          "name": "cell31 ~ gene34"
        },
        {
          "x": 30,
          "y": 34,
          "value": 19,
          "name": "cell31 ~ gene35"
        },
        {
          "x": 30,
          "y": 35,
          "value": 1,
          "name": "cell31 ~ gene36"
        },
        {
          "x": 30,
          "y": 36,
          "value": 21,
          "name": "cell31 ~ gene37"
        },
        {
          "x": 30,
          "y": 37,
          "value": 19,
          "name": "cell31 ~ gene38"
        },
        {
          "x": 30,
          "y": 38,
          "value": 12,
          "name": "cell31 ~ gene39"
        },
        {
          "x": 30,
          "y": 39,
          "value": 15,
          "name": "cell31 ~ gene40"
        },
        {
          "x": 30,
          "y": 40,
          "value": 0,
          "name": "cell31 ~ gene41"
        },
        {
          "x": 30,
          "y": 41,
          "value": 0,
          "name": "cell31 ~ gene42"
        },
        {
          "x": 30,
          "y": 42,
          "value": 0,
          "name": "cell31 ~ gene43"
        },
        {
          "x": 30,
          "y": 43,
          "value": 11,
          "name": "cell31 ~ gene44"
        },
        {
          "x": 30,
          "y": 44,
          "value": 0,
          "name": "cell31 ~ gene45"
        },
        {
          "x": 30,
          "y": 45,
          "value": 1,
          "name": "cell31 ~ gene46"
        },
        {
          "x": 30,
          "y": 46,
          "value": 2,
          "name": "cell31 ~ gene47"
        },
        {
          "x": 30,
          "y": 47,
          "value": 0,
          "name": "cell31 ~ gene48"
        },
        {
          "x": 30,
          "y": 48,
          "value": 8,
          "name": "cell31 ~ gene49"
        },
        {
          "x": 30,
          "y": 49,
          "value": 5,
          "name": "cell31 ~ gene50"
        },
        {
          "x": 30,
          "y": 50,
          "value": 4,
          "name": "cell31 ~ gene51"
        },
        {
          "x": 30,
          "y": 51,
          "value": 3,
          "name": "cell31 ~ gene52"
        },
        {
          "x": 30,
          "y": 52,
          "value": 0,
          "name": "cell31 ~ gene53"
        },
        {
          "x": 30,
          "y": 53,
          "value": 0,
          "name": "cell31 ~ gene54"
        },
        {
          "x": 30,
          "y": 54,
          "value": 0,
          "name": "cell31 ~ gene55"
        },
        {
          "x": 30,
          "y": 55,
          "value": 6,
          "name": "cell31 ~ gene56"
        },
        {
          "x": 30,
          "y": 56,
          "value": 9,
          "name": "cell31 ~ gene57"
        },
        {
          "x": 30,
          "y": 57,
          "value": 0,
          "name": "cell31 ~ gene58"
        },
        {
          "x": 30,
          "y": 58,
          "value": 5,
          "name": "cell31 ~ gene59"
        },
        {
          "x": 30,
          "y": 59,
          "value": 0,
          "name": "cell31 ~ gene60"
        },
        {
          "x": 30,
          "y": 60,
          "value": 1,
          "name": "cell31 ~ gene61"
        },
        {
          "x": 30,
          "y": 61,
          "value": 0,
          "name": "cell31 ~ gene62"
        },
        {
          "x": 30,
          "y": 62,
          "value": 27,
          "name": "cell31 ~ gene63"
        },
        {
          "x": 30,
          "y": 63,
          "value": 0,
          "name": "cell31 ~ gene64"
        },
        {
          "x": 30,
          "y": 64,
          "value": 15,
          "name": "cell31 ~ gene65"
        },
        {
          "x": 30,
          "y": 65,
          "value": 3,
          "name": "cell31 ~ gene66"
        },
        {
          "x": 30,
          "y": 66,
          "value": 0,
          "name": "cell31 ~ gene67"
        },
        {
          "x": 30,
          "y": 67,
          "value": 3,
          "name": "cell31 ~ gene68"
        },
        {
          "x": 30,
          "y": 68,
          "value": 8,
          "name": "cell31 ~ gene69"
        },
        {
          "x": 30,
          "y": 69,
          "value": 6,
          "name": "cell31 ~ gene70"
        },
        {
          "x": 30,
          "y": 70,
          "value": 16,
          "name": "cell31 ~ gene71"
        },
        {
          "x": 30,
          "y": 71,
          "value": 7,
          "name": "cell31 ~ gene72"
        },
        {
          "x": 30,
          "y": 72,
          "value": 7,
          "name": "cell31 ~ gene73"
        },
        {
          "x": 30,
          "y": 73,
          "value": 0,
          "name": "cell31 ~ gene74"
        },
        {
          "x": 30,
          "y": 74,
          "value": 0,
          "name": "cell31 ~ gene75"
        },
        {
          "x": 30,
          "y": 75,
          "value": 0,
          "name": "cell31 ~ gene76"
        },
        {
          "x": 30,
          "y": 76,
          "value": 0,
          "name": "cell31 ~ gene77"
        },
        {
          "x": 30,
          "y": 77,
          "value": 2,
          "name": "cell31 ~ gene78"
        },
        {
          "x": 30,
          "y": 78,
          "value": 6,
          "name": "cell31 ~ gene79"
        },
        {
          "x": 30,
          "y": 79,
          "value": 8,
          "name": "cell31 ~ gene80"
        },
        {
          "x": 30,
          "y": 80,
          "value": 0,
          "name": "cell31 ~ gene81"
        },
        {
          "x": 30,
          "y": 81,
          "value": 0,
          "name": "cell31 ~ gene82"
        },
        {
          "x": 30,
          "y": 82,
          "value": 0,
          "name": "cell31 ~ gene83"
        },
        {
          "x": 30,
          "y": 83,
          "value": 0,
          "name": "cell31 ~ gene84"
        },
        {
          "x": 30,
          "y": 84,
          "value": 9,
          "name": "cell31 ~ gene85"
        },
        {
          "x": 30,
          "y": 85,
          "value": 0,
          "name": "cell31 ~ gene86"
        },
        {
          "x": 30,
          "y": 86,
          "value": 0,
          "name": "cell31 ~ gene87"
        },
        {
          "x": 30,
          "y": 87,
          "value": 0,
          "name": "cell31 ~ gene88"
        },
        {
          "x": 30,
          "y": 88,
          "value": 7,
          "name": "cell31 ~ gene89"
        },
        {
          "x": 30,
          "y": 89,
          "value": 0,
          "name": "cell31 ~ gene90"
        },
        {
          "x": 30,
          "y": 90,
          "value": 0,
          "name": "cell31 ~ gene91"
        },
        {
          "x": 30,
          "y": 91,
          "value": 5,
          "name": "cell31 ~ gene92"
        },
        {
          "x": 30,
          "y": 92,
          "value": 2,
          "name": "cell31 ~ gene93"
        },
        {
          "x": 30,
          "y": 93,
          "value": 0,
          "name": "cell31 ~ gene94"
        },
        {
          "x": 30,
          "y": 94,
          "value": 12,
          "name": "cell31 ~ gene95"
        },
        {
          "x": 30,
          "y": 95,
          "value": 11,
          "name": "cell31 ~ gene96"
        },
        {
          "x": 30,
          "y": 96,
          "value": 0,
          "name": "cell31 ~ gene97"
        },
        {
          "x": 30,
          "y": 97,
          "value": 0,
          "name": "cell31 ~ gene98"
        },
        {
          "x": 30,
          "y": 98,
          "value": 0,
          "name": "cell31 ~ gene99"
        },
        {
          "x": 30,
          "y": 99,
          "value": 0,
          "name": "cell31 ~ gene100"
        },
        {
          "x": 31,
          "y": 0,
          "value": 1,
          "name": "cell32 ~ gene1"
        },
        {
          "x": 31,
          "y": 1,
          "value": 0,
          "name": "cell32 ~ gene2"
        },
        {
          "x": 31,
          "y": 2,
          "value": 0,
          "name": "cell32 ~ gene3"
        },
        {
          "x": 31,
          "y": 3,
          "value": 0,
          "name": "cell32 ~ gene4"
        },
        {
          "x": 31,
          "y": 4,
          "value": 7,
          "name": "cell32 ~ gene5"
        },
        {
          "x": 31,
          "y": 5,
          "value": 0,
          "name": "cell32 ~ gene6"
        },
        {
          "x": 31,
          "y": 6,
          "value": 26,
          "name": "cell32 ~ gene7"
        },
        {
          "x": 31,
          "y": 7,
          "value": 4,
          "name": "cell32 ~ gene8"
        },
        {
          "x": 31,
          "y": 8,
          "value": 0,
          "name": "cell32 ~ gene9"
        },
        {
          "x": 31,
          "y": 9,
          "value": 0,
          "name": "cell32 ~ gene10"
        },
        {
          "x": 31,
          "y": 10,
          "value": 0,
          "name": "cell32 ~ gene11"
        },
        {
          "x": 31,
          "y": 11,
          "value": 0,
          "name": "cell32 ~ gene12"
        },
        {
          "x": 31,
          "y": 12,
          "value": 4,
          "name": "cell32 ~ gene13"
        },
        {
          "x": 31,
          "y": 13,
          "value": 11,
          "name": "cell32 ~ gene14"
        },
        {
          "x": 31,
          "y": 14,
          "value": 0,
          "name": "cell32 ~ gene15"
        },
        {
          "x": 31,
          "y": 15,
          "value": 0,
          "name": "cell32 ~ gene16"
        },
        {
          "x": 31,
          "y": 16,
          "value": 0,
          "name": "cell32 ~ gene17"
        },
        {
          "x": 31,
          "y": 17,
          "value": 7,
          "name": "cell32 ~ gene18"
        },
        {
          "x": 31,
          "y": 18,
          "value": 20,
          "name": "cell32 ~ gene19"
        },
        {
          "x": 31,
          "y": 19,
          "value": 8,
          "name": "cell32 ~ gene20"
        },
        {
          "x": 31,
          "y": 20,
          "value": 10,
          "name": "cell32 ~ gene21"
        },
        {
          "x": 31,
          "y": 21,
          "value": 19,
          "name": "cell32 ~ gene22"
        },
        {
          "x": 31,
          "y": 22,
          "value": 0,
          "name": "cell32 ~ gene23"
        },
        {
          "x": 31,
          "y": 23,
          "value": 0,
          "name": "cell32 ~ gene24"
        },
        {
          "x": 31,
          "y": 24,
          "value": 21,
          "name": "cell32 ~ gene25"
        },
        {
          "x": 31,
          "y": 25,
          "value": 8,
          "name": "cell32 ~ gene26"
        },
        {
          "x": 31,
          "y": 26,
          "value": 25,
          "name": "cell32 ~ gene27"
        },
        {
          "x": 31,
          "y": 27,
          "value": 3,
          "name": "cell32 ~ gene28"
        },
        {
          "x": 31,
          "y": 28,
          "value": 9,
          "name": "cell32 ~ gene29"
        },
        {
          "x": 31,
          "y": 29,
          "value": 31,
          "name": "cell32 ~ gene30"
        },
        {
          "x": 31,
          "y": 30,
          "value": 39,
          "name": "cell32 ~ gene31"
        },
        {
          "x": 31,
          "y": 31,
          "value": 20,
          "name": "cell32 ~ gene32"
        },
        {
          "x": 31,
          "y": 32,
          "value": 0,
          "name": "cell32 ~ gene33"
        },
        {
          "x": 31,
          "y": 33,
          "value": 8,
          "name": "cell32 ~ gene34"
        },
        {
          "x": 31,
          "y": 34,
          "value": 13,
          "name": "cell32 ~ gene35"
        },
        {
          "x": 31,
          "y": 35,
          "value": 0,
          "name": "cell32 ~ gene36"
        },
        {
          "x": 31,
          "y": 36,
          "value": 4,
          "name": "cell32 ~ gene37"
        },
        {
          "x": 31,
          "y": 37,
          "value": 8,
          "name": "cell32 ~ gene38"
        },
        {
          "x": 31,
          "y": 38,
          "value": 0,
          "name": "cell32 ~ gene39"
        },
        {
          "x": 31,
          "y": 39,
          "value": 3,
          "name": "cell32 ~ gene40"
        },
        {
          "x": 31,
          "y": 40,
          "value": 0,
          "name": "cell32 ~ gene41"
        },
        {
          "x": 31,
          "y": 41,
          "value": 10,
          "name": "cell32 ~ gene42"
        },
        {
          "x": 31,
          "y": 42,
          "value": 7,
          "name": "cell32 ~ gene43"
        },
        {
          "x": 31,
          "y": 43,
          "value": 0,
          "name": "cell32 ~ gene44"
        },
        {
          "x": 31,
          "y": 44,
          "value": 0,
          "name": "cell32 ~ gene45"
        },
        {
          "x": 31,
          "y": 45,
          "value": 0,
          "name": "cell32 ~ gene46"
        },
        {
          "x": 31,
          "y": 46,
          "value": 5,
          "name": "cell32 ~ gene47"
        },
        {
          "x": 31,
          "y": 47,
          "value": 0,
          "name": "cell32 ~ gene48"
        },
        {
          "x": 31,
          "y": 48,
          "value": 4,
          "name": "cell32 ~ gene49"
        },
        {
          "x": 31,
          "y": 49,
          "value": 1,
          "name": "cell32 ~ gene50"
        },
        {
          "x": 31,
          "y": 50,
          "value": 7,
          "name": "cell32 ~ gene51"
        },
        {
          "x": 31,
          "y": 51,
          "value": 14,
          "name": "cell32 ~ gene52"
        },
        {
          "x": 31,
          "y": 52,
          "value": 10,
          "name": "cell32 ~ gene53"
        },
        {
          "x": 31,
          "y": 53,
          "value": 12,
          "name": "cell32 ~ gene54"
        },
        {
          "x": 31,
          "y": 54,
          "value": 8,
          "name": "cell32 ~ gene55"
        },
        {
          "x": 31,
          "y": 55,
          "value": 1,
          "name": "cell32 ~ gene56"
        },
        {
          "x": 31,
          "y": 56,
          "value": 0,
          "name": "cell32 ~ gene57"
        },
        {
          "x": 31,
          "y": 57,
          "value": 24,
          "name": "cell32 ~ gene58"
        },
        {
          "x": 31,
          "y": 58,
          "value": 1,
          "name": "cell32 ~ gene59"
        },
        {
          "x": 31,
          "y": 59,
          "value": 0,
          "name": "cell32 ~ gene60"
        },
        {
          "x": 31,
          "y": 60,
          "value": 0,
          "name": "cell32 ~ gene61"
        },
        {
          "x": 31,
          "y": 61,
          "value": 3,
          "name": "cell32 ~ gene62"
        },
        {
          "x": 31,
          "y": 62,
          "value": 0,
          "name": "cell32 ~ gene63"
        },
        {
          "x": 31,
          "y": 63,
          "value": 0,
          "name": "cell32 ~ gene64"
        },
        {
          "x": 31,
          "y": 64,
          "value": 0,
          "name": "cell32 ~ gene65"
        },
        {
          "x": 31,
          "y": 65,
          "value": 2,
          "name": "cell32 ~ gene66"
        },
        {
          "x": 31,
          "y": 66,
          "value": 0,
          "name": "cell32 ~ gene67"
        },
        {
          "x": 31,
          "y": 67,
          "value": 8,
          "name": "cell32 ~ gene68"
        },
        {
          "x": 31,
          "y": 68,
          "value": 0,
          "name": "cell32 ~ gene69"
        },
        {
          "x": 31,
          "y": 69,
          "value": 0,
          "name": "cell32 ~ gene70"
        },
        {
          "x": 31,
          "y": 70,
          "value": 12,
          "name": "cell32 ~ gene71"
        },
        {
          "x": 31,
          "y": 71,
          "value": 3,
          "name": "cell32 ~ gene72"
        },
        {
          "x": 31,
          "y": 72,
          "value": 23,
          "name": "cell32 ~ gene73"
        },
        {
          "x": 31,
          "y": 73,
          "value": 16,
          "name": "cell32 ~ gene74"
        },
        {
          "x": 31,
          "y": 74,
          "value": 0,
          "name": "cell32 ~ gene75"
        },
        {
          "x": 31,
          "y": 75,
          "value": 5,
          "name": "cell32 ~ gene76"
        },
        {
          "x": 31,
          "y": 76,
          "value": 17,
          "name": "cell32 ~ gene77"
        },
        {
          "x": 31,
          "y": 77,
          "value": 0,
          "name": "cell32 ~ gene78"
        },
        {
          "x": 31,
          "y": 78,
          "value": 1,
          "name": "cell32 ~ gene79"
        },
        {
          "x": 31,
          "y": 79,
          "value": 2,
          "name": "cell32 ~ gene80"
        },
        {
          "x": 31,
          "y": 80,
          "value": 0,
          "name": "cell32 ~ gene81"
        },
        {
          "x": 31,
          "y": 81,
          "value": 0,
          "name": "cell32 ~ gene82"
        },
        {
          "x": 31,
          "y": 82,
          "value": 0,
          "name": "cell32 ~ gene83"
        },
        {
          "x": 31,
          "y": 83,
          "value": 8,
          "name": "cell32 ~ gene84"
        },
        {
          "x": 31,
          "y": 84,
          "value": 2,
          "name": "cell32 ~ gene85"
        },
        {
          "x": 31,
          "y": 85,
          "value": 0,
          "name": "cell32 ~ gene86"
        },
        {
          "x": 31,
          "y": 86,
          "value": 5,
          "name": "cell32 ~ gene87"
        },
        {
          "x": 31,
          "y": 87,
          "value": 0,
          "name": "cell32 ~ gene88"
        },
        {
          "x": 31,
          "y": 88,
          "value": 4,
          "name": "cell32 ~ gene89"
        },
        {
          "x": 31,
          "y": 89,
          "value": 0,
          "name": "cell32 ~ gene90"
        },
        {
          "x": 31,
          "y": 90,
          "value": 0,
          "name": "cell32 ~ gene91"
        },
        {
          "x": 31,
          "y": 91,
          "value": 0,
          "name": "cell32 ~ gene92"
        },
        {
          "x": 31,
          "y": 92,
          "value": 0,
          "name": "cell32 ~ gene93"
        },
        {
          "x": 31,
          "y": 93,
          "value": 1,
          "name": "cell32 ~ gene94"
        },
        {
          "x": 31,
          "y": 94,
          "value": 0,
          "name": "cell32 ~ gene95"
        },
        {
          "x": 31,
          "y": 95,
          "value": 0,
          "name": "cell32 ~ gene96"
        },
        {
          "x": 31,
          "y": 96,
          "value": 2,
          "name": "cell32 ~ gene97"
        },
        {
          "x": 31,
          "y": 97,
          "value": 16,
          "name": "cell32 ~ gene98"
        },
        {
          "x": 31,
          "y": 98,
          "value": 19,
          "name": "cell32 ~ gene99"
        },
        {
          "x": 31,
          "y": 99,
          "value": 8,
          "name": "cell32 ~ gene100"
        },
        {
          "x": 32,
          "y": 0,
          "value": 15,
          "name": "cell33 ~ gene1"
        },
        {
          "x": 32,
          "y": 1,
          "value": 11,
          "name": "cell33 ~ gene2"
        },
        {
          "x": 32,
          "y": 2,
          "value": 0,
          "name": "cell33 ~ gene3"
        },
        {
          "x": 32,
          "y": 3,
          "value": 14,
          "name": "cell33 ~ gene4"
        },
        {
          "x": 32,
          "y": 4,
          "value": 0,
          "name": "cell33 ~ gene5"
        },
        {
          "x": 32,
          "y": 5,
          "value": 0,
          "name": "cell33 ~ gene6"
        },
        {
          "x": 32,
          "y": 6,
          "value": 1,
          "name": "cell33 ~ gene7"
        },
        {
          "x": 32,
          "y": 7,
          "value": 2,
          "name": "cell33 ~ gene8"
        },
        {
          "x": 32,
          "y": 8,
          "value": 0,
          "name": "cell33 ~ gene9"
        },
        {
          "x": 32,
          "y": 9,
          "value": 0,
          "name": "cell33 ~ gene10"
        },
        {
          "x": 32,
          "y": 10,
          "value": 1,
          "name": "cell33 ~ gene11"
        },
        {
          "x": 32,
          "y": 11,
          "value": 3,
          "name": "cell33 ~ gene12"
        },
        {
          "x": 32,
          "y": 12,
          "value": 3,
          "name": "cell33 ~ gene13"
        },
        {
          "x": 32,
          "y": 13,
          "value": 3,
          "name": "cell33 ~ gene14"
        },
        {
          "x": 32,
          "y": 14,
          "value": 12,
          "name": "cell33 ~ gene15"
        },
        {
          "x": 32,
          "y": 15,
          "value": 0,
          "name": "cell33 ~ gene16"
        },
        {
          "x": 32,
          "y": 16,
          "value": 0,
          "name": "cell33 ~ gene17"
        },
        {
          "x": 32,
          "y": 17,
          "value": 0,
          "name": "cell33 ~ gene18"
        },
        {
          "x": 32,
          "y": 18,
          "value": 20,
          "name": "cell33 ~ gene19"
        },
        {
          "x": 32,
          "y": 19,
          "value": 6,
          "name": "cell33 ~ gene20"
        },
        {
          "x": 32,
          "y": 20,
          "value": 0,
          "name": "cell33 ~ gene21"
        },
        {
          "x": 32,
          "y": 21,
          "value": 24,
          "name": "cell33 ~ gene22"
        },
        {
          "x": 32,
          "y": 22,
          "value": 1,
          "name": "cell33 ~ gene23"
        },
        {
          "x": 32,
          "y": 23,
          "value": 0,
          "name": "cell33 ~ gene24"
        },
        {
          "x": 32,
          "y": 24,
          "value": 33,
          "name": "cell33 ~ gene25"
        },
        {
          "x": 32,
          "y": 25,
          "value": 25,
          "name": "cell33 ~ gene26"
        },
        {
          "x": 32,
          "y": 26,
          "value": 3,
          "name": "cell33 ~ gene27"
        },
        {
          "x": 32,
          "y": 27,
          "value": 0,
          "name": "cell33 ~ gene28"
        },
        {
          "x": 32,
          "y": 28,
          "value": 0,
          "name": "cell33 ~ gene29"
        },
        {
          "x": 32,
          "y": 29,
          "value": 13,
          "name": "cell33 ~ gene30"
        },
        {
          "x": 32,
          "y": 30,
          "value": 32,
          "name": "cell33 ~ gene31"
        },
        {
          "x": 32,
          "y": 31,
          "value": 23,
          "name": "cell33 ~ gene32"
        },
        {
          "x": 32,
          "y": 32,
          "value": 0,
          "name": "cell33 ~ gene33"
        },
        {
          "x": 32,
          "y": 33,
          "value": 15,
          "name": "cell33 ~ gene34"
        },
        {
          "x": 32,
          "y": 34,
          "value": 8,
          "name": "cell33 ~ gene35"
        },
        {
          "x": 32,
          "y": 35,
          "value": 0,
          "name": "cell33 ~ gene36"
        },
        {
          "x": 32,
          "y": 36,
          "value": 22,
          "name": "cell33 ~ gene37"
        },
        {
          "x": 32,
          "y": 37,
          "value": 28,
          "name": "cell33 ~ gene38"
        },
        {
          "x": 32,
          "y": 38,
          "value": 4,
          "name": "cell33 ~ gene39"
        },
        {
          "x": 32,
          "y": 39,
          "value": 0,
          "name": "cell33 ~ gene40"
        },
        {
          "x": 32,
          "y": 40,
          "value": 6,
          "name": "cell33 ~ gene41"
        },
        {
          "x": 32,
          "y": 41,
          "value": 12,
          "name": "cell33 ~ gene42"
        },
        {
          "x": 32,
          "y": 42,
          "value": 0,
          "name": "cell33 ~ gene43"
        },
        {
          "x": 32,
          "y": 43,
          "value": 12,
          "name": "cell33 ~ gene44"
        },
        {
          "x": 32,
          "y": 44,
          "value": 0,
          "name": "cell33 ~ gene45"
        },
        {
          "x": 32,
          "y": 45,
          "value": 6,
          "name": "cell33 ~ gene46"
        },
        {
          "x": 32,
          "y": 46,
          "value": 0,
          "name": "cell33 ~ gene47"
        },
        {
          "x": 32,
          "y": 47,
          "value": 0,
          "name": "cell33 ~ gene48"
        },
        {
          "x": 32,
          "y": 48,
          "value": 1,
          "name": "cell33 ~ gene49"
        },
        {
          "x": 32,
          "y": 49,
          "value": 0,
          "name": "cell33 ~ gene50"
        },
        {
          "x": 32,
          "y": 50,
          "value": 0,
          "name": "cell33 ~ gene51"
        },
        {
          "x": 32,
          "y": 51,
          "value": 0,
          "name": "cell33 ~ gene52"
        },
        {
          "x": 32,
          "y": 52,
          "value": 0,
          "name": "cell33 ~ gene53"
        },
        {
          "x": 32,
          "y": 53,
          "value": 10,
          "name": "cell33 ~ gene54"
        },
        {
          "x": 32,
          "y": 54,
          "value": 0,
          "name": "cell33 ~ gene55"
        },
        {
          "x": 32,
          "y": 55,
          "value": 0,
          "name": "cell33 ~ gene56"
        },
        {
          "x": 32,
          "y": 56,
          "value": 8,
          "name": "cell33 ~ gene57"
        },
        {
          "x": 32,
          "y": 57,
          "value": 0,
          "name": "cell33 ~ gene58"
        },
        {
          "x": 32,
          "y": 58,
          "value": 0,
          "name": "cell33 ~ gene59"
        },
        {
          "x": 32,
          "y": 59,
          "value": 0,
          "name": "cell33 ~ gene60"
        },
        {
          "x": 32,
          "y": 60,
          "value": 13,
          "name": "cell33 ~ gene61"
        },
        {
          "x": 32,
          "y": 61,
          "value": 5,
          "name": "cell33 ~ gene62"
        },
        {
          "x": 32,
          "y": 62,
          "value": 0,
          "name": "cell33 ~ gene63"
        },
        {
          "x": 32,
          "y": 63,
          "value": 0,
          "name": "cell33 ~ gene64"
        },
        {
          "x": 32,
          "y": 64,
          "value": 0,
          "name": "cell33 ~ gene65"
        },
        {
          "x": 32,
          "y": 65,
          "value": 0,
          "name": "cell33 ~ gene66"
        },
        {
          "x": 32,
          "y": 66,
          "value": 12,
          "name": "cell33 ~ gene67"
        },
        {
          "x": 32,
          "y": 67,
          "value": 0,
          "name": "cell33 ~ gene68"
        },
        {
          "x": 32,
          "y": 68,
          "value": 0,
          "name": "cell33 ~ gene69"
        },
        {
          "x": 32,
          "y": 69,
          "value": 0,
          "name": "cell33 ~ gene70"
        },
        {
          "x": 32,
          "y": 70,
          "value": 0,
          "name": "cell33 ~ gene71"
        },
        {
          "x": 32,
          "y": 71,
          "value": 4,
          "name": "cell33 ~ gene72"
        },
        {
          "x": 32,
          "y": 72,
          "value": 0,
          "name": "cell33 ~ gene73"
        },
        {
          "x": 32,
          "y": 73,
          "value": 2,
          "name": "cell33 ~ gene74"
        },
        {
          "x": 32,
          "y": 74,
          "value": 0,
          "name": "cell33 ~ gene75"
        },
        {
          "x": 32,
          "y": 75,
          "value": 8,
          "name": "cell33 ~ gene76"
        },
        {
          "x": 32,
          "y": 76,
          "value": 3,
          "name": "cell33 ~ gene77"
        },
        {
          "x": 32,
          "y": 77,
          "value": 1,
          "name": "cell33 ~ gene78"
        },
        {
          "x": 32,
          "y": 78,
          "value": 19,
          "name": "cell33 ~ gene79"
        },
        {
          "x": 32,
          "y": 79,
          "value": 0,
          "name": "cell33 ~ gene80"
        },
        {
          "x": 32,
          "y": 80,
          "value": 12,
          "name": "cell33 ~ gene81"
        },
        {
          "x": 32,
          "y": 81,
          "value": 2,
          "name": "cell33 ~ gene82"
        },
        {
          "x": 32,
          "y": 82,
          "value": 0,
          "name": "cell33 ~ gene83"
        },
        {
          "x": 32,
          "y": 83,
          "value": 0,
          "name": "cell33 ~ gene84"
        },
        {
          "x": 32,
          "y": 84,
          "value": 0,
          "name": "cell33 ~ gene85"
        },
        {
          "x": 32,
          "y": 85,
          "value": 6,
          "name": "cell33 ~ gene86"
        },
        {
          "x": 32,
          "y": 86,
          "value": 0,
          "name": "cell33 ~ gene87"
        },
        {
          "x": 32,
          "y": 87,
          "value": 13,
          "name": "cell33 ~ gene88"
        },
        {
          "x": 32,
          "y": 88,
          "value": 3,
          "name": "cell33 ~ gene89"
        },
        {
          "x": 32,
          "y": 89,
          "value": 0,
          "name": "cell33 ~ gene90"
        },
        {
          "x": 32,
          "y": 90,
          "value": 0,
          "name": "cell33 ~ gene91"
        },
        {
          "x": 32,
          "y": 91,
          "value": 0,
          "name": "cell33 ~ gene92"
        },
        {
          "x": 32,
          "y": 92,
          "value": 0,
          "name": "cell33 ~ gene93"
        },
        {
          "x": 32,
          "y": 93,
          "value": 0,
          "name": "cell33 ~ gene94"
        },
        {
          "x": 32,
          "y": 94,
          "value": 1,
          "name": "cell33 ~ gene95"
        },
        {
          "x": 32,
          "y": 95,
          "value": 0,
          "name": "cell33 ~ gene96"
        },
        {
          "x": 32,
          "y": 96,
          "value": 1,
          "name": "cell33 ~ gene97"
        },
        {
          "x": 32,
          "y": 97,
          "value": 2,
          "name": "cell33 ~ gene98"
        },
        {
          "x": 32,
          "y": 98,
          "value": 0,
          "name": "cell33 ~ gene99"
        },
        {
          "x": 32,
          "y": 99,
          "value": 12,
          "name": "cell33 ~ gene100"
        },
        {
          "x": 33,
          "y": 0,
          "value": 16,
          "name": "cell34 ~ gene1"
        },
        {
          "x": 33,
          "y": 1,
          "value": 0,
          "name": "cell34 ~ gene2"
        },
        {
          "x": 33,
          "y": 2,
          "value": 4,
          "name": "cell34 ~ gene3"
        },
        {
          "x": 33,
          "y": 3,
          "value": 0,
          "name": "cell34 ~ gene4"
        },
        {
          "x": 33,
          "y": 4,
          "value": 6,
          "name": "cell34 ~ gene5"
        },
        {
          "x": 33,
          "y": 5,
          "value": 3,
          "name": "cell34 ~ gene6"
        },
        {
          "x": 33,
          "y": 6,
          "value": 0,
          "name": "cell34 ~ gene7"
        },
        {
          "x": 33,
          "y": 7,
          "value": 3,
          "name": "cell34 ~ gene8"
        },
        {
          "x": 33,
          "y": 8,
          "value": 5,
          "name": "cell34 ~ gene9"
        },
        {
          "x": 33,
          "y": 9,
          "value": 11,
          "name": "cell34 ~ gene10"
        },
        {
          "x": 33,
          "y": 10,
          "value": 0,
          "name": "cell34 ~ gene11"
        },
        {
          "x": 33,
          "y": 11,
          "value": 18,
          "name": "cell34 ~ gene12"
        },
        {
          "x": 33,
          "y": 12,
          "value": 0,
          "name": "cell34 ~ gene13"
        },
        {
          "x": 33,
          "y": 13,
          "value": 12,
          "name": "cell34 ~ gene14"
        },
        {
          "x": 33,
          "y": 14,
          "value": 12,
          "name": "cell34 ~ gene15"
        },
        {
          "x": 33,
          "y": 15,
          "value": 0,
          "name": "cell34 ~ gene16"
        },
        {
          "x": 33,
          "y": 16,
          "value": 0,
          "name": "cell34 ~ gene17"
        },
        {
          "x": 33,
          "y": 17,
          "value": 0,
          "name": "cell34 ~ gene18"
        },
        {
          "x": 33,
          "y": 18,
          "value": 5,
          "name": "cell34 ~ gene19"
        },
        {
          "x": 33,
          "y": 19,
          "value": 0,
          "name": "cell34 ~ gene20"
        },
        {
          "x": 33,
          "y": 20,
          "value": 2,
          "name": "cell34 ~ gene21"
        },
        {
          "x": 33,
          "y": 21,
          "value": 19,
          "name": "cell34 ~ gene22"
        },
        {
          "x": 33,
          "y": 22,
          "value": 7,
          "name": "cell34 ~ gene23"
        },
        {
          "x": 33,
          "y": 23,
          "value": 0,
          "name": "cell34 ~ gene24"
        },
        {
          "x": 33,
          "y": 24,
          "value": 18,
          "name": "cell34 ~ gene25"
        },
        {
          "x": 33,
          "y": 25,
          "value": 13,
          "name": "cell34 ~ gene26"
        },
        {
          "x": 33,
          "y": 26,
          "value": 19,
          "name": "cell34 ~ gene27"
        },
        {
          "x": 33,
          "y": 27,
          "value": 17,
          "name": "cell34 ~ gene28"
        },
        {
          "x": 33,
          "y": 28,
          "value": 0,
          "name": "cell34 ~ gene29"
        },
        {
          "x": 33,
          "y": 29,
          "value": 30,
          "name": "cell34 ~ gene30"
        },
        {
          "x": 33,
          "y": 30,
          "value": 46,
          "name": "cell34 ~ gene31"
        },
        {
          "x": 33,
          "y": 31,
          "value": 7,
          "name": "cell34 ~ gene32"
        },
        {
          "x": 33,
          "y": 32,
          "value": 0,
          "name": "cell34 ~ gene33"
        },
        {
          "x": 33,
          "y": 33,
          "value": 5,
          "name": "cell34 ~ gene34"
        },
        {
          "x": 33,
          "y": 34,
          "value": 23,
          "name": "cell34 ~ gene35"
        },
        {
          "x": 33,
          "y": 35,
          "value": 0,
          "name": "cell34 ~ gene36"
        },
        {
          "x": 33,
          "y": 36,
          "value": 22,
          "name": "cell34 ~ gene37"
        },
        {
          "x": 33,
          "y": 37,
          "value": 20,
          "name": "cell34 ~ gene38"
        },
        {
          "x": 33,
          "y": 38,
          "value": 0,
          "name": "cell34 ~ gene39"
        },
        {
          "x": 33,
          "y": 39,
          "value": 19,
          "name": "cell34 ~ gene40"
        },
        {
          "x": 33,
          "y": 40,
          "value": 0,
          "name": "cell34 ~ gene41"
        },
        {
          "x": 33,
          "y": 41,
          "value": 0,
          "name": "cell34 ~ gene42"
        },
        {
          "x": 33,
          "y": 42,
          "value": 11,
          "name": "cell34 ~ gene43"
        },
        {
          "x": 33,
          "y": 43,
          "value": 0,
          "name": "cell34 ~ gene44"
        },
        {
          "x": 33,
          "y": 44,
          "value": 4,
          "name": "cell34 ~ gene45"
        },
        {
          "x": 33,
          "y": 45,
          "value": 10,
          "name": "cell34 ~ gene46"
        },
        {
          "x": 33,
          "y": 46,
          "value": 1,
          "name": "cell34 ~ gene47"
        },
        {
          "x": 33,
          "y": 47,
          "value": 0,
          "name": "cell34 ~ gene48"
        },
        {
          "x": 33,
          "y": 48,
          "value": 0,
          "name": "cell34 ~ gene49"
        },
        {
          "x": 33,
          "y": 49,
          "value": 8,
          "name": "cell34 ~ gene50"
        },
        {
          "x": 33,
          "y": 50,
          "value": 6,
          "name": "cell34 ~ gene51"
        },
        {
          "x": 33,
          "y": 51,
          "value": 0,
          "name": "cell34 ~ gene52"
        },
        {
          "x": 33,
          "y": 52,
          "value": 0,
          "name": "cell34 ~ gene53"
        },
        {
          "x": 33,
          "y": 53,
          "value": 0,
          "name": "cell34 ~ gene54"
        },
        {
          "x": 33,
          "y": 54,
          "value": 9,
          "name": "cell34 ~ gene55"
        },
        {
          "x": 33,
          "y": 55,
          "value": 0,
          "name": "cell34 ~ gene56"
        },
        {
          "x": 33,
          "y": 56,
          "value": 1,
          "name": "cell34 ~ gene57"
        },
        {
          "x": 33,
          "y": 57,
          "value": 6,
          "name": "cell34 ~ gene58"
        },
        {
          "x": 33,
          "y": 58,
          "value": 0,
          "name": "cell34 ~ gene59"
        },
        {
          "x": 33,
          "y": 59,
          "value": 0,
          "name": "cell34 ~ gene60"
        },
        {
          "x": 33,
          "y": 60,
          "value": 0,
          "name": "cell34 ~ gene61"
        },
        {
          "x": 33,
          "y": 61,
          "value": 0,
          "name": "cell34 ~ gene62"
        },
        {
          "x": 33,
          "y": 62,
          "value": 0,
          "name": "cell34 ~ gene63"
        },
        {
          "x": 33,
          "y": 63,
          "value": 0,
          "name": "cell34 ~ gene64"
        },
        {
          "x": 33,
          "y": 64,
          "value": 11,
          "name": "cell34 ~ gene65"
        },
        {
          "x": 33,
          "y": 65,
          "value": 12,
          "name": "cell34 ~ gene66"
        },
        {
          "x": 33,
          "y": 66,
          "value": 0,
          "name": "cell34 ~ gene67"
        },
        {
          "x": 33,
          "y": 67,
          "value": 13,
          "name": "cell34 ~ gene68"
        },
        {
          "x": 33,
          "y": 68,
          "value": 0,
          "name": "cell34 ~ gene69"
        },
        {
          "x": 33,
          "y": 69,
          "value": 18,
          "name": "cell34 ~ gene70"
        },
        {
          "x": 33,
          "y": 70,
          "value": 0,
          "name": "cell34 ~ gene71"
        },
        {
          "x": 33,
          "y": 71,
          "value": 10,
          "name": "cell34 ~ gene72"
        },
        {
          "x": 33,
          "y": 72,
          "value": 0,
          "name": "cell34 ~ gene73"
        },
        {
          "x": 33,
          "y": 73,
          "value": 1,
          "name": "cell34 ~ gene74"
        },
        {
          "x": 33,
          "y": 74,
          "value": 0,
          "name": "cell34 ~ gene75"
        },
        {
          "x": 33,
          "y": 75,
          "value": 0,
          "name": "cell34 ~ gene76"
        },
        {
          "x": 33,
          "y": 76,
          "value": 0,
          "name": "cell34 ~ gene77"
        },
        {
          "x": 33,
          "y": 77,
          "value": 0,
          "name": "cell34 ~ gene78"
        },
        {
          "x": 33,
          "y": 78,
          "value": 1,
          "name": "cell34 ~ gene79"
        },
        {
          "x": 33,
          "y": 79,
          "value": 4,
          "name": "cell34 ~ gene80"
        },
        {
          "x": 33,
          "y": 80,
          "value": 0,
          "name": "cell34 ~ gene81"
        },
        {
          "x": 33,
          "y": 81,
          "value": 0,
          "name": "cell34 ~ gene82"
        },
        {
          "x": 33,
          "y": 82,
          "value": 0,
          "name": "cell34 ~ gene83"
        },
        {
          "x": 33,
          "y": 83,
          "value": 11,
          "name": "cell34 ~ gene84"
        },
        {
          "x": 33,
          "y": 84,
          "value": 0,
          "name": "cell34 ~ gene85"
        },
        {
          "x": 33,
          "y": 85,
          "value": 0,
          "name": "cell34 ~ gene86"
        },
        {
          "x": 33,
          "y": 86,
          "value": 0,
          "name": "cell34 ~ gene87"
        },
        {
          "x": 33,
          "y": 87,
          "value": 0,
          "name": "cell34 ~ gene88"
        },
        {
          "x": 33,
          "y": 88,
          "value": 7,
          "name": "cell34 ~ gene89"
        },
        {
          "x": 33,
          "y": 89,
          "value": 5,
          "name": "cell34 ~ gene90"
        },
        {
          "x": 33,
          "y": 90,
          "value": 0,
          "name": "cell34 ~ gene91"
        },
        {
          "x": 33,
          "y": 91,
          "value": 0,
          "name": "cell34 ~ gene92"
        },
        {
          "x": 33,
          "y": 92,
          "value": 0,
          "name": "cell34 ~ gene93"
        },
        {
          "x": 33,
          "y": 93,
          "value": 0,
          "name": "cell34 ~ gene94"
        },
        {
          "x": 33,
          "y": 94,
          "value": 0,
          "name": "cell34 ~ gene95"
        },
        {
          "x": 33,
          "y": 95,
          "value": 8,
          "name": "cell34 ~ gene96"
        },
        {
          "x": 33,
          "y": 96,
          "value": 4,
          "name": "cell34 ~ gene97"
        },
        {
          "x": 33,
          "y": 97,
          "value": 0,
          "name": "cell34 ~ gene98"
        },
        {
          "x": 33,
          "y": 98,
          "value": 4,
          "name": "cell34 ~ gene99"
        },
        {
          "x": 33,
          "y": 99,
          "value": 0,
          "name": "cell34 ~ gene100"
        },
        {
          "x": 34,
          "y": 0,
          "value": 0,
          "name": "cell35 ~ gene1"
        },
        {
          "x": 34,
          "y": 1,
          "value": 0,
          "name": "cell35 ~ gene2"
        },
        {
          "x": 34,
          "y": 2,
          "value": 7,
          "name": "cell35 ~ gene3"
        },
        {
          "x": 34,
          "y": 3,
          "value": 11,
          "name": "cell35 ~ gene4"
        },
        {
          "x": 34,
          "y": 4,
          "value": 0,
          "name": "cell35 ~ gene5"
        },
        {
          "x": 34,
          "y": 5,
          "value": 7,
          "name": "cell35 ~ gene6"
        },
        {
          "x": 34,
          "y": 6,
          "value": 1,
          "name": "cell35 ~ gene7"
        },
        {
          "x": 34,
          "y": 7,
          "value": 8,
          "name": "cell35 ~ gene8"
        },
        {
          "x": 34,
          "y": 8,
          "value": 13,
          "name": "cell35 ~ gene9"
        },
        {
          "x": 34,
          "y": 9,
          "value": 3,
          "name": "cell35 ~ gene10"
        },
        {
          "x": 34,
          "y": 10,
          "value": 0,
          "name": "cell35 ~ gene11"
        },
        {
          "x": 34,
          "y": 11,
          "value": 10,
          "name": "cell35 ~ gene12"
        },
        {
          "x": 34,
          "y": 12,
          "value": 10,
          "name": "cell35 ~ gene13"
        },
        {
          "x": 34,
          "y": 13,
          "value": 0,
          "name": "cell35 ~ gene14"
        },
        {
          "x": 34,
          "y": 14,
          "value": 0,
          "name": "cell35 ~ gene15"
        },
        {
          "x": 34,
          "y": 15,
          "value": 9,
          "name": "cell35 ~ gene16"
        },
        {
          "x": 34,
          "y": 16,
          "value": 5,
          "name": "cell35 ~ gene17"
        },
        {
          "x": 34,
          "y": 17,
          "value": 0,
          "name": "cell35 ~ gene18"
        },
        {
          "x": 34,
          "y": 18,
          "value": 9,
          "name": "cell35 ~ gene19"
        },
        {
          "x": 34,
          "y": 19,
          "value": 7,
          "name": "cell35 ~ gene20"
        },
        {
          "x": 34,
          "y": 20,
          "value": 9,
          "name": "cell35 ~ gene21"
        },
        {
          "x": 34,
          "y": 21,
          "value": 26,
          "name": "cell35 ~ gene22"
        },
        {
          "x": 34,
          "y": 22,
          "value": 0,
          "name": "cell35 ~ gene23"
        },
        {
          "x": 34,
          "y": 23,
          "value": 0,
          "name": "cell35 ~ gene24"
        },
        {
          "x": 34,
          "y": 24,
          "value": 28,
          "name": "cell35 ~ gene25"
        },
        {
          "x": 34,
          "y": 25,
          "value": 7,
          "name": "cell35 ~ gene26"
        },
        {
          "x": 34,
          "y": 26,
          "value": 3,
          "name": "cell35 ~ gene27"
        },
        {
          "x": 34,
          "y": 27,
          "value": 14,
          "name": "cell35 ~ gene28"
        },
        {
          "x": 34,
          "y": 28,
          "value": 0,
          "name": "cell35 ~ gene29"
        },
        {
          "x": 34,
          "y": 29,
          "value": 31,
          "name": "cell35 ~ gene30"
        },
        {
          "x": 34,
          "y": 30,
          "value": 33,
          "name": "cell35 ~ gene31"
        },
        {
          "x": 34,
          "y": 31,
          "value": 0,
          "name": "cell35 ~ gene32"
        },
        {
          "x": 34,
          "y": 32,
          "value": 13,
          "name": "cell35 ~ gene33"
        },
        {
          "x": 34,
          "y": 33,
          "value": 8,
          "name": "cell35 ~ gene34"
        },
        {
          "x": 34,
          "y": 34,
          "value": 18,
          "name": "cell35 ~ gene35"
        },
        {
          "x": 34,
          "y": 35,
          "value": 8,
          "name": "cell35 ~ gene36"
        },
        {
          "x": 34,
          "y": 36,
          "value": 22,
          "name": "cell35 ~ gene37"
        },
        {
          "x": 34,
          "y": 37,
          "value": 20,
          "name": "cell35 ~ gene38"
        },
        {
          "x": 34,
          "y": 38,
          "value": 0,
          "name": "cell35 ~ gene39"
        },
        {
          "x": 34,
          "y": 39,
          "value": 15,
          "name": "cell35 ~ gene40"
        },
        {
          "x": 34,
          "y": 40,
          "value": 21,
          "name": "cell35 ~ gene41"
        },
        {
          "x": 34,
          "y": 41,
          "value": 2,
          "name": "cell35 ~ gene42"
        },
        {
          "x": 34,
          "y": 42,
          "value": 0,
          "name": "cell35 ~ gene43"
        },
        {
          "x": 34,
          "y": 43,
          "value": 8,
          "name": "cell35 ~ gene44"
        },
        {
          "x": 34,
          "y": 44,
          "value": 12,
          "name": "cell35 ~ gene45"
        },
        {
          "x": 34,
          "y": 45,
          "value": 13,
          "name": "cell35 ~ gene46"
        },
        {
          "x": 34,
          "y": 46,
          "value": 4,
          "name": "cell35 ~ gene47"
        },
        {
          "x": 34,
          "y": 47,
          "value": 0,
          "name": "cell35 ~ gene48"
        },
        {
          "x": 34,
          "y": 48,
          "value": 3,
          "name": "cell35 ~ gene49"
        },
        {
          "x": 34,
          "y": 49,
          "value": 0,
          "name": "cell35 ~ gene50"
        },
        {
          "x": 34,
          "y": 50,
          "value": 15,
          "name": "cell35 ~ gene51"
        },
        {
          "x": 34,
          "y": 51,
          "value": 5,
          "name": "cell35 ~ gene52"
        },
        {
          "x": 34,
          "y": 52,
          "value": 11,
          "name": "cell35 ~ gene53"
        },
        {
          "x": 34,
          "y": 53,
          "value": 12,
          "name": "cell35 ~ gene54"
        },
        {
          "x": 34,
          "y": 54,
          "value": 0,
          "name": "cell35 ~ gene55"
        },
        {
          "x": 34,
          "y": 55,
          "value": 6,
          "name": "cell35 ~ gene56"
        },
        {
          "x": 34,
          "y": 56,
          "value": 1,
          "name": "cell35 ~ gene57"
        },
        {
          "x": 34,
          "y": 57,
          "value": 0,
          "name": "cell35 ~ gene58"
        },
        {
          "x": 34,
          "y": 58,
          "value": 0,
          "name": "cell35 ~ gene59"
        },
        {
          "x": 34,
          "y": 59,
          "value": 0,
          "name": "cell35 ~ gene60"
        },
        {
          "x": 34,
          "y": 60,
          "value": 0,
          "name": "cell35 ~ gene61"
        },
        {
          "x": 34,
          "y": 61,
          "value": 11,
          "name": "cell35 ~ gene62"
        },
        {
          "x": 34,
          "y": 62,
          "value": 0,
          "name": "cell35 ~ gene63"
        },
        {
          "x": 34,
          "y": 63,
          "value": 0,
          "name": "cell35 ~ gene64"
        },
        {
          "x": 34,
          "y": 64,
          "value": 10,
          "name": "cell35 ~ gene65"
        },
        {
          "x": 34,
          "y": 65,
          "value": 8,
          "name": "cell35 ~ gene66"
        },
        {
          "x": 34,
          "y": 66,
          "value": 3,
          "name": "cell35 ~ gene67"
        },
        {
          "x": 34,
          "y": 67,
          "value": 0,
          "name": "cell35 ~ gene68"
        },
        {
          "x": 34,
          "y": 68,
          "value": 12,
          "name": "cell35 ~ gene69"
        },
        {
          "x": 34,
          "y": 69,
          "value": 0,
          "name": "cell35 ~ gene70"
        },
        {
          "x": 34,
          "y": 70,
          "value": 0,
          "name": "cell35 ~ gene71"
        },
        {
          "x": 34,
          "y": 71,
          "value": 8,
          "name": "cell35 ~ gene72"
        },
        {
          "x": 34,
          "y": 72,
          "value": 0,
          "name": "cell35 ~ gene73"
        },
        {
          "x": 34,
          "y": 73,
          "value": 4,
          "name": "cell35 ~ gene74"
        },
        {
          "x": 34,
          "y": 74,
          "value": 0,
          "name": "cell35 ~ gene75"
        },
        {
          "x": 34,
          "y": 75,
          "value": 1,
          "name": "cell35 ~ gene76"
        },
        {
          "x": 34,
          "y": 76,
          "value": 0,
          "name": "cell35 ~ gene77"
        },
        {
          "x": 34,
          "y": 77,
          "value": 9,
          "name": "cell35 ~ gene78"
        },
        {
          "x": 34,
          "y": 78,
          "value": 0,
          "name": "cell35 ~ gene79"
        },
        {
          "x": 34,
          "y": 79,
          "value": 13,
          "name": "cell35 ~ gene80"
        },
        {
          "x": 34,
          "y": 80,
          "value": 0,
          "name": "cell35 ~ gene81"
        },
        {
          "x": 34,
          "y": 81,
          "value": 0,
          "name": "cell35 ~ gene82"
        },
        {
          "x": 34,
          "y": 82,
          "value": 13,
          "name": "cell35 ~ gene83"
        },
        {
          "x": 34,
          "y": 83,
          "value": 24,
          "name": "cell35 ~ gene84"
        },
        {
          "x": 34,
          "y": 84,
          "value": 0,
          "name": "cell35 ~ gene85"
        },
        {
          "x": 34,
          "y": 85,
          "value": 7,
          "name": "cell35 ~ gene86"
        },
        {
          "x": 34,
          "y": 86,
          "value": 0,
          "name": "cell35 ~ gene87"
        },
        {
          "x": 34,
          "y": 87,
          "value": 0,
          "name": "cell35 ~ gene88"
        },
        {
          "x": 34,
          "y": 88,
          "value": 0,
          "name": "cell35 ~ gene89"
        },
        {
          "x": 34,
          "y": 89,
          "value": 17,
          "name": "cell35 ~ gene90"
        },
        {
          "x": 34,
          "y": 90,
          "value": 14,
          "name": "cell35 ~ gene91"
        },
        {
          "x": 34,
          "y": 91,
          "value": 17,
          "name": "cell35 ~ gene92"
        },
        {
          "x": 34,
          "y": 92,
          "value": 12,
          "name": "cell35 ~ gene93"
        },
        {
          "x": 34,
          "y": 93,
          "value": 0,
          "name": "cell35 ~ gene94"
        },
        {
          "x": 34,
          "y": 94,
          "value": 0,
          "name": "cell35 ~ gene95"
        },
        {
          "x": 34,
          "y": 95,
          "value": 0,
          "name": "cell35 ~ gene96"
        },
        {
          "x": 34,
          "y": 96,
          "value": 13,
          "name": "cell35 ~ gene97"
        },
        {
          "x": 34,
          "y": 97,
          "value": 0,
          "name": "cell35 ~ gene98"
        },
        {
          "x": 34,
          "y": 98,
          "value": 19,
          "name": "cell35 ~ gene99"
        },
        {
          "x": 34,
          "y": 99,
          "value": 0,
          "name": "cell35 ~ gene100"
        },
        {
          "x": 35,
          "y": 0,
          "value": 0,
          "name": "cell36 ~ gene1"
        },
        {
          "x": 35,
          "y": 1,
          "value": 0,
          "name": "cell36 ~ gene2"
        },
        {
          "x": 35,
          "y": 2,
          "value": 3,
          "name": "cell36 ~ gene3"
        },
        {
          "x": 35,
          "y": 3,
          "value": 15,
          "name": "cell36 ~ gene4"
        },
        {
          "x": 35,
          "y": 4,
          "value": 10,
          "name": "cell36 ~ gene5"
        },
        {
          "x": 35,
          "y": 5,
          "value": 6,
          "name": "cell36 ~ gene6"
        },
        {
          "x": 35,
          "y": 6,
          "value": 9,
          "name": "cell36 ~ gene7"
        },
        {
          "x": 35,
          "y": 7,
          "value": 3,
          "name": "cell36 ~ gene8"
        },
        {
          "x": 35,
          "y": 8,
          "value": 0,
          "name": "cell36 ~ gene9"
        },
        {
          "x": 35,
          "y": 9,
          "value": 6,
          "name": "cell36 ~ gene10"
        },
        {
          "x": 35,
          "y": 10,
          "value": 9,
          "name": "cell36 ~ gene11"
        },
        {
          "x": 35,
          "y": 11,
          "value": 5,
          "name": "cell36 ~ gene12"
        },
        {
          "x": 35,
          "y": 12,
          "value": 0,
          "name": "cell36 ~ gene13"
        },
        {
          "x": 35,
          "y": 13,
          "value": 8,
          "name": "cell36 ~ gene14"
        },
        {
          "x": 35,
          "y": 14,
          "value": 0,
          "name": "cell36 ~ gene15"
        },
        {
          "x": 35,
          "y": 15,
          "value": 22,
          "name": "cell36 ~ gene16"
        },
        {
          "x": 35,
          "y": 16,
          "value": 0,
          "name": "cell36 ~ gene17"
        },
        {
          "x": 35,
          "y": 17,
          "value": 0,
          "name": "cell36 ~ gene18"
        },
        {
          "x": 35,
          "y": 18,
          "value": 16,
          "name": "cell36 ~ gene19"
        },
        {
          "x": 35,
          "y": 19,
          "value": 0,
          "name": "cell36 ~ gene20"
        },
        {
          "x": 35,
          "y": 20,
          "value": 0,
          "name": "cell36 ~ gene21"
        },
        {
          "x": 35,
          "y": 21,
          "value": 17,
          "name": "cell36 ~ gene22"
        },
        {
          "x": 35,
          "y": 22,
          "value": 20,
          "name": "cell36 ~ gene23"
        },
        {
          "x": 35,
          "y": 23,
          "value": 0,
          "name": "cell36 ~ gene24"
        },
        {
          "x": 35,
          "y": 24,
          "value": 6,
          "name": "cell36 ~ gene25"
        },
        {
          "x": 35,
          "y": 25,
          "value": 4,
          "name": "cell36 ~ gene26"
        },
        {
          "x": 35,
          "y": 26,
          "value": 22,
          "name": "cell36 ~ gene27"
        },
        {
          "x": 35,
          "y": 27,
          "value": 0,
          "name": "cell36 ~ gene28"
        },
        {
          "x": 35,
          "y": 28,
          "value": 0,
          "name": "cell36 ~ gene29"
        },
        {
          "x": 35,
          "y": 29,
          "value": 28,
          "name": "cell36 ~ gene30"
        },
        {
          "x": 35,
          "y": 30,
          "value": 18,
          "name": "cell36 ~ gene31"
        },
        {
          "x": 35,
          "y": 31,
          "value": 0,
          "name": "cell36 ~ gene32"
        },
        {
          "x": 35,
          "y": 32,
          "value": 1,
          "name": "cell36 ~ gene33"
        },
        {
          "x": 35,
          "y": 33,
          "value": 0,
          "name": "cell36 ~ gene34"
        },
        {
          "x": 35,
          "y": 34,
          "value": 12,
          "name": "cell36 ~ gene35"
        },
        {
          "x": 35,
          "y": 35,
          "value": 0,
          "name": "cell36 ~ gene36"
        },
        {
          "x": 35,
          "y": 36,
          "value": 30,
          "name": "cell36 ~ gene37"
        },
        {
          "x": 35,
          "y": 37,
          "value": 6,
          "name": "cell36 ~ gene38"
        },
        {
          "x": 35,
          "y": 38,
          "value": 0,
          "name": "cell36 ~ gene39"
        },
        {
          "x": 35,
          "y": 39,
          "value": 7,
          "name": "cell36 ~ gene40"
        },
        {
          "x": 35,
          "y": 40,
          "value": 2,
          "name": "cell36 ~ gene41"
        },
        {
          "x": 35,
          "y": 41,
          "value": 0,
          "name": "cell36 ~ gene42"
        },
        {
          "x": 35,
          "y": 42,
          "value": 9,
          "name": "cell36 ~ gene43"
        },
        {
          "x": 35,
          "y": 43,
          "value": 2,
          "name": "cell36 ~ gene44"
        },
        {
          "x": 35,
          "y": 44,
          "value": 0,
          "name": "cell36 ~ gene45"
        },
        {
          "x": 35,
          "y": 45,
          "value": 9,
          "name": "cell36 ~ gene46"
        },
        {
          "x": 35,
          "y": 46,
          "value": 11,
          "name": "cell36 ~ gene47"
        },
        {
          "x": 35,
          "y": 47,
          "value": 0,
          "name": "cell36 ~ gene48"
        },
        {
          "x": 35,
          "y": 48,
          "value": 0,
          "name": "cell36 ~ gene49"
        },
        {
          "x": 35,
          "y": 49,
          "value": 0,
          "name": "cell36 ~ gene50"
        },
        {
          "x": 35,
          "y": 50,
          "value": 5,
          "name": "cell36 ~ gene51"
        },
        {
          "x": 35,
          "y": 51,
          "value": 2,
          "name": "cell36 ~ gene52"
        },
        {
          "x": 35,
          "y": 52,
          "value": 0,
          "name": "cell36 ~ gene53"
        },
        {
          "x": 35,
          "y": 53,
          "value": 0,
          "name": "cell36 ~ gene54"
        },
        {
          "x": 35,
          "y": 54,
          "value": 5,
          "name": "cell36 ~ gene55"
        },
        {
          "x": 35,
          "y": 55,
          "value": 0,
          "name": "cell36 ~ gene56"
        },
        {
          "x": 35,
          "y": 56,
          "value": 0,
          "name": "cell36 ~ gene57"
        },
        {
          "x": 35,
          "y": 57,
          "value": 0,
          "name": "cell36 ~ gene58"
        },
        {
          "x": 35,
          "y": 58,
          "value": 0,
          "name": "cell36 ~ gene59"
        },
        {
          "x": 35,
          "y": 59,
          "value": 8,
          "name": "cell36 ~ gene60"
        },
        {
          "x": 35,
          "y": 60,
          "value": 0,
          "name": "cell36 ~ gene61"
        },
        {
          "x": 35,
          "y": 61,
          "value": 4,
          "name": "cell36 ~ gene62"
        },
        {
          "x": 35,
          "y": 62,
          "value": 0,
          "name": "cell36 ~ gene63"
        },
        {
          "x": 35,
          "y": 63,
          "value": 11,
          "name": "cell36 ~ gene64"
        },
        {
          "x": 35,
          "y": 64,
          "value": 2,
          "name": "cell36 ~ gene65"
        },
        {
          "x": 35,
          "y": 65,
          "value": 2,
          "name": "cell36 ~ gene66"
        },
        {
          "x": 35,
          "y": 66,
          "value": 4,
          "name": "cell36 ~ gene67"
        },
        {
          "x": 35,
          "y": 67,
          "value": 0,
          "name": "cell36 ~ gene68"
        },
        {
          "x": 35,
          "y": 68,
          "value": 7,
          "name": "cell36 ~ gene69"
        },
        {
          "x": 35,
          "y": 69,
          "value": 0,
          "name": "cell36 ~ gene70"
        },
        {
          "x": 35,
          "y": 70,
          "value": 0,
          "name": "cell36 ~ gene71"
        },
        {
          "x": 35,
          "y": 71,
          "value": 18,
          "name": "cell36 ~ gene72"
        },
        {
          "x": 35,
          "y": 72,
          "value": 11,
          "name": "cell36 ~ gene73"
        },
        {
          "x": 35,
          "y": 73,
          "value": 9,
          "name": "cell36 ~ gene74"
        },
        {
          "x": 35,
          "y": 74,
          "value": 5,
          "name": "cell36 ~ gene75"
        },
        {
          "x": 35,
          "y": 75,
          "value": 0,
          "name": "cell36 ~ gene76"
        },
        {
          "x": 35,
          "y": 76,
          "value": 4,
          "name": "cell36 ~ gene77"
        },
        {
          "x": 35,
          "y": 77,
          "value": 0,
          "name": "cell36 ~ gene78"
        },
        {
          "x": 35,
          "y": 78,
          "value": 0,
          "name": "cell36 ~ gene79"
        },
        {
          "x": 35,
          "y": 79,
          "value": 0,
          "name": "cell36 ~ gene80"
        },
        {
          "x": 35,
          "y": 80,
          "value": 0,
          "name": "cell36 ~ gene81"
        },
        {
          "x": 35,
          "y": 81,
          "value": 0,
          "name": "cell36 ~ gene82"
        },
        {
          "x": 35,
          "y": 82,
          "value": 20,
          "name": "cell36 ~ gene83"
        },
        {
          "x": 35,
          "y": 83,
          "value": 7,
          "name": "cell36 ~ gene84"
        },
        {
          "x": 35,
          "y": 84,
          "value": 0,
          "name": "cell36 ~ gene85"
        },
        {
          "x": 35,
          "y": 85,
          "value": 10,
          "name": "cell36 ~ gene86"
        },
        {
          "x": 35,
          "y": 86,
          "value": 7,
          "name": "cell36 ~ gene87"
        },
        {
          "x": 35,
          "y": 87,
          "value": 0,
          "name": "cell36 ~ gene88"
        },
        {
          "x": 35,
          "y": 88,
          "value": 0,
          "name": "cell36 ~ gene89"
        },
        {
          "x": 35,
          "y": 89,
          "value": 0,
          "name": "cell36 ~ gene90"
        },
        {
          "x": 35,
          "y": 90,
          "value": 15,
          "name": "cell36 ~ gene91"
        },
        {
          "x": 35,
          "y": 91,
          "value": 3,
          "name": "cell36 ~ gene92"
        },
        {
          "x": 35,
          "y": 92,
          "value": 0,
          "name": "cell36 ~ gene93"
        },
        {
          "x": 35,
          "y": 93,
          "value": 15,
          "name": "cell36 ~ gene94"
        },
        {
          "x": 35,
          "y": 94,
          "value": 18,
          "name": "cell36 ~ gene95"
        },
        {
          "x": 35,
          "y": 95,
          "value": 0,
          "name": "cell36 ~ gene96"
        },
        {
          "x": 35,
          "y": 96,
          "value": 0,
          "name": "cell36 ~ gene97"
        },
        {
          "x": 35,
          "y": 97,
          "value": 0,
          "name": "cell36 ~ gene98"
        },
        {
          "x": 35,
          "y": 98,
          "value": 0,
          "name": "cell36 ~ gene99"
        },
        {
          "x": 35,
          "y": 99,
          "value": 0,
          "name": "cell36 ~ gene100"
        },
        {
          "x": 36,
          "y": 0,
          "value": 0,
          "name": "cell37 ~ gene1"
        },
        {
          "x": 36,
          "y": 1,
          "value": 0,
          "name": "cell37 ~ gene2"
        },
        {
          "x": 36,
          "y": 2,
          "value": 10,
          "name": "cell37 ~ gene3"
        },
        {
          "x": 36,
          "y": 3,
          "value": 19,
          "name": "cell37 ~ gene4"
        },
        {
          "x": 36,
          "y": 4,
          "value": 19,
          "name": "cell37 ~ gene5"
        },
        {
          "x": 36,
          "y": 5,
          "value": 8,
          "name": "cell37 ~ gene6"
        },
        {
          "x": 36,
          "y": 6,
          "value": 0,
          "name": "cell37 ~ gene7"
        },
        {
          "x": 36,
          "y": 7,
          "value": 0,
          "name": "cell37 ~ gene8"
        },
        {
          "x": 36,
          "y": 8,
          "value": 0,
          "name": "cell37 ~ gene9"
        },
        {
          "x": 36,
          "y": 9,
          "value": 0,
          "name": "cell37 ~ gene10"
        },
        {
          "x": 36,
          "y": 10,
          "value": 0,
          "name": "cell37 ~ gene11"
        },
        {
          "x": 36,
          "y": 11,
          "value": 18,
          "name": "cell37 ~ gene12"
        },
        {
          "x": 36,
          "y": 12,
          "value": 2,
          "name": "cell37 ~ gene13"
        },
        {
          "x": 36,
          "y": 13,
          "value": 19,
          "name": "cell37 ~ gene14"
        },
        {
          "x": 36,
          "y": 14,
          "value": 1,
          "name": "cell37 ~ gene15"
        },
        {
          "x": 36,
          "y": 15,
          "value": 0,
          "name": "cell37 ~ gene16"
        },
        {
          "x": 36,
          "y": 16,
          "value": 0,
          "name": "cell37 ~ gene17"
        },
        {
          "x": 36,
          "y": 17,
          "value": 2,
          "name": "cell37 ~ gene18"
        },
        {
          "x": 36,
          "y": 18,
          "value": 0,
          "name": "cell37 ~ gene19"
        },
        {
          "x": 36,
          "y": 19,
          "value": 0,
          "name": "cell37 ~ gene20"
        },
        {
          "x": 36,
          "y": 20,
          "value": 0,
          "name": "cell37 ~ gene21"
        },
        {
          "x": 36,
          "y": 21,
          "value": 52,
          "name": "cell37 ~ gene22"
        },
        {
          "x": 36,
          "y": 22,
          "value": 0,
          "name": "cell37 ~ gene23"
        },
        {
          "x": 36,
          "y": 23,
          "value": 0,
          "name": "cell37 ~ gene24"
        },
        {
          "x": 36,
          "y": 24,
          "value": 10,
          "name": "cell37 ~ gene25"
        },
        {
          "x": 36,
          "y": 25,
          "value": 11,
          "name": "cell37 ~ gene26"
        },
        {
          "x": 36,
          "y": 26,
          "value": 8,
          "name": "cell37 ~ gene27"
        },
        {
          "x": 36,
          "y": 27,
          "value": 0,
          "name": "cell37 ~ gene28"
        },
        {
          "x": 36,
          "y": 28,
          "value": 8,
          "name": "cell37 ~ gene29"
        },
        {
          "x": 36,
          "y": 29,
          "value": 24,
          "name": "cell37 ~ gene30"
        },
        {
          "x": 36,
          "y": 30,
          "value": 43,
          "name": "cell37 ~ gene31"
        },
        {
          "x": 36,
          "y": 31,
          "value": 9,
          "name": "cell37 ~ gene32"
        },
        {
          "x": 36,
          "y": 32,
          "value": 0,
          "name": "cell37 ~ gene33"
        },
        {
          "x": 36,
          "y": 33,
          "value": 6,
          "name": "cell37 ~ gene34"
        },
        {
          "x": 36,
          "y": 34,
          "value": 22,
          "name": "cell37 ~ gene35"
        },
        {
          "x": 36,
          "y": 35,
          "value": 0,
          "name": "cell37 ~ gene36"
        },
        {
          "x": 36,
          "y": 36,
          "value": 37,
          "name": "cell37 ~ gene37"
        },
        {
          "x": 36,
          "y": 37,
          "value": 0,
          "name": "cell37 ~ gene38"
        },
        {
          "x": 36,
          "y": 38,
          "value": 0,
          "name": "cell37 ~ gene39"
        },
        {
          "x": 36,
          "y": 39,
          "value": 10,
          "name": "cell37 ~ gene40"
        },
        {
          "x": 36,
          "y": 40,
          "value": 0,
          "name": "cell37 ~ gene41"
        },
        {
          "x": 36,
          "y": 41,
          "value": 0,
          "name": "cell37 ~ gene42"
        },
        {
          "x": 36,
          "y": 42,
          "value": 0,
          "name": "cell37 ~ gene43"
        },
        {
          "x": 36,
          "y": 43,
          "value": 0,
          "name": "cell37 ~ gene44"
        },
        {
          "x": 36,
          "y": 44,
          "value": 4,
          "name": "cell37 ~ gene45"
        },
        {
          "x": 36,
          "y": 45,
          "value": 0,
          "name": "cell37 ~ gene46"
        },
        {
          "x": 36,
          "y": 46,
          "value": 11,
          "name": "cell37 ~ gene47"
        },
        {
          "x": 36,
          "y": 47,
          "value": 7,
          "name": "cell37 ~ gene48"
        },
        {
          "x": 36,
          "y": 48,
          "value": 0,
          "name": "cell37 ~ gene49"
        },
        {
          "x": 36,
          "y": 49,
          "value": 0,
          "name": "cell37 ~ gene50"
        },
        {
          "x": 36,
          "y": 50,
          "value": 6,
          "name": "cell37 ~ gene51"
        },
        {
          "x": 36,
          "y": 51,
          "value": 2,
          "name": "cell37 ~ gene52"
        },
        {
          "x": 36,
          "y": 52,
          "value": 2,
          "name": "cell37 ~ gene53"
        },
        {
          "x": 36,
          "y": 53,
          "value": 0,
          "name": "cell37 ~ gene54"
        },
        {
          "x": 36,
          "y": 54,
          "value": 15,
          "name": "cell37 ~ gene55"
        },
        {
          "x": 36,
          "y": 55,
          "value": 0,
          "name": "cell37 ~ gene56"
        },
        {
          "x": 36,
          "y": 56,
          "value": 18,
          "name": "cell37 ~ gene57"
        },
        {
          "x": 36,
          "y": 57,
          "value": 4,
          "name": "cell37 ~ gene58"
        },
        {
          "x": 36,
          "y": 58,
          "value": 3,
          "name": "cell37 ~ gene59"
        },
        {
          "x": 36,
          "y": 59,
          "value": 0,
          "name": "cell37 ~ gene60"
        },
        {
          "x": 36,
          "y": 60,
          "value": 0,
          "name": "cell37 ~ gene61"
        },
        {
          "x": 36,
          "y": 61,
          "value": 25,
          "name": "cell37 ~ gene62"
        },
        {
          "x": 36,
          "y": 62,
          "value": 0,
          "name": "cell37 ~ gene63"
        },
        {
          "x": 36,
          "y": 63,
          "value": 0,
          "name": "cell37 ~ gene64"
        },
        {
          "x": 36,
          "y": 64,
          "value": 0,
          "name": "cell37 ~ gene65"
        },
        {
          "x": 36,
          "y": 65,
          "value": 0,
          "name": "cell37 ~ gene66"
        },
        {
          "x": 36,
          "y": 66,
          "value": 0,
          "name": "cell37 ~ gene67"
        },
        {
          "x": 36,
          "y": 67,
          "value": 10,
          "name": "cell37 ~ gene68"
        },
        {
          "x": 36,
          "y": 68,
          "value": 13,
          "name": "cell37 ~ gene69"
        },
        {
          "x": 36,
          "y": 69,
          "value": 3,
          "name": "cell37 ~ gene70"
        },
        {
          "x": 36,
          "y": 70,
          "value": 0,
          "name": "cell37 ~ gene71"
        },
        {
          "x": 36,
          "y": 71,
          "value": 0,
          "name": "cell37 ~ gene72"
        },
        {
          "x": 36,
          "y": 72,
          "value": 18,
          "name": "cell37 ~ gene73"
        },
        {
          "x": 36,
          "y": 73,
          "value": 0,
          "name": "cell37 ~ gene74"
        },
        {
          "x": 36,
          "y": 74,
          "value": 0,
          "name": "cell37 ~ gene75"
        },
        {
          "x": 36,
          "y": 75,
          "value": 6,
          "name": "cell37 ~ gene76"
        },
        {
          "x": 36,
          "y": 76,
          "value": 18,
          "name": "cell37 ~ gene77"
        },
        {
          "x": 36,
          "y": 77,
          "value": 2,
          "name": "cell37 ~ gene78"
        },
        {
          "x": 36,
          "y": 78,
          "value": 0,
          "name": "cell37 ~ gene79"
        },
        {
          "x": 36,
          "y": 79,
          "value": 0,
          "name": "cell37 ~ gene80"
        },
        {
          "x": 36,
          "y": 80,
          "value": 20,
          "name": "cell37 ~ gene81"
        },
        {
          "x": 36,
          "y": 81,
          "value": 0,
          "name": "cell37 ~ gene82"
        },
        {
          "x": 36,
          "y": 82,
          "value": 2,
          "name": "cell37 ~ gene83"
        },
        {
          "x": 36,
          "y": 83,
          "value": 6,
          "name": "cell37 ~ gene84"
        },
        {
          "x": 36,
          "y": 84,
          "value": 5,
          "name": "cell37 ~ gene85"
        },
        {
          "x": 36,
          "y": 85,
          "value": 0,
          "name": "cell37 ~ gene86"
        },
        {
          "x": 36,
          "y": 86,
          "value": 5,
          "name": "cell37 ~ gene87"
        },
        {
          "x": 36,
          "y": 87,
          "value": 11,
          "name": "cell37 ~ gene88"
        },
        {
          "x": 36,
          "y": 88,
          "value": 0,
          "name": "cell37 ~ gene89"
        },
        {
          "x": 36,
          "y": 89,
          "value": 1,
          "name": "cell37 ~ gene90"
        },
        {
          "x": 36,
          "y": 90,
          "value": 23,
          "name": "cell37 ~ gene91"
        },
        {
          "x": 36,
          "y": 91,
          "value": 17,
          "name": "cell37 ~ gene92"
        },
        {
          "x": 36,
          "y": 92,
          "value": 0,
          "name": "cell37 ~ gene93"
        },
        {
          "x": 36,
          "y": 93,
          "value": 11,
          "name": "cell37 ~ gene94"
        },
        {
          "x": 36,
          "y": 94,
          "value": 0,
          "name": "cell37 ~ gene95"
        },
        {
          "x": 36,
          "y": 95,
          "value": 2,
          "name": "cell37 ~ gene96"
        },
        {
          "x": 36,
          "y": 96,
          "value": 3,
          "name": "cell37 ~ gene97"
        },
        {
          "x": 36,
          "y": 97,
          "value": 11,
          "name": "cell37 ~ gene98"
        },
        {
          "x": 36,
          "y": 98,
          "value": 14,
          "name": "cell37 ~ gene99"
        },
        {
          "x": 36,
          "y": 99,
          "value": 4,
          "name": "cell37 ~ gene100"
        },
        {
          "x": 37,
          "y": 0,
          "value": 18,
          "name": "cell38 ~ gene1"
        },
        {
          "x": 37,
          "y": 1,
          "value": 4,
          "name": "cell38 ~ gene2"
        },
        {
          "x": 37,
          "y": 2,
          "value": 8,
          "name": "cell38 ~ gene3"
        },
        {
          "x": 37,
          "y": 3,
          "value": 4,
          "name": "cell38 ~ gene4"
        },
        {
          "x": 37,
          "y": 4,
          "value": 12,
          "name": "cell38 ~ gene5"
        },
        {
          "x": 37,
          "y": 5,
          "value": 28,
          "name": "cell38 ~ gene6"
        },
        {
          "x": 37,
          "y": 6,
          "value": 10,
          "name": "cell38 ~ gene7"
        },
        {
          "x": 37,
          "y": 7,
          "value": 0,
          "name": "cell38 ~ gene8"
        },
        {
          "x": 37,
          "y": 8,
          "value": 0,
          "name": "cell38 ~ gene9"
        },
        {
          "x": 37,
          "y": 9,
          "value": 5,
          "name": "cell38 ~ gene10"
        },
        {
          "x": 37,
          "y": 10,
          "value": 0,
          "name": "cell38 ~ gene11"
        },
        {
          "x": 37,
          "y": 11,
          "value": 0,
          "name": "cell38 ~ gene12"
        },
        {
          "x": 37,
          "y": 12,
          "value": 0,
          "name": "cell38 ~ gene13"
        },
        {
          "x": 37,
          "y": 13,
          "value": 8,
          "name": "cell38 ~ gene14"
        },
        {
          "x": 37,
          "y": 14,
          "value": 1,
          "name": "cell38 ~ gene15"
        },
        {
          "x": 37,
          "y": 15,
          "value": 8,
          "name": "cell38 ~ gene16"
        },
        {
          "x": 37,
          "y": 16,
          "value": 1,
          "name": "cell38 ~ gene17"
        },
        {
          "x": 37,
          "y": 17,
          "value": 0,
          "name": "cell38 ~ gene18"
        },
        {
          "x": 37,
          "y": 18,
          "value": 11,
          "name": "cell38 ~ gene19"
        },
        {
          "x": 37,
          "y": 19,
          "value": 0,
          "name": "cell38 ~ gene20"
        },
        {
          "x": 37,
          "y": 20,
          "value": 16,
          "name": "cell38 ~ gene21"
        },
        {
          "x": 37,
          "y": 21,
          "value": 20,
          "name": "cell38 ~ gene22"
        },
        {
          "x": 37,
          "y": 22,
          "value": 5,
          "name": "cell38 ~ gene23"
        },
        {
          "x": 37,
          "y": 23,
          "value": 0,
          "name": "cell38 ~ gene24"
        },
        {
          "x": 37,
          "y": 24,
          "value": 0,
          "name": "cell38 ~ gene25"
        },
        {
          "x": 37,
          "y": 25,
          "value": 8,
          "name": "cell38 ~ gene26"
        },
        {
          "x": 37,
          "y": 26,
          "value": 10,
          "name": "cell38 ~ gene27"
        },
        {
          "x": 37,
          "y": 27,
          "value": 0,
          "name": "cell38 ~ gene28"
        },
        {
          "x": 37,
          "y": 28,
          "value": 4,
          "name": "cell38 ~ gene29"
        },
        {
          "x": 37,
          "y": 29,
          "value": 21,
          "name": "cell38 ~ gene30"
        },
        {
          "x": 37,
          "y": 30,
          "value": 40,
          "name": "cell38 ~ gene31"
        },
        {
          "x": 37,
          "y": 31,
          "value": 3,
          "name": "cell38 ~ gene32"
        },
        {
          "x": 37,
          "y": 32,
          "value": 6,
          "name": "cell38 ~ gene33"
        },
        {
          "x": 37,
          "y": 33,
          "value": 8,
          "name": "cell38 ~ gene34"
        },
        {
          "x": 37,
          "y": 34,
          "value": 15,
          "name": "cell38 ~ gene35"
        },
        {
          "x": 37,
          "y": 35,
          "value": 0,
          "name": "cell38 ~ gene36"
        },
        {
          "x": 37,
          "y": 36,
          "value": 22,
          "name": "cell38 ~ gene37"
        },
        {
          "x": 37,
          "y": 37,
          "value": 4,
          "name": "cell38 ~ gene38"
        },
        {
          "x": 37,
          "y": 38,
          "value": 0,
          "name": "cell38 ~ gene39"
        },
        {
          "x": 37,
          "y": 39,
          "value": 1,
          "name": "cell38 ~ gene40"
        },
        {
          "x": 37,
          "y": 40,
          "value": 0,
          "name": "cell38 ~ gene41"
        },
        {
          "x": 37,
          "y": 41,
          "value": 4,
          "name": "cell38 ~ gene42"
        },
        {
          "x": 37,
          "y": 42,
          "value": 0,
          "name": "cell38 ~ gene43"
        },
        {
          "x": 37,
          "y": 43,
          "value": 10,
          "name": "cell38 ~ gene44"
        },
        {
          "x": 37,
          "y": 44,
          "value": 0,
          "name": "cell38 ~ gene45"
        },
        {
          "x": 37,
          "y": 45,
          "value": 0,
          "name": "cell38 ~ gene46"
        },
        {
          "x": 37,
          "y": 46,
          "value": 3,
          "name": "cell38 ~ gene47"
        },
        {
          "x": 37,
          "y": 47,
          "value": 3,
          "name": "cell38 ~ gene48"
        },
        {
          "x": 37,
          "y": 48,
          "value": 5,
          "name": "cell38 ~ gene49"
        },
        {
          "x": 37,
          "y": 49,
          "value": 0,
          "name": "cell38 ~ gene50"
        },
        {
          "x": 37,
          "y": 50,
          "value": 0,
          "name": "cell38 ~ gene51"
        },
        {
          "x": 37,
          "y": 51,
          "value": 4,
          "name": "cell38 ~ gene52"
        },
        {
          "x": 37,
          "y": 52,
          "value": 0,
          "name": "cell38 ~ gene53"
        },
        {
          "x": 37,
          "y": 53,
          "value": 7,
          "name": "cell38 ~ gene54"
        },
        {
          "x": 37,
          "y": 54,
          "value": 0,
          "name": "cell38 ~ gene55"
        },
        {
          "x": 37,
          "y": 55,
          "value": 0,
          "name": "cell38 ~ gene56"
        },
        {
          "x": 37,
          "y": 56,
          "value": 0,
          "name": "cell38 ~ gene57"
        },
        {
          "x": 37,
          "y": 57,
          "value": 0,
          "name": "cell38 ~ gene58"
        },
        {
          "x": 37,
          "y": 58,
          "value": 0,
          "name": "cell38 ~ gene59"
        },
        {
          "x": 37,
          "y": 59,
          "value": 0,
          "name": "cell38 ~ gene60"
        },
        {
          "x": 37,
          "y": 60,
          "value": 0,
          "name": "cell38 ~ gene61"
        },
        {
          "x": 37,
          "y": 61,
          "value": 0,
          "name": "cell38 ~ gene62"
        },
        {
          "x": 37,
          "y": 62,
          "value": 0,
          "name": "cell38 ~ gene63"
        },
        {
          "x": 37,
          "y": 63,
          "value": 7,
          "name": "cell38 ~ gene64"
        },
        {
          "x": 37,
          "y": 64,
          "value": 6,
          "name": "cell38 ~ gene65"
        },
        {
          "x": 37,
          "y": 65,
          "value": 0,
          "name": "cell38 ~ gene66"
        },
        {
          "x": 37,
          "y": 66,
          "value": 15,
          "name": "cell38 ~ gene67"
        },
        {
          "x": 37,
          "y": 67,
          "value": 10,
          "name": "cell38 ~ gene68"
        },
        {
          "x": 37,
          "y": 68,
          "value": 17,
          "name": "cell38 ~ gene69"
        },
        {
          "x": 37,
          "y": 69,
          "value": 5,
          "name": "cell38 ~ gene70"
        },
        {
          "x": 37,
          "y": 70,
          "value": 6,
          "name": "cell38 ~ gene71"
        },
        {
          "x": 37,
          "y": 71,
          "value": 0,
          "name": "cell38 ~ gene72"
        },
        {
          "x": 37,
          "y": 72,
          "value": 2,
          "name": "cell38 ~ gene73"
        },
        {
          "x": 37,
          "y": 73,
          "value": 14,
          "name": "cell38 ~ gene74"
        },
        {
          "x": 37,
          "y": 74,
          "value": 5,
          "name": "cell38 ~ gene75"
        },
        {
          "x": 37,
          "y": 75,
          "value": 3,
          "name": "cell38 ~ gene76"
        },
        {
          "x": 37,
          "y": 76,
          "value": 5,
          "name": "cell38 ~ gene77"
        },
        {
          "x": 37,
          "y": 77,
          "value": 0,
          "name": "cell38 ~ gene78"
        },
        {
          "x": 37,
          "y": 78,
          "value": 0,
          "name": "cell38 ~ gene79"
        },
        {
          "x": 37,
          "y": 79,
          "value": 0,
          "name": "cell38 ~ gene80"
        },
        {
          "x": 37,
          "y": 80,
          "value": 0,
          "name": "cell38 ~ gene81"
        },
        {
          "x": 37,
          "y": 81,
          "value": 0,
          "name": "cell38 ~ gene82"
        },
        {
          "x": 37,
          "y": 82,
          "value": 0,
          "name": "cell38 ~ gene83"
        },
        {
          "x": 37,
          "y": 83,
          "value": 17,
          "name": "cell38 ~ gene84"
        },
        {
          "x": 37,
          "y": 84,
          "value": 15,
          "name": "cell38 ~ gene85"
        },
        {
          "x": 37,
          "y": 85,
          "value": 0,
          "name": "cell38 ~ gene86"
        },
        {
          "x": 37,
          "y": 86,
          "value": 0,
          "name": "cell38 ~ gene87"
        },
        {
          "x": 37,
          "y": 87,
          "value": 12,
          "name": "cell38 ~ gene88"
        },
        {
          "x": 37,
          "y": 88,
          "value": 0,
          "name": "cell38 ~ gene89"
        },
        {
          "x": 37,
          "y": 89,
          "value": 0,
          "name": "cell38 ~ gene90"
        },
        {
          "x": 37,
          "y": 90,
          "value": 6,
          "name": "cell38 ~ gene91"
        },
        {
          "x": 37,
          "y": 91,
          "value": 14,
          "name": "cell38 ~ gene92"
        },
        {
          "x": 37,
          "y": 92,
          "value": 0,
          "name": "cell38 ~ gene93"
        },
        {
          "x": 37,
          "y": 93,
          "value": 0,
          "name": "cell38 ~ gene94"
        },
        {
          "x": 37,
          "y": 94,
          "value": 0,
          "name": "cell38 ~ gene95"
        },
        {
          "x": 37,
          "y": 95,
          "value": 0,
          "name": "cell38 ~ gene96"
        },
        {
          "x": 37,
          "y": 96,
          "value": 0,
          "name": "cell38 ~ gene97"
        },
        {
          "x": 37,
          "y": 97,
          "value": 0,
          "name": "cell38 ~ gene98"
        },
        {
          "x": 37,
          "y": 98,
          "value": 4,
          "name": "cell38 ~ gene99"
        },
        {
          "x": 37,
          "y": 99,
          "value": 0,
          "name": "cell38 ~ gene100"
        },
        {
          "x": 38,
          "y": 0,
          "value": 0,
          "name": "cell39 ~ gene1"
        },
        {
          "x": 38,
          "y": 1,
          "value": 3,
          "name": "cell39 ~ gene2"
        },
        {
          "x": 38,
          "y": 2,
          "value": 1,
          "name": "cell39 ~ gene3"
        },
        {
          "x": 38,
          "y": 3,
          "value": 0,
          "name": "cell39 ~ gene4"
        },
        {
          "x": 38,
          "y": 4,
          "value": 0,
          "name": "cell39 ~ gene5"
        },
        {
          "x": 38,
          "y": 5,
          "value": 0,
          "name": "cell39 ~ gene6"
        },
        {
          "x": 38,
          "y": 6,
          "value": 8,
          "name": "cell39 ~ gene7"
        },
        {
          "x": 38,
          "y": 7,
          "value": 0,
          "name": "cell39 ~ gene8"
        },
        {
          "x": 38,
          "y": 8,
          "value": 1,
          "name": "cell39 ~ gene9"
        },
        {
          "x": 38,
          "y": 9,
          "value": 4,
          "name": "cell39 ~ gene10"
        },
        {
          "x": 38,
          "y": 10,
          "value": 14,
          "name": "cell39 ~ gene11"
        },
        {
          "x": 38,
          "y": 11,
          "value": 0,
          "name": "cell39 ~ gene12"
        },
        {
          "x": 38,
          "y": 12,
          "value": 10,
          "name": "cell39 ~ gene13"
        },
        {
          "x": 38,
          "y": 13,
          "value": 0,
          "name": "cell39 ~ gene14"
        },
        {
          "x": 38,
          "y": 14,
          "value": 13,
          "name": "cell39 ~ gene15"
        },
        {
          "x": 38,
          "y": 15,
          "value": 0,
          "name": "cell39 ~ gene16"
        },
        {
          "x": 38,
          "y": 16,
          "value": 0,
          "name": "cell39 ~ gene17"
        },
        {
          "x": 38,
          "y": 17,
          "value": 17,
          "name": "cell39 ~ gene18"
        },
        {
          "x": 38,
          "y": 18,
          "value": 11,
          "name": "cell39 ~ gene19"
        },
        {
          "x": 38,
          "y": 19,
          "value": 0,
          "name": "cell39 ~ gene20"
        },
        {
          "x": 38,
          "y": 20,
          "value": 0,
          "name": "cell39 ~ gene21"
        },
        {
          "x": 38,
          "y": 21,
          "value": 22,
          "name": "cell39 ~ gene22"
        },
        {
          "x": 38,
          "y": 22,
          "value": 1,
          "name": "cell39 ~ gene23"
        },
        {
          "x": 38,
          "y": 23,
          "value": 1,
          "name": "cell39 ~ gene24"
        },
        {
          "x": 38,
          "y": 24,
          "value": 21,
          "name": "cell39 ~ gene25"
        },
        {
          "x": 38,
          "y": 25,
          "value": 22,
          "name": "cell39 ~ gene26"
        },
        {
          "x": 38,
          "y": 26,
          "value": 5,
          "name": "cell39 ~ gene27"
        },
        {
          "x": 38,
          "y": 27,
          "value": 16,
          "name": "cell39 ~ gene28"
        },
        {
          "x": 38,
          "y": 28,
          "value": 13,
          "name": "cell39 ~ gene29"
        },
        {
          "x": 38,
          "y": 29,
          "value": 41,
          "name": "cell39 ~ gene30"
        },
        {
          "x": 38,
          "y": 30,
          "value": 24,
          "name": "cell39 ~ gene31"
        },
        {
          "x": 38,
          "y": 31,
          "value": 4,
          "name": "cell39 ~ gene32"
        },
        {
          "x": 38,
          "y": 32,
          "value": 0,
          "name": "cell39 ~ gene33"
        },
        {
          "x": 38,
          "y": 33,
          "value": 0,
          "name": "cell39 ~ gene34"
        },
        {
          "x": 38,
          "y": 34,
          "value": 11,
          "name": "cell39 ~ gene35"
        },
        {
          "x": 38,
          "y": 35,
          "value": 10,
          "name": "cell39 ~ gene36"
        },
        {
          "x": 38,
          "y": 36,
          "value": 12,
          "name": "cell39 ~ gene37"
        },
        {
          "x": 38,
          "y": 37,
          "value": 3,
          "name": "cell39 ~ gene38"
        },
        {
          "x": 38,
          "y": 38,
          "value": 6,
          "name": "cell39 ~ gene39"
        },
        {
          "x": 38,
          "y": 39,
          "value": 7,
          "name": "cell39 ~ gene40"
        },
        {
          "x": 38,
          "y": 40,
          "value": 0,
          "name": "cell39 ~ gene41"
        },
        {
          "x": 38,
          "y": 41,
          "value": 12,
          "name": "cell39 ~ gene42"
        },
        {
          "x": 38,
          "y": 42,
          "value": 1,
          "name": "cell39 ~ gene43"
        },
        {
          "x": 38,
          "y": 43,
          "value": 1,
          "name": "cell39 ~ gene44"
        },
        {
          "x": 38,
          "y": 44,
          "value": 4,
          "name": "cell39 ~ gene45"
        },
        {
          "x": 38,
          "y": 45,
          "value": 0,
          "name": "cell39 ~ gene46"
        },
        {
          "x": 38,
          "y": 46,
          "value": 4,
          "name": "cell39 ~ gene47"
        },
        {
          "x": 38,
          "y": 47,
          "value": 4,
          "name": "cell39 ~ gene48"
        },
        {
          "x": 38,
          "y": 48,
          "value": 8,
          "name": "cell39 ~ gene49"
        },
        {
          "x": 38,
          "y": 49,
          "value": 0,
          "name": "cell39 ~ gene50"
        },
        {
          "x": 38,
          "y": 50,
          "value": 19,
          "name": "cell39 ~ gene51"
        },
        {
          "x": 38,
          "y": 51,
          "value": 0,
          "name": "cell39 ~ gene52"
        },
        {
          "x": 38,
          "y": 52,
          "value": 0,
          "name": "cell39 ~ gene53"
        },
        {
          "x": 38,
          "y": 53,
          "value": 0,
          "name": "cell39 ~ gene54"
        },
        {
          "x": 38,
          "y": 54,
          "value": 3,
          "name": "cell39 ~ gene55"
        },
        {
          "x": 38,
          "y": 55,
          "value": 10,
          "name": "cell39 ~ gene56"
        },
        {
          "x": 38,
          "y": 56,
          "value": 0,
          "name": "cell39 ~ gene57"
        },
        {
          "x": 38,
          "y": 57,
          "value": 0,
          "name": "cell39 ~ gene58"
        },
        {
          "x": 38,
          "y": 58,
          "value": 0,
          "name": "cell39 ~ gene59"
        },
        {
          "x": 38,
          "y": 59,
          "value": 4,
          "name": "cell39 ~ gene60"
        },
        {
          "x": 38,
          "y": 60,
          "value": 4,
          "name": "cell39 ~ gene61"
        },
        {
          "x": 38,
          "y": 61,
          "value": 9,
          "name": "cell39 ~ gene62"
        },
        {
          "x": 38,
          "y": 62,
          "value": 0,
          "name": "cell39 ~ gene63"
        },
        {
          "x": 38,
          "y": 63,
          "value": 18,
          "name": "cell39 ~ gene64"
        },
        {
          "x": 38,
          "y": 64,
          "value": 12,
          "name": "cell39 ~ gene65"
        },
        {
          "x": 38,
          "y": 65,
          "value": 4,
          "name": "cell39 ~ gene66"
        },
        {
          "x": 38,
          "y": 66,
          "value": 1,
          "name": "cell39 ~ gene67"
        },
        {
          "x": 38,
          "y": 67,
          "value": 0,
          "name": "cell39 ~ gene68"
        },
        {
          "x": 38,
          "y": 68,
          "value": 0,
          "name": "cell39 ~ gene69"
        },
        {
          "x": 38,
          "y": 69,
          "value": 13,
          "name": "cell39 ~ gene70"
        },
        {
          "x": 38,
          "y": 70,
          "value": 7,
          "name": "cell39 ~ gene71"
        },
        {
          "x": 38,
          "y": 71,
          "value": 0,
          "name": "cell39 ~ gene72"
        },
        {
          "x": 38,
          "y": 72,
          "value": 0,
          "name": "cell39 ~ gene73"
        },
        {
          "x": 38,
          "y": 73,
          "value": 0,
          "name": "cell39 ~ gene74"
        },
        {
          "x": 38,
          "y": 74,
          "value": 0,
          "name": "cell39 ~ gene75"
        },
        {
          "x": 38,
          "y": 75,
          "value": 0,
          "name": "cell39 ~ gene76"
        },
        {
          "x": 38,
          "y": 76,
          "value": 0,
          "name": "cell39 ~ gene77"
        },
        {
          "x": 38,
          "y": 77,
          "value": 0,
          "name": "cell39 ~ gene78"
        },
        {
          "x": 38,
          "y": 78,
          "value": 0,
          "name": "cell39 ~ gene79"
        },
        {
          "x": 38,
          "y": 79,
          "value": 15,
          "name": "cell39 ~ gene80"
        },
        {
          "x": 38,
          "y": 80,
          "value": 13,
          "name": "cell39 ~ gene81"
        },
        {
          "x": 38,
          "y": 81,
          "value": 0,
          "name": "cell39 ~ gene82"
        },
        {
          "x": 38,
          "y": 82,
          "value": 0,
          "name": "cell39 ~ gene83"
        },
        {
          "x": 38,
          "y": 83,
          "value": 0,
          "name": "cell39 ~ gene84"
        },
        {
          "x": 38,
          "y": 84,
          "value": 0,
          "name": "cell39 ~ gene85"
        },
        {
          "x": 38,
          "y": 85,
          "value": 0,
          "name": "cell39 ~ gene86"
        },
        {
          "x": 38,
          "y": 86,
          "value": 0,
          "name": "cell39 ~ gene87"
        },
        {
          "x": 38,
          "y": 87,
          "value": 0,
          "name": "cell39 ~ gene88"
        },
        {
          "x": 38,
          "y": 88,
          "value": 0,
          "name": "cell39 ~ gene89"
        },
        {
          "x": 38,
          "y": 89,
          "value": 0,
          "name": "cell39 ~ gene90"
        },
        {
          "x": 38,
          "y": 90,
          "value": 3,
          "name": "cell39 ~ gene91"
        },
        {
          "x": 38,
          "y": 91,
          "value": 8,
          "name": "cell39 ~ gene92"
        },
        {
          "x": 38,
          "y": 92,
          "value": 23,
          "name": "cell39 ~ gene93"
        },
        {
          "x": 38,
          "y": 93,
          "value": 13,
          "name": "cell39 ~ gene94"
        },
        {
          "x": 38,
          "y": 94,
          "value": 0,
          "name": "cell39 ~ gene95"
        },
        {
          "x": 38,
          "y": 95,
          "value": 7,
          "name": "cell39 ~ gene96"
        },
        {
          "x": 38,
          "y": 96,
          "value": 0,
          "name": "cell39 ~ gene97"
        },
        {
          "x": 38,
          "y": 97,
          "value": 8,
          "name": "cell39 ~ gene98"
        },
        {
          "x": 38,
          "y": 98,
          "value": 0,
          "name": "cell39 ~ gene99"
        },
        {
          "x": 38,
          "y": 99,
          "value": 0,
          "name": "cell39 ~ gene100"
        },
        {
          "x": 39,
          "y": 0,
          "value": 0,
          "name": "cell40 ~ gene1"
        },
        {
          "x": 39,
          "y": 1,
          "value": 0,
          "name": "cell40 ~ gene2"
        },
        {
          "x": 39,
          "y": 2,
          "value": 0,
          "name": "cell40 ~ gene3"
        },
        {
          "x": 39,
          "y": 3,
          "value": 3,
          "name": "cell40 ~ gene4"
        },
        {
          "x": 39,
          "y": 4,
          "value": 7,
          "name": "cell40 ~ gene5"
        },
        {
          "x": 39,
          "y": 5,
          "value": 0,
          "name": "cell40 ~ gene6"
        },
        {
          "x": 39,
          "y": 6,
          "value": 2,
          "name": "cell40 ~ gene7"
        },
        {
          "x": 39,
          "y": 7,
          "value": 0,
          "name": "cell40 ~ gene8"
        },
        {
          "x": 39,
          "y": 8,
          "value": 17,
          "name": "cell40 ~ gene9"
        },
        {
          "x": 39,
          "y": 9,
          "value": 0,
          "name": "cell40 ~ gene10"
        },
        {
          "x": 39,
          "y": 10,
          "value": 0,
          "name": "cell40 ~ gene11"
        },
        {
          "x": 39,
          "y": 11,
          "value": 0,
          "name": "cell40 ~ gene12"
        },
        {
          "x": 39,
          "y": 12,
          "value": 0,
          "name": "cell40 ~ gene13"
        },
        {
          "x": 39,
          "y": 13,
          "value": 0,
          "name": "cell40 ~ gene14"
        },
        {
          "x": 39,
          "y": 14,
          "value": 5,
          "name": "cell40 ~ gene15"
        },
        {
          "x": 39,
          "y": 15,
          "value": 0,
          "name": "cell40 ~ gene16"
        },
        {
          "x": 39,
          "y": 16,
          "value": 0,
          "name": "cell40 ~ gene17"
        },
        {
          "x": 39,
          "y": 17,
          "value": 0,
          "name": "cell40 ~ gene18"
        },
        {
          "x": 39,
          "y": 18,
          "value": 5,
          "name": "cell40 ~ gene19"
        },
        {
          "x": 39,
          "y": 19,
          "value": 0,
          "name": "cell40 ~ gene20"
        },
        {
          "x": 39,
          "y": 20,
          "value": 14,
          "name": "cell40 ~ gene21"
        },
        {
          "x": 39,
          "y": 21,
          "value": 29,
          "name": "cell40 ~ gene22"
        },
        {
          "x": 39,
          "y": 22,
          "value": 11,
          "name": "cell40 ~ gene23"
        },
        {
          "x": 39,
          "y": 23,
          "value": 0,
          "name": "cell40 ~ gene24"
        },
        {
          "x": 39,
          "y": 24,
          "value": 43,
          "name": "cell40 ~ gene25"
        },
        {
          "x": 39,
          "y": 25,
          "value": 18,
          "name": "cell40 ~ gene26"
        },
        {
          "x": 39,
          "y": 26,
          "value": 17,
          "name": "cell40 ~ gene27"
        },
        {
          "x": 39,
          "y": 27,
          "value": 0,
          "name": "cell40 ~ gene28"
        },
        {
          "x": 39,
          "y": 28,
          "value": 0,
          "name": "cell40 ~ gene29"
        },
        {
          "x": 39,
          "y": 29,
          "value": 23,
          "name": "cell40 ~ gene30"
        },
        {
          "x": 39,
          "y": 30,
          "value": 39,
          "name": "cell40 ~ gene31"
        },
        {
          "x": 39,
          "y": 31,
          "value": 0,
          "name": "cell40 ~ gene32"
        },
        {
          "x": 39,
          "y": 32,
          "value": 0,
          "name": "cell40 ~ gene33"
        },
        {
          "x": 39,
          "y": 33,
          "value": 0,
          "name": "cell40 ~ gene34"
        },
        {
          "x": 39,
          "y": 34,
          "value": 15,
          "name": "cell40 ~ gene35"
        },
        {
          "x": 39,
          "y": 35,
          "value": 0,
          "name": "cell40 ~ gene36"
        },
        {
          "x": 39,
          "y": 36,
          "value": 2,
          "name": "cell40 ~ gene37"
        },
        {
          "x": 39,
          "y": 37,
          "value": 8,
          "name": "cell40 ~ gene38"
        },
        {
          "x": 39,
          "y": 38,
          "value": 1,
          "name": "cell40 ~ gene39"
        },
        {
          "x": 39,
          "y": 39,
          "value": 0,
          "name": "cell40 ~ gene40"
        },
        {
          "x": 39,
          "y": 40,
          "value": 0,
          "name": "cell40 ~ gene41"
        },
        {
          "x": 39,
          "y": 41,
          "value": 14,
          "name": "cell40 ~ gene42"
        },
        {
          "x": 39,
          "y": 42,
          "value": 0,
          "name": "cell40 ~ gene43"
        },
        {
          "x": 39,
          "y": 43,
          "value": 0,
          "name": "cell40 ~ gene44"
        },
        {
          "x": 39,
          "y": 44,
          "value": 0,
          "name": "cell40 ~ gene45"
        },
        {
          "x": 39,
          "y": 45,
          "value": 3,
          "name": "cell40 ~ gene46"
        },
        {
          "x": 39,
          "y": 46,
          "value": 11,
          "name": "cell40 ~ gene47"
        },
        {
          "x": 39,
          "y": 47,
          "value": 10,
          "name": "cell40 ~ gene48"
        },
        {
          "x": 39,
          "y": 48,
          "value": 0,
          "name": "cell40 ~ gene49"
        },
        {
          "x": 39,
          "y": 49,
          "value": 11,
          "name": "cell40 ~ gene50"
        },
        {
          "x": 39,
          "y": 50,
          "value": 0,
          "name": "cell40 ~ gene51"
        },
        {
          "x": 39,
          "y": 51,
          "value": 2,
          "name": "cell40 ~ gene52"
        },
        {
          "x": 39,
          "y": 52,
          "value": 10,
          "name": "cell40 ~ gene53"
        },
        {
          "x": 39,
          "y": 53,
          "value": 11,
          "name": "cell40 ~ gene54"
        },
        {
          "x": 39,
          "y": 54,
          "value": 0,
          "name": "cell40 ~ gene55"
        },
        {
          "x": 39,
          "y": 55,
          "value": 0,
          "name": "cell40 ~ gene56"
        },
        {
          "x": 39,
          "y": 56,
          "value": 14,
          "name": "cell40 ~ gene57"
        },
        {
          "x": 39,
          "y": 57,
          "value": 0,
          "name": "cell40 ~ gene58"
        },
        {
          "x": 39,
          "y": 58,
          "value": 0,
          "name": "cell40 ~ gene59"
        },
        {
          "x": 39,
          "y": 59,
          "value": 0,
          "name": "cell40 ~ gene60"
        },
        {
          "x": 39,
          "y": 60,
          "value": 0,
          "name": "cell40 ~ gene61"
        },
        {
          "x": 39,
          "y": 61,
          "value": 3,
          "name": "cell40 ~ gene62"
        },
        {
          "x": 39,
          "y": 62,
          "value": 0,
          "name": "cell40 ~ gene63"
        },
        {
          "x": 39,
          "y": 63,
          "value": 8,
          "name": "cell40 ~ gene64"
        },
        {
          "x": 39,
          "y": 64,
          "value": 0,
          "name": "cell40 ~ gene65"
        },
        {
          "x": 39,
          "y": 65,
          "value": 7,
          "name": "cell40 ~ gene66"
        },
        {
          "x": 39,
          "y": 66,
          "value": 0,
          "name": "cell40 ~ gene67"
        },
        {
          "x": 39,
          "y": 67,
          "value": 8,
          "name": "cell40 ~ gene68"
        },
        {
          "x": 39,
          "y": 68,
          "value": 1,
          "name": "cell40 ~ gene69"
        },
        {
          "x": 39,
          "y": 69,
          "value": 1,
          "name": "cell40 ~ gene70"
        },
        {
          "x": 39,
          "y": 70,
          "value": 5,
          "name": "cell40 ~ gene71"
        },
        {
          "x": 39,
          "y": 71,
          "value": 4,
          "name": "cell40 ~ gene72"
        },
        {
          "x": 39,
          "y": 72,
          "value": 0,
          "name": "cell40 ~ gene73"
        },
        {
          "x": 39,
          "y": 73,
          "value": 3,
          "name": "cell40 ~ gene74"
        },
        {
          "x": 39,
          "y": 74,
          "value": 0,
          "name": "cell40 ~ gene75"
        },
        {
          "x": 39,
          "y": 75,
          "value": 2,
          "name": "cell40 ~ gene76"
        },
        {
          "x": 39,
          "y": 76,
          "value": 0,
          "name": "cell40 ~ gene77"
        },
        {
          "x": 39,
          "y": 77,
          "value": 0,
          "name": "cell40 ~ gene78"
        },
        {
          "x": 39,
          "y": 78,
          "value": 1,
          "name": "cell40 ~ gene79"
        },
        {
          "x": 39,
          "y": 79,
          "value": 2,
          "name": "cell40 ~ gene80"
        },
        {
          "x": 39,
          "y": 80,
          "value": 0,
          "name": "cell40 ~ gene81"
        },
        {
          "x": 39,
          "y": 81,
          "value": 8,
          "name": "cell40 ~ gene82"
        },
        {
          "x": 39,
          "y": 82,
          "value": 15,
          "name": "cell40 ~ gene83"
        },
        {
          "x": 39,
          "y": 83,
          "value": 0,
          "name": "cell40 ~ gene84"
        },
        {
          "x": 39,
          "y": 84,
          "value": 0,
          "name": "cell40 ~ gene85"
        },
        {
          "x": 39,
          "y": 85,
          "value": 8,
          "name": "cell40 ~ gene86"
        },
        {
          "x": 39,
          "y": 86,
          "value": 0,
          "name": "cell40 ~ gene87"
        },
        {
          "x": 39,
          "y": 87,
          "value": 3,
          "name": "cell40 ~ gene88"
        },
        {
          "x": 39,
          "y": 88,
          "value": 1,
          "name": "cell40 ~ gene89"
        },
        {
          "x": 39,
          "y": 89,
          "value": 0,
          "name": "cell40 ~ gene90"
        },
        {
          "x": 39,
          "y": 90,
          "value": 5,
          "name": "cell40 ~ gene91"
        },
        {
          "x": 39,
          "y": 91,
          "value": 0,
          "name": "cell40 ~ gene92"
        },
        {
          "x": 39,
          "y": 92,
          "value": 5,
          "name": "cell40 ~ gene93"
        },
        {
          "x": 39,
          "y": 93,
          "value": 0,
          "name": "cell40 ~ gene94"
        },
        {
          "x": 39,
          "y": 94,
          "value": 0,
          "name": "cell40 ~ gene95"
        },
        {
          "x": 39,
          "y": 95,
          "value": 6,
          "name": "cell40 ~ gene96"
        },
        {
          "x": 39,
          "y": 96,
          "value": 0,
          "name": "cell40 ~ gene97"
        },
        {
          "x": 39,
          "y": 97,
          "value": 6,
          "name": "cell40 ~ gene98"
        },
        {
          "x": 39,
          "y": 98,
          "value": 1,
          "name": "cell40 ~ gene99"
        },
        {
          "x": 39,
          "y": 99,
          "value": 16,
          "name": "cell40 ~ gene100"
        },
        {
          "x": 40,
          "y": 0,
          "value": 8,
          "name": "cell41 ~ gene1"
        },
        {
          "x": 40,
          "y": 1,
          "value": 0,
          "name": "cell41 ~ gene2"
        },
        {
          "x": 40,
          "y": 2,
          "value": 0,
          "name": "cell41 ~ gene3"
        },
        {
          "x": 40,
          "y": 3,
          "value": 0,
          "name": "cell41 ~ gene4"
        },
        {
          "x": 40,
          "y": 4,
          "value": 0,
          "name": "cell41 ~ gene5"
        },
        {
          "x": 40,
          "y": 5,
          "value": 2,
          "name": "cell41 ~ gene6"
        },
        {
          "x": 40,
          "y": 6,
          "value": 1,
          "name": "cell41 ~ gene7"
        },
        {
          "x": 40,
          "y": 7,
          "value": 0,
          "name": "cell41 ~ gene8"
        },
        {
          "x": 40,
          "y": 8,
          "value": 0,
          "name": "cell41 ~ gene9"
        },
        {
          "x": 40,
          "y": 9,
          "value": 6,
          "name": "cell41 ~ gene10"
        },
        {
          "x": 40,
          "y": 10,
          "value": 9,
          "name": "cell41 ~ gene11"
        },
        {
          "x": 40,
          "y": 11,
          "value": 0,
          "name": "cell41 ~ gene12"
        },
        {
          "x": 40,
          "y": 12,
          "value": 0,
          "name": "cell41 ~ gene13"
        },
        {
          "x": 40,
          "y": 13,
          "value": 2,
          "name": "cell41 ~ gene14"
        },
        {
          "x": 40,
          "y": 14,
          "value": 3,
          "name": "cell41 ~ gene15"
        },
        {
          "x": 40,
          "y": 15,
          "value": 12,
          "name": "cell41 ~ gene16"
        },
        {
          "x": 40,
          "y": 16,
          "value": 0,
          "name": "cell41 ~ gene17"
        },
        {
          "x": 40,
          "y": 17,
          "value": 12,
          "name": "cell41 ~ gene18"
        },
        {
          "x": 40,
          "y": 18,
          "value": 0,
          "name": "cell41 ~ gene19"
        },
        {
          "x": 40,
          "y": 19,
          "value": 21,
          "name": "cell41 ~ gene20"
        },
        {
          "x": 40,
          "y": 20,
          "value": 0,
          "name": "cell41 ~ gene21"
        },
        {
          "x": 40,
          "y": 21,
          "value": 27,
          "name": "cell41 ~ gene22"
        },
        {
          "x": 40,
          "y": 22,
          "value": 5,
          "name": "cell41 ~ gene23"
        },
        {
          "x": 40,
          "y": 23,
          "value": 1,
          "name": "cell41 ~ gene24"
        },
        {
          "x": 40,
          "y": 24,
          "value": 14,
          "name": "cell41 ~ gene25"
        },
        {
          "x": 40,
          "y": 25,
          "value": 8,
          "name": "cell41 ~ gene26"
        },
        {
          "x": 40,
          "y": 26,
          "value": 17,
          "name": "cell41 ~ gene27"
        },
        {
          "x": 40,
          "y": 27,
          "value": 15,
          "name": "cell41 ~ gene28"
        },
        {
          "x": 40,
          "y": 28,
          "value": 0,
          "name": "cell41 ~ gene29"
        },
        {
          "x": 40,
          "y": 29,
          "value": 20,
          "name": "cell41 ~ gene30"
        },
        {
          "x": 40,
          "y": 30,
          "value": 32,
          "name": "cell41 ~ gene31"
        },
        {
          "x": 40,
          "y": 31,
          "value": 13,
          "name": "cell41 ~ gene32"
        },
        {
          "x": 40,
          "y": 32,
          "value": 1,
          "name": "cell41 ~ gene33"
        },
        {
          "x": 40,
          "y": 33,
          "value": 1,
          "name": "cell41 ~ gene34"
        },
        {
          "x": 40,
          "y": 34,
          "value": 13,
          "name": "cell41 ~ gene35"
        },
        {
          "x": 40,
          "y": 35,
          "value": 0,
          "name": "cell41 ~ gene36"
        },
        {
          "x": 40,
          "y": 36,
          "value": 30,
          "name": "cell41 ~ gene37"
        },
        {
          "x": 40,
          "y": 37,
          "value": 31,
          "name": "cell41 ~ gene38"
        },
        {
          "x": 40,
          "y": 38,
          "value": 1,
          "name": "cell41 ~ gene39"
        },
        {
          "x": 40,
          "y": 39,
          "value": 10,
          "name": "cell41 ~ gene40"
        },
        {
          "x": 40,
          "y": 40,
          "value": 1,
          "name": "cell41 ~ gene41"
        },
        {
          "x": 40,
          "y": 41,
          "value": 0,
          "name": "cell41 ~ gene42"
        },
        {
          "x": 40,
          "y": 42,
          "value": 0,
          "name": "cell41 ~ gene43"
        },
        {
          "x": 40,
          "y": 43,
          "value": 0,
          "name": "cell41 ~ gene44"
        },
        {
          "x": 40,
          "y": 44,
          "value": 5,
          "name": "cell41 ~ gene45"
        },
        {
          "x": 40,
          "y": 45,
          "value": 0,
          "name": "cell41 ~ gene46"
        },
        {
          "x": 40,
          "y": 46,
          "value": 10,
          "name": "cell41 ~ gene47"
        },
        {
          "x": 40,
          "y": 47,
          "value": 0,
          "name": "cell41 ~ gene48"
        },
        {
          "x": 40,
          "y": 48,
          "value": 5,
          "name": "cell41 ~ gene49"
        },
        {
          "x": 40,
          "y": 49,
          "value": 0,
          "name": "cell41 ~ gene50"
        },
        {
          "x": 40,
          "y": 50,
          "value": 2,
          "name": "cell41 ~ gene51"
        },
        {
          "x": 40,
          "y": 51,
          "value": 0,
          "name": "cell41 ~ gene52"
        },
        {
          "x": 40,
          "y": 52,
          "value": 8,
          "name": "cell41 ~ gene53"
        },
        {
          "x": 40,
          "y": 53,
          "value": 0,
          "name": "cell41 ~ gene54"
        },
        {
          "x": 40,
          "y": 54,
          "value": 0,
          "name": "cell41 ~ gene55"
        },
        {
          "x": 40,
          "y": 55,
          "value": 0,
          "name": "cell41 ~ gene56"
        },
        {
          "x": 40,
          "y": 56,
          "value": 11,
          "name": "cell41 ~ gene57"
        },
        {
          "x": 40,
          "y": 57,
          "value": 0,
          "name": "cell41 ~ gene58"
        },
        {
          "x": 40,
          "y": 58,
          "value": 0,
          "name": "cell41 ~ gene59"
        },
        {
          "x": 40,
          "y": 59,
          "value": 4,
          "name": "cell41 ~ gene60"
        },
        {
          "x": 40,
          "y": 60,
          "value": 17,
          "name": "cell41 ~ gene61"
        },
        {
          "x": 40,
          "y": 61,
          "value": 0,
          "name": "cell41 ~ gene62"
        },
        {
          "x": 40,
          "y": 62,
          "value": 0,
          "name": "cell41 ~ gene63"
        },
        {
          "x": 40,
          "y": 63,
          "value": 2,
          "name": "cell41 ~ gene64"
        },
        {
          "x": 40,
          "y": 64,
          "value": 5,
          "name": "cell41 ~ gene65"
        },
        {
          "x": 40,
          "y": 65,
          "value": 4,
          "name": "cell41 ~ gene66"
        },
        {
          "x": 40,
          "y": 66,
          "value": 0,
          "name": "cell41 ~ gene67"
        },
        {
          "x": 40,
          "y": 67,
          "value": 3,
          "name": "cell41 ~ gene68"
        },
        {
          "x": 40,
          "y": 68,
          "value": 0,
          "name": "cell41 ~ gene69"
        },
        {
          "x": 40,
          "y": 69,
          "value": 0,
          "name": "cell41 ~ gene70"
        },
        {
          "x": 40,
          "y": 70,
          "value": 0,
          "name": "cell41 ~ gene71"
        },
        {
          "x": 40,
          "y": 71,
          "value": 11,
          "name": "cell41 ~ gene72"
        },
        {
          "x": 40,
          "y": 72,
          "value": 0,
          "name": "cell41 ~ gene73"
        },
        {
          "x": 40,
          "y": 73,
          "value": 1,
          "name": "cell41 ~ gene74"
        },
        {
          "x": 40,
          "y": 74,
          "value": 0,
          "name": "cell41 ~ gene75"
        },
        {
          "x": 40,
          "y": 75,
          "value": 0,
          "name": "cell41 ~ gene76"
        },
        {
          "x": 40,
          "y": 76,
          "value": 10,
          "name": "cell41 ~ gene77"
        },
        {
          "x": 40,
          "y": 77,
          "value": 11,
          "name": "cell41 ~ gene78"
        },
        {
          "x": 40,
          "y": 78,
          "value": 3,
          "name": "cell41 ~ gene79"
        },
        {
          "x": 40,
          "y": 79,
          "value": 19,
          "name": "cell41 ~ gene80"
        },
        {
          "x": 40,
          "y": 80,
          "value": 10,
          "name": "cell41 ~ gene81"
        },
        {
          "x": 40,
          "y": 81,
          "value": 9,
          "name": "cell41 ~ gene82"
        },
        {
          "x": 40,
          "y": 82,
          "value": 1,
          "name": "cell41 ~ gene83"
        },
        {
          "x": 40,
          "y": 83,
          "value": 17,
          "name": "cell41 ~ gene84"
        },
        {
          "x": 40,
          "y": 84,
          "value": 0,
          "name": "cell41 ~ gene85"
        },
        {
          "x": 40,
          "y": 85,
          "value": 0,
          "name": "cell41 ~ gene86"
        },
        {
          "x": 40,
          "y": 86,
          "value": 6,
          "name": "cell41 ~ gene87"
        },
        {
          "x": 40,
          "y": 87,
          "value": 15,
          "name": "cell41 ~ gene88"
        },
        {
          "x": 40,
          "y": 88,
          "value": 0,
          "name": "cell41 ~ gene89"
        },
        {
          "x": 40,
          "y": 89,
          "value": 1,
          "name": "cell41 ~ gene90"
        },
        {
          "x": 40,
          "y": 90,
          "value": 0,
          "name": "cell41 ~ gene91"
        },
        {
          "x": 40,
          "y": 91,
          "value": 10,
          "name": "cell41 ~ gene92"
        },
        {
          "x": 40,
          "y": 92,
          "value": 0,
          "name": "cell41 ~ gene93"
        },
        {
          "x": 40,
          "y": 93,
          "value": 11,
          "name": "cell41 ~ gene94"
        },
        {
          "x": 40,
          "y": 94,
          "value": 0,
          "name": "cell41 ~ gene95"
        },
        {
          "x": 40,
          "y": 95,
          "value": 0,
          "name": "cell41 ~ gene96"
        },
        {
          "x": 40,
          "y": 96,
          "value": 4,
          "name": "cell41 ~ gene97"
        },
        {
          "x": 40,
          "y": 97,
          "value": 0,
          "name": "cell41 ~ gene98"
        },
        {
          "x": 40,
          "y": 98,
          "value": 0,
          "name": "cell41 ~ gene99"
        },
        {
          "x": 40,
          "y": 99,
          "value": 0,
          "name": "cell41 ~ gene100"
        },
        {
          "x": 41,
          "y": 0,
          "value": 0,
          "name": "cell42 ~ gene1"
        },
        {
          "x": 41,
          "y": 1,
          "value": 0,
          "name": "cell42 ~ gene2"
        },
        {
          "x": 41,
          "y": 2,
          "value": 15,
          "name": "cell42 ~ gene3"
        },
        {
          "x": 41,
          "y": 3,
          "value": 12,
          "name": "cell42 ~ gene4"
        },
        {
          "x": 41,
          "y": 4,
          "value": 2,
          "name": "cell42 ~ gene5"
        },
        {
          "x": 41,
          "y": 5,
          "value": 2,
          "name": "cell42 ~ gene6"
        },
        {
          "x": 41,
          "y": 6,
          "value": 8,
          "name": "cell42 ~ gene7"
        },
        {
          "x": 41,
          "y": 7,
          "value": 16,
          "name": "cell42 ~ gene8"
        },
        {
          "x": 41,
          "y": 8,
          "value": 12,
          "name": "cell42 ~ gene9"
        },
        {
          "x": 41,
          "y": 9,
          "value": 0,
          "name": "cell42 ~ gene10"
        },
        {
          "x": 41,
          "y": 10,
          "value": 0,
          "name": "cell42 ~ gene11"
        },
        {
          "x": 41,
          "y": 11,
          "value": 0,
          "name": "cell42 ~ gene12"
        },
        {
          "x": 41,
          "y": 12,
          "value": 0,
          "name": "cell42 ~ gene13"
        },
        {
          "x": 41,
          "y": 13,
          "value": 14,
          "name": "cell42 ~ gene14"
        },
        {
          "x": 41,
          "y": 14,
          "value": 9,
          "name": "cell42 ~ gene15"
        },
        {
          "x": 41,
          "y": 15,
          "value": 0,
          "name": "cell42 ~ gene16"
        },
        {
          "x": 41,
          "y": 16,
          "value": 0,
          "name": "cell42 ~ gene17"
        },
        {
          "x": 41,
          "y": 17,
          "value": 9,
          "name": "cell42 ~ gene18"
        },
        {
          "x": 41,
          "y": 18,
          "value": 11,
          "name": "cell42 ~ gene19"
        },
        {
          "x": 41,
          "y": 19,
          "value": 0,
          "name": "cell42 ~ gene20"
        },
        {
          "x": 41,
          "y": 20,
          "value": 0,
          "name": "cell42 ~ gene21"
        },
        {
          "x": 41,
          "y": 21,
          "value": 23,
          "name": "cell42 ~ gene22"
        },
        {
          "x": 41,
          "y": 22,
          "value": 0,
          "name": "cell42 ~ gene23"
        },
        {
          "x": 41,
          "y": 23,
          "value": 0,
          "name": "cell42 ~ gene24"
        },
        {
          "x": 41,
          "y": 24,
          "value": 11,
          "name": "cell42 ~ gene25"
        },
        {
          "x": 41,
          "y": 25,
          "value": 19,
          "name": "cell42 ~ gene26"
        },
        {
          "x": 41,
          "y": 26,
          "value": 16,
          "name": "cell42 ~ gene27"
        },
        {
          "x": 41,
          "y": 27,
          "value": 0,
          "name": "cell42 ~ gene28"
        },
        {
          "x": 41,
          "y": 28,
          "value": 3,
          "name": "cell42 ~ gene29"
        },
        {
          "x": 41,
          "y": 29,
          "value": 32,
          "name": "cell42 ~ gene30"
        },
        {
          "x": 41,
          "y": 30,
          "value": 34,
          "name": "cell42 ~ gene31"
        },
        {
          "x": 41,
          "y": 31,
          "value": 8,
          "name": "cell42 ~ gene32"
        },
        {
          "x": 41,
          "y": 32,
          "value": 3,
          "name": "cell42 ~ gene33"
        },
        {
          "x": 41,
          "y": 33,
          "value": 16,
          "name": "cell42 ~ gene34"
        },
        {
          "x": 41,
          "y": 34,
          "value": 8,
          "name": "cell42 ~ gene35"
        },
        {
          "x": 41,
          "y": 35,
          "value": 11,
          "name": "cell42 ~ gene36"
        },
        {
          "x": 41,
          "y": 36,
          "value": 10,
          "name": "cell42 ~ gene37"
        },
        {
          "x": 41,
          "y": 37,
          "value": 0,
          "name": "cell42 ~ gene38"
        },
        {
          "x": 41,
          "y": 38,
          "value": 0,
          "name": "cell42 ~ gene39"
        },
        {
          "x": 41,
          "y": 39,
          "value": 0,
          "name": "cell42 ~ gene40"
        },
        {
          "x": 41,
          "y": 40,
          "value": 15,
          "name": "cell42 ~ gene41"
        },
        {
          "x": 41,
          "y": 41,
          "value": 13,
          "name": "cell42 ~ gene42"
        },
        {
          "x": 41,
          "y": 42,
          "value": 15,
          "name": "cell42 ~ gene43"
        },
        {
          "x": 41,
          "y": 43,
          "value": 0,
          "name": "cell42 ~ gene44"
        },
        {
          "x": 41,
          "y": 44,
          "value": 0,
          "name": "cell42 ~ gene45"
        },
        {
          "x": 41,
          "y": 45,
          "value": 0,
          "name": "cell42 ~ gene46"
        },
        {
          "x": 41,
          "y": 46,
          "value": 12,
          "name": "cell42 ~ gene47"
        },
        {
          "x": 41,
          "y": 47,
          "value": 6,
          "name": "cell42 ~ gene48"
        },
        {
          "x": 41,
          "y": 48,
          "value": 0,
          "name": "cell42 ~ gene49"
        },
        {
          "x": 41,
          "y": 49,
          "value": 0,
          "name": "cell42 ~ gene50"
        },
        {
          "x": 41,
          "y": 50,
          "value": 0,
          "name": "cell42 ~ gene51"
        },
        {
          "x": 41,
          "y": 51,
          "value": 5,
          "name": "cell42 ~ gene52"
        },
        {
          "x": 41,
          "y": 52,
          "value": 0,
          "name": "cell42 ~ gene53"
        },
        {
          "x": 41,
          "y": 53,
          "value": 6,
          "name": "cell42 ~ gene54"
        },
        {
          "x": 41,
          "y": 54,
          "value": 2,
          "name": "cell42 ~ gene55"
        },
        {
          "x": 41,
          "y": 55,
          "value": 0,
          "name": "cell42 ~ gene56"
        },
        {
          "x": 41,
          "y": 56,
          "value": 0,
          "name": "cell42 ~ gene57"
        },
        {
          "x": 41,
          "y": 57,
          "value": 0,
          "name": "cell42 ~ gene58"
        },
        {
          "x": 41,
          "y": 58,
          "value": 0,
          "name": "cell42 ~ gene59"
        },
        {
          "x": 41,
          "y": 59,
          "value": 22,
          "name": "cell42 ~ gene60"
        },
        {
          "x": 41,
          "y": 60,
          "value": 13,
          "name": "cell42 ~ gene61"
        },
        {
          "x": 41,
          "y": 61,
          "value": 0,
          "name": "cell42 ~ gene62"
        },
        {
          "x": 41,
          "y": 62,
          "value": 5,
          "name": "cell42 ~ gene63"
        },
        {
          "x": 41,
          "y": 63,
          "value": 0,
          "name": "cell42 ~ gene64"
        },
        {
          "x": 41,
          "y": 64,
          "value": 0,
          "name": "cell42 ~ gene65"
        },
        {
          "x": 41,
          "y": 65,
          "value": 0,
          "name": "cell42 ~ gene66"
        },
        {
          "x": 41,
          "y": 66,
          "value": 0,
          "name": "cell42 ~ gene67"
        },
        {
          "x": 41,
          "y": 67,
          "value": 7,
          "name": "cell42 ~ gene68"
        },
        {
          "x": 41,
          "y": 68,
          "value": 0,
          "name": "cell42 ~ gene69"
        },
        {
          "x": 41,
          "y": 69,
          "value": 7,
          "name": "cell42 ~ gene70"
        },
        {
          "x": 41,
          "y": 70,
          "value": 2,
          "name": "cell42 ~ gene71"
        },
        {
          "x": 41,
          "y": 71,
          "value": 0,
          "name": "cell42 ~ gene72"
        },
        {
          "x": 41,
          "y": 72,
          "value": 0,
          "name": "cell42 ~ gene73"
        },
        {
          "x": 41,
          "y": 73,
          "value": 0,
          "name": "cell42 ~ gene74"
        },
        {
          "x": 41,
          "y": 74,
          "value": 0,
          "name": "cell42 ~ gene75"
        },
        {
          "x": 41,
          "y": 75,
          "value": 0,
          "name": "cell42 ~ gene76"
        },
        {
          "x": 41,
          "y": 76,
          "value": 0,
          "name": "cell42 ~ gene77"
        },
        {
          "x": 41,
          "y": 77,
          "value": 0,
          "name": "cell42 ~ gene78"
        },
        {
          "x": 41,
          "y": 78,
          "value": 0,
          "name": "cell42 ~ gene79"
        },
        {
          "x": 41,
          "y": 79,
          "value": 0,
          "name": "cell42 ~ gene80"
        },
        {
          "x": 41,
          "y": 80,
          "value": 6,
          "name": "cell42 ~ gene81"
        },
        {
          "x": 41,
          "y": 81,
          "value": 0,
          "name": "cell42 ~ gene82"
        },
        {
          "x": 41,
          "y": 82,
          "value": 6,
          "name": "cell42 ~ gene83"
        },
        {
          "x": 41,
          "y": 83,
          "value": 6,
          "name": "cell42 ~ gene84"
        },
        {
          "x": 41,
          "y": 84,
          "value": 8,
          "name": "cell42 ~ gene85"
        },
        {
          "x": 41,
          "y": 85,
          "value": 12,
          "name": "cell42 ~ gene86"
        },
        {
          "x": 41,
          "y": 86,
          "value": 0,
          "name": "cell42 ~ gene87"
        },
        {
          "x": 41,
          "y": 87,
          "value": 2,
          "name": "cell42 ~ gene88"
        },
        {
          "x": 41,
          "y": 88,
          "value": 3,
          "name": "cell42 ~ gene89"
        },
        {
          "x": 41,
          "y": 89,
          "value": 0,
          "name": "cell42 ~ gene90"
        },
        {
          "x": 41,
          "y": 90,
          "value": 0,
          "name": "cell42 ~ gene91"
        },
        {
          "x": 41,
          "y": 91,
          "value": 0,
          "name": "cell42 ~ gene92"
        },
        {
          "x": 41,
          "y": 92,
          "value": 0,
          "name": "cell42 ~ gene93"
        },
        {
          "x": 41,
          "y": 93,
          "value": 0,
          "name": "cell42 ~ gene94"
        },
        {
          "x": 41,
          "y": 94,
          "value": 18,
          "name": "cell42 ~ gene95"
        },
        {
          "x": 41,
          "y": 95,
          "value": 0,
          "name": "cell42 ~ gene96"
        },
        {
          "x": 41,
          "y": 96,
          "value": 0,
          "name": "cell42 ~ gene97"
        },
        {
          "x": 41,
          "y": 97,
          "value": 0,
          "name": "cell42 ~ gene98"
        },
        {
          "x": 41,
          "y": 98,
          "value": 4,
          "name": "cell42 ~ gene99"
        },
        {
          "x": 41,
          "y": 99,
          "value": 0,
          "name": "cell42 ~ gene100"
        },
        {
          "x": 42,
          "y": 0,
          "value": 9,
          "name": "cell43 ~ gene1"
        },
        {
          "x": 42,
          "y": 1,
          "value": 0,
          "name": "cell43 ~ gene2"
        },
        {
          "x": 42,
          "y": 2,
          "value": 12,
          "name": "cell43 ~ gene3"
        },
        {
          "x": 42,
          "y": 3,
          "value": 13,
          "name": "cell43 ~ gene4"
        },
        {
          "x": 42,
          "y": 4,
          "value": 9,
          "name": "cell43 ~ gene5"
        },
        {
          "x": 42,
          "y": 5,
          "value": 0,
          "name": "cell43 ~ gene6"
        },
        {
          "x": 42,
          "y": 6,
          "value": 0,
          "name": "cell43 ~ gene7"
        },
        {
          "x": 42,
          "y": 7,
          "value": 14,
          "name": "cell43 ~ gene8"
        },
        {
          "x": 42,
          "y": 8,
          "value": 0,
          "name": "cell43 ~ gene9"
        },
        {
          "x": 42,
          "y": 9,
          "value": 2,
          "name": "cell43 ~ gene10"
        },
        {
          "x": 42,
          "y": 10,
          "value": 0,
          "name": "cell43 ~ gene11"
        },
        {
          "x": 42,
          "y": 11,
          "value": 17,
          "name": "cell43 ~ gene12"
        },
        {
          "x": 42,
          "y": 12,
          "value": 0,
          "name": "cell43 ~ gene13"
        },
        {
          "x": 42,
          "y": 13,
          "value": 11,
          "name": "cell43 ~ gene14"
        },
        {
          "x": 42,
          "y": 14,
          "value": 7,
          "name": "cell43 ~ gene15"
        },
        {
          "x": 42,
          "y": 15,
          "value": 0,
          "name": "cell43 ~ gene16"
        },
        {
          "x": 42,
          "y": 16,
          "value": 10,
          "name": "cell43 ~ gene17"
        },
        {
          "x": 42,
          "y": 17,
          "value": 0,
          "name": "cell43 ~ gene18"
        },
        {
          "x": 42,
          "y": 18,
          "value": 9,
          "name": "cell43 ~ gene19"
        },
        {
          "x": 42,
          "y": 19,
          "value": 4,
          "name": "cell43 ~ gene20"
        },
        {
          "x": 42,
          "y": 20,
          "value": 0,
          "name": "cell43 ~ gene21"
        },
        {
          "x": 42,
          "y": 21,
          "value": 34,
          "name": "cell43 ~ gene22"
        },
        {
          "x": 42,
          "y": 22,
          "value": 1,
          "name": "cell43 ~ gene23"
        },
        {
          "x": 42,
          "y": 23,
          "value": 0,
          "name": "cell43 ~ gene24"
        },
        {
          "x": 42,
          "y": 24,
          "value": 9,
          "name": "cell43 ~ gene25"
        },
        {
          "x": 42,
          "y": 25,
          "value": 23,
          "name": "cell43 ~ gene26"
        },
        {
          "x": 42,
          "y": 26,
          "value": 34,
          "name": "cell43 ~ gene27"
        },
        {
          "x": 42,
          "y": 27,
          "value": 29,
          "name": "cell43 ~ gene28"
        },
        {
          "x": 42,
          "y": 28,
          "value": 1,
          "name": "cell43 ~ gene29"
        },
        {
          "x": 42,
          "y": 29,
          "value": 12,
          "name": "cell43 ~ gene30"
        },
        {
          "x": 42,
          "y": 30,
          "value": 44,
          "name": "cell43 ~ gene31"
        },
        {
          "x": 42,
          "y": 31,
          "value": 38,
          "name": "cell43 ~ gene32"
        },
        {
          "x": 42,
          "y": 32,
          "value": 11,
          "name": "cell43 ~ gene33"
        },
        {
          "x": 42,
          "y": 33,
          "value": 9,
          "name": "cell43 ~ gene34"
        },
        {
          "x": 42,
          "y": 34,
          "value": 14,
          "name": "cell43 ~ gene35"
        },
        {
          "x": 42,
          "y": 35,
          "value": 0,
          "name": "cell43 ~ gene36"
        },
        {
          "x": 42,
          "y": 36,
          "value": 18,
          "name": "cell43 ~ gene37"
        },
        {
          "x": 42,
          "y": 37,
          "value": 19,
          "name": "cell43 ~ gene38"
        },
        {
          "x": 42,
          "y": 38,
          "value": 1,
          "name": "cell43 ~ gene39"
        },
        {
          "x": 42,
          "y": 39,
          "value": 2,
          "name": "cell43 ~ gene40"
        },
        {
          "x": 42,
          "y": 40,
          "value": 0,
          "name": "cell43 ~ gene41"
        },
        {
          "x": 42,
          "y": 41,
          "value": 0,
          "name": "cell43 ~ gene42"
        },
        {
          "x": 42,
          "y": 42,
          "value": 0,
          "name": "cell43 ~ gene43"
        },
        {
          "x": 42,
          "y": 43,
          "value": 0,
          "name": "cell43 ~ gene44"
        },
        {
          "x": 42,
          "y": 44,
          "value": 5,
          "name": "cell43 ~ gene45"
        },
        {
          "x": 42,
          "y": 45,
          "value": 11,
          "name": "cell43 ~ gene46"
        },
        {
          "x": 42,
          "y": 46,
          "value": 0,
          "name": "cell43 ~ gene47"
        },
        {
          "x": 42,
          "y": 47,
          "value": 0,
          "name": "cell43 ~ gene48"
        },
        {
          "x": 42,
          "y": 48,
          "value": 0,
          "name": "cell43 ~ gene49"
        },
        {
          "x": 42,
          "y": 49,
          "value": 0,
          "name": "cell43 ~ gene50"
        },
        {
          "x": 42,
          "y": 50,
          "value": 0,
          "name": "cell43 ~ gene51"
        },
        {
          "x": 42,
          "y": 51,
          "value": 0,
          "name": "cell43 ~ gene52"
        },
        {
          "x": 42,
          "y": 52,
          "value": 0,
          "name": "cell43 ~ gene53"
        },
        {
          "x": 42,
          "y": 53,
          "value": 23,
          "name": "cell43 ~ gene54"
        },
        {
          "x": 42,
          "y": 54,
          "value": 5,
          "name": "cell43 ~ gene55"
        },
        {
          "x": 42,
          "y": 55,
          "value": 0,
          "name": "cell43 ~ gene56"
        },
        {
          "x": 42,
          "y": 56,
          "value": 0,
          "name": "cell43 ~ gene57"
        },
        {
          "x": 42,
          "y": 57,
          "value": 10,
          "name": "cell43 ~ gene58"
        },
        {
          "x": 42,
          "y": 58,
          "value": 0,
          "name": "cell43 ~ gene59"
        },
        {
          "x": 42,
          "y": 59,
          "value": 0,
          "name": "cell43 ~ gene60"
        },
        {
          "x": 42,
          "y": 60,
          "value": 2,
          "name": "cell43 ~ gene61"
        },
        {
          "x": 42,
          "y": 61,
          "value": 0,
          "name": "cell43 ~ gene62"
        },
        {
          "x": 42,
          "y": 62,
          "value": 5,
          "name": "cell43 ~ gene63"
        },
        {
          "x": 42,
          "y": 63,
          "value": 13,
          "name": "cell43 ~ gene64"
        },
        {
          "x": 42,
          "y": 64,
          "value": 4,
          "name": "cell43 ~ gene65"
        },
        {
          "x": 42,
          "y": 65,
          "value": 0,
          "name": "cell43 ~ gene66"
        },
        {
          "x": 42,
          "y": 66,
          "value": 0,
          "name": "cell43 ~ gene67"
        },
        {
          "x": 42,
          "y": 67,
          "value": 0,
          "name": "cell43 ~ gene68"
        },
        {
          "x": 42,
          "y": 68,
          "value": 2,
          "name": "cell43 ~ gene69"
        },
        {
          "x": 42,
          "y": 69,
          "value": 0,
          "name": "cell43 ~ gene70"
        },
        {
          "x": 42,
          "y": 70,
          "value": 19,
          "name": "cell43 ~ gene71"
        },
        {
          "x": 42,
          "y": 71,
          "value": 0,
          "name": "cell43 ~ gene72"
        },
        {
          "x": 42,
          "y": 72,
          "value": 0,
          "name": "cell43 ~ gene73"
        },
        {
          "x": 42,
          "y": 73,
          "value": 17,
          "name": "cell43 ~ gene74"
        },
        {
          "x": 42,
          "y": 74,
          "value": 0,
          "name": "cell43 ~ gene75"
        },
        {
          "x": 42,
          "y": 75,
          "value": 0,
          "name": "cell43 ~ gene76"
        },
        {
          "x": 42,
          "y": 76,
          "value": 0,
          "name": "cell43 ~ gene77"
        },
        {
          "x": 42,
          "y": 77,
          "value": 0,
          "name": "cell43 ~ gene78"
        },
        {
          "x": 42,
          "y": 78,
          "value": 10,
          "name": "cell43 ~ gene79"
        },
        {
          "x": 42,
          "y": 79,
          "value": 0,
          "name": "cell43 ~ gene80"
        },
        {
          "x": 42,
          "y": 80,
          "value": 6,
          "name": "cell43 ~ gene81"
        },
        {
          "x": 42,
          "y": 81,
          "value": 0,
          "name": "cell43 ~ gene82"
        },
        {
          "x": 42,
          "y": 82,
          "value": 1,
          "name": "cell43 ~ gene83"
        },
        {
          "x": 42,
          "y": 83,
          "value": 0,
          "name": "cell43 ~ gene84"
        },
        {
          "x": 42,
          "y": 84,
          "value": 0,
          "name": "cell43 ~ gene85"
        },
        {
          "x": 42,
          "y": 85,
          "value": 0,
          "name": "cell43 ~ gene86"
        },
        {
          "x": 42,
          "y": 86,
          "value": 8,
          "name": "cell43 ~ gene87"
        },
        {
          "x": 42,
          "y": 87,
          "value": 0,
          "name": "cell43 ~ gene88"
        },
        {
          "x": 42,
          "y": 88,
          "value": 0,
          "name": "cell43 ~ gene89"
        },
        {
          "x": 42,
          "y": 89,
          "value": 0,
          "name": "cell43 ~ gene90"
        },
        {
          "x": 42,
          "y": 90,
          "value": 15,
          "name": "cell43 ~ gene91"
        },
        {
          "x": 42,
          "y": 91,
          "value": 0,
          "name": "cell43 ~ gene92"
        },
        {
          "x": 42,
          "y": 92,
          "value": 7,
          "name": "cell43 ~ gene93"
        },
        {
          "x": 42,
          "y": 93,
          "value": 0,
          "name": "cell43 ~ gene94"
        },
        {
          "x": 42,
          "y": 94,
          "value": 8,
          "name": "cell43 ~ gene95"
        },
        {
          "x": 42,
          "y": 95,
          "value": 0,
          "name": "cell43 ~ gene96"
        },
        {
          "x": 42,
          "y": 96,
          "value": 17,
          "name": "cell43 ~ gene97"
        },
        {
          "x": 42,
          "y": 97,
          "value": 0,
          "name": "cell43 ~ gene98"
        },
        {
          "x": 42,
          "y": 98,
          "value": 0,
          "name": "cell43 ~ gene99"
        },
        {
          "x": 42,
          "y": 99,
          "value": 19,
          "name": "cell43 ~ gene100"
        },
        {
          "x": 43,
          "y": 0,
          "value": 24,
          "name": "cell44 ~ gene1"
        },
        {
          "x": 43,
          "y": 1,
          "value": 0,
          "name": "cell44 ~ gene2"
        },
        {
          "x": 43,
          "y": 2,
          "value": 11,
          "name": "cell44 ~ gene3"
        },
        {
          "x": 43,
          "y": 3,
          "value": 0,
          "name": "cell44 ~ gene4"
        },
        {
          "x": 43,
          "y": 4,
          "value": 0,
          "name": "cell44 ~ gene5"
        },
        {
          "x": 43,
          "y": 5,
          "value": 0,
          "name": "cell44 ~ gene6"
        },
        {
          "x": 43,
          "y": 6,
          "value": 5,
          "name": "cell44 ~ gene7"
        },
        {
          "x": 43,
          "y": 7,
          "value": 4,
          "name": "cell44 ~ gene8"
        },
        {
          "x": 43,
          "y": 8,
          "value": 0,
          "name": "cell44 ~ gene9"
        },
        {
          "x": 43,
          "y": 9,
          "value": 0,
          "name": "cell44 ~ gene10"
        },
        {
          "x": 43,
          "y": 10,
          "value": 0,
          "name": "cell44 ~ gene11"
        },
        {
          "x": 43,
          "y": 11,
          "value": 0,
          "name": "cell44 ~ gene12"
        },
        {
          "x": 43,
          "y": 12,
          "value": 0,
          "name": "cell44 ~ gene13"
        },
        {
          "x": 43,
          "y": 13,
          "value": 21,
          "name": "cell44 ~ gene14"
        },
        {
          "x": 43,
          "y": 14,
          "value": 0,
          "name": "cell44 ~ gene15"
        },
        {
          "x": 43,
          "y": 15,
          "value": 17,
          "name": "cell44 ~ gene16"
        },
        {
          "x": 43,
          "y": 16,
          "value": 0,
          "name": "cell44 ~ gene17"
        },
        {
          "x": 43,
          "y": 17,
          "value": 9,
          "name": "cell44 ~ gene18"
        },
        {
          "x": 43,
          "y": 18,
          "value": 13,
          "name": "cell44 ~ gene19"
        },
        {
          "x": 43,
          "y": 19,
          "value": 0,
          "name": "cell44 ~ gene20"
        },
        {
          "x": 43,
          "y": 20,
          "value": 0,
          "name": "cell44 ~ gene21"
        },
        {
          "x": 43,
          "y": 21,
          "value": 29,
          "name": "cell44 ~ gene22"
        },
        {
          "x": 43,
          "y": 22,
          "value": 17,
          "name": "cell44 ~ gene23"
        },
        {
          "x": 43,
          "y": 23,
          "value": 0,
          "name": "cell44 ~ gene24"
        },
        {
          "x": 43,
          "y": 24,
          "value": 10,
          "name": "cell44 ~ gene25"
        },
        {
          "x": 43,
          "y": 25,
          "value": 24,
          "name": "cell44 ~ gene26"
        },
        {
          "x": 43,
          "y": 26,
          "value": 20,
          "name": "cell44 ~ gene27"
        },
        {
          "x": 43,
          "y": 27,
          "value": 24,
          "name": "cell44 ~ gene28"
        },
        {
          "x": 43,
          "y": 28,
          "value": 9,
          "name": "cell44 ~ gene29"
        },
        {
          "x": 43,
          "y": 29,
          "value": 34,
          "name": "cell44 ~ gene30"
        },
        {
          "x": 43,
          "y": 30,
          "value": 27,
          "name": "cell44 ~ gene31"
        },
        {
          "x": 43,
          "y": 31,
          "value": 10,
          "name": "cell44 ~ gene32"
        },
        {
          "x": 43,
          "y": 32,
          "value": 4,
          "name": "cell44 ~ gene33"
        },
        {
          "x": 43,
          "y": 33,
          "value": 0,
          "name": "cell44 ~ gene34"
        },
        {
          "x": 43,
          "y": 34,
          "value": 0,
          "name": "cell44 ~ gene35"
        },
        {
          "x": 43,
          "y": 35,
          "value": 0,
          "name": "cell44 ~ gene36"
        },
        {
          "x": 43,
          "y": 36,
          "value": 27,
          "name": "cell44 ~ gene37"
        },
        {
          "x": 43,
          "y": 37,
          "value": 30,
          "name": "cell44 ~ gene38"
        },
        {
          "x": 43,
          "y": 38,
          "value": 13,
          "name": "cell44 ~ gene39"
        },
        {
          "x": 43,
          "y": 39,
          "value": 21,
          "name": "cell44 ~ gene40"
        },
        {
          "x": 43,
          "y": 40,
          "value": 6,
          "name": "cell44 ~ gene41"
        },
        {
          "x": 43,
          "y": 41,
          "value": 0,
          "name": "cell44 ~ gene42"
        },
        {
          "x": 43,
          "y": 42,
          "value": 3,
          "name": "cell44 ~ gene43"
        },
        {
          "x": 43,
          "y": 43,
          "value": 0,
          "name": "cell44 ~ gene44"
        },
        {
          "x": 43,
          "y": 44,
          "value": 0,
          "name": "cell44 ~ gene45"
        },
        {
          "x": 43,
          "y": 45,
          "value": 11,
          "name": "cell44 ~ gene46"
        },
        {
          "x": 43,
          "y": 46,
          "value": 0,
          "name": "cell44 ~ gene47"
        },
        {
          "x": 43,
          "y": 47,
          "value": 0,
          "name": "cell44 ~ gene48"
        },
        {
          "x": 43,
          "y": 48,
          "value": 0,
          "name": "cell44 ~ gene49"
        },
        {
          "x": 43,
          "y": 49,
          "value": 9,
          "name": "cell44 ~ gene50"
        },
        {
          "x": 43,
          "y": 50,
          "value": 5,
          "name": "cell44 ~ gene51"
        },
        {
          "x": 43,
          "y": 51,
          "value": 6,
          "name": "cell44 ~ gene52"
        },
        {
          "x": 43,
          "y": 52,
          "value": 11,
          "name": "cell44 ~ gene53"
        },
        {
          "x": 43,
          "y": 53,
          "value": 0,
          "name": "cell44 ~ gene54"
        },
        {
          "x": 43,
          "y": 54,
          "value": 2,
          "name": "cell44 ~ gene55"
        },
        {
          "x": 43,
          "y": 55,
          "value": 0,
          "name": "cell44 ~ gene56"
        },
        {
          "x": 43,
          "y": 56,
          "value": 16,
          "name": "cell44 ~ gene57"
        },
        {
          "x": 43,
          "y": 57,
          "value": 20,
          "name": "cell44 ~ gene58"
        },
        {
          "x": 43,
          "y": 58,
          "value": 15,
          "name": "cell44 ~ gene59"
        },
        {
          "x": 43,
          "y": 59,
          "value": 8,
          "name": "cell44 ~ gene60"
        },
        {
          "x": 43,
          "y": 60,
          "value": 14,
          "name": "cell44 ~ gene61"
        },
        {
          "x": 43,
          "y": 61,
          "value": 0,
          "name": "cell44 ~ gene62"
        },
        {
          "x": 43,
          "y": 62,
          "value": 0,
          "name": "cell44 ~ gene63"
        },
        {
          "x": 43,
          "y": 63,
          "value": 16,
          "name": "cell44 ~ gene64"
        },
        {
          "x": 43,
          "y": 64,
          "value": 0,
          "name": "cell44 ~ gene65"
        },
        {
          "x": 43,
          "y": 65,
          "value": 1,
          "name": "cell44 ~ gene66"
        },
        {
          "x": 43,
          "y": 66,
          "value": 17,
          "name": "cell44 ~ gene67"
        },
        {
          "x": 43,
          "y": 67,
          "value": 13,
          "name": "cell44 ~ gene68"
        },
        {
          "x": 43,
          "y": 68,
          "value": 0,
          "name": "cell44 ~ gene69"
        },
        {
          "x": 43,
          "y": 69,
          "value": 11,
          "name": "cell44 ~ gene70"
        },
        {
          "x": 43,
          "y": 70,
          "value": 0,
          "name": "cell44 ~ gene71"
        },
        {
          "x": 43,
          "y": 71,
          "value": 0,
          "name": "cell44 ~ gene72"
        },
        {
          "x": 43,
          "y": 72,
          "value": 0,
          "name": "cell44 ~ gene73"
        },
        {
          "x": 43,
          "y": 73,
          "value": 0,
          "name": "cell44 ~ gene74"
        },
        {
          "x": 43,
          "y": 74,
          "value": 0,
          "name": "cell44 ~ gene75"
        },
        {
          "x": 43,
          "y": 75,
          "value": 21,
          "name": "cell44 ~ gene76"
        },
        {
          "x": 43,
          "y": 76,
          "value": 0,
          "name": "cell44 ~ gene77"
        },
        {
          "x": 43,
          "y": 77,
          "value": 0,
          "name": "cell44 ~ gene78"
        },
        {
          "x": 43,
          "y": 78,
          "value": 0,
          "name": "cell44 ~ gene79"
        },
        {
          "x": 43,
          "y": 79,
          "value": 1,
          "name": "cell44 ~ gene80"
        },
        {
          "x": 43,
          "y": 80,
          "value": 0,
          "name": "cell44 ~ gene81"
        },
        {
          "x": 43,
          "y": 81,
          "value": 0,
          "name": "cell44 ~ gene82"
        },
        {
          "x": 43,
          "y": 82,
          "value": 0,
          "name": "cell44 ~ gene83"
        },
        {
          "x": 43,
          "y": 83,
          "value": 0,
          "name": "cell44 ~ gene84"
        },
        {
          "x": 43,
          "y": 84,
          "value": 7,
          "name": "cell44 ~ gene85"
        },
        {
          "x": 43,
          "y": 85,
          "value": 7,
          "name": "cell44 ~ gene86"
        },
        {
          "x": 43,
          "y": 86,
          "value": 0,
          "name": "cell44 ~ gene87"
        },
        {
          "x": 43,
          "y": 87,
          "value": 7,
          "name": "cell44 ~ gene88"
        },
        {
          "x": 43,
          "y": 88,
          "value": 0,
          "name": "cell44 ~ gene89"
        },
        {
          "x": 43,
          "y": 89,
          "value": 0,
          "name": "cell44 ~ gene90"
        },
        {
          "x": 43,
          "y": 90,
          "value": 4,
          "name": "cell44 ~ gene91"
        },
        {
          "x": 43,
          "y": 91,
          "value": 0,
          "name": "cell44 ~ gene92"
        },
        {
          "x": 43,
          "y": 92,
          "value": 0,
          "name": "cell44 ~ gene93"
        },
        {
          "x": 43,
          "y": 93,
          "value": 9,
          "name": "cell44 ~ gene94"
        },
        {
          "x": 43,
          "y": 94,
          "value": 0,
          "name": "cell44 ~ gene95"
        },
        {
          "x": 43,
          "y": 95,
          "value": 0,
          "name": "cell44 ~ gene96"
        },
        {
          "x": 43,
          "y": 96,
          "value": 0,
          "name": "cell44 ~ gene97"
        },
        {
          "x": 43,
          "y": 97,
          "value": 0,
          "name": "cell44 ~ gene98"
        },
        {
          "x": 43,
          "y": 98,
          "value": 0,
          "name": "cell44 ~ gene99"
        },
        {
          "x": 43,
          "y": 99,
          "value": 0,
          "name": "cell44 ~ gene100"
        },
        {
          "x": 44,
          "y": 0,
          "value": 6,
          "name": "cell45 ~ gene1"
        },
        {
          "x": 44,
          "y": 1,
          "value": 4,
          "name": "cell45 ~ gene2"
        },
        {
          "x": 44,
          "y": 2,
          "value": 8,
          "name": "cell45 ~ gene3"
        },
        {
          "x": 44,
          "y": 3,
          "value": 0,
          "name": "cell45 ~ gene4"
        },
        {
          "x": 44,
          "y": 4,
          "value": 0,
          "name": "cell45 ~ gene5"
        },
        {
          "x": 44,
          "y": 5,
          "value": 0,
          "name": "cell45 ~ gene6"
        },
        {
          "x": 44,
          "y": 6,
          "value": 0,
          "name": "cell45 ~ gene7"
        },
        {
          "x": 44,
          "y": 7,
          "value": 9,
          "name": "cell45 ~ gene8"
        },
        {
          "x": 44,
          "y": 8,
          "value": 12,
          "name": "cell45 ~ gene9"
        },
        {
          "x": 44,
          "y": 9,
          "value": 16,
          "name": "cell45 ~ gene10"
        },
        {
          "x": 44,
          "y": 10,
          "value": 0,
          "name": "cell45 ~ gene11"
        },
        {
          "x": 44,
          "y": 11,
          "value": 0,
          "name": "cell45 ~ gene12"
        },
        {
          "x": 44,
          "y": 12,
          "value": 8,
          "name": "cell45 ~ gene13"
        },
        {
          "x": 44,
          "y": 13,
          "value": 9,
          "name": "cell45 ~ gene14"
        },
        {
          "x": 44,
          "y": 14,
          "value": 6,
          "name": "cell45 ~ gene15"
        },
        {
          "x": 44,
          "y": 15,
          "value": 16,
          "name": "cell45 ~ gene16"
        },
        {
          "x": 44,
          "y": 16,
          "value": 0,
          "name": "cell45 ~ gene17"
        },
        {
          "x": 44,
          "y": 17,
          "value": 0,
          "name": "cell45 ~ gene18"
        },
        {
          "x": 44,
          "y": 18,
          "value": 10,
          "name": "cell45 ~ gene19"
        },
        {
          "x": 44,
          "y": 19,
          "value": 2,
          "name": "cell45 ~ gene20"
        },
        {
          "x": 44,
          "y": 20,
          "value": 0,
          "name": "cell45 ~ gene21"
        },
        {
          "x": 44,
          "y": 21,
          "value": 22,
          "name": "cell45 ~ gene22"
        },
        {
          "x": 44,
          "y": 22,
          "value": 1,
          "name": "cell45 ~ gene23"
        },
        {
          "x": 44,
          "y": 23,
          "value": 0,
          "name": "cell45 ~ gene24"
        },
        {
          "x": 44,
          "y": 24,
          "value": 6,
          "name": "cell45 ~ gene25"
        },
        {
          "x": 44,
          "y": 25,
          "value": 14,
          "name": "cell45 ~ gene26"
        },
        {
          "x": 44,
          "y": 26,
          "value": 11,
          "name": "cell45 ~ gene27"
        },
        {
          "x": 44,
          "y": 27,
          "value": 3,
          "name": "cell45 ~ gene28"
        },
        {
          "x": 44,
          "y": 28,
          "value": 0,
          "name": "cell45 ~ gene29"
        },
        {
          "x": 44,
          "y": 29,
          "value": 23,
          "name": "cell45 ~ gene30"
        },
        {
          "x": 44,
          "y": 30,
          "value": 37,
          "name": "cell45 ~ gene31"
        },
        {
          "x": 44,
          "y": 31,
          "value": 5,
          "name": "cell45 ~ gene32"
        },
        {
          "x": 44,
          "y": 32,
          "value": 0,
          "name": "cell45 ~ gene33"
        },
        {
          "x": 44,
          "y": 33,
          "value": 9,
          "name": "cell45 ~ gene34"
        },
        {
          "x": 44,
          "y": 34,
          "value": 15,
          "name": "cell45 ~ gene35"
        },
        {
          "x": 44,
          "y": 35,
          "value": 3,
          "name": "cell45 ~ gene36"
        },
        {
          "x": 44,
          "y": 36,
          "value": 13,
          "name": "cell45 ~ gene37"
        },
        {
          "x": 44,
          "y": 37,
          "value": 15,
          "name": "cell45 ~ gene38"
        },
        {
          "x": 44,
          "y": 38,
          "value": 0,
          "name": "cell45 ~ gene39"
        },
        {
          "x": 44,
          "y": 39,
          "value": 8,
          "name": "cell45 ~ gene40"
        },
        {
          "x": 44,
          "y": 40,
          "value": 0,
          "name": "cell45 ~ gene41"
        },
        {
          "x": 44,
          "y": 41,
          "value": 5,
          "name": "cell45 ~ gene42"
        },
        {
          "x": 44,
          "y": 42,
          "value": 0,
          "name": "cell45 ~ gene43"
        },
        {
          "x": 44,
          "y": 43,
          "value": 18,
          "name": "cell45 ~ gene44"
        },
        {
          "x": 44,
          "y": 44,
          "value": 0,
          "name": "cell45 ~ gene45"
        },
        {
          "x": 44,
          "y": 45,
          "value": 1,
          "name": "cell45 ~ gene46"
        },
        {
          "x": 44,
          "y": 46,
          "value": 0,
          "name": "cell45 ~ gene47"
        },
        {
          "x": 44,
          "y": 47,
          "value": 7,
          "name": "cell45 ~ gene48"
        },
        {
          "x": 44,
          "y": 48,
          "value": 5,
          "name": "cell45 ~ gene49"
        },
        {
          "x": 44,
          "y": 49,
          "value": 14,
          "name": "cell45 ~ gene50"
        },
        {
          "x": 44,
          "y": 50,
          "value": 0,
          "name": "cell45 ~ gene51"
        },
        {
          "x": 44,
          "y": 51,
          "value": 0,
          "name": "cell45 ~ gene52"
        },
        {
          "x": 44,
          "y": 52,
          "value": 7,
          "name": "cell45 ~ gene53"
        },
        {
          "x": 44,
          "y": 53,
          "value": 0,
          "name": "cell45 ~ gene54"
        },
        {
          "x": 44,
          "y": 54,
          "value": 0,
          "name": "cell45 ~ gene55"
        },
        {
          "x": 44,
          "y": 55,
          "value": 0,
          "name": "cell45 ~ gene56"
        },
        {
          "x": 44,
          "y": 56,
          "value": 2,
          "name": "cell45 ~ gene57"
        },
        {
          "x": 44,
          "y": 57,
          "value": 0,
          "name": "cell45 ~ gene58"
        },
        {
          "x": 44,
          "y": 58,
          "value": 0,
          "name": "cell45 ~ gene59"
        },
        {
          "x": 44,
          "y": 59,
          "value": 0,
          "name": "cell45 ~ gene60"
        },
        {
          "x": 44,
          "y": 60,
          "value": 0,
          "name": "cell45 ~ gene61"
        },
        {
          "x": 44,
          "y": 61,
          "value": 0,
          "name": "cell45 ~ gene62"
        },
        {
          "x": 44,
          "y": 62,
          "value": 7,
          "name": "cell45 ~ gene63"
        },
        {
          "x": 44,
          "y": 63,
          "value": 9,
          "name": "cell45 ~ gene64"
        },
        {
          "x": 44,
          "y": 64,
          "value": 4,
          "name": "cell45 ~ gene65"
        },
        {
          "x": 44,
          "y": 65,
          "value": 9,
          "name": "cell45 ~ gene66"
        },
        {
          "x": 44,
          "y": 66,
          "value": 9,
          "name": "cell45 ~ gene67"
        },
        {
          "x": 44,
          "y": 67,
          "value": 0,
          "name": "cell45 ~ gene68"
        },
        {
          "x": 44,
          "y": 68,
          "value": 10,
          "name": "cell45 ~ gene69"
        },
        {
          "x": 44,
          "y": 69,
          "value": 0,
          "name": "cell45 ~ gene70"
        },
        {
          "x": 44,
          "y": 70,
          "value": 5,
          "name": "cell45 ~ gene71"
        },
        {
          "x": 44,
          "y": 71,
          "value": 0,
          "name": "cell45 ~ gene72"
        },
        {
          "x": 44,
          "y": 72,
          "value": 10,
          "name": "cell45 ~ gene73"
        },
        {
          "x": 44,
          "y": 73,
          "value": 0,
          "name": "cell45 ~ gene74"
        },
        {
          "x": 44,
          "y": 74,
          "value": 0,
          "name": "cell45 ~ gene75"
        },
        {
          "x": 44,
          "y": 75,
          "value": 22,
          "name": "cell45 ~ gene76"
        },
        {
          "x": 44,
          "y": 76,
          "value": 0,
          "name": "cell45 ~ gene77"
        },
        {
          "x": 44,
          "y": 77,
          "value": 8,
          "name": "cell45 ~ gene78"
        },
        {
          "x": 44,
          "y": 78,
          "value": 22,
          "name": "cell45 ~ gene79"
        },
        {
          "x": 44,
          "y": 79,
          "value": 15,
          "name": "cell45 ~ gene80"
        },
        {
          "x": 44,
          "y": 80,
          "value": 12,
          "name": "cell45 ~ gene81"
        },
        {
          "x": 44,
          "y": 81,
          "value": 0,
          "name": "cell45 ~ gene82"
        },
        {
          "x": 44,
          "y": 82,
          "value": 0,
          "name": "cell45 ~ gene83"
        },
        {
          "x": 44,
          "y": 83,
          "value": 1,
          "name": "cell45 ~ gene84"
        },
        {
          "x": 44,
          "y": 84,
          "value": 0,
          "name": "cell45 ~ gene85"
        },
        {
          "x": 44,
          "y": 85,
          "value": 0,
          "name": "cell45 ~ gene86"
        },
        {
          "x": 44,
          "y": 86,
          "value": 4,
          "name": "cell45 ~ gene87"
        },
        {
          "x": 44,
          "y": 87,
          "value": 11,
          "name": "cell45 ~ gene88"
        },
        {
          "x": 44,
          "y": 88,
          "value": 3,
          "name": "cell45 ~ gene89"
        },
        {
          "x": 44,
          "y": 89,
          "value": 6,
          "name": "cell45 ~ gene90"
        },
        {
          "x": 44,
          "y": 90,
          "value": 0,
          "name": "cell45 ~ gene91"
        },
        {
          "x": 44,
          "y": 91,
          "value": 4,
          "name": "cell45 ~ gene92"
        },
        {
          "x": 44,
          "y": 92,
          "value": 0,
          "name": "cell45 ~ gene93"
        },
        {
          "x": 44,
          "y": 93,
          "value": 0,
          "name": "cell45 ~ gene94"
        },
        {
          "x": 44,
          "y": 94,
          "value": 0,
          "name": "cell45 ~ gene95"
        },
        {
          "x": 44,
          "y": 95,
          "value": 0,
          "name": "cell45 ~ gene96"
        },
        {
          "x": 44,
          "y": 96,
          "value": 5,
          "name": "cell45 ~ gene97"
        },
        {
          "x": 44,
          "y": 97,
          "value": 9,
          "name": "cell45 ~ gene98"
        },
        {
          "x": 44,
          "y": 98,
          "value": 11,
          "name": "cell45 ~ gene99"
        },
        {
          "x": 44,
          "y": 99,
          "value": 8,
          "name": "cell45 ~ gene100"
        },
        {
          "x": 45,
          "y": 0,
          "value": 0,
          "name": "cell46 ~ gene1"
        },
        {
          "x": 45,
          "y": 1,
          "value": 27,
          "name": "cell46 ~ gene2"
        },
        {
          "x": 45,
          "y": 2,
          "value": 0,
          "name": "cell46 ~ gene3"
        },
        {
          "x": 45,
          "y": 3,
          "value": 6,
          "name": "cell46 ~ gene4"
        },
        {
          "x": 45,
          "y": 4,
          "value": 6,
          "name": "cell46 ~ gene5"
        },
        {
          "x": 45,
          "y": 5,
          "value": 0,
          "name": "cell46 ~ gene6"
        },
        {
          "x": 45,
          "y": 6,
          "value": 6,
          "name": "cell46 ~ gene7"
        },
        {
          "x": 45,
          "y": 7,
          "value": 6,
          "name": "cell46 ~ gene8"
        },
        {
          "x": 45,
          "y": 8,
          "value": 0,
          "name": "cell46 ~ gene9"
        },
        {
          "x": 45,
          "y": 9,
          "value": 0,
          "name": "cell46 ~ gene10"
        },
        {
          "x": 45,
          "y": 10,
          "value": 0,
          "name": "cell46 ~ gene11"
        },
        {
          "x": 45,
          "y": 11,
          "value": 0,
          "name": "cell46 ~ gene12"
        },
        {
          "x": 45,
          "y": 12,
          "value": 10,
          "name": "cell46 ~ gene13"
        },
        {
          "x": 45,
          "y": 13,
          "value": 0,
          "name": "cell46 ~ gene14"
        },
        {
          "x": 45,
          "y": 14,
          "value": 0,
          "name": "cell46 ~ gene15"
        },
        {
          "x": 45,
          "y": 15,
          "value": 8,
          "name": "cell46 ~ gene16"
        },
        {
          "x": 45,
          "y": 16,
          "value": 9,
          "name": "cell46 ~ gene17"
        },
        {
          "x": 45,
          "y": 17,
          "value": 3,
          "name": "cell46 ~ gene18"
        },
        {
          "x": 45,
          "y": 18,
          "value": 12,
          "name": "cell46 ~ gene19"
        },
        {
          "x": 45,
          "y": 19,
          "value": 17,
          "name": "cell46 ~ gene20"
        },
        {
          "x": 45,
          "y": 20,
          "value": 0,
          "name": "cell46 ~ gene21"
        },
        {
          "x": 45,
          "y": 21,
          "value": 41,
          "name": "cell46 ~ gene22"
        },
        {
          "x": 45,
          "y": 22,
          "value": 7,
          "name": "cell46 ~ gene23"
        },
        {
          "x": 45,
          "y": 23,
          "value": 0,
          "name": "cell46 ~ gene24"
        },
        {
          "x": 45,
          "y": 24,
          "value": 17,
          "name": "cell46 ~ gene25"
        },
        {
          "x": 45,
          "y": 25,
          "value": 13,
          "name": "cell46 ~ gene26"
        },
        {
          "x": 45,
          "y": 26,
          "value": 15,
          "name": "cell46 ~ gene27"
        },
        {
          "x": 45,
          "y": 27,
          "value": 0,
          "name": "cell46 ~ gene28"
        },
        {
          "x": 45,
          "y": 28,
          "value": 0,
          "name": "cell46 ~ gene29"
        },
        {
          "x": 45,
          "y": 29,
          "value": 26,
          "name": "cell46 ~ gene30"
        },
        {
          "x": 45,
          "y": 30,
          "value": 13,
          "name": "cell46 ~ gene31"
        },
        {
          "x": 45,
          "y": 31,
          "value": 8,
          "name": "cell46 ~ gene32"
        },
        {
          "x": 45,
          "y": 32,
          "value": 16,
          "name": "cell46 ~ gene33"
        },
        {
          "x": 45,
          "y": 33,
          "value": 9,
          "name": "cell46 ~ gene34"
        },
        {
          "x": 45,
          "y": 34,
          "value": 10,
          "name": "cell46 ~ gene35"
        },
        {
          "x": 45,
          "y": 35,
          "value": 2,
          "name": "cell46 ~ gene36"
        },
        {
          "x": 45,
          "y": 36,
          "value": 27,
          "name": "cell46 ~ gene37"
        },
        {
          "x": 45,
          "y": 37,
          "value": 21,
          "name": "cell46 ~ gene38"
        },
        {
          "x": 45,
          "y": 38,
          "value": 0,
          "name": "cell46 ~ gene39"
        },
        {
          "x": 45,
          "y": 39,
          "value": 0,
          "name": "cell46 ~ gene40"
        },
        {
          "x": 45,
          "y": 40,
          "value": 2,
          "name": "cell46 ~ gene41"
        },
        {
          "x": 45,
          "y": 41,
          "value": 4,
          "name": "cell46 ~ gene42"
        },
        {
          "x": 45,
          "y": 42,
          "value": 0,
          "name": "cell46 ~ gene43"
        },
        {
          "x": 45,
          "y": 43,
          "value": 13,
          "name": "cell46 ~ gene44"
        },
        {
          "x": 45,
          "y": 44,
          "value": 0,
          "name": "cell46 ~ gene45"
        },
        {
          "x": 45,
          "y": 45,
          "value": 0,
          "name": "cell46 ~ gene46"
        },
        {
          "x": 45,
          "y": 46,
          "value": 7,
          "name": "cell46 ~ gene47"
        },
        {
          "x": 45,
          "y": 47,
          "value": 8,
          "name": "cell46 ~ gene48"
        },
        {
          "x": 45,
          "y": 48,
          "value": 9,
          "name": "cell46 ~ gene49"
        },
        {
          "x": 45,
          "y": 49,
          "value": 0,
          "name": "cell46 ~ gene50"
        },
        {
          "x": 45,
          "y": 50,
          "value": 0,
          "name": "cell46 ~ gene51"
        },
        {
          "x": 45,
          "y": 51,
          "value": 0,
          "name": "cell46 ~ gene52"
        },
        {
          "x": 45,
          "y": 52,
          "value": 3,
          "name": "cell46 ~ gene53"
        },
        {
          "x": 45,
          "y": 53,
          "value": 6,
          "name": "cell46 ~ gene54"
        },
        {
          "x": 45,
          "y": 54,
          "value": 5,
          "name": "cell46 ~ gene55"
        },
        {
          "x": 45,
          "y": 55,
          "value": 4,
          "name": "cell46 ~ gene56"
        },
        {
          "x": 45,
          "y": 56,
          "value": 2,
          "name": "cell46 ~ gene57"
        },
        {
          "x": 45,
          "y": 57,
          "value": 10,
          "name": "cell46 ~ gene58"
        },
        {
          "x": 45,
          "y": 58,
          "value": 8,
          "name": "cell46 ~ gene59"
        },
        {
          "x": 45,
          "y": 59,
          "value": 3,
          "name": "cell46 ~ gene60"
        },
        {
          "x": 45,
          "y": 60,
          "value": 0,
          "name": "cell46 ~ gene61"
        },
        {
          "x": 45,
          "y": 61,
          "value": 6,
          "name": "cell46 ~ gene62"
        },
        {
          "x": 45,
          "y": 62,
          "value": 0,
          "name": "cell46 ~ gene63"
        },
        {
          "x": 45,
          "y": 63,
          "value": 0,
          "name": "cell46 ~ gene64"
        },
        {
          "x": 45,
          "y": 64,
          "value": 18,
          "name": "cell46 ~ gene65"
        },
        {
          "x": 45,
          "y": 65,
          "value": 6,
          "name": "cell46 ~ gene66"
        },
        {
          "x": 45,
          "y": 66,
          "value": 12,
          "name": "cell46 ~ gene67"
        },
        {
          "x": 45,
          "y": 67,
          "value": 1,
          "name": "cell46 ~ gene68"
        },
        {
          "x": 45,
          "y": 68,
          "value": 0,
          "name": "cell46 ~ gene69"
        },
        {
          "x": 45,
          "y": 69,
          "value": 10,
          "name": "cell46 ~ gene70"
        },
        {
          "x": 45,
          "y": 70,
          "value": 5,
          "name": "cell46 ~ gene71"
        },
        {
          "x": 45,
          "y": 71,
          "value": 0,
          "name": "cell46 ~ gene72"
        },
        {
          "x": 45,
          "y": 72,
          "value": 2,
          "name": "cell46 ~ gene73"
        },
        {
          "x": 45,
          "y": 73,
          "value": 11,
          "name": "cell46 ~ gene74"
        },
        {
          "x": 45,
          "y": 74,
          "value": 4,
          "name": "cell46 ~ gene75"
        },
        {
          "x": 45,
          "y": 75,
          "value": 6,
          "name": "cell46 ~ gene76"
        },
        {
          "x": 45,
          "y": 76,
          "value": 11,
          "name": "cell46 ~ gene77"
        },
        {
          "x": 45,
          "y": 77,
          "value": 0,
          "name": "cell46 ~ gene78"
        },
        {
          "x": 45,
          "y": 78,
          "value": 5,
          "name": "cell46 ~ gene79"
        },
        {
          "x": 45,
          "y": 79,
          "value": 2,
          "name": "cell46 ~ gene80"
        },
        {
          "x": 45,
          "y": 80,
          "value": 0,
          "name": "cell46 ~ gene81"
        },
        {
          "x": 45,
          "y": 81,
          "value": 0,
          "name": "cell46 ~ gene82"
        },
        {
          "x": 45,
          "y": 82,
          "value": 0,
          "name": "cell46 ~ gene83"
        },
        {
          "x": 45,
          "y": 83,
          "value": 15,
          "name": "cell46 ~ gene84"
        },
        {
          "x": 45,
          "y": 84,
          "value": 0,
          "name": "cell46 ~ gene85"
        },
        {
          "x": 45,
          "y": 85,
          "value": 2,
          "name": "cell46 ~ gene86"
        },
        {
          "x": 45,
          "y": 86,
          "value": 12,
          "name": "cell46 ~ gene87"
        },
        {
          "x": 45,
          "y": 87,
          "value": 0,
          "name": "cell46 ~ gene88"
        },
        {
          "x": 45,
          "y": 88,
          "value": 13,
          "name": "cell46 ~ gene89"
        },
        {
          "x": 45,
          "y": 89,
          "value": 0,
          "name": "cell46 ~ gene90"
        },
        {
          "x": 45,
          "y": 90,
          "value": 0,
          "name": "cell46 ~ gene91"
        },
        {
          "x": 45,
          "y": 91,
          "value": 7,
          "name": "cell46 ~ gene92"
        },
        {
          "x": 45,
          "y": 92,
          "value": 0,
          "name": "cell46 ~ gene93"
        },
        {
          "x": 45,
          "y": 93,
          "value": 0,
          "name": "cell46 ~ gene94"
        },
        {
          "x": 45,
          "y": 94,
          "value": 27,
          "name": "cell46 ~ gene95"
        },
        {
          "x": 45,
          "y": 95,
          "value": 0,
          "name": "cell46 ~ gene96"
        },
        {
          "x": 45,
          "y": 96,
          "value": 13,
          "name": "cell46 ~ gene97"
        },
        {
          "x": 45,
          "y": 97,
          "value": 8,
          "name": "cell46 ~ gene98"
        },
        {
          "x": 45,
          "y": 98,
          "value": 3,
          "name": "cell46 ~ gene99"
        },
        {
          "x": 45,
          "y": 99,
          "value": 10,
          "name": "cell46 ~ gene100"
        },
        {
          "x": 46,
          "y": 0,
          "value": 0,
          "name": "cell47 ~ gene1"
        },
        {
          "x": 46,
          "y": 1,
          "value": 1,
          "name": "cell47 ~ gene2"
        },
        {
          "x": 46,
          "y": 2,
          "value": 0,
          "name": "cell47 ~ gene3"
        },
        {
          "x": 46,
          "y": 3,
          "value": 18,
          "name": "cell47 ~ gene4"
        },
        {
          "x": 46,
          "y": 4,
          "value": 0,
          "name": "cell47 ~ gene5"
        },
        {
          "x": 46,
          "y": 5,
          "value": 0,
          "name": "cell47 ~ gene6"
        },
        {
          "x": 46,
          "y": 6,
          "value": 1,
          "name": "cell47 ~ gene7"
        },
        {
          "x": 46,
          "y": 7,
          "value": 12,
          "name": "cell47 ~ gene8"
        },
        {
          "x": 46,
          "y": 8,
          "value": 0,
          "name": "cell47 ~ gene9"
        },
        {
          "x": 46,
          "y": 9,
          "value": 1,
          "name": "cell47 ~ gene10"
        },
        {
          "x": 46,
          "y": 10,
          "value": 0,
          "name": "cell47 ~ gene11"
        },
        {
          "x": 46,
          "y": 11,
          "value": 5,
          "name": "cell47 ~ gene12"
        },
        {
          "x": 46,
          "y": 12,
          "value": 9,
          "name": "cell47 ~ gene13"
        },
        {
          "x": 46,
          "y": 13,
          "value": 0,
          "name": "cell47 ~ gene14"
        },
        {
          "x": 46,
          "y": 14,
          "value": 0,
          "name": "cell47 ~ gene15"
        },
        {
          "x": 46,
          "y": 15,
          "value": 13,
          "name": "cell47 ~ gene16"
        },
        {
          "x": 46,
          "y": 16,
          "value": 10,
          "name": "cell47 ~ gene17"
        },
        {
          "x": 46,
          "y": 17,
          "value": 2,
          "name": "cell47 ~ gene18"
        },
        {
          "x": 46,
          "y": 18,
          "value": 0,
          "name": "cell47 ~ gene19"
        },
        {
          "x": 46,
          "y": 19,
          "value": 0,
          "name": "cell47 ~ gene20"
        },
        {
          "x": 46,
          "y": 20,
          "value": 4,
          "name": "cell47 ~ gene21"
        },
        {
          "x": 46,
          "y": 21,
          "value": 17,
          "name": "cell47 ~ gene22"
        },
        {
          "x": 46,
          "y": 22,
          "value": 4,
          "name": "cell47 ~ gene23"
        },
        {
          "x": 46,
          "y": 23,
          "value": 5,
          "name": "cell47 ~ gene24"
        },
        {
          "x": 46,
          "y": 24,
          "value": 17,
          "name": "cell47 ~ gene25"
        },
        {
          "x": 46,
          "y": 25,
          "value": 11,
          "name": "cell47 ~ gene26"
        },
        {
          "x": 46,
          "y": 26,
          "value": 3,
          "name": "cell47 ~ gene27"
        },
        {
          "x": 46,
          "y": 27,
          "value": 0,
          "name": "cell47 ~ gene28"
        },
        {
          "x": 46,
          "y": 28,
          "value": 22,
          "name": "cell47 ~ gene29"
        },
        {
          "x": 46,
          "y": 29,
          "value": 14,
          "name": "cell47 ~ gene30"
        },
        {
          "x": 46,
          "y": 30,
          "value": 22,
          "name": "cell47 ~ gene31"
        },
        {
          "x": 46,
          "y": 31,
          "value": 0,
          "name": "cell47 ~ gene32"
        },
        {
          "x": 46,
          "y": 32,
          "value": 4,
          "name": "cell47 ~ gene33"
        },
        {
          "x": 46,
          "y": 33,
          "value": 4,
          "name": "cell47 ~ gene34"
        },
        {
          "x": 46,
          "y": 34,
          "value": 22,
          "name": "cell47 ~ gene35"
        },
        {
          "x": 46,
          "y": 35,
          "value": 11,
          "name": "cell47 ~ gene36"
        },
        {
          "x": 46,
          "y": 36,
          "value": 19,
          "name": "cell47 ~ gene37"
        },
        {
          "x": 46,
          "y": 37,
          "value": 17,
          "name": "cell47 ~ gene38"
        },
        {
          "x": 46,
          "y": 38,
          "value": 21,
          "name": "cell47 ~ gene39"
        },
        {
          "x": 46,
          "y": 39,
          "value": 16,
          "name": "cell47 ~ gene40"
        },
        {
          "x": 46,
          "y": 40,
          "value": 0,
          "name": "cell47 ~ gene41"
        },
        {
          "x": 46,
          "y": 41,
          "value": 2,
          "name": "cell47 ~ gene42"
        },
        {
          "x": 46,
          "y": 42,
          "value": 6,
          "name": "cell47 ~ gene43"
        },
        {
          "x": 46,
          "y": 43,
          "value": 0,
          "name": "cell47 ~ gene44"
        },
        {
          "x": 46,
          "y": 44,
          "value": 0,
          "name": "cell47 ~ gene45"
        },
        {
          "x": 46,
          "y": 45,
          "value": 0,
          "name": "cell47 ~ gene46"
        },
        {
          "x": 46,
          "y": 46,
          "value": 5,
          "name": "cell47 ~ gene47"
        },
        {
          "x": 46,
          "y": 47,
          "value": 0,
          "name": "cell47 ~ gene48"
        },
        {
          "x": 46,
          "y": 48,
          "value": 0,
          "name": "cell47 ~ gene49"
        },
        {
          "x": 46,
          "y": 49,
          "value": 26,
          "name": "cell47 ~ gene50"
        },
        {
          "x": 46,
          "y": 50,
          "value": 6,
          "name": "cell47 ~ gene51"
        },
        {
          "x": 46,
          "y": 51,
          "value": 0,
          "name": "cell47 ~ gene52"
        },
        {
          "x": 46,
          "y": 52,
          "value": 2,
          "name": "cell47 ~ gene53"
        },
        {
          "x": 46,
          "y": 53,
          "value": 0,
          "name": "cell47 ~ gene54"
        },
        {
          "x": 46,
          "y": 54,
          "value": 0,
          "name": "cell47 ~ gene55"
        },
        {
          "x": 46,
          "y": 55,
          "value": 0,
          "name": "cell47 ~ gene56"
        },
        {
          "x": 46,
          "y": 56,
          "value": 0,
          "name": "cell47 ~ gene57"
        },
        {
          "x": 46,
          "y": 57,
          "value": 0,
          "name": "cell47 ~ gene58"
        },
        {
          "x": 46,
          "y": 58,
          "value": 15,
          "name": "cell47 ~ gene59"
        },
        {
          "x": 46,
          "y": 59,
          "value": 2,
          "name": "cell47 ~ gene60"
        },
        {
          "x": 46,
          "y": 60,
          "value": 2,
          "name": "cell47 ~ gene61"
        },
        {
          "x": 46,
          "y": 61,
          "value": 0,
          "name": "cell47 ~ gene62"
        },
        {
          "x": 46,
          "y": 62,
          "value": 0,
          "name": "cell47 ~ gene63"
        },
        {
          "x": 46,
          "y": 63,
          "value": 0,
          "name": "cell47 ~ gene64"
        },
        {
          "x": 46,
          "y": 64,
          "value": 0,
          "name": "cell47 ~ gene65"
        },
        {
          "x": 46,
          "y": 65,
          "value": 14,
          "name": "cell47 ~ gene66"
        },
        {
          "x": 46,
          "y": 66,
          "value": 0,
          "name": "cell47 ~ gene67"
        },
        {
          "x": 46,
          "y": 67,
          "value": 0,
          "name": "cell47 ~ gene68"
        },
        {
          "x": 46,
          "y": 68,
          "value": 21,
          "name": "cell47 ~ gene69"
        },
        {
          "x": 46,
          "y": 69,
          "value": 6,
          "name": "cell47 ~ gene70"
        },
        {
          "x": 46,
          "y": 70,
          "value": 0,
          "name": "cell47 ~ gene71"
        },
        {
          "x": 46,
          "y": 71,
          "value": 11,
          "name": "cell47 ~ gene72"
        },
        {
          "x": 46,
          "y": 72,
          "value": 0,
          "name": "cell47 ~ gene73"
        },
        {
          "x": 46,
          "y": 73,
          "value": 7,
          "name": "cell47 ~ gene74"
        },
        {
          "x": 46,
          "y": 74,
          "value": 4,
          "name": "cell47 ~ gene75"
        },
        {
          "x": 46,
          "y": 75,
          "value": 0,
          "name": "cell47 ~ gene76"
        },
        {
          "x": 46,
          "y": 76,
          "value": 7,
          "name": "cell47 ~ gene77"
        },
        {
          "x": 46,
          "y": 77,
          "value": 0,
          "name": "cell47 ~ gene78"
        },
        {
          "x": 46,
          "y": 78,
          "value": 0,
          "name": "cell47 ~ gene79"
        },
        {
          "x": 46,
          "y": 79,
          "value": 13,
          "name": "cell47 ~ gene80"
        },
        {
          "x": 46,
          "y": 80,
          "value": 0,
          "name": "cell47 ~ gene81"
        },
        {
          "x": 46,
          "y": 81,
          "value": 0,
          "name": "cell47 ~ gene82"
        },
        {
          "x": 46,
          "y": 82,
          "value": 0,
          "name": "cell47 ~ gene83"
        },
        {
          "x": 46,
          "y": 83,
          "value": 0,
          "name": "cell47 ~ gene84"
        },
        {
          "x": 46,
          "y": 84,
          "value": 0,
          "name": "cell47 ~ gene85"
        },
        {
          "x": 46,
          "y": 85,
          "value": 13,
          "name": "cell47 ~ gene86"
        },
        {
          "x": 46,
          "y": 86,
          "value": 6,
          "name": "cell47 ~ gene87"
        },
        {
          "x": 46,
          "y": 87,
          "value": 13,
          "name": "cell47 ~ gene88"
        },
        {
          "x": 46,
          "y": 88,
          "value": 0,
          "name": "cell47 ~ gene89"
        },
        {
          "x": 46,
          "y": 89,
          "value": 0,
          "name": "cell47 ~ gene90"
        },
        {
          "x": 46,
          "y": 90,
          "value": 0,
          "name": "cell47 ~ gene91"
        },
        {
          "x": 46,
          "y": 91,
          "value": 0,
          "name": "cell47 ~ gene92"
        },
        {
          "x": 46,
          "y": 92,
          "value": 5,
          "name": "cell47 ~ gene93"
        },
        {
          "x": 46,
          "y": 93,
          "value": 7,
          "name": "cell47 ~ gene94"
        },
        {
          "x": 46,
          "y": 94,
          "value": 0,
          "name": "cell47 ~ gene95"
        },
        {
          "x": 46,
          "y": 95,
          "value": 8,
          "name": "cell47 ~ gene96"
        },
        {
          "x": 46,
          "y": 96,
          "value": 23,
          "name": "cell47 ~ gene97"
        },
        {
          "x": 46,
          "y": 97,
          "value": 12,
          "name": "cell47 ~ gene98"
        },
        {
          "x": 46,
          "y": 98,
          "value": 14,
          "name": "cell47 ~ gene99"
        },
        {
          "x": 46,
          "y": 99,
          "value": 0,
          "name": "cell47 ~ gene100"
        },
        {
          "x": 47,
          "y": 0,
          "value": 0,
          "name": "cell48 ~ gene1"
        },
        {
          "x": 47,
          "y": 1,
          "value": 21,
          "name": "cell48 ~ gene2"
        },
        {
          "x": 47,
          "y": 2,
          "value": 3,
          "name": "cell48 ~ gene3"
        },
        {
          "x": 47,
          "y": 3,
          "value": 0,
          "name": "cell48 ~ gene4"
        },
        {
          "x": 47,
          "y": 4,
          "value": 7,
          "name": "cell48 ~ gene5"
        },
        {
          "x": 47,
          "y": 5,
          "value": 13,
          "name": "cell48 ~ gene6"
        },
        {
          "x": 47,
          "y": 6,
          "value": 7,
          "name": "cell48 ~ gene7"
        },
        {
          "x": 47,
          "y": 7,
          "value": 16,
          "name": "cell48 ~ gene8"
        },
        {
          "x": 47,
          "y": 8,
          "value": 14,
          "name": "cell48 ~ gene9"
        },
        {
          "x": 47,
          "y": 9,
          "value": 2,
          "name": "cell48 ~ gene10"
        },
        {
          "x": 47,
          "y": 10,
          "value": 0,
          "name": "cell48 ~ gene11"
        },
        {
          "x": 47,
          "y": 11,
          "value": 13,
          "name": "cell48 ~ gene12"
        },
        {
          "x": 47,
          "y": 12,
          "value": 0,
          "name": "cell48 ~ gene13"
        },
        {
          "x": 47,
          "y": 13,
          "value": 15,
          "name": "cell48 ~ gene14"
        },
        {
          "x": 47,
          "y": 14,
          "value": 3,
          "name": "cell48 ~ gene15"
        },
        {
          "x": 47,
          "y": 15,
          "value": 0,
          "name": "cell48 ~ gene16"
        },
        {
          "x": 47,
          "y": 16,
          "value": 14,
          "name": "cell48 ~ gene17"
        },
        {
          "x": 47,
          "y": 17,
          "value": 3,
          "name": "cell48 ~ gene18"
        },
        {
          "x": 47,
          "y": 18,
          "value": 27,
          "name": "cell48 ~ gene19"
        },
        {
          "x": 47,
          "y": 19,
          "value": 4,
          "name": "cell48 ~ gene20"
        },
        {
          "x": 47,
          "y": 20,
          "value": 4,
          "name": "cell48 ~ gene21"
        },
        {
          "x": 47,
          "y": 21,
          "value": 13,
          "name": "cell48 ~ gene22"
        },
        {
          "x": 47,
          "y": 22,
          "value": 6,
          "name": "cell48 ~ gene23"
        },
        {
          "x": 47,
          "y": 23,
          "value": 0,
          "name": "cell48 ~ gene24"
        },
        {
          "x": 47,
          "y": 24,
          "value": 19,
          "name": "cell48 ~ gene25"
        },
        {
          "x": 47,
          "y": 25,
          "value": 0,
          "name": "cell48 ~ gene26"
        },
        {
          "x": 47,
          "y": 26,
          "value": 20,
          "name": "cell48 ~ gene27"
        },
        {
          "x": 47,
          "y": 27,
          "value": 0,
          "name": "cell48 ~ gene28"
        },
        {
          "x": 47,
          "y": 28,
          "value": 0,
          "name": "cell48 ~ gene29"
        },
        {
          "x": 47,
          "y": 29,
          "value": 17,
          "name": "cell48 ~ gene30"
        },
        {
          "x": 47,
          "y": 30,
          "value": 14,
          "name": "cell48 ~ gene31"
        },
        {
          "x": 47,
          "y": 31,
          "value": 18,
          "name": "cell48 ~ gene32"
        },
        {
          "x": 47,
          "y": 32,
          "value": 0,
          "name": "cell48 ~ gene33"
        },
        {
          "x": 47,
          "y": 33,
          "value": 7,
          "name": "cell48 ~ gene34"
        },
        {
          "x": 47,
          "y": 34,
          "value": 3,
          "name": "cell48 ~ gene35"
        },
        {
          "x": 47,
          "y": 35,
          "value": 0,
          "name": "cell48 ~ gene36"
        },
        {
          "x": 47,
          "y": 36,
          "value": 25,
          "name": "cell48 ~ gene37"
        },
        {
          "x": 47,
          "y": 37,
          "value": 8,
          "name": "cell48 ~ gene38"
        },
        {
          "x": 47,
          "y": 38,
          "value": 10,
          "name": "cell48 ~ gene39"
        },
        {
          "x": 47,
          "y": 39,
          "value": 10,
          "name": "cell48 ~ gene40"
        },
        {
          "x": 47,
          "y": 40,
          "value": 8,
          "name": "cell48 ~ gene41"
        },
        {
          "x": 47,
          "y": 41,
          "value": 26,
          "name": "cell48 ~ gene42"
        },
        {
          "x": 47,
          "y": 42,
          "value": 12,
          "name": "cell48 ~ gene43"
        },
        {
          "x": 47,
          "y": 43,
          "value": 0,
          "name": "cell48 ~ gene44"
        },
        {
          "x": 47,
          "y": 44,
          "value": 0,
          "name": "cell48 ~ gene45"
        },
        {
          "x": 47,
          "y": 45,
          "value": 0,
          "name": "cell48 ~ gene46"
        },
        {
          "x": 47,
          "y": 46,
          "value": 0,
          "name": "cell48 ~ gene47"
        },
        {
          "x": 47,
          "y": 47,
          "value": 0,
          "name": "cell48 ~ gene48"
        },
        {
          "x": 47,
          "y": 48,
          "value": 16,
          "name": "cell48 ~ gene49"
        },
        {
          "x": 47,
          "y": 49,
          "value": 3,
          "name": "cell48 ~ gene50"
        },
        {
          "x": 47,
          "y": 50,
          "value": 3,
          "name": "cell48 ~ gene51"
        },
        {
          "x": 47,
          "y": 51,
          "value": 25,
          "name": "cell48 ~ gene52"
        },
        {
          "x": 47,
          "y": 52,
          "value": 0,
          "name": "cell48 ~ gene53"
        },
        {
          "x": 47,
          "y": 53,
          "value": 0,
          "name": "cell48 ~ gene54"
        },
        {
          "x": 47,
          "y": 54,
          "value": 17,
          "name": "cell48 ~ gene55"
        },
        {
          "x": 47,
          "y": 55,
          "value": 0,
          "name": "cell48 ~ gene56"
        },
        {
          "x": 47,
          "y": 56,
          "value": 2,
          "name": "cell48 ~ gene57"
        },
        {
          "x": 47,
          "y": 57,
          "value": 0,
          "name": "cell48 ~ gene58"
        },
        {
          "x": 47,
          "y": 58,
          "value": 0,
          "name": "cell48 ~ gene59"
        },
        {
          "x": 47,
          "y": 59,
          "value": 0,
          "name": "cell48 ~ gene60"
        },
        {
          "x": 47,
          "y": 60,
          "value": 0,
          "name": "cell48 ~ gene61"
        },
        {
          "x": 47,
          "y": 61,
          "value": 0,
          "name": "cell48 ~ gene62"
        },
        {
          "x": 47,
          "y": 62,
          "value": 0,
          "name": "cell48 ~ gene63"
        },
        {
          "x": 47,
          "y": 63,
          "value": 0,
          "name": "cell48 ~ gene64"
        },
        {
          "x": 47,
          "y": 64,
          "value": 0,
          "name": "cell48 ~ gene65"
        },
        {
          "x": 47,
          "y": 65,
          "value": 13,
          "name": "cell48 ~ gene66"
        },
        {
          "x": 47,
          "y": 66,
          "value": 0,
          "name": "cell48 ~ gene67"
        },
        {
          "x": 47,
          "y": 67,
          "value": 0,
          "name": "cell48 ~ gene68"
        },
        {
          "x": 47,
          "y": 68,
          "value": 0,
          "name": "cell48 ~ gene69"
        },
        {
          "x": 47,
          "y": 69,
          "value": 0,
          "name": "cell48 ~ gene70"
        },
        {
          "x": 47,
          "y": 70,
          "value": 0,
          "name": "cell48 ~ gene71"
        },
        {
          "x": 47,
          "y": 71,
          "value": 0,
          "name": "cell48 ~ gene72"
        },
        {
          "x": 47,
          "y": 72,
          "value": 1,
          "name": "cell48 ~ gene73"
        },
        {
          "x": 47,
          "y": 73,
          "value": 0,
          "name": "cell48 ~ gene74"
        },
        {
          "x": 47,
          "y": 74,
          "value": 0,
          "name": "cell48 ~ gene75"
        },
        {
          "x": 47,
          "y": 75,
          "value": 0,
          "name": "cell48 ~ gene76"
        },
        {
          "x": 47,
          "y": 76,
          "value": 0,
          "name": "cell48 ~ gene77"
        },
        {
          "x": 47,
          "y": 77,
          "value": 0,
          "name": "cell48 ~ gene78"
        },
        {
          "x": 47,
          "y": 78,
          "value": 8,
          "name": "cell48 ~ gene79"
        },
        {
          "x": 47,
          "y": 79,
          "value": 6,
          "name": "cell48 ~ gene80"
        },
        {
          "x": 47,
          "y": 80,
          "value": 0,
          "name": "cell48 ~ gene81"
        },
        {
          "x": 47,
          "y": 81,
          "value": 0,
          "name": "cell48 ~ gene82"
        },
        {
          "x": 47,
          "y": 82,
          "value": 0,
          "name": "cell48 ~ gene83"
        },
        {
          "x": 47,
          "y": 83,
          "value": 0,
          "name": "cell48 ~ gene84"
        },
        {
          "x": 47,
          "y": 84,
          "value": 0,
          "name": "cell48 ~ gene85"
        },
        {
          "x": 47,
          "y": 85,
          "value": 8,
          "name": "cell48 ~ gene86"
        },
        {
          "x": 47,
          "y": 86,
          "value": 2,
          "name": "cell48 ~ gene87"
        },
        {
          "x": 47,
          "y": 87,
          "value": 3,
          "name": "cell48 ~ gene88"
        },
        {
          "x": 47,
          "y": 88,
          "value": 2,
          "name": "cell48 ~ gene89"
        },
        {
          "x": 47,
          "y": 89,
          "value": 21,
          "name": "cell48 ~ gene90"
        },
        {
          "x": 47,
          "y": 90,
          "value": 0,
          "name": "cell48 ~ gene91"
        },
        {
          "x": 47,
          "y": 91,
          "value": 0,
          "name": "cell48 ~ gene92"
        },
        {
          "x": 47,
          "y": 92,
          "value": 0,
          "name": "cell48 ~ gene93"
        },
        {
          "x": 47,
          "y": 93,
          "value": 2,
          "name": "cell48 ~ gene94"
        },
        {
          "x": 47,
          "y": 94,
          "value": 0,
          "name": "cell48 ~ gene95"
        },
        {
          "x": 47,
          "y": 95,
          "value": 1,
          "name": "cell48 ~ gene96"
        },
        {
          "x": 47,
          "y": 96,
          "value": 0,
          "name": "cell48 ~ gene97"
        },
        {
          "x": 47,
          "y": 97,
          "value": 18,
          "name": "cell48 ~ gene98"
        },
        {
          "x": 47,
          "y": 98,
          "value": 7,
          "name": "cell48 ~ gene99"
        },
        {
          "x": 47,
          "y": 99,
          "value": 5,
          "name": "cell48 ~ gene100"
        },
        {
          "x": 48,
          "y": 0,
          "value": 0,
          "name": "cell49 ~ gene1"
        },
        {
          "x": 48,
          "y": 1,
          "value": 5,
          "name": "cell49 ~ gene2"
        },
        {
          "x": 48,
          "y": 2,
          "value": 19,
          "name": "cell49 ~ gene3"
        },
        {
          "x": 48,
          "y": 3,
          "value": 0,
          "name": "cell49 ~ gene4"
        },
        {
          "x": 48,
          "y": 4,
          "value": 0,
          "name": "cell49 ~ gene5"
        },
        {
          "x": 48,
          "y": 5,
          "value": 0,
          "name": "cell49 ~ gene6"
        },
        {
          "x": 48,
          "y": 6,
          "value": 0,
          "name": "cell49 ~ gene7"
        },
        {
          "x": 48,
          "y": 7,
          "value": 0,
          "name": "cell49 ~ gene8"
        },
        {
          "x": 48,
          "y": 8,
          "value": 0,
          "name": "cell49 ~ gene9"
        },
        {
          "x": 48,
          "y": 9,
          "value": 0,
          "name": "cell49 ~ gene10"
        },
        {
          "x": 48,
          "y": 10,
          "value": 0,
          "name": "cell49 ~ gene11"
        },
        {
          "x": 48,
          "y": 11,
          "value": 0,
          "name": "cell49 ~ gene12"
        },
        {
          "x": 48,
          "y": 12,
          "value": 11,
          "name": "cell49 ~ gene13"
        },
        {
          "x": 48,
          "y": 13,
          "value": 0,
          "name": "cell49 ~ gene14"
        },
        {
          "x": 48,
          "y": 14,
          "value": 4,
          "name": "cell49 ~ gene15"
        },
        {
          "x": 48,
          "y": 15,
          "value": 2,
          "name": "cell49 ~ gene16"
        },
        {
          "x": 48,
          "y": 16,
          "value": 1,
          "name": "cell49 ~ gene17"
        },
        {
          "x": 48,
          "y": 17,
          "value": 14,
          "name": "cell49 ~ gene18"
        },
        {
          "x": 48,
          "y": 18,
          "value": 0,
          "name": "cell49 ~ gene19"
        },
        {
          "x": 48,
          "y": 19,
          "value": 0,
          "name": "cell49 ~ gene20"
        },
        {
          "x": 48,
          "y": 20,
          "value": 10,
          "name": "cell49 ~ gene21"
        },
        {
          "x": 48,
          "y": 21,
          "value": 32,
          "name": "cell49 ~ gene22"
        },
        {
          "x": 48,
          "y": 22,
          "value": 6,
          "name": "cell49 ~ gene23"
        },
        {
          "x": 48,
          "y": 23,
          "value": 0,
          "name": "cell49 ~ gene24"
        },
        {
          "x": 48,
          "y": 24,
          "value": 30,
          "name": "cell49 ~ gene25"
        },
        {
          "x": 48,
          "y": 25,
          "value": 17,
          "name": "cell49 ~ gene26"
        },
        {
          "x": 48,
          "y": 26,
          "value": 33,
          "name": "cell49 ~ gene27"
        },
        {
          "x": 48,
          "y": 27,
          "value": 10,
          "name": "cell49 ~ gene28"
        },
        {
          "x": 48,
          "y": 28,
          "value": 5,
          "name": "cell49 ~ gene29"
        },
        {
          "x": 48,
          "y": 29,
          "value": 23,
          "name": "cell49 ~ gene30"
        },
        {
          "x": 48,
          "y": 30,
          "value": 38,
          "name": "cell49 ~ gene31"
        },
        {
          "x": 48,
          "y": 31,
          "value": 18,
          "name": "cell49 ~ gene32"
        },
        {
          "x": 48,
          "y": 32,
          "value": 2,
          "name": "cell49 ~ gene33"
        },
        {
          "x": 48,
          "y": 33,
          "value": 0,
          "name": "cell49 ~ gene34"
        },
        {
          "x": 48,
          "y": 34,
          "value": 28,
          "name": "cell49 ~ gene35"
        },
        {
          "x": 48,
          "y": 35,
          "value": 0,
          "name": "cell49 ~ gene36"
        },
        {
          "x": 48,
          "y": 36,
          "value": 17,
          "name": "cell49 ~ gene37"
        },
        {
          "x": 48,
          "y": 37,
          "value": 0,
          "name": "cell49 ~ gene38"
        },
        {
          "x": 48,
          "y": 38,
          "value": 0,
          "name": "cell49 ~ gene39"
        },
        {
          "x": 48,
          "y": 39,
          "value": 8,
          "name": "cell49 ~ gene40"
        },
        {
          "x": 48,
          "y": 40,
          "value": 17,
          "name": "cell49 ~ gene41"
        },
        {
          "x": 48,
          "y": 41,
          "value": 0,
          "name": "cell49 ~ gene42"
        },
        {
          "x": 48,
          "y": 42,
          "value": 0,
          "name": "cell49 ~ gene43"
        },
        {
          "x": 48,
          "y": 43,
          "value": 2,
          "name": "cell49 ~ gene44"
        },
        {
          "x": 48,
          "y": 44,
          "value": 7,
          "name": "cell49 ~ gene45"
        },
        {
          "x": 48,
          "y": 45,
          "value": 0,
          "name": "cell49 ~ gene46"
        },
        {
          "x": 48,
          "y": 46,
          "value": 0,
          "name": "cell49 ~ gene47"
        },
        {
          "x": 48,
          "y": 47,
          "value": 0,
          "name": "cell49 ~ gene48"
        },
        {
          "x": 48,
          "y": 48,
          "value": 0,
          "name": "cell49 ~ gene49"
        },
        {
          "x": 48,
          "y": 49,
          "value": 14,
          "name": "cell49 ~ gene50"
        },
        {
          "x": 48,
          "y": 50,
          "value": 0,
          "name": "cell49 ~ gene51"
        },
        {
          "x": 48,
          "y": 51,
          "value": 2,
          "name": "cell49 ~ gene52"
        },
        {
          "x": 48,
          "y": 52,
          "value": 18,
          "name": "cell49 ~ gene53"
        },
        {
          "x": 48,
          "y": 53,
          "value": 0,
          "name": "cell49 ~ gene54"
        },
        {
          "x": 48,
          "y": 54,
          "value": 0,
          "name": "cell49 ~ gene55"
        },
        {
          "x": 48,
          "y": 55,
          "value": 0,
          "name": "cell49 ~ gene56"
        },
        {
          "x": 48,
          "y": 56,
          "value": 7,
          "name": "cell49 ~ gene57"
        },
        {
          "x": 48,
          "y": 57,
          "value": 0,
          "name": "cell49 ~ gene58"
        },
        {
          "x": 48,
          "y": 58,
          "value": 4,
          "name": "cell49 ~ gene59"
        },
        {
          "x": 48,
          "y": 59,
          "value": 0,
          "name": "cell49 ~ gene60"
        },
        {
          "x": 48,
          "y": 60,
          "value": 4,
          "name": "cell49 ~ gene61"
        },
        {
          "x": 48,
          "y": 61,
          "value": 9,
          "name": "cell49 ~ gene62"
        },
        {
          "x": 48,
          "y": 62,
          "value": 0,
          "name": "cell49 ~ gene63"
        },
        {
          "x": 48,
          "y": 63,
          "value": 1,
          "name": "cell49 ~ gene64"
        },
        {
          "x": 48,
          "y": 64,
          "value": 10,
          "name": "cell49 ~ gene65"
        },
        {
          "x": 48,
          "y": 65,
          "value": 5,
          "name": "cell49 ~ gene66"
        },
        {
          "x": 48,
          "y": 66,
          "value": 0,
          "name": "cell49 ~ gene67"
        },
        {
          "x": 48,
          "y": 67,
          "value": 8,
          "name": "cell49 ~ gene68"
        },
        {
          "x": 48,
          "y": 68,
          "value": 12,
          "name": "cell49 ~ gene69"
        },
        {
          "x": 48,
          "y": 69,
          "value": 0,
          "name": "cell49 ~ gene70"
        },
        {
          "x": 48,
          "y": 70,
          "value": 0,
          "name": "cell49 ~ gene71"
        },
        {
          "x": 48,
          "y": 71,
          "value": 0,
          "name": "cell49 ~ gene72"
        },
        {
          "x": 48,
          "y": 72,
          "value": 0,
          "name": "cell49 ~ gene73"
        },
        {
          "x": 48,
          "y": 73,
          "value": 2,
          "name": "cell49 ~ gene74"
        },
        {
          "x": 48,
          "y": 74,
          "value": 0,
          "name": "cell49 ~ gene75"
        },
        {
          "x": 48,
          "y": 75,
          "value": 0,
          "name": "cell49 ~ gene76"
        },
        {
          "x": 48,
          "y": 76,
          "value": 5,
          "name": "cell49 ~ gene77"
        },
        {
          "x": 48,
          "y": 77,
          "value": 0,
          "name": "cell49 ~ gene78"
        },
        {
          "x": 48,
          "y": 78,
          "value": 10,
          "name": "cell49 ~ gene79"
        },
        {
          "x": 48,
          "y": 79,
          "value": 6,
          "name": "cell49 ~ gene80"
        },
        {
          "x": 48,
          "y": 80,
          "value": 6,
          "name": "cell49 ~ gene81"
        },
        {
          "x": 48,
          "y": 81,
          "value": 0,
          "name": "cell49 ~ gene82"
        },
        {
          "x": 48,
          "y": 82,
          "value": 0,
          "name": "cell49 ~ gene83"
        },
        {
          "x": 48,
          "y": 83,
          "value": 0,
          "name": "cell49 ~ gene84"
        },
        {
          "x": 48,
          "y": 84,
          "value": 10,
          "name": "cell49 ~ gene85"
        },
        {
          "x": 48,
          "y": 85,
          "value": 0,
          "name": "cell49 ~ gene86"
        },
        {
          "x": 48,
          "y": 86,
          "value": 6,
          "name": "cell49 ~ gene87"
        },
        {
          "x": 48,
          "y": 87,
          "value": 5,
          "name": "cell49 ~ gene88"
        },
        {
          "x": 48,
          "y": 88,
          "value": 8,
          "name": "cell49 ~ gene89"
        },
        {
          "x": 48,
          "y": 89,
          "value": 5,
          "name": "cell49 ~ gene90"
        },
        {
          "x": 48,
          "y": 90,
          "value": 1,
          "name": "cell49 ~ gene91"
        },
        {
          "x": 48,
          "y": 91,
          "value": 1,
          "name": "cell49 ~ gene92"
        },
        {
          "x": 48,
          "y": 92,
          "value": 13,
          "name": "cell49 ~ gene93"
        },
        {
          "x": 48,
          "y": 93,
          "value": 8,
          "name": "cell49 ~ gene94"
        },
        {
          "x": 48,
          "y": 94,
          "value": 0,
          "name": "cell49 ~ gene95"
        },
        {
          "x": 48,
          "y": 95,
          "value": 0,
          "name": "cell49 ~ gene96"
        },
        {
          "x": 48,
          "y": 96,
          "value": 3,
          "name": "cell49 ~ gene97"
        },
        {
          "x": 48,
          "y": 97,
          "value": 11,
          "name": "cell49 ~ gene98"
        },
        {
          "x": 48,
          "y": 98,
          "value": 0,
          "name": "cell49 ~ gene99"
        },
        {
          "x": 48,
          "y": 99,
          "value": 9,
          "name": "cell49 ~ gene100"
        },
        {
          "x": 49,
          "y": 0,
          "value": 8,
          "name": "cell50 ~ gene1"
        },
        {
          "x": 49,
          "y": 1,
          "value": 1,
          "name": "cell50 ~ gene2"
        },
        {
          "x": 49,
          "y": 2,
          "value": 0,
          "name": "cell50 ~ gene3"
        },
        {
          "x": 49,
          "y": 3,
          "value": 0,
          "name": "cell50 ~ gene4"
        },
        {
          "x": 49,
          "y": 4,
          "value": 14,
          "name": "cell50 ~ gene5"
        },
        {
          "x": 49,
          "y": 5,
          "value": 7,
          "name": "cell50 ~ gene6"
        },
        {
          "x": 49,
          "y": 6,
          "value": 5,
          "name": "cell50 ~ gene7"
        },
        {
          "x": 49,
          "y": 7,
          "value": 0,
          "name": "cell50 ~ gene8"
        },
        {
          "x": 49,
          "y": 8,
          "value": 0,
          "name": "cell50 ~ gene9"
        },
        {
          "x": 49,
          "y": 9,
          "value": 0,
          "name": "cell50 ~ gene10"
        },
        {
          "x": 49,
          "y": 10,
          "value": 1,
          "name": "cell50 ~ gene11"
        },
        {
          "x": 49,
          "y": 11,
          "value": 5,
          "name": "cell50 ~ gene12"
        },
        {
          "x": 49,
          "y": 12,
          "value": 0,
          "name": "cell50 ~ gene13"
        },
        {
          "x": 49,
          "y": 13,
          "value": 0,
          "name": "cell50 ~ gene14"
        },
        {
          "x": 49,
          "y": 14,
          "value": 24,
          "name": "cell50 ~ gene15"
        },
        {
          "x": 49,
          "y": 15,
          "value": 0,
          "name": "cell50 ~ gene16"
        },
        {
          "x": 49,
          "y": 16,
          "value": 10,
          "name": "cell50 ~ gene17"
        },
        {
          "x": 49,
          "y": 17,
          "value": 0,
          "name": "cell50 ~ gene18"
        },
        {
          "x": 49,
          "y": 18,
          "value": 0,
          "name": "cell50 ~ gene19"
        },
        {
          "x": 49,
          "y": 19,
          "value": 0,
          "name": "cell50 ~ gene20"
        },
        {
          "x": 49,
          "y": 20,
          "value": 10,
          "name": "cell50 ~ gene21"
        },
        {
          "x": 49,
          "y": 21,
          "value": 15,
          "name": "cell50 ~ gene22"
        },
        {
          "x": 49,
          "y": 22,
          "value": 10,
          "name": "cell50 ~ gene23"
        },
        {
          "x": 49,
          "y": 23,
          "value": 0,
          "name": "cell50 ~ gene24"
        },
        {
          "x": 49,
          "y": 24,
          "value": 7,
          "name": "cell50 ~ gene25"
        },
        {
          "x": 49,
          "y": 25,
          "value": 18,
          "name": "cell50 ~ gene26"
        },
        {
          "x": 49,
          "y": 26,
          "value": 31,
          "name": "cell50 ~ gene27"
        },
        {
          "x": 49,
          "y": 27,
          "value": 20,
          "name": "cell50 ~ gene28"
        },
        {
          "x": 49,
          "y": 28,
          "value": 2,
          "name": "cell50 ~ gene29"
        },
        {
          "x": 49,
          "y": 29,
          "value": 25,
          "name": "cell50 ~ gene30"
        },
        {
          "x": 49,
          "y": 30,
          "value": 42,
          "name": "cell50 ~ gene31"
        },
        {
          "x": 49,
          "y": 31,
          "value": 13,
          "name": "cell50 ~ gene32"
        },
        {
          "x": 49,
          "y": 32,
          "value": 0,
          "name": "cell50 ~ gene33"
        },
        {
          "x": 49,
          "y": 33,
          "value": 4,
          "name": "cell50 ~ gene34"
        },
        {
          "x": 49,
          "y": 34,
          "value": 0,
          "name": "cell50 ~ gene35"
        },
        {
          "x": 49,
          "y": 35,
          "value": 0,
          "name": "cell50 ~ gene36"
        },
        {
          "x": 49,
          "y": 36,
          "value": 24,
          "name": "cell50 ~ gene37"
        },
        {
          "x": 49,
          "y": 37,
          "value": 4,
          "name": "cell50 ~ gene38"
        },
        {
          "x": 49,
          "y": 38,
          "value": 0,
          "name": "cell50 ~ gene39"
        },
        {
          "x": 49,
          "y": 39,
          "value": 18,
          "name": "cell50 ~ gene40"
        },
        {
          "x": 49,
          "y": 40,
          "value": 17,
          "name": "cell50 ~ gene41"
        },
        {
          "x": 49,
          "y": 41,
          "value": 0,
          "name": "cell50 ~ gene42"
        },
        {
          "x": 49,
          "y": 42,
          "value": 20,
          "name": "cell50 ~ gene43"
        },
        {
          "x": 49,
          "y": 43,
          "value": 4,
          "name": "cell50 ~ gene44"
        },
        {
          "x": 49,
          "y": 44,
          "value": 0,
          "name": "cell50 ~ gene45"
        },
        {
          "x": 49,
          "y": 45,
          "value": 18,
          "name": "cell50 ~ gene46"
        },
        {
          "x": 49,
          "y": 46,
          "value": 0,
          "name": "cell50 ~ gene47"
        },
        {
          "x": 49,
          "y": 47,
          "value": 0,
          "name": "cell50 ~ gene48"
        },
        {
          "x": 49,
          "y": 48,
          "value": 0,
          "name": "cell50 ~ gene49"
        },
        {
          "x": 49,
          "y": 49,
          "value": 0,
          "name": "cell50 ~ gene50"
        },
        {
          "x": 49,
          "y": 50,
          "value": 0,
          "name": "cell50 ~ gene51"
        },
        {
          "x": 49,
          "y": 51,
          "value": 0,
          "name": "cell50 ~ gene52"
        },
        {
          "x": 49,
          "y": 52,
          "value": 0,
          "name": "cell50 ~ gene53"
        },
        {
          "x": 49,
          "y": 53,
          "value": 8,
          "name": "cell50 ~ gene54"
        },
        {
          "x": 49,
          "y": 54,
          "value": 0,
          "name": "cell50 ~ gene55"
        },
        {
          "x": 49,
          "y": 55,
          "value": 0,
          "name": "cell50 ~ gene56"
        },
        {
          "x": 49,
          "y": 56,
          "value": 0,
          "name": "cell50 ~ gene57"
        },
        {
          "x": 49,
          "y": 57,
          "value": 0,
          "name": "cell50 ~ gene58"
        },
        {
          "x": 49,
          "y": 58,
          "value": 7,
          "name": "cell50 ~ gene59"
        },
        {
          "x": 49,
          "y": 59,
          "value": 12,
          "name": "cell50 ~ gene60"
        },
        {
          "x": 49,
          "y": 60,
          "value": 2,
          "name": "cell50 ~ gene61"
        },
        {
          "x": 49,
          "y": 61,
          "value": 0,
          "name": "cell50 ~ gene62"
        },
        {
          "x": 49,
          "y": 62,
          "value": 0,
          "name": "cell50 ~ gene63"
        },
        {
          "x": 49,
          "y": 63,
          "value": 13,
          "name": "cell50 ~ gene64"
        },
        {
          "x": 49,
          "y": 64,
          "value": 0,
          "name": "cell50 ~ gene65"
        },
        {
          "x": 49,
          "y": 65,
          "value": 0,
          "name": "cell50 ~ gene66"
        },
        {
          "x": 49,
          "y": 66,
          "value": 0,
          "name": "cell50 ~ gene67"
        },
        {
          "x": 49,
          "y": 67,
          "value": 0,
          "name": "cell50 ~ gene68"
        },
        {
          "x": 49,
          "y": 68,
          "value": 0,
          "name": "cell50 ~ gene69"
        },
        {
          "x": 49,
          "y": 69,
          "value": 14,
          "name": "cell50 ~ gene70"
        },
        {
          "x": 49,
          "y": 70,
          "value": 0,
          "name": "cell50 ~ gene71"
        },
        {
          "x": 49,
          "y": 71,
          "value": 5,
          "name": "cell50 ~ gene72"
        },
        {
          "x": 49,
          "y": 72,
          "value": 0,
          "name": "cell50 ~ gene73"
        },
        {
          "x": 49,
          "y": 73,
          "value": 0,
          "name": "cell50 ~ gene74"
        },
        {
          "x": 49,
          "y": 74,
          "value": 0,
          "name": "cell50 ~ gene75"
        },
        {
          "x": 49,
          "y": 75,
          "value": 0,
          "name": "cell50 ~ gene76"
        },
        {
          "x": 49,
          "y": 76,
          "value": 0,
          "name": "cell50 ~ gene77"
        },
        {
          "x": 49,
          "y": 77,
          "value": 11,
          "name": "cell50 ~ gene78"
        },
        {
          "x": 49,
          "y": 78,
          "value": 2,
          "name": "cell50 ~ gene79"
        },
        {
          "x": 49,
          "y": 79,
          "value": 0,
          "name": "cell50 ~ gene80"
        },
        {
          "x": 49,
          "y": 80,
          "value": 16,
          "name": "cell50 ~ gene81"
        },
        {
          "x": 49,
          "y": 81,
          "value": 0,
          "name": "cell50 ~ gene82"
        },
        {
          "x": 49,
          "y": 82,
          "value": 0,
          "name": "cell50 ~ gene83"
        },
        {
          "x": 49,
          "y": 83,
          "value": 2,
          "name": "cell50 ~ gene84"
        },
        {
          "x": 49,
          "y": 84,
          "value": 9,
          "name": "cell50 ~ gene85"
        },
        {
          "x": 49,
          "y": 85,
          "value": 2,
          "name": "cell50 ~ gene86"
        },
        {
          "x": 49,
          "y": 86,
          "value": 0,
          "name": "cell50 ~ gene87"
        },
        {
          "x": 49,
          "y": 87,
          "value": 0,
          "name": "cell50 ~ gene88"
        },
        {
          "x": 49,
          "y": 88,
          "value": 0,
          "name": "cell50 ~ gene89"
        },
        {
          "x": 49,
          "y": 89,
          "value": 0,
          "name": "cell50 ~ gene90"
        },
        {
          "x": 49,
          "y": 90,
          "value": 5,
          "name": "cell50 ~ gene91"
        },
        {
          "x": 49,
          "y": 91,
          "value": 17,
          "name": "cell50 ~ gene92"
        },
        {
          "x": 49,
          "y": 92,
          "value": 2,
          "name": "cell50 ~ gene93"
        },
        {
          "x": 49,
          "y": 93,
          "value": 0,
          "name": "cell50 ~ gene94"
        },
        {
          "x": 49,
          "y": 94,
          "value": 0,
          "name": "cell50 ~ gene95"
        },
        {
          "x": 49,
          "y": 95,
          "value": 19,
          "name": "cell50 ~ gene96"
        },
        {
          "x": 49,
          "y": 96,
          "value": 22,
          "name": "cell50 ~ gene97"
        },
        {
          "x": 49,
          "y": 97,
          "value": 12,
          "name": "cell50 ~ gene98"
        },
        {
          "x": 49,
          "y": 98,
          "value": 6,
          "name": "cell50 ~ gene99"
        },
        {
          "x": 49,
          "y": 99,
          "value": 0,
          "name": "cell50 ~ gene100"
        },
        {
          "x": 50,
          "y": 0,
          "value": 4,
          "name": "cell51 ~ gene1"
        },
        {
          "x": 50,
          "y": 1,
          "value": 5,
          "name": "cell51 ~ gene2"
        },
        {
          "x": 50,
          "y": 2,
          "value": 0,
          "name": "cell51 ~ gene3"
        },
        {
          "x": 50,
          "y": 3,
          "value": 0,
          "name": "cell51 ~ gene4"
        },
        {
          "x": 50,
          "y": 4,
          "value": 10,
          "name": "cell51 ~ gene5"
        },
        {
          "x": 50,
          "y": 5,
          "value": 0,
          "name": "cell51 ~ gene6"
        },
        {
          "x": 50,
          "y": 6,
          "value": 1,
          "name": "cell51 ~ gene7"
        },
        {
          "x": 50,
          "y": 7,
          "value": 6,
          "name": "cell51 ~ gene8"
        },
        {
          "x": 50,
          "y": 8,
          "value": 8,
          "name": "cell51 ~ gene9"
        },
        {
          "x": 50,
          "y": 9,
          "value": 0,
          "name": "cell51 ~ gene10"
        },
        {
          "x": 50,
          "y": 10,
          "value": 0,
          "name": "cell51 ~ gene11"
        },
        {
          "x": 50,
          "y": 11,
          "value": 0,
          "name": "cell51 ~ gene12"
        },
        {
          "x": 50,
          "y": 12,
          "value": 7,
          "name": "cell51 ~ gene13"
        },
        {
          "x": 50,
          "y": 13,
          "value": 5,
          "name": "cell51 ~ gene14"
        },
        {
          "x": 50,
          "y": 14,
          "value": 8,
          "name": "cell51 ~ gene15"
        },
        {
          "x": 50,
          "y": 15,
          "value": 0,
          "name": "cell51 ~ gene16"
        },
        {
          "x": 50,
          "y": 16,
          "value": 14,
          "name": "cell51 ~ gene17"
        },
        {
          "x": 50,
          "y": 17,
          "value": 9,
          "name": "cell51 ~ gene18"
        },
        {
          "x": 50,
          "y": 18,
          "value": 2,
          "name": "cell51 ~ gene19"
        },
        {
          "x": 50,
          "y": 19,
          "value": 0,
          "name": "cell51 ~ gene20"
        },
        {
          "x": 50,
          "y": 20,
          "value": 12,
          "name": "cell51 ~ gene21"
        },
        {
          "x": 50,
          "y": 21,
          "value": 13,
          "name": "cell51 ~ gene22"
        },
        {
          "x": 50,
          "y": 22,
          "value": 21,
          "name": "cell51 ~ gene23"
        },
        {
          "x": 50,
          "y": 23,
          "value": 0,
          "name": "cell51 ~ gene24"
        },
        {
          "x": 50,
          "y": 24,
          "value": 18,
          "name": "cell51 ~ gene25"
        },
        {
          "x": 50,
          "y": 25,
          "value": 26,
          "name": "cell51 ~ gene26"
        },
        {
          "x": 50,
          "y": 26,
          "value": 8,
          "name": "cell51 ~ gene27"
        },
        {
          "x": 50,
          "y": 27,
          "value": 11,
          "name": "cell51 ~ gene28"
        },
        {
          "x": 50,
          "y": 28,
          "value": 0,
          "name": "cell51 ~ gene29"
        },
        {
          "x": 50,
          "y": 29,
          "value": 4,
          "name": "cell51 ~ gene30"
        },
        {
          "x": 50,
          "y": 30,
          "value": 26,
          "name": "cell51 ~ gene31"
        },
        {
          "x": 50,
          "y": 31,
          "value": 5,
          "name": "cell51 ~ gene32"
        },
        {
          "x": 50,
          "y": 32,
          "value": 19,
          "name": "cell51 ~ gene33"
        },
        {
          "x": 50,
          "y": 33,
          "value": 0,
          "name": "cell51 ~ gene34"
        },
        {
          "x": 50,
          "y": 34,
          "value": 2,
          "name": "cell51 ~ gene35"
        },
        {
          "x": 50,
          "y": 35,
          "value": 0,
          "name": "cell51 ~ gene36"
        },
        {
          "x": 50,
          "y": 36,
          "value": 0,
          "name": "cell51 ~ gene37"
        },
        {
          "x": 50,
          "y": 37,
          "value": 15,
          "name": "cell51 ~ gene38"
        },
        {
          "x": 50,
          "y": 38,
          "value": 8,
          "name": "cell51 ~ gene39"
        },
        {
          "x": 50,
          "y": 39,
          "value": 9,
          "name": "cell51 ~ gene40"
        },
        {
          "x": 50,
          "y": 40,
          "value": 0,
          "name": "cell51 ~ gene41"
        },
        {
          "x": 50,
          "y": 41,
          "value": 4,
          "name": "cell51 ~ gene42"
        },
        {
          "x": 50,
          "y": 42,
          "value": 10,
          "name": "cell51 ~ gene43"
        },
        {
          "x": 50,
          "y": 43,
          "value": 11,
          "name": "cell51 ~ gene44"
        },
        {
          "x": 50,
          "y": 44,
          "value": 14,
          "name": "cell51 ~ gene45"
        },
        {
          "x": 50,
          "y": 45,
          "value": 0,
          "name": "cell51 ~ gene46"
        },
        {
          "x": 50,
          "y": 46,
          "value": 0,
          "name": "cell51 ~ gene47"
        },
        {
          "x": 50,
          "y": 47,
          "value": 0,
          "name": "cell51 ~ gene48"
        },
        {
          "x": 50,
          "y": 48,
          "value": 0,
          "name": "cell51 ~ gene49"
        },
        {
          "x": 50,
          "y": 49,
          "value": 0,
          "name": "cell51 ~ gene50"
        },
        {
          "x": 50,
          "y": 50,
          "value": 5,
          "name": "cell51 ~ gene51"
        },
        {
          "x": 50,
          "y": 51,
          "value": 4,
          "name": "cell51 ~ gene52"
        },
        {
          "x": 50,
          "y": 52,
          "value": 0,
          "name": "cell51 ~ gene53"
        },
        {
          "x": 50,
          "y": 53,
          "value": 0,
          "name": "cell51 ~ gene54"
        },
        {
          "x": 50,
          "y": 54,
          "value": 5,
          "name": "cell51 ~ gene55"
        },
        {
          "x": 50,
          "y": 55,
          "value": 17,
          "name": "cell51 ~ gene56"
        },
        {
          "x": 50,
          "y": 56,
          "value": 14,
          "name": "cell51 ~ gene57"
        },
        {
          "x": 50,
          "y": 57,
          "value": 0,
          "name": "cell51 ~ gene58"
        },
        {
          "x": 50,
          "y": 58,
          "value": 17,
          "name": "cell51 ~ gene59"
        },
        {
          "x": 50,
          "y": 59,
          "value": 0,
          "name": "cell51 ~ gene60"
        },
        {
          "x": 50,
          "y": 60,
          "value": 15,
          "name": "cell51 ~ gene61"
        },
        {
          "x": 50,
          "y": 61,
          "value": 9,
          "name": "cell51 ~ gene62"
        },
        {
          "x": 50,
          "y": 62,
          "value": 0,
          "name": "cell51 ~ gene63"
        },
        {
          "x": 50,
          "y": 63,
          "value": 21,
          "name": "cell51 ~ gene64"
        },
        {
          "x": 50,
          "y": 64,
          "value": 2,
          "name": "cell51 ~ gene65"
        },
        {
          "x": 50,
          "y": 65,
          "value": 2,
          "name": "cell51 ~ gene66"
        },
        {
          "x": 50,
          "y": 66,
          "value": 8,
          "name": "cell51 ~ gene67"
        },
        {
          "x": 50,
          "y": 67,
          "value": 0,
          "name": "cell51 ~ gene68"
        },
        {
          "x": 50,
          "y": 68,
          "value": 0,
          "name": "cell51 ~ gene69"
        },
        {
          "x": 50,
          "y": 69,
          "value": 0,
          "name": "cell51 ~ gene70"
        },
        {
          "x": 50,
          "y": 70,
          "value": 5,
          "name": "cell51 ~ gene71"
        },
        {
          "x": 50,
          "y": 71,
          "value": 0,
          "name": "cell51 ~ gene72"
        },
        {
          "x": 50,
          "y": 72,
          "value": 0,
          "name": "cell51 ~ gene73"
        },
        {
          "x": 50,
          "y": 73,
          "value": 0,
          "name": "cell51 ~ gene74"
        },
        {
          "x": 50,
          "y": 74,
          "value": 1,
          "name": "cell51 ~ gene75"
        },
        {
          "x": 50,
          "y": 75,
          "value": 12,
          "name": "cell51 ~ gene76"
        },
        {
          "x": 50,
          "y": 76,
          "value": 5,
          "name": "cell51 ~ gene77"
        },
        {
          "x": 50,
          "y": 77,
          "value": 8,
          "name": "cell51 ~ gene78"
        },
        {
          "x": 50,
          "y": 78,
          "value": 4,
          "name": "cell51 ~ gene79"
        },
        {
          "x": 50,
          "y": 79,
          "value": 0,
          "name": "cell51 ~ gene80"
        },
        {
          "x": 50,
          "y": 80,
          "value": 0,
          "name": "cell51 ~ gene81"
        },
        {
          "x": 50,
          "y": 81,
          "value": 0,
          "name": "cell51 ~ gene82"
        },
        {
          "x": 50,
          "y": 82,
          "value": 8,
          "name": "cell51 ~ gene83"
        },
        {
          "x": 50,
          "y": 83,
          "value": 0,
          "name": "cell51 ~ gene84"
        },
        {
          "x": 50,
          "y": 84,
          "value": 11,
          "name": "cell51 ~ gene85"
        },
        {
          "x": 50,
          "y": 85,
          "value": 4,
          "name": "cell51 ~ gene86"
        },
        {
          "x": 50,
          "y": 86,
          "value": 19,
          "name": "cell51 ~ gene87"
        },
        {
          "x": 50,
          "y": 87,
          "value": 6,
          "name": "cell51 ~ gene88"
        },
        {
          "x": 50,
          "y": 88,
          "value": 0,
          "name": "cell51 ~ gene89"
        },
        {
          "x": 50,
          "y": 89,
          "value": 0,
          "name": "cell51 ~ gene90"
        },
        {
          "x": 50,
          "y": 90,
          "value": 1,
          "name": "cell51 ~ gene91"
        },
        {
          "x": 50,
          "y": 91,
          "value": 11,
          "name": "cell51 ~ gene92"
        },
        {
          "x": 50,
          "y": 92,
          "value": 0,
          "name": "cell51 ~ gene93"
        },
        {
          "x": 50,
          "y": 93,
          "value": 0,
          "name": "cell51 ~ gene94"
        },
        {
          "x": 50,
          "y": 94,
          "value": 0,
          "name": "cell51 ~ gene95"
        },
        {
          "x": 50,
          "y": 95,
          "value": 0,
          "name": "cell51 ~ gene96"
        },
        {
          "x": 50,
          "y": 96,
          "value": 1,
          "name": "cell51 ~ gene97"
        },
        {
          "x": 50,
          "y": 97,
          "value": 8,
          "name": "cell51 ~ gene98"
        },
        {
          "x": 50,
          "y": 98,
          "value": 0,
          "name": "cell51 ~ gene99"
        },
        {
          "x": 50,
          "y": 99,
          "value": 12,
          "name": "cell51 ~ gene100"
        },
        {
          "x": 51,
          "y": 0,
          "value": 10,
          "name": "cell52 ~ gene1"
        },
        {
          "x": 51,
          "y": 1,
          "value": 9,
          "name": "cell52 ~ gene2"
        },
        {
          "x": 51,
          "y": 2,
          "value": 6,
          "name": "cell52 ~ gene3"
        },
        {
          "x": 51,
          "y": 3,
          "value": 0,
          "name": "cell52 ~ gene4"
        },
        {
          "x": 51,
          "y": 4,
          "value": 0,
          "name": "cell52 ~ gene5"
        },
        {
          "x": 51,
          "y": 5,
          "value": 12,
          "name": "cell52 ~ gene6"
        },
        {
          "x": 51,
          "y": 6,
          "value": 12,
          "name": "cell52 ~ gene7"
        },
        {
          "x": 51,
          "y": 7,
          "value": 16,
          "name": "cell52 ~ gene8"
        },
        {
          "x": 51,
          "y": 8,
          "value": 0,
          "name": "cell52 ~ gene9"
        },
        {
          "x": 51,
          "y": 9,
          "value": 7,
          "name": "cell52 ~ gene10"
        },
        {
          "x": 51,
          "y": 10,
          "value": 0,
          "name": "cell52 ~ gene11"
        },
        {
          "x": 51,
          "y": 11,
          "value": 0,
          "name": "cell52 ~ gene12"
        },
        {
          "x": 51,
          "y": 12,
          "value": 0,
          "name": "cell52 ~ gene13"
        },
        {
          "x": 51,
          "y": 13,
          "value": 13,
          "name": "cell52 ~ gene14"
        },
        {
          "x": 51,
          "y": 14,
          "value": 11,
          "name": "cell52 ~ gene15"
        },
        {
          "x": 51,
          "y": 15,
          "value": 0,
          "name": "cell52 ~ gene16"
        },
        {
          "x": 51,
          "y": 16,
          "value": 8,
          "name": "cell52 ~ gene17"
        },
        {
          "x": 51,
          "y": 17,
          "value": 0,
          "name": "cell52 ~ gene18"
        },
        {
          "x": 51,
          "y": 18,
          "value": 1,
          "name": "cell52 ~ gene19"
        },
        {
          "x": 51,
          "y": 19,
          "value": 12,
          "name": "cell52 ~ gene20"
        },
        {
          "x": 51,
          "y": 20,
          "value": 0,
          "name": "cell52 ~ gene21"
        },
        {
          "x": 51,
          "y": 21,
          "value": 26,
          "name": "cell52 ~ gene22"
        },
        {
          "x": 51,
          "y": 22,
          "value": 7,
          "name": "cell52 ~ gene23"
        },
        {
          "x": 51,
          "y": 23,
          "value": 0,
          "name": "cell52 ~ gene24"
        },
        {
          "x": 51,
          "y": 24,
          "value": 7,
          "name": "cell52 ~ gene25"
        },
        {
          "x": 51,
          "y": 25,
          "value": 6,
          "name": "cell52 ~ gene26"
        },
        {
          "x": 51,
          "y": 26,
          "value": 0,
          "name": "cell52 ~ gene27"
        },
        {
          "x": 51,
          "y": 27,
          "value": 7,
          "name": "cell52 ~ gene28"
        },
        {
          "x": 51,
          "y": 28,
          "value": 0,
          "name": "cell52 ~ gene29"
        },
        {
          "x": 51,
          "y": 29,
          "value": 27,
          "name": "cell52 ~ gene30"
        },
        {
          "x": 51,
          "y": 30,
          "value": 21,
          "name": "cell52 ~ gene31"
        },
        {
          "x": 51,
          "y": 31,
          "value": 17,
          "name": "cell52 ~ gene32"
        },
        {
          "x": 51,
          "y": 32,
          "value": 0,
          "name": "cell52 ~ gene33"
        },
        {
          "x": 51,
          "y": 33,
          "value": 9,
          "name": "cell52 ~ gene34"
        },
        {
          "x": 51,
          "y": 34,
          "value": 26,
          "name": "cell52 ~ gene35"
        },
        {
          "x": 51,
          "y": 35,
          "value": 0,
          "name": "cell52 ~ gene36"
        },
        {
          "x": 51,
          "y": 36,
          "value": 7,
          "name": "cell52 ~ gene37"
        },
        {
          "x": 51,
          "y": 37,
          "value": 17,
          "name": "cell52 ~ gene38"
        },
        {
          "x": 51,
          "y": 38,
          "value": 0,
          "name": "cell52 ~ gene39"
        },
        {
          "x": 51,
          "y": 39,
          "value": 26,
          "name": "cell52 ~ gene40"
        },
        {
          "x": 51,
          "y": 40,
          "value": 0,
          "name": "cell52 ~ gene41"
        },
        {
          "x": 51,
          "y": 41,
          "value": 12,
          "name": "cell52 ~ gene42"
        },
        {
          "x": 51,
          "y": 42,
          "value": 0,
          "name": "cell52 ~ gene43"
        },
        {
          "x": 51,
          "y": 43,
          "value": 7,
          "name": "cell52 ~ gene44"
        },
        {
          "x": 51,
          "y": 44,
          "value": 4,
          "name": "cell52 ~ gene45"
        },
        {
          "x": 51,
          "y": 45,
          "value": 8,
          "name": "cell52 ~ gene46"
        },
        {
          "x": 51,
          "y": 46,
          "value": 0,
          "name": "cell52 ~ gene47"
        },
        {
          "x": 51,
          "y": 47,
          "value": 0,
          "name": "cell52 ~ gene48"
        },
        {
          "x": 51,
          "y": 48,
          "value": 0,
          "name": "cell52 ~ gene49"
        },
        {
          "x": 51,
          "y": 49,
          "value": 9,
          "name": "cell52 ~ gene50"
        },
        {
          "x": 51,
          "y": 50,
          "value": 0,
          "name": "cell52 ~ gene51"
        },
        {
          "x": 51,
          "y": 51,
          "value": 19,
          "name": "cell52 ~ gene52"
        },
        {
          "x": 51,
          "y": 52,
          "value": 0,
          "name": "cell52 ~ gene53"
        },
        {
          "x": 51,
          "y": 53,
          "value": 0,
          "name": "cell52 ~ gene54"
        },
        {
          "x": 51,
          "y": 54,
          "value": 19,
          "name": "cell52 ~ gene55"
        },
        {
          "x": 51,
          "y": 55,
          "value": 0,
          "name": "cell52 ~ gene56"
        },
        {
          "x": 51,
          "y": 56,
          "value": 1,
          "name": "cell52 ~ gene57"
        },
        {
          "x": 51,
          "y": 57,
          "value": 2,
          "name": "cell52 ~ gene58"
        },
        {
          "x": 51,
          "y": 58,
          "value": 0,
          "name": "cell52 ~ gene59"
        },
        {
          "x": 51,
          "y": 59,
          "value": 0,
          "name": "cell52 ~ gene60"
        },
        {
          "x": 51,
          "y": 60,
          "value": 0,
          "name": "cell52 ~ gene61"
        },
        {
          "x": 51,
          "y": 61,
          "value": 0,
          "name": "cell52 ~ gene62"
        },
        {
          "x": 51,
          "y": 62,
          "value": 5,
          "name": "cell52 ~ gene63"
        },
        {
          "x": 51,
          "y": 63,
          "value": 0,
          "name": "cell52 ~ gene64"
        },
        {
          "x": 51,
          "y": 64,
          "value": 0,
          "name": "cell52 ~ gene65"
        },
        {
          "x": 51,
          "y": 65,
          "value": 6,
          "name": "cell52 ~ gene66"
        },
        {
          "x": 51,
          "y": 66,
          "value": 0,
          "name": "cell52 ~ gene67"
        },
        {
          "x": 51,
          "y": 67,
          "value": 0,
          "name": "cell52 ~ gene68"
        },
        {
          "x": 51,
          "y": 68,
          "value": 0,
          "name": "cell52 ~ gene69"
        },
        {
          "x": 51,
          "y": 69,
          "value": 1,
          "name": "cell52 ~ gene70"
        },
        {
          "x": 51,
          "y": 70,
          "value": 0,
          "name": "cell52 ~ gene71"
        },
        {
          "x": 51,
          "y": 71,
          "value": 0,
          "name": "cell52 ~ gene72"
        },
        {
          "x": 51,
          "y": 72,
          "value": 0,
          "name": "cell52 ~ gene73"
        },
        {
          "x": 51,
          "y": 73,
          "value": 12,
          "name": "cell52 ~ gene74"
        },
        {
          "x": 51,
          "y": 74,
          "value": 0,
          "name": "cell52 ~ gene75"
        },
        {
          "x": 51,
          "y": 75,
          "value": 7,
          "name": "cell52 ~ gene76"
        },
        {
          "x": 51,
          "y": 76,
          "value": 11,
          "name": "cell52 ~ gene77"
        },
        {
          "x": 51,
          "y": 77,
          "value": 2,
          "name": "cell52 ~ gene78"
        },
        {
          "x": 51,
          "y": 78,
          "value": 0,
          "name": "cell52 ~ gene79"
        },
        {
          "x": 51,
          "y": 79,
          "value": 14,
          "name": "cell52 ~ gene80"
        },
        {
          "x": 51,
          "y": 80,
          "value": 0,
          "name": "cell52 ~ gene81"
        },
        {
          "x": 51,
          "y": 81,
          "value": 0,
          "name": "cell52 ~ gene82"
        },
        {
          "x": 51,
          "y": 82,
          "value": 7,
          "name": "cell52 ~ gene83"
        },
        {
          "x": 51,
          "y": 83,
          "value": 12,
          "name": "cell52 ~ gene84"
        },
        {
          "x": 51,
          "y": 84,
          "value": 9,
          "name": "cell52 ~ gene85"
        },
        {
          "x": 51,
          "y": 85,
          "value": 5,
          "name": "cell52 ~ gene86"
        },
        {
          "x": 51,
          "y": 86,
          "value": 5,
          "name": "cell52 ~ gene87"
        },
        {
          "x": 51,
          "y": 87,
          "value": 4,
          "name": "cell52 ~ gene88"
        },
        {
          "x": 51,
          "y": 88,
          "value": 6,
          "name": "cell52 ~ gene89"
        },
        {
          "x": 51,
          "y": 89,
          "value": 13,
          "name": "cell52 ~ gene90"
        },
        {
          "x": 51,
          "y": 90,
          "value": 0,
          "name": "cell52 ~ gene91"
        },
        {
          "x": 51,
          "y": 91,
          "value": 0,
          "name": "cell52 ~ gene92"
        },
        {
          "x": 51,
          "y": 92,
          "value": 4,
          "name": "cell52 ~ gene93"
        },
        {
          "x": 51,
          "y": 93,
          "value": 9,
          "name": "cell52 ~ gene94"
        },
        {
          "x": 51,
          "y": 94,
          "value": 0,
          "name": "cell52 ~ gene95"
        },
        {
          "x": 51,
          "y": 95,
          "value": 0,
          "name": "cell52 ~ gene96"
        },
        {
          "x": 51,
          "y": 96,
          "value": 0,
          "name": "cell52 ~ gene97"
        },
        {
          "x": 51,
          "y": 97,
          "value": 3,
          "name": "cell52 ~ gene98"
        },
        {
          "x": 51,
          "y": 98,
          "value": 0,
          "name": "cell52 ~ gene99"
        },
        {
          "x": 51,
          "y": 99,
          "value": 0,
          "name": "cell52 ~ gene100"
        },
        {
          "x": 52,
          "y": 0,
          "value": 0,
          "name": "cell53 ~ gene1"
        },
        {
          "x": 52,
          "y": 1,
          "value": 12,
          "name": "cell53 ~ gene2"
        },
        {
          "x": 52,
          "y": 2,
          "value": 14,
          "name": "cell53 ~ gene3"
        },
        {
          "x": 52,
          "y": 3,
          "value": 0,
          "name": "cell53 ~ gene4"
        },
        {
          "x": 52,
          "y": 4,
          "value": 0,
          "name": "cell53 ~ gene5"
        },
        {
          "x": 52,
          "y": 5,
          "value": 0,
          "name": "cell53 ~ gene6"
        },
        {
          "x": 52,
          "y": 6,
          "value": 9,
          "name": "cell53 ~ gene7"
        },
        {
          "x": 52,
          "y": 7,
          "value": 0,
          "name": "cell53 ~ gene8"
        },
        {
          "x": 52,
          "y": 8,
          "value": 3,
          "name": "cell53 ~ gene9"
        },
        {
          "x": 52,
          "y": 9,
          "value": 11,
          "name": "cell53 ~ gene10"
        },
        {
          "x": 52,
          "y": 10,
          "value": 6,
          "name": "cell53 ~ gene11"
        },
        {
          "x": 52,
          "y": 11,
          "value": 0,
          "name": "cell53 ~ gene12"
        },
        {
          "x": 52,
          "y": 12,
          "value": 0,
          "name": "cell53 ~ gene13"
        },
        {
          "x": 52,
          "y": 13,
          "value": 14,
          "name": "cell53 ~ gene14"
        },
        {
          "x": 52,
          "y": 14,
          "value": 3,
          "name": "cell53 ~ gene15"
        },
        {
          "x": 52,
          "y": 15,
          "value": 0,
          "name": "cell53 ~ gene16"
        },
        {
          "x": 52,
          "y": 16,
          "value": 20,
          "name": "cell53 ~ gene17"
        },
        {
          "x": 52,
          "y": 17,
          "value": 0,
          "name": "cell53 ~ gene18"
        },
        {
          "x": 52,
          "y": 18,
          "value": 0,
          "name": "cell53 ~ gene19"
        },
        {
          "x": 52,
          "y": 19,
          "value": 0,
          "name": "cell53 ~ gene20"
        },
        {
          "x": 52,
          "y": 20,
          "value": 1,
          "name": "cell53 ~ gene21"
        },
        {
          "x": 52,
          "y": 21,
          "value": 46,
          "name": "cell53 ~ gene22"
        },
        {
          "x": 52,
          "y": 22,
          "value": 12,
          "name": "cell53 ~ gene23"
        },
        {
          "x": 52,
          "y": 23,
          "value": 0,
          "name": "cell53 ~ gene24"
        },
        {
          "x": 52,
          "y": 24,
          "value": 0,
          "name": "cell53 ~ gene25"
        },
        {
          "x": 52,
          "y": 25,
          "value": 13,
          "name": "cell53 ~ gene26"
        },
        {
          "x": 52,
          "y": 26,
          "value": 16,
          "name": "cell53 ~ gene27"
        },
        {
          "x": 52,
          "y": 27,
          "value": 22,
          "name": "cell53 ~ gene28"
        },
        {
          "x": 52,
          "y": 28,
          "value": 0,
          "name": "cell53 ~ gene29"
        },
        {
          "x": 52,
          "y": 29,
          "value": 30,
          "name": "cell53 ~ gene30"
        },
        {
          "x": 52,
          "y": 30,
          "value": 45,
          "name": "cell53 ~ gene31"
        },
        {
          "x": 52,
          "y": 31,
          "value": 10,
          "name": "cell53 ~ gene32"
        },
        {
          "x": 52,
          "y": 32,
          "value": 9,
          "name": "cell53 ~ gene33"
        },
        {
          "x": 52,
          "y": 33,
          "value": 9,
          "name": "cell53 ~ gene34"
        },
        {
          "x": 52,
          "y": 34,
          "value": 23,
          "name": "cell53 ~ gene35"
        },
        {
          "x": 52,
          "y": 35,
          "value": 0,
          "name": "cell53 ~ gene36"
        },
        {
          "x": 52,
          "y": 36,
          "value": 22,
          "name": "cell53 ~ gene37"
        },
        {
          "x": 52,
          "y": 37,
          "value": 25,
          "name": "cell53 ~ gene38"
        },
        {
          "x": 52,
          "y": 38,
          "value": 11,
          "name": "cell53 ~ gene39"
        },
        {
          "x": 52,
          "y": 39,
          "value": 10,
          "name": "cell53 ~ gene40"
        },
        {
          "x": 52,
          "y": 40,
          "value": 10,
          "name": "cell53 ~ gene41"
        },
        {
          "x": 52,
          "y": 41,
          "value": 11,
          "name": "cell53 ~ gene42"
        },
        {
          "x": 52,
          "y": 42,
          "value": 0,
          "name": "cell53 ~ gene43"
        },
        {
          "x": 52,
          "y": 43,
          "value": 0,
          "name": "cell53 ~ gene44"
        },
        {
          "x": 52,
          "y": 44,
          "value": 23,
          "name": "cell53 ~ gene45"
        },
        {
          "x": 52,
          "y": 45,
          "value": 9,
          "name": "cell53 ~ gene46"
        },
        {
          "x": 52,
          "y": 46,
          "value": 2,
          "name": "cell53 ~ gene47"
        },
        {
          "x": 52,
          "y": 47,
          "value": 3,
          "name": "cell53 ~ gene48"
        },
        {
          "x": 52,
          "y": 48,
          "value": 0,
          "name": "cell53 ~ gene49"
        },
        {
          "x": 52,
          "y": 49,
          "value": 1,
          "name": "cell53 ~ gene50"
        },
        {
          "x": 52,
          "y": 50,
          "value": 0,
          "name": "cell53 ~ gene51"
        },
        {
          "x": 52,
          "y": 51,
          "value": 4,
          "name": "cell53 ~ gene52"
        },
        {
          "x": 52,
          "y": 52,
          "value": 14,
          "name": "cell53 ~ gene53"
        },
        {
          "x": 52,
          "y": 53,
          "value": 5,
          "name": "cell53 ~ gene54"
        },
        {
          "x": 52,
          "y": 54,
          "value": 0,
          "name": "cell53 ~ gene55"
        },
        {
          "x": 52,
          "y": 55,
          "value": 8,
          "name": "cell53 ~ gene56"
        },
        {
          "x": 52,
          "y": 56,
          "value": 19,
          "name": "cell53 ~ gene57"
        },
        {
          "x": 52,
          "y": 57,
          "value": 4,
          "name": "cell53 ~ gene58"
        },
        {
          "x": 52,
          "y": 58,
          "value": 16,
          "name": "cell53 ~ gene59"
        },
        {
          "x": 52,
          "y": 59,
          "value": 0,
          "name": "cell53 ~ gene60"
        },
        {
          "x": 52,
          "y": 60,
          "value": 0,
          "name": "cell53 ~ gene61"
        },
        {
          "x": 52,
          "y": 61,
          "value": 1,
          "name": "cell53 ~ gene62"
        },
        {
          "x": 52,
          "y": 62,
          "value": 10,
          "name": "cell53 ~ gene63"
        },
        {
          "x": 52,
          "y": 63,
          "value": 2,
          "name": "cell53 ~ gene64"
        },
        {
          "x": 52,
          "y": 64,
          "value": 0,
          "name": "cell53 ~ gene65"
        },
        {
          "x": 52,
          "y": 65,
          "value": 0,
          "name": "cell53 ~ gene66"
        },
        {
          "x": 52,
          "y": 66,
          "value": 0,
          "name": "cell53 ~ gene67"
        },
        {
          "x": 52,
          "y": 67,
          "value": 14,
          "name": "cell53 ~ gene68"
        },
        {
          "x": 52,
          "y": 68,
          "value": 13,
          "name": "cell53 ~ gene69"
        },
        {
          "x": 52,
          "y": 69,
          "value": 10,
          "name": "cell53 ~ gene70"
        },
        {
          "x": 52,
          "y": 70,
          "value": 10,
          "name": "cell53 ~ gene71"
        },
        {
          "x": 52,
          "y": 71,
          "value": 5,
          "name": "cell53 ~ gene72"
        },
        {
          "x": 52,
          "y": 72,
          "value": 0,
          "name": "cell53 ~ gene73"
        },
        {
          "x": 52,
          "y": 73,
          "value": 1,
          "name": "cell53 ~ gene74"
        },
        {
          "x": 52,
          "y": 74,
          "value": 0,
          "name": "cell53 ~ gene75"
        },
        {
          "x": 52,
          "y": 75,
          "value": 10,
          "name": "cell53 ~ gene76"
        },
        {
          "x": 52,
          "y": 76,
          "value": 0,
          "name": "cell53 ~ gene77"
        },
        {
          "x": 52,
          "y": 77,
          "value": 1,
          "name": "cell53 ~ gene78"
        },
        {
          "x": 52,
          "y": 78,
          "value": 2,
          "name": "cell53 ~ gene79"
        },
        {
          "x": 52,
          "y": 79,
          "value": 7,
          "name": "cell53 ~ gene80"
        },
        {
          "x": 52,
          "y": 80,
          "value": 0,
          "name": "cell53 ~ gene81"
        },
        {
          "x": 52,
          "y": 81,
          "value": 0,
          "name": "cell53 ~ gene82"
        },
        {
          "x": 52,
          "y": 82,
          "value": 0,
          "name": "cell53 ~ gene83"
        },
        {
          "x": 52,
          "y": 83,
          "value": 4,
          "name": "cell53 ~ gene84"
        },
        {
          "x": 52,
          "y": 84,
          "value": 7,
          "name": "cell53 ~ gene85"
        },
        {
          "x": 52,
          "y": 85,
          "value": 0,
          "name": "cell53 ~ gene86"
        },
        {
          "x": 52,
          "y": 86,
          "value": 7,
          "name": "cell53 ~ gene87"
        },
        {
          "x": 52,
          "y": 87,
          "value": 15,
          "name": "cell53 ~ gene88"
        },
        {
          "x": 52,
          "y": 88,
          "value": 3,
          "name": "cell53 ~ gene89"
        },
        {
          "x": 52,
          "y": 89,
          "value": 10,
          "name": "cell53 ~ gene90"
        },
        {
          "x": 52,
          "y": 90,
          "value": 2,
          "name": "cell53 ~ gene91"
        },
        {
          "x": 52,
          "y": 91,
          "value": 1,
          "name": "cell53 ~ gene92"
        },
        {
          "x": 52,
          "y": 92,
          "value": 0,
          "name": "cell53 ~ gene93"
        },
        {
          "x": 52,
          "y": 93,
          "value": 0,
          "name": "cell53 ~ gene94"
        },
        {
          "x": 52,
          "y": 94,
          "value": 0,
          "name": "cell53 ~ gene95"
        },
        {
          "x": 52,
          "y": 95,
          "value": 10,
          "name": "cell53 ~ gene96"
        },
        {
          "x": 52,
          "y": 96,
          "value": 0,
          "name": "cell53 ~ gene97"
        },
        {
          "x": 52,
          "y": 97,
          "value": 0,
          "name": "cell53 ~ gene98"
        },
        {
          "x": 52,
          "y": 98,
          "value": 5,
          "name": "cell53 ~ gene99"
        },
        {
          "x": 52,
          "y": 99,
          "value": 7,
          "name": "cell53 ~ gene100"
        },
        {
          "x": 53,
          "y": 0,
          "value": 0,
          "name": "cell54 ~ gene1"
        },
        {
          "x": 53,
          "y": 1,
          "value": 17,
          "name": "cell54 ~ gene2"
        },
        {
          "x": 53,
          "y": 2,
          "value": 0,
          "name": "cell54 ~ gene3"
        },
        {
          "x": 53,
          "y": 3,
          "value": 0,
          "name": "cell54 ~ gene4"
        },
        {
          "x": 53,
          "y": 4,
          "value": 0,
          "name": "cell54 ~ gene5"
        },
        {
          "x": 53,
          "y": 5,
          "value": 6,
          "name": "cell54 ~ gene6"
        },
        {
          "x": 53,
          "y": 6,
          "value": 0,
          "name": "cell54 ~ gene7"
        },
        {
          "x": 53,
          "y": 7,
          "value": 0,
          "name": "cell54 ~ gene8"
        },
        {
          "x": 53,
          "y": 8,
          "value": 0,
          "name": "cell54 ~ gene9"
        },
        {
          "x": 53,
          "y": 9,
          "value": 3,
          "name": "cell54 ~ gene10"
        },
        {
          "x": 53,
          "y": 10,
          "value": 13,
          "name": "cell54 ~ gene11"
        },
        {
          "x": 53,
          "y": 11,
          "value": 0,
          "name": "cell54 ~ gene12"
        },
        {
          "x": 53,
          "y": 12,
          "value": 3,
          "name": "cell54 ~ gene13"
        },
        {
          "x": 53,
          "y": 13,
          "value": 0,
          "name": "cell54 ~ gene14"
        },
        {
          "x": 53,
          "y": 14,
          "value": 0,
          "name": "cell54 ~ gene15"
        },
        {
          "x": 53,
          "y": 15,
          "value": 8,
          "name": "cell54 ~ gene16"
        },
        {
          "x": 53,
          "y": 16,
          "value": 16,
          "name": "cell54 ~ gene17"
        },
        {
          "x": 53,
          "y": 17,
          "value": 0,
          "name": "cell54 ~ gene18"
        },
        {
          "x": 53,
          "y": 18,
          "value": 3,
          "name": "cell54 ~ gene19"
        },
        {
          "x": 53,
          "y": 19,
          "value": 1,
          "name": "cell54 ~ gene20"
        },
        {
          "x": 53,
          "y": 20,
          "value": 15,
          "name": "cell54 ~ gene21"
        },
        {
          "x": 53,
          "y": 21,
          "value": 24,
          "name": "cell54 ~ gene22"
        },
        {
          "x": 53,
          "y": 22,
          "value": 14,
          "name": "cell54 ~ gene23"
        },
        {
          "x": 53,
          "y": 23,
          "value": 0,
          "name": "cell54 ~ gene24"
        },
        {
          "x": 53,
          "y": 24,
          "value": 14,
          "name": "cell54 ~ gene25"
        },
        {
          "x": 53,
          "y": 25,
          "value": 15,
          "name": "cell54 ~ gene26"
        },
        {
          "x": 53,
          "y": 26,
          "value": 16,
          "name": "cell54 ~ gene27"
        },
        {
          "x": 53,
          "y": 27,
          "value": 7,
          "name": "cell54 ~ gene28"
        },
        {
          "x": 53,
          "y": 28,
          "value": 2,
          "name": "cell54 ~ gene29"
        },
        {
          "x": 53,
          "y": 29,
          "value": 23,
          "name": "cell54 ~ gene30"
        },
        {
          "x": 53,
          "y": 30,
          "value": 42,
          "name": "cell54 ~ gene31"
        },
        {
          "x": 53,
          "y": 31,
          "value": 17,
          "name": "cell54 ~ gene32"
        },
        {
          "x": 53,
          "y": 32,
          "value": 0,
          "name": "cell54 ~ gene33"
        },
        {
          "x": 53,
          "y": 33,
          "value": 12,
          "name": "cell54 ~ gene34"
        },
        {
          "x": 53,
          "y": 34,
          "value": 12,
          "name": "cell54 ~ gene35"
        },
        {
          "x": 53,
          "y": 35,
          "value": 0,
          "name": "cell54 ~ gene36"
        },
        {
          "x": 53,
          "y": 36,
          "value": 28,
          "name": "cell54 ~ gene37"
        },
        {
          "x": 53,
          "y": 37,
          "value": 14,
          "name": "cell54 ~ gene38"
        },
        {
          "x": 53,
          "y": 38,
          "value": 3,
          "name": "cell54 ~ gene39"
        },
        {
          "x": 53,
          "y": 39,
          "value": 0,
          "name": "cell54 ~ gene40"
        },
        {
          "x": 53,
          "y": 40,
          "value": 0,
          "name": "cell54 ~ gene41"
        },
        {
          "x": 53,
          "y": 41,
          "value": 2,
          "name": "cell54 ~ gene42"
        },
        {
          "x": 53,
          "y": 42,
          "value": 0,
          "name": "cell54 ~ gene43"
        },
        {
          "x": 53,
          "y": 43,
          "value": 0,
          "name": "cell54 ~ gene44"
        },
        {
          "x": 53,
          "y": 44,
          "value": 0,
          "name": "cell54 ~ gene45"
        },
        {
          "x": 53,
          "y": 45,
          "value": 0,
          "name": "cell54 ~ gene46"
        },
        {
          "x": 53,
          "y": 46,
          "value": 4,
          "name": "cell54 ~ gene47"
        },
        {
          "x": 53,
          "y": 47,
          "value": 8,
          "name": "cell54 ~ gene48"
        },
        {
          "x": 53,
          "y": 48,
          "value": 6,
          "name": "cell54 ~ gene49"
        },
        {
          "x": 53,
          "y": 49,
          "value": 0,
          "name": "cell54 ~ gene50"
        },
        {
          "x": 53,
          "y": 50,
          "value": 0,
          "name": "cell54 ~ gene51"
        },
        {
          "x": 53,
          "y": 51,
          "value": 0,
          "name": "cell54 ~ gene52"
        },
        {
          "x": 53,
          "y": 52,
          "value": 0,
          "name": "cell54 ~ gene53"
        },
        {
          "x": 53,
          "y": 53,
          "value": 2,
          "name": "cell54 ~ gene54"
        },
        {
          "x": 53,
          "y": 54,
          "value": 0,
          "name": "cell54 ~ gene55"
        },
        {
          "x": 53,
          "y": 55,
          "value": 3,
          "name": "cell54 ~ gene56"
        },
        {
          "x": 53,
          "y": 56,
          "value": 0,
          "name": "cell54 ~ gene57"
        },
        {
          "x": 53,
          "y": 57,
          "value": 2,
          "name": "cell54 ~ gene58"
        },
        {
          "x": 53,
          "y": 58,
          "value": 0,
          "name": "cell54 ~ gene59"
        },
        {
          "x": 53,
          "y": 59,
          "value": 0,
          "name": "cell54 ~ gene60"
        },
        {
          "x": 53,
          "y": 60,
          "value": 0,
          "name": "cell54 ~ gene61"
        },
        {
          "x": 53,
          "y": 61,
          "value": 5,
          "name": "cell54 ~ gene62"
        },
        {
          "x": 53,
          "y": 62,
          "value": 0,
          "name": "cell54 ~ gene63"
        },
        {
          "x": 53,
          "y": 63,
          "value": 2,
          "name": "cell54 ~ gene64"
        },
        {
          "x": 53,
          "y": 64,
          "value": 7,
          "name": "cell54 ~ gene65"
        },
        {
          "x": 53,
          "y": 65,
          "value": 8,
          "name": "cell54 ~ gene66"
        },
        {
          "x": 53,
          "y": 66,
          "value": 0,
          "name": "cell54 ~ gene67"
        },
        {
          "x": 53,
          "y": 67,
          "value": 5,
          "name": "cell54 ~ gene68"
        },
        {
          "x": 53,
          "y": 68,
          "value": 4,
          "name": "cell54 ~ gene69"
        },
        {
          "x": 53,
          "y": 69,
          "value": 0,
          "name": "cell54 ~ gene70"
        },
        {
          "x": 53,
          "y": 70,
          "value": 0,
          "name": "cell54 ~ gene71"
        },
        {
          "x": 53,
          "y": 71,
          "value": 0,
          "name": "cell54 ~ gene72"
        },
        {
          "x": 53,
          "y": 72,
          "value": 8,
          "name": "cell54 ~ gene73"
        },
        {
          "x": 53,
          "y": 73,
          "value": 0,
          "name": "cell54 ~ gene74"
        },
        {
          "x": 53,
          "y": 74,
          "value": 0,
          "name": "cell54 ~ gene75"
        },
        {
          "x": 53,
          "y": 75,
          "value": 7,
          "name": "cell54 ~ gene76"
        },
        {
          "x": 53,
          "y": 76,
          "value": 6,
          "name": "cell54 ~ gene77"
        },
        {
          "x": 53,
          "y": 77,
          "value": 0,
          "name": "cell54 ~ gene78"
        },
        {
          "x": 53,
          "y": 78,
          "value": 4,
          "name": "cell54 ~ gene79"
        },
        {
          "x": 53,
          "y": 79,
          "value": 3,
          "name": "cell54 ~ gene80"
        },
        {
          "x": 53,
          "y": 80,
          "value": 23,
          "name": "cell54 ~ gene81"
        },
        {
          "x": 53,
          "y": 81,
          "value": 0,
          "name": "cell54 ~ gene82"
        },
        {
          "x": 53,
          "y": 82,
          "value": 8,
          "name": "cell54 ~ gene83"
        },
        {
          "x": 53,
          "y": 83,
          "value": 8,
          "name": "cell54 ~ gene84"
        },
        {
          "x": 53,
          "y": 84,
          "value": 0,
          "name": "cell54 ~ gene85"
        },
        {
          "x": 53,
          "y": 85,
          "value": 0,
          "name": "cell54 ~ gene86"
        },
        {
          "x": 53,
          "y": 86,
          "value": 18,
          "name": "cell54 ~ gene87"
        },
        {
          "x": 53,
          "y": 87,
          "value": 0,
          "name": "cell54 ~ gene88"
        },
        {
          "x": 53,
          "y": 88,
          "value": 0,
          "name": "cell54 ~ gene89"
        },
        {
          "x": 53,
          "y": 89,
          "value": 0,
          "name": "cell54 ~ gene90"
        },
        {
          "x": 53,
          "y": 90,
          "value": 4,
          "name": "cell54 ~ gene91"
        },
        {
          "x": 53,
          "y": 91,
          "value": 0,
          "name": "cell54 ~ gene92"
        },
        {
          "x": 53,
          "y": 92,
          "value": 23,
          "name": "cell54 ~ gene93"
        },
        {
          "x": 53,
          "y": 93,
          "value": 0,
          "name": "cell54 ~ gene94"
        },
        {
          "x": 53,
          "y": 94,
          "value": 3,
          "name": "cell54 ~ gene95"
        },
        {
          "x": 53,
          "y": 95,
          "value": 6,
          "name": "cell54 ~ gene96"
        },
        {
          "x": 53,
          "y": 96,
          "value": 0,
          "name": "cell54 ~ gene97"
        },
        {
          "x": 53,
          "y": 97,
          "value": 0,
          "name": "cell54 ~ gene98"
        },
        {
          "x": 53,
          "y": 98,
          "value": 0,
          "name": "cell54 ~ gene99"
        },
        {
          "x": 53,
          "y": 99,
          "value": 0,
          "name": "cell54 ~ gene100"
        },
        {
          "x": 54,
          "y": 0,
          "value": 8,
          "name": "cell55 ~ gene1"
        },
        {
          "x": 54,
          "y": 1,
          "value": 8,
          "name": "cell55 ~ gene2"
        },
        {
          "x": 54,
          "y": 2,
          "value": 4,
          "name": "cell55 ~ gene3"
        },
        {
          "x": 54,
          "y": 3,
          "value": 9,
          "name": "cell55 ~ gene4"
        },
        {
          "x": 54,
          "y": 4,
          "value": 0,
          "name": "cell55 ~ gene5"
        },
        {
          "x": 54,
          "y": 5,
          "value": 0,
          "name": "cell55 ~ gene6"
        },
        {
          "x": 54,
          "y": 6,
          "value": 0,
          "name": "cell55 ~ gene7"
        },
        {
          "x": 54,
          "y": 7,
          "value": 10,
          "name": "cell55 ~ gene8"
        },
        {
          "x": 54,
          "y": 8,
          "value": 0,
          "name": "cell55 ~ gene9"
        },
        {
          "x": 54,
          "y": 9,
          "value": 8,
          "name": "cell55 ~ gene10"
        },
        {
          "x": 54,
          "y": 10,
          "value": 9,
          "name": "cell55 ~ gene11"
        },
        {
          "x": 54,
          "y": 11,
          "value": 4,
          "name": "cell55 ~ gene12"
        },
        {
          "x": 54,
          "y": 12,
          "value": 0,
          "name": "cell55 ~ gene13"
        },
        {
          "x": 54,
          "y": 13,
          "value": 13,
          "name": "cell55 ~ gene14"
        },
        {
          "x": 54,
          "y": 14,
          "value": 1,
          "name": "cell55 ~ gene15"
        },
        {
          "x": 54,
          "y": 15,
          "value": 12,
          "name": "cell55 ~ gene16"
        },
        {
          "x": 54,
          "y": 16,
          "value": 0,
          "name": "cell55 ~ gene17"
        },
        {
          "x": 54,
          "y": 17,
          "value": 13,
          "name": "cell55 ~ gene18"
        },
        {
          "x": 54,
          "y": 18,
          "value": 0,
          "name": "cell55 ~ gene19"
        },
        {
          "x": 54,
          "y": 19,
          "value": 0,
          "name": "cell55 ~ gene20"
        },
        {
          "x": 54,
          "y": 20,
          "value": 5,
          "name": "cell55 ~ gene21"
        },
        {
          "x": 54,
          "y": 21,
          "value": 35,
          "name": "cell55 ~ gene22"
        },
        {
          "x": 54,
          "y": 22,
          "value": 8,
          "name": "cell55 ~ gene23"
        },
        {
          "x": 54,
          "y": 23,
          "value": 0,
          "name": "cell55 ~ gene24"
        },
        {
          "x": 54,
          "y": 24,
          "value": 0,
          "name": "cell55 ~ gene25"
        },
        {
          "x": 54,
          "y": 25,
          "value": 25,
          "name": "cell55 ~ gene26"
        },
        {
          "x": 54,
          "y": 26,
          "value": 17,
          "name": "cell55 ~ gene27"
        },
        {
          "x": 54,
          "y": 27,
          "value": 9,
          "name": "cell55 ~ gene28"
        },
        {
          "x": 54,
          "y": 28,
          "value": 3,
          "name": "cell55 ~ gene29"
        },
        {
          "x": 54,
          "y": 29,
          "value": 21,
          "name": "cell55 ~ gene30"
        },
        {
          "x": 54,
          "y": 30,
          "value": 43,
          "name": "cell55 ~ gene31"
        },
        {
          "x": 54,
          "y": 31,
          "value": 0,
          "name": "cell55 ~ gene32"
        },
        {
          "x": 54,
          "y": 32,
          "value": 1,
          "name": "cell55 ~ gene33"
        },
        {
          "x": 54,
          "y": 33,
          "value": 0,
          "name": "cell55 ~ gene34"
        },
        {
          "x": 54,
          "y": 34,
          "value": 1,
          "name": "cell55 ~ gene35"
        },
        {
          "x": 54,
          "y": 35,
          "value": 7,
          "name": "cell55 ~ gene36"
        },
        {
          "x": 54,
          "y": 36,
          "value": 28,
          "name": "cell55 ~ gene37"
        },
        {
          "x": 54,
          "y": 37,
          "value": 22,
          "name": "cell55 ~ gene38"
        },
        {
          "x": 54,
          "y": 38,
          "value": 12,
          "name": "cell55 ~ gene39"
        },
        {
          "x": 54,
          "y": 39,
          "value": 7,
          "name": "cell55 ~ gene40"
        },
        {
          "x": 54,
          "y": 40,
          "value": 0,
          "name": "cell55 ~ gene41"
        },
        {
          "x": 54,
          "y": 41,
          "value": 0,
          "name": "cell55 ~ gene42"
        },
        {
          "x": 54,
          "y": 42,
          "value": 0,
          "name": "cell55 ~ gene43"
        },
        {
          "x": 54,
          "y": 43,
          "value": 12,
          "name": "cell55 ~ gene44"
        },
        {
          "x": 54,
          "y": 44,
          "value": 0,
          "name": "cell55 ~ gene45"
        },
        {
          "x": 54,
          "y": 45,
          "value": 18,
          "name": "cell55 ~ gene46"
        },
        {
          "x": 54,
          "y": 46,
          "value": 0,
          "name": "cell55 ~ gene47"
        },
        {
          "x": 54,
          "y": 47,
          "value": 0,
          "name": "cell55 ~ gene48"
        },
        {
          "x": 54,
          "y": 48,
          "value": 4,
          "name": "cell55 ~ gene49"
        },
        {
          "x": 54,
          "y": 49,
          "value": 10,
          "name": "cell55 ~ gene50"
        },
        {
          "x": 54,
          "y": 50,
          "value": 7,
          "name": "cell55 ~ gene51"
        },
        {
          "x": 54,
          "y": 51,
          "value": 1,
          "name": "cell55 ~ gene52"
        },
        {
          "x": 54,
          "y": 52,
          "value": 5,
          "name": "cell55 ~ gene53"
        },
        {
          "x": 54,
          "y": 53,
          "value": 5,
          "name": "cell55 ~ gene54"
        },
        {
          "x": 54,
          "y": 54,
          "value": 17,
          "name": "cell55 ~ gene55"
        },
        {
          "x": 54,
          "y": 55,
          "value": 0,
          "name": "cell55 ~ gene56"
        },
        {
          "x": 54,
          "y": 56,
          "value": 10,
          "name": "cell55 ~ gene57"
        },
        {
          "x": 54,
          "y": 57,
          "value": 4,
          "name": "cell55 ~ gene58"
        },
        {
          "x": 54,
          "y": 58,
          "value": 16,
          "name": "cell55 ~ gene59"
        },
        {
          "x": 54,
          "y": 59,
          "value": 0,
          "name": "cell55 ~ gene60"
        },
        {
          "x": 54,
          "y": 60,
          "value": 0,
          "name": "cell55 ~ gene61"
        },
        {
          "x": 54,
          "y": 61,
          "value": 2,
          "name": "cell55 ~ gene62"
        },
        {
          "x": 54,
          "y": 62,
          "value": 14,
          "name": "cell55 ~ gene63"
        },
        {
          "x": 54,
          "y": 63,
          "value": 0,
          "name": "cell55 ~ gene64"
        },
        {
          "x": 54,
          "y": 64,
          "value": 4,
          "name": "cell55 ~ gene65"
        },
        {
          "x": 54,
          "y": 65,
          "value": 5,
          "name": "cell55 ~ gene66"
        },
        {
          "x": 54,
          "y": 66,
          "value": 0,
          "name": "cell55 ~ gene67"
        },
        {
          "x": 54,
          "y": 67,
          "value": 0,
          "name": "cell55 ~ gene68"
        },
        {
          "x": 54,
          "y": 68,
          "value": 0,
          "name": "cell55 ~ gene69"
        },
        {
          "x": 54,
          "y": 69,
          "value": 1,
          "name": "cell55 ~ gene70"
        },
        {
          "x": 54,
          "y": 70,
          "value": 7,
          "name": "cell55 ~ gene71"
        },
        {
          "x": 54,
          "y": 71,
          "value": 0,
          "name": "cell55 ~ gene72"
        },
        {
          "x": 54,
          "y": 72,
          "value": 0,
          "name": "cell55 ~ gene73"
        },
        {
          "x": 54,
          "y": 73,
          "value": 2,
          "name": "cell55 ~ gene74"
        },
        {
          "x": 54,
          "y": 74,
          "value": 4,
          "name": "cell55 ~ gene75"
        },
        {
          "x": 54,
          "y": 75,
          "value": 0,
          "name": "cell55 ~ gene76"
        },
        {
          "x": 54,
          "y": 76,
          "value": 0,
          "name": "cell55 ~ gene77"
        },
        {
          "x": 54,
          "y": 77,
          "value": 0,
          "name": "cell55 ~ gene78"
        },
        {
          "x": 54,
          "y": 78,
          "value": 4,
          "name": "cell55 ~ gene79"
        },
        {
          "x": 54,
          "y": 79,
          "value": 0,
          "name": "cell55 ~ gene80"
        },
        {
          "x": 54,
          "y": 80,
          "value": 0,
          "name": "cell55 ~ gene81"
        },
        {
          "x": 54,
          "y": 81,
          "value": 0,
          "name": "cell55 ~ gene82"
        },
        {
          "x": 54,
          "y": 82,
          "value": 0,
          "name": "cell55 ~ gene83"
        },
        {
          "x": 54,
          "y": 83,
          "value": 0,
          "name": "cell55 ~ gene84"
        },
        {
          "x": 54,
          "y": 84,
          "value": 19,
          "name": "cell55 ~ gene85"
        },
        {
          "x": 54,
          "y": 85,
          "value": 0,
          "name": "cell55 ~ gene86"
        },
        {
          "x": 54,
          "y": 86,
          "value": 6,
          "name": "cell55 ~ gene87"
        },
        {
          "x": 54,
          "y": 87,
          "value": 1,
          "name": "cell55 ~ gene88"
        },
        {
          "x": 54,
          "y": 88,
          "value": 13,
          "name": "cell55 ~ gene89"
        },
        {
          "x": 54,
          "y": 89,
          "value": 13,
          "name": "cell55 ~ gene90"
        },
        {
          "x": 54,
          "y": 90,
          "value": 7,
          "name": "cell55 ~ gene91"
        },
        {
          "x": 54,
          "y": 91,
          "value": 0,
          "name": "cell55 ~ gene92"
        },
        {
          "x": 54,
          "y": 92,
          "value": 17,
          "name": "cell55 ~ gene93"
        },
        {
          "x": 54,
          "y": 93,
          "value": 2,
          "name": "cell55 ~ gene94"
        },
        {
          "x": 54,
          "y": 94,
          "value": 1,
          "name": "cell55 ~ gene95"
        },
        {
          "x": 54,
          "y": 95,
          "value": 19,
          "name": "cell55 ~ gene96"
        },
        {
          "x": 54,
          "y": 96,
          "value": 7,
          "name": "cell55 ~ gene97"
        },
        {
          "x": 54,
          "y": 97,
          "value": 20,
          "name": "cell55 ~ gene98"
        },
        {
          "x": 54,
          "y": 98,
          "value": 0,
          "name": "cell55 ~ gene99"
        },
        {
          "x": 54,
          "y": 99,
          "value": 2,
          "name": "cell55 ~ gene100"
        },
        {
          "x": 55,
          "y": 0,
          "value": 0,
          "name": "cell56 ~ gene1"
        },
        {
          "x": 55,
          "y": 1,
          "value": 0,
          "name": "cell56 ~ gene2"
        },
        {
          "x": 55,
          "y": 2,
          "value": 3,
          "name": "cell56 ~ gene3"
        },
        {
          "x": 55,
          "y": 3,
          "value": 15,
          "name": "cell56 ~ gene4"
        },
        {
          "x": 55,
          "y": 4,
          "value": 6,
          "name": "cell56 ~ gene5"
        },
        {
          "x": 55,
          "y": 5,
          "value": 0,
          "name": "cell56 ~ gene6"
        },
        {
          "x": 55,
          "y": 6,
          "value": 2,
          "name": "cell56 ~ gene7"
        },
        {
          "x": 55,
          "y": 7,
          "value": 6,
          "name": "cell56 ~ gene8"
        },
        {
          "x": 55,
          "y": 8,
          "value": 0,
          "name": "cell56 ~ gene9"
        },
        {
          "x": 55,
          "y": 9,
          "value": 0,
          "name": "cell56 ~ gene10"
        },
        {
          "x": 55,
          "y": 10,
          "value": 5,
          "name": "cell56 ~ gene11"
        },
        {
          "x": 55,
          "y": 11,
          "value": 8,
          "name": "cell56 ~ gene12"
        },
        {
          "x": 55,
          "y": 12,
          "value": 0,
          "name": "cell56 ~ gene13"
        },
        {
          "x": 55,
          "y": 13,
          "value": 3,
          "name": "cell56 ~ gene14"
        },
        {
          "x": 55,
          "y": 14,
          "value": 0,
          "name": "cell56 ~ gene15"
        },
        {
          "x": 55,
          "y": 15,
          "value": 5,
          "name": "cell56 ~ gene16"
        },
        {
          "x": 55,
          "y": 16,
          "value": 0,
          "name": "cell56 ~ gene17"
        },
        {
          "x": 55,
          "y": 17,
          "value": 0,
          "name": "cell56 ~ gene18"
        },
        {
          "x": 55,
          "y": 18,
          "value": 0,
          "name": "cell56 ~ gene19"
        },
        {
          "x": 55,
          "y": 19,
          "value": 0,
          "name": "cell56 ~ gene20"
        },
        {
          "x": 55,
          "y": 20,
          "value": 0,
          "name": "cell56 ~ gene21"
        },
        {
          "x": 55,
          "y": 21,
          "value": 33,
          "name": "cell56 ~ gene22"
        },
        {
          "x": 55,
          "y": 22,
          "value": 6,
          "name": "cell56 ~ gene23"
        },
        {
          "x": 55,
          "y": 23,
          "value": 0,
          "name": "cell56 ~ gene24"
        },
        {
          "x": 55,
          "y": 24,
          "value": 17,
          "name": "cell56 ~ gene25"
        },
        {
          "x": 55,
          "y": 25,
          "value": 20,
          "name": "cell56 ~ gene26"
        },
        {
          "x": 55,
          "y": 26,
          "value": 12,
          "name": "cell56 ~ gene27"
        },
        {
          "x": 55,
          "y": 27,
          "value": 0,
          "name": "cell56 ~ gene28"
        },
        {
          "x": 55,
          "y": 28,
          "value": 0,
          "name": "cell56 ~ gene29"
        },
        {
          "x": 55,
          "y": 29,
          "value": 31,
          "name": "cell56 ~ gene30"
        },
        {
          "x": 55,
          "y": 30,
          "value": 37,
          "name": "cell56 ~ gene31"
        },
        {
          "x": 55,
          "y": 31,
          "value": 10,
          "name": "cell56 ~ gene32"
        },
        {
          "x": 55,
          "y": 32,
          "value": 0,
          "name": "cell56 ~ gene33"
        },
        {
          "x": 55,
          "y": 33,
          "value": 0,
          "name": "cell56 ~ gene34"
        },
        {
          "x": 55,
          "y": 34,
          "value": 11,
          "name": "cell56 ~ gene35"
        },
        {
          "x": 55,
          "y": 35,
          "value": 0,
          "name": "cell56 ~ gene36"
        },
        {
          "x": 55,
          "y": 36,
          "value": 3,
          "name": "cell56 ~ gene37"
        },
        {
          "x": 55,
          "y": 37,
          "value": 25,
          "name": "cell56 ~ gene38"
        },
        {
          "x": 55,
          "y": 38,
          "value": 10,
          "name": "cell56 ~ gene39"
        },
        {
          "x": 55,
          "y": 39,
          "value": 0,
          "name": "cell56 ~ gene40"
        },
        {
          "x": 55,
          "y": 40,
          "value": 0,
          "name": "cell56 ~ gene41"
        },
        {
          "x": 55,
          "y": 41,
          "value": 0,
          "name": "cell56 ~ gene42"
        },
        {
          "x": 55,
          "y": 42,
          "value": 0,
          "name": "cell56 ~ gene43"
        },
        {
          "x": 55,
          "y": 43,
          "value": 0,
          "name": "cell56 ~ gene44"
        },
        {
          "x": 55,
          "y": 44,
          "value": 0,
          "name": "cell56 ~ gene45"
        },
        {
          "x": 55,
          "y": 45,
          "value": 7,
          "name": "cell56 ~ gene46"
        },
        {
          "x": 55,
          "y": 46,
          "value": 1,
          "name": "cell56 ~ gene47"
        },
        {
          "x": 55,
          "y": 47,
          "value": 0,
          "name": "cell56 ~ gene48"
        },
        {
          "x": 55,
          "y": 48,
          "value": 0,
          "name": "cell56 ~ gene49"
        },
        {
          "x": 55,
          "y": 49,
          "value": 4,
          "name": "cell56 ~ gene50"
        },
        {
          "x": 55,
          "y": 50,
          "value": 5,
          "name": "cell56 ~ gene51"
        },
        {
          "x": 55,
          "y": 51,
          "value": 0,
          "name": "cell56 ~ gene52"
        },
        {
          "x": 55,
          "y": 52,
          "value": 0,
          "name": "cell56 ~ gene53"
        },
        {
          "x": 55,
          "y": 53,
          "value": 0,
          "name": "cell56 ~ gene54"
        },
        {
          "x": 55,
          "y": 54,
          "value": 0,
          "name": "cell56 ~ gene55"
        },
        {
          "x": 55,
          "y": 55,
          "value": 0,
          "name": "cell56 ~ gene56"
        },
        {
          "x": 55,
          "y": 56,
          "value": 0,
          "name": "cell56 ~ gene57"
        },
        {
          "x": 55,
          "y": 57,
          "value": 22,
          "name": "cell56 ~ gene58"
        },
        {
          "x": 55,
          "y": 58,
          "value": 3,
          "name": "cell56 ~ gene59"
        },
        {
          "x": 55,
          "y": 59,
          "value": 0,
          "name": "cell56 ~ gene60"
        },
        {
          "x": 55,
          "y": 60,
          "value": 0,
          "name": "cell56 ~ gene61"
        },
        {
          "x": 55,
          "y": 61,
          "value": 9,
          "name": "cell56 ~ gene62"
        },
        {
          "x": 55,
          "y": 62,
          "value": 0,
          "name": "cell56 ~ gene63"
        },
        {
          "x": 55,
          "y": 63,
          "value": 0,
          "name": "cell56 ~ gene64"
        },
        {
          "x": 55,
          "y": 64,
          "value": 0,
          "name": "cell56 ~ gene65"
        },
        {
          "x": 55,
          "y": 65,
          "value": 4,
          "name": "cell56 ~ gene66"
        },
        {
          "x": 55,
          "y": 66,
          "value": 16,
          "name": "cell56 ~ gene67"
        },
        {
          "x": 55,
          "y": 67,
          "value": 0,
          "name": "cell56 ~ gene68"
        },
        {
          "x": 55,
          "y": 68,
          "value": 5,
          "name": "cell56 ~ gene69"
        },
        {
          "x": 55,
          "y": 69,
          "value": 0,
          "name": "cell56 ~ gene70"
        },
        {
          "x": 55,
          "y": 70,
          "value": 8,
          "name": "cell56 ~ gene71"
        },
        {
          "x": 55,
          "y": 71,
          "value": 28,
          "name": "cell56 ~ gene72"
        },
        {
          "x": 55,
          "y": 72,
          "value": 6,
          "name": "cell56 ~ gene73"
        },
        {
          "x": 55,
          "y": 73,
          "value": 0,
          "name": "cell56 ~ gene74"
        },
        {
          "x": 55,
          "y": 74,
          "value": 0,
          "name": "cell56 ~ gene75"
        },
        {
          "x": 55,
          "y": 75,
          "value": 11,
          "name": "cell56 ~ gene76"
        },
        {
          "x": 55,
          "y": 76,
          "value": 0,
          "name": "cell56 ~ gene77"
        },
        {
          "x": 55,
          "y": 77,
          "value": 0,
          "name": "cell56 ~ gene78"
        },
        {
          "x": 55,
          "y": 78,
          "value": 0,
          "name": "cell56 ~ gene79"
        },
        {
          "x": 55,
          "y": 79,
          "value": 0,
          "name": "cell56 ~ gene80"
        },
        {
          "x": 55,
          "y": 80,
          "value": 0,
          "name": "cell56 ~ gene81"
        },
        {
          "x": 55,
          "y": 81,
          "value": 0,
          "name": "cell56 ~ gene82"
        },
        {
          "x": 55,
          "y": 82,
          "value": 6,
          "name": "cell56 ~ gene83"
        },
        {
          "x": 55,
          "y": 83,
          "value": 12,
          "name": "cell56 ~ gene84"
        },
        {
          "x": 55,
          "y": 84,
          "value": 0,
          "name": "cell56 ~ gene85"
        },
        {
          "x": 55,
          "y": 85,
          "value": 0,
          "name": "cell56 ~ gene86"
        },
        {
          "x": 55,
          "y": 86,
          "value": 0,
          "name": "cell56 ~ gene87"
        },
        {
          "x": 55,
          "y": 87,
          "value": 0,
          "name": "cell56 ~ gene88"
        },
        {
          "x": 55,
          "y": 88,
          "value": 2,
          "name": "cell56 ~ gene89"
        },
        {
          "x": 55,
          "y": 89,
          "value": 0,
          "name": "cell56 ~ gene90"
        },
        {
          "x": 55,
          "y": 90,
          "value": 4,
          "name": "cell56 ~ gene91"
        },
        {
          "x": 55,
          "y": 91,
          "value": 28,
          "name": "cell56 ~ gene92"
        },
        {
          "x": 55,
          "y": 92,
          "value": 0,
          "name": "cell56 ~ gene93"
        },
        {
          "x": 55,
          "y": 93,
          "value": 0,
          "name": "cell56 ~ gene94"
        },
        {
          "x": 55,
          "y": 94,
          "value": 0,
          "name": "cell56 ~ gene95"
        },
        {
          "x": 55,
          "y": 95,
          "value": 0,
          "name": "cell56 ~ gene96"
        },
        {
          "x": 55,
          "y": 96,
          "value": 0,
          "name": "cell56 ~ gene97"
        },
        {
          "x": 55,
          "y": 97,
          "value": 0,
          "name": "cell56 ~ gene98"
        },
        {
          "x": 55,
          "y": 98,
          "value": 17,
          "name": "cell56 ~ gene99"
        },
        {
          "x": 55,
          "y": 99,
          "value": 6,
          "name": "cell56 ~ gene100"
        },
        {
          "x": 56,
          "y": 0,
          "value": 0,
          "name": "cell57 ~ gene1"
        },
        {
          "x": 56,
          "y": 1,
          "value": 12,
          "name": "cell57 ~ gene2"
        },
        {
          "x": 56,
          "y": 2,
          "value": 4,
          "name": "cell57 ~ gene3"
        },
        {
          "x": 56,
          "y": 3,
          "value": 0,
          "name": "cell57 ~ gene4"
        },
        {
          "x": 56,
          "y": 4,
          "value": 0,
          "name": "cell57 ~ gene5"
        },
        {
          "x": 56,
          "y": 5,
          "value": 15,
          "name": "cell57 ~ gene6"
        },
        {
          "x": 56,
          "y": 6,
          "value": 0,
          "name": "cell57 ~ gene7"
        },
        {
          "x": 56,
          "y": 7,
          "value": 0,
          "name": "cell57 ~ gene8"
        },
        {
          "x": 56,
          "y": 8,
          "value": 0,
          "name": "cell57 ~ gene9"
        },
        {
          "x": 56,
          "y": 9,
          "value": 0,
          "name": "cell57 ~ gene10"
        },
        {
          "x": 56,
          "y": 10,
          "value": 0,
          "name": "cell57 ~ gene11"
        },
        {
          "x": 56,
          "y": 11,
          "value": 0,
          "name": "cell57 ~ gene12"
        },
        {
          "x": 56,
          "y": 12,
          "value": 6,
          "name": "cell57 ~ gene13"
        },
        {
          "x": 56,
          "y": 13,
          "value": 0,
          "name": "cell57 ~ gene14"
        },
        {
          "x": 56,
          "y": 14,
          "value": 0,
          "name": "cell57 ~ gene15"
        },
        {
          "x": 56,
          "y": 15,
          "value": 0,
          "name": "cell57 ~ gene16"
        },
        {
          "x": 56,
          "y": 16,
          "value": 0,
          "name": "cell57 ~ gene17"
        },
        {
          "x": 56,
          "y": 17,
          "value": 9,
          "name": "cell57 ~ gene18"
        },
        {
          "x": 56,
          "y": 18,
          "value": 0,
          "name": "cell57 ~ gene19"
        },
        {
          "x": 56,
          "y": 19,
          "value": 0,
          "name": "cell57 ~ gene20"
        },
        {
          "x": 56,
          "y": 20,
          "value": 0,
          "name": "cell57 ~ gene21"
        },
        {
          "x": 56,
          "y": 21,
          "value": 45,
          "name": "cell57 ~ gene22"
        },
        {
          "x": 56,
          "y": 22,
          "value": 10,
          "name": "cell57 ~ gene23"
        },
        {
          "x": 56,
          "y": 23,
          "value": 0,
          "name": "cell57 ~ gene24"
        },
        {
          "x": 56,
          "y": 24,
          "value": 21,
          "name": "cell57 ~ gene25"
        },
        {
          "x": 56,
          "y": 25,
          "value": 11,
          "name": "cell57 ~ gene26"
        },
        {
          "x": 56,
          "y": 26,
          "value": 1,
          "name": "cell57 ~ gene27"
        },
        {
          "x": 56,
          "y": 27,
          "value": 0,
          "name": "cell57 ~ gene28"
        },
        {
          "x": 56,
          "y": 28,
          "value": 0,
          "name": "cell57 ~ gene29"
        },
        {
          "x": 56,
          "y": 29,
          "value": 32,
          "name": "cell57 ~ gene30"
        },
        {
          "x": 56,
          "y": 30,
          "value": 45,
          "name": "cell57 ~ gene31"
        },
        {
          "x": 56,
          "y": 31,
          "value": 15,
          "name": "cell57 ~ gene32"
        },
        {
          "x": 56,
          "y": 32,
          "value": 0,
          "name": "cell57 ~ gene33"
        },
        {
          "x": 56,
          "y": 33,
          "value": 0,
          "name": "cell57 ~ gene34"
        },
        {
          "x": 56,
          "y": 34,
          "value": 3,
          "name": "cell57 ~ gene35"
        },
        {
          "x": 56,
          "y": 35,
          "value": 0,
          "name": "cell57 ~ gene36"
        },
        {
          "x": 56,
          "y": 36,
          "value": 6,
          "name": "cell57 ~ gene37"
        },
        {
          "x": 56,
          "y": 37,
          "value": 16,
          "name": "cell57 ~ gene38"
        },
        {
          "x": 56,
          "y": 38,
          "value": 29,
          "name": "cell57 ~ gene39"
        },
        {
          "x": 56,
          "y": 39,
          "value": 7,
          "name": "cell57 ~ gene40"
        },
        {
          "x": 56,
          "y": 40,
          "value": 6,
          "name": "cell57 ~ gene41"
        },
        {
          "x": 56,
          "y": 41,
          "value": 0,
          "name": "cell57 ~ gene42"
        },
        {
          "x": 56,
          "y": 42,
          "value": 0,
          "name": "cell57 ~ gene43"
        },
        {
          "x": 56,
          "y": 43,
          "value": 0,
          "name": "cell57 ~ gene44"
        },
        {
          "x": 56,
          "y": 44,
          "value": 27,
          "name": "cell57 ~ gene45"
        },
        {
          "x": 56,
          "y": 45,
          "value": 5,
          "name": "cell57 ~ gene46"
        },
        {
          "x": 56,
          "y": 46,
          "value": 13,
          "name": "cell57 ~ gene47"
        },
        {
          "x": 56,
          "y": 47,
          "value": 5,
          "name": "cell57 ~ gene48"
        },
        {
          "x": 56,
          "y": 48,
          "value": 1,
          "name": "cell57 ~ gene49"
        },
        {
          "x": 56,
          "y": 49,
          "value": 0,
          "name": "cell57 ~ gene50"
        },
        {
          "x": 56,
          "y": 50,
          "value": 0,
          "name": "cell57 ~ gene51"
        },
        {
          "x": 56,
          "y": 51,
          "value": 3,
          "name": "cell57 ~ gene52"
        },
        {
          "x": 56,
          "y": 52,
          "value": 14,
          "name": "cell57 ~ gene53"
        },
        {
          "x": 56,
          "y": 53,
          "value": 1,
          "name": "cell57 ~ gene54"
        },
        {
          "x": 56,
          "y": 54,
          "value": 3,
          "name": "cell57 ~ gene55"
        },
        {
          "x": 56,
          "y": 55,
          "value": 0,
          "name": "cell57 ~ gene56"
        },
        {
          "x": 56,
          "y": 56,
          "value": 0,
          "name": "cell57 ~ gene57"
        },
        {
          "x": 56,
          "y": 57,
          "value": 8,
          "name": "cell57 ~ gene58"
        },
        {
          "x": 56,
          "y": 58,
          "value": 6,
          "name": "cell57 ~ gene59"
        },
        {
          "x": 56,
          "y": 59,
          "value": 12,
          "name": "cell57 ~ gene60"
        },
        {
          "x": 56,
          "y": 60,
          "value": 10,
          "name": "cell57 ~ gene61"
        },
        {
          "x": 56,
          "y": 61,
          "value": 9,
          "name": "cell57 ~ gene62"
        },
        {
          "x": 56,
          "y": 62,
          "value": 7,
          "name": "cell57 ~ gene63"
        },
        {
          "x": 56,
          "y": 63,
          "value": 5,
          "name": "cell57 ~ gene64"
        },
        {
          "x": 56,
          "y": 64,
          "value": 11,
          "name": "cell57 ~ gene65"
        },
        {
          "x": 56,
          "y": 65,
          "value": 0,
          "name": "cell57 ~ gene66"
        },
        {
          "x": 56,
          "y": 66,
          "value": 0,
          "name": "cell57 ~ gene67"
        },
        {
          "x": 56,
          "y": 67,
          "value": 0,
          "name": "cell57 ~ gene68"
        },
        {
          "x": 56,
          "y": 68,
          "value": 20,
          "name": "cell57 ~ gene69"
        },
        {
          "x": 56,
          "y": 69,
          "value": 0,
          "name": "cell57 ~ gene70"
        },
        {
          "x": 56,
          "y": 70,
          "value": 5,
          "name": "cell57 ~ gene71"
        },
        {
          "x": 56,
          "y": 71,
          "value": 0,
          "name": "cell57 ~ gene72"
        },
        {
          "x": 56,
          "y": 72,
          "value": 0,
          "name": "cell57 ~ gene73"
        },
        {
          "x": 56,
          "y": 73,
          "value": 13,
          "name": "cell57 ~ gene74"
        },
        {
          "x": 56,
          "y": 74,
          "value": 0,
          "name": "cell57 ~ gene75"
        },
        {
          "x": 56,
          "y": 75,
          "value": 0,
          "name": "cell57 ~ gene76"
        },
        {
          "x": 56,
          "y": 76,
          "value": 0,
          "name": "cell57 ~ gene77"
        },
        {
          "x": 56,
          "y": 77,
          "value": 0,
          "name": "cell57 ~ gene78"
        },
        {
          "x": 56,
          "y": 78,
          "value": 1,
          "name": "cell57 ~ gene79"
        },
        {
          "x": 56,
          "y": 79,
          "value": 0,
          "name": "cell57 ~ gene80"
        },
        {
          "x": 56,
          "y": 80,
          "value": 0,
          "name": "cell57 ~ gene81"
        },
        {
          "x": 56,
          "y": 81,
          "value": 19,
          "name": "cell57 ~ gene82"
        },
        {
          "x": 56,
          "y": 82,
          "value": 0,
          "name": "cell57 ~ gene83"
        },
        {
          "x": 56,
          "y": 83,
          "value": 21,
          "name": "cell57 ~ gene84"
        },
        {
          "x": 56,
          "y": 84,
          "value": 0,
          "name": "cell57 ~ gene85"
        },
        {
          "x": 56,
          "y": 85,
          "value": 6,
          "name": "cell57 ~ gene86"
        },
        {
          "x": 56,
          "y": 86,
          "value": 0,
          "name": "cell57 ~ gene87"
        },
        {
          "x": 56,
          "y": 87,
          "value": 0,
          "name": "cell57 ~ gene88"
        },
        {
          "x": 56,
          "y": 88,
          "value": 16,
          "name": "cell57 ~ gene89"
        },
        {
          "x": 56,
          "y": 89,
          "value": 13,
          "name": "cell57 ~ gene90"
        },
        {
          "x": 56,
          "y": 90,
          "value": 0,
          "name": "cell57 ~ gene91"
        },
        {
          "x": 56,
          "y": 91,
          "value": 0,
          "name": "cell57 ~ gene92"
        },
        {
          "x": 56,
          "y": 92,
          "value": 7,
          "name": "cell57 ~ gene93"
        },
        {
          "x": 56,
          "y": 93,
          "value": 0,
          "name": "cell57 ~ gene94"
        },
        {
          "x": 56,
          "y": 94,
          "value": 14,
          "name": "cell57 ~ gene95"
        },
        {
          "x": 56,
          "y": 95,
          "value": 4,
          "name": "cell57 ~ gene96"
        },
        {
          "x": 56,
          "y": 96,
          "value": 15,
          "name": "cell57 ~ gene97"
        },
        {
          "x": 56,
          "y": 97,
          "value": 30,
          "name": "cell57 ~ gene98"
        },
        {
          "x": 56,
          "y": 98,
          "value": 12,
          "name": "cell57 ~ gene99"
        },
        {
          "x": 56,
          "y": 99,
          "value": 0,
          "name": "cell57 ~ gene100"
        },
        {
          "x": 57,
          "y": 0,
          "value": 0,
          "name": "cell58 ~ gene1"
        },
        {
          "x": 57,
          "y": 1,
          "value": 0,
          "name": "cell58 ~ gene2"
        },
        {
          "x": 57,
          "y": 2,
          "value": 13,
          "name": "cell58 ~ gene3"
        },
        {
          "x": 57,
          "y": 3,
          "value": 0,
          "name": "cell58 ~ gene4"
        },
        {
          "x": 57,
          "y": 4,
          "value": 11,
          "name": "cell58 ~ gene5"
        },
        {
          "x": 57,
          "y": 5,
          "value": 8,
          "name": "cell58 ~ gene6"
        },
        {
          "x": 57,
          "y": 6,
          "value": 11,
          "name": "cell58 ~ gene7"
        },
        {
          "x": 57,
          "y": 7,
          "value": 0,
          "name": "cell58 ~ gene8"
        },
        {
          "x": 57,
          "y": 8,
          "value": 16,
          "name": "cell58 ~ gene9"
        },
        {
          "x": 57,
          "y": 9,
          "value": 0,
          "name": "cell58 ~ gene10"
        },
        {
          "x": 57,
          "y": 10,
          "value": 0,
          "name": "cell58 ~ gene11"
        },
        {
          "x": 57,
          "y": 11,
          "value": 0,
          "name": "cell58 ~ gene12"
        },
        {
          "x": 57,
          "y": 12,
          "value": 13,
          "name": "cell58 ~ gene13"
        },
        {
          "x": 57,
          "y": 13,
          "value": 0,
          "name": "cell58 ~ gene14"
        },
        {
          "x": 57,
          "y": 14,
          "value": 0,
          "name": "cell58 ~ gene15"
        },
        {
          "x": 57,
          "y": 15,
          "value": 4,
          "name": "cell58 ~ gene16"
        },
        {
          "x": 57,
          "y": 16,
          "value": 9,
          "name": "cell58 ~ gene17"
        },
        {
          "x": 57,
          "y": 17,
          "value": 0,
          "name": "cell58 ~ gene18"
        },
        {
          "x": 57,
          "y": 18,
          "value": 0,
          "name": "cell58 ~ gene19"
        },
        {
          "x": 57,
          "y": 19,
          "value": 15,
          "name": "cell58 ~ gene20"
        },
        {
          "x": 57,
          "y": 20,
          "value": 0,
          "name": "cell58 ~ gene21"
        },
        {
          "x": 57,
          "y": 21,
          "value": 13,
          "name": "cell58 ~ gene22"
        },
        {
          "x": 57,
          "y": 22,
          "value": 1,
          "name": "cell58 ~ gene23"
        },
        {
          "x": 57,
          "y": 23,
          "value": 0,
          "name": "cell58 ~ gene24"
        },
        {
          "x": 57,
          "y": 24,
          "value": 3,
          "name": "cell58 ~ gene25"
        },
        {
          "x": 57,
          "y": 25,
          "value": 2,
          "name": "cell58 ~ gene26"
        },
        {
          "x": 57,
          "y": 26,
          "value": 0,
          "name": "cell58 ~ gene27"
        },
        {
          "x": 57,
          "y": 27,
          "value": 2,
          "name": "cell58 ~ gene28"
        },
        {
          "x": 57,
          "y": 28,
          "value": 0,
          "name": "cell58 ~ gene29"
        },
        {
          "x": 57,
          "y": 29,
          "value": 30,
          "name": "cell58 ~ gene30"
        },
        {
          "x": 57,
          "y": 30,
          "value": 40,
          "name": "cell58 ~ gene31"
        },
        {
          "x": 57,
          "y": 31,
          "value": 16,
          "name": "cell58 ~ gene32"
        },
        {
          "x": 57,
          "y": 32,
          "value": 0,
          "name": "cell58 ~ gene33"
        },
        {
          "x": 57,
          "y": 33,
          "value": 5,
          "name": "cell58 ~ gene34"
        },
        {
          "x": 57,
          "y": 34,
          "value": 23,
          "name": "cell58 ~ gene35"
        },
        {
          "x": 57,
          "y": 35,
          "value": 0,
          "name": "cell58 ~ gene36"
        },
        {
          "x": 57,
          "y": 36,
          "value": 32,
          "name": "cell58 ~ gene37"
        },
        {
          "x": 57,
          "y": 37,
          "value": 13,
          "name": "cell58 ~ gene38"
        },
        {
          "x": 57,
          "y": 38,
          "value": 1,
          "name": "cell58 ~ gene39"
        },
        {
          "x": 57,
          "y": 39,
          "value": 15,
          "name": "cell58 ~ gene40"
        },
        {
          "x": 57,
          "y": 40,
          "value": 10,
          "name": "cell58 ~ gene41"
        },
        {
          "x": 57,
          "y": 41,
          "value": 13,
          "name": "cell58 ~ gene42"
        },
        {
          "x": 57,
          "y": 42,
          "value": 13,
          "name": "cell58 ~ gene43"
        },
        {
          "x": 57,
          "y": 43,
          "value": 0,
          "name": "cell58 ~ gene44"
        },
        {
          "x": 57,
          "y": 44,
          "value": 8,
          "name": "cell58 ~ gene45"
        },
        {
          "x": 57,
          "y": 45,
          "value": 19,
          "name": "cell58 ~ gene46"
        },
        {
          "x": 57,
          "y": 46,
          "value": 0,
          "name": "cell58 ~ gene47"
        },
        {
          "x": 57,
          "y": 47,
          "value": 1,
          "name": "cell58 ~ gene48"
        },
        {
          "x": 57,
          "y": 48,
          "value": 2,
          "name": "cell58 ~ gene49"
        },
        {
          "x": 57,
          "y": 49,
          "value": 7,
          "name": "cell58 ~ gene50"
        },
        {
          "x": 57,
          "y": 50,
          "value": 6,
          "name": "cell58 ~ gene51"
        },
        {
          "x": 57,
          "y": 51,
          "value": 4,
          "name": "cell58 ~ gene52"
        },
        {
          "x": 57,
          "y": 52,
          "value": 0,
          "name": "cell58 ~ gene53"
        },
        {
          "x": 57,
          "y": 53,
          "value": 4,
          "name": "cell58 ~ gene54"
        },
        {
          "x": 57,
          "y": 54,
          "value": 0,
          "name": "cell58 ~ gene55"
        },
        {
          "x": 57,
          "y": 55,
          "value": 0,
          "name": "cell58 ~ gene56"
        },
        {
          "x": 57,
          "y": 56,
          "value": 0,
          "name": "cell58 ~ gene57"
        },
        {
          "x": 57,
          "y": 57,
          "value": 0,
          "name": "cell58 ~ gene58"
        },
        {
          "x": 57,
          "y": 58,
          "value": 0,
          "name": "cell58 ~ gene59"
        },
        {
          "x": 57,
          "y": 59,
          "value": 14,
          "name": "cell58 ~ gene60"
        },
        {
          "x": 57,
          "y": 60,
          "value": 0,
          "name": "cell58 ~ gene61"
        },
        {
          "x": 57,
          "y": 61,
          "value": 0,
          "name": "cell58 ~ gene62"
        },
        {
          "x": 57,
          "y": 62,
          "value": 9,
          "name": "cell58 ~ gene63"
        },
        {
          "x": 57,
          "y": 63,
          "value": 0,
          "name": "cell58 ~ gene64"
        },
        {
          "x": 57,
          "y": 64,
          "value": 0,
          "name": "cell58 ~ gene65"
        },
        {
          "x": 57,
          "y": 65,
          "value": 0,
          "name": "cell58 ~ gene66"
        },
        {
          "x": 57,
          "y": 66,
          "value": 4,
          "name": "cell58 ~ gene67"
        },
        {
          "x": 57,
          "y": 67,
          "value": 3,
          "name": "cell58 ~ gene68"
        },
        {
          "x": 57,
          "y": 68,
          "value": 0,
          "name": "cell58 ~ gene69"
        },
        {
          "x": 57,
          "y": 69,
          "value": 0,
          "name": "cell58 ~ gene70"
        },
        {
          "x": 57,
          "y": 70,
          "value": 0,
          "name": "cell58 ~ gene71"
        },
        {
          "x": 57,
          "y": 71,
          "value": 11,
          "name": "cell58 ~ gene72"
        },
        {
          "x": 57,
          "y": 72,
          "value": 0,
          "name": "cell58 ~ gene73"
        },
        {
          "x": 57,
          "y": 73,
          "value": 13,
          "name": "cell58 ~ gene74"
        },
        {
          "x": 57,
          "y": 74,
          "value": 0,
          "name": "cell58 ~ gene75"
        },
        {
          "x": 57,
          "y": 75,
          "value": 0,
          "name": "cell58 ~ gene76"
        },
        {
          "x": 57,
          "y": 76,
          "value": 24,
          "name": "cell58 ~ gene77"
        },
        {
          "x": 57,
          "y": 77,
          "value": 0,
          "name": "cell58 ~ gene78"
        },
        {
          "x": 57,
          "y": 78,
          "value": 0,
          "name": "cell58 ~ gene79"
        },
        {
          "x": 57,
          "y": 79,
          "value": 7,
          "name": "cell58 ~ gene80"
        },
        {
          "x": 57,
          "y": 80,
          "value": 11,
          "name": "cell58 ~ gene81"
        },
        {
          "x": 57,
          "y": 81,
          "value": 0,
          "name": "cell58 ~ gene82"
        },
        {
          "x": 57,
          "y": 82,
          "value": 0,
          "name": "cell58 ~ gene83"
        },
        {
          "x": 57,
          "y": 83,
          "value": 0,
          "name": "cell58 ~ gene84"
        },
        {
          "x": 57,
          "y": 84,
          "value": 1,
          "name": "cell58 ~ gene85"
        },
        {
          "x": 57,
          "y": 85,
          "value": 9,
          "name": "cell58 ~ gene86"
        },
        {
          "x": 57,
          "y": 86,
          "value": 13,
          "name": "cell58 ~ gene87"
        },
        {
          "x": 57,
          "y": 87,
          "value": 12,
          "name": "cell58 ~ gene88"
        },
        {
          "x": 57,
          "y": 88,
          "value": 6,
          "name": "cell58 ~ gene89"
        },
        {
          "x": 57,
          "y": 89,
          "value": 9,
          "name": "cell58 ~ gene90"
        },
        {
          "x": 57,
          "y": 90,
          "value": 0,
          "name": "cell58 ~ gene91"
        },
        {
          "x": 57,
          "y": 91,
          "value": 0,
          "name": "cell58 ~ gene92"
        },
        {
          "x": 57,
          "y": 92,
          "value": 5,
          "name": "cell58 ~ gene93"
        },
        {
          "x": 57,
          "y": 93,
          "value": 0,
          "name": "cell58 ~ gene94"
        },
        {
          "x": 57,
          "y": 94,
          "value": 5,
          "name": "cell58 ~ gene95"
        },
        {
          "x": 57,
          "y": 95,
          "value": 0,
          "name": "cell58 ~ gene96"
        },
        {
          "x": 57,
          "y": 96,
          "value": 3,
          "name": "cell58 ~ gene97"
        },
        {
          "x": 57,
          "y": 97,
          "value": 0,
          "name": "cell58 ~ gene98"
        },
        {
          "x": 57,
          "y": 98,
          "value": 6,
          "name": "cell58 ~ gene99"
        },
        {
          "x": 57,
          "y": 99,
          "value": 8,
          "name": "cell58 ~ gene100"
        },
        {
          "x": 58,
          "y": 0,
          "value": 0,
          "name": "cell59 ~ gene1"
        },
        {
          "x": 58,
          "y": 1,
          "value": 0,
          "name": "cell59 ~ gene2"
        },
        {
          "x": 58,
          "y": 2,
          "value": 0,
          "name": "cell59 ~ gene3"
        },
        {
          "x": 58,
          "y": 3,
          "value": 0,
          "name": "cell59 ~ gene4"
        },
        {
          "x": 58,
          "y": 4,
          "value": 3,
          "name": "cell59 ~ gene5"
        },
        {
          "x": 58,
          "y": 5,
          "value": 0,
          "name": "cell59 ~ gene6"
        },
        {
          "x": 58,
          "y": 6,
          "value": 0,
          "name": "cell59 ~ gene7"
        },
        {
          "x": 58,
          "y": 7,
          "value": 0,
          "name": "cell59 ~ gene8"
        },
        {
          "x": 58,
          "y": 8,
          "value": 2,
          "name": "cell59 ~ gene9"
        },
        {
          "x": 58,
          "y": 9,
          "value": 4,
          "name": "cell59 ~ gene10"
        },
        {
          "x": 58,
          "y": 10,
          "value": 8,
          "name": "cell59 ~ gene11"
        },
        {
          "x": 58,
          "y": 11,
          "value": 4,
          "name": "cell59 ~ gene12"
        },
        {
          "x": 58,
          "y": 12,
          "value": 6,
          "name": "cell59 ~ gene13"
        },
        {
          "x": 58,
          "y": 13,
          "value": 0,
          "name": "cell59 ~ gene14"
        },
        {
          "x": 58,
          "y": 14,
          "value": 8,
          "name": "cell59 ~ gene15"
        },
        {
          "x": 58,
          "y": 15,
          "value": 0,
          "name": "cell59 ~ gene16"
        },
        {
          "x": 58,
          "y": 16,
          "value": 6,
          "name": "cell59 ~ gene17"
        },
        {
          "x": 58,
          "y": 17,
          "value": 0,
          "name": "cell59 ~ gene18"
        },
        {
          "x": 58,
          "y": 18,
          "value": 15,
          "name": "cell59 ~ gene19"
        },
        {
          "x": 58,
          "y": 19,
          "value": 11,
          "name": "cell59 ~ gene20"
        },
        {
          "x": 58,
          "y": 20,
          "value": 0,
          "name": "cell59 ~ gene21"
        },
        {
          "x": 58,
          "y": 21,
          "value": 17,
          "name": "cell59 ~ gene22"
        },
        {
          "x": 58,
          "y": 22,
          "value": 15,
          "name": "cell59 ~ gene23"
        },
        {
          "x": 58,
          "y": 23,
          "value": 0,
          "name": "cell59 ~ gene24"
        },
        {
          "x": 58,
          "y": 24,
          "value": 0,
          "name": "cell59 ~ gene25"
        },
        {
          "x": 58,
          "y": 25,
          "value": 4,
          "name": "cell59 ~ gene26"
        },
        {
          "x": 58,
          "y": 26,
          "value": 0,
          "name": "cell59 ~ gene27"
        },
        {
          "x": 58,
          "y": 27,
          "value": 9,
          "name": "cell59 ~ gene28"
        },
        {
          "x": 58,
          "y": 28,
          "value": 0,
          "name": "cell59 ~ gene29"
        },
        {
          "x": 58,
          "y": 29,
          "value": 40,
          "name": "cell59 ~ gene30"
        },
        {
          "x": 58,
          "y": 30,
          "value": 24,
          "name": "cell59 ~ gene31"
        },
        {
          "x": 58,
          "y": 31,
          "value": 0,
          "name": "cell59 ~ gene32"
        },
        {
          "x": 58,
          "y": 32,
          "value": 0,
          "name": "cell59 ~ gene33"
        },
        {
          "x": 58,
          "y": 33,
          "value": 18,
          "name": "cell59 ~ gene34"
        },
        {
          "x": 58,
          "y": 34,
          "value": 11,
          "name": "cell59 ~ gene35"
        },
        {
          "x": 58,
          "y": 35,
          "value": 0,
          "name": "cell59 ~ gene36"
        },
        {
          "x": 58,
          "y": 36,
          "value": 10,
          "name": "cell59 ~ gene37"
        },
        {
          "x": 58,
          "y": 37,
          "value": 4,
          "name": "cell59 ~ gene38"
        },
        {
          "x": 58,
          "y": 38,
          "value": 0,
          "name": "cell59 ~ gene39"
        },
        {
          "x": 58,
          "y": 39,
          "value": 13,
          "name": "cell59 ~ gene40"
        },
        {
          "x": 58,
          "y": 40,
          "value": 0,
          "name": "cell59 ~ gene41"
        },
        {
          "x": 58,
          "y": 41,
          "value": 5,
          "name": "cell59 ~ gene42"
        },
        {
          "x": 58,
          "y": 42,
          "value": 0,
          "name": "cell59 ~ gene43"
        },
        {
          "x": 58,
          "y": 43,
          "value": 9,
          "name": "cell59 ~ gene44"
        },
        {
          "x": 58,
          "y": 44,
          "value": 0,
          "name": "cell59 ~ gene45"
        },
        {
          "x": 58,
          "y": 45,
          "value": 0,
          "name": "cell59 ~ gene46"
        },
        {
          "x": 58,
          "y": 46,
          "value": 0,
          "name": "cell59 ~ gene47"
        },
        {
          "x": 58,
          "y": 47,
          "value": 5,
          "name": "cell59 ~ gene48"
        },
        {
          "x": 58,
          "y": 48,
          "value": 8,
          "name": "cell59 ~ gene49"
        },
        {
          "x": 58,
          "y": 49,
          "value": 0,
          "name": "cell59 ~ gene50"
        },
        {
          "x": 58,
          "y": 50,
          "value": 0,
          "name": "cell59 ~ gene51"
        },
        {
          "x": 58,
          "y": 51,
          "value": 3,
          "name": "cell59 ~ gene52"
        },
        {
          "x": 58,
          "y": 52,
          "value": 0,
          "name": "cell59 ~ gene53"
        },
        {
          "x": 58,
          "y": 53,
          "value": 0,
          "name": "cell59 ~ gene54"
        },
        {
          "x": 58,
          "y": 54,
          "value": 2,
          "name": "cell59 ~ gene55"
        },
        {
          "x": 58,
          "y": 55,
          "value": 0,
          "name": "cell59 ~ gene56"
        },
        {
          "x": 58,
          "y": 56,
          "value": 2,
          "name": "cell59 ~ gene57"
        },
        {
          "x": 58,
          "y": 57,
          "value": 1,
          "name": "cell59 ~ gene58"
        },
        {
          "x": 58,
          "y": 58,
          "value": 0,
          "name": "cell59 ~ gene59"
        },
        {
          "x": 58,
          "y": 59,
          "value": 0,
          "name": "cell59 ~ gene60"
        },
        {
          "x": 58,
          "y": 60,
          "value": 0,
          "name": "cell59 ~ gene61"
        },
        {
          "x": 58,
          "y": 61,
          "value": 11,
          "name": "cell59 ~ gene62"
        },
        {
          "x": 58,
          "y": 62,
          "value": 8,
          "name": "cell59 ~ gene63"
        },
        {
          "x": 58,
          "y": 63,
          "value": 1,
          "name": "cell59 ~ gene64"
        },
        {
          "x": 58,
          "y": 64,
          "value": 25,
          "name": "cell59 ~ gene65"
        },
        {
          "x": 58,
          "y": 65,
          "value": 6,
          "name": "cell59 ~ gene66"
        },
        {
          "x": 58,
          "y": 66,
          "value": 3,
          "name": "cell59 ~ gene67"
        },
        {
          "x": 58,
          "y": 67,
          "value": 27,
          "name": "cell59 ~ gene68"
        },
        {
          "x": 58,
          "y": 68,
          "value": 0,
          "name": "cell59 ~ gene69"
        },
        {
          "x": 58,
          "y": 69,
          "value": 0,
          "name": "cell59 ~ gene70"
        },
        {
          "x": 58,
          "y": 70,
          "value": 0,
          "name": "cell59 ~ gene71"
        },
        {
          "x": 58,
          "y": 71,
          "value": 0,
          "name": "cell59 ~ gene72"
        },
        {
          "x": 58,
          "y": 72,
          "value": 12,
          "name": "cell59 ~ gene73"
        },
        {
          "x": 58,
          "y": 73,
          "value": 0,
          "name": "cell59 ~ gene74"
        },
        {
          "x": 58,
          "y": 74,
          "value": 0,
          "name": "cell59 ~ gene75"
        },
        {
          "x": 58,
          "y": 75,
          "value": 7,
          "name": "cell59 ~ gene76"
        },
        {
          "x": 58,
          "y": 76,
          "value": 11,
          "name": "cell59 ~ gene77"
        },
        {
          "x": 58,
          "y": 77,
          "value": 0,
          "name": "cell59 ~ gene78"
        },
        {
          "x": 58,
          "y": 78,
          "value": 12,
          "name": "cell59 ~ gene79"
        },
        {
          "x": 58,
          "y": 79,
          "value": 10,
          "name": "cell59 ~ gene80"
        },
        {
          "x": 58,
          "y": 80,
          "value": 6,
          "name": "cell59 ~ gene81"
        },
        {
          "x": 58,
          "y": 81,
          "value": 8,
          "name": "cell59 ~ gene82"
        },
        {
          "x": 58,
          "y": 82,
          "value": 0,
          "name": "cell59 ~ gene83"
        },
        {
          "x": 58,
          "y": 83,
          "value": 9,
          "name": "cell59 ~ gene84"
        },
        {
          "x": 58,
          "y": 84,
          "value": 0,
          "name": "cell59 ~ gene85"
        },
        {
          "x": 58,
          "y": 85,
          "value": 0,
          "name": "cell59 ~ gene86"
        },
        {
          "x": 58,
          "y": 86,
          "value": 3,
          "name": "cell59 ~ gene87"
        },
        {
          "x": 58,
          "y": 87,
          "value": 0,
          "name": "cell59 ~ gene88"
        },
        {
          "x": 58,
          "y": 88,
          "value": 0,
          "name": "cell59 ~ gene89"
        },
        {
          "x": 58,
          "y": 89,
          "value": 0,
          "name": "cell59 ~ gene90"
        },
        {
          "x": 58,
          "y": 90,
          "value": 0,
          "name": "cell59 ~ gene91"
        },
        {
          "x": 58,
          "y": 91,
          "value": 8,
          "name": "cell59 ~ gene92"
        },
        {
          "x": 58,
          "y": 92,
          "value": 2,
          "name": "cell59 ~ gene93"
        },
        {
          "x": 58,
          "y": 93,
          "value": 5,
          "name": "cell59 ~ gene94"
        },
        {
          "x": 58,
          "y": 94,
          "value": 0,
          "name": "cell59 ~ gene95"
        },
        {
          "x": 58,
          "y": 95,
          "value": 0,
          "name": "cell59 ~ gene96"
        },
        {
          "x": 58,
          "y": 96,
          "value": 3,
          "name": "cell59 ~ gene97"
        },
        {
          "x": 58,
          "y": 97,
          "value": 4,
          "name": "cell59 ~ gene98"
        },
        {
          "x": 58,
          "y": 98,
          "value": 5,
          "name": "cell59 ~ gene99"
        },
        {
          "x": 58,
          "y": 99,
          "value": 15,
          "name": "cell59 ~ gene100"
        },
        {
          "x": 59,
          "y": 0,
          "value": 9,
          "name": "cell60 ~ gene1"
        },
        {
          "x": 59,
          "y": 1,
          "value": 15,
          "name": "cell60 ~ gene2"
        },
        {
          "x": 59,
          "y": 2,
          "value": 14,
          "name": "cell60 ~ gene3"
        },
        {
          "x": 59,
          "y": 3,
          "value": 0,
          "name": "cell60 ~ gene4"
        },
        {
          "x": 59,
          "y": 4,
          "value": 13,
          "name": "cell60 ~ gene5"
        },
        {
          "x": 59,
          "y": 5,
          "value": 0,
          "name": "cell60 ~ gene6"
        },
        {
          "x": 59,
          "y": 6,
          "value": 0,
          "name": "cell60 ~ gene7"
        },
        {
          "x": 59,
          "y": 7,
          "value": 2,
          "name": "cell60 ~ gene8"
        },
        {
          "x": 59,
          "y": 8,
          "value": 16,
          "name": "cell60 ~ gene9"
        },
        {
          "x": 59,
          "y": 9,
          "value": 0,
          "name": "cell60 ~ gene10"
        },
        {
          "x": 59,
          "y": 10,
          "value": 3,
          "name": "cell60 ~ gene11"
        },
        {
          "x": 59,
          "y": 11,
          "value": 0,
          "name": "cell60 ~ gene12"
        },
        {
          "x": 59,
          "y": 12,
          "value": 0,
          "name": "cell60 ~ gene13"
        },
        {
          "x": 59,
          "y": 13,
          "value": 0,
          "name": "cell60 ~ gene14"
        },
        {
          "x": 59,
          "y": 14,
          "value": 7,
          "name": "cell60 ~ gene15"
        },
        {
          "x": 59,
          "y": 15,
          "value": 28,
          "name": "cell60 ~ gene16"
        },
        {
          "x": 59,
          "y": 16,
          "value": 5,
          "name": "cell60 ~ gene17"
        },
        {
          "x": 59,
          "y": 17,
          "value": 8,
          "name": "cell60 ~ gene18"
        },
        {
          "x": 59,
          "y": 18,
          "value": 4,
          "name": "cell60 ~ gene19"
        },
        {
          "x": 59,
          "y": 19,
          "value": 10,
          "name": "cell60 ~ gene20"
        },
        {
          "x": 59,
          "y": 20,
          "value": 11,
          "name": "cell60 ~ gene21"
        },
        {
          "x": 59,
          "y": 21,
          "value": 28,
          "name": "cell60 ~ gene22"
        },
        {
          "x": 59,
          "y": 22,
          "value": 8,
          "name": "cell60 ~ gene23"
        },
        {
          "x": 59,
          "y": 23,
          "value": 0,
          "name": "cell60 ~ gene24"
        },
        {
          "x": 59,
          "y": 24,
          "value": 14,
          "name": "cell60 ~ gene25"
        },
        {
          "x": 59,
          "y": 25,
          "value": 3,
          "name": "cell60 ~ gene26"
        },
        {
          "x": 59,
          "y": 26,
          "value": 16,
          "name": "cell60 ~ gene27"
        },
        {
          "x": 59,
          "y": 27,
          "value": 0,
          "name": "cell60 ~ gene28"
        },
        {
          "x": 59,
          "y": 28,
          "value": 10,
          "name": "cell60 ~ gene29"
        },
        {
          "x": 59,
          "y": 29,
          "value": 37,
          "name": "cell60 ~ gene30"
        },
        {
          "x": 59,
          "y": 30,
          "value": 27,
          "name": "cell60 ~ gene31"
        },
        {
          "x": 59,
          "y": 31,
          "value": 0,
          "name": "cell60 ~ gene32"
        },
        {
          "x": 59,
          "y": 32,
          "value": 0,
          "name": "cell60 ~ gene33"
        },
        {
          "x": 59,
          "y": 33,
          "value": 10,
          "name": "cell60 ~ gene34"
        },
        {
          "x": 59,
          "y": 34,
          "value": 0,
          "name": "cell60 ~ gene35"
        },
        {
          "x": 59,
          "y": 35,
          "value": 6,
          "name": "cell60 ~ gene36"
        },
        {
          "x": 59,
          "y": 36,
          "value": 17,
          "name": "cell60 ~ gene37"
        },
        {
          "x": 59,
          "y": 37,
          "value": 20,
          "name": "cell60 ~ gene38"
        },
        {
          "x": 59,
          "y": 38,
          "value": 16,
          "name": "cell60 ~ gene39"
        },
        {
          "x": 59,
          "y": 39,
          "value": 28,
          "name": "cell60 ~ gene40"
        },
        {
          "x": 59,
          "y": 40,
          "value": 9,
          "name": "cell60 ~ gene41"
        },
        {
          "x": 59,
          "y": 41,
          "value": 7,
          "name": "cell60 ~ gene42"
        },
        {
          "x": 59,
          "y": 42,
          "value": 0,
          "name": "cell60 ~ gene43"
        },
        {
          "x": 59,
          "y": 43,
          "value": 6,
          "name": "cell60 ~ gene44"
        },
        {
          "x": 59,
          "y": 44,
          "value": 18,
          "name": "cell60 ~ gene45"
        },
        {
          "x": 59,
          "y": 45,
          "value": 2,
          "name": "cell60 ~ gene46"
        },
        {
          "x": 59,
          "y": 46,
          "value": 4,
          "name": "cell60 ~ gene47"
        },
        {
          "x": 59,
          "y": 47,
          "value": 5,
          "name": "cell60 ~ gene48"
        },
        {
          "x": 59,
          "y": 48,
          "value": 3,
          "name": "cell60 ~ gene49"
        },
        {
          "x": 59,
          "y": 49,
          "value": 6,
          "name": "cell60 ~ gene50"
        },
        {
          "x": 59,
          "y": 50,
          "value": 9,
          "name": "cell60 ~ gene51"
        },
        {
          "x": 59,
          "y": 51,
          "value": 3,
          "name": "cell60 ~ gene52"
        },
        {
          "x": 59,
          "y": 52,
          "value": 0,
          "name": "cell60 ~ gene53"
        },
        {
          "x": 59,
          "y": 53,
          "value": 0,
          "name": "cell60 ~ gene54"
        },
        {
          "x": 59,
          "y": 54,
          "value": 4,
          "name": "cell60 ~ gene55"
        },
        {
          "x": 59,
          "y": 55,
          "value": 0,
          "name": "cell60 ~ gene56"
        },
        {
          "x": 59,
          "y": 56,
          "value": 0,
          "name": "cell60 ~ gene57"
        },
        {
          "x": 59,
          "y": 57,
          "value": 5,
          "name": "cell60 ~ gene58"
        },
        {
          "x": 59,
          "y": 58,
          "value": 0,
          "name": "cell60 ~ gene59"
        },
        {
          "x": 59,
          "y": 59,
          "value": 0,
          "name": "cell60 ~ gene60"
        },
        {
          "x": 59,
          "y": 60,
          "value": 0,
          "name": "cell60 ~ gene61"
        },
        {
          "x": 59,
          "y": 61,
          "value": 11,
          "name": "cell60 ~ gene62"
        },
        {
          "x": 59,
          "y": 62,
          "value": 9,
          "name": "cell60 ~ gene63"
        },
        {
          "x": 59,
          "y": 63,
          "value": 23,
          "name": "cell60 ~ gene64"
        },
        {
          "x": 59,
          "y": 64,
          "value": 21,
          "name": "cell60 ~ gene65"
        },
        {
          "x": 59,
          "y": 65,
          "value": 13,
          "name": "cell60 ~ gene66"
        },
        {
          "x": 59,
          "y": 66,
          "value": 22,
          "name": "cell60 ~ gene67"
        },
        {
          "x": 59,
          "y": 67,
          "value": 15,
          "name": "cell60 ~ gene68"
        },
        {
          "x": 59,
          "y": 68,
          "value": 0,
          "name": "cell60 ~ gene69"
        },
        {
          "x": 59,
          "y": 69,
          "value": 14,
          "name": "cell60 ~ gene70"
        },
        {
          "x": 59,
          "y": 70,
          "value": 0,
          "name": "cell60 ~ gene71"
        },
        {
          "x": 59,
          "y": 71,
          "value": 3,
          "name": "cell60 ~ gene72"
        },
        {
          "x": 59,
          "y": 72,
          "value": 0,
          "name": "cell60 ~ gene73"
        },
        {
          "x": 59,
          "y": 73,
          "value": 0,
          "name": "cell60 ~ gene74"
        },
        {
          "x": 59,
          "y": 74,
          "value": 0,
          "name": "cell60 ~ gene75"
        },
        {
          "x": 59,
          "y": 75,
          "value": 10,
          "name": "cell60 ~ gene76"
        },
        {
          "x": 59,
          "y": 76,
          "value": 0,
          "name": "cell60 ~ gene77"
        },
        {
          "x": 59,
          "y": 77,
          "value": 0,
          "name": "cell60 ~ gene78"
        },
        {
          "x": 59,
          "y": 78,
          "value": 13,
          "name": "cell60 ~ gene79"
        },
        {
          "x": 59,
          "y": 79,
          "value": 0,
          "name": "cell60 ~ gene80"
        },
        {
          "x": 59,
          "y": 80,
          "value": 0,
          "name": "cell60 ~ gene81"
        },
        {
          "x": 59,
          "y": 81,
          "value": 0,
          "name": "cell60 ~ gene82"
        },
        {
          "x": 59,
          "y": 82,
          "value": 2,
          "name": "cell60 ~ gene83"
        },
        {
          "x": 59,
          "y": 83,
          "value": 0,
          "name": "cell60 ~ gene84"
        },
        {
          "x": 59,
          "y": 84,
          "value": 1,
          "name": "cell60 ~ gene85"
        },
        {
          "x": 59,
          "y": 85,
          "value": 9,
          "name": "cell60 ~ gene86"
        },
        {
          "x": 59,
          "y": 86,
          "value": 2,
          "name": "cell60 ~ gene87"
        },
        {
          "x": 59,
          "y": 87,
          "value": 0,
          "name": "cell60 ~ gene88"
        },
        {
          "x": 59,
          "y": 88,
          "value": 10,
          "name": "cell60 ~ gene89"
        },
        {
          "x": 59,
          "y": 89,
          "value": 11,
          "name": "cell60 ~ gene90"
        },
        {
          "x": 59,
          "y": 90,
          "value": 7,
          "name": "cell60 ~ gene91"
        },
        {
          "x": 59,
          "y": 91,
          "value": 0,
          "name": "cell60 ~ gene92"
        },
        {
          "x": 59,
          "y": 92,
          "value": 0,
          "name": "cell60 ~ gene93"
        },
        {
          "x": 59,
          "y": 93,
          "value": 8,
          "name": "cell60 ~ gene94"
        },
        {
          "x": 59,
          "y": 94,
          "value": 0,
          "name": "cell60 ~ gene95"
        },
        {
          "x": 59,
          "y": 95,
          "value": 17,
          "name": "cell60 ~ gene96"
        },
        {
          "x": 59,
          "y": 96,
          "value": 0,
          "name": "cell60 ~ gene97"
        },
        {
          "x": 59,
          "y": 97,
          "value": 11,
          "name": "cell60 ~ gene98"
        },
        {
          "x": 59,
          "y": 98,
          "value": 0,
          "name": "cell60 ~ gene99"
        },
        {
          "x": 59,
          "y": 99,
          "value": 0,
          "name": "cell60 ~ gene100"
        },
        {
          "x": 60,
          "y": 0,
          "value": 0,
          "name": "cell61 ~ gene1"
        },
        {
          "x": 60,
          "y": 1,
          "value": 0,
          "name": "cell61 ~ gene2"
        },
        {
          "x": 60,
          "y": 2,
          "value": 8,
          "name": "cell61 ~ gene3"
        },
        {
          "x": 60,
          "y": 3,
          "value": 5,
          "name": "cell61 ~ gene4"
        },
        {
          "x": 60,
          "y": 4,
          "value": 19,
          "name": "cell61 ~ gene5"
        },
        {
          "x": 60,
          "y": 5,
          "value": 4,
          "name": "cell61 ~ gene6"
        },
        {
          "x": 60,
          "y": 6,
          "value": 3,
          "name": "cell61 ~ gene7"
        },
        {
          "x": 60,
          "y": 7,
          "value": 0,
          "name": "cell61 ~ gene8"
        },
        {
          "x": 60,
          "y": 8,
          "value": 14,
          "name": "cell61 ~ gene9"
        },
        {
          "x": 60,
          "y": 9,
          "value": 7,
          "name": "cell61 ~ gene10"
        },
        {
          "x": 60,
          "y": 10,
          "value": 14,
          "name": "cell61 ~ gene11"
        },
        {
          "x": 60,
          "y": 11,
          "value": 1,
          "name": "cell61 ~ gene12"
        },
        {
          "x": 60,
          "y": 12,
          "value": 12,
          "name": "cell61 ~ gene13"
        },
        {
          "x": 60,
          "y": 13,
          "value": 14,
          "name": "cell61 ~ gene14"
        },
        {
          "x": 60,
          "y": 14,
          "value": 13,
          "name": "cell61 ~ gene15"
        },
        {
          "x": 60,
          "y": 15,
          "value": 9,
          "name": "cell61 ~ gene16"
        },
        {
          "x": 60,
          "y": 16,
          "value": 12,
          "name": "cell61 ~ gene17"
        },
        {
          "x": 60,
          "y": 17,
          "value": 0,
          "name": "cell61 ~ gene18"
        },
        {
          "x": 60,
          "y": 18,
          "value": 0,
          "name": "cell61 ~ gene19"
        },
        {
          "x": 60,
          "y": 19,
          "value": 0,
          "name": "cell61 ~ gene20"
        },
        {
          "x": 60,
          "y": 20,
          "value": 1,
          "name": "cell61 ~ gene21"
        },
        {
          "x": 60,
          "y": 21,
          "value": 0,
          "name": "cell61 ~ gene22"
        },
        {
          "x": 60,
          "y": 22,
          "value": 0,
          "name": "cell61 ~ gene23"
        },
        {
          "x": 60,
          "y": 23,
          "value": 0,
          "name": "cell61 ~ gene24"
        },
        {
          "x": 60,
          "y": 24,
          "value": 1,
          "name": "cell61 ~ gene25"
        },
        {
          "x": 60,
          "y": 25,
          "value": 0,
          "name": "cell61 ~ gene26"
        },
        {
          "x": 60,
          "y": 26,
          "value": 0,
          "name": "cell61 ~ gene27"
        },
        {
          "x": 60,
          "y": 27,
          "value": 0,
          "name": "cell61 ~ gene28"
        },
        {
          "x": 60,
          "y": 28,
          "value": 11,
          "name": "cell61 ~ gene29"
        },
        {
          "x": 60,
          "y": 29,
          "value": 0,
          "name": "cell61 ~ gene30"
        },
        {
          "x": 60,
          "y": 30,
          "value": 5,
          "name": "cell61 ~ gene31"
        },
        {
          "x": 60,
          "y": 31,
          "value": 5,
          "name": "cell61 ~ gene32"
        },
        {
          "x": 60,
          "y": 32,
          "value": 0,
          "name": "cell61 ~ gene33"
        },
        {
          "x": 60,
          "y": 33,
          "value": 0,
          "name": "cell61 ~ gene34"
        },
        {
          "x": 60,
          "y": 34,
          "value": 6,
          "name": "cell61 ~ gene35"
        },
        {
          "x": 60,
          "y": 35,
          "value": 0,
          "name": "cell61 ~ gene36"
        },
        {
          "x": 60,
          "y": 36,
          "value": 0,
          "name": "cell61 ~ gene37"
        },
        {
          "x": 60,
          "y": 37,
          "value": 0,
          "name": "cell61 ~ gene38"
        },
        {
          "x": 60,
          "y": 38,
          "value": 0,
          "name": "cell61 ~ gene39"
        },
        {
          "x": 60,
          "y": 39,
          "value": 0,
          "name": "cell61 ~ gene40"
        },
        {
          "x": 60,
          "y": 40,
          "value": 20,
          "name": "cell61 ~ gene41"
        },
        {
          "x": 60,
          "y": 41,
          "value": 21,
          "name": "cell61 ~ gene42"
        },
        {
          "x": 60,
          "y": 42,
          "value": 0,
          "name": "cell61 ~ gene43"
        },
        {
          "x": 60,
          "y": 43,
          "value": 2,
          "name": "cell61 ~ gene44"
        },
        {
          "x": 60,
          "y": 44,
          "value": 10,
          "name": "cell61 ~ gene45"
        },
        {
          "x": 60,
          "y": 45,
          "value": 19,
          "name": "cell61 ~ gene46"
        },
        {
          "x": 60,
          "y": 46,
          "value": 8,
          "name": "cell61 ~ gene47"
        },
        {
          "x": 60,
          "y": 47,
          "value": 3,
          "name": "cell61 ~ gene48"
        },
        {
          "x": 60,
          "y": 48,
          "value": 0,
          "name": "cell61 ~ gene49"
        },
        {
          "x": 60,
          "y": 49,
          "value": 10,
          "name": "cell61 ~ gene50"
        },
        {
          "x": 60,
          "y": 50,
          "value": 16,
          "name": "cell61 ~ gene51"
        },
        {
          "x": 60,
          "y": 51,
          "value": 14,
          "name": "cell61 ~ gene52"
        },
        {
          "x": 60,
          "y": 52,
          "value": 24,
          "name": "cell61 ~ gene53"
        },
        {
          "x": 60,
          "y": 53,
          "value": 21,
          "name": "cell61 ~ gene54"
        },
        {
          "x": 60,
          "y": 54,
          "value": 31,
          "name": "cell61 ~ gene55"
        },
        {
          "x": 60,
          "y": 55,
          "value": 30,
          "name": "cell61 ~ gene56"
        },
        {
          "x": 60,
          "y": 56,
          "value": 30,
          "name": "cell61 ~ gene57"
        },
        {
          "x": 60,
          "y": 57,
          "value": 32,
          "name": "cell61 ~ gene58"
        },
        {
          "x": 60,
          "y": 58,
          "value": 10,
          "name": "cell61 ~ gene59"
        },
        {
          "x": 60,
          "y": 59,
          "value": 5,
          "name": "cell61 ~ gene60"
        },
        {
          "x": 60,
          "y": 60,
          "value": 8,
          "name": "cell61 ~ gene61"
        },
        {
          "x": 60,
          "y": 61,
          "value": 3,
          "name": "cell61 ~ gene62"
        },
        {
          "x": 60,
          "y": 62,
          "value": 0,
          "name": "cell61 ~ gene63"
        },
        {
          "x": 60,
          "y": 63,
          "value": 0,
          "name": "cell61 ~ gene64"
        },
        {
          "x": 60,
          "y": 64,
          "value": 0,
          "name": "cell61 ~ gene65"
        },
        {
          "x": 60,
          "y": 65,
          "value": 0,
          "name": "cell61 ~ gene66"
        },
        {
          "x": 60,
          "y": 66,
          "value": 2,
          "name": "cell61 ~ gene67"
        },
        {
          "x": 60,
          "y": 67,
          "value": 13,
          "name": "cell61 ~ gene68"
        },
        {
          "x": 60,
          "y": 68,
          "value": 24,
          "name": "cell61 ~ gene69"
        },
        {
          "x": 60,
          "y": 69,
          "value": 7,
          "name": "cell61 ~ gene70"
        },
        {
          "x": 60,
          "y": 70,
          "value": 0,
          "name": "cell61 ~ gene71"
        },
        {
          "x": 60,
          "y": 71,
          "value": 7,
          "name": "cell61 ~ gene72"
        },
        {
          "x": 60,
          "y": 72,
          "value": 19,
          "name": "cell61 ~ gene73"
        },
        {
          "x": 60,
          "y": 73,
          "value": 0,
          "name": "cell61 ~ gene74"
        },
        {
          "x": 60,
          "y": 74,
          "value": 0,
          "name": "cell61 ~ gene75"
        },
        {
          "x": 60,
          "y": 75,
          "value": 0,
          "name": "cell61 ~ gene76"
        },
        {
          "x": 60,
          "y": 76,
          "value": 0,
          "name": "cell61 ~ gene77"
        },
        {
          "x": 60,
          "y": 77,
          "value": 12,
          "name": "cell61 ~ gene78"
        },
        {
          "x": 60,
          "y": 78,
          "value": 3,
          "name": "cell61 ~ gene79"
        },
        {
          "x": 60,
          "y": 79,
          "value": 0,
          "name": "cell61 ~ gene80"
        },
        {
          "x": 60,
          "y": 80,
          "value": 0,
          "name": "cell61 ~ gene81"
        },
        {
          "x": 60,
          "y": 81,
          "value": 3,
          "name": "cell61 ~ gene82"
        },
        {
          "x": 60,
          "y": 82,
          "value": 0,
          "name": "cell61 ~ gene83"
        },
        {
          "x": 60,
          "y": 83,
          "value": 0,
          "name": "cell61 ~ gene84"
        },
        {
          "x": 60,
          "y": 84,
          "value": 10,
          "name": "cell61 ~ gene85"
        },
        {
          "x": 60,
          "y": 85,
          "value": 0,
          "name": "cell61 ~ gene86"
        },
        {
          "x": 60,
          "y": 86,
          "value": 11,
          "name": "cell61 ~ gene87"
        },
        {
          "x": 60,
          "y": 87,
          "value": 0,
          "name": "cell61 ~ gene88"
        },
        {
          "x": 60,
          "y": 88,
          "value": 2,
          "name": "cell61 ~ gene89"
        },
        {
          "x": 60,
          "y": 89,
          "value": 0,
          "name": "cell61 ~ gene90"
        },
        {
          "x": 60,
          "y": 90,
          "value": 0,
          "name": "cell61 ~ gene91"
        },
        {
          "x": 60,
          "y": 91,
          "value": 11,
          "name": "cell61 ~ gene92"
        },
        {
          "x": 60,
          "y": 92,
          "value": 0,
          "name": "cell61 ~ gene93"
        },
        {
          "x": 60,
          "y": 93,
          "value": 0,
          "name": "cell61 ~ gene94"
        },
        {
          "x": 60,
          "y": 94,
          "value": 0,
          "name": "cell61 ~ gene95"
        },
        {
          "x": 60,
          "y": 95,
          "value": 2,
          "name": "cell61 ~ gene96"
        },
        {
          "x": 60,
          "y": 96,
          "value": 11,
          "name": "cell61 ~ gene97"
        },
        {
          "x": 60,
          "y": 97,
          "value": 0,
          "name": "cell61 ~ gene98"
        },
        {
          "x": 60,
          "y": 98,
          "value": 5,
          "name": "cell61 ~ gene99"
        },
        {
          "x": 60,
          "y": 99,
          "value": 0,
          "name": "cell61 ~ gene100"
        },
        {
          "x": 61,
          "y": 0,
          "value": 0,
          "name": "cell62 ~ gene1"
        },
        {
          "x": 61,
          "y": 1,
          "value": 1,
          "name": "cell62 ~ gene2"
        },
        {
          "x": 61,
          "y": 2,
          "value": 3,
          "name": "cell62 ~ gene3"
        },
        {
          "x": 61,
          "y": 3,
          "value": 16,
          "name": "cell62 ~ gene4"
        },
        {
          "x": 61,
          "y": 4,
          "value": 1,
          "name": "cell62 ~ gene5"
        },
        {
          "x": 61,
          "y": 5,
          "value": 0,
          "name": "cell62 ~ gene6"
        },
        {
          "x": 61,
          "y": 6,
          "value": 0,
          "name": "cell62 ~ gene7"
        },
        {
          "x": 61,
          "y": 7,
          "value": 0,
          "name": "cell62 ~ gene8"
        },
        {
          "x": 61,
          "y": 8,
          "value": 0,
          "name": "cell62 ~ gene9"
        },
        {
          "x": 61,
          "y": 9,
          "value": 0,
          "name": "cell62 ~ gene10"
        },
        {
          "x": 61,
          "y": 10,
          "value": 0,
          "name": "cell62 ~ gene11"
        },
        {
          "x": 61,
          "y": 11,
          "value": 0,
          "name": "cell62 ~ gene12"
        },
        {
          "x": 61,
          "y": 12,
          "value": 16,
          "name": "cell62 ~ gene13"
        },
        {
          "x": 61,
          "y": 13,
          "value": 0,
          "name": "cell62 ~ gene14"
        },
        {
          "x": 61,
          "y": 14,
          "value": 3,
          "name": "cell62 ~ gene15"
        },
        {
          "x": 61,
          "y": 15,
          "value": 0,
          "name": "cell62 ~ gene16"
        },
        {
          "x": 61,
          "y": 16,
          "value": 3,
          "name": "cell62 ~ gene17"
        },
        {
          "x": 61,
          "y": 17,
          "value": 0,
          "name": "cell62 ~ gene18"
        },
        {
          "x": 61,
          "y": 18,
          "value": 0,
          "name": "cell62 ~ gene19"
        },
        {
          "x": 61,
          "y": 19,
          "value": 0,
          "name": "cell62 ~ gene20"
        },
        {
          "x": 61,
          "y": 20,
          "value": 0,
          "name": "cell62 ~ gene21"
        },
        {
          "x": 61,
          "y": 21,
          "value": 0,
          "name": "cell62 ~ gene22"
        },
        {
          "x": 61,
          "y": 22,
          "value": 0,
          "name": "cell62 ~ gene23"
        },
        {
          "x": 61,
          "y": 23,
          "value": 0,
          "name": "cell62 ~ gene24"
        },
        {
          "x": 61,
          "y": 24,
          "value": 0,
          "name": "cell62 ~ gene25"
        },
        {
          "x": 61,
          "y": 25,
          "value": 0,
          "name": "cell62 ~ gene26"
        },
        {
          "x": 61,
          "y": 26,
          "value": 10,
          "name": "cell62 ~ gene27"
        },
        {
          "x": 61,
          "y": 27,
          "value": 11,
          "name": "cell62 ~ gene28"
        },
        {
          "x": 61,
          "y": 28,
          "value": 2,
          "name": "cell62 ~ gene29"
        },
        {
          "x": 61,
          "y": 29,
          "value": 0,
          "name": "cell62 ~ gene30"
        },
        {
          "x": 61,
          "y": 30,
          "value": 0,
          "name": "cell62 ~ gene31"
        },
        {
          "x": 61,
          "y": 31,
          "value": 7,
          "name": "cell62 ~ gene32"
        },
        {
          "x": 61,
          "y": 32,
          "value": 5,
          "name": "cell62 ~ gene33"
        },
        {
          "x": 61,
          "y": 33,
          "value": 0,
          "name": "cell62 ~ gene34"
        },
        {
          "x": 61,
          "y": 34,
          "value": 0,
          "name": "cell62 ~ gene35"
        },
        {
          "x": 61,
          "y": 35,
          "value": 0,
          "name": "cell62 ~ gene36"
        },
        {
          "x": 61,
          "y": 36,
          "value": 0,
          "name": "cell62 ~ gene37"
        },
        {
          "x": 61,
          "y": 37,
          "value": 4,
          "name": "cell62 ~ gene38"
        },
        {
          "x": 61,
          "y": 38,
          "value": 29,
          "name": "cell62 ~ gene39"
        },
        {
          "x": 61,
          "y": 39,
          "value": 0,
          "name": "cell62 ~ gene40"
        },
        {
          "x": 61,
          "y": 40,
          "value": 0,
          "name": "cell62 ~ gene41"
        },
        {
          "x": 61,
          "y": 41,
          "value": 25,
          "name": "cell62 ~ gene42"
        },
        {
          "x": 61,
          "y": 42,
          "value": 0,
          "name": "cell62 ~ gene43"
        },
        {
          "x": 61,
          "y": 43,
          "value": 0,
          "name": "cell62 ~ gene44"
        },
        {
          "x": 61,
          "y": 44,
          "value": 1,
          "name": "cell62 ~ gene45"
        },
        {
          "x": 61,
          "y": 45,
          "value": 25,
          "name": "cell62 ~ gene46"
        },
        {
          "x": 61,
          "y": 46,
          "value": 14,
          "name": "cell62 ~ gene47"
        },
        {
          "x": 61,
          "y": 47,
          "value": 8,
          "name": "cell62 ~ gene48"
        },
        {
          "x": 61,
          "y": 48,
          "value": 0,
          "name": "cell62 ~ gene49"
        },
        {
          "x": 61,
          "y": 49,
          "value": 0,
          "name": "cell62 ~ gene50"
        },
        {
          "x": 61,
          "y": 50,
          "value": 26,
          "name": "cell62 ~ gene51"
        },
        {
          "x": 61,
          "y": 51,
          "value": 25,
          "name": "cell62 ~ gene52"
        },
        {
          "x": 61,
          "y": 52,
          "value": 0,
          "name": "cell62 ~ gene53"
        },
        {
          "x": 61,
          "y": 53,
          "value": 1,
          "name": "cell62 ~ gene54"
        },
        {
          "x": 61,
          "y": 54,
          "value": 10,
          "name": "cell62 ~ gene55"
        },
        {
          "x": 61,
          "y": 55,
          "value": 40,
          "name": "cell62 ~ gene56"
        },
        {
          "x": 61,
          "y": 56,
          "value": 17,
          "name": "cell62 ~ gene57"
        },
        {
          "x": 61,
          "y": 57,
          "value": 25,
          "name": "cell62 ~ gene58"
        },
        {
          "x": 61,
          "y": 58,
          "value": 0,
          "name": "cell62 ~ gene59"
        },
        {
          "x": 61,
          "y": 59,
          "value": 0,
          "name": "cell62 ~ gene60"
        },
        {
          "x": 61,
          "y": 60,
          "value": 9,
          "name": "cell62 ~ gene61"
        },
        {
          "x": 61,
          "y": 61,
          "value": 0,
          "name": "cell62 ~ gene62"
        },
        {
          "x": 61,
          "y": 62,
          "value": 0,
          "name": "cell62 ~ gene63"
        },
        {
          "x": 61,
          "y": 63,
          "value": 10,
          "name": "cell62 ~ gene64"
        },
        {
          "x": 61,
          "y": 64,
          "value": 1,
          "name": "cell62 ~ gene65"
        },
        {
          "x": 61,
          "y": 65,
          "value": 8,
          "name": "cell62 ~ gene66"
        },
        {
          "x": 61,
          "y": 66,
          "value": 0,
          "name": "cell62 ~ gene67"
        },
        {
          "x": 61,
          "y": 67,
          "value": 0,
          "name": "cell62 ~ gene68"
        },
        {
          "x": 61,
          "y": 68,
          "value": 4,
          "name": "cell62 ~ gene69"
        },
        {
          "x": 61,
          "y": 69,
          "value": 0,
          "name": "cell62 ~ gene70"
        },
        {
          "x": 61,
          "y": 70,
          "value": 0,
          "name": "cell62 ~ gene71"
        },
        {
          "x": 61,
          "y": 71,
          "value": 0,
          "name": "cell62 ~ gene72"
        },
        {
          "x": 61,
          "y": 72,
          "value": 5,
          "name": "cell62 ~ gene73"
        },
        {
          "x": 61,
          "y": 73,
          "value": 0,
          "name": "cell62 ~ gene74"
        },
        {
          "x": 61,
          "y": 74,
          "value": 4,
          "name": "cell62 ~ gene75"
        },
        {
          "x": 61,
          "y": 75,
          "value": 0,
          "name": "cell62 ~ gene76"
        },
        {
          "x": 61,
          "y": 76,
          "value": 12,
          "name": "cell62 ~ gene77"
        },
        {
          "x": 61,
          "y": 77,
          "value": 0,
          "name": "cell62 ~ gene78"
        },
        {
          "x": 61,
          "y": 78,
          "value": 4,
          "name": "cell62 ~ gene79"
        },
        {
          "x": 61,
          "y": 79,
          "value": 11,
          "name": "cell62 ~ gene80"
        },
        {
          "x": 61,
          "y": 80,
          "value": 0,
          "name": "cell62 ~ gene81"
        },
        {
          "x": 61,
          "y": 81,
          "value": 3,
          "name": "cell62 ~ gene82"
        },
        {
          "x": 61,
          "y": 82,
          "value": 0,
          "name": "cell62 ~ gene83"
        },
        {
          "x": 61,
          "y": 83,
          "value": 7,
          "name": "cell62 ~ gene84"
        },
        {
          "x": 61,
          "y": 84,
          "value": 7,
          "name": "cell62 ~ gene85"
        },
        {
          "x": 61,
          "y": 85,
          "value": 8,
          "name": "cell62 ~ gene86"
        },
        {
          "x": 61,
          "y": 86,
          "value": 1,
          "name": "cell62 ~ gene87"
        },
        {
          "x": 61,
          "y": 87,
          "value": 0,
          "name": "cell62 ~ gene88"
        },
        {
          "x": 61,
          "y": 88,
          "value": 0,
          "name": "cell62 ~ gene89"
        },
        {
          "x": 61,
          "y": 89,
          "value": 0,
          "name": "cell62 ~ gene90"
        },
        {
          "x": 61,
          "y": 90,
          "value": 0,
          "name": "cell62 ~ gene91"
        },
        {
          "x": 61,
          "y": 91,
          "value": 23,
          "name": "cell62 ~ gene92"
        },
        {
          "x": 61,
          "y": 92,
          "value": 0,
          "name": "cell62 ~ gene93"
        },
        {
          "x": 61,
          "y": 93,
          "value": 4,
          "name": "cell62 ~ gene94"
        },
        {
          "x": 61,
          "y": 94,
          "value": 0,
          "name": "cell62 ~ gene95"
        },
        {
          "x": 61,
          "y": 95,
          "value": 10,
          "name": "cell62 ~ gene96"
        },
        {
          "x": 61,
          "y": 96,
          "value": 7,
          "name": "cell62 ~ gene97"
        },
        {
          "x": 61,
          "y": 97,
          "value": 0,
          "name": "cell62 ~ gene98"
        },
        {
          "x": 61,
          "y": 98,
          "value": 1,
          "name": "cell62 ~ gene99"
        },
        {
          "x": 61,
          "y": 99,
          "value": 0,
          "name": "cell62 ~ gene100"
        },
        {
          "x": 62,
          "y": 0,
          "value": 0,
          "name": "cell63 ~ gene1"
        },
        {
          "x": 62,
          "y": 1,
          "value": 1,
          "name": "cell63 ~ gene2"
        },
        {
          "x": 62,
          "y": 2,
          "value": 12,
          "name": "cell63 ~ gene3"
        },
        {
          "x": 62,
          "y": 3,
          "value": 0,
          "name": "cell63 ~ gene4"
        },
        {
          "x": 62,
          "y": 4,
          "value": 18,
          "name": "cell63 ~ gene5"
        },
        {
          "x": 62,
          "y": 5,
          "value": 8,
          "name": "cell63 ~ gene6"
        },
        {
          "x": 62,
          "y": 6,
          "value": 1,
          "name": "cell63 ~ gene7"
        },
        {
          "x": 62,
          "y": 7,
          "value": 9,
          "name": "cell63 ~ gene8"
        },
        {
          "x": 62,
          "y": 8,
          "value": 0,
          "name": "cell63 ~ gene9"
        },
        {
          "x": 62,
          "y": 9,
          "value": 0,
          "name": "cell63 ~ gene10"
        },
        {
          "x": 62,
          "y": 10,
          "value": 1,
          "name": "cell63 ~ gene11"
        },
        {
          "x": 62,
          "y": 11,
          "value": 0,
          "name": "cell63 ~ gene12"
        },
        {
          "x": 62,
          "y": 12,
          "value": 0,
          "name": "cell63 ~ gene13"
        },
        {
          "x": 62,
          "y": 13,
          "value": 0,
          "name": "cell63 ~ gene14"
        },
        {
          "x": 62,
          "y": 14,
          "value": 0,
          "name": "cell63 ~ gene15"
        },
        {
          "x": 62,
          "y": 15,
          "value": 0,
          "name": "cell63 ~ gene16"
        },
        {
          "x": 62,
          "y": 16,
          "value": 0,
          "name": "cell63 ~ gene17"
        },
        {
          "x": 62,
          "y": 17,
          "value": 2,
          "name": "cell63 ~ gene18"
        },
        {
          "x": 62,
          "y": 18,
          "value": 13,
          "name": "cell63 ~ gene19"
        },
        {
          "x": 62,
          "y": 19,
          "value": 0,
          "name": "cell63 ~ gene20"
        },
        {
          "x": 62,
          "y": 20,
          "value": 0,
          "name": "cell63 ~ gene21"
        },
        {
          "x": 62,
          "y": 21,
          "value": 8,
          "name": "cell63 ~ gene22"
        },
        {
          "x": 62,
          "y": 22,
          "value": 0,
          "name": "cell63 ~ gene23"
        },
        {
          "x": 62,
          "y": 23,
          "value": 0,
          "name": "cell63 ~ gene24"
        },
        {
          "x": 62,
          "y": 24,
          "value": 9,
          "name": "cell63 ~ gene25"
        },
        {
          "x": 62,
          "y": 25,
          "value": 0,
          "name": "cell63 ~ gene26"
        },
        {
          "x": 62,
          "y": 26,
          "value": 0,
          "name": "cell63 ~ gene27"
        },
        {
          "x": 62,
          "y": 27,
          "value": 6,
          "name": "cell63 ~ gene28"
        },
        {
          "x": 62,
          "y": 28,
          "value": 0,
          "name": "cell63 ~ gene29"
        },
        {
          "x": 62,
          "y": 29,
          "value": 15,
          "name": "cell63 ~ gene30"
        },
        {
          "x": 62,
          "y": 30,
          "value": 0,
          "name": "cell63 ~ gene31"
        },
        {
          "x": 62,
          "y": 31,
          "value": 9,
          "name": "cell63 ~ gene32"
        },
        {
          "x": 62,
          "y": 32,
          "value": 3,
          "name": "cell63 ~ gene33"
        },
        {
          "x": 62,
          "y": 33,
          "value": 10,
          "name": "cell63 ~ gene34"
        },
        {
          "x": 62,
          "y": 34,
          "value": 7,
          "name": "cell63 ~ gene35"
        },
        {
          "x": 62,
          "y": 35,
          "value": 0,
          "name": "cell63 ~ gene36"
        },
        {
          "x": 62,
          "y": 36,
          "value": 9,
          "name": "cell63 ~ gene37"
        },
        {
          "x": 62,
          "y": 37,
          "value": 9,
          "name": "cell63 ~ gene38"
        },
        {
          "x": 62,
          "y": 38,
          "value": 8,
          "name": "cell63 ~ gene39"
        },
        {
          "x": 62,
          "y": 39,
          "value": 0,
          "name": "cell63 ~ gene40"
        },
        {
          "x": 62,
          "y": 40,
          "value": 15,
          "name": "cell63 ~ gene41"
        },
        {
          "x": 62,
          "y": 41,
          "value": 12,
          "name": "cell63 ~ gene42"
        },
        {
          "x": 62,
          "y": 42,
          "value": 7,
          "name": "cell63 ~ gene43"
        },
        {
          "x": 62,
          "y": 43,
          "value": 12,
          "name": "cell63 ~ gene44"
        },
        {
          "x": 62,
          "y": 44,
          "value": 5,
          "name": "cell63 ~ gene45"
        },
        {
          "x": 62,
          "y": 45,
          "value": 30,
          "name": "cell63 ~ gene46"
        },
        {
          "x": 62,
          "y": 46,
          "value": 18,
          "name": "cell63 ~ gene47"
        },
        {
          "x": 62,
          "y": 47,
          "value": 14,
          "name": "cell63 ~ gene48"
        },
        {
          "x": 62,
          "y": 48,
          "value": 0,
          "name": "cell63 ~ gene49"
        },
        {
          "x": 62,
          "y": 49,
          "value": 17,
          "name": "cell63 ~ gene50"
        },
        {
          "x": 62,
          "y": 50,
          "value": 35,
          "name": "cell63 ~ gene51"
        },
        {
          "x": 62,
          "y": 51,
          "value": 21,
          "name": "cell63 ~ gene52"
        },
        {
          "x": 62,
          "y": 52,
          "value": 6,
          "name": "cell63 ~ gene53"
        },
        {
          "x": 62,
          "y": 53,
          "value": 10,
          "name": "cell63 ~ gene54"
        },
        {
          "x": 62,
          "y": 54,
          "value": 22,
          "name": "cell63 ~ gene55"
        },
        {
          "x": 62,
          "y": 55,
          "value": 14,
          "name": "cell63 ~ gene56"
        },
        {
          "x": 62,
          "y": 56,
          "value": 12,
          "name": "cell63 ~ gene57"
        },
        {
          "x": 62,
          "y": 57,
          "value": 30,
          "name": "cell63 ~ gene58"
        },
        {
          "x": 62,
          "y": 58,
          "value": 11,
          "name": "cell63 ~ gene59"
        },
        {
          "x": 62,
          "y": 59,
          "value": 7,
          "name": "cell63 ~ gene60"
        },
        {
          "x": 62,
          "y": 60,
          "value": 0,
          "name": "cell63 ~ gene61"
        },
        {
          "x": 62,
          "y": 61,
          "value": 2,
          "name": "cell63 ~ gene62"
        },
        {
          "x": 62,
          "y": 62,
          "value": 1,
          "name": "cell63 ~ gene63"
        },
        {
          "x": 62,
          "y": 63,
          "value": 0,
          "name": "cell63 ~ gene64"
        },
        {
          "x": 62,
          "y": 64,
          "value": 0,
          "name": "cell63 ~ gene65"
        },
        {
          "x": 62,
          "y": 65,
          "value": 14,
          "name": "cell63 ~ gene66"
        },
        {
          "x": 62,
          "y": 66,
          "value": 2,
          "name": "cell63 ~ gene67"
        },
        {
          "x": 62,
          "y": 67,
          "value": 0,
          "name": "cell63 ~ gene68"
        },
        {
          "x": 62,
          "y": 68,
          "value": 0,
          "name": "cell63 ~ gene69"
        },
        {
          "x": 62,
          "y": 69,
          "value": 7,
          "name": "cell63 ~ gene70"
        },
        {
          "x": 62,
          "y": 70,
          "value": 11,
          "name": "cell63 ~ gene71"
        },
        {
          "x": 62,
          "y": 71,
          "value": 0,
          "name": "cell63 ~ gene72"
        },
        {
          "x": 62,
          "y": 72,
          "value": 1,
          "name": "cell63 ~ gene73"
        },
        {
          "x": 62,
          "y": 73,
          "value": 11,
          "name": "cell63 ~ gene74"
        },
        {
          "x": 62,
          "y": 74,
          "value": 2,
          "name": "cell63 ~ gene75"
        },
        {
          "x": 62,
          "y": 75,
          "value": 0,
          "name": "cell63 ~ gene76"
        },
        {
          "x": 62,
          "y": 76,
          "value": 0,
          "name": "cell63 ~ gene77"
        },
        {
          "x": 62,
          "y": 77,
          "value": 19,
          "name": "cell63 ~ gene78"
        },
        {
          "x": 62,
          "y": 78,
          "value": 0,
          "name": "cell63 ~ gene79"
        },
        {
          "x": 62,
          "y": 79,
          "value": 0,
          "name": "cell63 ~ gene80"
        },
        {
          "x": 62,
          "y": 80,
          "value": 11,
          "name": "cell63 ~ gene81"
        },
        {
          "x": 62,
          "y": 81,
          "value": 6,
          "name": "cell63 ~ gene82"
        },
        {
          "x": 62,
          "y": 82,
          "value": 26,
          "name": "cell63 ~ gene83"
        },
        {
          "x": 62,
          "y": 83,
          "value": 0,
          "name": "cell63 ~ gene84"
        },
        {
          "x": 62,
          "y": 84,
          "value": 0,
          "name": "cell63 ~ gene85"
        },
        {
          "x": 62,
          "y": 85,
          "value": 3,
          "name": "cell63 ~ gene86"
        },
        {
          "x": 62,
          "y": 86,
          "value": 0,
          "name": "cell63 ~ gene87"
        },
        {
          "x": 62,
          "y": 87,
          "value": 23,
          "name": "cell63 ~ gene88"
        },
        {
          "x": 62,
          "y": 88,
          "value": 0,
          "name": "cell63 ~ gene89"
        },
        {
          "x": 62,
          "y": 89,
          "value": 0,
          "name": "cell63 ~ gene90"
        },
        {
          "x": 62,
          "y": 90,
          "value": 6,
          "name": "cell63 ~ gene91"
        },
        {
          "x": 62,
          "y": 91,
          "value": 0,
          "name": "cell63 ~ gene92"
        },
        {
          "x": 62,
          "y": 92,
          "value": 11,
          "name": "cell63 ~ gene93"
        },
        {
          "x": 62,
          "y": 93,
          "value": 3,
          "name": "cell63 ~ gene94"
        },
        {
          "x": 62,
          "y": 94,
          "value": 0,
          "name": "cell63 ~ gene95"
        },
        {
          "x": 62,
          "y": 95,
          "value": 0,
          "name": "cell63 ~ gene96"
        },
        {
          "x": 62,
          "y": 96,
          "value": 6,
          "name": "cell63 ~ gene97"
        },
        {
          "x": 62,
          "y": 97,
          "value": 18,
          "name": "cell63 ~ gene98"
        },
        {
          "x": 62,
          "y": 98,
          "value": 6,
          "name": "cell63 ~ gene99"
        },
        {
          "x": 62,
          "y": 99,
          "value": 0,
          "name": "cell63 ~ gene100"
        },
        {
          "x": 63,
          "y": 0,
          "value": 13,
          "name": "cell64 ~ gene1"
        },
        {
          "x": 63,
          "y": 1,
          "value": 0,
          "name": "cell64 ~ gene2"
        },
        {
          "x": 63,
          "y": 2,
          "value": 3,
          "name": "cell64 ~ gene3"
        },
        {
          "x": 63,
          "y": 3,
          "value": 0,
          "name": "cell64 ~ gene4"
        },
        {
          "x": 63,
          "y": 4,
          "value": 22,
          "name": "cell64 ~ gene5"
        },
        {
          "x": 63,
          "y": 5,
          "value": 11,
          "name": "cell64 ~ gene6"
        },
        {
          "x": 63,
          "y": 6,
          "value": 0,
          "name": "cell64 ~ gene7"
        },
        {
          "x": 63,
          "y": 7,
          "value": 7,
          "name": "cell64 ~ gene8"
        },
        {
          "x": 63,
          "y": 8,
          "value": 8,
          "name": "cell64 ~ gene9"
        },
        {
          "x": 63,
          "y": 9,
          "value": 3,
          "name": "cell64 ~ gene10"
        },
        {
          "x": 63,
          "y": 10,
          "value": 0,
          "name": "cell64 ~ gene11"
        },
        {
          "x": 63,
          "y": 11,
          "value": 22,
          "name": "cell64 ~ gene12"
        },
        {
          "x": 63,
          "y": 12,
          "value": 0,
          "name": "cell64 ~ gene13"
        },
        {
          "x": 63,
          "y": 13,
          "value": 15,
          "name": "cell64 ~ gene14"
        },
        {
          "x": 63,
          "y": 14,
          "value": 3,
          "name": "cell64 ~ gene15"
        },
        {
          "x": 63,
          "y": 15,
          "value": 6,
          "name": "cell64 ~ gene16"
        },
        {
          "x": 63,
          "y": 16,
          "value": 0,
          "name": "cell64 ~ gene17"
        },
        {
          "x": 63,
          "y": 17,
          "value": 0,
          "name": "cell64 ~ gene18"
        },
        {
          "x": 63,
          "y": 18,
          "value": 5,
          "name": "cell64 ~ gene19"
        },
        {
          "x": 63,
          "y": 19,
          "value": 0,
          "name": "cell64 ~ gene20"
        },
        {
          "x": 63,
          "y": 20,
          "value": 0,
          "name": "cell64 ~ gene21"
        },
        {
          "x": 63,
          "y": 21,
          "value": 3,
          "name": "cell64 ~ gene22"
        },
        {
          "x": 63,
          "y": 22,
          "value": 19,
          "name": "cell64 ~ gene23"
        },
        {
          "x": 63,
          "y": 23,
          "value": 0,
          "name": "cell64 ~ gene24"
        },
        {
          "x": 63,
          "y": 24,
          "value": 3,
          "name": "cell64 ~ gene25"
        },
        {
          "x": 63,
          "y": 25,
          "value": 0,
          "name": "cell64 ~ gene26"
        },
        {
          "x": 63,
          "y": 26,
          "value": 2,
          "name": "cell64 ~ gene27"
        },
        {
          "x": 63,
          "y": 27,
          "value": 0,
          "name": "cell64 ~ gene28"
        },
        {
          "x": 63,
          "y": 28,
          "value": 17,
          "name": "cell64 ~ gene29"
        },
        {
          "x": 63,
          "y": 29,
          "value": 4,
          "name": "cell64 ~ gene30"
        },
        {
          "x": 63,
          "y": 30,
          "value": 0,
          "name": "cell64 ~ gene31"
        },
        {
          "x": 63,
          "y": 31,
          "value": 10,
          "name": "cell64 ~ gene32"
        },
        {
          "x": 63,
          "y": 32,
          "value": 0,
          "name": "cell64 ~ gene33"
        },
        {
          "x": 63,
          "y": 33,
          "value": 2,
          "name": "cell64 ~ gene34"
        },
        {
          "x": 63,
          "y": 34,
          "value": 4,
          "name": "cell64 ~ gene35"
        },
        {
          "x": 63,
          "y": 35,
          "value": 8,
          "name": "cell64 ~ gene36"
        },
        {
          "x": 63,
          "y": 36,
          "value": 0,
          "name": "cell64 ~ gene37"
        },
        {
          "x": 63,
          "y": 37,
          "value": 6,
          "name": "cell64 ~ gene38"
        },
        {
          "x": 63,
          "y": 38,
          "value": 0,
          "name": "cell64 ~ gene39"
        },
        {
          "x": 63,
          "y": 39,
          "value": 0,
          "name": "cell64 ~ gene40"
        },
        {
          "x": 63,
          "y": 40,
          "value": 2,
          "name": "cell64 ~ gene41"
        },
        {
          "x": 63,
          "y": 41,
          "value": 25,
          "name": "cell64 ~ gene42"
        },
        {
          "x": 63,
          "y": 42,
          "value": 8,
          "name": "cell64 ~ gene43"
        },
        {
          "x": 63,
          "y": 43,
          "value": 15,
          "name": "cell64 ~ gene44"
        },
        {
          "x": 63,
          "y": 44,
          "value": 0,
          "name": "cell64 ~ gene45"
        },
        {
          "x": 63,
          "y": 45,
          "value": 8,
          "name": "cell64 ~ gene46"
        },
        {
          "x": 63,
          "y": 46,
          "value": 14,
          "name": "cell64 ~ gene47"
        },
        {
          "x": 63,
          "y": 47,
          "value": 8,
          "name": "cell64 ~ gene48"
        },
        {
          "x": 63,
          "y": 48,
          "value": 0,
          "name": "cell64 ~ gene49"
        },
        {
          "x": 63,
          "y": 49,
          "value": 13,
          "name": "cell64 ~ gene50"
        },
        {
          "x": 63,
          "y": 50,
          "value": 21,
          "name": "cell64 ~ gene51"
        },
        {
          "x": 63,
          "y": 51,
          "value": 21,
          "name": "cell64 ~ gene52"
        },
        {
          "x": 63,
          "y": 52,
          "value": 0,
          "name": "cell64 ~ gene53"
        },
        {
          "x": 63,
          "y": 53,
          "value": 0,
          "name": "cell64 ~ gene54"
        },
        {
          "x": 63,
          "y": 54,
          "value": 26,
          "name": "cell64 ~ gene55"
        },
        {
          "x": 63,
          "y": 55,
          "value": 44,
          "name": "cell64 ~ gene56"
        },
        {
          "x": 63,
          "y": 56,
          "value": 13,
          "name": "cell64 ~ gene57"
        },
        {
          "x": 63,
          "y": 57,
          "value": 15,
          "name": "cell64 ~ gene58"
        },
        {
          "x": 63,
          "y": 58,
          "value": 9,
          "name": "cell64 ~ gene59"
        },
        {
          "x": 63,
          "y": 59,
          "value": 0,
          "name": "cell64 ~ gene60"
        },
        {
          "x": 63,
          "y": 60,
          "value": 0,
          "name": "cell64 ~ gene61"
        },
        {
          "x": 63,
          "y": 61,
          "value": 0,
          "name": "cell64 ~ gene62"
        },
        {
          "x": 63,
          "y": 62,
          "value": 0,
          "name": "cell64 ~ gene63"
        },
        {
          "x": 63,
          "y": 63,
          "value": 0,
          "name": "cell64 ~ gene64"
        },
        {
          "x": 63,
          "y": 64,
          "value": 0,
          "name": "cell64 ~ gene65"
        },
        {
          "x": 63,
          "y": 65,
          "value": 7,
          "name": "cell64 ~ gene66"
        },
        {
          "x": 63,
          "y": 66,
          "value": 8,
          "name": "cell64 ~ gene67"
        },
        {
          "x": 63,
          "y": 67,
          "value": 14,
          "name": "cell64 ~ gene68"
        },
        {
          "x": 63,
          "y": 68,
          "value": 0,
          "name": "cell64 ~ gene69"
        },
        {
          "x": 63,
          "y": 69,
          "value": 0,
          "name": "cell64 ~ gene70"
        },
        {
          "x": 63,
          "y": 70,
          "value": 4,
          "name": "cell64 ~ gene71"
        },
        {
          "x": 63,
          "y": 71,
          "value": 0,
          "name": "cell64 ~ gene72"
        },
        {
          "x": 63,
          "y": 72,
          "value": 6,
          "name": "cell64 ~ gene73"
        },
        {
          "x": 63,
          "y": 73,
          "value": 0,
          "name": "cell64 ~ gene74"
        },
        {
          "x": 63,
          "y": 74,
          "value": 0,
          "name": "cell64 ~ gene75"
        },
        {
          "x": 63,
          "y": 75,
          "value": 14,
          "name": "cell64 ~ gene76"
        },
        {
          "x": 63,
          "y": 76,
          "value": 12,
          "name": "cell64 ~ gene77"
        },
        {
          "x": 63,
          "y": 77,
          "value": 0,
          "name": "cell64 ~ gene78"
        },
        {
          "x": 63,
          "y": 78,
          "value": 4,
          "name": "cell64 ~ gene79"
        },
        {
          "x": 63,
          "y": 79,
          "value": 0,
          "name": "cell64 ~ gene80"
        },
        {
          "x": 63,
          "y": 80,
          "value": 0,
          "name": "cell64 ~ gene81"
        },
        {
          "x": 63,
          "y": 81,
          "value": 16,
          "name": "cell64 ~ gene82"
        },
        {
          "x": 63,
          "y": 82,
          "value": 0,
          "name": "cell64 ~ gene83"
        },
        {
          "x": 63,
          "y": 83,
          "value": 2,
          "name": "cell64 ~ gene84"
        },
        {
          "x": 63,
          "y": 84,
          "value": 0,
          "name": "cell64 ~ gene85"
        },
        {
          "x": 63,
          "y": 85,
          "value": 1,
          "name": "cell64 ~ gene86"
        },
        {
          "x": 63,
          "y": 86,
          "value": 0,
          "name": "cell64 ~ gene87"
        },
        {
          "x": 63,
          "y": 87,
          "value": 7,
          "name": "cell64 ~ gene88"
        },
        {
          "x": 63,
          "y": 88,
          "value": 1,
          "name": "cell64 ~ gene89"
        },
        {
          "x": 63,
          "y": 89,
          "value": 10,
          "name": "cell64 ~ gene90"
        },
        {
          "x": 63,
          "y": 90,
          "value": 0,
          "name": "cell64 ~ gene91"
        },
        {
          "x": 63,
          "y": 91,
          "value": 0,
          "name": "cell64 ~ gene92"
        },
        {
          "x": 63,
          "y": 92,
          "value": 0,
          "name": "cell64 ~ gene93"
        },
        {
          "x": 63,
          "y": 93,
          "value": 0,
          "name": "cell64 ~ gene94"
        },
        {
          "x": 63,
          "y": 94,
          "value": 0,
          "name": "cell64 ~ gene95"
        },
        {
          "x": 63,
          "y": 95,
          "value": 0,
          "name": "cell64 ~ gene96"
        },
        {
          "x": 63,
          "y": 96,
          "value": 2,
          "name": "cell64 ~ gene97"
        },
        {
          "x": 63,
          "y": 97,
          "value": 0,
          "name": "cell64 ~ gene98"
        },
        {
          "x": 63,
          "y": 98,
          "value": 7,
          "name": "cell64 ~ gene99"
        },
        {
          "x": 63,
          "y": 99,
          "value": 11,
          "name": "cell64 ~ gene100"
        },
        {
          "x": 64,
          "y": 0,
          "value": 0,
          "name": "cell65 ~ gene1"
        },
        {
          "x": 64,
          "y": 1,
          "value": 4,
          "name": "cell65 ~ gene2"
        },
        {
          "x": 64,
          "y": 2,
          "value": 0,
          "name": "cell65 ~ gene3"
        },
        {
          "x": 64,
          "y": 3,
          "value": 2,
          "name": "cell65 ~ gene4"
        },
        {
          "x": 64,
          "y": 4,
          "value": 5,
          "name": "cell65 ~ gene5"
        },
        {
          "x": 64,
          "y": 5,
          "value": 2,
          "name": "cell65 ~ gene6"
        },
        {
          "x": 64,
          "y": 6,
          "value": 0,
          "name": "cell65 ~ gene7"
        },
        {
          "x": 64,
          "y": 7,
          "value": 10,
          "name": "cell65 ~ gene8"
        },
        {
          "x": 64,
          "y": 8,
          "value": 2,
          "name": "cell65 ~ gene9"
        },
        {
          "x": 64,
          "y": 9,
          "value": 2,
          "name": "cell65 ~ gene10"
        },
        {
          "x": 64,
          "y": 10,
          "value": 13,
          "name": "cell65 ~ gene11"
        },
        {
          "x": 64,
          "y": 11,
          "value": 0,
          "name": "cell65 ~ gene12"
        },
        {
          "x": 64,
          "y": 12,
          "value": 4,
          "name": "cell65 ~ gene13"
        },
        {
          "x": 64,
          "y": 13,
          "value": 0,
          "name": "cell65 ~ gene14"
        },
        {
          "x": 64,
          "y": 14,
          "value": 0,
          "name": "cell65 ~ gene15"
        },
        {
          "x": 64,
          "y": 15,
          "value": 3,
          "name": "cell65 ~ gene16"
        },
        {
          "x": 64,
          "y": 16,
          "value": 0,
          "name": "cell65 ~ gene17"
        },
        {
          "x": 64,
          "y": 17,
          "value": 5,
          "name": "cell65 ~ gene18"
        },
        {
          "x": 64,
          "y": 18,
          "value": 0,
          "name": "cell65 ~ gene19"
        },
        {
          "x": 64,
          "y": 19,
          "value": 0,
          "name": "cell65 ~ gene20"
        },
        {
          "x": 64,
          "y": 20,
          "value": 0,
          "name": "cell65 ~ gene21"
        },
        {
          "x": 64,
          "y": 21,
          "value": 0,
          "name": "cell65 ~ gene22"
        },
        {
          "x": 64,
          "y": 22,
          "value": 0,
          "name": "cell65 ~ gene23"
        },
        {
          "x": 64,
          "y": 23,
          "value": 0,
          "name": "cell65 ~ gene24"
        },
        {
          "x": 64,
          "y": 24,
          "value": 0,
          "name": "cell65 ~ gene25"
        },
        {
          "x": 64,
          "y": 25,
          "value": 5,
          "name": "cell65 ~ gene26"
        },
        {
          "x": 64,
          "y": 26,
          "value": 13,
          "name": "cell65 ~ gene27"
        },
        {
          "x": 64,
          "y": 27,
          "value": 0,
          "name": "cell65 ~ gene28"
        },
        {
          "x": 64,
          "y": 28,
          "value": 7,
          "name": "cell65 ~ gene29"
        },
        {
          "x": 64,
          "y": 29,
          "value": 0,
          "name": "cell65 ~ gene30"
        },
        {
          "x": 64,
          "y": 30,
          "value": 0,
          "name": "cell65 ~ gene31"
        },
        {
          "x": 64,
          "y": 31,
          "value": 0,
          "name": "cell65 ~ gene32"
        },
        {
          "x": 64,
          "y": 32,
          "value": 14,
          "name": "cell65 ~ gene33"
        },
        {
          "x": 64,
          "y": 33,
          "value": 0,
          "name": "cell65 ~ gene34"
        },
        {
          "x": 64,
          "y": 34,
          "value": 19,
          "name": "cell65 ~ gene35"
        },
        {
          "x": 64,
          "y": 35,
          "value": 0,
          "name": "cell65 ~ gene36"
        },
        {
          "x": 64,
          "y": 36,
          "value": 0,
          "name": "cell65 ~ gene37"
        },
        {
          "x": 64,
          "y": 37,
          "value": 0,
          "name": "cell65 ~ gene38"
        },
        {
          "x": 64,
          "y": 38,
          "value": 0,
          "name": "cell65 ~ gene39"
        },
        {
          "x": 64,
          "y": 39,
          "value": 0,
          "name": "cell65 ~ gene40"
        },
        {
          "x": 64,
          "y": 40,
          "value": 17,
          "name": "cell65 ~ gene41"
        },
        {
          "x": 64,
          "y": 41,
          "value": 37,
          "name": "cell65 ~ gene42"
        },
        {
          "x": 64,
          "y": 42,
          "value": 9,
          "name": "cell65 ~ gene43"
        },
        {
          "x": 64,
          "y": 43,
          "value": 0,
          "name": "cell65 ~ gene44"
        },
        {
          "x": 64,
          "y": 44,
          "value": 33,
          "name": "cell65 ~ gene45"
        },
        {
          "x": 64,
          "y": 45,
          "value": 24,
          "name": "cell65 ~ gene46"
        },
        {
          "x": 64,
          "y": 46,
          "value": 17,
          "name": "cell65 ~ gene47"
        },
        {
          "x": 64,
          "y": 47,
          "value": 22,
          "name": "cell65 ~ gene48"
        },
        {
          "x": 64,
          "y": 48,
          "value": 20,
          "name": "cell65 ~ gene49"
        },
        {
          "x": 64,
          "y": 49,
          "value": 4,
          "name": "cell65 ~ gene50"
        },
        {
          "x": 64,
          "y": 50,
          "value": 23,
          "name": "cell65 ~ gene51"
        },
        {
          "x": 64,
          "y": 51,
          "value": 20,
          "name": "cell65 ~ gene52"
        },
        {
          "x": 64,
          "y": 52,
          "value": 0,
          "name": "cell65 ~ gene53"
        },
        {
          "x": 64,
          "y": 53,
          "value": 10,
          "name": "cell65 ~ gene54"
        },
        {
          "x": 64,
          "y": 54,
          "value": 36,
          "name": "cell65 ~ gene55"
        },
        {
          "x": 64,
          "y": 55,
          "value": 29,
          "name": "cell65 ~ gene56"
        },
        {
          "x": 64,
          "y": 56,
          "value": 12,
          "name": "cell65 ~ gene57"
        },
        {
          "x": 64,
          "y": 57,
          "value": 17,
          "name": "cell65 ~ gene58"
        },
        {
          "x": 64,
          "y": 58,
          "value": 23,
          "name": "cell65 ~ gene59"
        },
        {
          "x": 64,
          "y": 59,
          "value": 0,
          "name": "cell65 ~ gene60"
        },
        {
          "x": 64,
          "y": 60,
          "value": 0,
          "name": "cell65 ~ gene61"
        },
        {
          "x": 64,
          "y": 61,
          "value": 9,
          "name": "cell65 ~ gene62"
        },
        {
          "x": 64,
          "y": 62,
          "value": 0,
          "name": "cell65 ~ gene63"
        },
        {
          "x": 64,
          "y": 63,
          "value": 0,
          "name": "cell65 ~ gene64"
        },
        {
          "x": 64,
          "y": 64,
          "value": 0,
          "name": "cell65 ~ gene65"
        },
        {
          "x": 64,
          "y": 65,
          "value": 1,
          "name": "cell65 ~ gene66"
        },
        {
          "x": 64,
          "y": 66,
          "value": 3,
          "name": "cell65 ~ gene67"
        },
        {
          "x": 64,
          "y": 67,
          "value": 0,
          "name": "cell65 ~ gene68"
        },
        {
          "x": 64,
          "y": 68,
          "value": 3,
          "name": "cell65 ~ gene69"
        },
        {
          "x": 64,
          "y": 69,
          "value": 0,
          "name": "cell65 ~ gene70"
        },
        {
          "x": 64,
          "y": 70,
          "value": 0,
          "name": "cell65 ~ gene71"
        },
        {
          "x": 64,
          "y": 71,
          "value": 0,
          "name": "cell65 ~ gene72"
        },
        {
          "x": 64,
          "y": 72,
          "value": 6,
          "name": "cell65 ~ gene73"
        },
        {
          "x": 64,
          "y": 73,
          "value": 8,
          "name": "cell65 ~ gene74"
        },
        {
          "x": 64,
          "y": 74,
          "value": 13,
          "name": "cell65 ~ gene75"
        },
        {
          "x": 64,
          "y": 75,
          "value": 12,
          "name": "cell65 ~ gene76"
        },
        {
          "x": 64,
          "y": 76,
          "value": 10,
          "name": "cell65 ~ gene77"
        },
        {
          "x": 64,
          "y": 77,
          "value": 0,
          "name": "cell65 ~ gene78"
        },
        {
          "x": 64,
          "y": 78,
          "value": 10,
          "name": "cell65 ~ gene79"
        },
        {
          "x": 64,
          "y": 79,
          "value": 0,
          "name": "cell65 ~ gene80"
        },
        {
          "x": 64,
          "y": 80,
          "value": 0,
          "name": "cell65 ~ gene81"
        },
        {
          "x": 64,
          "y": 81,
          "value": 0,
          "name": "cell65 ~ gene82"
        },
        {
          "x": 64,
          "y": 82,
          "value": 0,
          "name": "cell65 ~ gene83"
        },
        {
          "x": 64,
          "y": 83,
          "value": 0,
          "name": "cell65 ~ gene84"
        },
        {
          "x": 64,
          "y": 84,
          "value": 0,
          "name": "cell65 ~ gene85"
        },
        {
          "x": 64,
          "y": 85,
          "value": 0,
          "name": "cell65 ~ gene86"
        },
        {
          "x": 64,
          "y": 86,
          "value": 0,
          "name": "cell65 ~ gene87"
        },
        {
          "x": 64,
          "y": 87,
          "value": 0,
          "name": "cell65 ~ gene88"
        },
        {
          "x": 64,
          "y": 88,
          "value": 2,
          "name": "cell65 ~ gene89"
        },
        {
          "x": 64,
          "y": 89,
          "value": 0,
          "name": "cell65 ~ gene90"
        },
        {
          "x": 64,
          "y": 90,
          "value": 5,
          "name": "cell65 ~ gene91"
        },
        {
          "x": 64,
          "y": 91,
          "value": 0,
          "name": "cell65 ~ gene92"
        },
        {
          "x": 64,
          "y": 92,
          "value": 12,
          "name": "cell65 ~ gene93"
        },
        {
          "x": 64,
          "y": 93,
          "value": 0,
          "name": "cell65 ~ gene94"
        },
        {
          "x": 64,
          "y": 94,
          "value": 0,
          "name": "cell65 ~ gene95"
        },
        {
          "x": 64,
          "y": 95,
          "value": 6,
          "name": "cell65 ~ gene96"
        },
        {
          "x": 64,
          "y": 96,
          "value": 0,
          "name": "cell65 ~ gene97"
        },
        {
          "x": 64,
          "y": 97,
          "value": 0,
          "name": "cell65 ~ gene98"
        },
        {
          "x": 64,
          "y": 98,
          "value": 1,
          "name": "cell65 ~ gene99"
        },
        {
          "x": 64,
          "y": 99,
          "value": 0,
          "name": "cell65 ~ gene100"
        },
        {
          "x": 65,
          "y": 0,
          "value": 11,
          "name": "cell66 ~ gene1"
        },
        {
          "x": 65,
          "y": 1,
          "value": 14,
          "name": "cell66 ~ gene2"
        },
        {
          "x": 65,
          "y": 2,
          "value": 0,
          "name": "cell66 ~ gene3"
        },
        {
          "x": 65,
          "y": 3,
          "value": 12,
          "name": "cell66 ~ gene4"
        },
        {
          "x": 65,
          "y": 4,
          "value": 26,
          "name": "cell66 ~ gene5"
        },
        {
          "x": 65,
          "y": 5,
          "value": 3,
          "name": "cell66 ~ gene6"
        },
        {
          "x": 65,
          "y": 6,
          "value": 5,
          "name": "cell66 ~ gene7"
        },
        {
          "x": 65,
          "y": 7,
          "value": 5,
          "name": "cell66 ~ gene8"
        },
        {
          "x": 65,
          "y": 8,
          "value": 0,
          "name": "cell66 ~ gene9"
        },
        {
          "x": 65,
          "y": 9,
          "value": 0,
          "name": "cell66 ~ gene10"
        },
        {
          "x": 65,
          "y": 10,
          "value": 1,
          "name": "cell66 ~ gene11"
        },
        {
          "x": 65,
          "y": 11,
          "value": 0,
          "name": "cell66 ~ gene12"
        },
        {
          "x": 65,
          "y": 12,
          "value": 2,
          "name": "cell66 ~ gene13"
        },
        {
          "x": 65,
          "y": 13,
          "value": 2,
          "name": "cell66 ~ gene14"
        },
        {
          "x": 65,
          "y": 14,
          "value": 6,
          "name": "cell66 ~ gene15"
        },
        {
          "x": 65,
          "y": 15,
          "value": 15,
          "name": "cell66 ~ gene16"
        },
        {
          "x": 65,
          "y": 16,
          "value": 0,
          "name": "cell66 ~ gene17"
        },
        {
          "x": 65,
          "y": 17,
          "value": 3,
          "name": "cell66 ~ gene18"
        },
        {
          "x": 65,
          "y": 18,
          "value": 0,
          "name": "cell66 ~ gene19"
        },
        {
          "x": 65,
          "y": 19,
          "value": 10,
          "name": "cell66 ~ gene20"
        },
        {
          "x": 65,
          "y": 20,
          "value": 11,
          "name": "cell66 ~ gene21"
        },
        {
          "x": 65,
          "y": 21,
          "value": 11,
          "name": "cell66 ~ gene22"
        },
        {
          "x": 65,
          "y": 22,
          "value": 0,
          "name": "cell66 ~ gene23"
        },
        {
          "x": 65,
          "y": 23,
          "value": 0,
          "name": "cell66 ~ gene24"
        },
        {
          "x": 65,
          "y": 24,
          "value": 7,
          "name": "cell66 ~ gene25"
        },
        {
          "x": 65,
          "y": 25,
          "value": 3,
          "name": "cell66 ~ gene26"
        },
        {
          "x": 65,
          "y": 26,
          "value": 2,
          "name": "cell66 ~ gene27"
        },
        {
          "x": 65,
          "y": 27,
          "value": 8,
          "name": "cell66 ~ gene28"
        },
        {
          "x": 65,
          "y": 28,
          "value": 4,
          "name": "cell66 ~ gene29"
        },
        {
          "x": 65,
          "y": 29,
          "value": 18,
          "name": "cell66 ~ gene30"
        },
        {
          "x": 65,
          "y": 30,
          "value": 13,
          "name": "cell66 ~ gene31"
        },
        {
          "x": 65,
          "y": 31,
          "value": 15,
          "name": "cell66 ~ gene32"
        },
        {
          "x": 65,
          "y": 32,
          "value": 0,
          "name": "cell66 ~ gene33"
        },
        {
          "x": 65,
          "y": 33,
          "value": 13,
          "name": "cell66 ~ gene34"
        },
        {
          "x": 65,
          "y": 34,
          "value": 0,
          "name": "cell66 ~ gene35"
        },
        {
          "x": 65,
          "y": 35,
          "value": 0,
          "name": "cell66 ~ gene36"
        },
        {
          "x": 65,
          "y": 36,
          "value": 0,
          "name": "cell66 ~ gene37"
        },
        {
          "x": 65,
          "y": 37,
          "value": 7,
          "name": "cell66 ~ gene38"
        },
        {
          "x": 65,
          "y": 38,
          "value": 0,
          "name": "cell66 ~ gene39"
        },
        {
          "x": 65,
          "y": 39,
          "value": 0,
          "name": "cell66 ~ gene40"
        },
        {
          "x": 65,
          "y": 40,
          "value": 0,
          "name": "cell66 ~ gene41"
        },
        {
          "x": 65,
          "y": 41,
          "value": 17,
          "name": "cell66 ~ gene42"
        },
        {
          "x": 65,
          "y": 42,
          "value": 0,
          "name": "cell66 ~ gene43"
        },
        {
          "x": 65,
          "y": 43,
          "value": 10,
          "name": "cell66 ~ gene44"
        },
        {
          "x": 65,
          "y": 44,
          "value": 0,
          "name": "cell66 ~ gene45"
        },
        {
          "x": 65,
          "y": 45,
          "value": 13,
          "name": "cell66 ~ gene46"
        },
        {
          "x": 65,
          "y": 46,
          "value": 17,
          "name": "cell66 ~ gene47"
        },
        {
          "x": 65,
          "y": 47,
          "value": 3,
          "name": "cell66 ~ gene48"
        },
        {
          "x": 65,
          "y": 48,
          "value": 0,
          "name": "cell66 ~ gene49"
        },
        {
          "x": 65,
          "y": 49,
          "value": 0,
          "name": "cell66 ~ gene50"
        },
        {
          "x": 65,
          "y": 50,
          "value": 21,
          "name": "cell66 ~ gene51"
        },
        {
          "x": 65,
          "y": 51,
          "value": 4,
          "name": "cell66 ~ gene52"
        },
        {
          "x": 65,
          "y": 52,
          "value": 6,
          "name": "cell66 ~ gene53"
        },
        {
          "x": 65,
          "y": 53,
          "value": 0,
          "name": "cell66 ~ gene54"
        },
        {
          "x": 65,
          "y": 54,
          "value": 13,
          "name": "cell66 ~ gene55"
        },
        {
          "x": 65,
          "y": 55,
          "value": 18,
          "name": "cell66 ~ gene56"
        },
        {
          "x": 65,
          "y": 56,
          "value": 13,
          "name": "cell66 ~ gene57"
        },
        {
          "x": 65,
          "y": 57,
          "value": 16,
          "name": "cell66 ~ gene58"
        },
        {
          "x": 65,
          "y": 58,
          "value": 17,
          "name": "cell66 ~ gene59"
        },
        {
          "x": 65,
          "y": 59,
          "value": 0,
          "name": "cell66 ~ gene60"
        },
        {
          "x": 65,
          "y": 60,
          "value": 0,
          "name": "cell66 ~ gene61"
        },
        {
          "x": 65,
          "y": 61,
          "value": 0,
          "name": "cell66 ~ gene62"
        },
        {
          "x": 65,
          "y": 62,
          "value": 4,
          "name": "cell66 ~ gene63"
        },
        {
          "x": 65,
          "y": 63,
          "value": 0,
          "name": "cell66 ~ gene64"
        },
        {
          "x": 65,
          "y": 64,
          "value": 0,
          "name": "cell66 ~ gene65"
        },
        {
          "x": 65,
          "y": 65,
          "value": 11,
          "name": "cell66 ~ gene66"
        },
        {
          "x": 65,
          "y": 66,
          "value": 6,
          "name": "cell66 ~ gene67"
        },
        {
          "x": 65,
          "y": 67,
          "value": 0,
          "name": "cell66 ~ gene68"
        },
        {
          "x": 65,
          "y": 68,
          "value": 19,
          "name": "cell66 ~ gene69"
        },
        {
          "x": 65,
          "y": 69,
          "value": 0,
          "name": "cell66 ~ gene70"
        },
        {
          "x": 65,
          "y": 70,
          "value": 2,
          "name": "cell66 ~ gene71"
        },
        {
          "x": 65,
          "y": 71,
          "value": 6,
          "name": "cell66 ~ gene72"
        },
        {
          "x": 65,
          "y": 72,
          "value": 14,
          "name": "cell66 ~ gene73"
        },
        {
          "x": 65,
          "y": 73,
          "value": 0,
          "name": "cell66 ~ gene74"
        },
        {
          "x": 65,
          "y": 74,
          "value": 0,
          "name": "cell66 ~ gene75"
        },
        {
          "x": 65,
          "y": 75,
          "value": 0,
          "name": "cell66 ~ gene76"
        },
        {
          "x": 65,
          "y": 76,
          "value": 0,
          "name": "cell66 ~ gene77"
        },
        {
          "x": 65,
          "y": 77,
          "value": 17,
          "name": "cell66 ~ gene78"
        },
        {
          "x": 65,
          "y": 78,
          "value": 2,
          "name": "cell66 ~ gene79"
        },
        {
          "x": 65,
          "y": 79,
          "value": 5,
          "name": "cell66 ~ gene80"
        },
        {
          "x": 65,
          "y": 80,
          "value": 9,
          "name": "cell66 ~ gene81"
        },
        {
          "x": 65,
          "y": 81,
          "value": 0,
          "name": "cell66 ~ gene82"
        },
        {
          "x": 65,
          "y": 82,
          "value": 5,
          "name": "cell66 ~ gene83"
        },
        {
          "x": 65,
          "y": 83,
          "value": 21,
          "name": "cell66 ~ gene84"
        },
        {
          "x": 65,
          "y": 84,
          "value": 0,
          "name": "cell66 ~ gene85"
        },
        {
          "x": 65,
          "y": 85,
          "value": 0,
          "name": "cell66 ~ gene86"
        },
        {
          "x": 65,
          "y": 86,
          "value": 0,
          "name": "cell66 ~ gene87"
        },
        {
          "x": 65,
          "y": 87,
          "value": 0,
          "name": "cell66 ~ gene88"
        },
        {
          "x": 65,
          "y": 88,
          "value": 14,
          "name": "cell66 ~ gene89"
        },
        {
          "x": 65,
          "y": 89,
          "value": 0,
          "name": "cell66 ~ gene90"
        },
        {
          "x": 65,
          "y": 90,
          "value": 3,
          "name": "cell66 ~ gene91"
        },
        {
          "x": 65,
          "y": 91,
          "value": 0,
          "name": "cell66 ~ gene92"
        },
        {
          "x": 65,
          "y": 92,
          "value": 5,
          "name": "cell66 ~ gene93"
        },
        {
          "x": 65,
          "y": 93,
          "value": 11,
          "name": "cell66 ~ gene94"
        },
        {
          "x": 65,
          "y": 94,
          "value": 7,
          "name": "cell66 ~ gene95"
        },
        {
          "x": 65,
          "y": 95,
          "value": 3,
          "name": "cell66 ~ gene96"
        },
        {
          "x": 65,
          "y": 96,
          "value": 0,
          "name": "cell66 ~ gene97"
        },
        {
          "x": 65,
          "y": 97,
          "value": 0,
          "name": "cell66 ~ gene98"
        },
        {
          "x": 65,
          "y": 98,
          "value": 14,
          "name": "cell66 ~ gene99"
        },
        {
          "x": 65,
          "y": 99,
          "value": 0,
          "name": "cell66 ~ gene100"
        },
        {
          "x": 66,
          "y": 0,
          "value": 0,
          "name": "cell67 ~ gene1"
        },
        {
          "x": 66,
          "y": 1,
          "value": 4,
          "name": "cell67 ~ gene2"
        },
        {
          "x": 66,
          "y": 2,
          "value": 3,
          "name": "cell67 ~ gene3"
        },
        {
          "x": 66,
          "y": 3,
          "value": 0,
          "name": "cell67 ~ gene4"
        },
        {
          "x": 66,
          "y": 4,
          "value": 12,
          "name": "cell67 ~ gene5"
        },
        {
          "x": 66,
          "y": 5,
          "value": 0,
          "name": "cell67 ~ gene6"
        },
        {
          "x": 66,
          "y": 6,
          "value": 18,
          "name": "cell67 ~ gene7"
        },
        {
          "x": 66,
          "y": 7,
          "value": 0,
          "name": "cell67 ~ gene8"
        },
        {
          "x": 66,
          "y": 8,
          "value": 5,
          "name": "cell67 ~ gene9"
        },
        {
          "x": 66,
          "y": 9,
          "value": 0,
          "name": "cell67 ~ gene10"
        },
        {
          "x": 66,
          "y": 10,
          "value": 5,
          "name": "cell67 ~ gene11"
        },
        {
          "x": 66,
          "y": 11,
          "value": 22,
          "name": "cell67 ~ gene12"
        },
        {
          "x": 66,
          "y": 12,
          "value": 8,
          "name": "cell67 ~ gene13"
        },
        {
          "x": 66,
          "y": 13,
          "value": 14,
          "name": "cell67 ~ gene14"
        },
        {
          "x": 66,
          "y": 14,
          "value": 36,
          "name": "cell67 ~ gene15"
        },
        {
          "x": 66,
          "y": 15,
          "value": 8,
          "name": "cell67 ~ gene16"
        },
        {
          "x": 66,
          "y": 16,
          "value": 0,
          "name": "cell67 ~ gene17"
        },
        {
          "x": 66,
          "y": 17,
          "value": 0,
          "name": "cell67 ~ gene18"
        },
        {
          "x": 66,
          "y": 18,
          "value": 0,
          "name": "cell67 ~ gene19"
        },
        {
          "x": 66,
          "y": 19,
          "value": 0,
          "name": "cell67 ~ gene20"
        },
        {
          "x": 66,
          "y": 20,
          "value": 26,
          "name": "cell67 ~ gene21"
        },
        {
          "x": 66,
          "y": 21,
          "value": 0,
          "name": "cell67 ~ gene22"
        },
        {
          "x": 66,
          "y": 22,
          "value": 4,
          "name": "cell67 ~ gene23"
        },
        {
          "x": 66,
          "y": 23,
          "value": 9,
          "name": "cell67 ~ gene24"
        },
        {
          "x": 66,
          "y": 24,
          "value": 0,
          "name": "cell67 ~ gene25"
        },
        {
          "x": 66,
          "y": 25,
          "value": 0,
          "name": "cell67 ~ gene26"
        },
        {
          "x": 66,
          "y": 26,
          "value": 1,
          "name": "cell67 ~ gene27"
        },
        {
          "x": 66,
          "y": 27,
          "value": 0,
          "name": "cell67 ~ gene28"
        },
        {
          "x": 66,
          "y": 28,
          "value": 0,
          "name": "cell67 ~ gene29"
        },
        {
          "x": 66,
          "y": 29,
          "value": 0,
          "name": "cell67 ~ gene30"
        },
        {
          "x": 66,
          "y": 30,
          "value": 7,
          "name": "cell67 ~ gene31"
        },
        {
          "x": 66,
          "y": 31,
          "value": 1,
          "name": "cell67 ~ gene32"
        },
        {
          "x": 66,
          "y": 32,
          "value": 0,
          "name": "cell67 ~ gene33"
        },
        {
          "x": 66,
          "y": 33,
          "value": 13,
          "name": "cell67 ~ gene34"
        },
        {
          "x": 66,
          "y": 34,
          "value": 12,
          "name": "cell67 ~ gene35"
        },
        {
          "x": 66,
          "y": 35,
          "value": 0,
          "name": "cell67 ~ gene36"
        },
        {
          "x": 66,
          "y": 36,
          "value": 0,
          "name": "cell67 ~ gene37"
        },
        {
          "x": 66,
          "y": 37,
          "value": 14,
          "name": "cell67 ~ gene38"
        },
        {
          "x": 66,
          "y": 38,
          "value": 5,
          "name": "cell67 ~ gene39"
        },
        {
          "x": 66,
          "y": 39,
          "value": 0,
          "name": "cell67 ~ gene40"
        },
        {
          "x": 66,
          "y": 40,
          "value": 7,
          "name": "cell67 ~ gene41"
        },
        {
          "x": 66,
          "y": 41,
          "value": 20,
          "name": "cell67 ~ gene42"
        },
        {
          "x": 66,
          "y": 42,
          "value": 0,
          "name": "cell67 ~ gene43"
        },
        {
          "x": 66,
          "y": 43,
          "value": 8,
          "name": "cell67 ~ gene44"
        },
        {
          "x": 66,
          "y": 44,
          "value": 3,
          "name": "cell67 ~ gene45"
        },
        {
          "x": 66,
          "y": 45,
          "value": 18,
          "name": "cell67 ~ gene46"
        },
        {
          "x": 66,
          "y": 46,
          "value": 19,
          "name": "cell67 ~ gene47"
        },
        {
          "x": 66,
          "y": 47,
          "value": 26,
          "name": "cell67 ~ gene48"
        },
        {
          "x": 66,
          "y": 48,
          "value": 0,
          "name": "cell67 ~ gene49"
        },
        {
          "x": 66,
          "y": 49,
          "value": 11,
          "name": "cell67 ~ gene50"
        },
        {
          "x": 66,
          "y": 50,
          "value": 29,
          "name": "cell67 ~ gene51"
        },
        {
          "x": 66,
          "y": 51,
          "value": 18,
          "name": "cell67 ~ gene52"
        },
        {
          "x": 66,
          "y": 52,
          "value": 9,
          "name": "cell67 ~ gene53"
        },
        {
          "x": 66,
          "y": 53,
          "value": 9,
          "name": "cell67 ~ gene54"
        },
        {
          "x": 66,
          "y": 54,
          "value": 15,
          "name": "cell67 ~ gene55"
        },
        {
          "x": 66,
          "y": 55,
          "value": 37,
          "name": "cell67 ~ gene56"
        },
        {
          "x": 66,
          "y": 56,
          "value": 2,
          "name": "cell67 ~ gene57"
        },
        {
          "x": 66,
          "y": 57,
          "value": 32,
          "name": "cell67 ~ gene58"
        },
        {
          "x": 66,
          "y": 58,
          "value": 11,
          "name": "cell67 ~ gene59"
        },
        {
          "x": 66,
          "y": 59,
          "value": 0,
          "name": "cell67 ~ gene60"
        },
        {
          "x": 66,
          "y": 60,
          "value": 0,
          "name": "cell67 ~ gene61"
        },
        {
          "x": 66,
          "y": 61,
          "value": 5,
          "name": "cell67 ~ gene62"
        },
        {
          "x": 66,
          "y": 62,
          "value": 9,
          "name": "cell67 ~ gene63"
        },
        {
          "x": 66,
          "y": 63,
          "value": 3,
          "name": "cell67 ~ gene64"
        },
        {
          "x": 66,
          "y": 64,
          "value": 0,
          "name": "cell67 ~ gene65"
        },
        {
          "x": 66,
          "y": 65,
          "value": 4,
          "name": "cell67 ~ gene66"
        },
        {
          "x": 66,
          "y": 66,
          "value": 8,
          "name": "cell67 ~ gene67"
        },
        {
          "x": 66,
          "y": 67,
          "value": 0,
          "name": "cell67 ~ gene68"
        },
        {
          "x": 66,
          "y": 68,
          "value": 3,
          "name": "cell67 ~ gene69"
        },
        {
          "x": 66,
          "y": 69,
          "value": 11,
          "name": "cell67 ~ gene70"
        },
        {
          "x": 66,
          "y": 70,
          "value": 0,
          "name": "cell67 ~ gene71"
        },
        {
          "x": 66,
          "y": 71,
          "value": 6,
          "name": "cell67 ~ gene72"
        },
        {
          "x": 66,
          "y": 72,
          "value": 2,
          "name": "cell67 ~ gene73"
        },
        {
          "x": 66,
          "y": 73,
          "value": 13,
          "name": "cell67 ~ gene74"
        },
        {
          "x": 66,
          "y": 74,
          "value": 10,
          "name": "cell67 ~ gene75"
        },
        {
          "x": 66,
          "y": 75,
          "value": 3,
          "name": "cell67 ~ gene76"
        },
        {
          "x": 66,
          "y": 76,
          "value": 4,
          "name": "cell67 ~ gene77"
        },
        {
          "x": 66,
          "y": 77,
          "value": 11,
          "name": "cell67 ~ gene78"
        },
        {
          "x": 66,
          "y": 78,
          "value": 0,
          "name": "cell67 ~ gene79"
        },
        {
          "x": 66,
          "y": 79,
          "value": 0,
          "name": "cell67 ~ gene80"
        },
        {
          "x": 66,
          "y": 80,
          "value": 0,
          "name": "cell67 ~ gene81"
        },
        {
          "x": 66,
          "y": 81,
          "value": 0,
          "name": "cell67 ~ gene82"
        },
        {
          "x": 66,
          "y": 82,
          "value": 4,
          "name": "cell67 ~ gene83"
        },
        {
          "x": 66,
          "y": 83,
          "value": 1,
          "name": "cell67 ~ gene84"
        },
        {
          "x": 66,
          "y": 84,
          "value": 11,
          "name": "cell67 ~ gene85"
        },
        {
          "x": 66,
          "y": 85,
          "value": 0,
          "name": "cell67 ~ gene86"
        },
        {
          "x": 66,
          "y": 86,
          "value": 0,
          "name": "cell67 ~ gene87"
        },
        {
          "x": 66,
          "y": 87,
          "value": 4,
          "name": "cell67 ~ gene88"
        },
        {
          "x": 66,
          "y": 88,
          "value": 20,
          "name": "cell67 ~ gene89"
        },
        {
          "x": 66,
          "y": 89,
          "value": 4,
          "name": "cell67 ~ gene90"
        },
        {
          "x": 66,
          "y": 90,
          "value": 0,
          "name": "cell67 ~ gene91"
        },
        {
          "x": 66,
          "y": 91,
          "value": 0,
          "name": "cell67 ~ gene92"
        },
        {
          "x": 66,
          "y": 92,
          "value": 0,
          "name": "cell67 ~ gene93"
        },
        {
          "x": 66,
          "y": 93,
          "value": 14,
          "name": "cell67 ~ gene94"
        },
        {
          "x": 66,
          "y": 94,
          "value": 0,
          "name": "cell67 ~ gene95"
        },
        {
          "x": 66,
          "y": 95,
          "value": 0,
          "name": "cell67 ~ gene96"
        },
        {
          "x": 66,
          "y": 96,
          "value": 20,
          "name": "cell67 ~ gene97"
        },
        {
          "x": 66,
          "y": 97,
          "value": 9,
          "name": "cell67 ~ gene98"
        },
        {
          "x": 66,
          "y": 98,
          "value": 0,
          "name": "cell67 ~ gene99"
        },
        {
          "x": 66,
          "y": 99,
          "value": 7,
          "name": "cell67 ~ gene100"
        },
        {
          "x": 67,
          "y": 0,
          "value": 17,
          "name": "cell68 ~ gene1"
        },
        {
          "x": 67,
          "y": 1,
          "value": 0,
          "name": "cell68 ~ gene2"
        },
        {
          "x": 67,
          "y": 2,
          "value": 5,
          "name": "cell68 ~ gene3"
        },
        {
          "x": 67,
          "y": 3,
          "value": 5,
          "name": "cell68 ~ gene4"
        },
        {
          "x": 67,
          "y": 4,
          "value": 9,
          "name": "cell68 ~ gene5"
        },
        {
          "x": 67,
          "y": 5,
          "value": 0,
          "name": "cell68 ~ gene6"
        },
        {
          "x": 67,
          "y": 6,
          "value": 0,
          "name": "cell68 ~ gene7"
        },
        {
          "x": 67,
          "y": 7,
          "value": 0,
          "name": "cell68 ~ gene8"
        },
        {
          "x": 67,
          "y": 8,
          "value": 8,
          "name": "cell68 ~ gene9"
        },
        {
          "x": 67,
          "y": 9,
          "value": 0,
          "name": "cell68 ~ gene10"
        },
        {
          "x": 67,
          "y": 10,
          "value": 11,
          "name": "cell68 ~ gene11"
        },
        {
          "x": 67,
          "y": 11,
          "value": 0,
          "name": "cell68 ~ gene12"
        },
        {
          "x": 67,
          "y": 12,
          "value": 11,
          "name": "cell68 ~ gene13"
        },
        {
          "x": 67,
          "y": 13,
          "value": 0,
          "name": "cell68 ~ gene14"
        },
        {
          "x": 67,
          "y": 14,
          "value": 0,
          "name": "cell68 ~ gene15"
        },
        {
          "x": 67,
          "y": 15,
          "value": 5,
          "name": "cell68 ~ gene16"
        },
        {
          "x": 67,
          "y": 16,
          "value": 0,
          "name": "cell68 ~ gene17"
        },
        {
          "x": 67,
          "y": 17,
          "value": 1,
          "name": "cell68 ~ gene18"
        },
        {
          "x": 67,
          "y": 18,
          "value": 0,
          "name": "cell68 ~ gene19"
        },
        {
          "x": 67,
          "y": 19,
          "value": 0,
          "name": "cell68 ~ gene20"
        },
        {
          "x": 67,
          "y": 20,
          "value": 0,
          "name": "cell68 ~ gene21"
        },
        {
          "x": 67,
          "y": 21,
          "value": 2,
          "name": "cell68 ~ gene22"
        },
        {
          "x": 67,
          "y": 22,
          "value": 0,
          "name": "cell68 ~ gene23"
        },
        {
          "x": 67,
          "y": 23,
          "value": 0,
          "name": "cell68 ~ gene24"
        },
        {
          "x": 67,
          "y": 24,
          "value": 0,
          "name": "cell68 ~ gene25"
        },
        {
          "x": 67,
          "y": 25,
          "value": 0,
          "name": "cell68 ~ gene26"
        },
        {
          "x": 67,
          "y": 26,
          "value": 13,
          "name": "cell68 ~ gene27"
        },
        {
          "x": 67,
          "y": 27,
          "value": 0,
          "name": "cell68 ~ gene28"
        },
        {
          "x": 67,
          "y": 28,
          "value": 11,
          "name": "cell68 ~ gene29"
        },
        {
          "x": 67,
          "y": 29,
          "value": 10,
          "name": "cell68 ~ gene30"
        },
        {
          "x": 67,
          "y": 30,
          "value": 0,
          "name": "cell68 ~ gene31"
        },
        {
          "x": 67,
          "y": 31,
          "value": 11,
          "name": "cell68 ~ gene32"
        },
        {
          "x": 67,
          "y": 32,
          "value": 0,
          "name": "cell68 ~ gene33"
        },
        {
          "x": 67,
          "y": 33,
          "value": 0,
          "name": "cell68 ~ gene34"
        },
        {
          "x": 67,
          "y": 34,
          "value": 0,
          "name": "cell68 ~ gene35"
        },
        {
          "x": 67,
          "y": 35,
          "value": 12,
          "name": "cell68 ~ gene36"
        },
        {
          "x": 67,
          "y": 36,
          "value": 0,
          "name": "cell68 ~ gene37"
        },
        {
          "x": 67,
          "y": 37,
          "value": 0,
          "name": "cell68 ~ gene38"
        },
        {
          "x": 67,
          "y": 38,
          "value": 0,
          "name": "cell68 ~ gene39"
        },
        {
          "x": 67,
          "y": 39,
          "value": 14,
          "name": "cell68 ~ gene40"
        },
        {
          "x": 67,
          "y": 40,
          "value": 19,
          "name": "cell68 ~ gene41"
        },
        {
          "x": 67,
          "y": 41,
          "value": 9,
          "name": "cell68 ~ gene42"
        },
        {
          "x": 67,
          "y": 42,
          "value": 0,
          "name": "cell68 ~ gene43"
        },
        {
          "x": 67,
          "y": 43,
          "value": 0,
          "name": "cell68 ~ gene44"
        },
        {
          "x": 67,
          "y": 44,
          "value": 0,
          "name": "cell68 ~ gene45"
        },
        {
          "x": 67,
          "y": 45,
          "value": 15,
          "name": "cell68 ~ gene46"
        },
        {
          "x": 67,
          "y": 46,
          "value": 6,
          "name": "cell68 ~ gene47"
        },
        {
          "x": 67,
          "y": 47,
          "value": 7,
          "name": "cell68 ~ gene48"
        },
        {
          "x": 67,
          "y": 48,
          "value": 0,
          "name": "cell68 ~ gene49"
        },
        {
          "x": 67,
          "y": 49,
          "value": 9,
          "name": "cell68 ~ gene50"
        },
        {
          "x": 67,
          "y": 50,
          "value": 22,
          "name": "cell68 ~ gene51"
        },
        {
          "x": 67,
          "y": 51,
          "value": 17,
          "name": "cell68 ~ gene52"
        },
        {
          "x": 67,
          "y": 52,
          "value": 8,
          "name": "cell68 ~ gene53"
        },
        {
          "x": 67,
          "y": 53,
          "value": 0,
          "name": "cell68 ~ gene54"
        },
        {
          "x": 67,
          "y": 54,
          "value": 10,
          "name": "cell68 ~ gene55"
        },
        {
          "x": 67,
          "y": 55,
          "value": 32,
          "name": "cell68 ~ gene56"
        },
        {
          "x": 67,
          "y": 56,
          "value": 0,
          "name": "cell68 ~ gene57"
        },
        {
          "x": 67,
          "y": 57,
          "value": 6,
          "name": "cell68 ~ gene58"
        },
        {
          "x": 67,
          "y": 58,
          "value": 5,
          "name": "cell68 ~ gene59"
        },
        {
          "x": 67,
          "y": 59,
          "value": 0,
          "name": "cell68 ~ gene60"
        },
        {
          "x": 67,
          "y": 60,
          "value": 0,
          "name": "cell68 ~ gene61"
        },
        {
          "x": 67,
          "y": 61,
          "value": 0,
          "name": "cell68 ~ gene62"
        },
        {
          "x": 67,
          "y": 62,
          "value": 0,
          "name": "cell68 ~ gene63"
        },
        {
          "x": 67,
          "y": 63,
          "value": 3,
          "name": "cell68 ~ gene64"
        },
        {
          "x": 67,
          "y": 64,
          "value": 5,
          "name": "cell68 ~ gene65"
        },
        {
          "x": 67,
          "y": 65,
          "value": 6,
          "name": "cell68 ~ gene66"
        },
        {
          "x": 67,
          "y": 66,
          "value": 0,
          "name": "cell68 ~ gene67"
        },
        {
          "x": 67,
          "y": 67,
          "value": 0,
          "name": "cell68 ~ gene68"
        },
        {
          "x": 67,
          "y": 68,
          "value": 0,
          "name": "cell68 ~ gene69"
        },
        {
          "x": 67,
          "y": 69,
          "value": 6,
          "name": "cell68 ~ gene70"
        },
        {
          "x": 67,
          "y": 70,
          "value": 0,
          "name": "cell68 ~ gene71"
        },
        {
          "x": 67,
          "y": 71,
          "value": 11,
          "name": "cell68 ~ gene72"
        },
        {
          "x": 67,
          "y": 72,
          "value": 8,
          "name": "cell68 ~ gene73"
        },
        {
          "x": 67,
          "y": 73,
          "value": 0,
          "name": "cell68 ~ gene74"
        },
        {
          "x": 67,
          "y": 74,
          "value": 7,
          "name": "cell68 ~ gene75"
        },
        {
          "x": 67,
          "y": 75,
          "value": 17,
          "name": "cell68 ~ gene76"
        },
        {
          "x": 67,
          "y": 76,
          "value": 9,
          "name": "cell68 ~ gene77"
        },
        {
          "x": 67,
          "y": 77,
          "value": 10,
          "name": "cell68 ~ gene78"
        },
        {
          "x": 67,
          "y": 78,
          "value": 0,
          "name": "cell68 ~ gene79"
        },
        {
          "x": 67,
          "y": 79,
          "value": 13,
          "name": "cell68 ~ gene80"
        },
        {
          "x": 67,
          "y": 80,
          "value": 0,
          "name": "cell68 ~ gene81"
        },
        {
          "x": 67,
          "y": 81,
          "value": 0,
          "name": "cell68 ~ gene82"
        },
        {
          "x": 67,
          "y": 82,
          "value": 0,
          "name": "cell68 ~ gene83"
        },
        {
          "x": 67,
          "y": 83,
          "value": 6,
          "name": "cell68 ~ gene84"
        },
        {
          "x": 67,
          "y": 84,
          "value": 0,
          "name": "cell68 ~ gene85"
        },
        {
          "x": 67,
          "y": 85,
          "value": 11,
          "name": "cell68 ~ gene86"
        },
        {
          "x": 67,
          "y": 86,
          "value": 2,
          "name": "cell68 ~ gene87"
        },
        {
          "x": 67,
          "y": 87,
          "value": 0,
          "name": "cell68 ~ gene88"
        },
        {
          "x": 67,
          "y": 88,
          "value": 0,
          "name": "cell68 ~ gene89"
        },
        {
          "x": 67,
          "y": 89,
          "value": 0,
          "name": "cell68 ~ gene90"
        },
        {
          "x": 67,
          "y": 90,
          "value": 9,
          "name": "cell68 ~ gene91"
        },
        {
          "x": 67,
          "y": 91,
          "value": 0,
          "name": "cell68 ~ gene92"
        },
        {
          "x": 67,
          "y": 92,
          "value": 0,
          "name": "cell68 ~ gene93"
        },
        {
          "x": 67,
          "y": 93,
          "value": 1,
          "name": "cell68 ~ gene94"
        },
        {
          "x": 67,
          "y": 94,
          "value": 0,
          "name": "cell68 ~ gene95"
        },
        {
          "x": 67,
          "y": 95,
          "value": 4,
          "name": "cell68 ~ gene96"
        },
        {
          "x": 67,
          "y": 96,
          "value": 0,
          "name": "cell68 ~ gene97"
        },
        {
          "x": 67,
          "y": 97,
          "value": 15,
          "name": "cell68 ~ gene98"
        },
        {
          "x": 67,
          "y": 98,
          "value": 0,
          "name": "cell68 ~ gene99"
        },
        {
          "x": 67,
          "y": 99,
          "value": 6,
          "name": "cell68 ~ gene100"
        },
        {
          "x": 68,
          "y": 0,
          "value": 5,
          "name": "cell69 ~ gene1"
        },
        {
          "x": 68,
          "y": 1,
          "value": 7,
          "name": "cell69 ~ gene2"
        },
        {
          "x": 68,
          "y": 2,
          "value": 17,
          "name": "cell69 ~ gene3"
        },
        {
          "x": 68,
          "y": 3,
          "value": 2,
          "name": "cell69 ~ gene4"
        },
        {
          "x": 68,
          "y": 4,
          "value": 0,
          "name": "cell69 ~ gene5"
        },
        {
          "x": 68,
          "y": 5,
          "value": 0,
          "name": "cell69 ~ gene6"
        },
        {
          "x": 68,
          "y": 6,
          "value": 0,
          "name": "cell69 ~ gene7"
        },
        {
          "x": 68,
          "y": 7,
          "value": 0,
          "name": "cell69 ~ gene8"
        },
        {
          "x": 68,
          "y": 8,
          "value": 0,
          "name": "cell69 ~ gene9"
        },
        {
          "x": 68,
          "y": 9,
          "value": 9,
          "name": "cell69 ~ gene10"
        },
        {
          "x": 68,
          "y": 10,
          "value": 0,
          "name": "cell69 ~ gene11"
        },
        {
          "x": 68,
          "y": 11,
          "value": 0,
          "name": "cell69 ~ gene12"
        },
        {
          "x": 68,
          "y": 12,
          "value": 0,
          "name": "cell69 ~ gene13"
        },
        {
          "x": 68,
          "y": 13,
          "value": 0,
          "name": "cell69 ~ gene14"
        },
        {
          "x": 68,
          "y": 14,
          "value": 0,
          "name": "cell69 ~ gene15"
        },
        {
          "x": 68,
          "y": 15,
          "value": 6,
          "name": "cell69 ~ gene16"
        },
        {
          "x": 68,
          "y": 16,
          "value": 0,
          "name": "cell69 ~ gene17"
        },
        {
          "x": 68,
          "y": 17,
          "value": 10,
          "name": "cell69 ~ gene18"
        },
        {
          "x": 68,
          "y": 18,
          "value": 5,
          "name": "cell69 ~ gene19"
        },
        {
          "x": 68,
          "y": 19,
          "value": 0,
          "name": "cell69 ~ gene20"
        },
        {
          "x": 68,
          "y": 20,
          "value": 16,
          "name": "cell69 ~ gene21"
        },
        {
          "x": 68,
          "y": 21,
          "value": 2,
          "name": "cell69 ~ gene22"
        },
        {
          "x": 68,
          "y": 22,
          "value": 0,
          "name": "cell69 ~ gene23"
        },
        {
          "x": 68,
          "y": 23,
          "value": 2,
          "name": "cell69 ~ gene24"
        },
        {
          "x": 68,
          "y": 24,
          "value": 2,
          "name": "cell69 ~ gene25"
        },
        {
          "x": 68,
          "y": 25,
          "value": 0,
          "name": "cell69 ~ gene26"
        },
        {
          "x": 68,
          "y": 26,
          "value": 0,
          "name": "cell69 ~ gene27"
        },
        {
          "x": 68,
          "y": 27,
          "value": 0,
          "name": "cell69 ~ gene28"
        },
        {
          "x": 68,
          "y": 28,
          "value": 0,
          "name": "cell69 ~ gene29"
        },
        {
          "x": 68,
          "y": 29,
          "value": 0,
          "name": "cell69 ~ gene30"
        },
        {
          "x": 68,
          "y": 30,
          "value": 0,
          "name": "cell69 ~ gene31"
        },
        {
          "x": 68,
          "y": 31,
          "value": 0,
          "name": "cell69 ~ gene32"
        },
        {
          "x": 68,
          "y": 32,
          "value": 3,
          "name": "cell69 ~ gene33"
        },
        {
          "x": 68,
          "y": 33,
          "value": 0,
          "name": "cell69 ~ gene34"
        },
        {
          "x": 68,
          "y": 34,
          "value": 0,
          "name": "cell69 ~ gene35"
        },
        {
          "x": 68,
          "y": 35,
          "value": 0,
          "name": "cell69 ~ gene36"
        },
        {
          "x": 68,
          "y": 36,
          "value": 1,
          "name": "cell69 ~ gene37"
        },
        {
          "x": 68,
          "y": 37,
          "value": 19,
          "name": "cell69 ~ gene38"
        },
        {
          "x": 68,
          "y": 38,
          "value": 0,
          "name": "cell69 ~ gene39"
        },
        {
          "x": 68,
          "y": 39,
          "value": 1,
          "name": "cell69 ~ gene40"
        },
        {
          "x": 68,
          "y": 40,
          "value": 20,
          "name": "cell69 ~ gene41"
        },
        {
          "x": 68,
          "y": 41,
          "value": 13,
          "name": "cell69 ~ gene42"
        },
        {
          "x": 68,
          "y": 42,
          "value": 0,
          "name": "cell69 ~ gene43"
        },
        {
          "x": 68,
          "y": 43,
          "value": 1,
          "name": "cell69 ~ gene44"
        },
        {
          "x": 68,
          "y": 44,
          "value": 6,
          "name": "cell69 ~ gene45"
        },
        {
          "x": 68,
          "y": 45,
          "value": 38,
          "name": "cell69 ~ gene46"
        },
        {
          "x": 68,
          "y": 46,
          "value": 10,
          "name": "cell69 ~ gene47"
        },
        {
          "x": 68,
          "y": 47,
          "value": 24,
          "name": "cell69 ~ gene48"
        },
        {
          "x": 68,
          "y": 48,
          "value": 0,
          "name": "cell69 ~ gene49"
        },
        {
          "x": 68,
          "y": 49,
          "value": 0,
          "name": "cell69 ~ gene50"
        },
        {
          "x": 68,
          "y": 50,
          "value": 24,
          "name": "cell69 ~ gene51"
        },
        {
          "x": 68,
          "y": 51,
          "value": 20,
          "name": "cell69 ~ gene52"
        },
        {
          "x": 68,
          "y": 52,
          "value": 13,
          "name": "cell69 ~ gene53"
        },
        {
          "x": 68,
          "y": 53,
          "value": 14,
          "name": "cell69 ~ gene54"
        },
        {
          "x": 68,
          "y": 54,
          "value": 28,
          "name": "cell69 ~ gene55"
        },
        {
          "x": 68,
          "y": 55,
          "value": 38,
          "name": "cell69 ~ gene56"
        },
        {
          "x": 68,
          "y": 56,
          "value": 6,
          "name": "cell69 ~ gene57"
        },
        {
          "x": 68,
          "y": 57,
          "value": 17,
          "name": "cell69 ~ gene58"
        },
        {
          "x": 68,
          "y": 58,
          "value": 0,
          "name": "cell69 ~ gene59"
        },
        {
          "x": 68,
          "y": 59,
          "value": 0,
          "name": "cell69 ~ gene60"
        },
        {
          "x": 68,
          "y": 60,
          "value": 4,
          "name": "cell69 ~ gene61"
        },
        {
          "x": 68,
          "y": 61,
          "value": 0,
          "name": "cell69 ~ gene62"
        },
        {
          "x": 68,
          "y": 62,
          "value": 5,
          "name": "cell69 ~ gene63"
        },
        {
          "x": 68,
          "y": 63,
          "value": 0,
          "name": "cell69 ~ gene64"
        },
        {
          "x": 68,
          "y": 64,
          "value": 0,
          "name": "cell69 ~ gene65"
        },
        {
          "x": 68,
          "y": 65,
          "value": 0,
          "name": "cell69 ~ gene66"
        },
        {
          "x": 68,
          "y": 66,
          "value": 4,
          "name": "cell69 ~ gene67"
        },
        {
          "x": 68,
          "y": 67,
          "value": 0,
          "name": "cell69 ~ gene68"
        },
        {
          "x": 68,
          "y": 68,
          "value": 0,
          "name": "cell69 ~ gene69"
        },
        {
          "x": 68,
          "y": 69,
          "value": 10,
          "name": "cell69 ~ gene70"
        },
        {
          "x": 68,
          "y": 70,
          "value": 0,
          "name": "cell69 ~ gene71"
        },
        {
          "x": 68,
          "y": 71,
          "value": 6,
          "name": "cell69 ~ gene72"
        },
        {
          "x": 68,
          "y": 72,
          "value": 0,
          "name": "cell69 ~ gene73"
        },
        {
          "x": 68,
          "y": 73,
          "value": 0,
          "name": "cell69 ~ gene74"
        },
        {
          "x": 68,
          "y": 74,
          "value": 0,
          "name": "cell69 ~ gene75"
        },
        {
          "x": 68,
          "y": 75,
          "value": 5,
          "name": "cell69 ~ gene76"
        },
        {
          "x": 68,
          "y": 76,
          "value": 0,
          "name": "cell69 ~ gene77"
        },
        {
          "x": 68,
          "y": 77,
          "value": 7,
          "name": "cell69 ~ gene78"
        },
        {
          "x": 68,
          "y": 78,
          "value": 15,
          "name": "cell69 ~ gene79"
        },
        {
          "x": 68,
          "y": 79,
          "value": 0,
          "name": "cell69 ~ gene80"
        },
        {
          "x": 68,
          "y": 80,
          "value": 10,
          "name": "cell69 ~ gene81"
        },
        {
          "x": 68,
          "y": 81,
          "value": 0,
          "name": "cell69 ~ gene82"
        },
        {
          "x": 68,
          "y": 82,
          "value": 5,
          "name": "cell69 ~ gene83"
        },
        {
          "x": 68,
          "y": 83,
          "value": 0,
          "name": "cell69 ~ gene84"
        },
        {
          "x": 68,
          "y": 84,
          "value": 0,
          "name": "cell69 ~ gene85"
        },
        {
          "x": 68,
          "y": 85,
          "value": 0,
          "name": "cell69 ~ gene86"
        },
        {
          "x": 68,
          "y": 86,
          "value": 0,
          "name": "cell69 ~ gene87"
        },
        {
          "x": 68,
          "y": 87,
          "value": 0,
          "name": "cell69 ~ gene88"
        },
        {
          "x": 68,
          "y": 88,
          "value": 5,
          "name": "cell69 ~ gene89"
        },
        {
          "x": 68,
          "y": 89,
          "value": 0,
          "name": "cell69 ~ gene90"
        },
        {
          "x": 68,
          "y": 90,
          "value": 5,
          "name": "cell69 ~ gene91"
        },
        {
          "x": 68,
          "y": 91,
          "value": 14,
          "name": "cell69 ~ gene92"
        },
        {
          "x": 68,
          "y": 92,
          "value": 7,
          "name": "cell69 ~ gene93"
        },
        {
          "x": 68,
          "y": 93,
          "value": 7,
          "name": "cell69 ~ gene94"
        },
        {
          "x": 68,
          "y": 94,
          "value": 3,
          "name": "cell69 ~ gene95"
        },
        {
          "x": 68,
          "y": 95,
          "value": 0,
          "name": "cell69 ~ gene96"
        },
        {
          "x": 68,
          "y": 96,
          "value": 7,
          "name": "cell69 ~ gene97"
        },
        {
          "x": 68,
          "y": 97,
          "value": 6,
          "name": "cell69 ~ gene98"
        },
        {
          "x": 68,
          "y": 98,
          "value": 0,
          "name": "cell69 ~ gene99"
        },
        {
          "x": 68,
          "y": 99,
          "value": 0,
          "name": "cell69 ~ gene100"
        },
        {
          "x": 69,
          "y": 0,
          "value": 8,
          "name": "cell70 ~ gene1"
        },
        {
          "x": 69,
          "y": 1,
          "value": 0,
          "name": "cell70 ~ gene2"
        },
        {
          "x": 69,
          "y": 2,
          "value": 0,
          "name": "cell70 ~ gene3"
        },
        {
          "x": 69,
          "y": 3,
          "value": 0,
          "name": "cell70 ~ gene4"
        },
        {
          "x": 69,
          "y": 4,
          "value": 0,
          "name": "cell70 ~ gene5"
        },
        {
          "x": 69,
          "y": 5,
          "value": 0,
          "name": "cell70 ~ gene6"
        },
        {
          "x": 69,
          "y": 6,
          "value": 4,
          "name": "cell70 ~ gene7"
        },
        {
          "x": 69,
          "y": 7,
          "value": 0,
          "name": "cell70 ~ gene8"
        },
        {
          "x": 69,
          "y": 8,
          "value": 1,
          "name": "cell70 ~ gene9"
        },
        {
          "x": 69,
          "y": 9,
          "value": 3,
          "name": "cell70 ~ gene10"
        },
        {
          "x": 69,
          "y": 10,
          "value": 0,
          "name": "cell70 ~ gene11"
        },
        {
          "x": 69,
          "y": 11,
          "value": 0,
          "name": "cell70 ~ gene12"
        },
        {
          "x": 69,
          "y": 12,
          "value": 14,
          "name": "cell70 ~ gene13"
        },
        {
          "x": 69,
          "y": 13,
          "value": 1,
          "name": "cell70 ~ gene14"
        },
        {
          "x": 69,
          "y": 14,
          "value": 0,
          "name": "cell70 ~ gene15"
        },
        {
          "x": 69,
          "y": 15,
          "value": 3,
          "name": "cell70 ~ gene16"
        },
        {
          "x": 69,
          "y": 16,
          "value": 0,
          "name": "cell70 ~ gene17"
        },
        {
          "x": 69,
          "y": 17,
          "value": 0,
          "name": "cell70 ~ gene18"
        },
        {
          "x": 69,
          "y": 18,
          "value": 0,
          "name": "cell70 ~ gene19"
        },
        {
          "x": 69,
          "y": 19,
          "value": 13,
          "name": "cell70 ~ gene20"
        },
        {
          "x": 69,
          "y": 20,
          "value": 0,
          "name": "cell70 ~ gene21"
        },
        {
          "x": 69,
          "y": 21,
          "value": 0,
          "name": "cell70 ~ gene22"
        },
        {
          "x": 69,
          "y": 22,
          "value": 0,
          "name": "cell70 ~ gene23"
        },
        {
          "x": 69,
          "y": 23,
          "value": 0,
          "name": "cell70 ~ gene24"
        },
        {
          "x": 69,
          "y": 24,
          "value": 0,
          "name": "cell70 ~ gene25"
        },
        {
          "x": 69,
          "y": 25,
          "value": 0,
          "name": "cell70 ~ gene26"
        },
        {
          "x": 69,
          "y": 26,
          "value": 4,
          "name": "cell70 ~ gene27"
        },
        {
          "x": 69,
          "y": 27,
          "value": 0,
          "name": "cell70 ~ gene28"
        },
        {
          "x": 69,
          "y": 28,
          "value": 10,
          "name": "cell70 ~ gene29"
        },
        {
          "x": 69,
          "y": 29,
          "value": 0,
          "name": "cell70 ~ gene30"
        },
        {
          "x": 69,
          "y": 30,
          "value": 0,
          "name": "cell70 ~ gene31"
        },
        {
          "x": 69,
          "y": 31,
          "value": 4,
          "name": "cell70 ~ gene32"
        },
        {
          "x": 69,
          "y": 32,
          "value": 5,
          "name": "cell70 ~ gene33"
        },
        {
          "x": 69,
          "y": 33,
          "value": 5,
          "name": "cell70 ~ gene34"
        },
        {
          "x": 69,
          "y": 34,
          "value": 21,
          "name": "cell70 ~ gene35"
        },
        {
          "x": 69,
          "y": 35,
          "value": 0,
          "name": "cell70 ~ gene36"
        },
        {
          "x": 69,
          "y": 36,
          "value": 0,
          "name": "cell70 ~ gene37"
        },
        {
          "x": 69,
          "y": 37,
          "value": 0,
          "name": "cell70 ~ gene38"
        },
        {
          "x": 69,
          "y": 38,
          "value": 0,
          "name": "cell70 ~ gene39"
        },
        {
          "x": 69,
          "y": 39,
          "value": 0,
          "name": "cell70 ~ gene40"
        },
        {
          "x": 69,
          "y": 40,
          "value": 40,
          "name": "cell70 ~ gene41"
        },
        {
          "x": 69,
          "y": 41,
          "value": 18,
          "name": "cell70 ~ gene42"
        },
        {
          "x": 69,
          "y": 42,
          "value": 14,
          "name": "cell70 ~ gene43"
        },
        {
          "x": 69,
          "y": 43,
          "value": 0,
          "name": "cell70 ~ gene44"
        },
        {
          "x": 69,
          "y": 44,
          "value": 0,
          "name": "cell70 ~ gene45"
        },
        {
          "x": 69,
          "y": 45,
          "value": 18,
          "name": "cell70 ~ gene46"
        },
        {
          "x": 69,
          "y": 46,
          "value": 10,
          "name": "cell70 ~ gene47"
        },
        {
          "x": 69,
          "y": 47,
          "value": 16,
          "name": "cell70 ~ gene48"
        },
        {
          "x": 69,
          "y": 48,
          "value": 10,
          "name": "cell70 ~ gene49"
        },
        {
          "x": 69,
          "y": 49,
          "value": 22,
          "name": "cell70 ~ gene50"
        },
        {
          "x": 69,
          "y": 50,
          "value": 19,
          "name": "cell70 ~ gene51"
        },
        {
          "x": 69,
          "y": 51,
          "value": 14,
          "name": "cell70 ~ gene52"
        },
        {
          "x": 69,
          "y": 52,
          "value": 12,
          "name": "cell70 ~ gene53"
        },
        {
          "x": 69,
          "y": 53,
          "value": 12,
          "name": "cell70 ~ gene54"
        },
        {
          "x": 69,
          "y": 54,
          "value": 15,
          "name": "cell70 ~ gene55"
        },
        {
          "x": 69,
          "y": 55,
          "value": 32,
          "name": "cell70 ~ gene56"
        },
        {
          "x": 69,
          "y": 56,
          "value": 19,
          "name": "cell70 ~ gene57"
        },
        {
          "x": 69,
          "y": 57,
          "value": 1,
          "name": "cell70 ~ gene58"
        },
        {
          "x": 69,
          "y": 58,
          "value": 19,
          "name": "cell70 ~ gene59"
        },
        {
          "x": 69,
          "y": 59,
          "value": 0,
          "name": "cell70 ~ gene60"
        },
        {
          "x": 69,
          "y": 60,
          "value": 7,
          "name": "cell70 ~ gene61"
        },
        {
          "x": 69,
          "y": 61,
          "value": 0,
          "name": "cell70 ~ gene62"
        },
        {
          "x": 69,
          "y": 62,
          "value": 0,
          "name": "cell70 ~ gene63"
        },
        {
          "x": 69,
          "y": 63,
          "value": 0,
          "name": "cell70 ~ gene64"
        },
        {
          "x": 69,
          "y": 64,
          "value": 11,
          "name": "cell70 ~ gene65"
        },
        {
          "x": 69,
          "y": 65,
          "value": 0,
          "name": "cell70 ~ gene66"
        },
        {
          "x": 69,
          "y": 66,
          "value": 5,
          "name": "cell70 ~ gene67"
        },
        {
          "x": 69,
          "y": 67,
          "value": 5,
          "name": "cell70 ~ gene68"
        },
        {
          "x": 69,
          "y": 68,
          "value": 0,
          "name": "cell70 ~ gene69"
        },
        {
          "x": 69,
          "y": 69,
          "value": 7,
          "name": "cell70 ~ gene70"
        },
        {
          "x": 69,
          "y": 70,
          "value": 9,
          "name": "cell70 ~ gene71"
        },
        {
          "x": 69,
          "y": 71,
          "value": 0,
          "name": "cell70 ~ gene72"
        },
        {
          "x": 69,
          "y": 72,
          "value": 0,
          "name": "cell70 ~ gene73"
        },
        {
          "x": 69,
          "y": 73,
          "value": 17,
          "name": "cell70 ~ gene74"
        },
        {
          "x": 69,
          "y": 74,
          "value": 0,
          "name": "cell70 ~ gene75"
        },
        {
          "x": 69,
          "y": 75,
          "value": 15,
          "name": "cell70 ~ gene76"
        },
        {
          "x": 69,
          "y": 76,
          "value": 12,
          "name": "cell70 ~ gene77"
        },
        {
          "x": 69,
          "y": 77,
          "value": 0,
          "name": "cell70 ~ gene78"
        },
        {
          "x": 69,
          "y": 78,
          "value": 5,
          "name": "cell70 ~ gene79"
        },
        {
          "x": 69,
          "y": 79,
          "value": 10,
          "name": "cell70 ~ gene80"
        },
        {
          "x": 69,
          "y": 80,
          "value": 0,
          "name": "cell70 ~ gene81"
        },
        {
          "x": 69,
          "y": 81,
          "value": 0,
          "name": "cell70 ~ gene82"
        },
        {
          "x": 69,
          "y": 82,
          "value": 0,
          "name": "cell70 ~ gene83"
        },
        {
          "x": 69,
          "y": 83,
          "value": 8,
          "name": "cell70 ~ gene84"
        },
        {
          "x": 69,
          "y": 84,
          "value": 7,
          "name": "cell70 ~ gene85"
        },
        {
          "x": 69,
          "y": 85,
          "value": 0,
          "name": "cell70 ~ gene86"
        },
        {
          "x": 69,
          "y": 86,
          "value": 19,
          "name": "cell70 ~ gene87"
        },
        {
          "x": 69,
          "y": 87,
          "value": 5,
          "name": "cell70 ~ gene88"
        },
        {
          "x": 69,
          "y": 88,
          "value": 0,
          "name": "cell70 ~ gene89"
        },
        {
          "x": 69,
          "y": 89,
          "value": 19,
          "name": "cell70 ~ gene90"
        },
        {
          "x": 69,
          "y": 90,
          "value": 22,
          "name": "cell70 ~ gene91"
        },
        {
          "x": 69,
          "y": 91,
          "value": 0,
          "name": "cell70 ~ gene92"
        },
        {
          "x": 69,
          "y": 92,
          "value": 0,
          "name": "cell70 ~ gene93"
        },
        {
          "x": 69,
          "y": 93,
          "value": 15,
          "name": "cell70 ~ gene94"
        },
        {
          "x": 69,
          "y": 94,
          "value": 0,
          "name": "cell70 ~ gene95"
        },
        {
          "x": 69,
          "y": 95,
          "value": 3,
          "name": "cell70 ~ gene96"
        },
        {
          "x": 69,
          "y": 96,
          "value": 6,
          "name": "cell70 ~ gene97"
        },
        {
          "x": 69,
          "y": 97,
          "value": 0,
          "name": "cell70 ~ gene98"
        },
        {
          "x": 69,
          "y": 98,
          "value": 9,
          "name": "cell70 ~ gene99"
        },
        {
          "x": 69,
          "y": 99,
          "value": 1,
          "name": "cell70 ~ gene100"
        },
        {
          "x": 70,
          "y": 0,
          "value": 6,
          "name": "cell71 ~ gene1"
        },
        {
          "x": 70,
          "y": 1,
          "value": 0,
          "name": "cell71 ~ gene2"
        },
        {
          "x": 70,
          "y": 2,
          "value": 9,
          "name": "cell71 ~ gene3"
        },
        {
          "x": 70,
          "y": 3,
          "value": 0,
          "name": "cell71 ~ gene4"
        },
        {
          "x": 70,
          "y": 4,
          "value": 6,
          "name": "cell71 ~ gene5"
        },
        {
          "x": 70,
          "y": 5,
          "value": 0,
          "name": "cell71 ~ gene6"
        },
        {
          "x": 70,
          "y": 6,
          "value": 18,
          "name": "cell71 ~ gene7"
        },
        {
          "x": 70,
          "y": 7,
          "value": 7,
          "name": "cell71 ~ gene8"
        },
        {
          "x": 70,
          "y": 8,
          "value": 5,
          "name": "cell71 ~ gene9"
        },
        {
          "x": 70,
          "y": 9,
          "value": 3,
          "name": "cell71 ~ gene10"
        },
        {
          "x": 70,
          "y": 10,
          "value": 0,
          "name": "cell71 ~ gene11"
        },
        {
          "x": 70,
          "y": 11,
          "value": 0,
          "name": "cell71 ~ gene12"
        },
        {
          "x": 70,
          "y": 12,
          "value": 0,
          "name": "cell71 ~ gene13"
        },
        {
          "x": 70,
          "y": 13,
          "value": 4,
          "name": "cell71 ~ gene14"
        },
        {
          "x": 70,
          "y": 14,
          "value": 6,
          "name": "cell71 ~ gene15"
        },
        {
          "x": 70,
          "y": 15,
          "value": 0,
          "name": "cell71 ~ gene16"
        },
        {
          "x": 70,
          "y": 16,
          "value": 0,
          "name": "cell71 ~ gene17"
        },
        {
          "x": 70,
          "y": 17,
          "value": 8,
          "name": "cell71 ~ gene18"
        },
        {
          "x": 70,
          "y": 18,
          "value": 16,
          "name": "cell71 ~ gene19"
        },
        {
          "x": 70,
          "y": 19,
          "value": 14,
          "name": "cell71 ~ gene20"
        },
        {
          "x": 70,
          "y": 20,
          "value": 11,
          "name": "cell71 ~ gene21"
        },
        {
          "x": 70,
          "y": 21,
          "value": 13,
          "name": "cell71 ~ gene22"
        },
        {
          "x": 70,
          "y": 22,
          "value": 0,
          "name": "cell71 ~ gene23"
        },
        {
          "x": 70,
          "y": 23,
          "value": 0,
          "name": "cell71 ~ gene24"
        },
        {
          "x": 70,
          "y": 24,
          "value": 0,
          "name": "cell71 ~ gene25"
        },
        {
          "x": 70,
          "y": 25,
          "value": 14,
          "name": "cell71 ~ gene26"
        },
        {
          "x": 70,
          "y": 26,
          "value": 0,
          "name": "cell71 ~ gene27"
        },
        {
          "x": 70,
          "y": 27,
          "value": 0,
          "name": "cell71 ~ gene28"
        },
        {
          "x": 70,
          "y": 28,
          "value": 2,
          "name": "cell71 ~ gene29"
        },
        {
          "x": 70,
          "y": 29,
          "value": 15,
          "name": "cell71 ~ gene30"
        },
        {
          "x": 70,
          "y": 30,
          "value": 0,
          "name": "cell71 ~ gene31"
        },
        {
          "x": 70,
          "y": 31,
          "value": 0,
          "name": "cell71 ~ gene32"
        },
        {
          "x": 70,
          "y": 32,
          "value": 0,
          "name": "cell71 ~ gene33"
        },
        {
          "x": 70,
          "y": 33,
          "value": 0,
          "name": "cell71 ~ gene34"
        },
        {
          "x": 70,
          "y": 34,
          "value": 0,
          "name": "cell71 ~ gene35"
        },
        {
          "x": 70,
          "y": 35,
          "value": 3,
          "name": "cell71 ~ gene36"
        },
        {
          "x": 70,
          "y": 36,
          "value": 2,
          "name": "cell71 ~ gene37"
        },
        {
          "x": 70,
          "y": 37,
          "value": 2,
          "name": "cell71 ~ gene38"
        },
        {
          "x": 70,
          "y": 38,
          "value": 6,
          "name": "cell71 ~ gene39"
        },
        {
          "x": 70,
          "y": 39,
          "value": 2,
          "name": "cell71 ~ gene40"
        },
        {
          "x": 70,
          "y": 40,
          "value": 28,
          "name": "cell71 ~ gene41"
        },
        {
          "x": 70,
          "y": 41,
          "value": 15,
          "name": "cell71 ~ gene42"
        },
        {
          "x": 70,
          "y": 42,
          "value": 0,
          "name": "cell71 ~ gene43"
        },
        {
          "x": 70,
          "y": 43,
          "value": 0,
          "name": "cell71 ~ gene44"
        },
        {
          "x": 70,
          "y": 44,
          "value": 0,
          "name": "cell71 ~ gene45"
        },
        {
          "x": 70,
          "y": 45,
          "value": 23,
          "name": "cell71 ~ gene46"
        },
        {
          "x": 70,
          "y": 46,
          "value": 7,
          "name": "cell71 ~ gene47"
        },
        {
          "x": 70,
          "y": 47,
          "value": 9,
          "name": "cell71 ~ gene48"
        },
        {
          "x": 70,
          "y": 48,
          "value": 0,
          "name": "cell71 ~ gene49"
        },
        {
          "x": 70,
          "y": 49,
          "value": 5,
          "name": "cell71 ~ gene50"
        },
        {
          "x": 70,
          "y": 50,
          "value": 20,
          "name": "cell71 ~ gene51"
        },
        {
          "x": 70,
          "y": 51,
          "value": 13,
          "name": "cell71 ~ gene52"
        },
        {
          "x": 70,
          "y": 52,
          "value": 9,
          "name": "cell71 ~ gene53"
        },
        {
          "x": 70,
          "y": 53,
          "value": 1,
          "name": "cell71 ~ gene54"
        },
        {
          "x": 70,
          "y": 54,
          "value": 15,
          "name": "cell71 ~ gene55"
        },
        {
          "x": 70,
          "y": 55,
          "value": 31,
          "name": "cell71 ~ gene56"
        },
        {
          "x": 70,
          "y": 56,
          "value": 18,
          "name": "cell71 ~ gene57"
        },
        {
          "x": 70,
          "y": 57,
          "value": 12,
          "name": "cell71 ~ gene58"
        },
        {
          "x": 70,
          "y": 58,
          "value": 12,
          "name": "cell71 ~ gene59"
        },
        {
          "x": 70,
          "y": 59,
          "value": 8,
          "name": "cell71 ~ gene60"
        },
        {
          "x": 70,
          "y": 60,
          "value": 0,
          "name": "cell71 ~ gene61"
        },
        {
          "x": 70,
          "y": 61,
          "value": 0,
          "name": "cell71 ~ gene62"
        },
        {
          "x": 70,
          "y": 62,
          "value": 0,
          "name": "cell71 ~ gene63"
        },
        {
          "x": 70,
          "y": 63,
          "value": 0,
          "name": "cell71 ~ gene64"
        },
        {
          "x": 70,
          "y": 64,
          "value": 1,
          "name": "cell71 ~ gene65"
        },
        {
          "x": 70,
          "y": 65,
          "value": 12,
          "name": "cell71 ~ gene66"
        },
        {
          "x": 70,
          "y": 66,
          "value": 0,
          "name": "cell71 ~ gene67"
        },
        {
          "x": 70,
          "y": 67,
          "value": 0,
          "name": "cell71 ~ gene68"
        },
        {
          "x": 70,
          "y": 68,
          "value": 1,
          "name": "cell71 ~ gene69"
        },
        {
          "x": 70,
          "y": 69,
          "value": 0,
          "name": "cell71 ~ gene70"
        },
        {
          "x": 70,
          "y": 70,
          "value": 0,
          "name": "cell71 ~ gene71"
        },
        {
          "x": 70,
          "y": 71,
          "value": 10,
          "name": "cell71 ~ gene72"
        },
        {
          "x": 70,
          "y": 72,
          "value": 3,
          "name": "cell71 ~ gene73"
        },
        {
          "x": 70,
          "y": 73,
          "value": 5,
          "name": "cell71 ~ gene74"
        },
        {
          "x": 70,
          "y": 74,
          "value": 11,
          "name": "cell71 ~ gene75"
        },
        {
          "x": 70,
          "y": 75,
          "value": 0,
          "name": "cell71 ~ gene76"
        },
        {
          "x": 70,
          "y": 76,
          "value": 9,
          "name": "cell71 ~ gene77"
        },
        {
          "x": 70,
          "y": 77,
          "value": 5,
          "name": "cell71 ~ gene78"
        },
        {
          "x": 70,
          "y": 78,
          "value": 8,
          "name": "cell71 ~ gene79"
        },
        {
          "x": 70,
          "y": 79,
          "value": 3,
          "name": "cell71 ~ gene80"
        },
        {
          "x": 70,
          "y": 80,
          "value": 6,
          "name": "cell71 ~ gene81"
        },
        {
          "x": 70,
          "y": 81,
          "value": 12,
          "name": "cell71 ~ gene82"
        },
        {
          "x": 70,
          "y": 82,
          "value": 2,
          "name": "cell71 ~ gene83"
        },
        {
          "x": 70,
          "y": 83,
          "value": 0,
          "name": "cell71 ~ gene84"
        },
        {
          "x": 70,
          "y": 84,
          "value": 19,
          "name": "cell71 ~ gene85"
        },
        {
          "x": 70,
          "y": 85,
          "value": 0,
          "name": "cell71 ~ gene86"
        },
        {
          "x": 70,
          "y": 86,
          "value": 0,
          "name": "cell71 ~ gene87"
        },
        {
          "x": 70,
          "y": 87,
          "value": 0,
          "name": "cell71 ~ gene88"
        },
        {
          "x": 70,
          "y": 88,
          "value": 4,
          "name": "cell71 ~ gene89"
        },
        {
          "x": 70,
          "y": 89,
          "value": 0,
          "name": "cell71 ~ gene90"
        },
        {
          "x": 70,
          "y": 90,
          "value": 28,
          "name": "cell71 ~ gene91"
        },
        {
          "x": 70,
          "y": 91,
          "value": 0,
          "name": "cell71 ~ gene92"
        },
        {
          "x": 70,
          "y": 92,
          "value": 0,
          "name": "cell71 ~ gene93"
        },
        {
          "x": 70,
          "y": 93,
          "value": 2,
          "name": "cell71 ~ gene94"
        },
        {
          "x": 70,
          "y": 94,
          "value": 10,
          "name": "cell71 ~ gene95"
        },
        {
          "x": 70,
          "y": 95,
          "value": 20,
          "name": "cell71 ~ gene96"
        },
        {
          "x": 70,
          "y": 96,
          "value": 0,
          "name": "cell71 ~ gene97"
        },
        {
          "x": 70,
          "y": 97,
          "value": 8,
          "name": "cell71 ~ gene98"
        },
        {
          "x": 70,
          "y": 98,
          "value": 20,
          "name": "cell71 ~ gene99"
        },
        {
          "x": 70,
          "y": 99,
          "value": 3,
          "name": "cell71 ~ gene100"
        },
        {
          "x": 71,
          "y": 0,
          "value": 4,
          "name": "cell72 ~ gene1"
        },
        {
          "x": 71,
          "y": 1,
          "value": 0,
          "name": "cell72 ~ gene2"
        },
        {
          "x": 71,
          "y": 2,
          "value": 12,
          "name": "cell72 ~ gene3"
        },
        {
          "x": 71,
          "y": 3,
          "value": 1,
          "name": "cell72 ~ gene4"
        },
        {
          "x": 71,
          "y": 4,
          "value": 13,
          "name": "cell72 ~ gene5"
        },
        {
          "x": 71,
          "y": 5,
          "value": 2,
          "name": "cell72 ~ gene6"
        },
        {
          "x": 71,
          "y": 6,
          "value": 14,
          "name": "cell72 ~ gene7"
        },
        {
          "x": 71,
          "y": 7,
          "value": 0,
          "name": "cell72 ~ gene8"
        },
        {
          "x": 71,
          "y": 8,
          "value": 21,
          "name": "cell72 ~ gene9"
        },
        {
          "x": 71,
          "y": 9,
          "value": 3,
          "name": "cell72 ~ gene10"
        },
        {
          "x": 71,
          "y": 10,
          "value": 0,
          "name": "cell72 ~ gene11"
        },
        {
          "x": 71,
          "y": 11,
          "value": 0,
          "name": "cell72 ~ gene12"
        },
        {
          "x": 71,
          "y": 12,
          "value": 15,
          "name": "cell72 ~ gene13"
        },
        {
          "x": 71,
          "y": 13,
          "value": 5,
          "name": "cell72 ~ gene14"
        },
        {
          "x": 71,
          "y": 14,
          "value": 0,
          "name": "cell72 ~ gene15"
        },
        {
          "x": 71,
          "y": 15,
          "value": 0,
          "name": "cell72 ~ gene16"
        },
        {
          "x": 71,
          "y": 16,
          "value": 0,
          "name": "cell72 ~ gene17"
        },
        {
          "x": 71,
          "y": 17,
          "value": 0,
          "name": "cell72 ~ gene18"
        },
        {
          "x": 71,
          "y": 18,
          "value": 5,
          "name": "cell72 ~ gene19"
        },
        {
          "x": 71,
          "y": 19,
          "value": 0,
          "name": "cell72 ~ gene20"
        },
        {
          "x": 71,
          "y": 20,
          "value": 0,
          "name": "cell72 ~ gene21"
        },
        {
          "x": 71,
          "y": 21,
          "value": 0,
          "name": "cell72 ~ gene22"
        },
        {
          "x": 71,
          "y": 22,
          "value": 0,
          "name": "cell72 ~ gene23"
        },
        {
          "x": 71,
          "y": 23,
          "value": 0,
          "name": "cell72 ~ gene24"
        },
        {
          "x": 71,
          "y": 24,
          "value": 4,
          "name": "cell72 ~ gene25"
        },
        {
          "x": 71,
          "y": 25,
          "value": 5,
          "name": "cell72 ~ gene26"
        },
        {
          "x": 71,
          "y": 26,
          "value": 0,
          "name": "cell72 ~ gene27"
        },
        {
          "x": 71,
          "y": 27,
          "value": 8,
          "name": "cell72 ~ gene28"
        },
        {
          "x": 71,
          "y": 28,
          "value": 0,
          "name": "cell72 ~ gene29"
        },
        {
          "x": 71,
          "y": 29,
          "value": 0,
          "name": "cell72 ~ gene30"
        },
        {
          "x": 71,
          "y": 30,
          "value": 9,
          "name": "cell72 ~ gene31"
        },
        {
          "x": 71,
          "y": 31,
          "value": 0,
          "name": "cell72 ~ gene32"
        },
        {
          "x": 71,
          "y": 32,
          "value": 0,
          "name": "cell72 ~ gene33"
        },
        {
          "x": 71,
          "y": 33,
          "value": 11,
          "name": "cell72 ~ gene34"
        },
        {
          "x": 71,
          "y": 34,
          "value": 0,
          "name": "cell72 ~ gene35"
        },
        {
          "x": 71,
          "y": 35,
          "value": 11,
          "name": "cell72 ~ gene36"
        },
        {
          "x": 71,
          "y": 36,
          "value": 4,
          "name": "cell72 ~ gene37"
        },
        {
          "x": 71,
          "y": 37,
          "value": 19,
          "name": "cell72 ~ gene38"
        },
        {
          "x": 71,
          "y": 38,
          "value": 0,
          "name": "cell72 ~ gene39"
        },
        {
          "x": 71,
          "y": 39,
          "value": 0,
          "name": "cell72 ~ gene40"
        },
        {
          "x": 71,
          "y": 40,
          "value": 0,
          "name": "cell72 ~ gene41"
        },
        {
          "x": 71,
          "y": 41,
          "value": 48,
          "name": "cell72 ~ gene42"
        },
        {
          "x": 71,
          "y": 42,
          "value": 0,
          "name": "cell72 ~ gene43"
        },
        {
          "x": 71,
          "y": 43,
          "value": 0,
          "name": "cell72 ~ gene44"
        },
        {
          "x": 71,
          "y": 44,
          "value": 8,
          "name": "cell72 ~ gene45"
        },
        {
          "x": 71,
          "y": 45,
          "value": 4,
          "name": "cell72 ~ gene46"
        },
        {
          "x": 71,
          "y": 46,
          "value": 12,
          "name": "cell72 ~ gene47"
        },
        {
          "x": 71,
          "y": 47,
          "value": 0,
          "name": "cell72 ~ gene48"
        },
        {
          "x": 71,
          "y": 48,
          "value": 1,
          "name": "cell72 ~ gene49"
        },
        {
          "x": 71,
          "y": 49,
          "value": 23,
          "name": "cell72 ~ gene50"
        },
        {
          "x": 71,
          "y": 50,
          "value": 27,
          "name": "cell72 ~ gene51"
        },
        {
          "x": 71,
          "y": 51,
          "value": 14,
          "name": "cell72 ~ gene52"
        },
        {
          "x": 71,
          "y": 52,
          "value": 18,
          "name": "cell72 ~ gene53"
        },
        {
          "x": 71,
          "y": 53,
          "value": 17,
          "name": "cell72 ~ gene54"
        },
        {
          "x": 71,
          "y": 54,
          "value": 27,
          "name": "cell72 ~ gene55"
        },
        {
          "x": 71,
          "y": 55,
          "value": 25,
          "name": "cell72 ~ gene56"
        },
        {
          "x": 71,
          "y": 56,
          "value": 7,
          "name": "cell72 ~ gene57"
        },
        {
          "x": 71,
          "y": 57,
          "value": 3,
          "name": "cell72 ~ gene58"
        },
        {
          "x": 71,
          "y": 58,
          "value": 4,
          "name": "cell72 ~ gene59"
        },
        {
          "x": 71,
          "y": 59,
          "value": 0,
          "name": "cell72 ~ gene60"
        },
        {
          "x": 71,
          "y": 60,
          "value": 4,
          "name": "cell72 ~ gene61"
        },
        {
          "x": 71,
          "y": 61,
          "value": 3,
          "name": "cell72 ~ gene62"
        },
        {
          "x": 71,
          "y": 62,
          "value": 0,
          "name": "cell72 ~ gene63"
        },
        {
          "x": 71,
          "y": 63,
          "value": 0,
          "name": "cell72 ~ gene64"
        },
        {
          "x": 71,
          "y": 64,
          "value": 2,
          "name": "cell72 ~ gene65"
        },
        {
          "x": 71,
          "y": 65,
          "value": 0,
          "name": "cell72 ~ gene66"
        },
        {
          "x": 71,
          "y": 66,
          "value": 0,
          "name": "cell72 ~ gene67"
        },
        {
          "x": 71,
          "y": 67,
          "value": 2,
          "name": "cell72 ~ gene68"
        },
        {
          "x": 71,
          "y": 68,
          "value": 0,
          "name": "cell72 ~ gene69"
        },
        {
          "x": 71,
          "y": 69,
          "value": 6,
          "name": "cell72 ~ gene70"
        },
        {
          "x": 71,
          "y": 70,
          "value": 0,
          "name": "cell72 ~ gene71"
        },
        {
          "x": 71,
          "y": 71,
          "value": 9,
          "name": "cell72 ~ gene72"
        },
        {
          "x": 71,
          "y": 72,
          "value": 0,
          "name": "cell72 ~ gene73"
        },
        {
          "x": 71,
          "y": 73,
          "value": 0,
          "name": "cell72 ~ gene74"
        },
        {
          "x": 71,
          "y": 74,
          "value": 0,
          "name": "cell72 ~ gene75"
        },
        {
          "x": 71,
          "y": 75,
          "value": 9,
          "name": "cell72 ~ gene76"
        },
        {
          "x": 71,
          "y": 76,
          "value": 11,
          "name": "cell72 ~ gene77"
        },
        {
          "x": 71,
          "y": 77,
          "value": 0,
          "name": "cell72 ~ gene78"
        },
        {
          "x": 71,
          "y": 78,
          "value": 0,
          "name": "cell72 ~ gene79"
        },
        {
          "x": 71,
          "y": 79,
          "value": 6,
          "name": "cell72 ~ gene80"
        },
        {
          "x": 71,
          "y": 80,
          "value": 0,
          "name": "cell72 ~ gene81"
        },
        {
          "x": 71,
          "y": 81,
          "value": 0,
          "name": "cell72 ~ gene82"
        },
        {
          "x": 71,
          "y": 82,
          "value": 13,
          "name": "cell72 ~ gene83"
        },
        {
          "x": 71,
          "y": 83,
          "value": 0,
          "name": "cell72 ~ gene84"
        },
        {
          "x": 71,
          "y": 84,
          "value": 0,
          "name": "cell72 ~ gene85"
        },
        {
          "x": 71,
          "y": 85,
          "value": 0,
          "name": "cell72 ~ gene86"
        },
        {
          "x": 71,
          "y": 86,
          "value": 8,
          "name": "cell72 ~ gene87"
        },
        {
          "x": 71,
          "y": 87,
          "value": 1,
          "name": "cell72 ~ gene88"
        },
        {
          "x": 71,
          "y": 88,
          "value": 0,
          "name": "cell72 ~ gene89"
        },
        {
          "x": 71,
          "y": 89,
          "value": 0,
          "name": "cell72 ~ gene90"
        },
        {
          "x": 71,
          "y": 90,
          "value": 0,
          "name": "cell72 ~ gene91"
        },
        {
          "x": 71,
          "y": 91,
          "value": 0,
          "name": "cell72 ~ gene92"
        },
        {
          "x": 71,
          "y": 92,
          "value": 0,
          "name": "cell72 ~ gene93"
        },
        {
          "x": 71,
          "y": 93,
          "value": 4,
          "name": "cell72 ~ gene94"
        },
        {
          "x": 71,
          "y": 94,
          "value": 0,
          "name": "cell72 ~ gene95"
        },
        {
          "x": 71,
          "y": 95,
          "value": 7,
          "name": "cell72 ~ gene96"
        },
        {
          "x": 71,
          "y": 96,
          "value": 5,
          "name": "cell72 ~ gene97"
        },
        {
          "x": 71,
          "y": 97,
          "value": 13,
          "name": "cell72 ~ gene98"
        },
        {
          "x": 71,
          "y": 98,
          "value": 2,
          "name": "cell72 ~ gene99"
        },
        {
          "x": 71,
          "y": 99,
          "value": 0,
          "name": "cell72 ~ gene100"
        },
        {
          "x": 72,
          "y": 0,
          "value": 4,
          "name": "cell73 ~ gene1"
        },
        {
          "x": 72,
          "y": 1,
          "value": 9,
          "name": "cell73 ~ gene2"
        },
        {
          "x": 72,
          "y": 2,
          "value": 6,
          "name": "cell73 ~ gene3"
        },
        {
          "x": 72,
          "y": 3,
          "value": 10,
          "name": "cell73 ~ gene4"
        },
        {
          "x": 72,
          "y": 4,
          "value": 5,
          "name": "cell73 ~ gene5"
        },
        {
          "x": 72,
          "y": 5,
          "value": 0,
          "name": "cell73 ~ gene6"
        },
        {
          "x": 72,
          "y": 6,
          "value": 0,
          "name": "cell73 ~ gene7"
        },
        {
          "x": 72,
          "y": 7,
          "value": 0,
          "name": "cell73 ~ gene8"
        },
        {
          "x": 72,
          "y": 8,
          "value": 23,
          "name": "cell73 ~ gene9"
        },
        {
          "x": 72,
          "y": 9,
          "value": 5,
          "name": "cell73 ~ gene10"
        },
        {
          "x": 72,
          "y": 10,
          "value": 3,
          "name": "cell73 ~ gene11"
        },
        {
          "x": 72,
          "y": 11,
          "value": 9,
          "name": "cell73 ~ gene12"
        },
        {
          "x": 72,
          "y": 12,
          "value": 7,
          "name": "cell73 ~ gene13"
        },
        {
          "x": 72,
          "y": 13,
          "value": 8,
          "name": "cell73 ~ gene14"
        },
        {
          "x": 72,
          "y": 14,
          "value": 3,
          "name": "cell73 ~ gene15"
        },
        {
          "x": 72,
          "y": 15,
          "value": 0,
          "name": "cell73 ~ gene16"
        },
        {
          "x": 72,
          "y": 16,
          "value": 0,
          "name": "cell73 ~ gene17"
        },
        {
          "x": 72,
          "y": 17,
          "value": 7,
          "name": "cell73 ~ gene18"
        },
        {
          "x": 72,
          "y": 18,
          "value": 0,
          "name": "cell73 ~ gene19"
        },
        {
          "x": 72,
          "y": 19,
          "value": 0,
          "name": "cell73 ~ gene20"
        },
        {
          "x": 72,
          "y": 20,
          "value": 4,
          "name": "cell73 ~ gene21"
        },
        {
          "x": 72,
          "y": 21,
          "value": 9,
          "name": "cell73 ~ gene22"
        },
        {
          "x": 72,
          "y": 22,
          "value": 0,
          "name": "cell73 ~ gene23"
        },
        {
          "x": 72,
          "y": 23,
          "value": 0,
          "name": "cell73 ~ gene24"
        },
        {
          "x": 72,
          "y": 24,
          "value": 20,
          "name": "cell73 ~ gene25"
        },
        {
          "x": 72,
          "y": 25,
          "value": 0,
          "name": "cell73 ~ gene26"
        },
        {
          "x": 72,
          "y": 26,
          "value": 0,
          "name": "cell73 ~ gene27"
        },
        {
          "x": 72,
          "y": 27,
          "value": 0,
          "name": "cell73 ~ gene28"
        },
        {
          "x": 72,
          "y": 28,
          "value": 0,
          "name": "cell73 ~ gene29"
        },
        {
          "x": 72,
          "y": 29,
          "value": 18,
          "name": "cell73 ~ gene30"
        },
        {
          "x": 72,
          "y": 30,
          "value": 1,
          "name": "cell73 ~ gene31"
        },
        {
          "x": 72,
          "y": 31,
          "value": 16,
          "name": "cell73 ~ gene32"
        },
        {
          "x": 72,
          "y": 32,
          "value": 0,
          "name": "cell73 ~ gene33"
        },
        {
          "x": 72,
          "y": 33,
          "value": 0,
          "name": "cell73 ~ gene34"
        },
        {
          "x": 72,
          "y": 34,
          "value": 6,
          "name": "cell73 ~ gene35"
        },
        {
          "x": 72,
          "y": 35,
          "value": 10,
          "name": "cell73 ~ gene36"
        },
        {
          "x": 72,
          "y": 36,
          "value": 0,
          "name": "cell73 ~ gene37"
        },
        {
          "x": 72,
          "y": 37,
          "value": 0,
          "name": "cell73 ~ gene38"
        },
        {
          "x": 72,
          "y": 38,
          "value": 0,
          "name": "cell73 ~ gene39"
        },
        {
          "x": 72,
          "y": 39,
          "value": 6,
          "name": "cell73 ~ gene40"
        },
        {
          "x": 72,
          "y": 40,
          "value": 8,
          "name": "cell73 ~ gene41"
        },
        {
          "x": 72,
          "y": 41,
          "value": 11,
          "name": "cell73 ~ gene42"
        },
        {
          "x": 72,
          "y": 42,
          "value": 12,
          "name": "cell73 ~ gene43"
        },
        {
          "x": 72,
          "y": 43,
          "value": 0,
          "name": "cell73 ~ gene44"
        },
        {
          "x": 72,
          "y": 44,
          "value": 0,
          "name": "cell73 ~ gene45"
        },
        {
          "x": 72,
          "y": 45,
          "value": 5,
          "name": "cell73 ~ gene46"
        },
        {
          "x": 72,
          "y": 46,
          "value": 20,
          "name": "cell73 ~ gene47"
        },
        {
          "x": 72,
          "y": 47,
          "value": 0,
          "name": "cell73 ~ gene48"
        },
        {
          "x": 72,
          "y": 48,
          "value": 5,
          "name": "cell73 ~ gene49"
        },
        {
          "x": 72,
          "y": 49,
          "value": 15,
          "name": "cell73 ~ gene50"
        },
        {
          "x": 72,
          "y": 50,
          "value": 25,
          "name": "cell73 ~ gene51"
        },
        {
          "x": 72,
          "y": 51,
          "value": 18,
          "name": "cell73 ~ gene52"
        },
        {
          "x": 72,
          "y": 52,
          "value": 13,
          "name": "cell73 ~ gene53"
        },
        {
          "x": 72,
          "y": 53,
          "value": 1,
          "name": "cell73 ~ gene54"
        },
        {
          "x": 72,
          "y": 54,
          "value": 30,
          "name": "cell73 ~ gene55"
        },
        {
          "x": 72,
          "y": 55,
          "value": 34,
          "name": "cell73 ~ gene56"
        },
        {
          "x": 72,
          "y": 56,
          "value": 3,
          "name": "cell73 ~ gene57"
        },
        {
          "x": 72,
          "y": 57,
          "value": 18,
          "name": "cell73 ~ gene58"
        },
        {
          "x": 72,
          "y": 58,
          "value": 9,
          "name": "cell73 ~ gene59"
        },
        {
          "x": 72,
          "y": 59,
          "value": 0,
          "name": "cell73 ~ gene60"
        },
        {
          "x": 72,
          "y": 60,
          "value": 0,
          "name": "cell73 ~ gene61"
        },
        {
          "x": 72,
          "y": 61,
          "value": 0,
          "name": "cell73 ~ gene62"
        },
        {
          "x": 72,
          "y": 62,
          "value": 0,
          "name": "cell73 ~ gene63"
        },
        {
          "x": 72,
          "y": 63,
          "value": 0,
          "name": "cell73 ~ gene64"
        },
        {
          "x": 72,
          "y": 64,
          "value": 1,
          "name": "cell73 ~ gene65"
        },
        {
          "x": 72,
          "y": 65,
          "value": 0,
          "name": "cell73 ~ gene66"
        },
        {
          "x": 72,
          "y": 66,
          "value": 6,
          "name": "cell73 ~ gene67"
        },
        {
          "x": 72,
          "y": 67,
          "value": 11,
          "name": "cell73 ~ gene68"
        },
        {
          "x": 72,
          "y": 68,
          "value": 10,
          "name": "cell73 ~ gene69"
        },
        {
          "x": 72,
          "y": 69,
          "value": 0,
          "name": "cell73 ~ gene70"
        },
        {
          "x": 72,
          "y": 70,
          "value": 22,
          "name": "cell73 ~ gene71"
        },
        {
          "x": 72,
          "y": 71,
          "value": 7,
          "name": "cell73 ~ gene72"
        },
        {
          "x": 72,
          "y": 72,
          "value": 0,
          "name": "cell73 ~ gene73"
        },
        {
          "x": 72,
          "y": 73,
          "value": 0,
          "name": "cell73 ~ gene74"
        },
        {
          "x": 72,
          "y": 74,
          "value": 9,
          "name": "cell73 ~ gene75"
        },
        {
          "x": 72,
          "y": 75,
          "value": 2,
          "name": "cell73 ~ gene76"
        },
        {
          "x": 72,
          "y": 76,
          "value": 0,
          "name": "cell73 ~ gene77"
        },
        {
          "x": 72,
          "y": 77,
          "value": 0,
          "name": "cell73 ~ gene78"
        },
        {
          "x": 72,
          "y": 78,
          "value": 4,
          "name": "cell73 ~ gene79"
        },
        {
          "x": 72,
          "y": 79,
          "value": 10,
          "name": "cell73 ~ gene80"
        },
        {
          "x": 72,
          "y": 80,
          "value": 0,
          "name": "cell73 ~ gene81"
        },
        {
          "x": 72,
          "y": 81,
          "value": 1,
          "name": "cell73 ~ gene82"
        },
        {
          "x": 72,
          "y": 82,
          "value": 5,
          "name": "cell73 ~ gene83"
        },
        {
          "x": 72,
          "y": 83,
          "value": 0,
          "name": "cell73 ~ gene84"
        },
        {
          "x": 72,
          "y": 84,
          "value": 6,
          "name": "cell73 ~ gene85"
        },
        {
          "x": 72,
          "y": 85,
          "value": 0,
          "name": "cell73 ~ gene86"
        },
        {
          "x": 72,
          "y": 86,
          "value": 0,
          "name": "cell73 ~ gene87"
        },
        {
          "x": 72,
          "y": 87,
          "value": 0,
          "name": "cell73 ~ gene88"
        },
        {
          "x": 72,
          "y": 88,
          "value": 0,
          "name": "cell73 ~ gene89"
        },
        {
          "x": 72,
          "y": 89,
          "value": 0,
          "name": "cell73 ~ gene90"
        },
        {
          "x": 72,
          "y": 90,
          "value": 0,
          "name": "cell73 ~ gene91"
        },
        {
          "x": 72,
          "y": 91,
          "value": 11,
          "name": "cell73 ~ gene92"
        },
        {
          "x": 72,
          "y": 92,
          "value": 10,
          "name": "cell73 ~ gene93"
        },
        {
          "x": 72,
          "y": 93,
          "value": 0,
          "name": "cell73 ~ gene94"
        },
        {
          "x": 72,
          "y": 94,
          "value": 6,
          "name": "cell73 ~ gene95"
        },
        {
          "x": 72,
          "y": 95,
          "value": 0,
          "name": "cell73 ~ gene96"
        },
        {
          "x": 72,
          "y": 96,
          "value": 2,
          "name": "cell73 ~ gene97"
        },
        {
          "x": 72,
          "y": 97,
          "value": 2,
          "name": "cell73 ~ gene98"
        },
        {
          "x": 72,
          "y": 98,
          "value": 7,
          "name": "cell73 ~ gene99"
        },
        {
          "x": 72,
          "y": 99,
          "value": 0,
          "name": "cell73 ~ gene100"
        },
        {
          "x": 73,
          "y": 0,
          "value": 0,
          "name": "cell74 ~ gene1"
        },
        {
          "x": 73,
          "y": 1,
          "value": 0,
          "name": "cell74 ~ gene2"
        },
        {
          "x": 73,
          "y": 2,
          "value": 3,
          "name": "cell74 ~ gene3"
        },
        {
          "x": 73,
          "y": 3,
          "value": 8,
          "name": "cell74 ~ gene4"
        },
        {
          "x": 73,
          "y": 4,
          "value": 16,
          "name": "cell74 ~ gene5"
        },
        {
          "x": 73,
          "y": 5,
          "value": 0,
          "name": "cell74 ~ gene6"
        },
        {
          "x": 73,
          "y": 6,
          "value": 0,
          "name": "cell74 ~ gene7"
        },
        {
          "x": 73,
          "y": 7,
          "value": 9,
          "name": "cell74 ~ gene8"
        },
        {
          "x": 73,
          "y": 8,
          "value": 5,
          "name": "cell74 ~ gene9"
        },
        {
          "x": 73,
          "y": 9,
          "value": 0,
          "name": "cell74 ~ gene10"
        },
        {
          "x": 73,
          "y": 10,
          "value": 0,
          "name": "cell74 ~ gene11"
        },
        {
          "x": 73,
          "y": 11,
          "value": 0,
          "name": "cell74 ~ gene12"
        },
        {
          "x": 73,
          "y": 12,
          "value": 0,
          "name": "cell74 ~ gene13"
        },
        {
          "x": 73,
          "y": 13,
          "value": 0,
          "name": "cell74 ~ gene14"
        },
        {
          "x": 73,
          "y": 14,
          "value": 0,
          "name": "cell74 ~ gene15"
        },
        {
          "x": 73,
          "y": 15,
          "value": 17,
          "name": "cell74 ~ gene16"
        },
        {
          "x": 73,
          "y": 16,
          "value": 0,
          "name": "cell74 ~ gene17"
        },
        {
          "x": 73,
          "y": 17,
          "value": 11,
          "name": "cell74 ~ gene18"
        },
        {
          "x": 73,
          "y": 18,
          "value": 0,
          "name": "cell74 ~ gene19"
        },
        {
          "x": 73,
          "y": 19,
          "value": 0,
          "name": "cell74 ~ gene20"
        },
        {
          "x": 73,
          "y": 20,
          "value": 7,
          "name": "cell74 ~ gene21"
        },
        {
          "x": 73,
          "y": 21,
          "value": 0,
          "name": "cell74 ~ gene22"
        },
        {
          "x": 73,
          "y": 22,
          "value": 0,
          "name": "cell74 ~ gene23"
        },
        {
          "x": 73,
          "y": 23,
          "value": 0,
          "name": "cell74 ~ gene24"
        },
        {
          "x": 73,
          "y": 24,
          "value": 0,
          "name": "cell74 ~ gene25"
        },
        {
          "x": 73,
          "y": 25,
          "value": 0,
          "name": "cell74 ~ gene26"
        },
        {
          "x": 73,
          "y": 26,
          "value": 1,
          "name": "cell74 ~ gene27"
        },
        {
          "x": 73,
          "y": 27,
          "value": 0,
          "name": "cell74 ~ gene28"
        },
        {
          "x": 73,
          "y": 28,
          "value": 0,
          "name": "cell74 ~ gene29"
        },
        {
          "x": 73,
          "y": 29,
          "value": 6,
          "name": "cell74 ~ gene30"
        },
        {
          "x": 73,
          "y": 30,
          "value": 0,
          "name": "cell74 ~ gene31"
        },
        {
          "x": 73,
          "y": 31,
          "value": 13,
          "name": "cell74 ~ gene32"
        },
        {
          "x": 73,
          "y": 32,
          "value": 0,
          "name": "cell74 ~ gene33"
        },
        {
          "x": 73,
          "y": 33,
          "value": 9,
          "name": "cell74 ~ gene34"
        },
        {
          "x": 73,
          "y": 34,
          "value": 5,
          "name": "cell74 ~ gene35"
        },
        {
          "x": 73,
          "y": 35,
          "value": 0,
          "name": "cell74 ~ gene36"
        },
        {
          "x": 73,
          "y": 36,
          "value": 16,
          "name": "cell74 ~ gene37"
        },
        {
          "x": 73,
          "y": 37,
          "value": 0,
          "name": "cell74 ~ gene38"
        },
        {
          "x": 73,
          "y": 38,
          "value": 0,
          "name": "cell74 ~ gene39"
        },
        {
          "x": 73,
          "y": 39,
          "value": 0,
          "name": "cell74 ~ gene40"
        },
        {
          "x": 73,
          "y": 40,
          "value": 31,
          "name": "cell74 ~ gene41"
        },
        {
          "x": 73,
          "y": 41,
          "value": 17,
          "name": "cell74 ~ gene42"
        },
        {
          "x": 73,
          "y": 42,
          "value": 0,
          "name": "cell74 ~ gene43"
        },
        {
          "x": 73,
          "y": 43,
          "value": 0,
          "name": "cell74 ~ gene44"
        },
        {
          "x": 73,
          "y": 44,
          "value": 0,
          "name": "cell74 ~ gene45"
        },
        {
          "x": 73,
          "y": 45,
          "value": 5,
          "name": "cell74 ~ gene46"
        },
        {
          "x": 73,
          "y": 46,
          "value": 28,
          "name": "cell74 ~ gene47"
        },
        {
          "x": 73,
          "y": 47,
          "value": 21,
          "name": "cell74 ~ gene48"
        },
        {
          "x": 73,
          "y": 48,
          "value": 0,
          "name": "cell74 ~ gene49"
        },
        {
          "x": 73,
          "y": 49,
          "value": 5,
          "name": "cell74 ~ gene50"
        },
        {
          "x": 73,
          "y": 50,
          "value": 30,
          "name": "cell74 ~ gene51"
        },
        {
          "x": 73,
          "y": 51,
          "value": 12,
          "name": "cell74 ~ gene52"
        },
        {
          "x": 73,
          "y": 52,
          "value": 20,
          "name": "cell74 ~ gene53"
        },
        {
          "x": 73,
          "y": 53,
          "value": 1,
          "name": "cell74 ~ gene54"
        },
        {
          "x": 73,
          "y": 54,
          "value": 4,
          "name": "cell74 ~ gene55"
        },
        {
          "x": 73,
          "y": 55,
          "value": 30,
          "name": "cell74 ~ gene56"
        },
        {
          "x": 73,
          "y": 56,
          "value": 13,
          "name": "cell74 ~ gene57"
        },
        {
          "x": 73,
          "y": 57,
          "value": 33,
          "name": "cell74 ~ gene58"
        },
        {
          "x": 73,
          "y": 58,
          "value": 0,
          "name": "cell74 ~ gene59"
        },
        {
          "x": 73,
          "y": 59,
          "value": 0,
          "name": "cell74 ~ gene60"
        },
        {
          "x": 73,
          "y": 60,
          "value": 0,
          "name": "cell74 ~ gene61"
        },
        {
          "x": 73,
          "y": 61,
          "value": 8,
          "name": "cell74 ~ gene62"
        },
        {
          "x": 73,
          "y": 62,
          "value": 6,
          "name": "cell74 ~ gene63"
        },
        {
          "x": 73,
          "y": 63,
          "value": 0,
          "name": "cell74 ~ gene64"
        },
        {
          "x": 73,
          "y": 64,
          "value": 1,
          "name": "cell74 ~ gene65"
        },
        {
          "x": 73,
          "y": 65,
          "value": 19,
          "name": "cell74 ~ gene66"
        },
        {
          "x": 73,
          "y": 66,
          "value": 0,
          "name": "cell74 ~ gene67"
        },
        {
          "x": 73,
          "y": 67,
          "value": 0,
          "name": "cell74 ~ gene68"
        },
        {
          "x": 73,
          "y": 68,
          "value": 10,
          "name": "cell74 ~ gene69"
        },
        {
          "x": 73,
          "y": 69,
          "value": 0,
          "name": "cell74 ~ gene70"
        },
        {
          "x": 73,
          "y": 70,
          "value": 13,
          "name": "cell74 ~ gene71"
        },
        {
          "x": 73,
          "y": 71,
          "value": 0,
          "name": "cell74 ~ gene72"
        },
        {
          "x": 73,
          "y": 72,
          "value": 6,
          "name": "cell74 ~ gene73"
        },
        {
          "x": 73,
          "y": 73,
          "value": 0,
          "name": "cell74 ~ gene74"
        },
        {
          "x": 73,
          "y": 74,
          "value": 7,
          "name": "cell74 ~ gene75"
        },
        {
          "x": 73,
          "y": 75,
          "value": 5,
          "name": "cell74 ~ gene76"
        },
        {
          "x": 73,
          "y": 76,
          "value": 6,
          "name": "cell74 ~ gene77"
        },
        {
          "x": 73,
          "y": 77,
          "value": 0,
          "name": "cell74 ~ gene78"
        },
        {
          "x": 73,
          "y": 78,
          "value": 6,
          "name": "cell74 ~ gene79"
        },
        {
          "x": 73,
          "y": 79,
          "value": 8,
          "name": "cell74 ~ gene80"
        },
        {
          "x": 73,
          "y": 80,
          "value": 0,
          "name": "cell74 ~ gene81"
        },
        {
          "x": 73,
          "y": 81,
          "value": 0,
          "name": "cell74 ~ gene82"
        },
        {
          "x": 73,
          "y": 82,
          "value": 16,
          "name": "cell74 ~ gene83"
        },
        {
          "x": 73,
          "y": 83,
          "value": 0,
          "name": "cell74 ~ gene84"
        },
        {
          "x": 73,
          "y": 84,
          "value": 0,
          "name": "cell74 ~ gene85"
        },
        {
          "x": 73,
          "y": 85,
          "value": 0,
          "name": "cell74 ~ gene86"
        },
        {
          "x": 73,
          "y": 86,
          "value": 4,
          "name": "cell74 ~ gene87"
        },
        {
          "x": 73,
          "y": 87,
          "value": 0,
          "name": "cell74 ~ gene88"
        },
        {
          "x": 73,
          "y": 88,
          "value": 0,
          "name": "cell74 ~ gene89"
        },
        {
          "x": 73,
          "y": 89,
          "value": 4,
          "name": "cell74 ~ gene90"
        },
        {
          "x": 73,
          "y": 90,
          "value": 0,
          "name": "cell74 ~ gene91"
        },
        {
          "x": 73,
          "y": 91,
          "value": 10,
          "name": "cell74 ~ gene92"
        },
        {
          "x": 73,
          "y": 92,
          "value": 0,
          "name": "cell74 ~ gene93"
        },
        {
          "x": 73,
          "y": 93,
          "value": 3,
          "name": "cell74 ~ gene94"
        },
        {
          "x": 73,
          "y": 94,
          "value": 0,
          "name": "cell74 ~ gene95"
        },
        {
          "x": 73,
          "y": 95,
          "value": 0,
          "name": "cell74 ~ gene96"
        },
        {
          "x": 73,
          "y": 96,
          "value": 7,
          "name": "cell74 ~ gene97"
        },
        {
          "x": 73,
          "y": 97,
          "value": 0,
          "name": "cell74 ~ gene98"
        },
        {
          "x": 73,
          "y": 98,
          "value": 0,
          "name": "cell74 ~ gene99"
        },
        {
          "x": 73,
          "y": 99,
          "value": 9,
          "name": "cell74 ~ gene100"
        },
        {
          "x": 74,
          "y": 0,
          "value": 1,
          "name": "cell75 ~ gene1"
        },
        {
          "x": 74,
          "y": 1,
          "value": 22,
          "name": "cell75 ~ gene2"
        },
        {
          "x": 74,
          "y": 2,
          "value": 0,
          "name": "cell75 ~ gene3"
        },
        {
          "x": 74,
          "y": 3,
          "value": 0,
          "name": "cell75 ~ gene4"
        },
        {
          "x": 74,
          "y": 4,
          "value": 5,
          "name": "cell75 ~ gene5"
        },
        {
          "x": 74,
          "y": 5,
          "value": 3,
          "name": "cell75 ~ gene6"
        },
        {
          "x": 74,
          "y": 6,
          "value": 0,
          "name": "cell75 ~ gene7"
        },
        {
          "x": 74,
          "y": 7,
          "value": 15,
          "name": "cell75 ~ gene8"
        },
        {
          "x": 74,
          "y": 8,
          "value": 22,
          "name": "cell75 ~ gene9"
        },
        {
          "x": 74,
          "y": 9,
          "value": 6,
          "name": "cell75 ~ gene10"
        },
        {
          "x": 74,
          "y": 10,
          "value": 0,
          "name": "cell75 ~ gene11"
        },
        {
          "x": 74,
          "y": 11,
          "value": 0,
          "name": "cell75 ~ gene12"
        },
        {
          "x": 74,
          "y": 12,
          "value": 0,
          "name": "cell75 ~ gene13"
        },
        {
          "x": 74,
          "y": 13,
          "value": 1,
          "name": "cell75 ~ gene14"
        },
        {
          "x": 74,
          "y": 14,
          "value": 0,
          "name": "cell75 ~ gene15"
        },
        {
          "x": 74,
          "y": 15,
          "value": 0,
          "name": "cell75 ~ gene16"
        },
        {
          "x": 74,
          "y": 16,
          "value": 0,
          "name": "cell75 ~ gene17"
        },
        {
          "x": 74,
          "y": 17,
          "value": 1,
          "name": "cell75 ~ gene18"
        },
        {
          "x": 74,
          "y": 18,
          "value": 0,
          "name": "cell75 ~ gene19"
        },
        {
          "x": 74,
          "y": 19,
          "value": 0,
          "name": "cell75 ~ gene20"
        },
        {
          "x": 74,
          "y": 20,
          "value": 0,
          "name": "cell75 ~ gene21"
        },
        {
          "x": 74,
          "y": 21,
          "value": 0,
          "name": "cell75 ~ gene22"
        },
        {
          "x": 74,
          "y": 22,
          "value": 10,
          "name": "cell75 ~ gene23"
        },
        {
          "x": 74,
          "y": 23,
          "value": 14,
          "name": "cell75 ~ gene24"
        },
        {
          "x": 74,
          "y": 24,
          "value": 0,
          "name": "cell75 ~ gene25"
        },
        {
          "x": 74,
          "y": 25,
          "value": 0,
          "name": "cell75 ~ gene26"
        },
        {
          "x": 74,
          "y": 26,
          "value": 0,
          "name": "cell75 ~ gene27"
        },
        {
          "x": 74,
          "y": 27,
          "value": 0,
          "name": "cell75 ~ gene28"
        },
        {
          "x": 74,
          "y": 28,
          "value": 5,
          "name": "cell75 ~ gene29"
        },
        {
          "x": 74,
          "y": 29,
          "value": 1,
          "name": "cell75 ~ gene30"
        },
        {
          "x": 74,
          "y": 30,
          "value": 0,
          "name": "cell75 ~ gene31"
        },
        {
          "x": 74,
          "y": 31,
          "value": 8,
          "name": "cell75 ~ gene32"
        },
        {
          "x": 74,
          "y": 32,
          "value": 0,
          "name": "cell75 ~ gene33"
        },
        {
          "x": 74,
          "y": 33,
          "value": 0,
          "name": "cell75 ~ gene34"
        },
        {
          "x": 74,
          "y": 34,
          "value": 0,
          "name": "cell75 ~ gene35"
        },
        {
          "x": 74,
          "y": 35,
          "value": 0,
          "name": "cell75 ~ gene36"
        },
        {
          "x": 74,
          "y": 36,
          "value": 1,
          "name": "cell75 ~ gene37"
        },
        {
          "x": 74,
          "y": 37,
          "value": 0,
          "name": "cell75 ~ gene38"
        },
        {
          "x": 74,
          "y": 38,
          "value": 0,
          "name": "cell75 ~ gene39"
        },
        {
          "x": 74,
          "y": 39,
          "value": 0,
          "name": "cell75 ~ gene40"
        },
        {
          "x": 74,
          "y": 40,
          "value": 0,
          "name": "cell75 ~ gene41"
        },
        {
          "x": 74,
          "y": 41,
          "value": 0,
          "name": "cell75 ~ gene42"
        },
        {
          "x": 74,
          "y": 42,
          "value": 0,
          "name": "cell75 ~ gene43"
        },
        {
          "x": 74,
          "y": 43,
          "value": 4,
          "name": "cell75 ~ gene44"
        },
        {
          "x": 74,
          "y": 44,
          "value": 14,
          "name": "cell75 ~ gene45"
        },
        {
          "x": 74,
          "y": 45,
          "value": 31,
          "name": "cell75 ~ gene46"
        },
        {
          "x": 74,
          "y": 46,
          "value": 19,
          "name": "cell75 ~ gene47"
        },
        {
          "x": 74,
          "y": 47,
          "value": 10,
          "name": "cell75 ~ gene48"
        },
        {
          "x": 74,
          "y": 48,
          "value": 6,
          "name": "cell75 ~ gene49"
        },
        {
          "x": 74,
          "y": 49,
          "value": 10,
          "name": "cell75 ~ gene50"
        },
        {
          "x": 74,
          "y": 50,
          "value": 12,
          "name": "cell75 ~ gene51"
        },
        {
          "x": 74,
          "y": 51,
          "value": 12,
          "name": "cell75 ~ gene52"
        },
        {
          "x": 74,
          "y": 52,
          "value": 3,
          "name": "cell75 ~ gene53"
        },
        {
          "x": 74,
          "y": 53,
          "value": 11,
          "name": "cell75 ~ gene54"
        },
        {
          "x": 74,
          "y": 54,
          "value": 25,
          "name": "cell75 ~ gene55"
        },
        {
          "x": 74,
          "y": 55,
          "value": 41,
          "name": "cell75 ~ gene56"
        },
        {
          "x": 74,
          "y": 56,
          "value": 11,
          "name": "cell75 ~ gene57"
        },
        {
          "x": 74,
          "y": 57,
          "value": 22,
          "name": "cell75 ~ gene58"
        },
        {
          "x": 74,
          "y": 58,
          "value": 2,
          "name": "cell75 ~ gene59"
        },
        {
          "x": 74,
          "y": 59,
          "value": 0,
          "name": "cell75 ~ gene60"
        },
        {
          "x": 74,
          "y": 60,
          "value": 0,
          "name": "cell75 ~ gene61"
        },
        {
          "x": 74,
          "y": 61,
          "value": 14,
          "name": "cell75 ~ gene62"
        },
        {
          "x": 74,
          "y": 62,
          "value": 8,
          "name": "cell75 ~ gene63"
        },
        {
          "x": 74,
          "y": 63,
          "value": 8,
          "name": "cell75 ~ gene64"
        },
        {
          "x": 74,
          "y": 64,
          "value": 11,
          "name": "cell75 ~ gene65"
        },
        {
          "x": 74,
          "y": 65,
          "value": 0,
          "name": "cell75 ~ gene66"
        },
        {
          "x": 74,
          "y": 66,
          "value": 0,
          "name": "cell75 ~ gene67"
        },
        {
          "x": 74,
          "y": 67,
          "value": 0,
          "name": "cell75 ~ gene68"
        },
        {
          "x": 74,
          "y": 68,
          "value": 0,
          "name": "cell75 ~ gene69"
        },
        {
          "x": 74,
          "y": 69,
          "value": 0,
          "name": "cell75 ~ gene70"
        },
        {
          "x": 74,
          "y": 70,
          "value": 0,
          "name": "cell75 ~ gene71"
        },
        {
          "x": 74,
          "y": 71,
          "value": 0,
          "name": "cell75 ~ gene72"
        },
        {
          "x": 74,
          "y": 72,
          "value": 0,
          "name": "cell75 ~ gene73"
        },
        {
          "x": 74,
          "y": 73,
          "value": 1,
          "name": "cell75 ~ gene74"
        },
        {
          "x": 74,
          "y": 74,
          "value": 6,
          "name": "cell75 ~ gene75"
        },
        {
          "x": 74,
          "y": 75,
          "value": 0,
          "name": "cell75 ~ gene76"
        },
        {
          "x": 74,
          "y": 76,
          "value": 11,
          "name": "cell75 ~ gene77"
        },
        {
          "x": 74,
          "y": 77,
          "value": 5,
          "name": "cell75 ~ gene78"
        },
        {
          "x": 74,
          "y": 78,
          "value": 8,
          "name": "cell75 ~ gene79"
        },
        {
          "x": 74,
          "y": 79,
          "value": 0,
          "name": "cell75 ~ gene80"
        },
        {
          "x": 74,
          "y": 80,
          "value": 4,
          "name": "cell75 ~ gene81"
        },
        {
          "x": 74,
          "y": 81,
          "value": 0,
          "name": "cell75 ~ gene82"
        },
        {
          "x": 74,
          "y": 82,
          "value": 3,
          "name": "cell75 ~ gene83"
        },
        {
          "x": 74,
          "y": 83,
          "value": 8,
          "name": "cell75 ~ gene84"
        },
        {
          "x": 74,
          "y": 84,
          "value": 4,
          "name": "cell75 ~ gene85"
        },
        {
          "x": 74,
          "y": 85,
          "value": 0,
          "name": "cell75 ~ gene86"
        },
        {
          "x": 74,
          "y": 86,
          "value": 0,
          "name": "cell75 ~ gene87"
        },
        {
          "x": 74,
          "y": 87,
          "value": 0,
          "name": "cell75 ~ gene88"
        },
        {
          "x": 74,
          "y": 88,
          "value": 3,
          "name": "cell75 ~ gene89"
        },
        {
          "x": 74,
          "y": 89,
          "value": 3,
          "name": "cell75 ~ gene90"
        },
        {
          "x": 74,
          "y": 90,
          "value": 0,
          "name": "cell75 ~ gene91"
        },
        {
          "x": 74,
          "y": 91,
          "value": 0,
          "name": "cell75 ~ gene92"
        },
        {
          "x": 74,
          "y": 92,
          "value": 3,
          "name": "cell75 ~ gene93"
        },
        {
          "x": 74,
          "y": 93,
          "value": 5,
          "name": "cell75 ~ gene94"
        },
        {
          "x": 74,
          "y": 94,
          "value": 5,
          "name": "cell75 ~ gene95"
        },
        {
          "x": 74,
          "y": 95,
          "value": 0,
          "name": "cell75 ~ gene96"
        },
        {
          "x": 74,
          "y": 96,
          "value": 11,
          "name": "cell75 ~ gene97"
        },
        {
          "x": 74,
          "y": 97,
          "value": 1,
          "name": "cell75 ~ gene98"
        },
        {
          "x": 74,
          "y": 98,
          "value": 0,
          "name": "cell75 ~ gene99"
        },
        {
          "x": 74,
          "y": 99,
          "value": 15,
          "name": "cell75 ~ gene100"
        },
        {
          "x": 75,
          "y": 0,
          "value": 8,
          "name": "cell76 ~ gene1"
        },
        {
          "x": 75,
          "y": 1,
          "value": 19,
          "name": "cell76 ~ gene2"
        },
        {
          "x": 75,
          "y": 2,
          "value": 0,
          "name": "cell76 ~ gene3"
        },
        {
          "x": 75,
          "y": 3,
          "value": 0,
          "name": "cell76 ~ gene4"
        },
        {
          "x": 75,
          "y": 4,
          "value": 2,
          "name": "cell76 ~ gene5"
        },
        {
          "x": 75,
          "y": 5,
          "value": 0,
          "name": "cell76 ~ gene6"
        },
        {
          "x": 75,
          "y": 6,
          "value": 6,
          "name": "cell76 ~ gene7"
        },
        {
          "x": 75,
          "y": 7,
          "value": 0,
          "name": "cell76 ~ gene8"
        },
        {
          "x": 75,
          "y": 8,
          "value": 4,
          "name": "cell76 ~ gene9"
        },
        {
          "x": 75,
          "y": 9,
          "value": 18,
          "name": "cell76 ~ gene10"
        },
        {
          "x": 75,
          "y": 10,
          "value": 0,
          "name": "cell76 ~ gene11"
        },
        {
          "x": 75,
          "y": 11,
          "value": 0,
          "name": "cell76 ~ gene12"
        },
        {
          "x": 75,
          "y": 12,
          "value": 0,
          "name": "cell76 ~ gene13"
        },
        {
          "x": 75,
          "y": 13,
          "value": 10,
          "name": "cell76 ~ gene14"
        },
        {
          "x": 75,
          "y": 14,
          "value": 0,
          "name": "cell76 ~ gene15"
        },
        {
          "x": 75,
          "y": 15,
          "value": 2,
          "name": "cell76 ~ gene16"
        },
        {
          "x": 75,
          "y": 16,
          "value": 8,
          "name": "cell76 ~ gene17"
        },
        {
          "x": 75,
          "y": 17,
          "value": 0,
          "name": "cell76 ~ gene18"
        },
        {
          "x": 75,
          "y": 18,
          "value": 5,
          "name": "cell76 ~ gene19"
        },
        {
          "x": 75,
          "y": 19,
          "value": 0,
          "name": "cell76 ~ gene20"
        },
        {
          "x": 75,
          "y": 20,
          "value": 0,
          "name": "cell76 ~ gene21"
        },
        {
          "x": 75,
          "y": 21,
          "value": 2,
          "name": "cell76 ~ gene22"
        },
        {
          "x": 75,
          "y": 22,
          "value": 0,
          "name": "cell76 ~ gene23"
        },
        {
          "x": 75,
          "y": 23,
          "value": 0,
          "name": "cell76 ~ gene24"
        },
        {
          "x": 75,
          "y": 24,
          "value": 10,
          "name": "cell76 ~ gene25"
        },
        {
          "x": 75,
          "y": 25,
          "value": 7,
          "name": "cell76 ~ gene26"
        },
        {
          "x": 75,
          "y": 26,
          "value": 0,
          "name": "cell76 ~ gene27"
        },
        {
          "x": 75,
          "y": 27,
          "value": 11,
          "name": "cell76 ~ gene28"
        },
        {
          "x": 75,
          "y": 28,
          "value": 0,
          "name": "cell76 ~ gene29"
        },
        {
          "x": 75,
          "y": 29,
          "value": 8,
          "name": "cell76 ~ gene30"
        },
        {
          "x": 75,
          "y": 30,
          "value": 0,
          "name": "cell76 ~ gene31"
        },
        {
          "x": 75,
          "y": 31,
          "value": 6,
          "name": "cell76 ~ gene32"
        },
        {
          "x": 75,
          "y": 32,
          "value": 0,
          "name": "cell76 ~ gene33"
        },
        {
          "x": 75,
          "y": 33,
          "value": 0,
          "name": "cell76 ~ gene34"
        },
        {
          "x": 75,
          "y": 34,
          "value": 16,
          "name": "cell76 ~ gene35"
        },
        {
          "x": 75,
          "y": 35,
          "value": 0,
          "name": "cell76 ~ gene36"
        },
        {
          "x": 75,
          "y": 36,
          "value": 19,
          "name": "cell76 ~ gene37"
        },
        {
          "x": 75,
          "y": 37,
          "value": 17,
          "name": "cell76 ~ gene38"
        },
        {
          "x": 75,
          "y": 38,
          "value": 16,
          "name": "cell76 ~ gene39"
        },
        {
          "x": 75,
          "y": 39,
          "value": 0,
          "name": "cell76 ~ gene40"
        },
        {
          "x": 75,
          "y": 40,
          "value": 5,
          "name": "cell76 ~ gene41"
        },
        {
          "x": 75,
          "y": 41,
          "value": 27,
          "name": "cell76 ~ gene42"
        },
        {
          "x": 75,
          "y": 42,
          "value": 2,
          "name": "cell76 ~ gene43"
        },
        {
          "x": 75,
          "y": 43,
          "value": 0,
          "name": "cell76 ~ gene44"
        },
        {
          "x": 75,
          "y": 44,
          "value": 5,
          "name": "cell76 ~ gene45"
        },
        {
          "x": 75,
          "y": 45,
          "value": 26,
          "name": "cell76 ~ gene46"
        },
        {
          "x": 75,
          "y": 46,
          "value": 7,
          "name": "cell76 ~ gene47"
        },
        {
          "x": 75,
          "y": 47,
          "value": 15,
          "name": "cell76 ~ gene48"
        },
        {
          "x": 75,
          "y": 48,
          "value": 0,
          "name": "cell76 ~ gene49"
        },
        {
          "x": 75,
          "y": 49,
          "value": 3,
          "name": "cell76 ~ gene50"
        },
        {
          "x": 75,
          "y": 50,
          "value": 25,
          "name": "cell76 ~ gene51"
        },
        {
          "x": 75,
          "y": 51,
          "value": 35,
          "name": "cell76 ~ gene52"
        },
        {
          "x": 75,
          "y": 52,
          "value": 0,
          "name": "cell76 ~ gene53"
        },
        {
          "x": 75,
          "y": 53,
          "value": 19,
          "name": "cell76 ~ gene54"
        },
        {
          "x": 75,
          "y": 54,
          "value": 27,
          "name": "cell76 ~ gene55"
        },
        {
          "x": 75,
          "y": 55,
          "value": 30,
          "name": "cell76 ~ gene56"
        },
        {
          "x": 75,
          "y": 56,
          "value": 0,
          "name": "cell76 ~ gene57"
        },
        {
          "x": 75,
          "y": 57,
          "value": 4,
          "name": "cell76 ~ gene58"
        },
        {
          "x": 75,
          "y": 58,
          "value": 2,
          "name": "cell76 ~ gene59"
        },
        {
          "x": 75,
          "y": 59,
          "value": 18,
          "name": "cell76 ~ gene60"
        },
        {
          "x": 75,
          "y": 60,
          "value": 0,
          "name": "cell76 ~ gene61"
        },
        {
          "x": 75,
          "y": 61,
          "value": 0,
          "name": "cell76 ~ gene62"
        },
        {
          "x": 75,
          "y": 62,
          "value": 21,
          "name": "cell76 ~ gene63"
        },
        {
          "x": 75,
          "y": 63,
          "value": 3,
          "name": "cell76 ~ gene64"
        },
        {
          "x": 75,
          "y": 64,
          "value": 0,
          "name": "cell76 ~ gene65"
        },
        {
          "x": 75,
          "y": 65,
          "value": 4,
          "name": "cell76 ~ gene66"
        },
        {
          "x": 75,
          "y": 66,
          "value": 9,
          "name": "cell76 ~ gene67"
        },
        {
          "x": 75,
          "y": 67,
          "value": 0,
          "name": "cell76 ~ gene68"
        },
        {
          "x": 75,
          "y": 68,
          "value": 0,
          "name": "cell76 ~ gene69"
        },
        {
          "x": 75,
          "y": 69,
          "value": 0,
          "name": "cell76 ~ gene70"
        },
        {
          "x": 75,
          "y": 70,
          "value": 6,
          "name": "cell76 ~ gene71"
        },
        {
          "x": 75,
          "y": 71,
          "value": 0,
          "name": "cell76 ~ gene72"
        },
        {
          "x": 75,
          "y": 72,
          "value": 2,
          "name": "cell76 ~ gene73"
        },
        {
          "x": 75,
          "y": 73,
          "value": 8,
          "name": "cell76 ~ gene74"
        },
        {
          "x": 75,
          "y": 74,
          "value": 0,
          "name": "cell76 ~ gene75"
        },
        {
          "x": 75,
          "y": 75,
          "value": 0,
          "name": "cell76 ~ gene76"
        },
        {
          "x": 75,
          "y": 76,
          "value": 0,
          "name": "cell76 ~ gene77"
        },
        {
          "x": 75,
          "y": 77,
          "value": 0,
          "name": "cell76 ~ gene78"
        },
        {
          "x": 75,
          "y": 78,
          "value": 0,
          "name": "cell76 ~ gene79"
        },
        {
          "x": 75,
          "y": 79,
          "value": 5,
          "name": "cell76 ~ gene80"
        },
        {
          "x": 75,
          "y": 80,
          "value": 0,
          "name": "cell76 ~ gene81"
        },
        {
          "x": 75,
          "y": 81,
          "value": 16,
          "name": "cell76 ~ gene82"
        },
        {
          "x": 75,
          "y": 82,
          "value": 4,
          "name": "cell76 ~ gene83"
        },
        {
          "x": 75,
          "y": 83,
          "value": 4,
          "name": "cell76 ~ gene84"
        },
        {
          "x": 75,
          "y": 84,
          "value": 0,
          "name": "cell76 ~ gene85"
        },
        {
          "x": 75,
          "y": 85,
          "value": 0,
          "name": "cell76 ~ gene86"
        },
        {
          "x": 75,
          "y": 86,
          "value": 6,
          "name": "cell76 ~ gene87"
        },
        {
          "x": 75,
          "y": 87,
          "value": 1,
          "name": "cell76 ~ gene88"
        },
        {
          "x": 75,
          "y": 88,
          "value": 0,
          "name": "cell76 ~ gene89"
        },
        {
          "x": 75,
          "y": 89,
          "value": 0,
          "name": "cell76 ~ gene90"
        },
        {
          "x": 75,
          "y": 90,
          "value": 0,
          "name": "cell76 ~ gene91"
        },
        {
          "x": 75,
          "y": 91,
          "value": 16,
          "name": "cell76 ~ gene92"
        },
        {
          "x": 75,
          "y": 92,
          "value": 0,
          "name": "cell76 ~ gene93"
        },
        {
          "x": 75,
          "y": 93,
          "value": 2,
          "name": "cell76 ~ gene94"
        },
        {
          "x": 75,
          "y": 94,
          "value": 3,
          "name": "cell76 ~ gene95"
        },
        {
          "x": 75,
          "y": 95,
          "value": 32,
          "name": "cell76 ~ gene96"
        },
        {
          "x": 75,
          "y": 96,
          "value": 5,
          "name": "cell76 ~ gene97"
        },
        {
          "x": 75,
          "y": 97,
          "value": 14,
          "name": "cell76 ~ gene98"
        },
        {
          "x": 75,
          "y": 98,
          "value": 15,
          "name": "cell76 ~ gene99"
        },
        {
          "x": 75,
          "y": 99,
          "value": 10,
          "name": "cell76 ~ gene100"
        },
        {
          "x": 76,
          "y": 0,
          "value": 0,
          "name": "cell77 ~ gene1"
        },
        {
          "x": 76,
          "y": 1,
          "value": 8,
          "name": "cell77 ~ gene2"
        },
        {
          "x": 76,
          "y": 2,
          "value": 11,
          "name": "cell77 ~ gene3"
        },
        {
          "x": 76,
          "y": 3,
          "value": 0,
          "name": "cell77 ~ gene4"
        },
        {
          "x": 76,
          "y": 4,
          "value": 0,
          "name": "cell77 ~ gene5"
        },
        {
          "x": 76,
          "y": 5,
          "value": 0,
          "name": "cell77 ~ gene6"
        },
        {
          "x": 76,
          "y": 6,
          "value": 2,
          "name": "cell77 ~ gene7"
        },
        {
          "x": 76,
          "y": 7,
          "value": 1,
          "name": "cell77 ~ gene8"
        },
        {
          "x": 76,
          "y": 8,
          "value": 1,
          "name": "cell77 ~ gene9"
        },
        {
          "x": 76,
          "y": 9,
          "value": 17,
          "name": "cell77 ~ gene10"
        },
        {
          "x": 76,
          "y": 10,
          "value": 2,
          "name": "cell77 ~ gene11"
        },
        {
          "x": 76,
          "y": 11,
          "value": 0,
          "name": "cell77 ~ gene12"
        },
        {
          "x": 76,
          "y": 12,
          "value": 3,
          "name": "cell77 ~ gene13"
        },
        {
          "x": 76,
          "y": 13,
          "value": 7,
          "name": "cell77 ~ gene14"
        },
        {
          "x": 76,
          "y": 14,
          "value": 0,
          "name": "cell77 ~ gene15"
        },
        {
          "x": 76,
          "y": 15,
          "value": 12,
          "name": "cell77 ~ gene16"
        },
        {
          "x": 76,
          "y": 16,
          "value": 0,
          "name": "cell77 ~ gene17"
        },
        {
          "x": 76,
          "y": 17,
          "value": 8,
          "name": "cell77 ~ gene18"
        },
        {
          "x": 76,
          "y": 18,
          "value": 4,
          "name": "cell77 ~ gene19"
        },
        {
          "x": 76,
          "y": 19,
          "value": 17,
          "name": "cell77 ~ gene20"
        },
        {
          "x": 76,
          "y": 20,
          "value": 9,
          "name": "cell77 ~ gene21"
        },
        {
          "x": 76,
          "y": 21,
          "value": 1,
          "name": "cell77 ~ gene22"
        },
        {
          "x": 76,
          "y": 22,
          "value": 0,
          "name": "cell77 ~ gene23"
        },
        {
          "x": 76,
          "y": 23,
          "value": 0,
          "name": "cell77 ~ gene24"
        },
        {
          "x": 76,
          "y": 24,
          "value": 13,
          "name": "cell77 ~ gene25"
        },
        {
          "x": 76,
          "y": 25,
          "value": 2,
          "name": "cell77 ~ gene26"
        },
        {
          "x": 76,
          "y": 26,
          "value": 9,
          "name": "cell77 ~ gene27"
        },
        {
          "x": 76,
          "y": 27,
          "value": 0,
          "name": "cell77 ~ gene28"
        },
        {
          "x": 76,
          "y": 28,
          "value": 28,
          "name": "cell77 ~ gene29"
        },
        {
          "x": 76,
          "y": 29,
          "value": 0,
          "name": "cell77 ~ gene30"
        },
        {
          "x": 76,
          "y": 30,
          "value": 14,
          "name": "cell77 ~ gene31"
        },
        {
          "x": 76,
          "y": 31,
          "value": 1,
          "name": "cell77 ~ gene32"
        },
        {
          "x": 76,
          "y": 32,
          "value": 13,
          "name": "cell77 ~ gene33"
        },
        {
          "x": 76,
          "y": 33,
          "value": 9,
          "name": "cell77 ~ gene34"
        },
        {
          "x": 76,
          "y": 34,
          "value": 10,
          "name": "cell77 ~ gene35"
        },
        {
          "x": 76,
          "y": 35,
          "value": 3,
          "name": "cell77 ~ gene36"
        },
        {
          "x": 76,
          "y": 36,
          "value": 0,
          "name": "cell77 ~ gene37"
        },
        {
          "x": 76,
          "y": 37,
          "value": 13,
          "name": "cell77 ~ gene38"
        },
        {
          "x": 76,
          "y": 38,
          "value": 10,
          "name": "cell77 ~ gene39"
        },
        {
          "x": 76,
          "y": 39,
          "value": 0,
          "name": "cell77 ~ gene40"
        },
        {
          "x": 76,
          "y": 40,
          "value": 23,
          "name": "cell77 ~ gene41"
        },
        {
          "x": 76,
          "y": 41,
          "value": 17,
          "name": "cell77 ~ gene42"
        },
        {
          "x": 76,
          "y": 42,
          "value": 5,
          "name": "cell77 ~ gene43"
        },
        {
          "x": 76,
          "y": 43,
          "value": 6,
          "name": "cell77 ~ gene44"
        },
        {
          "x": 76,
          "y": 44,
          "value": 5,
          "name": "cell77 ~ gene45"
        },
        {
          "x": 76,
          "y": 45,
          "value": 35,
          "name": "cell77 ~ gene46"
        },
        {
          "x": 76,
          "y": 46,
          "value": 24,
          "name": "cell77 ~ gene47"
        },
        {
          "x": 76,
          "y": 47,
          "value": 0,
          "name": "cell77 ~ gene48"
        },
        {
          "x": 76,
          "y": 48,
          "value": 0,
          "name": "cell77 ~ gene49"
        },
        {
          "x": 76,
          "y": 49,
          "value": 30,
          "name": "cell77 ~ gene50"
        },
        {
          "x": 76,
          "y": 50,
          "value": 12,
          "name": "cell77 ~ gene51"
        },
        {
          "x": 76,
          "y": 51,
          "value": 0,
          "name": "cell77 ~ gene52"
        },
        {
          "x": 76,
          "y": 52,
          "value": 6,
          "name": "cell77 ~ gene53"
        },
        {
          "x": 76,
          "y": 53,
          "value": 0,
          "name": "cell77 ~ gene54"
        },
        {
          "x": 76,
          "y": 54,
          "value": 8,
          "name": "cell77 ~ gene55"
        },
        {
          "x": 76,
          "y": 55,
          "value": 28,
          "name": "cell77 ~ gene56"
        },
        {
          "x": 76,
          "y": 56,
          "value": 0,
          "name": "cell77 ~ gene57"
        },
        {
          "x": 76,
          "y": 57,
          "value": 12,
          "name": "cell77 ~ gene58"
        },
        {
          "x": 76,
          "y": 58,
          "value": 7,
          "name": "cell77 ~ gene59"
        },
        {
          "x": 76,
          "y": 59,
          "value": 0,
          "name": "cell77 ~ gene60"
        },
        {
          "x": 76,
          "y": 60,
          "value": 16,
          "name": "cell77 ~ gene61"
        },
        {
          "x": 76,
          "y": 61,
          "value": 3,
          "name": "cell77 ~ gene62"
        },
        {
          "x": 76,
          "y": 62,
          "value": 0,
          "name": "cell77 ~ gene63"
        },
        {
          "x": 76,
          "y": 63,
          "value": 2,
          "name": "cell77 ~ gene64"
        },
        {
          "x": 76,
          "y": 64,
          "value": 0,
          "name": "cell77 ~ gene65"
        },
        {
          "x": 76,
          "y": 65,
          "value": 20,
          "name": "cell77 ~ gene66"
        },
        {
          "x": 76,
          "y": 66,
          "value": 9,
          "name": "cell77 ~ gene67"
        },
        {
          "x": 76,
          "y": 67,
          "value": 5,
          "name": "cell77 ~ gene68"
        },
        {
          "x": 76,
          "y": 68,
          "value": 2,
          "name": "cell77 ~ gene69"
        },
        {
          "x": 76,
          "y": 69,
          "value": 0,
          "name": "cell77 ~ gene70"
        },
        {
          "x": 76,
          "y": 70,
          "value": 2,
          "name": "cell77 ~ gene71"
        },
        {
          "x": 76,
          "y": 71,
          "value": 0,
          "name": "cell77 ~ gene72"
        },
        {
          "x": 76,
          "y": 72,
          "value": 15,
          "name": "cell77 ~ gene73"
        },
        {
          "x": 76,
          "y": 73,
          "value": 16,
          "name": "cell77 ~ gene74"
        },
        {
          "x": 76,
          "y": 74,
          "value": 3,
          "name": "cell77 ~ gene75"
        },
        {
          "x": 76,
          "y": 75,
          "value": 0,
          "name": "cell77 ~ gene76"
        },
        {
          "x": 76,
          "y": 76,
          "value": 0,
          "name": "cell77 ~ gene77"
        },
        {
          "x": 76,
          "y": 77,
          "value": 0,
          "name": "cell77 ~ gene78"
        },
        {
          "x": 76,
          "y": 78,
          "value": 13,
          "name": "cell77 ~ gene79"
        },
        {
          "x": 76,
          "y": 79,
          "value": 0,
          "name": "cell77 ~ gene80"
        },
        {
          "x": 76,
          "y": 80,
          "value": 5,
          "name": "cell77 ~ gene81"
        },
        {
          "x": 76,
          "y": 81,
          "value": 9,
          "name": "cell77 ~ gene82"
        },
        {
          "x": 76,
          "y": 82,
          "value": 8,
          "name": "cell77 ~ gene83"
        },
        {
          "x": 76,
          "y": 83,
          "value": 0,
          "name": "cell77 ~ gene84"
        },
        {
          "x": 76,
          "y": 84,
          "value": 0,
          "name": "cell77 ~ gene85"
        },
        {
          "x": 76,
          "y": 85,
          "value": 0,
          "name": "cell77 ~ gene86"
        },
        {
          "x": 76,
          "y": 86,
          "value": 3,
          "name": "cell77 ~ gene87"
        },
        {
          "x": 76,
          "y": 87,
          "value": 0,
          "name": "cell77 ~ gene88"
        },
        {
          "x": 76,
          "y": 88,
          "value": 5,
          "name": "cell77 ~ gene89"
        },
        {
          "x": 76,
          "y": 89,
          "value": 0,
          "name": "cell77 ~ gene90"
        },
        {
          "x": 76,
          "y": 90,
          "value": 5,
          "name": "cell77 ~ gene91"
        },
        {
          "x": 76,
          "y": 91,
          "value": 20,
          "name": "cell77 ~ gene92"
        },
        {
          "x": 76,
          "y": 92,
          "value": 0,
          "name": "cell77 ~ gene93"
        },
        {
          "x": 76,
          "y": 93,
          "value": 0,
          "name": "cell77 ~ gene94"
        },
        {
          "x": 76,
          "y": 94,
          "value": 0,
          "name": "cell77 ~ gene95"
        },
        {
          "x": 76,
          "y": 95,
          "value": 0,
          "name": "cell77 ~ gene96"
        },
        {
          "x": 76,
          "y": 96,
          "value": 14,
          "name": "cell77 ~ gene97"
        },
        {
          "x": 76,
          "y": 97,
          "value": 0,
          "name": "cell77 ~ gene98"
        },
        {
          "x": 76,
          "y": 98,
          "value": 7,
          "name": "cell77 ~ gene99"
        },
        {
          "x": 76,
          "y": 99,
          "value": 6,
          "name": "cell77 ~ gene100"
        },
        {
          "x": 77,
          "y": 0,
          "value": 18,
          "name": "cell78 ~ gene1"
        },
        {
          "x": 77,
          "y": 1,
          "value": 18,
          "name": "cell78 ~ gene2"
        },
        {
          "x": 77,
          "y": 2,
          "value": 0,
          "name": "cell78 ~ gene3"
        },
        {
          "x": 77,
          "y": 3,
          "value": 7,
          "name": "cell78 ~ gene4"
        },
        {
          "x": 77,
          "y": 4,
          "value": 0,
          "name": "cell78 ~ gene5"
        },
        {
          "x": 77,
          "y": 5,
          "value": 20,
          "name": "cell78 ~ gene6"
        },
        {
          "x": 77,
          "y": 6,
          "value": 0,
          "name": "cell78 ~ gene7"
        },
        {
          "x": 77,
          "y": 7,
          "value": 13,
          "name": "cell78 ~ gene8"
        },
        {
          "x": 77,
          "y": 8,
          "value": 0,
          "name": "cell78 ~ gene9"
        },
        {
          "x": 77,
          "y": 9,
          "value": 12,
          "name": "cell78 ~ gene10"
        },
        {
          "x": 77,
          "y": 10,
          "value": 11,
          "name": "cell78 ~ gene11"
        },
        {
          "x": 77,
          "y": 11,
          "value": 13,
          "name": "cell78 ~ gene12"
        },
        {
          "x": 77,
          "y": 12,
          "value": 14,
          "name": "cell78 ~ gene13"
        },
        {
          "x": 77,
          "y": 13,
          "value": 3,
          "name": "cell78 ~ gene14"
        },
        {
          "x": 77,
          "y": 14,
          "value": 0,
          "name": "cell78 ~ gene15"
        },
        {
          "x": 77,
          "y": 15,
          "value": 9,
          "name": "cell78 ~ gene16"
        },
        {
          "x": 77,
          "y": 16,
          "value": 0,
          "name": "cell78 ~ gene17"
        },
        {
          "x": 77,
          "y": 17,
          "value": 0,
          "name": "cell78 ~ gene18"
        },
        {
          "x": 77,
          "y": 18,
          "value": 3,
          "name": "cell78 ~ gene19"
        },
        {
          "x": 77,
          "y": 19,
          "value": 0,
          "name": "cell78 ~ gene20"
        },
        {
          "x": 77,
          "y": 20,
          "value": 20,
          "name": "cell78 ~ gene21"
        },
        {
          "x": 77,
          "y": 21,
          "value": 8,
          "name": "cell78 ~ gene22"
        },
        {
          "x": 77,
          "y": 22,
          "value": 0,
          "name": "cell78 ~ gene23"
        },
        {
          "x": 77,
          "y": 23,
          "value": 11,
          "name": "cell78 ~ gene24"
        },
        {
          "x": 77,
          "y": 24,
          "value": 23,
          "name": "cell78 ~ gene25"
        },
        {
          "x": 77,
          "y": 25,
          "value": 0,
          "name": "cell78 ~ gene26"
        },
        {
          "x": 77,
          "y": 26,
          "value": 25,
          "name": "cell78 ~ gene27"
        },
        {
          "x": 77,
          "y": 27,
          "value": 2,
          "name": "cell78 ~ gene28"
        },
        {
          "x": 77,
          "y": 28,
          "value": 0,
          "name": "cell78 ~ gene29"
        },
        {
          "x": 77,
          "y": 29,
          "value": 0,
          "name": "cell78 ~ gene30"
        },
        {
          "x": 77,
          "y": 30,
          "value": 0,
          "name": "cell78 ~ gene31"
        },
        {
          "x": 77,
          "y": 31,
          "value": 9,
          "name": "cell78 ~ gene32"
        },
        {
          "x": 77,
          "y": 32,
          "value": 11,
          "name": "cell78 ~ gene33"
        },
        {
          "x": 77,
          "y": 33,
          "value": 0,
          "name": "cell78 ~ gene34"
        },
        {
          "x": 77,
          "y": 34,
          "value": 11,
          "name": "cell78 ~ gene35"
        },
        {
          "x": 77,
          "y": 35,
          "value": 0,
          "name": "cell78 ~ gene36"
        },
        {
          "x": 77,
          "y": 36,
          "value": 1,
          "name": "cell78 ~ gene37"
        },
        {
          "x": 77,
          "y": 37,
          "value": 19,
          "name": "cell78 ~ gene38"
        },
        {
          "x": 77,
          "y": 38,
          "value": 0,
          "name": "cell78 ~ gene39"
        },
        {
          "x": 77,
          "y": 39,
          "value": 0,
          "name": "cell78 ~ gene40"
        },
        {
          "x": 77,
          "y": 40,
          "value": 10,
          "name": "cell78 ~ gene41"
        },
        {
          "x": 77,
          "y": 41,
          "value": 35,
          "name": "cell78 ~ gene42"
        },
        {
          "x": 77,
          "y": 42,
          "value": 0,
          "name": "cell78 ~ gene43"
        },
        {
          "x": 77,
          "y": 43,
          "value": 0,
          "name": "cell78 ~ gene44"
        },
        {
          "x": 77,
          "y": 44,
          "value": 20,
          "name": "cell78 ~ gene45"
        },
        {
          "x": 77,
          "y": 45,
          "value": 12,
          "name": "cell78 ~ gene46"
        },
        {
          "x": 77,
          "y": 46,
          "value": 3,
          "name": "cell78 ~ gene47"
        },
        {
          "x": 77,
          "y": 47,
          "value": 24,
          "name": "cell78 ~ gene48"
        },
        {
          "x": 77,
          "y": 48,
          "value": 0,
          "name": "cell78 ~ gene49"
        },
        {
          "x": 77,
          "y": 49,
          "value": 2,
          "name": "cell78 ~ gene50"
        },
        {
          "x": 77,
          "y": 50,
          "value": 20,
          "name": "cell78 ~ gene51"
        },
        {
          "x": 77,
          "y": 51,
          "value": 29,
          "name": "cell78 ~ gene52"
        },
        {
          "x": 77,
          "y": 52,
          "value": 6,
          "name": "cell78 ~ gene53"
        },
        {
          "x": 77,
          "y": 53,
          "value": 30,
          "name": "cell78 ~ gene54"
        },
        {
          "x": 77,
          "y": 54,
          "value": 30,
          "name": "cell78 ~ gene55"
        },
        {
          "x": 77,
          "y": 55,
          "value": 17,
          "name": "cell78 ~ gene56"
        },
        {
          "x": 77,
          "y": 56,
          "value": 14,
          "name": "cell78 ~ gene57"
        },
        {
          "x": 77,
          "y": 57,
          "value": 13,
          "name": "cell78 ~ gene58"
        },
        {
          "x": 77,
          "y": 58,
          "value": 4,
          "name": "cell78 ~ gene59"
        },
        {
          "x": 77,
          "y": 59,
          "value": 7,
          "name": "cell78 ~ gene60"
        },
        {
          "x": 77,
          "y": 60,
          "value": 0,
          "name": "cell78 ~ gene61"
        },
        {
          "x": 77,
          "y": 61,
          "value": 14,
          "name": "cell78 ~ gene62"
        },
        {
          "x": 77,
          "y": 62,
          "value": 0,
          "name": "cell78 ~ gene63"
        },
        {
          "x": 77,
          "y": 63,
          "value": 1,
          "name": "cell78 ~ gene64"
        },
        {
          "x": 77,
          "y": 64,
          "value": 13,
          "name": "cell78 ~ gene65"
        },
        {
          "x": 77,
          "y": 65,
          "value": 0,
          "name": "cell78 ~ gene66"
        },
        {
          "x": 77,
          "y": 66,
          "value": 6,
          "name": "cell78 ~ gene67"
        },
        {
          "x": 77,
          "y": 67,
          "value": 11,
          "name": "cell78 ~ gene68"
        },
        {
          "x": 77,
          "y": 68,
          "value": 22,
          "name": "cell78 ~ gene69"
        },
        {
          "x": 77,
          "y": 69,
          "value": 0,
          "name": "cell78 ~ gene70"
        },
        {
          "x": 77,
          "y": 70,
          "value": 3,
          "name": "cell78 ~ gene71"
        },
        {
          "x": 77,
          "y": 71,
          "value": 0,
          "name": "cell78 ~ gene72"
        },
        {
          "x": 77,
          "y": 72,
          "value": 0,
          "name": "cell78 ~ gene73"
        },
        {
          "x": 77,
          "y": 73,
          "value": 22,
          "name": "cell78 ~ gene74"
        },
        {
          "x": 77,
          "y": 74,
          "value": 0,
          "name": "cell78 ~ gene75"
        },
        {
          "x": 77,
          "y": 75,
          "value": 0,
          "name": "cell78 ~ gene76"
        },
        {
          "x": 77,
          "y": 76,
          "value": 0,
          "name": "cell78 ~ gene77"
        },
        {
          "x": 77,
          "y": 77,
          "value": 0,
          "name": "cell78 ~ gene78"
        },
        {
          "x": 77,
          "y": 78,
          "value": 2,
          "name": "cell78 ~ gene79"
        },
        {
          "x": 77,
          "y": 79,
          "value": 0,
          "name": "cell78 ~ gene80"
        },
        {
          "x": 77,
          "y": 80,
          "value": 4,
          "name": "cell78 ~ gene81"
        },
        {
          "x": 77,
          "y": 81,
          "value": 0,
          "name": "cell78 ~ gene82"
        },
        {
          "x": 77,
          "y": 82,
          "value": 10,
          "name": "cell78 ~ gene83"
        },
        {
          "x": 77,
          "y": 83,
          "value": 1,
          "name": "cell78 ~ gene84"
        },
        {
          "x": 77,
          "y": 84,
          "value": 9,
          "name": "cell78 ~ gene85"
        },
        {
          "x": 77,
          "y": 85,
          "value": 3,
          "name": "cell78 ~ gene86"
        },
        {
          "x": 77,
          "y": 86,
          "value": 0,
          "name": "cell78 ~ gene87"
        },
        {
          "x": 77,
          "y": 87,
          "value": 17,
          "name": "cell78 ~ gene88"
        },
        {
          "x": 77,
          "y": 88,
          "value": 18,
          "name": "cell78 ~ gene89"
        },
        {
          "x": 77,
          "y": 89,
          "value": 0,
          "name": "cell78 ~ gene90"
        },
        {
          "x": 77,
          "y": 90,
          "value": 2,
          "name": "cell78 ~ gene91"
        },
        {
          "x": 77,
          "y": 91,
          "value": 2,
          "name": "cell78 ~ gene92"
        },
        {
          "x": 77,
          "y": 92,
          "value": 6,
          "name": "cell78 ~ gene93"
        },
        {
          "x": 77,
          "y": 93,
          "value": 23,
          "name": "cell78 ~ gene94"
        },
        {
          "x": 77,
          "y": 94,
          "value": 12,
          "name": "cell78 ~ gene95"
        },
        {
          "x": 77,
          "y": 95,
          "value": 0,
          "name": "cell78 ~ gene96"
        },
        {
          "x": 77,
          "y": 96,
          "value": 6,
          "name": "cell78 ~ gene97"
        },
        {
          "x": 77,
          "y": 97,
          "value": 0,
          "name": "cell78 ~ gene98"
        },
        {
          "x": 77,
          "y": 98,
          "value": 9,
          "name": "cell78 ~ gene99"
        },
        {
          "x": 77,
          "y": 99,
          "value": 4,
          "name": "cell78 ~ gene100"
        },
        {
          "x": 78,
          "y": 0,
          "value": 0,
          "name": "cell79 ~ gene1"
        },
        {
          "x": 78,
          "y": 1,
          "value": 11,
          "name": "cell79 ~ gene2"
        },
        {
          "x": 78,
          "y": 2,
          "value": 0,
          "name": "cell79 ~ gene3"
        },
        {
          "x": 78,
          "y": 3,
          "value": 0,
          "name": "cell79 ~ gene4"
        },
        {
          "x": 78,
          "y": 4,
          "value": 0,
          "name": "cell79 ~ gene5"
        },
        {
          "x": 78,
          "y": 5,
          "value": 0,
          "name": "cell79 ~ gene6"
        },
        {
          "x": 78,
          "y": 6,
          "value": 0,
          "name": "cell79 ~ gene7"
        },
        {
          "x": 78,
          "y": 7,
          "value": 11,
          "name": "cell79 ~ gene8"
        },
        {
          "x": 78,
          "y": 8,
          "value": 0,
          "name": "cell79 ~ gene9"
        },
        {
          "x": 78,
          "y": 9,
          "value": 0,
          "name": "cell79 ~ gene10"
        },
        {
          "x": 78,
          "y": 10,
          "value": 8,
          "name": "cell79 ~ gene11"
        },
        {
          "x": 78,
          "y": 11,
          "value": 12,
          "name": "cell79 ~ gene12"
        },
        {
          "x": 78,
          "y": 12,
          "value": 0,
          "name": "cell79 ~ gene13"
        },
        {
          "x": 78,
          "y": 13,
          "value": 0,
          "name": "cell79 ~ gene14"
        },
        {
          "x": 78,
          "y": 14,
          "value": 0,
          "name": "cell79 ~ gene15"
        },
        {
          "x": 78,
          "y": 15,
          "value": 11,
          "name": "cell79 ~ gene16"
        },
        {
          "x": 78,
          "y": 16,
          "value": 6,
          "name": "cell79 ~ gene17"
        },
        {
          "x": 78,
          "y": 17,
          "value": 0,
          "name": "cell79 ~ gene18"
        },
        {
          "x": 78,
          "y": 18,
          "value": 0,
          "name": "cell79 ~ gene19"
        },
        {
          "x": 78,
          "y": 19,
          "value": 10,
          "name": "cell79 ~ gene20"
        },
        {
          "x": 78,
          "y": 20,
          "value": 7,
          "name": "cell79 ~ gene21"
        },
        {
          "x": 78,
          "y": 21,
          "value": 1,
          "name": "cell79 ~ gene22"
        },
        {
          "x": 78,
          "y": 22,
          "value": 8,
          "name": "cell79 ~ gene23"
        },
        {
          "x": 78,
          "y": 23,
          "value": 6,
          "name": "cell79 ~ gene24"
        },
        {
          "x": 78,
          "y": 24,
          "value": 11,
          "name": "cell79 ~ gene25"
        },
        {
          "x": 78,
          "y": 25,
          "value": 0,
          "name": "cell79 ~ gene26"
        },
        {
          "x": 78,
          "y": 26,
          "value": 0,
          "name": "cell79 ~ gene27"
        },
        {
          "x": 78,
          "y": 27,
          "value": 9,
          "name": "cell79 ~ gene28"
        },
        {
          "x": 78,
          "y": 28,
          "value": 2,
          "name": "cell79 ~ gene29"
        },
        {
          "x": 78,
          "y": 29,
          "value": 0,
          "name": "cell79 ~ gene30"
        },
        {
          "x": 78,
          "y": 30,
          "value": 4,
          "name": "cell79 ~ gene31"
        },
        {
          "x": 78,
          "y": 31,
          "value": 10,
          "name": "cell79 ~ gene32"
        },
        {
          "x": 78,
          "y": 32,
          "value": 0,
          "name": "cell79 ~ gene33"
        },
        {
          "x": 78,
          "y": 33,
          "value": 4,
          "name": "cell79 ~ gene34"
        },
        {
          "x": 78,
          "y": 34,
          "value": 9,
          "name": "cell79 ~ gene35"
        },
        {
          "x": 78,
          "y": 35,
          "value": 10,
          "name": "cell79 ~ gene36"
        },
        {
          "x": 78,
          "y": 36,
          "value": 5,
          "name": "cell79 ~ gene37"
        },
        {
          "x": 78,
          "y": 37,
          "value": 9,
          "name": "cell79 ~ gene38"
        },
        {
          "x": 78,
          "y": 38,
          "value": 0,
          "name": "cell79 ~ gene39"
        },
        {
          "x": 78,
          "y": 39,
          "value": 6,
          "name": "cell79 ~ gene40"
        },
        {
          "x": 78,
          "y": 40,
          "value": 3,
          "name": "cell79 ~ gene41"
        },
        {
          "x": 78,
          "y": 41,
          "value": 26,
          "name": "cell79 ~ gene42"
        },
        {
          "x": 78,
          "y": 42,
          "value": 0,
          "name": "cell79 ~ gene43"
        },
        {
          "x": 78,
          "y": 43,
          "value": 3,
          "name": "cell79 ~ gene44"
        },
        {
          "x": 78,
          "y": 44,
          "value": 2,
          "name": "cell79 ~ gene45"
        },
        {
          "x": 78,
          "y": 45,
          "value": 10,
          "name": "cell79 ~ gene46"
        },
        {
          "x": 78,
          "y": 46,
          "value": 0,
          "name": "cell79 ~ gene47"
        },
        {
          "x": 78,
          "y": 47,
          "value": 3,
          "name": "cell79 ~ gene48"
        },
        {
          "x": 78,
          "y": 48,
          "value": 5,
          "name": "cell79 ~ gene49"
        },
        {
          "x": 78,
          "y": 49,
          "value": 16,
          "name": "cell79 ~ gene50"
        },
        {
          "x": 78,
          "y": 50,
          "value": 15,
          "name": "cell79 ~ gene51"
        },
        {
          "x": 78,
          "y": 51,
          "value": 16,
          "name": "cell79 ~ gene52"
        },
        {
          "x": 78,
          "y": 52,
          "value": 23,
          "name": "cell79 ~ gene53"
        },
        {
          "x": 78,
          "y": 53,
          "value": 20,
          "name": "cell79 ~ gene54"
        },
        {
          "x": 78,
          "y": 54,
          "value": 22,
          "name": "cell79 ~ gene55"
        },
        {
          "x": 78,
          "y": 55,
          "value": 20,
          "name": "cell79 ~ gene56"
        },
        {
          "x": 78,
          "y": 56,
          "value": 3,
          "name": "cell79 ~ gene57"
        },
        {
          "x": 78,
          "y": 57,
          "value": 0,
          "name": "cell79 ~ gene58"
        },
        {
          "x": 78,
          "y": 58,
          "value": 0,
          "name": "cell79 ~ gene59"
        },
        {
          "x": 78,
          "y": 59,
          "value": 9,
          "name": "cell79 ~ gene60"
        },
        {
          "x": 78,
          "y": 60,
          "value": 0,
          "name": "cell79 ~ gene61"
        },
        {
          "x": 78,
          "y": 61,
          "value": 5,
          "name": "cell79 ~ gene62"
        },
        {
          "x": 78,
          "y": 62,
          "value": 4,
          "name": "cell79 ~ gene63"
        },
        {
          "x": 78,
          "y": 63,
          "value": 0,
          "name": "cell79 ~ gene64"
        },
        {
          "x": 78,
          "y": 64,
          "value": 0,
          "name": "cell79 ~ gene65"
        },
        {
          "x": 78,
          "y": 65,
          "value": 2,
          "name": "cell79 ~ gene66"
        },
        {
          "x": 78,
          "y": 66,
          "value": 5,
          "name": "cell79 ~ gene67"
        },
        {
          "x": 78,
          "y": 67,
          "value": 7,
          "name": "cell79 ~ gene68"
        },
        {
          "x": 78,
          "y": 68,
          "value": 0,
          "name": "cell79 ~ gene69"
        },
        {
          "x": 78,
          "y": 69,
          "value": 0,
          "name": "cell79 ~ gene70"
        },
        {
          "x": 78,
          "y": 70,
          "value": 0,
          "name": "cell79 ~ gene71"
        },
        {
          "x": 78,
          "y": 71,
          "value": 0,
          "name": "cell79 ~ gene72"
        },
        {
          "x": 78,
          "y": 72,
          "value": 0,
          "name": "cell79 ~ gene73"
        },
        {
          "x": 78,
          "y": 73,
          "value": 7,
          "name": "cell79 ~ gene74"
        },
        {
          "x": 78,
          "y": 74,
          "value": 9,
          "name": "cell79 ~ gene75"
        },
        {
          "x": 78,
          "y": 75,
          "value": 8,
          "name": "cell79 ~ gene76"
        },
        {
          "x": 78,
          "y": 76,
          "value": 26,
          "name": "cell79 ~ gene77"
        },
        {
          "x": 78,
          "y": 77,
          "value": 4,
          "name": "cell79 ~ gene78"
        },
        {
          "x": 78,
          "y": 78,
          "value": 8,
          "name": "cell79 ~ gene79"
        },
        {
          "x": 78,
          "y": 79,
          "value": 0,
          "name": "cell79 ~ gene80"
        },
        {
          "x": 78,
          "y": 80,
          "value": 8,
          "name": "cell79 ~ gene81"
        },
        {
          "x": 78,
          "y": 81,
          "value": 3,
          "name": "cell79 ~ gene82"
        },
        {
          "x": 78,
          "y": 82,
          "value": 0,
          "name": "cell79 ~ gene83"
        },
        {
          "x": 78,
          "y": 83,
          "value": 0,
          "name": "cell79 ~ gene84"
        },
        {
          "x": 78,
          "y": 84,
          "value": 5,
          "name": "cell79 ~ gene85"
        },
        {
          "x": 78,
          "y": 85,
          "value": 0,
          "name": "cell79 ~ gene86"
        },
        {
          "x": 78,
          "y": 86,
          "value": 16,
          "name": "cell79 ~ gene87"
        },
        {
          "x": 78,
          "y": 87,
          "value": 19,
          "name": "cell79 ~ gene88"
        },
        {
          "x": 78,
          "y": 88,
          "value": 20,
          "name": "cell79 ~ gene89"
        },
        {
          "x": 78,
          "y": 89,
          "value": 11,
          "name": "cell79 ~ gene90"
        },
        {
          "x": 78,
          "y": 90,
          "value": 5,
          "name": "cell79 ~ gene91"
        },
        {
          "x": 78,
          "y": 91,
          "value": 21,
          "name": "cell79 ~ gene92"
        },
        {
          "x": 78,
          "y": 92,
          "value": 14,
          "name": "cell79 ~ gene93"
        },
        {
          "x": 78,
          "y": 93,
          "value": 0,
          "name": "cell79 ~ gene94"
        },
        {
          "x": 78,
          "y": 94,
          "value": 3,
          "name": "cell79 ~ gene95"
        },
        {
          "x": 78,
          "y": 95,
          "value": 1,
          "name": "cell79 ~ gene96"
        },
        {
          "x": 78,
          "y": 96,
          "value": 4,
          "name": "cell79 ~ gene97"
        },
        {
          "x": 78,
          "y": 97,
          "value": 1,
          "name": "cell79 ~ gene98"
        },
        {
          "x": 78,
          "y": 98,
          "value": 0,
          "name": "cell79 ~ gene99"
        },
        {
          "x": 78,
          "y": 99,
          "value": 7,
          "name": "cell79 ~ gene100"
        },
        {
          "x": 79,
          "y": 0,
          "value": 0,
          "name": "cell80 ~ gene1"
        },
        {
          "x": 79,
          "y": 1,
          "value": 2,
          "name": "cell80 ~ gene2"
        },
        {
          "x": 79,
          "y": 2,
          "value": 4,
          "name": "cell80 ~ gene3"
        },
        {
          "x": 79,
          "y": 3,
          "value": 0,
          "name": "cell80 ~ gene4"
        },
        {
          "x": 79,
          "y": 4,
          "value": 7,
          "name": "cell80 ~ gene5"
        },
        {
          "x": 79,
          "y": 5,
          "value": 14,
          "name": "cell80 ~ gene6"
        },
        {
          "x": 79,
          "y": 6,
          "value": 0,
          "name": "cell80 ~ gene7"
        },
        {
          "x": 79,
          "y": 7,
          "value": 0,
          "name": "cell80 ~ gene8"
        },
        {
          "x": 79,
          "y": 8,
          "value": 0,
          "name": "cell80 ~ gene9"
        },
        {
          "x": 79,
          "y": 9,
          "value": 19,
          "name": "cell80 ~ gene10"
        },
        {
          "x": 79,
          "y": 10,
          "value": 0,
          "name": "cell80 ~ gene11"
        },
        {
          "x": 79,
          "y": 11,
          "value": 0,
          "name": "cell80 ~ gene12"
        },
        {
          "x": 79,
          "y": 12,
          "value": 0,
          "name": "cell80 ~ gene13"
        },
        {
          "x": 79,
          "y": 13,
          "value": 0,
          "name": "cell80 ~ gene14"
        },
        {
          "x": 79,
          "y": 14,
          "value": 12,
          "name": "cell80 ~ gene15"
        },
        {
          "x": 79,
          "y": 15,
          "value": 0,
          "name": "cell80 ~ gene16"
        },
        {
          "x": 79,
          "y": 16,
          "value": 7,
          "name": "cell80 ~ gene17"
        },
        {
          "x": 79,
          "y": 17,
          "value": 0,
          "name": "cell80 ~ gene18"
        },
        {
          "x": 79,
          "y": 18,
          "value": 6,
          "name": "cell80 ~ gene19"
        },
        {
          "x": 79,
          "y": 19,
          "value": 5,
          "name": "cell80 ~ gene20"
        },
        {
          "x": 79,
          "y": 20,
          "value": 7,
          "name": "cell80 ~ gene21"
        },
        {
          "x": 79,
          "y": 21,
          "value": 0,
          "name": "cell80 ~ gene22"
        },
        {
          "x": 79,
          "y": 22,
          "value": 17,
          "name": "cell80 ~ gene23"
        },
        {
          "x": 79,
          "y": 23,
          "value": 1,
          "name": "cell80 ~ gene24"
        },
        {
          "x": 79,
          "y": 24,
          "value": 0,
          "name": "cell80 ~ gene25"
        },
        {
          "x": 79,
          "y": 25,
          "value": 0,
          "name": "cell80 ~ gene26"
        },
        {
          "x": 79,
          "y": 26,
          "value": 0,
          "name": "cell80 ~ gene27"
        },
        {
          "x": 79,
          "y": 27,
          "value": 0,
          "name": "cell80 ~ gene28"
        },
        {
          "x": 79,
          "y": 28,
          "value": 5,
          "name": "cell80 ~ gene29"
        },
        {
          "x": 79,
          "y": 29,
          "value": 0,
          "name": "cell80 ~ gene30"
        },
        {
          "x": 79,
          "y": 30,
          "value": 8,
          "name": "cell80 ~ gene31"
        },
        {
          "x": 79,
          "y": 31,
          "value": 0,
          "name": "cell80 ~ gene32"
        },
        {
          "x": 79,
          "y": 32,
          "value": 1,
          "name": "cell80 ~ gene33"
        },
        {
          "x": 79,
          "y": 33,
          "value": 0,
          "name": "cell80 ~ gene34"
        },
        {
          "x": 79,
          "y": 34,
          "value": 3,
          "name": "cell80 ~ gene35"
        },
        {
          "x": 79,
          "y": 35,
          "value": 3,
          "name": "cell80 ~ gene36"
        },
        {
          "x": 79,
          "y": 36,
          "value": 0,
          "name": "cell80 ~ gene37"
        },
        {
          "x": 79,
          "y": 37,
          "value": 0,
          "name": "cell80 ~ gene38"
        },
        {
          "x": 79,
          "y": 38,
          "value": 0,
          "name": "cell80 ~ gene39"
        },
        {
          "x": 79,
          "y": 39,
          "value": 0,
          "name": "cell80 ~ gene40"
        },
        {
          "x": 79,
          "y": 40,
          "value": 16,
          "name": "cell80 ~ gene41"
        },
        {
          "x": 79,
          "y": 41,
          "value": 25,
          "name": "cell80 ~ gene42"
        },
        {
          "x": 79,
          "y": 42,
          "value": 0,
          "name": "cell80 ~ gene43"
        },
        {
          "x": 79,
          "y": 43,
          "value": 4,
          "name": "cell80 ~ gene44"
        },
        {
          "x": 79,
          "y": 44,
          "value": 12,
          "name": "cell80 ~ gene45"
        },
        {
          "x": 79,
          "y": 45,
          "value": 24,
          "name": "cell80 ~ gene46"
        },
        {
          "x": 79,
          "y": 46,
          "value": 10,
          "name": "cell80 ~ gene47"
        },
        {
          "x": 79,
          "y": 47,
          "value": 0,
          "name": "cell80 ~ gene48"
        },
        {
          "x": 79,
          "y": 48,
          "value": 0,
          "name": "cell80 ~ gene49"
        },
        {
          "x": 79,
          "y": 49,
          "value": 22,
          "name": "cell80 ~ gene50"
        },
        {
          "x": 79,
          "y": 50,
          "value": 7,
          "name": "cell80 ~ gene51"
        },
        {
          "x": 79,
          "y": 51,
          "value": 21,
          "name": "cell80 ~ gene52"
        },
        {
          "x": 79,
          "y": 52,
          "value": 6,
          "name": "cell80 ~ gene53"
        },
        {
          "x": 79,
          "y": 53,
          "value": 7,
          "name": "cell80 ~ gene54"
        },
        {
          "x": 79,
          "y": 54,
          "value": 16,
          "name": "cell80 ~ gene55"
        },
        {
          "x": 79,
          "y": 55,
          "value": 32,
          "name": "cell80 ~ gene56"
        },
        {
          "x": 79,
          "y": 56,
          "value": 9,
          "name": "cell80 ~ gene57"
        },
        {
          "x": 79,
          "y": 57,
          "value": 26,
          "name": "cell80 ~ gene58"
        },
        {
          "x": 79,
          "y": 58,
          "value": 8,
          "name": "cell80 ~ gene59"
        },
        {
          "x": 79,
          "y": 59,
          "value": 0,
          "name": "cell80 ~ gene60"
        },
        {
          "x": 79,
          "y": 60,
          "value": 0,
          "name": "cell80 ~ gene61"
        },
        {
          "x": 79,
          "y": 61,
          "value": 14,
          "name": "cell80 ~ gene62"
        },
        {
          "x": 79,
          "y": 62,
          "value": 0,
          "name": "cell80 ~ gene63"
        },
        {
          "x": 79,
          "y": 63,
          "value": 7,
          "name": "cell80 ~ gene64"
        },
        {
          "x": 79,
          "y": 64,
          "value": 0,
          "name": "cell80 ~ gene65"
        },
        {
          "x": 79,
          "y": 65,
          "value": 8,
          "name": "cell80 ~ gene66"
        },
        {
          "x": 79,
          "y": 66,
          "value": 3,
          "name": "cell80 ~ gene67"
        },
        {
          "x": 79,
          "y": 67,
          "value": 6,
          "name": "cell80 ~ gene68"
        },
        {
          "x": 79,
          "y": 68,
          "value": 0,
          "name": "cell80 ~ gene69"
        },
        {
          "x": 79,
          "y": 69,
          "value": 0,
          "name": "cell80 ~ gene70"
        },
        {
          "x": 79,
          "y": 70,
          "value": 7,
          "name": "cell80 ~ gene71"
        },
        {
          "x": 79,
          "y": 71,
          "value": 6,
          "name": "cell80 ~ gene72"
        },
        {
          "x": 79,
          "y": 72,
          "value": 0,
          "name": "cell80 ~ gene73"
        },
        {
          "x": 79,
          "y": 73,
          "value": 0,
          "name": "cell80 ~ gene74"
        },
        {
          "x": 79,
          "y": 74,
          "value": 7,
          "name": "cell80 ~ gene75"
        },
        {
          "x": 79,
          "y": 75,
          "value": 6,
          "name": "cell80 ~ gene76"
        },
        {
          "x": 79,
          "y": 76,
          "value": 0,
          "name": "cell80 ~ gene77"
        },
        {
          "x": 79,
          "y": 77,
          "value": 7,
          "name": "cell80 ~ gene78"
        },
        {
          "x": 79,
          "y": 78,
          "value": 0,
          "name": "cell80 ~ gene79"
        },
        {
          "x": 79,
          "y": 79,
          "value": 5,
          "name": "cell80 ~ gene80"
        },
        {
          "x": 79,
          "y": 80,
          "value": 0,
          "name": "cell80 ~ gene81"
        },
        {
          "x": 79,
          "y": 81,
          "value": 7,
          "name": "cell80 ~ gene82"
        },
        {
          "x": 79,
          "y": 82,
          "value": 0,
          "name": "cell80 ~ gene83"
        },
        {
          "x": 79,
          "y": 83,
          "value": 0,
          "name": "cell80 ~ gene84"
        },
        {
          "x": 79,
          "y": 84,
          "value": 7,
          "name": "cell80 ~ gene85"
        },
        {
          "x": 79,
          "y": 85,
          "value": 0,
          "name": "cell80 ~ gene86"
        },
        {
          "x": 79,
          "y": 86,
          "value": 0,
          "name": "cell80 ~ gene87"
        },
        {
          "x": 79,
          "y": 87,
          "value": 0,
          "name": "cell80 ~ gene88"
        },
        {
          "x": 79,
          "y": 88,
          "value": 13,
          "name": "cell80 ~ gene89"
        },
        {
          "x": 79,
          "y": 89,
          "value": 0,
          "name": "cell80 ~ gene90"
        },
        {
          "x": 79,
          "y": 90,
          "value": 0,
          "name": "cell80 ~ gene91"
        },
        {
          "x": 79,
          "y": 91,
          "value": 0,
          "name": "cell80 ~ gene92"
        },
        {
          "x": 79,
          "y": 92,
          "value": 0,
          "name": "cell80 ~ gene93"
        },
        {
          "x": 79,
          "y": 93,
          "value": 0,
          "name": "cell80 ~ gene94"
        },
        {
          "x": 79,
          "y": 94,
          "value": 0,
          "name": "cell80 ~ gene95"
        },
        {
          "x": 79,
          "y": 95,
          "value": 0,
          "name": "cell80 ~ gene96"
        },
        {
          "x": 79,
          "y": 96,
          "value": 22,
          "name": "cell80 ~ gene97"
        },
        {
          "x": 79,
          "y": 97,
          "value": 9,
          "name": "cell80 ~ gene98"
        },
        {
          "x": 79,
          "y": 98,
          "value": 0,
          "name": "cell80 ~ gene99"
        },
        {
          "x": 79,
          "y": 99,
          "value": 0,
          "name": "cell80 ~ gene100"
        },
        {
          "x": 80,
          "y": 0,
          "value": 2,
          "name": "cell81 ~ gene1"
        },
        {
          "x": 80,
          "y": 1,
          "value": 19,
          "name": "cell81 ~ gene2"
        },
        {
          "x": 80,
          "y": 2,
          "value": 0,
          "name": "cell81 ~ gene3"
        },
        {
          "x": 80,
          "y": 3,
          "value": 2,
          "name": "cell81 ~ gene4"
        },
        {
          "x": 80,
          "y": 4,
          "value": 1,
          "name": "cell81 ~ gene5"
        },
        {
          "x": 80,
          "y": 5,
          "value": 16,
          "name": "cell81 ~ gene6"
        },
        {
          "x": 80,
          "y": 6,
          "value": 0,
          "name": "cell81 ~ gene7"
        },
        {
          "x": 80,
          "y": 7,
          "value": 0,
          "name": "cell81 ~ gene8"
        },
        {
          "x": 80,
          "y": 8,
          "value": 0,
          "name": "cell81 ~ gene9"
        },
        {
          "x": 80,
          "y": 9,
          "value": 0,
          "name": "cell81 ~ gene10"
        },
        {
          "x": 80,
          "y": 10,
          "value": 0,
          "name": "cell81 ~ gene11"
        },
        {
          "x": 80,
          "y": 11,
          "value": 5,
          "name": "cell81 ~ gene12"
        },
        {
          "x": 80,
          "y": 12,
          "value": 0,
          "name": "cell81 ~ gene13"
        },
        {
          "x": 80,
          "y": 13,
          "value": 7,
          "name": "cell81 ~ gene14"
        },
        {
          "x": 80,
          "y": 14,
          "value": 1,
          "name": "cell81 ~ gene15"
        },
        {
          "x": 80,
          "y": 15,
          "value": 0,
          "name": "cell81 ~ gene16"
        },
        {
          "x": 80,
          "y": 16,
          "value": 4,
          "name": "cell81 ~ gene17"
        },
        {
          "x": 80,
          "y": 17,
          "value": 0,
          "name": "cell81 ~ gene18"
        },
        {
          "x": 80,
          "y": 18,
          "value": 1,
          "name": "cell81 ~ gene19"
        },
        {
          "x": 80,
          "y": 19,
          "value": 8,
          "name": "cell81 ~ gene20"
        },
        {
          "x": 80,
          "y": 20,
          "value": 0,
          "name": "cell81 ~ gene21"
        },
        {
          "x": 80,
          "y": 21,
          "value": 0,
          "name": "cell81 ~ gene22"
        },
        {
          "x": 80,
          "y": 22,
          "value": 1,
          "name": "cell81 ~ gene23"
        },
        {
          "x": 80,
          "y": 23,
          "value": 0,
          "name": "cell81 ~ gene24"
        },
        {
          "x": 80,
          "y": 24,
          "value": 3,
          "name": "cell81 ~ gene25"
        },
        {
          "x": 80,
          "y": 25,
          "value": 0,
          "name": "cell81 ~ gene26"
        },
        {
          "x": 80,
          "y": 26,
          "value": 14,
          "name": "cell81 ~ gene27"
        },
        {
          "x": 80,
          "y": 27,
          "value": 0,
          "name": "cell81 ~ gene28"
        },
        {
          "x": 80,
          "y": 28,
          "value": 0,
          "name": "cell81 ~ gene29"
        },
        {
          "x": 80,
          "y": 29,
          "value": 0,
          "name": "cell81 ~ gene30"
        },
        {
          "x": 80,
          "y": 30,
          "value": 0,
          "name": "cell81 ~ gene31"
        },
        {
          "x": 80,
          "y": 31,
          "value": 0,
          "name": "cell81 ~ gene32"
        },
        {
          "x": 80,
          "y": 32,
          "value": 2,
          "name": "cell81 ~ gene33"
        },
        {
          "x": 80,
          "y": 33,
          "value": 7,
          "name": "cell81 ~ gene34"
        },
        {
          "x": 80,
          "y": 34,
          "value": 4,
          "name": "cell81 ~ gene35"
        },
        {
          "x": 80,
          "y": 35,
          "value": 17,
          "name": "cell81 ~ gene36"
        },
        {
          "x": 80,
          "y": 36,
          "value": 0,
          "name": "cell81 ~ gene37"
        },
        {
          "x": 80,
          "y": 37,
          "value": 11,
          "name": "cell81 ~ gene38"
        },
        {
          "x": 80,
          "y": 38,
          "value": 2,
          "name": "cell81 ~ gene39"
        },
        {
          "x": 80,
          "y": 39,
          "value": 0,
          "name": "cell81 ~ gene40"
        },
        {
          "x": 80,
          "y": 40,
          "value": 2,
          "name": "cell81 ~ gene41"
        },
        {
          "x": 80,
          "y": 41,
          "value": 33,
          "name": "cell81 ~ gene42"
        },
        {
          "x": 80,
          "y": 42,
          "value": 0,
          "name": "cell81 ~ gene43"
        },
        {
          "x": 80,
          "y": 43,
          "value": 9,
          "name": "cell81 ~ gene44"
        },
        {
          "x": 80,
          "y": 44,
          "value": 12,
          "name": "cell81 ~ gene45"
        },
        {
          "x": 80,
          "y": 45,
          "value": 11,
          "name": "cell81 ~ gene46"
        },
        {
          "x": 80,
          "y": 46,
          "value": 6,
          "name": "cell81 ~ gene47"
        },
        {
          "x": 80,
          "y": 47,
          "value": 13,
          "name": "cell81 ~ gene48"
        },
        {
          "x": 80,
          "y": 48,
          "value": 0,
          "name": "cell81 ~ gene49"
        },
        {
          "x": 80,
          "y": 49,
          "value": 11,
          "name": "cell81 ~ gene50"
        },
        {
          "x": 80,
          "y": 50,
          "value": 0,
          "name": "cell81 ~ gene51"
        },
        {
          "x": 80,
          "y": 51,
          "value": 23,
          "name": "cell81 ~ gene52"
        },
        {
          "x": 80,
          "y": 52,
          "value": 8,
          "name": "cell81 ~ gene53"
        },
        {
          "x": 80,
          "y": 53,
          "value": 12,
          "name": "cell81 ~ gene54"
        },
        {
          "x": 80,
          "y": 54,
          "value": 27,
          "name": "cell81 ~ gene55"
        },
        {
          "x": 80,
          "y": 55,
          "value": 29,
          "name": "cell81 ~ gene56"
        },
        {
          "x": 80,
          "y": 56,
          "value": 19,
          "name": "cell81 ~ gene57"
        },
        {
          "x": 80,
          "y": 57,
          "value": 0,
          "name": "cell81 ~ gene58"
        },
        {
          "x": 80,
          "y": 58,
          "value": 0,
          "name": "cell81 ~ gene59"
        },
        {
          "x": 80,
          "y": 59,
          "value": 0,
          "name": "cell81 ~ gene60"
        },
        {
          "x": 80,
          "y": 60,
          "value": 0,
          "name": "cell81 ~ gene61"
        },
        {
          "x": 80,
          "y": 61,
          "value": 1,
          "name": "cell81 ~ gene62"
        },
        {
          "x": 80,
          "y": 62,
          "value": 0,
          "name": "cell81 ~ gene63"
        },
        {
          "x": 80,
          "y": 63,
          "value": 3,
          "name": "cell81 ~ gene64"
        },
        {
          "x": 80,
          "y": 64,
          "value": 2,
          "name": "cell81 ~ gene65"
        },
        {
          "x": 80,
          "y": 65,
          "value": 0,
          "name": "cell81 ~ gene66"
        },
        {
          "x": 80,
          "y": 66,
          "value": 0,
          "name": "cell81 ~ gene67"
        },
        {
          "x": 80,
          "y": 67,
          "value": 5,
          "name": "cell81 ~ gene68"
        },
        {
          "x": 80,
          "y": 68,
          "value": 0,
          "name": "cell81 ~ gene69"
        },
        {
          "x": 80,
          "y": 69,
          "value": 0,
          "name": "cell81 ~ gene70"
        },
        {
          "x": 80,
          "y": 70,
          "value": 0,
          "name": "cell81 ~ gene71"
        },
        {
          "x": 80,
          "y": 71,
          "value": 0,
          "name": "cell81 ~ gene72"
        },
        {
          "x": 80,
          "y": 72,
          "value": 0,
          "name": "cell81 ~ gene73"
        },
        {
          "x": 80,
          "y": 73,
          "value": 2,
          "name": "cell81 ~ gene74"
        },
        {
          "x": 80,
          "y": 74,
          "value": 13,
          "name": "cell81 ~ gene75"
        },
        {
          "x": 80,
          "y": 75,
          "value": 0,
          "name": "cell81 ~ gene76"
        },
        {
          "x": 80,
          "y": 76,
          "value": 16,
          "name": "cell81 ~ gene77"
        },
        {
          "x": 80,
          "y": 77,
          "value": 0,
          "name": "cell81 ~ gene78"
        },
        {
          "x": 80,
          "y": 78,
          "value": 0,
          "name": "cell81 ~ gene79"
        },
        {
          "x": 80,
          "y": 79,
          "value": 0,
          "name": "cell81 ~ gene80"
        },
        {
          "x": 80,
          "y": 80,
          "value": 0,
          "name": "cell81 ~ gene81"
        },
        {
          "x": 80,
          "y": 81,
          "value": 0,
          "name": "cell81 ~ gene82"
        },
        {
          "x": 80,
          "y": 82,
          "value": 2,
          "name": "cell81 ~ gene83"
        },
        {
          "x": 80,
          "y": 83,
          "value": 0,
          "name": "cell81 ~ gene84"
        },
        {
          "x": 80,
          "y": 84,
          "value": 6,
          "name": "cell81 ~ gene85"
        },
        {
          "x": 80,
          "y": 85,
          "value": 6,
          "name": "cell81 ~ gene86"
        },
        {
          "x": 80,
          "y": 86,
          "value": 0,
          "name": "cell81 ~ gene87"
        },
        {
          "x": 80,
          "y": 87,
          "value": 0,
          "name": "cell81 ~ gene88"
        },
        {
          "x": 80,
          "y": 88,
          "value": 0,
          "name": "cell81 ~ gene89"
        },
        {
          "x": 80,
          "y": 89,
          "value": 1,
          "name": "cell81 ~ gene90"
        },
        {
          "x": 80,
          "y": 90,
          "value": 1,
          "name": "cell81 ~ gene91"
        },
        {
          "x": 80,
          "y": 91,
          "value": 12,
          "name": "cell81 ~ gene92"
        },
        {
          "x": 80,
          "y": 92,
          "value": 12,
          "name": "cell81 ~ gene93"
        },
        {
          "x": 80,
          "y": 93,
          "value": 0,
          "name": "cell81 ~ gene94"
        },
        {
          "x": 80,
          "y": 94,
          "value": 0,
          "name": "cell81 ~ gene95"
        },
        {
          "x": 80,
          "y": 95,
          "value": 0,
          "name": "cell81 ~ gene96"
        },
        {
          "x": 80,
          "y": 96,
          "value": 0,
          "name": "cell81 ~ gene97"
        },
        {
          "x": 80,
          "y": 97,
          "value": 0,
          "name": "cell81 ~ gene98"
        },
        {
          "x": 80,
          "y": 98,
          "value": 0,
          "name": "cell81 ~ gene99"
        },
        {
          "x": 80,
          "y": 99,
          "value": 17,
          "name": "cell81 ~ gene100"
        },
        {
          "x": 81,
          "y": 0,
          "value": 0,
          "name": "cell82 ~ gene1"
        },
        {
          "x": 81,
          "y": 1,
          "value": 0,
          "name": "cell82 ~ gene2"
        },
        {
          "x": 81,
          "y": 2,
          "value": 16,
          "name": "cell82 ~ gene3"
        },
        {
          "x": 81,
          "y": 3,
          "value": 17,
          "name": "cell82 ~ gene4"
        },
        {
          "x": 81,
          "y": 4,
          "value": 0,
          "name": "cell82 ~ gene5"
        },
        {
          "x": 81,
          "y": 5,
          "value": 9,
          "name": "cell82 ~ gene6"
        },
        {
          "x": 81,
          "y": 6,
          "value": 2,
          "name": "cell82 ~ gene7"
        },
        {
          "x": 81,
          "y": 7,
          "value": 8,
          "name": "cell82 ~ gene8"
        },
        {
          "x": 81,
          "y": 8,
          "value": 0,
          "name": "cell82 ~ gene9"
        },
        {
          "x": 81,
          "y": 9,
          "value": 10,
          "name": "cell82 ~ gene10"
        },
        {
          "x": 81,
          "y": 10,
          "value": 0,
          "name": "cell82 ~ gene11"
        },
        {
          "x": 81,
          "y": 11,
          "value": 0,
          "name": "cell82 ~ gene12"
        },
        {
          "x": 81,
          "y": 12,
          "value": 0,
          "name": "cell82 ~ gene13"
        },
        {
          "x": 81,
          "y": 13,
          "value": 11,
          "name": "cell82 ~ gene14"
        },
        {
          "x": 81,
          "y": 14,
          "value": 0,
          "name": "cell82 ~ gene15"
        },
        {
          "x": 81,
          "y": 15,
          "value": 0,
          "name": "cell82 ~ gene16"
        },
        {
          "x": 81,
          "y": 16,
          "value": 0,
          "name": "cell82 ~ gene17"
        },
        {
          "x": 81,
          "y": 17,
          "value": 0,
          "name": "cell82 ~ gene18"
        },
        {
          "x": 81,
          "y": 18,
          "value": 13,
          "name": "cell82 ~ gene19"
        },
        {
          "x": 81,
          "y": 19,
          "value": 6,
          "name": "cell82 ~ gene20"
        },
        {
          "x": 81,
          "y": 20,
          "value": 12,
          "name": "cell82 ~ gene21"
        },
        {
          "x": 81,
          "y": 21,
          "value": 0,
          "name": "cell82 ~ gene22"
        },
        {
          "x": 81,
          "y": 22,
          "value": 11,
          "name": "cell82 ~ gene23"
        },
        {
          "x": 81,
          "y": 23,
          "value": 4,
          "name": "cell82 ~ gene24"
        },
        {
          "x": 81,
          "y": 24,
          "value": 0,
          "name": "cell82 ~ gene25"
        },
        {
          "x": 81,
          "y": 25,
          "value": 7,
          "name": "cell82 ~ gene26"
        },
        {
          "x": 81,
          "y": 26,
          "value": 14,
          "name": "cell82 ~ gene27"
        },
        {
          "x": 81,
          "y": 27,
          "value": 0,
          "name": "cell82 ~ gene28"
        },
        {
          "x": 81,
          "y": 28,
          "value": 0,
          "name": "cell82 ~ gene29"
        },
        {
          "x": 81,
          "y": 29,
          "value": 7,
          "name": "cell82 ~ gene30"
        },
        {
          "x": 81,
          "y": 30,
          "value": 0,
          "name": "cell82 ~ gene31"
        },
        {
          "x": 81,
          "y": 31,
          "value": 4,
          "name": "cell82 ~ gene32"
        },
        {
          "x": 81,
          "y": 32,
          "value": 4,
          "name": "cell82 ~ gene33"
        },
        {
          "x": 81,
          "y": 33,
          "value": 0,
          "name": "cell82 ~ gene34"
        },
        {
          "x": 81,
          "y": 34,
          "value": 7,
          "name": "cell82 ~ gene35"
        },
        {
          "x": 81,
          "y": 35,
          "value": 3,
          "name": "cell82 ~ gene36"
        },
        {
          "x": 81,
          "y": 36,
          "value": 0,
          "name": "cell82 ~ gene37"
        },
        {
          "x": 81,
          "y": 37,
          "value": 0,
          "name": "cell82 ~ gene38"
        },
        {
          "x": 81,
          "y": 38,
          "value": 2,
          "name": "cell82 ~ gene39"
        },
        {
          "x": 81,
          "y": 39,
          "value": 0,
          "name": "cell82 ~ gene40"
        },
        {
          "x": 81,
          "y": 40,
          "value": 31,
          "name": "cell82 ~ gene41"
        },
        {
          "x": 81,
          "y": 41,
          "value": 31,
          "name": "cell82 ~ gene42"
        },
        {
          "x": 81,
          "y": 42,
          "value": 0,
          "name": "cell82 ~ gene43"
        },
        {
          "x": 81,
          "y": 43,
          "value": 0,
          "name": "cell82 ~ gene44"
        },
        {
          "x": 81,
          "y": 44,
          "value": 14,
          "name": "cell82 ~ gene45"
        },
        {
          "x": 81,
          "y": 45,
          "value": 3,
          "name": "cell82 ~ gene46"
        },
        {
          "x": 81,
          "y": 46,
          "value": 12,
          "name": "cell82 ~ gene47"
        },
        {
          "x": 81,
          "y": 47,
          "value": 16,
          "name": "cell82 ~ gene48"
        },
        {
          "x": 81,
          "y": 48,
          "value": 0,
          "name": "cell82 ~ gene49"
        },
        {
          "x": 81,
          "y": 49,
          "value": 13,
          "name": "cell82 ~ gene50"
        },
        {
          "x": 81,
          "y": 50,
          "value": 20,
          "name": "cell82 ~ gene51"
        },
        {
          "x": 81,
          "y": 51,
          "value": 13,
          "name": "cell82 ~ gene52"
        },
        {
          "x": 81,
          "y": 52,
          "value": 21,
          "name": "cell82 ~ gene53"
        },
        {
          "x": 81,
          "y": 53,
          "value": 17,
          "name": "cell82 ~ gene54"
        },
        {
          "x": 81,
          "y": 54,
          "value": 29,
          "name": "cell82 ~ gene55"
        },
        {
          "x": 81,
          "y": 55,
          "value": 41,
          "name": "cell82 ~ gene56"
        },
        {
          "x": 81,
          "y": 56,
          "value": 0,
          "name": "cell82 ~ gene57"
        },
        {
          "x": 81,
          "y": 57,
          "value": 3,
          "name": "cell82 ~ gene58"
        },
        {
          "x": 81,
          "y": 58,
          "value": 8,
          "name": "cell82 ~ gene59"
        },
        {
          "x": 81,
          "y": 59,
          "value": 0,
          "name": "cell82 ~ gene60"
        },
        {
          "x": 81,
          "y": 60,
          "value": 0,
          "name": "cell82 ~ gene61"
        },
        {
          "x": 81,
          "y": 61,
          "value": 14,
          "name": "cell82 ~ gene62"
        },
        {
          "x": 81,
          "y": 62,
          "value": 0,
          "name": "cell82 ~ gene63"
        },
        {
          "x": 81,
          "y": 63,
          "value": 0,
          "name": "cell82 ~ gene64"
        },
        {
          "x": 81,
          "y": 64,
          "value": 12,
          "name": "cell82 ~ gene65"
        },
        {
          "x": 81,
          "y": 65,
          "value": 4,
          "name": "cell82 ~ gene66"
        },
        {
          "x": 81,
          "y": 66,
          "value": 0,
          "name": "cell82 ~ gene67"
        },
        {
          "x": 81,
          "y": 67,
          "value": 1,
          "name": "cell82 ~ gene68"
        },
        {
          "x": 81,
          "y": 68,
          "value": 0,
          "name": "cell82 ~ gene69"
        },
        {
          "x": 81,
          "y": 69,
          "value": 14,
          "name": "cell82 ~ gene70"
        },
        {
          "x": 81,
          "y": 70,
          "value": 0,
          "name": "cell82 ~ gene71"
        },
        {
          "x": 81,
          "y": 71,
          "value": 9,
          "name": "cell82 ~ gene72"
        },
        {
          "x": 81,
          "y": 72,
          "value": 12,
          "name": "cell82 ~ gene73"
        },
        {
          "x": 81,
          "y": 73,
          "value": 19,
          "name": "cell82 ~ gene74"
        },
        {
          "x": 81,
          "y": 74,
          "value": 0,
          "name": "cell82 ~ gene75"
        },
        {
          "x": 81,
          "y": 75,
          "value": 0,
          "name": "cell82 ~ gene76"
        },
        {
          "x": 81,
          "y": 76,
          "value": 0,
          "name": "cell82 ~ gene77"
        },
        {
          "x": 81,
          "y": 77,
          "value": 0,
          "name": "cell82 ~ gene78"
        },
        {
          "x": 81,
          "y": 78,
          "value": 10,
          "name": "cell82 ~ gene79"
        },
        {
          "x": 81,
          "y": 79,
          "value": 11,
          "name": "cell82 ~ gene80"
        },
        {
          "x": 81,
          "y": 80,
          "value": 3,
          "name": "cell82 ~ gene81"
        },
        {
          "x": 81,
          "y": 81,
          "value": 0,
          "name": "cell82 ~ gene82"
        },
        {
          "x": 81,
          "y": 82,
          "value": 0,
          "name": "cell82 ~ gene83"
        },
        {
          "x": 81,
          "y": 83,
          "value": 0,
          "name": "cell82 ~ gene84"
        },
        {
          "x": 81,
          "y": 84,
          "value": 0,
          "name": "cell82 ~ gene85"
        },
        {
          "x": 81,
          "y": 85,
          "value": 0,
          "name": "cell82 ~ gene86"
        },
        {
          "x": 81,
          "y": 86,
          "value": 0,
          "name": "cell82 ~ gene87"
        },
        {
          "x": 81,
          "y": 87,
          "value": 0,
          "name": "cell82 ~ gene88"
        },
        {
          "x": 81,
          "y": 88,
          "value": 0,
          "name": "cell82 ~ gene89"
        },
        {
          "x": 81,
          "y": 89,
          "value": 10,
          "name": "cell82 ~ gene90"
        },
        {
          "x": 81,
          "y": 90,
          "value": 3,
          "name": "cell82 ~ gene91"
        },
        {
          "x": 81,
          "y": 91,
          "value": 0,
          "name": "cell82 ~ gene92"
        },
        {
          "x": 81,
          "y": 92,
          "value": 9,
          "name": "cell82 ~ gene93"
        },
        {
          "x": 81,
          "y": 93,
          "value": 3,
          "name": "cell82 ~ gene94"
        },
        {
          "x": 81,
          "y": 94,
          "value": 11,
          "name": "cell82 ~ gene95"
        },
        {
          "x": 81,
          "y": 95,
          "value": 1,
          "name": "cell82 ~ gene96"
        },
        {
          "x": 81,
          "y": 96,
          "value": 9,
          "name": "cell82 ~ gene97"
        },
        {
          "x": 81,
          "y": 97,
          "value": 0,
          "name": "cell82 ~ gene98"
        },
        {
          "x": 81,
          "y": 98,
          "value": 8,
          "name": "cell82 ~ gene99"
        },
        {
          "x": 81,
          "y": 99,
          "value": 20,
          "name": "cell82 ~ gene100"
        },
        {
          "x": 82,
          "y": 0,
          "value": 3,
          "name": "cell83 ~ gene1"
        },
        {
          "x": 82,
          "y": 1,
          "value": 0,
          "name": "cell83 ~ gene2"
        },
        {
          "x": 82,
          "y": 2,
          "value": 0,
          "name": "cell83 ~ gene3"
        },
        {
          "x": 82,
          "y": 3,
          "value": 0,
          "name": "cell83 ~ gene4"
        },
        {
          "x": 82,
          "y": 4,
          "value": 15,
          "name": "cell83 ~ gene5"
        },
        {
          "x": 82,
          "y": 5,
          "value": 0,
          "name": "cell83 ~ gene6"
        },
        {
          "x": 82,
          "y": 6,
          "value": 0,
          "name": "cell83 ~ gene7"
        },
        {
          "x": 82,
          "y": 7,
          "value": 0,
          "name": "cell83 ~ gene8"
        },
        {
          "x": 82,
          "y": 8,
          "value": 0,
          "name": "cell83 ~ gene9"
        },
        {
          "x": 82,
          "y": 9,
          "value": 0,
          "name": "cell83 ~ gene10"
        },
        {
          "x": 82,
          "y": 10,
          "value": 6,
          "name": "cell83 ~ gene11"
        },
        {
          "x": 82,
          "y": 11,
          "value": 0,
          "name": "cell83 ~ gene12"
        },
        {
          "x": 82,
          "y": 12,
          "value": 0,
          "name": "cell83 ~ gene13"
        },
        {
          "x": 82,
          "y": 13,
          "value": 5,
          "name": "cell83 ~ gene14"
        },
        {
          "x": 82,
          "y": 14,
          "value": 15,
          "name": "cell83 ~ gene15"
        },
        {
          "x": 82,
          "y": 15,
          "value": 0,
          "name": "cell83 ~ gene16"
        },
        {
          "x": 82,
          "y": 16,
          "value": 0,
          "name": "cell83 ~ gene17"
        },
        {
          "x": 82,
          "y": 17,
          "value": 0,
          "name": "cell83 ~ gene18"
        },
        {
          "x": 82,
          "y": 18,
          "value": 4,
          "name": "cell83 ~ gene19"
        },
        {
          "x": 82,
          "y": 19,
          "value": 0,
          "name": "cell83 ~ gene20"
        },
        {
          "x": 82,
          "y": 20,
          "value": 5,
          "name": "cell83 ~ gene21"
        },
        {
          "x": 82,
          "y": 21,
          "value": 0,
          "name": "cell83 ~ gene22"
        },
        {
          "x": 82,
          "y": 22,
          "value": 8,
          "name": "cell83 ~ gene23"
        },
        {
          "x": 82,
          "y": 23,
          "value": 0,
          "name": "cell83 ~ gene24"
        },
        {
          "x": 82,
          "y": 24,
          "value": 10,
          "name": "cell83 ~ gene25"
        },
        {
          "x": 82,
          "y": 25,
          "value": 0,
          "name": "cell83 ~ gene26"
        },
        {
          "x": 82,
          "y": 26,
          "value": 0,
          "name": "cell83 ~ gene27"
        },
        {
          "x": 82,
          "y": 27,
          "value": 4,
          "name": "cell83 ~ gene28"
        },
        {
          "x": 82,
          "y": 28,
          "value": 0,
          "name": "cell83 ~ gene29"
        },
        {
          "x": 82,
          "y": 29,
          "value": 4,
          "name": "cell83 ~ gene30"
        },
        {
          "x": 82,
          "y": 30,
          "value": 9,
          "name": "cell83 ~ gene31"
        },
        {
          "x": 82,
          "y": 31,
          "value": 0,
          "name": "cell83 ~ gene32"
        },
        {
          "x": 82,
          "y": 32,
          "value": 4,
          "name": "cell83 ~ gene33"
        },
        {
          "x": 82,
          "y": 33,
          "value": 0,
          "name": "cell83 ~ gene34"
        },
        {
          "x": 82,
          "y": 34,
          "value": 17,
          "name": "cell83 ~ gene35"
        },
        {
          "x": 82,
          "y": 35,
          "value": 0,
          "name": "cell83 ~ gene36"
        },
        {
          "x": 82,
          "y": 36,
          "value": 9,
          "name": "cell83 ~ gene37"
        },
        {
          "x": 82,
          "y": 37,
          "value": 0,
          "name": "cell83 ~ gene38"
        },
        {
          "x": 82,
          "y": 38,
          "value": 0,
          "name": "cell83 ~ gene39"
        },
        {
          "x": 82,
          "y": 39,
          "value": 3,
          "name": "cell83 ~ gene40"
        },
        {
          "x": 82,
          "y": 40,
          "value": 0,
          "name": "cell83 ~ gene41"
        },
        {
          "x": 82,
          "y": 41,
          "value": 12,
          "name": "cell83 ~ gene42"
        },
        {
          "x": 82,
          "y": 42,
          "value": 6,
          "name": "cell83 ~ gene43"
        },
        {
          "x": 82,
          "y": 43,
          "value": 0,
          "name": "cell83 ~ gene44"
        },
        {
          "x": 82,
          "y": 44,
          "value": 0,
          "name": "cell83 ~ gene45"
        },
        {
          "x": 82,
          "y": 45,
          "value": 20,
          "name": "cell83 ~ gene46"
        },
        {
          "x": 82,
          "y": 46,
          "value": 2,
          "name": "cell83 ~ gene47"
        },
        {
          "x": 82,
          "y": 47,
          "value": 9,
          "name": "cell83 ~ gene48"
        },
        {
          "x": 82,
          "y": 48,
          "value": 16,
          "name": "cell83 ~ gene49"
        },
        {
          "x": 82,
          "y": 49,
          "value": 9,
          "name": "cell83 ~ gene50"
        },
        {
          "x": 82,
          "y": 50,
          "value": 7,
          "name": "cell83 ~ gene51"
        },
        {
          "x": 82,
          "y": 51,
          "value": 2,
          "name": "cell83 ~ gene52"
        },
        {
          "x": 82,
          "y": 52,
          "value": 1,
          "name": "cell83 ~ gene53"
        },
        {
          "x": 82,
          "y": 53,
          "value": 0,
          "name": "cell83 ~ gene54"
        },
        {
          "x": 82,
          "y": 54,
          "value": 27,
          "name": "cell83 ~ gene55"
        },
        {
          "x": 82,
          "y": 55,
          "value": 19,
          "name": "cell83 ~ gene56"
        },
        {
          "x": 82,
          "y": 56,
          "value": 0,
          "name": "cell83 ~ gene57"
        },
        {
          "x": 82,
          "y": 57,
          "value": 12,
          "name": "cell83 ~ gene58"
        },
        {
          "x": 82,
          "y": 58,
          "value": 2,
          "name": "cell83 ~ gene59"
        },
        {
          "x": 82,
          "y": 59,
          "value": 0,
          "name": "cell83 ~ gene60"
        },
        {
          "x": 82,
          "y": 60,
          "value": 8,
          "name": "cell83 ~ gene61"
        },
        {
          "x": 82,
          "y": 61,
          "value": 0,
          "name": "cell83 ~ gene62"
        },
        {
          "x": 82,
          "y": 62,
          "value": 0,
          "name": "cell83 ~ gene63"
        },
        {
          "x": 82,
          "y": 63,
          "value": 0,
          "name": "cell83 ~ gene64"
        },
        {
          "x": 82,
          "y": 64,
          "value": 0,
          "name": "cell83 ~ gene65"
        },
        {
          "x": 82,
          "y": 65,
          "value": 8,
          "name": "cell83 ~ gene66"
        },
        {
          "x": 82,
          "y": 66,
          "value": 0,
          "name": "cell83 ~ gene67"
        },
        {
          "x": 82,
          "y": 67,
          "value": 9,
          "name": "cell83 ~ gene68"
        },
        {
          "x": 82,
          "y": 68,
          "value": 0,
          "name": "cell83 ~ gene69"
        },
        {
          "x": 82,
          "y": 69,
          "value": 18,
          "name": "cell83 ~ gene70"
        },
        {
          "x": 82,
          "y": 70,
          "value": 17,
          "name": "cell83 ~ gene71"
        },
        {
          "x": 82,
          "y": 71,
          "value": 0,
          "name": "cell83 ~ gene72"
        },
        {
          "x": 82,
          "y": 72,
          "value": 0,
          "name": "cell83 ~ gene73"
        },
        {
          "x": 82,
          "y": 73,
          "value": 3,
          "name": "cell83 ~ gene74"
        },
        {
          "x": 82,
          "y": 74,
          "value": 0,
          "name": "cell83 ~ gene75"
        },
        {
          "x": 82,
          "y": 75,
          "value": 7,
          "name": "cell83 ~ gene76"
        },
        {
          "x": 82,
          "y": 76,
          "value": 1,
          "name": "cell83 ~ gene77"
        },
        {
          "x": 82,
          "y": 77,
          "value": 0,
          "name": "cell83 ~ gene78"
        },
        {
          "x": 82,
          "y": 78,
          "value": 10,
          "name": "cell83 ~ gene79"
        },
        {
          "x": 82,
          "y": 79,
          "value": 0,
          "name": "cell83 ~ gene80"
        },
        {
          "x": 82,
          "y": 80,
          "value": 1,
          "name": "cell83 ~ gene81"
        },
        {
          "x": 82,
          "y": 81,
          "value": 0,
          "name": "cell83 ~ gene82"
        },
        {
          "x": 82,
          "y": 82,
          "value": 6,
          "name": "cell83 ~ gene83"
        },
        {
          "x": 82,
          "y": 83,
          "value": 0,
          "name": "cell83 ~ gene84"
        },
        {
          "x": 82,
          "y": 84,
          "value": 13,
          "name": "cell83 ~ gene85"
        },
        {
          "x": 82,
          "y": 85,
          "value": 0,
          "name": "cell83 ~ gene86"
        },
        {
          "x": 82,
          "y": 86,
          "value": 26,
          "name": "cell83 ~ gene87"
        },
        {
          "x": 82,
          "y": 87,
          "value": 0,
          "name": "cell83 ~ gene88"
        },
        {
          "x": 82,
          "y": 88,
          "value": 14,
          "name": "cell83 ~ gene89"
        },
        {
          "x": 82,
          "y": 89,
          "value": 8,
          "name": "cell83 ~ gene90"
        },
        {
          "x": 82,
          "y": 90,
          "value": 7,
          "name": "cell83 ~ gene91"
        },
        {
          "x": 82,
          "y": 91,
          "value": 0,
          "name": "cell83 ~ gene92"
        },
        {
          "x": 82,
          "y": 92,
          "value": 0,
          "name": "cell83 ~ gene93"
        },
        {
          "x": 82,
          "y": 93,
          "value": 0,
          "name": "cell83 ~ gene94"
        },
        {
          "x": 82,
          "y": 94,
          "value": 6,
          "name": "cell83 ~ gene95"
        },
        {
          "x": 82,
          "y": 95,
          "value": 0,
          "name": "cell83 ~ gene96"
        },
        {
          "x": 82,
          "y": 96,
          "value": 2,
          "name": "cell83 ~ gene97"
        },
        {
          "x": 82,
          "y": 97,
          "value": 8,
          "name": "cell83 ~ gene98"
        },
        {
          "x": 82,
          "y": 98,
          "value": 1,
          "name": "cell83 ~ gene99"
        },
        {
          "x": 82,
          "y": 99,
          "value": 9,
          "name": "cell83 ~ gene100"
        },
        {
          "x": 83,
          "y": 0,
          "value": 5,
          "name": "cell84 ~ gene1"
        },
        {
          "x": 83,
          "y": 1,
          "value": 6,
          "name": "cell84 ~ gene2"
        },
        {
          "x": 83,
          "y": 2,
          "value": 15,
          "name": "cell84 ~ gene3"
        },
        {
          "x": 83,
          "y": 3,
          "value": 0,
          "name": "cell84 ~ gene4"
        },
        {
          "x": 83,
          "y": 4,
          "value": 0,
          "name": "cell84 ~ gene5"
        },
        {
          "x": 83,
          "y": 5,
          "value": 7,
          "name": "cell84 ~ gene6"
        },
        {
          "x": 83,
          "y": 6,
          "value": 0,
          "name": "cell84 ~ gene7"
        },
        {
          "x": 83,
          "y": 7,
          "value": 4,
          "name": "cell84 ~ gene8"
        },
        {
          "x": 83,
          "y": 8,
          "value": 0,
          "name": "cell84 ~ gene9"
        },
        {
          "x": 83,
          "y": 9,
          "value": 0,
          "name": "cell84 ~ gene10"
        },
        {
          "x": 83,
          "y": 10,
          "value": 0,
          "name": "cell84 ~ gene11"
        },
        {
          "x": 83,
          "y": 11,
          "value": 0,
          "name": "cell84 ~ gene12"
        },
        {
          "x": 83,
          "y": 12,
          "value": 4,
          "name": "cell84 ~ gene13"
        },
        {
          "x": 83,
          "y": 13,
          "value": 3,
          "name": "cell84 ~ gene14"
        },
        {
          "x": 83,
          "y": 14,
          "value": 0,
          "name": "cell84 ~ gene15"
        },
        {
          "x": 83,
          "y": 15,
          "value": 5,
          "name": "cell84 ~ gene16"
        },
        {
          "x": 83,
          "y": 16,
          "value": 0,
          "name": "cell84 ~ gene17"
        },
        {
          "x": 83,
          "y": 17,
          "value": 0,
          "name": "cell84 ~ gene18"
        },
        {
          "x": 83,
          "y": 18,
          "value": 0,
          "name": "cell84 ~ gene19"
        },
        {
          "x": 83,
          "y": 19,
          "value": 1,
          "name": "cell84 ~ gene20"
        },
        {
          "x": 83,
          "y": 20,
          "value": 0,
          "name": "cell84 ~ gene21"
        },
        {
          "x": 83,
          "y": 21,
          "value": 18,
          "name": "cell84 ~ gene22"
        },
        {
          "x": 83,
          "y": 22,
          "value": 0,
          "name": "cell84 ~ gene23"
        },
        {
          "x": 83,
          "y": 23,
          "value": 0,
          "name": "cell84 ~ gene24"
        },
        {
          "x": 83,
          "y": 24,
          "value": 0,
          "name": "cell84 ~ gene25"
        },
        {
          "x": 83,
          "y": 25,
          "value": 0,
          "name": "cell84 ~ gene26"
        },
        {
          "x": 83,
          "y": 26,
          "value": 4,
          "name": "cell84 ~ gene27"
        },
        {
          "x": 83,
          "y": 27,
          "value": 29,
          "name": "cell84 ~ gene28"
        },
        {
          "x": 83,
          "y": 28,
          "value": 24,
          "name": "cell84 ~ gene29"
        },
        {
          "x": 83,
          "y": 29,
          "value": 0,
          "name": "cell84 ~ gene30"
        },
        {
          "x": 83,
          "y": 30,
          "value": 0,
          "name": "cell84 ~ gene31"
        },
        {
          "x": 83,
          "y": 31,
          "value": 0,
          "name": "cell84 ~ gene32"
        },
        {
          "x": 83,
          "y": 32,
          "value": 3,
          "name": "cell84 ~ gene33"
        },
        {
          "x": 83,
          "y": 33,
          "value": 0,
          "name": "cell84 ~ gene34"
        },
        {
          "x": 83,
          "y": 34,
          "value": 0,
          "name": "cell84 ~ gene35"
        },
        {
          "x": 83,
          "y": 35,
          "value": 2,
          "name": "cell84 ~ gene36"
        },
        {
          "x": 83,
          "y": 36,
          "value": 0,
          "name": "cell84 ~ gene37"
        },
        {
          "x": 83,
          "y": 37,
          "value": 0,
          "name": "cell84 ~ gene38"
        },
        {
          "x": 83,
          "y": 38,
          "value": 1,
          "name": "cell84 ~ gene39"
        },
        {
          "x": 83,
          "y": 39,
          "value": 0,
          "name": "cell84 ~ gene40"
        },
        {
          "x": 83,
          "y": 40,
          "value": 3,
          "name": "cell84 ~ gene41"
        },
        {
          "x": 83,
          "y": 41,
          "value": 28,
          "name": "cell84 ~ gene42"
        },
        {
          "x": 83,
          "y": 42,
          "value": 10,
          "name": "cell84 ~ gene43"
        },
        {
          "x": 83,
          "y": 43,
          "value": 18,
          "name": "cell84 ~ gene44"
        },
        {
          "x": 83,
          "y": 44,
          "value": 22,
          "name": "cell84 ~ gene45"
        },
        {
          "x": 83,
          "y": 45,
          "value": 18,
          "name": "cell84 ~ gene46"
        },
        {
          "x": 83,
          "y": 46,
          "value": 25,
          "name": "cell84 ~ gene47"
        },
        {
          "x": 83,
          "y": 47,
          "value": 3,
          "name": "cell84 ~ gene48"
        },
        {
          "x": 83,
          "y": 48,
          "value": 0,
          "name": "cell84 ~ gene49"
        },
        {
          "x": 83,
          "y": 49,
          "value": 32,
          "name": "cell84 ~ gene50"
        },
        {
          "x": 83,
          "y": 50,
          "value": 6,
          "name": "cell84 ~ gene51"
        },
        {
          "x": 83,
          "y": 51,
          "value": 13,
          "name": "cell84 ~ gene52"
        },
        {
          "x": 83,
          "y": 52,
          "value": 8,
          "name": "cell84 ~ gene53"
        },
        {
          "x": 83,
          "y": 53,
          "value": 14,
          "name": "cell84 ~ gene54"
        },
        {
          "x": 83,
          "y": 54,
          "value": 20,
          "name": "cell84 ~ gene55"
        },
        {
          "x": 83,
          "y": 55,
          "value": 23,
          "name": "cell84 ~ gene56"
        },
        {
          "x": 83,
          "y": 56,
          "value": 9,
          "name": "cell84 ~ gene57"
        },
        {
          "x": 83,
          "y": 57,
          "value": 15,
          "name": "cell84 ~ gene58"
        },
        {
          "x": 83,
          "y": 58,
          "value": 0,
          "name": "cell84 ~ gene59"
        },
        {
          "x": 83,
          "y": 59,
          "value": 0,
          "name": "cell84 ~ gene60"
        },
        {
          "x": 83,
          "y": 60,
          "value": 10,
          "name": "cell84 ~ gene61"
        },
        {
          "x": 83,
          "y": 61,
          "value": 0,
          "name": "cell84 ~ gene62"
        },
        {
          "x": 83,
          "y": 62,
          "value": 8,
          "name": "cell84 ~ gene63"
        },
        {
          "x": 83,
          "y": 63,
          "value": 7,
          "name": "cell84 ~ gene64"
        },
        {
          "x": 83,
          "y": 64,
          "value": 0,
          "name": "cell84 ~ gene65"
        },
        {
          "x": 83,
          "y": 65,
          "value": 2,
          "name": "cell84 ~ gene66"
        },
        {
          "x": 83,
          "y": 66,
          "value": 4,
          "name": "cell84 ~ gene67"
        },
        {
          "x": 83,
          "y": 67,
          "value": 13,
          "name": "cell84 ~ gene68"
        },
        {
          "x": 83,
          "y": 68,
          "value": 0,
          "name": "cell84 ~ gene69"
        },
        {
          "x": 83,
          "y": 69,
          "value": 0,
          "name": "cell84 ~ gene70"
        },
        {
          "x": 83,
          "y": 70,
          "value": 0,
          "name": "cell84 ~ gene71"
        },
        {
          "x": 83,
          "y": 71,
          "value": 10,
          "name": "cell84 ~ gene72"
        },
        {
          "x": 83,
          "y": 72,
          "value": 0,
          "name": "cell84 ~ gene73"
        },
        {
          "x": 83,
          "y": 73,
          "value": 0,
          "name": "cell84 ~ gene74"
        },
        {
          "x": 83,
          "y": 74,
          "value": 6,
          "name": "cell84 ~ gene75"
        },
        {
          "x": 83,
          "y": 75,
          "value": 0,
          "name": "cell84 ~ gene76"
        },
        {
          "x": 83,
          "y": 76,
          "value": 0,
          "name": "cell84 ~ gene77"
        },
        {
          "x": 83,
          "y": 77,
          "value": 0,
          "name": "cell84 ~ gene78"
        },
        {
          "x": 83,
          "y": 78,
          "value": 0,
          "name": "cell84 ~ gene79"
        },
        {
          "x": 83,
          "y": 79,
          "value": 0,
          "name": "cell84 ~ gene80"
        },
        {
          "x": 83,
          "y": 80,
          "value": 6,
          "name": "cell84 ~ gene81"
        },
        {
          "x": 83,
          "y": 81,
          "value": 0,
          "name": "cell84 ~ gene82"
        },
        {
          "x": 83,
          "y": 82,
          "value": 0,
          "name": "cell84 ~ gene83"
        },
        {
          "x": 83,
          "y": 83,
          "value": 8,
          "name": "cell84 ~ gene84"
        },
        {
          "x": 83,
          "y": 84,
          "value": 9,
          "name": "cell84 ~ gene85"
        },
        {
          "x": 83,
          "y": 85,
          "value": 10,
          "name": "cell84 ~ gene86"
        },
        {
          "x": 83,
          "y": 86,
          "value": 0,
          "name": "cell84 ~ gene87"
        },
        {
          "x": 83,
          "y": 87,
          "value": 10,
          "name": "cell84 ~ gene88"
        },
        {
          "x": 83,
          "y": 88,
          "value": 0,
          "name": "cell84 ~ gene89"
        },
        {
          "x": 83,
          "y": 89,
          "value": 0,
          "name": "cell84 ~ gene90"
        },
        {
          "x": 83,
          "y": 90,
          "value": 5,
          "name": "cell84 ~ gene91"
        },
        {
          "x": 83,
          "y": 91,
          "value": 0,
          "name": "cell84 ~ gene92"
        },
        {
          "x": 83,
          "y": 92,
          "value": 5,
          "name": "cell84 ~ gene93"
        },
        {
          "x": 83,
          "y": 93,
          "value": 1,
          "name": "cell84 ~ gene94"
        },
        {
          "x": 83,
          "y": 94,
          "value": 0,
          "name": "cell84 ~ gene95"
        },
        {
          "x": 83,
          "y": 95,
          "value": 0,
          "name": "cell84 ~ gene96"
        },
        {
          "x": 83,
          "y": 96,
          "value": 0,
          "name": "cell84 ~ gene97"
        },
        {
          "x": 83,
          "y": 97,
          "value": 0,
          "name": "cell84 ~ gene98"
        },
        {
          "x": 83,
          "y": 98,
          "value": 7,
          "name": "cell84 ~ gene99"
        },
        {
          "x": 83,
          "y": 99,
          "value": 0,
          "name": "cell84 ~ gene100"
        },
        {
          "x": 84,
          "y": 0,
          "value": 0,
          "name": "cell85 ~ gene1"
        },
        {
          "x": 84,
          "y": 1,
          "value": 1,
          "name": "cell85 ~ gene2"
        },
        {
          "x": 84,
          "y": 2,
          "value": 0,
          "name": "cell85 ~ gene3"
        },
        {
          "x": 84,
          "y": 3,
          "value": 0,
          "name": "cell85 ~ gene4"
        },
        {
          "x": 84,
          "y": 4,
          "value": 9,
          "name": "cell85 ~ gene5"
        },
        {
          "x": 84,
          "y": 5,
          "value": 0,
          "name": "cell85 ~ gene6"
        },
        {
          "x": 84,
          "y": 6,
          "value": 0,
          "name": "cell85 ~ gene7"
        },
        {
          "x": 84,
          "y": 7,
          "value": 1,
          "name": "cell85 ~ gene8"
        },
        {
          "x": 84,
          "y": 8,
          "value": 0,
          "name": "cell85 ~ gene9"
        },
        {
          "x": 84,
          "y": 9,
          "value": 0,
          "name": "cell85 ~ gene10"
        },
        {
          "x": 84,
          "y": 10,
          "value": 0,
          "name": "cell85 ~ gene11"
        },
        {
          "x": 84,
          "y": 11,
          "value": 11,
          "name": "cell85 ~ gene12"
        },
        {
          "x": 84,
          "y": 12,
          "value": 28,
          "name": "cell85 ~ gene13"
        },
        {
          "x": 84,
          "y": 13,
          "value": 3,
          "name": "cell85 ~ gene14"
        },
        {
          "x": 84,
          "y": 14,
          "value": 1,
          "name": "cell85 ~ gene15"
        },
        {
          "x": 84,
          "y": 15,
          "value": 10,
          "name": "cell85 ~ gene16"
        },
        {
          "x": 84,
          "y": 16,
          "value": 0,
          "name": "cell85 ~ gene17"
        },
        {
          "x": 84,
          "y": 17,
          "value": 0,
          "name": "cell85 ~ gene18"
        },
        {
          "x": 84,
          "y": 18,
          "value": 0,
          "name": "cell85 ~ gene19"
        },
        {
          "x": 84,
          "y": 19,
          "value": 1,
          "name": "cell85 ~ gene20"
        },
        {
          "x": 84,
          "y": 20,
          "value": 0,
          "name": "cell85 ~ gene21"
        },
        {
          "x": 84,
          "y": 21,
          "value": 0,
          "name": "cell85 ~ gene22"
        },
        {
          "x": 84,
          "y": 22,
          "value": 0,
          "name": "cell85 ~ gene23"
        },
        {
          "x": 84,
          "y": 23,
          "value": 8,
          "name": "cell85 ~ gene24"
        },
        {
          "x": 84,
          "y": 24,
          "value": 6,
          "name": "cell85 ~ gene25"
        },
        {
          "x": 84,
          "y": 25,
          "value": 0,
          "name": "cell85 ~ gene26"
        },
        {
          "x": 84,
          "y": 26,
          "value": 4,
          "name": "cell85 ~ gene27"
        },
        {
          "x": 84,
          "y": 27,
          "value": 0,
          "name": "cell85 ~ gene28"
        },
        {
          "x": 84,
          "y": 28,
          "value": 0,
          "name": "cell85 ~ gene29"
        },
        {
          "x": 84,
          "y": 29,
          "value": 0,
          "name": "cell85 ~ gene30"
        },
        {
          "x": 84,
          "y": 30,
          "value": 0,
          "name": "cell85 ~ gene31"
        },
        {
          "x": 84,
          "y": 31,
          "value": 0,
          "name": "cell85 ~ gene32"
        },
        {
          "x": 84,
          "y": 32,
          "value": 0,
          "name": "cell85 ~ gene33"
        },
        {
          "x": 84,
          "y": 33,
          "value": 3,
          "name": "cell85 ~ gene34"
        },
        {
          "x": 84,
          "y": 34,
          "value": 7,
          "name": "cell85 ~ gene35"
        },
        {
          "x": 84,
          "y": 35,
          "value": 0,
          "name": "cell85 ~ gene36"
        },
        {
          "x": 84,
          "y": 36,
          "value": 12,
          "name": "cell85 ~ gene37"
        },
        {
          "x": 84,
          "y": 37,
          "value": 0,
          "name": "cell85 ~ gene38"
        },
        {
          "x": 84,
          "y": 38,
          "value": 0,
          "name": "cell85 ~ gene39"
        },
        {
          "x": 84,
          "y": 39,
          "value": 13,
          "name": "cell85 ~ gene40"
        },
        {
          "x": 84,
          "y": 40,
          "value": 1,
          "name": "cell85 ~ gene41"
        },
        {
          "x": 84,
          "y": 41,
          "value": 27,
          "name": "cell85 ~ gene42"
        },
        {
          "x": 84,
          "y": 42,
          "value": 3,
          "name": "cell85 ~ gene43"
        },
        {
          "x": 84,
          "y": 43,
          "value": 0,
          "name": "cell85 ~ gene44"
        },
        {
          "x": 84,
          "y": 44,
          "value": 4,
          "name": "cell85 ~ gene45"
        },
        {
          "x": 84,
          "y": 45,
          "value": 18,
          "name": "cell85 ~ gene46"
        },
        {
          "x": 84,
          "y": 46,
          "value": 19,
          "name": "cell85 ~ gene47"
        },
        {
          "x": 84,
          "y": 47,
          "value": 0,
          "name": "cell85 ~ gene48"
        },
        {
          "x": 84,
          "y": 48,
          "value": 5,
          "name": "cell85 ~ gene49"
        },
        {
          "x": 84,
          "y": 49,
          "value": 1,
          "name": "cell85 ~ gene50"
        },
        {
          "x": 84,
          "y": 50,
          "value": 19,
          "name": "cell85 ~ gene51"
        },
        {
          "x": 84,
          "y": 51,
          "value": 6,
          "name": "cell85 ~ gene52"
        },
        {
          "x": 84,
          "y": 52,
          "value": 44,
          "name": "cell85 ~ gene53"
        },
        {
          "x": 84,
          "y": 53,
          "value": 11,
          "name": "cell85 ~ gene54"
        },
        {
          "x": 84,
          "y": 54,
          "value": 32,
          "name": "cell85 ~ gene55"
        },
        {
          "x": 84,
          "y": 55,
          "value": 33,
          "name": "cell85 ~ gene56"
        },
        {
          "x": 84,
          "y": 56,
          "value": 13,
          "name": "cell85 ~ gene57"
        },
        {
          "x": 84,
          "y": 57,
          "value": 34,
          "name": "cell85 ~ gene58"
        },
        {
          "x": 84,
          "y": 58,
          "value": 0,
          "name": "cell85 ~ gene59"
        },
        {
          "x": 84,
          "y": 59,
          "value": 0,
          "name": "cell85 ~ gene60"
        },
        {
          "x": 84,
          "y": 60,
          "value": 0,
          "name": "cell85 ~ gene61"
        },
        {
          "x": 84,
          "y": 61,
          "value": 0,
          "name": "cell85 ~ gene62"
        },
        {
          "x": 84,
          "y": 62,
          "value": 0,
          "name": "cell85 ~ gene63"
        },
        {
          "x": 84,
          "y": 63,
          "value": 0,
          "name": "cell85 ~ gene64"
        },
        {
          "x": 84,
          "y": 64,
          "value": 0,
          "name": "cell85 ~ gene65"
        },
        {
          "x": 84,
          "y": 65,
          "value": 0,
          "name": "cell85 ~ gene66"
        },
        {
          "x": 84,
          "y": 66,
          "value": 5,
          "name": "cell85 ~ gene67"
        },
        {
          "x": 84,
          "y": 67,
          "value": 0,
          "name": "cell85 ~ gene68"
        },
        {
          "x": 84,
          "y": 68,
          "value": 3,
          "name": "cell85 ~ gene69"
        },
        {
          "x": 84,
          "y": 69,
          "value": 0,
          "name": "cell85 ~ gene70"
        },
        {
          "x": 84,
          "y": 70,
          "value": 5,
          "name": "cell85 ~ gene71"
        },
        {
          "x": 84,
          "y": 71,
          "value": 0,
          "name": "cell85 ~ gene72"
        },
        {
          "x": 84,
          "y": 72,
          "value": 0,
          "name": "cell85 ~ gene73"
        },
        {
          "x": 84,
          "y": 73,
          "value": 0,
          "name": "cell85 ~ gene74"
        },
        {
          "x": 84,
          "y": 74,
          "value": 7,
          "name": "cell85 ~ gene75"
        },
        {
          "x": 84,
          "y": 75,
          "value": 0,
          "name": "cell85 ~ gene76"
        },
        {
          "x": 84,
          "y": 76,
          "value": 0,
          "name": "cell85 ~ gene77"
        },
        {
          "x": 84,
          "y": 77,
          "value": 0,
          "name": "cell85 ~ gene78"
        },
        {
          "x": 84,
          "y": 78,
          "value": 0,
          "name": "cell85 ~ gene79"
        },
        {
          "x": 84,
          "y": 79,
          "value": 1,
          "name": "cell85 ~ gene80"
        },
        {
          "x": 84,
          "y": 80,
          "value": 0,
          "name": "cell85 ~ gene81"
        },
        {
          "x": 84,
          "y": 81,
          "value": 0,
          "name": "cell85 ~ gene82"
        },
        {
          "x": 84,
          "y": 82,
          "value": 0,
          "name": "cell85 ~ gene83"
        },
        {
          "x": 84,
          "y": 83,
          "value": 9,
          "name": "cell85 ~ gene84"
        },
        {
          "x": 84,
          "y": 84,
          "value": 0,
          "name": "cell85 ~ gene85"
        },
        {
          "x": 84,
          "y": 85,
          "value": 3,
          "name": "cell85 ~ gene86"
        },
        {
          "x": 84,
          "y": 86,
          "value": 0,
          "name": "cell85 ~ gene87"
        },
        {
          "x": 84,
          "y": 87,
          "value": 9,
          "name": "cell85 ~ gene88"
        },
        {
          "x": 84,
          "y": 88,
          "value": 0,
          "name": "cell85 ~ gene89"
        },
        {
          "x": 84,
          "y": 89,
          "value": 4,
          "name": "cell85 ~ gene90"
        },
        {
          "x": 84,
          "y": 90,
          "value": 0,
          "name": "cell85 ~ gene91"
        },
        {
          "x": 84,
          "y": 91,
          "value": 0,
        },
        {
        },