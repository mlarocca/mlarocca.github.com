---
layout: post
title: Karger Randomized Contraction algorithm for finding Minimum Cut in undirected Graphs
---

[Karger's algorithm](http://en.wikipedia.org/wiki/Karger's_algorithm) is a randomized algorithm to compute a minimum cut of a connected graph. It was invented by [David Karger](http://en.wikipedia.org/wiki/David_Karger) and first published in 1993.

A [cut](http://en.wikipedia.org/wiki/Cut_(graph_theory)) is a set of edges that, if removed, would disconnect the Graph; a [minimum cut](http://en.wikipedia.org/wiki/Minimum_cut) is the smallest possible set of edges that, when removed, produce a disconnected Graph.
Every minimum cut corresponds to a partitioning of the Graph vertices into two non-empty subsets, such that the edges in the cut only have their endpoints in the two different subsets. 

Karger algorithm builds a cut of the graph by randomly creating this partitions, and in particular by choosing at each iteration a random edge, and contracting the graph around it: basically, merging its two endpoints in a single vertex, and updating the remaining edges, such that the self-loops introduced (like the chosen edge itself) are removed from the new Graph, and storing parallel-edges (if the algorithm chooses an edge (_u_,_v_) and both _u_ and _v_ have edges to a third vertex _w_, then the new Graph will have two edges between the new vertex _z_ and _w_)
After _n-2_ iterations, only two macro-vertex will be left, and the parallel edges between them will form the cut.

The algorithm is a Montecarlo algorithm, i.e. its running time is deterministic, but it isn't guaranteed that at every iteration the best solution will be found.

Actually the probability of finding the minimum cut in one run of the algorithm is pretty low, with an upper bound of 1/(_n_ ^ 2), where _n_ is the number of vertices in the Graph. Nonetheless, by running the algorithm multiple times and storing the best result found, the probability that none of the runs founds the minimum cut becomes very small: 1/_e_ (Neper) for _n_ squared runs, and 1/_n_ for _n_ ^2 * log(_n_) runs - for large values of _n_, i.e. for large Graphs, that's a negligible probability.

The [implementation](https://github.com/mlarocca/Algorithms/tree/master/karger) provided is written in Python, assumes the Graph represented with adjacency list (as a Dictionary) and is restricted to having only integer vertices labels (ideally the number from 0 to n-1): this limitation allows to exploit the _union-find_ implementation provided, and can be easily overcome by mapping the original labels to the range [0..n-1].