---
description: >-
  A point-in-time traversal (walk) of a past version of the graph, with the
  option to apply additional post-filters to the result.
---

# Traverse

## The Story So Far...

> Dogbert has been asking some very uncomfortable questions ever since he arrived at the plant. Just recently, Asok from the IT department only barely escaped with his internship intact \(and with a strong reprimand on record\).
>
> Eric hasn't had it easy either. He is currently under the scanner for his lax diligence while hiring what turned out be an unqualified Safety Officer \(Kenny had faked his certificates\). This lapse could have been easily avoided, had Eric had the foresight to hire a background-checking agency.
>
> Incidentally, Dogbert is a board member of just such an agency, and has openly expressed his intention to strongly recommend mandatory background checks for new hires to ACME's upper management \(no doubt as an unbiased 3rd party\).
>
> Eric can't wait to see the last of this pesky auditor, and does all he can to expedite the ongoing investigation.

## Current Org Structure

We want to start by looking at the entire organization hierarchy in its current state. This can be achieved by running an AQL traversal, but RecallGraph is also capable of this \(and more, when it's time to time-travel\). We fire a `POST` query at the `TRAVERSE` endpoint with the following parameters:

| Param | Value |
| :--- | :--- |
| `svid` | `departments/44787802` |
| `depth` | `2` |
| `body` | `{   "edges": {     "reporting": "inbound",     "membership": "inbound"   } }` |

**Response:**

```text
{
  "vertices": [
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
      "role": "Senior Plant Manager",
      "last_name": "Marsh",
      "first_name": "Stan",
      "_rev": "_acx_g1C--_",
      "_id": "employees/44794453",
      "_key": "44794453",
      "valid_from": "2020-05-07"
    },
    {
      "role": "Plant Manager",
      "last_name": "Broflovski",
      "first_name": "Kyle",
      "_rev": "_aclQHSm---",
      "_id": "employees/44794457",
      "_key": "44794457"
    }
  ],
  "edges": [
    {
      "_rev": "_aclYlUy---",
      "_to": "departments/44787802",
      "_from": "employees/44794449",
      "_id": "membership/44795272",
      "_key": "44795272"
    },
    {
      "_rev": "_aclYlVu---",
      "_to": "departments/44787802",
      "_from": "employees/44794453",
      "_id": "membership/44795280",
      "_key": "44795280"
    },
    {
      "_rev": "_aclYlWC---",
      "_to": "departments/44787802",
      "_from": "employees/44794457",
      "_id": "membership/44795286",
      "_key": "44795286"
    },
    {
      "_rev": "_acldI2S---",
      "_to": "employees/44794449",
      "_from": "employees/44794453",
      "_id": "reporting/44795731",
      "_key": "44795731"
    },
    {
      "_rev": "_acldI3i---",
      "_to": "employees/44794449",
      "_from": "employees/44794457",
      "_id": "reporting/44795739",
      "_key": "44795739"
    }
  ]
}
```

This gives us all the departments, employees and their membership and reporting relations as of their current state.

{% hint style="info" %}
Note that Kenny is absent in the above result, even though his employee node was not deleted. This is because all edges connecting his node to the rest of the graph have been severed.
{% endhint %}

## When Kenny was Actively Employed

Next, we want to travel back in time to a point when Kenny was still actively connected to the department and reported to Eric. We obtain a suitable timestamp by running a `LOG` query and picking a timestamp before his deactivation. By now, the reader should know how to run this query.

**Request:**

| Param | Value |
| :--- | :--- |
| `timestamp` | `1588815039.3459842` |
| `svid` | `departments/44787802` |
| `depth` | `2` |
| `body` | `{   "edges": {     "reporting": "inbound",     "membership": "inbound"   } }` |

Response:

```text
{
  "vertices": [
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
      "role": "Senior Plant Manager",
      "last_name": "Marsh",
      "first_name": "Stan",
      "_rev": "_acx_g1C--_",
      "_id": "employees/44794453",
      "_key": "44794453",
      "valid_from": "2020-05-07"
    },
    {
      "role": "Plant Manager",
      "last_name": "Broflovski",
      "first_name": "Kyle",
      "_rev": "_aclQHSm---",
      "_id": "employees/44794457",
      "_key": "44794457"
    },
    {
      "role": "Safety Officer",
      "last_name": "McCormick",
      "first_name": "Kenny",
      "_rev": "_acmFz6u---",
      "_id": "employees/44799683",
      "_key": "44799683"
    }
  ],
  "edges": [
    {
      "_rev": "_aclYlUy---",
      "_to": "departments/44787802",
      "_from": "employees/44794449",
      "_id": "membership/44795272",
      "_key": "44795272"
    },
    {
      "_rev": "_aclYlVu---",
      "_to": "departments/44787802",
      "_from": "employees/44794453",
      "_id": "membership/44795280",
      "_key": "44795280"
    },
    {
      "_rev": "_aclYlWC---",
      "_to": "departments/44787802",
      "_from": "employees/44794457",
      "_id": "membership/44795286",
      "_key": "44795286"
    },
    {
      "_rev": "_acmGvEa---",
      "_to": "departments/44787802",
      "_from": "employees/44799683",
      "_id": "membership/44799786",
      "_key": "44799786"
    },
    {
      "_rev": "_acldI2S---",
      "_to": "employees/44794449",
      "_from": "employees/44794453",
      "_id": "reporting/44795731",
      "_key": "44795731"
    },
    {
      "_rev": "_acldI3i---",
      "_to": "employees/44794449",
      "_from": "employees/44794457",
      "_id": "reporting/44795739",
      "_key": "44795739"
    },
    {
      "_rev": "_acmeb3S--_",
      "_to": "employees/44794449",
      "_from": "employees/44799683",
      "_id": "reporting/44799849",
      "_key": "44799849"
    }
  ]
}
```

Here, we see Kenny's node present in the result, along with both its connections \(`reporting` and `membership`\).

## When Kenny was 'Reporting' to Kyle

Let's look at the state of the graph when Kenny was incorrectly marked as reporting to Kyle. The request parameters are left as an exercise for the reader to work out. The response is shown below:

```text
{
  "vertices": [
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
    },
    {
      "role": "Plant Manager",
      "last_name": "Broflovski",
      "first_name": "Kyle",
      "_rev": "_aclQHSm---",
      "_id": "employees/44794457",
      "_key": "44794457"
    },
    {
      "role": "Safety Officer",
      "last_name": "McCormick",
      "first_name": "Kenny",
      "_rev": "_acmFz6u---",
      "_id": "employees/44799683",
      "_key": "44799683"
    }
  ],
  "edges": [
    {
      "_rev": "_aclYlUy---",
      "_to": "departments/44787802",
      "_from": "employees/44794449",
      "_id": "membership/44795272",
      "_key": "44795272"
    },
    {
      "_rev": "_aclYlVu---",
      "_to": "departments/44787802",
      "_from": "employees/44794453",
      "_id": "membership/44795280",
      "_key": "44795280"
    },
    {
      "_rev": "_aclYlWC---",
      "_to": "departments/44787802",
      "_from": "employees/44794457",
      "_id": "membership/44795286",
      "_key": "44795286"
    },
    {
      "_rev": "_acmGvEa---",
      "_to": "departments/44787802",
      "_from": "employees/44799683",
      "_id": "membership/44799786",
      "_key": "44799786"
    },
    {
      "_rev": "_acldI2S---",
      "_to": "employees/44794449",
      "_from": "employees/44794453",
      "_id": "reporting/44795731",
      "_key": "44795731"
    },
    {
      "_rev": "_acldI3i---",
      "_to": "employees/44794449",
      "_from": "employees/44794457",
      "_id": "reporting/44795739",
      "_key": "44795739"
    },
    {
      "_rev": "_acmHVwO---",
      "_to": "employees/44794457",
      "_from": "employees/44799683",
      "_id": "reporting/44799849",
      "_key": "44799849"
    }
  ]
}
```

{% hint style="info" %}
Note that Stan had not been promoted at the time Kenny was 'reporting' to Kyle. As expected, he is shown as just a Plant Manager in the above result.

Also note the last entry in the edge list \(the outbound reporting edge from Kenny\), where it clearly points to Kyle.
{% endhint %}

## Before Kenny Joined

Finally, let's go back in time to a point when all was well, before Kenny had joined the team. The result would be:

```text
{
  "vertices": [
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
    },
    {
      "role": "Plant Manager",
      "last_name": "Broflovski",
      "first_name": "Kyle",
      "_rev": "_aclQHSm---",
      "_id": "employees/44794457",
      "_key": "44794457"
    }
  ],
  "edges": [
    {
      "_rev": "_aclYlUy---",
      "_to": "departments/44787802",
      "_from": "employees/44794449",
      "_id": "membership/44795272",
      "_key": "44795272"
    },
    {
      "_rev": "_aclYlVu---",
      "_to": "departments/44787802",
      "_from": "employees/44794453",
      "_id": "membership/44795280",
      "_key": "44795280"
    },
    {
      "_rev": "_aclYlWC---",
      "_to": "departments/44787802",
      "_from": "employees/44794457",
      "_id": "membership/44795286",
      "_key": "44795286"
    },
    {
      "_rev": "_acldI2S---",
      "_to": "employees/44794449",
      "_from": "employees/44794453",
      "_id": "reporting/44795731",
      "_key": "44795731"
    },
    {
      "_rev": "_acldI3i---",
      "_to": "employees/44794449",
      "_from": "employees/44794457",
      "_id": "reporting/44795739",
      "_key": "44795739"
    }
  ]
```

## The Conclusion

> Dogbert has left the premises for now, but not before inking a sweet deal for his background-checking agency, in exchange for leaving some critical lapses out of his audit report. However, he realizes that such convenient omissions will be impossible to sneak in the next time, as ACME Inc. has decided to purchase a full-blown ERP from the same firm that sold them the HRMS. **The ERP also uses RecallGraph as its underlying data store. By now, Dogbert knows only too well that you can't hide anything from this clever foxx!**
>
> All is well in ACME land again. All 10000 explosive tennis balls have been shipped to an ecstatic Mr. Coyote. Eric now has a few minutes to himself, until the next batch of orders comes through. He spends these moments wondering, for the first time, what anyone could possibly use these devils for. If you're wondering the same thing, wonder no more. See the recorded demo at [https://www.dailymotion.com/video/x569rr7?start=374](https://www.dailymotion.com/video/x569rr7?start=374)

