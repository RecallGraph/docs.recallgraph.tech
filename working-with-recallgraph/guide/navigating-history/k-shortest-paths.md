---
description: 'Point-in-time, weighted, shortest paths between two endpoints.'
---

# k Shortest Paths

## The Setup

Finally, we take a moment to run a few weighted shortest path queries. For this, we will use a specific historical version of the graph - the one at the [end of the Create section](../persisting-documents/create.md#end-result). This graph has Kenny wrongly marked as reporting to Kyle. Let us look at the graph again \(replicated below\):

![How many paths exist from Kenny to Stan?](../../../.gitbook/assets/examples-create-7.png)

Let us first manually enumerate the number of undirected paths \(with unique vertices\) that go from Kenny to Stan. They are as follows:

1. KY -&gt; M -&gt; S \(length = 2\)
2. KY -&gt; KL -&gt; E -&gt; S \(length = 3\)
3. KY -&gt; KL -&gt; M -&gt; S \(length = 3\)
4. KY -&gt; M -&gt; E -&gt; S \(length = 3\)
5. KY -&gt; KL -&gt; E -&gt; M -&gt; S \(length = 4\)
6. KY -&gt; KL -&gt; M -&gt; E -&gt; S \(length = 4\)
7. KY -&gt; M -&gt; KL  -&gt; E -&gt; S \(length = 4\)

Let us run some shortest path queries on this past version to see if get the results we expect.

## No Weight Function \(weight = 1\)

### Undirected Paths

We have not provided a weight function for the edges. This means **every edge carries a default weight of 1**. We allow **traversing in any direction over all edges**. We fix the max depth to 4, and set `k = 2` to return the 2 shortest paths.

**Request:**

| Param | Value |
| :--- | :--- |
| `timestamp` | `1588769414.948146` |
| `svid` | `employees/44799683` |
| `evid` | `employees/44794453` |
| `depth` | `4` |
| `k` | `1` |
| `body` | `{   "edges": {"reporting": "any", "membership": "any"} }` |

**Response:**

```text
[
  {
    "vertices": [
      {
        "role": "Safety Officer",
        "last_name": "McCormick",
        "first_name": "Kenny",
        "_rev": "_acmFz6u---",
        "_id": "employees/44799683",
        "_key": "44799683"
      },
      {
        "org": "ACME Inc.",
        "name": "Manufacturing",
        "_rev": "_ackLRwW---",
        "_id": "departments/44787802",
        "_key": "44787802"
      },
      {
        "role": "Plant Manager",
        "last_name": "Marsh",
        "first_name": "Stan",
        "_rev": "_aclQHSa---",
        "_id": "employees/44794453",
        "_key": "44794453"
      }
    ],
    "edges": [
      {
        "_rev": "_acmGvEa---",
        "_to": "departments/44787802",
        "_from": "employees/44799683",
        "_id": "membership/44799786",
        "_key": "44799786"
      },
      {
        "_rev": "_aclYlVu---",
        "_to": "departments/44787802",
        "_from": "employees/44794453",
        "_id": "membership/44795280",
        "_key": "44795280"
      }
    ],
    "cost": 2
  },
  {
    "vertices": [
      {
        "role": "Safety Officer",
        "last_name": "McCormick",
        "first_name": "Kenny",
        "_rev": "_acmFz6u---",
        "_id": "employees/44799683",
        "_key": "44799683"
      },
      {
        "org": "ACME Inc.",
        "name": "Manufacturing",
        "_rev": "_ackLRwW---",
        "_id": "departments/44787802",
        "_key": "44787802"
      },
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
      }
    ],
    "edges": [
      {
        "_rev": "_acmGvEa---",
        "_to": "departments/44787802",
        "_from": "employees/44799683",
        "_id": "membership/44799786",
        "_key": "44799786"
      },
      {
        "_rev": "_aclYlUy---",
        "_to": "departments/44787802",
        "_from": "employees/44794449",
        "_id": "membership/44795272",
        "_key": "44795272"
      },
      {
        "_rev": "_acldI2S---",
        "_to": "employees/44794449",
        "_from": "employees/44794453",
        "_id": "reporting/44795731",
        "_key": "44795731"
      }
    ],
    "cost": 3
  }
]
```

We get 2 paths, one of length 2, and one of length 3, viz:

1. KY -&gt; M -&gt; S \(length = 2\)
2. KY -&gt; M -&gt; E -&gt; S \(length = 3\)

{% hint style="warning" %}
There are multiple paths of length 3 having equal weight for the given parameters, of which only one can be returned. In such cases, we **cannot predict** which one will be selected.
{% endhint %}

