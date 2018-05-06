---
title: "Creating simulated data with correlations between them"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Trying to conduct Multidimensional Scaling (MDS) for concept mapping

So the data needs to be in the format all items by all items and then have people place all items into as many piles (themes as they want). For each item that is placed into the same pile put a one if not put a zero.  Do this for every participant. Need to have row names for (e.g. Item1 through ItemX) 
```{r}
library(MASS)
data(eurodist)
head(eurodist)
as.matrix(eurodist)
```
Now run the data analysis.  Basically PCA finds factors and which items load on those factors.  So we want a two dimensional analysis, because we want to plot the data on two dimensions I think?

Need Non-metric multidimensional scaling at some point.
```{r}
library(magrittr)
library(dplyr)
library(ggpubr)
data(swiss)
swiss
```
Here create data and figure out how to sum them and use code above to run it.
We need data with rownames and columns that are the same and then insert data like a matrix

I don't think you need to scale, because the data is on the same scale already.
```{r}
datTest = read.csv("datCorMD.csv", header = FALSE)
# Creating the two dimensions that the factors are on
mds = datTest %>%
  dist() %>%
  cmdscale() %>%
  as_tibble() 

colnames(mds) = c("Dim1", "Dim2")
mds

# Now trying to plot them start to see which ones are closest together on two dimensions
ggscatter(mds, x = "Dim1", y = "Dim2", label = rownames(datTest), size = 1, repel = TRUE)

# Now ones these two dimensions try to find out how they are related
# This places each item into its own cluster in the two dimensional space
clust = kmeans(mds, 3)$cluster %>%
  as.factor()
clust

# Now place them into the clusters?
mds = mds %>%
  mutate(groups = clust)
mds
# Now we want to create the graphs that are pretty
ggscatter(mds, x= "Dim1", y = "Dim2", label = rownames(datTest), color = "groups", palette = "jco", size = 1, ellipse = TRUE, ellipse.type = "convex", repel = TRUE)
```
Now same process but with the non-metric
```{r}
mds = datTest %>%
  dist() %>%
  isoMDS() %>%
  .$points %>%
  as_tibble()
colnames(mds) = c("Dim1", "Dim2")
mds = mds %>%
  mutate(groups = clust)
ggscatter(mds, x= "Dim1", y = "Dim2", label = rownames(datTest), color = "groups", palette = "jco", size = 1, ellipse = TRUE, ellipse.type = "convex", repel = TRUE)
```
Now try with hieractical clustering, which I think is better, because that is a better, but still need to know why: https://uc-r.github.io/hc_clustering
```{r}
install.packages("tidyverse")
install.packages("cluster")
install.packages("factoextra")
install.packages("dendextend")

library(tidyverse)
library(cluster)

```

