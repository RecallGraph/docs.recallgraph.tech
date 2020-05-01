---
description: Step 2 in the execution pipeline for many bulk read endpoints.
---

# Grouping

Many bulk read endpoints support an optional grouping parameter, using which results can be grouped together on some attribute. Common grouping attributes include:

1. **Node:** Group by the `_id` of the node \(vertex or edge\) represented by an entity \([event](./#event), [command](./#command) or historical version\) in the result set.
2. **Collection:** Group by the _collection_ of the node represented by an entity in the result set.
3. **Type:** Group by the _type_ \(vertex or edge\) of node represented by an entity in the result set.
4. **Event:** Group by the _type_ \(created/updated/deleted\) of event that the entity in a result set represents.

It is not necessary that all attributes are available for all endpoints. It depends on the core structure of the data requested.

Grouping clauses are applied within the DB query that is used to fetch intermediate results.

