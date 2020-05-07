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
| `countsOnly` | `true` |
| `path` | `/` |

The result obtained will have a structure as shown below. Your actual numbers may vary. We will shortly see why.

```text
[
  {
    "total": 27
  }
]
```

This tells us that there have been a total of 27 events recorded in the system \(in the instance from which this result was pulled\). That looks a bit strange since we have not performed 27 write operations so far in our walkthrough.

{% hint style="success" %}
You can also use the `POST` variant to achieve the same results. In the `POST` variant, the `path` and the `postFilter` parameters are sent in the request body rather than as query parameters. This lets you specify large path patterns and/or filter expressions that would otherwise cause an `HTTP 414 URI Too Long` exception in the `GET` variant.
{% endhint %}

### Counts by Type

The root cause behind this anomaly cannot be determined from this singular number. Therefore, we decide to drill down a bit by using a more fine grained grouping clause. We split event counts by event type by using the parameters given below:

| Param | Value |
| :--- | :--- |
| `countsOnly` | `true` |
| `groupBy` | `event` |
| `path` | `/` |

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

Now we're getting somewhere. Since we're dealing with a very small dataset, we can easily keep track of how many legitimate create, update and delete operations we should have performed so far. These numbers are:

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
| `countsOnly` | `true` |
| `groupBy` | `collection` |
| `path` | `/` |

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
| `path` | `/c/departments` |
| `sort` | `asc` |

We get the following result:

```text
[
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
  }
]
```

Now we have an answer! The first two events were a `created`, followed by a `deleted` of the same node \(`departments/44787328`, see the `meta.id` field\). Someone created a department node, and for some reason subsequently deleted it. However RecallGraph remembers everything, and we will see how we can get a peek into the contents of this deleted node \(and possibly a motive behind deleting it\) when we explore the [`SHOW`](../navigating-history/show.md) endpoint. For now, we know part of the answer to the discrepancy in the total count.

We finish the exercise by similarly accessing the individual events for the `reporting` and the `employees` collections respectively \(`memberships` counts already match up\).

### Reporting

#### Counts by Node

We fetch event counts grouped by node for the `reporting` collection by using the parameters shown below:

| Param | Value |
| :--- | :--- |
| `path` | `/c/reporting` |
| `groupBy` | `node` |
| `countsOnly` | `true` |

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

The two nodes with 1 event each seem to be in order, most likely representing the reporting relations between `Kyle --> Eric` and `Stan --> Eric`. We know that Kenny's reporting relationship had to undergo one modification, followed by a deletion. Including the initial creation, that should be a total of 3 events, but here we see 5. The next step is to drill down into this.

#### Post-Filtered Events

