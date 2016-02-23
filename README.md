# Part 1: PCA & K-Means with Airplane Data

### Step 1: Set Working Directory and Load Delta CSV File
The data is from Delta's website. It is saved locally into the "Downloads" folder. The code below sets up the working directory and loads the csv file into R.

```r
# Delta Airlines Fleet analysis
# Originally by Myles Harrison
# http://www.everydayanalytics.ca
# Data from Delta.com:
# http://www.delta.com/content/www/en_US/traveling-with-us/airports-and-aircraft/Aircraft.html

setwd("~/Downloads/")
data <- read.csv(file="Delta.csv", header=T, sep=",", row.names=1)
```
### Step 2: Scatterplot and Naive Principal Components
First, I explore the data by creating a scatterplot and running (naive) principal components.


```r
# scatterplot matrix of intermediary (size/non-categorical) variables
plot(data[,16:22])

# Naively apply principal components analysis to raw data and plot
pc <- princomp(data)
plot(pc)
```
#### Figure 1: Variable Scatterplot

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044048/ab4994f2-86ed-11e5-9471-2fdf4f9d3d94.png)

#### Figure 2: Naive Principal Component Variances

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044060/c0ce5646-86ed-11e5-8c1d-c3f315392ce9.png)

From Figure 1 & 2, I notice three observations:

1. Most variables are numeric, except for "Engines" which appears to be a binary variable.
2. There is positive correlation between Airline "Tail Height", "Wingspan" and "Length". This makes sense as the bigger an aircraft, the larger (in proportion) I would expect the airplane's components.
3. From Figure 2, almost all of the variance in the (naive) PCA is contained in the first principal component. We'll see in the next step which variables are primarily contributing to the 1st principal component.  


### Step 3: Examine Naive Principal Components through Plotting Column Variance

Plot the variances of each principal components.

```r
# First component dominates greatly. What are the loadings?
summary(pc) # 1 component has > 99% variance
loadings(pc) # Can see all variance is in the range in miles 

# verify by plotting variance of columns
mar <- par()$mar
par(mar=mar+c(0,5,0,0))
barplot(sapply(data, var), horiz=T, las=1, cex.names=0.8)
barplot(sapply(data, var), horiz=T, las=1, cex.names=0.8, log='x')
par(mar=mar)
```
#### Figure 3: Variance of Principal Components
 
