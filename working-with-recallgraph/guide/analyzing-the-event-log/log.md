---
description: >-
  Fetch a log of events for a given scope, optionally
  grouped/sorted/sliced/post-filtered.
---

# Log

{% hint style="info" %}
Before proceeding with this section of the guide, make sure you have understood the concepts of [patterns](../../../understanding-recallgraph/terminology/#patterns), [scopes](../../../understanding-recallgraph/terminology/#scopes) and [paths](../../../understanding-recallgraph/terminology/#path), explained in the [terminology](../../../understanding-recallgraph/terminology/) page.
{% endhint %}

{% hint style="info" %}
The `LOG` endpoint is a very powerful and flexible querying tool with lots of option combinations, all of which cannot possibly be explored in this guide. However, hopefully, this guide will cover enough ground to make readers comfortable in exploring the rest of the options by themselves.
{% endhint %}

## All Events

We don't know which events are of interest to us yet, so we begin with an ad hoc, exploratory analysis to get an idea of the number and types of events in the event log.

