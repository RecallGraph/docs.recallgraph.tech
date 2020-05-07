---
description: >-
  Fetch a log of events for a given scope, optionally
  grouped/sorted/sliced/post-filtered.
---

# Log

{% hint style="info" %}
Before proceeding with this section of the guide, make sure you have understood the concepts of [patterns](../../../understanding-recallgraph/terminology/#patterns), [scopes](../../../understanding-recallgraph/terminology/#scopes) and [paths](../../../understanding-recallgraph/terminology/#path), explained in the [terminology](../../../understanding-recallgraph/terminology/) page.
{% endhint %}

{% hint style="info" %}
The `LOG` endpoint is a very powerful and flexible querying tool with lots of option combinations, all of which cannot possibly be explored in this guide. However, hopefully, this guide will cover enough ground to make readers comfortable in exploring the rest of the options by themselves.
{% endhint %}

## All Events

### Overall Count

We don't know which events are of interest to us yet, so we begin with an ad hoc, exploratory analysis to get an idea of the number and types of events in the event log. We will use the `GET` variant here. The request parameters are shown below:

| Param | Value |
| :--- | :--- |
| countsOnly | true |
| path | / |

The result obtained will have a structure as shown below. Your actual numbers may vary. We will shortly see why.

```text
[
  {
    "total": 27
  }
]
```

This tells us that there have been a total of 27 events recorded in the system \(in the instance from which this result was pulled\). That looks a bit strange since we have not performed 27 write operations so far in our walkthrough.

### Counts by Type

The root cause behind this anomaly cannot be determined from this singular number. Therefore, we decide to drill down a bit by using a more fine grained grouping clause. We split event counts by event type by using the parameters given below:

| Param | Value |
| :--- | :--- |
| countsOnly | true |
| groupBy | event |
| path | / |

We get the following result:

```text
[
  {
    "event": "created",
    "total": 16
  },
  {
    "event": "deleted",
    "total": 6
  },
  {
    "event": "updated",
    "total": 5
  }
]
```

Now we're getting somewhere. Since we're dealing with a very small data set, we can easily keep track of how many legitimate create, update and delete operations we should have performed so far. These numbers are:

#### Expected Entity Creates

| Collection | Count |
| :--- | ---: |
| Departments | 1 |
| Employees | 4 |
| Memberships | 4 |
| Reporting | 3 |
| **Total** | **12** |

#### Expected Entity Updates

| Collection | Count |
| :--- | ---: |
| Employees | 2 |
| Reporting | 1 |
| **Total** | **3** |

#### Expected Entity Deletes

| Collection | Count |
| :--- | ---: |
| Reporting | 1 |
| Membership | 1 |
| **Total** | **2** |

Something fishy is going on here. There should be a total of only 17 events while we're getting 27 \(your number may vary or even be exactly 17 - we will see why\)! Let us run a grouping by collection to figure out where our tally goes out of sync.

### Counts by Collection

We split event counts by collection by using the parameters given below:

| Param | Value |
| :--- | :--- |
| countsOnly | true |
| groupBy | collection |
| path | / |

We get the following result:

```text
[
  {
    "collection": "employees",
    "total": 12
  },
  {
    "collection": "reporting",
    "total": 7
  },
  {
    "collection": "membership",
    "total": 5
  },
  {
    "collection": "departments",
    "total": 3
  }
]
```

Hmm, the `departments` collection has 3 events instead of the single create event that we would expect. Let's take a closer look.

## Events for Collection

### Departments \(No Aggregation\)

We fetch all events for the `departments` collection by using the parameters shown below:

| Param | Value |
| :--- | :--- |
| path | /c/departments |

We get the following result:

```text
[
  {
    "_key": "44787803",
    "_id": "recallgraph_events/44787803",
    "_rev": "_ackLRwq---",
    "meta": {
      "id": "departments/44787802",
      "key": "44787802",
      "rev": "_ackLRwW---"
    },
    "ctime": 1588761284.3908088,
    "event": "created",
    "collection": "departments",
    "last-snapshot": "recallgraph_snapshots/origin-44778885",
    "hops-from-last-snapshot": 2,
    "hops-till-next-snapshot": 5,
    "hops-from-origin": 1
  },
  {
    "_key": "44787535",
    "_id": "recallgraph_events/44787535",
    "_rev": "_ackIsmy---",
    "meta": {
      "id": "departments/44787328",
      "key": "44787328",
      "rev": "_ackGsYm---"
    },
    "ctime": 1588761115.2729917,
    "event": "deleted",
    "collection": "departments",
    "last-snapshot": "recallgraph_snapshots/origin-44778885",
    "hops-from-last-snapshot": 3,
    "hops-from-origin": 2
  },
  {
    "_key": "44787331",
    "_id": "recallgraph_events/44787331",
    "_rev": "_ackGsY2---",
    "meta": {
      "id": "departments/44787328",
      "key": "44787328",
      "rev": "_ackGsYm---"
    },
    "ctime": 1588760983.9785848,
    "event": "created",
    "collection": "departments",
    "last-snapshot": "recallgraph_snapshots/origin-44778885",
    "hops-from-last-snapshot": 2,
    "hops-till-next-snapshot": 5,
    "hops-from-origin": 1
  }
]
```

Now we have an answer! The last two events were a `create`, followed by a `delete` of the same node \(`departments/44787328`, see the `meta.id` field\). Someone created a department node, and for some reason subsequently deleted it. However RecallGraph remembers everything, and we will see how we can get a peek into the contents of this deleted node \(and possibly a motive behind deleting it\) when we explore the [`SHOW`](../navigating-history/show.md) endpoint. For now, we know part of the answer to the discrepancy in the total count.

We finish the exercise by similarly accessing the individual events for the `reporting` and the `employees` collections respectively \(`memberships` counts already match up\).

### Reporting

#### Counts by Node

We fetch event counts grouped by node for the `reporting` collection by using the parameters shown below:

| Param | Value |
| :--- | :--- |
| path | /c/reporting |
| groupBy | node |
| countsOnly | true |

We get the following result:

```text
[
  {
    "node": "reporting/44799849",
    "total": 5
  },
  {
    "node": "reporting/44795731",
    "total": 1
  },
  {
    "node": "reporting/44795739",
    "total": 1
  }
]
```