![alt text](https://cloud.githubusercontent.com/assets/7621432/11185163/81a76df2-8c49-11e5-8040-6c4e521f493f.png)

From Figure 3, "Range (miles)" is playing a major role in the first principal component. This result is peculiar but makes sense. For example, the range of the aircraft is the only metric measured in miles. Most of the other metrics are measured on different scales (e.g. feet for aircraft features or binary flags like Business Class). Therefore, this variance is because the range variable is on a different scale than the other variables and should be normalized (along with the other variables) to put all the variables on the same scale.

#### Figure 4: Variance of Principal Components (Log X)

![alt text](https://cloud.githubusercontent.com/assets/7621432/11185245/e70db1ec-8c49-11e5-84cb-3874148a12ff.png)

However, in Figure 4, we standardize all the variables (by taking a log) to "normalize" the variables taking the log. After doing this, we see that the range variable variance is not far outstripping the other variables. Range still has the highest variance, but not as large relative to the other variables.

### Step 4: Updated Principal Component Analysis
In this step, we "scale" (normalize) the variables to ensure that the variables are all on the same scale. 

```r
# Scale
data2 <- data.frame(scale(data))
# Verify variance is uniform
plot(sapply(data2, var))
```
#### Figure 5: Variance of Principal Components after Normalizing

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044072/cd0796d4-86ed-11e5-9b1a-abfbcea0ae44.png)

Figure 5 below shows that the effect after "scaling" the data, i.e. they all have the same (unit) variance.
```r
# Proceed with principal components
pc <- princomp(data2)
plot(pc)
plot(pc, type='l')
summary(pc) # 4 components is both 'elbow' and explains >85% variance

                          PC1    PC2    PC3     PC4     PC5     PC6     PC7    PC8     PC9    PC10
Standard deviation     3.6822 2.8206 2.2482 1.22458 1.15392 1.02668 0.88837 0.7129 0.59165 0.53520
Proportion of Variance 0.4109 0.2411 0.1532 0.04544 0.04035 0.03194 0.02392 0.0154 0.01061 0.00868
Cumulative Proportion  0.4109 0.6520 0.8051 0.85057 0.89092 0.92286 0.94678 0.9622 0.97279 0.98147
                          PC11    PC12    PC13    PC14    PC15    PC16    PC17   PC18    PC19    PC20
Standard deviation     0.41258 0.36636 0.30228 0.24134 0.20039 0.17779 0.14607 0.1402 0.11869 0.10572
Proportion of Variance 0.00516 0.00407 0.00277 0.00177 0.00122 0.00096 0.00065 0.0006 0.00043 0.00034
Cumulative Proportion  0.98662 0.99069 0.99346 0.99523 0.99644 0.99740 0.99805 0.9986 0.99907 0.99941
                          PC21    PC22    PC23    PC24    PC25    PC26    PC27     PC28     PC29
Standard deviation     0.08635 0.08003 0.05238 0.04521 0.02237 0.01548 0.01093 0.003661 1.89e-15
Proportion of Variance 0.00023 0.00019 0.00008 0.00006 0.00002 0.00001 0.00000 0.000000 0.00e+00
Cumulative Proportion  0.99963 0.99983 0.99991 0.99997 0.99999 1.00000 1.00000 1.000000 1.00e+00
                            PC30      PC31      PC32      PC33
Standard deviation     1.067e-15 3.182e-16 3.182e-16 3.182e-16
Proportion of Variance 0.000e+00 0.000e+00 0.000e+00 0.000e+00
Cumulative Proportion  1.000e+00 1.000e+00 1.000e+00 1.000e+00
```

The output above shows for each principal component, how much variance is explained by that PC, listed in order from the PC with the highest variance (PC1) to the lowest (PC33). In addition, the cumulative proportion includes the variances from highest to each PC. For example, PC4 explains 0.04544 of the variance but, including the first three PC's, the cumulative variances for the first four principal components is around .85. Said differently, the first four principal components explain around 85% of the variance. 

#### Figure 6: Variance Plot for PCA

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044075/d2c34e2e-86ed-11e5-9ca2-566ff91b0177.png)

Figure 6 plots the variance by each principal component. For example, By looking at this plot, we find that after the fourth principal component, there is little "marginal" variance explained by including additional principal components (i.e. the elbow of the chart). This is consistent with our observations from the output above.

### Step 5: Rerun Principal Component Analysis

```r
# Get principal component vectors using prcomp instead of princomp
pc <- prcomp(data2)

# First four principal components
comp <- data.frame(pc$x[,1:4])
# Plot
plot(comp, pch=16, col=rgb(0,0,0,0.5))
```
Figure 7 shows the scatterplot for first four principal components.

#### Figure 7: First Four Principal Components Scatterplot

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044079/d7887f4c-86ed-11e5-8b95-d4e34436222f.png)

```r
install.packages("rgl")
library(rgl)
# Multi 3D plot
plot3d(comp$PC1, comp$PC2, comp$PC3)
plot3d(comp$PC1, comp$PC3, comp$PC4)
```
The two figures below explore the scatterplots of the principal components in 3D space. From Figure 8, it appears that there are four clusters of points. In particular, notice the "outlier" cluster of one point that is near the "bottom-right" of the 3D plot (high PC1, low PC2 and PC3).

Figure 9 replaces PC3 with PC4. In this plot, several of the clusters seem to break up but there does appear to be more clusters of points.

#### Figure 8: 3D Scatterplot for PC1, PC2 and PC3

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044082/dc1ca72c-86ed-11e5-8398-e875b310d2a5.png)

#### Figure 9: 3D Scatterplot for PC1, PC2 and PC4

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044091/e0d74376-86ed-11e5-9502-1ef4ec2b7e95.png)

### Step 6: K-Means to Determine K (Number of Clusters)

