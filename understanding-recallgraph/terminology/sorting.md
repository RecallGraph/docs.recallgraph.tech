---
description: Step 3 in the execution pipeline for many bulk read endpoints.
---

# Sorting

Many bulk read endpoints support an optional sorting parameter, using which results can be sorted on some attribute.

The sorting attribute is dependent on the specific API endpoint, and the grouping parameter in use. The user gets to specify whether the sort order should be ascending or descending.

For ungrouped results, the sorting attribute is a primary field of the members of the ungrouped intermediate result set \(for example, `ctime` or `_id`\). For grouped results, the sorting attribute is always the grouping attribute.

{% hint style="info" %}
Sorting clauses are applied within the DB query that is used to fetch intermediate results.
{% endhint %}

{% hint style="info" %}
See the API docs for more details on attribute-specific sorting.
{% endhint %}

