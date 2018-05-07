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
library(cluster)
library(factoextra)
library(magrittr)
library(dplyr)
library(ggpubr)
```
Now run the data analysis.  Basically PCA finds factors and which items load on those factors.  So we want a two dimensional analysis, because we want to plot the data on two dimensions I think?

Need Non-metric multidimensional scaling at some point.

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
mds
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
Messing around with stuff from book
K-means means you need to specifcy the number of cluster first

How to check kmeans with the elbow plot

Dist function computes euclidean distance
Then you need to create the clusters
I think you call the cophenetic function to identitfy the correlation between distatnces between the model implied distances and the actual distances

Good website: https://uc-r.github.io/hc_clustering

Create distance matrix first can use euclidean without scaleing, because everything is on the same scale.  But could scale, because the actual units don't matter.  Anges does the dismiliatiry matrix all together.  AC measures the clustering structure found with closer values to 1 indicating a better fit

I believe the dim percentages are the amount of variation explained by the two different factors.

Need to determine: the method (ward or something else), how many clusters(elbow, shib, and gap)
```{r}
setwd("~/Desktop")
datTest = read.csv("datCorMD.csv", header = TRUE)
datTest
hc = agnes(datTest, diss = FALSE, stand = TRUE, method = "ward")
hc$ac
hcTree = cutree(hc, k=5)

#Add back to the original data set to see which cluster the person belongs to
datTest %>%
  mutate(cluster = hcTree)
fviz_cluster(list(data =datTest, cluster = hcTree))
```
How to figure out the number of optimal clusters
Elbow method
Average silhouette method = measures how well each object or item lies within the 
Gap Cluster = This is comparison of the intracluster variation to what we would expect with a simulated data set with no inherent clustering structure
```{r}
fviz_nbclust(datTest, FUN = hcut, method = "wss")
fviz_nbclust(datTest, FUN = hcut, method = "silhouette")
gap_stat = clusGap(datTest, FUN = hcut, nstart = 25, K.max = 10, B = 10)
fviz_gap_stat(gap_stat)
```

