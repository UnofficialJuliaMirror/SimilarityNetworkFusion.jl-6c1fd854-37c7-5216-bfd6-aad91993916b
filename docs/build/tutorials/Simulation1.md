# SNFtool Simulation tutorial
 Here we will go through an example for running SNFtool on a simulated data.
 The data was generated by paper's authors and the details appears in the
 supplementary data, look for the second simulation. In short, there are two
 data sets representing 200 samples from two classes. In the first data set
 the first class have a Normal noise added, and the second class a Gamma
 distributed noise. In the second data set the opposite.

 This tutorial was adopted from the Readme of the SNFtool R package on github:
 https://github.com/cran/SNFtool

 ```julia
## Install the package
Pkg.add("SNFtool")
using SNFtool

## First, set all the parameters:
K = 20;		# number of neighbors, usually (10~30)
alpha = 0.5;  	# hyperparameter, usually (0.3~0.8)
T = 10; 	# Number of Iterations, usually (10~20)

## Data1 is of size n x d_1, where n is the number of patients, d_1 is the number of genes, e.g.
## Data2 is of size n x d_2, where n is the number of patients, d_2 is the number of methylation, e.g.
Data1, Data2 = dataset("Simulation1")

## Here, the simulation data (Data1, Data2) has two data types. They are complementary to each other. And two data types have the same number of points. The first half data belongs to the first cluster; the rest belongs to the second cluster.

# The ground truth of the simulated data`
truelabels = vcat(repeat([1]; outer=[100]),
                  repeat([2]; outer=[100]));


## Calculate distance matrices(here we calculate Euclidean Distance, you can use other distance, e.g,correlation)

## If the data are all continuous values, we recommend the users to perform standard normalization before using SNF, though it is optional depending on the data the users want to use.  
Data1 = sapply_df(Data1,standardize)
Data2 = sapply_df(Data2,standardize)

## Calculate the pair-wise distance; If the data is continuous, we recommend to use the function "dist2" as follows; if the data is discrete, we recommend the users to use ???""
Dist1 = dist2(Data1)
Dist2 = dist2(Data2)

## next, construct similarity graphs
W1 = affinityMatrix(Dist1, K, alpha)
W2 = affinityMatrix(Dist2, K, alpha)

## These similarity graphs have complementary information about clusters.
p1 = displayClusters(W1,truelabels);
p2 = displayClusters(W2,truelabels);
Gadfly.hstack(p1, p2)

## next, we fuse all the graphs
## then the overall matrix can be computed by similarity network fusion(SNF):
W = SNF([W1,W2], K, T)

## With this unified graph W of size n x n, you can do either spectral clustering or Kernel NMF. If you need help with further clustering, please let us know.
## for example, spectral clustering

C = 2 					# number of clusters
group = spectralClustering(W, C); 	# the final subtypes information

## you can evaluate the goodness of the obtained clustering results by calculate Normalized mutual information (NMI): if NMI is close to 1, it indicates that the obtained clustering is very close to the "true" cluster information; if NMI is close to 0, it indicates the obtained clustering is not similar to the "true" cluster information.

displayClusters(W, group)
SNFNMI = calNMI(group, truelabel)

## you can also find the concordance between each individual network and the fused network

#ConcordanceMatrix = concordanceNetworkNMI(list(W, W1,W2));


```@docs
SNFtool.dataset
SNTtoo.SNF
```