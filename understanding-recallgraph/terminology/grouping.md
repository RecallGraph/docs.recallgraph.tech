---
description: Step 2 in the execution pipeline for many bulk read endpoints.
---

# Grouping

Many bulk read endpoints support an optional grouping parameter, using which results can be grouped together on some attribute. Common grouping attributes include:

1. **Node:** Group by the `_id` of the node \(vertex or edge\) represented by an entity \([event](./#event), [command](./#command) or historical version\) in the result set.
2. **Collection:** Group by the _collection_ of the node represented by an entity in the result set.
3. **Type:** Group by the _type_ \(vertex or edge\) of node represented by an entity in the result set.
4. **Event:** Group by the _type_ \(created/updated/deleted\) of event that the entity in a result set represents.

{% hint style="info" %}
It is not necessary that all of the above grouping attributes are available for all endpoints. It depends on the core structure of the data requested.
{% endhint %}

If a grouping attribute is specified, an additional set of sub-parameters can be optionally specified that determine how the entries within a group are arranged. These are:

1. **Group Sort:** Sorts entries within a group by a specific attribute. The attribute choice depends on the specific endpoint. See API docs for more details.
2. **Group Slice:** Skip and/or limit entries within a group, post group-sort.
3. **Aggregation \(Count\):** Rather than return an entity list within a group, return their count \(group-wise totals\). If this sub-parameter is specified, the other sub-parameters are ignored.

{% hint style="info" %}
There is a special case of aggregation, wherein counts are requested, but the grouping attribute is absent. In this case, the API returns the total count of ALL entities, rather than group-wise totals. In this case, the upper level sorting and slicing parameters are ignored.
{% endhint %}

{% hint style="info" %}
Grouping clauses are applied within the DB query that is used to fetch intermediate results.
{% endhint %}

