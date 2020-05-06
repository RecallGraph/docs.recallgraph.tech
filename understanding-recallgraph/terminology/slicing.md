---
description: Step 4 in the execution pipeline for many bulk read endpoints.
---

# Slicing

Many bulk read endpoints support an optional slicing \(`skip` + `limit`\) parameter pair, using which results can be paged.

The `skip` attribute only takes effect if the `limit` attribute is present and has a non-zero value. This is a peculiarity of how the [limit clause works in AQL](https://www.arangodb.com/docs/stable/aql/operations-limit.html).

For grouped results, slicing is applied on the groups themselves.

{% hint style="info" %}
Slicing clauses are applied within the DB query that is used to fetch intermediate results.
{% endhint %}

