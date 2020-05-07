---
description: Replace single / multiple nodes.
---

# Update

## The Story So Far...

> Kenny has joined the manufacturing department as the Safety Officer in-charge, and all employee entries are now correctly recorded in the HRMS.
>
> It is the annual appraisal time, and Stan has been working extra hard the last few weeks to stay on the good books of Eric. Eric has taken note of this and has decided to promote Stan to the rank of _Senior Plant Manager_. Kyle is not too happy about this, since he too has been going the extra mile in hopes of earning that promotion, but alas, he failed to make his efforts adequately visible to Eric. Bummer, but Kyle has learnt a few tricks by observing Stan, so there's still hope for next time.

We need to update Stan's role in the HRMS. However, we note that it is not enough to just update the role, although RecallGraph will track history for the change - we need to also somehow keep track of the _effective date_ from which his promotion is valid. In real-world scenarios, this may not be the same as the time of entry of the new information, which may be done preemptively or retroactively.

{% hint style="warning" %}
RecallGraph is not yet fully [bi-temporal](../../../understanding-recallgraph/concepts.md#versioned-graph-stores). If it were, there would be a dedicated _valid time_ field which could be used to track the effective promotion date. As of this writing, RecallGraph only tracks _transaction time_, i.e. the time at which data was actually entered into the system.

For a special case of _real-time applications_, the valid time may be considered to be the same as the transaction time, but an HRMS is not such a case. Through this guide, we will also explore the possibility of any tricks that may be available to work around the lack of an explicit _valid time_ field.
{% endhint %}

To keep track of the effective promotion date, we will start marking each write operation with a user-defined timestamp, which we will call `valid_from`. Note that this can only work for creates and updates, and not for delete operations.

{% hint style="warning" %}
User-defined _valid time_ fields can only be used for create and update operations, not for delete operations.
{% endhint %}

## Entering Data

In the Swagger console locate the tab with the ![](../../../.gitbook/assets/image%20%282%29.png) button. The `collection` parameter should be set to `employees` and the body should contain **only the new and the changed content** of the entity.

{% hint style="info" %}
One of `_key` or `_id` would still need to be present.
{% endhint %}

**Request:**

```text
{
  "_key": "44794453",
  "role": "Senior Plant Manager",
  "valid_from": "2020-05-07"
}
```

**Response:**

```text
{
  "_id": "employees/44794453",
  "_key": "44794453",
  "_rev": "_acx_g1C--_",
  "_oldRev": "_aclQHSa---"
}
```