As noticed in the last step, there appears to be "clusters" of points in the PCA space. In this step, I determine the correct number of clusters via weighted sum of squares. This process is from [R in Action](http://www.statmethods.net/advstats/cluster.html).

The process for this step is:

1. Run for loop to consider up to 15 clusters
2. Determine the "within-cluster" sum of squares to determine number of clusters to use.

```r
wss <- (nrow(comp)-1)*sum(apply(comp,2,var))
for (i in 2:15) wss[i] <- sum(kmeans(comp, centers=i, nstart=100, iter.max=1000)$withinss)
plot(1:15, wss, type="b", xlab="Number of Clusters",
     ylab="Within groups sum of squares")
```
#### Figure 10: Within Group Sum of Squares by Clusters

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044097/e6e14adc-86ed-11e5-82be-5fdfb364ed56.png)

From Figure 10, we can use the "elbow" rule to determine that k should equal four. 

```r
# From scree plot elbow occurs at k = 4
# Apply k-means with k=4
k <- kmeans(comp, 4, nstart=25, iter.max=1000)
library(RColorBrewer)
library(scales)
palette(alpha(brewer.pal(9,'Set1'), 0.5))
plot(comp, col=k$clust, pch=16)
```
#### Figure 11: PC Scatterplot of First Four Principal Components

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044104/ebb31afe-86ed-11e5-9358-46661b9dbc9a.png)

Figure 11 shows the scatterplot for the first four principal components but now coloring the points based on the k-means clusters when k=4. As expected, the colors correspond to clusters in the data. 

```r
# 3D plot
plot3d(comp$PC1, comp$PC2, comp$PC3, col=k$clust)
plot3d(comp$PC1, comp$PC3, comp$PC4, col=k$clust)
```
#### Figure 12: 3D Scatterplot for PC1, PC2 and PC3 with K=4 Clusters

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044112/f112f8fc-86ed-11e5-9fa3-d70543ec4ef1.png)

#### Figure 13: 3D Scatterplot for PC1, PC2 and PC4 with K=4 Clusters

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044114/f624a2d2-86ed-11e5-86ce-72e2d8b6ab9a.png)

The plots above now add in the new K=4 clusters found in the k-means algorithm. Not surprising, they are similar to those identified from Figure 8 in the previous step. 

### Step 7: Cluster Sizes

```r
# Cluster sizes
sort(table(k$clust))
clust <- names(sort(table(k$clust)))

##I changed the code so that the First Cluster = k$clust == 1, not row.names(data[k$clust==clust[1],]) 

# First cluster
row.names(data[k$clust==1,])
# Second Cluster
row.names(data[k$clust==2,])
# Third Cluster
row.names(data[k$clust==3,])
# Fourth Cluster
row.names(data[k$clust==4,])

# Cluster 1
> row.names(data[k$clust==1,])
 [1] "Airbus A319"            "Airbus A320"            "Airbus A320 32-R"      
 [4] "Boeing 717"             "Boeing 737-700 (73W)"   "Boeing 737-800 (738)"  
 [7] "Boeing 737-800 (73H)"   "Boeing 737-900ER (739)" "Boeing 757-200 (75A)"  
[10] "Boeing 757-200 (75M)"   "Boeing 757-200 (75N)"   "Boeing 757-200 (757)"  
[13] "Boeing 757-200 (75V)"   "Boeing 757-300"         "Boeing 767-300 (76P)"  
[16] "Boeing 767-300 (76Q)"   "Boeing 767-300 (76U)"   "CRJ 700"               
[19] "CRJ 900"                "E170"                   "E175"                  
[22] "MD-88"                  "MD-90"                  "MD-DC9-50"             
> # Second Cluster
> row.names(data[k$clust==2,])
[1] "Airbus A319 VIP"
> # Third Cluster
> row.names(data[k$clust==3,])
[1] "CRJ 100/200 Pinnacle/SkyWest" "CRJ 100/200 ExpressJet"       "E120"                        
[4] "ERJ-145"                     
> # Fourth Cluster
> row.names(data[k$clust==4,])
 [1] "Airbus A330-200"          "Airbus A330-200 (3L2)"    "Airbus A330-200 (3L3)"   
 [4] "Airbus A330-300"          "Boeing 747-400 (74S)"     "Boeing 757-200 (75E)"    
 [7] "Boeing 757-200 (75X)"     "Boeing 767-300 (76G)"     "Boeing 767-300 (76L)"    
[10] "Boeing 767-300 (76T)"     "Boeing 767-300 (76Z V.1)" "Boeing 767-300 (76Z V.2)"
[13] "Boeing 767-400 (76D)"     "Boeing 777-200ER"         "Boeing 777-200LR" 
```
For the four clusters:
1. This is the largest cluster (24 planes).
2. There is one plane. It is unique because it is a "VIP" aircraft.
3. There are four planes in this cluster.
4. There are fifteen planes in this cluster.

