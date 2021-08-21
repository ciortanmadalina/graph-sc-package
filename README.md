# Graph-sc

This repository contains the pytorch implementation of the paper "Clustering scRNA-seq data with graph neural networks", by Madalina Ciortan under the supervision of Matthieu Defrance.

We propose graph-sc, a method modeling scRNA-seq data as a graph, processed with a graph autoencoder network to create representations (embeddings) for each cell. The resulting embeddings are clustered with a general clustering algorithm (i.e. KMeans, Leiden) to produce cell class assignments.
An extensive experimental study was performed on 24 simulated and 15 real-world scRNA-seq datasets. graph-sc was compared with 11 competing state-of-the-art techniques on 4 clustering scores, reflecting both the external and the internal clustering performance. The results indicate that although there is no consistently best method across all analyzed datasets, graph-sc compared favorably with the competing techniques across all types of datasets. A large ablation study evaluates numerous strategies to create the input graph, the graph autoencoder network and also the clustering phase. The proposed method is stable across consecutive runs, robust to input down-sampling, generally insensitive to changes in the network architecture or training parameters and more computationally efficient than other competing methods based on neural networks. Moreover, modeling the data as a graph provides an increased flexibility to define custom features characterizing the genes, the cells and their interactions as well as the possibility to enrich the graph with external data (i.e. gene correlations).

## Installation

The package requires python >= 3.6 and can be installed by running:

```
pip install graph-sc
```

## Tutorial

First import required libraries:
```
import h5py
import matplotlib.pyplot as plt
import numpy as np
import pkg_resources
import graph_sc.models as models
import graph_sc.train as train

device = train.get_device(use_cpu=True)
print(f"Running on device: {device}")
```

Then load the data to be analyzed. The package provides an example dataset:

```
DATA_PATH = pkg_resources.resource_filename("graph_sc", "data/")
data_mat = h5py.File(f"{DATA_PATH}/worm_neuron_cell.h5", "r")

X = np.array(data_mat["X"])

Y = np.array(data_mat["Y"]) # this is optional
n_clusters = len(np.unique(Y)) # this is required for KMeans
```

Run the model training:

```
scores = train.fit(X, Y, n_clusters, cluster_methods=["KMeans"])
print(scores)
```

Get the resulting embedding:

```
embeddings = scores["features"]
```

This embedding can be used as input to any other downstream task.



Plot the latent space and the underlying prediction:

```
pca = PCA(2).fit_transform(embeddings)
plt.figure(figsize=(12, 4))
plt.subplot(121)
plt.title("Ground truth")
plt.scatter(pca[:, 0], pca[:, 1], c=Y, s=4)

plt.subplot(122)
plt.title("K-Means pred")
plt.scatter(pca[:, 0], pca[:, 1], c=scores["kmeans_pred"], s=4)
```