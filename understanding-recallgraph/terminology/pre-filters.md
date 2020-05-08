---
description: >-
  Step 1 in the execution pipeline for all bulk read and some bulk write
  endpoints.
---

# Pre-Filters

A pre-filter is the first step to execute in the operator pipeline for many bulk read/write requests in RecallGraph. This filter acts in two ways:

1. As a **compulsory** entity selector that determines the [scope](./#scopes) within which to search for associated entities \([events](./#event), [commands](./#command) or historical versions\), and an `_id` pattern that every vertex/edge in that scope must match in order for its associated entities to proceed further down the pipeline. This is determined by the [path](./#path) parameter in the request.
2. As Transaction-Time \(time of record for an event\) bounds which include:
   1. An **optional** lower bound \(inclusive\) - `since`,
   2. An **optional** upper bound \(inclusive\) - `until`

{% hint style="warning" %}
Conditions defined by the pre-filters apply to individual documents \(scope\) and events \(time bounds\). They do not apply to groups, which may be created later in the operator execution pipeline.
{% endhint %}

{% hint style="success" %}
Pre-filters are applied within the DB query that is used to fetch intermediate results, and so it is desirable to use them to narrow down intermediate results as much as possible, before they are processed further in the rest of the pipeline.
{% endhint %}

