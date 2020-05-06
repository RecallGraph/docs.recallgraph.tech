---
description: Replace single / multiple nodes.
---

# Replace

## The Story So Far...

> All data for the organization has been entered, albeit with an error. Kenny, the newly hired Safety Officer, has been recorded as reporting to Kyle rather than the Unit Supervisor, Eric.

We will correct this data entry error in our database by replacing the incorrect reporting edge. After replacement, Kenny should be correctly shown to be reporting to Eric.

{% hint style="warning" %}
**Important:** Note that the `REPLACE` operation behaves exactly as in the ArangoDB core REST API. It **does not** replace the underlying document, i.e. the body is replaced but the `_key` remains the same.
{% endhint %}

{% hint style="success" %}
If you missed taking note of the edge `_id` while inserting the reporting relationship between Kenny and Kyle, you can easily retrieve it by firing a simple AQL query. This is an exercise left for the reader.
{% endhint %}

## Entering Data

In the Swagger console locate the tab with the ![](../../../.gitbook/assets/image%20%283%29.png) button. The `collection` parameter should be set to `reporting` and the body should contain the **entire contents** of the new reporting relationship.

{% hint style="info" %}
1. Only one of `_key` or `_id` need be present.
2. `_rev` can be omitted \(ignored if present\).
{% endhint %}

```text
{
    "_id": "reporting/44799849",
    "_from": "employees/44799683",
    "_to": "employees/44794449"
}
```

## End Result

Running the graph query should now yield the correct relations:

![Kenny is now correctly shown to be reporting to Eric.](../../../.gitbook/assets/examples-replace.png)

{% hint style="success" %}
Although not demonstrated in this example, this endpoint also supports bulk replace, similar to [`CREATE`](create.md#employee-information).
{% endhint %}

