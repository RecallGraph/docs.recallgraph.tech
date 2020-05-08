---
description: >-
  Fetch a list of forward or reverse commands (diffs) between commits for
  specified documents.
---

# Diff

## The Story So Far...

As we noticed in the previous section, one of the reporting edges had undergone [more updates than expected](log.md#post-filtered-events). We will dig deeper into those extra updates in this section.

The offending edge in question had the id `reporting/44799849`.

{% hint style="info" %}
We will use the `GET` variant of the `DIFF` endpoint in this guide, although the `POST` variant can also be used to achieve the same results.
{% endhint %}

## Diffs in Node-Brace Scope

Since we're only interested in fetching [diffs](../../../understanding-recallgraph/terminology/#diff) for a single entity, we will use the [node-brace scope](../../../understanding-recallgraph/terminology/#node-brace-scope) here, which is the most efficient [pre-filter](../../../understanding-recallgraph/terminology/pre-filters.md) to use when node ids are known beforehand.

Since we're only interested in the `updated` events, we will omit the `created` and `deleted` events by using upper and lower time-bounds in the pre-filters:

1. The lower time bound will be the `ctime` of the first `updated` event in chronological order, since it is inclusive.
2. The upper time bound will be the `ctime` of the final `deleted` event, since it is exclusive.

We can get these values from the event log described in the [last section](log.md#post-filtered-events). The request parameters would be as follows:

| Param | Value |
| :--- | :--- |
| `since` | `1588770714.352307` |
| `until` | `1588816959.5006447` |
| `path` | `/n/reporting/44799849` |

**Response:**

```text
[
  {
    "node": "reporting/44799849",
    "commandsAreReversed": false,
    "events": [
      {
        "_id": "recallgraph_events/44801785",
        "_key": "44801785",
        "ctime": 1588770714.352307,
        "event": "updated",
        "last-snapshot": "recallgraph_snapshots/origin-44778934",
        "meta": {
          "rev": "_acmbKt---_"
        }
      },
      {
        "_id": "recallgraph_events/44802064",
        "_key": "44802064",
        "ctime": 1588770902.498967,
        "event": "updated",
        "last-snapshot": "recallgraph_snapshots/origin-44778934",
        "meta": {
          "rev": "_acmeCcG--_"
        }
      },
      {
        "_id": "recallgraph_events/44802110",
        "_key": "44802110",
        "ctime": 1588770928.5333633,
        "event": "updated",
        "last-snapshot": "recallgraph_snapshots/origin-44778934",
        "meta": {
          "rev": "_acmeb3S--_"
        }
      }
    ],
    "commands": [
      [
        {
          "op": "test",
          "path": "/_rev",
          "value": "_acmHVwO---"
        },
        {
          "op": "replace",
          "path": "/_rev",
          "value": "_acmbKt---_"
        },
        {
          "op": "test",
          "path": "/_to",
          "value": "employees/44794457"
        },
        {
          "op": "replace",
          "path": "/_to",
          "value": "employees/44794449"
        }
      ],
      [
        {
          "op": "test",
          "path": "/_rev",
          "value": "_acmbKt---_"
        },
        {
          "op": "replace",
          "path": "/_rev",
          "value": "_acmeCcG--_"
        }
      ],
      [
        {
          "op": "test",
          "path": "/_rev",
          "value": "_acmeCcG--_"
        },
        {
          "op": "replace",
          "path": "/_rev",
          "value": "_acmeb3S--_"
        }
      ]
    ]
  }
]
```

We can see the diffs, grouped by node \(in this case, there's only one node\), with each group containing a list of event in chronological order, followed by a list of [commands](../../../understanding-recallgraph/terminology/#command) corresponding to each event. These co are listed in the [JSON Patch \(RFC6902\)](https://tools.ietf.org/html/rfc6902) format, and are human-readable.

## Conclusion

We see that the first diff is the one that effected a change in reporting structure, replacing the `_to` destination from Kyle to Eric. The remaining two commands did not record anything significant. They have been executed by the HRMS operator with no content changes, and so have no impact on the actual edge.

