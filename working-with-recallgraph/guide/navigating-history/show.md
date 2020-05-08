---
description: >-
  Fetch a set of documents, optionally grouped/sorted/sliced, that match a given
  path pattern, at a given point in time.
---

# Show

## The Story So Far...

As we noticed in a couple of sections \([here](../analyzing-the-event-log/log.md#departments) and [here](../analyzing-the-event-log/log.md#post-filtered-events-1)\) in the Log guide, a few extra nodes were added and later deleted. We will dig deeper into the contents of those extra nodes in this section.

{% hint style="success" %}
We will use the `GET` variant of the `SHOW` endpoint in this guide, although the `POST` variant can also be used to achieve the same results.
{% endhint %}

## Department

### The Deleted Entry

We last saw an extra department that was created and then deleted. We will now look into the contents of the now deleted department by invoking `SHOW` with a timestamp just at the time of creation \(any timestamp between creation \(inclusive\) and deletion \(exclusive\) will do\).

{% hint style="info" %}
We can find the department's id from the logs we fetched earlier.
{% endhint %}

**Request:**

| Param | Value |
| :--- | :--- |
| `timestamp` | `1588760983.9785848` |
| `path` | `/n/departments/44787328` |

**Response:**

```text
[
  {
    "org": "ACME Inc.",
    "name": "Manufacturing",
    "_rev": "_ackGsYm---",
    "_id": "departments/44787328",
    "_key": "44787328"
  }
]
```

Nothing wrong with the content, it seems.

### The Current Entry

Just to be sure, let us fetch the currently active department's contents for comparison. This can be done using just an AQL query \(since it is an active, current object in the database\), but we can also use `SHOW` to fetch it.

**Request:**

| Param | Value |
| :--- | :--- |
| `path` | `/c/departments` |

**Response:**

```text
[
  {
    "org": "ACME Inc.",
    "name": "Manufacturing",
    "_rev": "_ackLRwW---",
    "_id": "departments/44787802",
    "_key": "44787802"
  }
]
```

The contents are identical and there is no apparent reason for the previous node to have been deleted. This is looking more and more like a case of operator confusion/incompetence than any actual data issue.

Let us next look at the extra employee data.

## Employees

We know that some employee data was created and immediately deleted from the database. The operator has been up to something. It is not yet clear, but investigations so far are building up towards a case of inexperience or incompetence rather than deliberate malice or any real problem with the data itself.

### Log for Deleted Entries

We have already determined the offending employee ids in the log section. We will use them to fetch their contents. To help us identify a convenient time point to fire the query, we first fetch an ungrouped `LOG` for these nodes using the following request parameters:

| Param | Value |
| :--- | :--- |
| path | /n/employees/{44794101,44794107,44794111} |
| sort | asc |

**Response:**

```text
[
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
```

We see that all 3 `created` events occurred before the first `deleted` event.

### Content of Deleted Entries

We can therefore use the timestamp of the last `created` event to fetch the contents of all 3 employees in one go as follows:

**Request:**

| Param | Value |
| :--- | :--- |
| `path` | `/n/employees/{44794101,44794107,44794111}` |
| `timestamp` | `1588765579.7735815` |

**Response:**

```text
[
  {
    "role": "Unit Supervisor",
    "last_name": "Cartman",
    "first_name": "Eric",
    "_rev": "_aclM0cq---",
    "_id": "employees/44794101",
    "_key": "44794101"
  },
  {
    "role": "Plant Manager",
    "last_name": "Marsh",
    "first_name": "Stan",
    "_rev": "_aclM0di---",
    "_id": "employees/44794107",
    "_key": "44794107"
  },
  {
    "role": "Plant Manager",
    "last_name": "Broflovski",
    "first_name": "Kyle",
    "_rev": "_aclM0dy---",
    "_id": "employees/44794111",
    "_key": "44794111"
  }
]
```

Nothing wrong with these either! Just to be doubly sure, let us fetch the current employee set, as of a time when the first 3 employee records \(Eric, Stan and Kyle\) were just created.

### Available Entries

Request:

| Param | Value |
| :--- | :--- |
| path | /c/employees |
| timestamp | 1588765795.6586728 |

Response:

```text
[
  {
    "role": "Unit Supervisor",
    "last_name": "Cartman",
    "first_name": "Eric",
    "_rev": "_aclQHR6---",
    "_id": "employees/44794449",
    "_key": "44794449"
  },
  {
    "role": "Plant Manager",
    "last_name": "Marsh",
    "first_name": "Stan",
    "_rev": "_aclQHSa---",
    "_id": "employees/44794453",
    "_key": "44794453"
  },
  {
    "role": "Plant Manager",
    "last_name": "Broflovski",
    "first_name": "Kyle",
    "_rev": "_aclQHSm---",
    "_id": "employees/44794457",
    "_key": "44794457"
  }
]
```

It all matches up!

## The Closing Chapters

> Looks like the HRMS operator has a few questions to answer. Dogbert, the auditor, calls him in. It turns out, the operator, named Asok, is just an intern, fresh out of college, and this was his first assignment. He was confused and it took some fumbling around with the system before he got the hang of it. Those deleted nodes were just Asok playing around, trying to understand its myriad options.
>
> Dogbert realizes that this is no one's fault \(except to an extent, Asok's boss who sent an untrained intern over to operate a complex production system\), and poor Asok is let off the hook. However, he is sternly advised to [RTFM](https://en.wikipedia.org/wiki/RTFM) before touching the system further, and is pointed to the excellent documentation hosted at [https://docs.recallgraph.tech/](https://docs.recallgraph.tech/), that has a well-written guide with a realistic \(and eerily familiar\) storyline to help him learn the ropes.
>
> Dogbert's work here is nearly done. As a final item in his checklist, he asks to see the state of the org hierarchy at a few important time points. We will cover this in the next section, where we explore the [`TRAVERSE`](traverse.md) endpoint.

![Asok does not handle infinite recursions very well.](../../../.gitbook/assets/image%20%287%29.png)

