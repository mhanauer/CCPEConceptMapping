---
title: "Hierarchical Clustering for Concept Mapping in R"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Here I am providing an example of how to analyze concept mapping data in R.  This tutorial assumes that you have already collected and summed the data.  The data set that is presented is in a final version where the number of times each person placed each item (i.e. suggestion based on the prompt) into the same pile (e.g. theme, category) is summed.

Much of this example is based on the University of Cincinnati's R Programming Guide for Hierarchical Cluster Analysis: https://uc-r.github.io/hc_clustering.

Below are some packages that will be necessary for this tutorial. 
```{r}
library(MASS)
library(cluster)
library(factoextra)
library(ggpubr)
library(tidyverse)
library(cluster)
```
The artificial data set that we have included fifteen items summed across 15 people in a concept mapping brainstorming session.  The structure of the dataset is similar to a correlation matrix where each side of the diagonals (zero in this case) are a mirror of each other.  10 is placed into the diagonals because each item is always placed into the same category by each person; therefore the diagonals will always equal the number of people who sorted the responses.
```{r}
setwd("~/Desktop")
datTest = read.csv("datCorMD.csv", header = TRUE)
samp = c(1:5)
ratings = sample(samp, 15, replace = TRUE)
importance = sample(samp, 15, replace = TRUE)
```
Next, we use Agglomerative clustering using the agnes function in the cluster package.  AGNES starts by placing each item into its cluster and then finds items that are most similar and combine them into similar clusters.  This process is continued until all items are placed into one cluster.  Because in this example, I am using the original data set and not a dissimilatory matrix as the data set, I set diss to FALSE.  I then standardize the data, because AGNES automatically reduces the variables to two allowing us to plot the data later on x and y coordinates.  The multicollinearity that can be present and thus decrease the accuracy of the data reduction process can be reduced by scaling the variables (i.e. turning them into z-scores).  Then the method of partitioning into clusters is selected.  Ward's method attempts to minimize the total within-cluster variance.  For information about other methods see: https://uc-r.github.io/hc_clustering  
```{r}
hcWard = agnes(datTest, diss = FALSE, method = "ward")
hcC = agnes(datTest, diss = FALSE, method = "complete")
hcWeighted = agnes(datTestDaisy, diss = FALSE, method = "weighted")
```
A bonus of hieratical clustering in the agnes function is that it produces an agglomerative coefficient, which provides an indication of model fit where a coefficient closer to one indicates a better fit.  Below we show that Ward's is the best fit relative to the complete, single, and weighted methods because it has the highest agglomerative coefficient.
```{r}
hcWard$ac
hcC$ac
hcWeighted$ac
```
Now that we have the data sorted into two dimensions using, hierarchical clustering, we can them group the items into different clusters.  I will start by selecting four clusters; however, we will evaluate this decision in the next step.  

Then so we can see how each item is placed into which cluster, we use the mutate function to combine the clusters identification variable with the items.
```{r}
hcTree = cutree(hcWard, k=3)
datTest$cluster = hcTree
datTest$ratings = ratings
datTest$importance = importance
datTest
```
Now we want to figure out how many clusters is the best fit for the data given the specified method for partitioning.  Three common methods are:

Elbow method = visually inspecting where the elbow is on the graph of variance accounted for 

Average silhouette method = measures how well each object or item lies within a cluster of different numbers of clusters

Gap Cluster = This is a comparison of the intracluster variation to what we would expect with a simulated data set with no inherent clustering structure given different amounts of clusters.

Unfortunately, our results are presenting different answers.  This is to be expected since I randomly generated the data.  We will stick with three for the sake of the tutorial and move on to plotting.
```{r}
fviz_nbclust(datTest, FUN = hcut, method = "wss")
fviz_nbclust(datTest, FUN = hcut, method = "silhouette")
gap_stat = clusGap(datTest, FUN = hcut, nstart = 25, K.max = 10, B = 10)
fviz_gap_stat(gap_stat)
```
Finally, we can plot data onto a two-dimensional map where the clusters are linked together with lines and highlighted different colors. 
```{r}
conceptMap = fviz_cluster(list(data =datTest, cluster = hcTree, repel = TRUE))
conceptMap
```
Keep trying to figure out how to add layers
```{r}
fviz_cluster(list(data =datTest, cluster = hcTree, repel = TRUE)) +
  layer(geom = "point", data = datTest, stat = "ratings", position = "ratings")
```
Try creating the go no go zones with thinking about the relationshipb between feasibility and importance.  Try just getting the subset of items with 3's on both importance and feasibility

Let us assume that feasability and importance are averages across items where row one indicates the rating and importance for item one and so on. 
```{r}
datFeasImport = data.frame(ratings, importance)
datFeasImport 
datFeasImport = subset(datFeasImport, ratings > 2 & importance > 2)
head(datFeasImport)
```

Explaination
---
title: "Bayes Loop Results"
output:
  pdf_document: default
  html_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Library the packages
```{r}
library(MCMCpack)
library(descr)
library(ggplot2)
library(psych)

library(MASS)
library(cluster)
library(factoextra)
library(ggpubr)
library(tidyverse)
```
Now load the data
We get the distance by taking using this formula: http://rosalind.info/glossary/euclidean-distance/

sqrt(x2-x1)^2+(y2-y1)^2...+(z2-z1)^2) for each variable in the data set.  Then for column in the distance matrix the difference between two and three for each variable and so on. 
```{r}
setwd("~/Desktop")
datTest = read.csv("datCorMD.csv", header = TRUE)
head(datTest)
get_dist(datTest)

datTest2 = datTest[,1:2]
get_dist(datTest2)

sqrt((4-10)^2+(10-4)^2)

sqrt((5-4)^2+(9-10)^2)


datTest3 = datTest[,1:3]
get_dist(datTest3)
sqrt((4-10)^2+(10-4)^2+(9-5)^2)

```