The ideal way to fetch events when node ids are known is to use the [node brace scope](../../../understanding-recallgraph/terminology/#node-brace-scope), but since this is a small dataset, we can get away with using the collection scope with a [post-filter](../../../understanding-recallgraph/terminology/post-filters.md), just to give an example of post-filter usage. We use request parameters as shown below:

| Param | Value |
| :--- | :--- |
| `path` | `/c/reporting` |
| `groupBy` | `node` |
| `groupSort` | `asc` |
| `postFilter` | `node === "reporting/44799849"` |

We get the following result:

```text
[
  {
    "node": "reporting/44799849",
    "events": [
      {
        "_key": "44799850",
        "_id": "recallgraph_events/44799850",
        "_rev": "_acmHVwO--A",
        "meta": {
          "id": "reporting/44799849",
          "key": "44799849",
          "rev": "_acmHVwO---",
          "from": "employees/44799683",
          "to": "employees/44794457"
        },
        "ctime": 1588769414.948146,
        "event": "created",
        "collection": "reporting",
        "last-snapshot": "recallgraph_snapshots/origin-44778934",
        "hops-from-last-snapshot": 2,
        "hops-till-next-snapshot": 5,
        "hops-from-origin": 1
      },
      {
        "_key": "44801785",
        "_id": "recallgraph_events/44801785",
        "_rev": "_acmbKtO---",
        "meta": {
          "id": "reporting/44799849",
          "key": "44799849",
          "rev": "_acmbKt---_",
          "oldRev": "_acmHVwO---",
          "toNew": "employees/44794449",
          "toOld": "employees/44794457"
        },
        "ctime": 1588770714.352307,
        "event": "updated",
        "collection": "reporting",
        "last-snapshot": "recallgraph_snapshots/origin-44778934",
        "hops-from-last-snapshot": 3,
        "hops-till-next-snapshot": 4,
        "hops-from-origin": 2
      },
      {
        "_key": "44802064",
        "_id": "recallgraph_events/44802064",
        "_rev": "_acmeCcm---",
        "meta": {
          "id": "reporting/44799849",
          "key": "44799849",
          "rev": "_acmeCcG--_",
          "oldRev": "_acmbKt---_"
        },
        "ctime": 1588770902.498967,
        "event": "updated",
        "collection": "reporting",
        "last-snapshot": "recallgraph_snapshots/origin-44778934",
        "hops-from-last-snapshot": 4,
        "hops-till-next-snapshot": 3,
        "hops-from-origin": 3
      },
      {
        "_key": "44802110",
        "_id": "recallgraph_events/44802110",
        "_rev": "_acmeb3a---",
        "meta": {
          "id": "reporting/44799849",
          "key": "44799849",
          "rev": "_acmeb3S--_",
          "oldRev": "_acmeCcG--_"
        },
        "ctime": 1588770928.5333633,
        "event": "updated",
        "collection": "reporting",
        "last-snapshot": "recallgraph_snapshots/origin-44778934",
        "hops-from-last-snapshot": 5,
        "hops-till-next-snapshot": 2,
        "hops-from-origin": 4
      },
      {
        "_key": "44869570",
        "_id": "recallgraph_events/44869570",
        "_rev": "_acxc0_C---",
        "meta": {
          "id": "reporting/44799849",
          "key": "44799849",
          "rev": "_acmeb3S--_"
        },
        "ctime": 1588816959.5006447,
        "event": "deleted",
        "collection": "reporting",
        "last-snapshot": "recallgraph_snapshots/origin-44778934",
        "hops-from-last-snapshot": 6,
        "hops-from-origin": 5
      }
    ]
  }
]
```

{% hint style="info" %}
Note that events within a group are sorted in descending order in `ctime` by default, but we have reversed this order by using the `groupSort` parameter, so that now, the `created` event is listed at the top.
{% endhint %}

As expected, we start with a `created` event and end with a `deleted` event. However, instead of 1 `updated` event, we have 3! What could have been modified in those extra 2 updates? We will find out when we explore the [`DIFF`](diff.md) endpoint.

{% hint style="success" %}
Executing `LOG` with the following parameters would yield the same result:

```text
path: /n/reporting/44799849
groupBy: node
groupSort: asc
```

No post-filter is needed in this case, as the pre-filter \(in the scope defined by the path parameter\) is restricting the node list to identical effect. Can you guess which method is generally faster?
{% endhint %}

Finally, we dig into events related to the `employees` collection.

### Employees

#### Counts by Node

We fetch event counts grouped by node for the `employees` collection by using the parameters shown below:

| Param | Value |
| :--- | :--- |
| `path` | `/c/employees` |
| `groupBy` | `node` |
| `countsOnly` | `true` |

We get the following result:

```text
[
  {
    "node": "employees/44794101",
    "total": 2
  },
  {
    "node": "employees/44794107",
    "total": 2
  },
  {
    "node": "employees/44794111",
    "total": 2
  },
  {
    "node": "employees/44794453",
    "total": 2
  },
  {
    "node": "employees/44799683",
    "total": 2
  },
  {
    "node": "employees/44794449",
    "total": 1
  },
  {
    "node": "employees/44794457",
    "total": 1
  }
]
```

There are 7 employee records instead of the expected 4! We know the ids of the 4 employees we have been working with. Let us drill down into the events related to the remaining 3 mysterious employees. We will again use a post-filter to filter out nodes that are already known to us.

#### Post-Filtered Events

We use request parameters as shown below:

| Param | Value |
| :--- | :--- |
| `path` | `/c/employees` |
| `groupBy` | `node` |
| `groupSort` | `asc` |
| `postFilter` | `!(node =* "employees/{44794449,44794453,44794457,44799683}" )` |

We get the following result:

```text
[
  {
    "node": "employees/44794111",
    "events": [
      {
        "_key": "44794112",
        "_id": "recallgraph_events/44794112",
        "_rev": "_aclM0d2---",
        "meta": {
          "id": "employees/44794111",
          "key": "44794111",
          "rev": "_aclM0dy---"
        },
        "ctime": 1588765579.7735815,
        "event": "created",
        "collection": "employees",
        "last-snapshot": "recallgraph_snapshots/origin-44778911",
        "hops-from-last-snapshot": 2,
        "hops-till-next-snapshot": 5,
        "hops-from-origin": 1
      },
      {
        "_key": "44794417",
        "_id": "recallgraph_events/44794417",
        "_rev": "_aclP1Oa---",
        "meta": {
          "id": "employees/44794111",
          "key": "44794111",
          "rev": "_aclM0dy---"
        },
        "ctime": 1588765777.157764,
        "event": "deleted",
        "collection": "employees",
        "last-snapshot": "recallgraph_snapshots/origin-44778911",
        "hops-from-last-snapshot": 3,
        "hops-from-origin": 2
      }
    ]
  },
  {
    "node": "employees/44794107",
    "events": [
      {
        "_key": "44794108",
        "_id": "recallgraph_events/44794108",
        "_rev": "_aclM0dm---",
        "meta": {
          "id": "employees/44794107",
          "key": "44794107",
          "rev": "_aclM0di---"
        },
        "ctime": 1588765579.7697582,
        "event": "created",
        "collection": "employees",
        "last-snapshot": "recallgraph_snapshots/origin-44778911",
        "hops-from-last-snapshot": 2,
        "hops-till-next-snapshot": 5,
        "hops-from-origin": 1
      },
      {
        "_key": "44794414",
        "_id": "recallgraph_events/44794414",
        "_rev": "_aclP1OK---",
        "meta": {
          "id": "employees/44794107",
          "key": "44794107",
          "rev": "_aclM0di---"
        },
        "ctime": 1588765777.1531398,
        "event": "deleted",
        "collection": "employees",
        "last-snapshot": "recallgraph_snapshots/origin-44778911",
        "hops-from-last-snapshot": 3,
        "hops-from-origin": 2
      }
    ]
  },
  {
    "node": "employees/44794101",
    "events": [
      {
        "_key": "44794104",
        "_id": "recallgraph_events/44794104",
        "_rev": "_aclM0dK---",
        "meta": {
          "id": "employees/44794101",
          "key": "44794101",
          "rev": "_aclM0cq---"
        },
        "ctime": 1588765579.7554286,
        "event": "created",
        "collection": "employees",
        "last-snapshot": "recallgraph_snapshots/origin-44778911",
        "hops-from-last-snapshot": 2,
        "hops-till-next-snapshot": 5,
        "hops-from-origin": 1
      },
      {
        "_key": "44794411",
        "_id": "recallgraph_events/44794411",
        "_rev": "_aclP1Nm---",
        "meta": {
          "id": "employees/44794101",
          "key": "44794101",
          "rev": "_aclM0cq---"
        },
        "ctime": 1588765777.142716,
        "event": "deleted",
        "collection": "employees",
        "last-snapshot": "recallgraph_snapshots/origin-44778911",
        "hops-from-last-snapshot": 3,
        "hops-from-origin": 2
      }
    ]
  }
]
```

We now see that someone created 3 `employee` nodes, and for some reason subsequently deleted them. We will see how we can get a peek into the contents of these deleted nodes \(and possibly a motive behind deleting them\) when we explore the [`SHOW`](../navigating-history/show.md) endpoint.

{% hint style="success" %}
We could also use the following parameters to get exactly the same results:

```text
path: /n/employees/{44794101,44794107,44794111}
groupBy: node
groupSort: asc
```

No post-filter is needed in this case, as the pre-filter \(in the scope defined by the path parameter\) is restricting the node list to identical effect. Can you guess which method is generally faster?
{% endhint %}

