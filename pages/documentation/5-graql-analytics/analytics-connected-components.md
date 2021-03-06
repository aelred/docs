---
title: Connected Components
last_updated: December 12, 2016
tags: [analytics]
summary: "This page introduces the connected components algorithm and explains how to use it."
sidebar: documentation_sidebar
permalink: /documentation/graql-analytics/analytics-connected-components.html
folder: documentation
comment_issue_id: 71
---

The connected components algorithm can be used to find clusters of instances in the graph that are connected.
The algorithm finds all instances (relations, resources and entities) that are connected via relations in the graph and gives each set a unique label.
In the graph below you can see three connected components that correspond to groups of friends.
In this graph three unique labels will be created one corresponding to each of the sets of connected instances.

 ![Three connected components representing groups of friends.](/images/analytics_conn_comp.png)

You can call the cluster algorithm to find the clusters above using:

```
compute cluster in person, knows;
```

The results you would actually get involve 3 clusters with sizes: 5, 7, 3.
If you want to see the actual members of the clusters you have to use the modifier `members`.

```
compute cluster in person, knows; members;
```

Here, the [subgraph](./analytics-overview.html) functionality has been used to get more meaningful results, because executing the cluster algorithm without specifying a subgraph will not result in meaningful information.


### Persisting the Connected Components

When the clusters have meaning, it is useful to be able to persist it to the graph, so it is available for querying.

For this reason we have introduced `persist`, which is [documented further on a separate page](./analytics-persist.html). 

To persist the cluster to the graph we call:

```
compute cluster in person, knows; members; persist;
```

{% include links.html %}

## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/71" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of this page.


