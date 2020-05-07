---
description: >-
  Searching for, and identifying important events, deltas and crucial time
  points.
---

# Analyzing the Event Log

Now that we have entered, modified and deleted some data in our fictional HRMS, we will explore the history of these operations using the [event log](../../../understanding-recallgraph/terminology/#event-log).

RecallGraph's Event API supports 2 endpoints:

1. A `LOG` endpoint to trace the sequence of [events](../../../understanding-recallgraph/terminology/#event) within a specified [scope](../../../understanding-recallgraph/terminology/#scopes), with support for [grouping](../../../understanding-recallgraph/terminology/grouping.md), [sorting](../../../understanding-recallgraph/terminology/sorting.md), [slicing](../../../understanding-recallgraph/terminology/slicing.md) and [post-filtering](../../../understanding-recallgraph/terminology/post-filters.md).
2. A `DIFF` endpoint to extract [commands](../../../understanding-recallgraph/terminology/#command) between successive events, again within the confines of a scope, and supporting sorting, slicing and post-filtering \(but not grouping\).

We will explore these endpoints in the sections linked below:

{% page-ref page="log.md" %}

{% page-ref page="diff.md" %}

