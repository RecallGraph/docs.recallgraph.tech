---
description: >-
  Get historical states of individual vertices/edges, and run traversals and
  shortest path queries on past versions of a graph.
---

# Navigating History

In this section, we will see how to look back at historical states of nodes and edges, and run traversals and shortest-path queries on them. As with the previous exercises, we will use the HRMS example as the backdrop in which we keep building the narrative.

RecallGraph's History API supports 3 endpoints:

1. A `SHOW` endpoint to rebuild previous states of nodes at specified points in time within a specified [scope](../../../understanding-recallgraph/terminology/#scopes), with support for [grouping](../../../understanding-recallgraph/terminology/grouping.md), [sorting](../../../understanding-recallgraph/terminology/sorting.md), [slicing](../../../understanding-recallgraph/terminology/slicing.md) and [post-filtering](../../../understanding-recallgraph/terminology/post-filters.md).
2. A `TRAVERSE` endpoint to run [AQL](https://www.arangodb.com/docs/stable/aql/index.html)-like [traversals](https://www.arangodb.com/docs/stable/aql/graphs-traversals-explained.html) over historic data, with the option to post-filter on vertex and edge sets.
3. A `k SHORTEST PATHS` endpoint to run [weighted shortest path](https://www.arangodb.com/docs/stable/aql/graphs-kshortest-paths.html) queries over historic versions of the graph.

We will explore these endpoints in the sections linked below:

{% page-ref page="show.md" %}

{% page-ref page="traverse.md" %}

{% page-ref page="k-shortest-paths.md" %}