```r
# Compare accommodation by cluster in boxplot
boxplot(data$Accommodation ~ k$cluster, 
        xlab='Cluster', ylab='Accommodation', 
        main='Plane Accommodation by Cluster')
```
#### Figure 14: Box-Plot of Plane Accomodation (Seats) by K-Means Cluster

![alt text](https://cloud.githubusercontent.com/assets/7621432/11044119/fb75b5c8-86ed-11e5-859d-8f5f8693d5c5.png)

A key difference is based on the size of seating ("accommodation"). "Accomodation" is shown in Figure 14. As shown in a box-plot, Cluster 4 has (on average) the most number of seats and correspond to larger planes (average 250 seats but range from about 175 to 375 seats). Whereas Clusters 2 and 3 on average have much fewer number of seats. In fact, Cluster 2 only consists of one plane, hence why its box plot is simply a line. Cluster 1 is made up of planes that on average have about 150 seats but range from about 60 to 250 seats.

```r
# Compare presence of seat classes in largest clusters
> data[k$clust==4,30:33]
                         First.Class Business Eco.Comfort Economy
Airbus A330-200                    0        1           1       1
Airbus A330-200 (3L2)              0        1           1       1
Airbus A330-200 (3L3)              0        1           1       1
Airbus A330-300                    0        1           1       1
Boeing 747-400 (74S)               0        1           1       1
Boeing 757-200 (75E)               0        1           1       1
Boeing 757-200 (75X)               0        1           1       1
Boeing 767-300 (76G)               0        1           1       1
Boeing 767-300 (76L)               0        1           1       1
Boeing 767-300 (76T)               0        1           1       1
Boeing 767-300 (76Z V.1)           0        1           1       1
Boeing 767-300 (76Z V.2)           0        1           1       1
Boeing 767-400 (76D)               0        1           1       1
Boeing 777-200ER                   0        1           1       1
Boeing 777-200LR                   0        1           1       1
> data[k$clust==1,30:33]
                       First.Class Business Eco.Comfort Economy
Airbus A319                      1        0           1       1
Airbus A320                      1        0           1       1
Airbus A320 32-R                 1        0           1       1
Boeing 717                       1        0           1       1
Boeing 737-700 (73W)             1        0           1       1
Boeing 737-800 (738)             1        0           1       1
Boeing 737-800 (73H)             1        0           1       1
Boeing 737-900ER (739)           1        0           1       1
Boeing 757-200 (75A)             1        0           1       1
Boeing 757-200 (75M)             1        0           1       1
Boeing 757-200 (75N)             1        0           1       1
Boeing 757-200 (757)             1        0           1       1
Boeing 757-200 (75V)             1        0           1       1
Boeing 757-300                   1        0           1       1
Boeing 767-300 (76P)             1        0           1       1
Boeing 767-300 (76Q)             1        0           1       1
Boeing 767-300 (76U)             0        1           1       1
CRJ 700                          1        0           1       1
CRJ 900                          1        0           1       1
E170                             1        0           1       1
E175                             1        0           1       1
MD-88                            1        0           1       1
MD-90                            1        0           1       1
MD-DC9-50                        1        0           1       1
```
A major difference between clusters 1 and 4 is seating arrangement. For example, cluster 4 does not have first class. On the other hand, cluster 1 has first class but not business class.

The previous two analyses help to create "labels" for each cluster.

## Four Cluster Labels:
1. "Commercial Planes with First Class"
2. "VIP Plane"
3. "Small Planes"
4. "Commercial Planes with Business Class"


## Major Takeaways from PCA Analysis

1. Need to standardization; avoid "naive" PCA, i.e. PCA w/o standardization.
2. There are several ways (e.g. scatterplots) to determine which variables are driving each principal component.
3. Use the Elbow rule to determine which principal components to use.
4. Use K-Means (and for loop) to determine appropriate number of clusters.
5. Use profile (e.g. domain expertise or attributes like First Class vs Business Class) to determine the cluster "labels".


# Part 2: Principal Components Component with Wine Data

## PCA Red

### Step 1: PCA for Red Wine

Read in the file and run PCA (scaling) on the data.

```r
data_file = "winequality-red.csv"

wine <- read.csv( data_file, sep=';', header = TRUE )

# number of elements
numel = length( as.matrix( wine )) / length( wine )

# pca analysis
pcx <- prcomp( wine, scale = TRUE )
biplot( pcx, xlabs = rep( '.', numel ))
```

#### Figure 15: PCA Plot (Red Wine) with variable vectors
![alt text](https://cloud.githubusercontent.com/assets/7621432/11044412/c51b6b42-86ef-11e5-95c5-a1b1eee19a9b.png)

Figure 15 is a Scatterplot of the first two principal components for the Red Wine Dataset. In addition, it overlays the vectors that correspond to each variable. This helps to explain the variance in each principal component. For example, a high PC1 corresponds to a high alcohol content while a low PC1 value corresponds to higher density and residual sugar levels.

On the other hand, a high PC2 corresponds to high levels of acidity (fixed, citric) while low levels of PC2 correspond to high pH (i.e. more basic, less acidic). 

Therefore, PC1 is primarily concerned with the alcohol content of the wine and PC2 corresponds with the level of acidity (pH) in the wine.

```r
# another window
dev.new()

# principal components
bar_colors = c( 'red', 'red', rep( 'gray', 10 ))
plot( pcx, col = bar_colors )
```
#### Figure 16: PCA Plot (Red Wine) with variances by Principal Components
![alt text](https://cloud.githubusercontent.com/assets/7621432/11044418/cbdac040-86ef-11e5-9e3c-5e39fe9625fc.png)

Figure 16 shows the variance associated with each principal component. Highlighted we see that the first two principal compnents make up a significant proportion of variance. In fact, we do not see the "elbow" until around the sixth PC; therefore, for further analysis, we may want to include additional principal components.

## PCA White

### Step 2: PCA for White Wine

Rerun PCA on the White Wine dataset.

```r
data_file = "winequality-white.csv"

wine <- read.csv( data_file, sep=';', header = TRUE )

numel = length( as.matrix( wine )) / length( wine )

pcx <- prcomp( wine, scale = TRUE )
biplot( pcx, xlabs = rep( '.', numel ))
```

#### Figure 17: PCA Plot with variable vectors
![alt text](https://cloud.githubusercontent.com/assets/7621432/11044421/d18e6c08-86ef-11e5-8efe-321e2ec2ba11.png)

We can intepret Figure 17 to help interpret what factors are driving each of the first two principal components. For example, PC1 is correlated with the level of alcohol (high PC1 corresponds to high alcohol content). Similarly, a high PC2 denotes a higher acidity and a lower pH level. This is similar to Red Wine found above.

```r
# another window
dev.new()

bar_colors = c( 'red', 'red', rep( 'gray', 10 ))
plot( pcx, col = bar_colors )
```
#### Figure 18: PCA Plot (White) with variances by Principal Components
![alt text](https://cloud.githubusercontent.com/assets/7621432/11044423/d7532638-86ef-11e5-91fc-42ee2c985f85.png)

Figure 18 shows the variance associated with each principal component. Highlighted in red, we see that the first principal components significantly make up a lot of the variance. However unlike the Red Wine dataset, there is a big drop from the 1st to the 2nd principal component and the variance by principal component begins to level off after the 2nd principal component. Therefore, for White Wine, the 1st principal component is much more "important" given that it has a higher % of variance explained than the 1st principal component for Red Wine. Said differently, alcohol (PC1) explains more variance for White Wine than Red Wine.
