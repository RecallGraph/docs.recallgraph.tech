---
description: Settings for RecallGraph
---

# Configuration

RecallGraph exports two settings - `snapshot-intervals` and `sampling-probability`. These are explained in the sections below.

## `snapshot-intervals`

This is a JSON object that stores the [snapshot](../../understanding-recallgraph/terminology/#snapshot) interval for each collection. Its keys are collection names and values are non-negative integers. If the value is 0, then snapshots are disabled for that collection. For a number greater than 0, RecallGraph interprets the number as the minimum number of [events](../../understanding-recallgraph/terminology/#event) that must occur on a document until a snapshot is generated for that document. A snapshot is created only if the event is not a `DELETE` event.

There is also a global setting for snapshot intervals which applies to all collections which are not explicitly listed in this object. This global setting is stored in the key `_default` in the same object. The default value for this setting is:

```text
{
  "_default": 5
}
```

A sample object with some collection-specific values might look like:

```text
{
  "_default": 7,
  "flights": 10,
  "airports": 0 //snapshots disabled
}
```

If this setting is changed globally, or for a collection, after some snapshots have already been created, the existing snapshots are left untouched and the new value\(s\) only apply to snapshots created thereafter.

## `sampling-probability`

This setting is used for determining whether to collect a [tracing](tracing.md) sample. The default is 0. Leave it at 0 if you're not going to use tracing in your deployment.

If set to a value between 0 and 1 \(inclusive\), it is interpreted as the probability with which a tracing sample may be collected for an incoming HTTP request. Regardless of the value set here, a trace may be force-collected or force-suppressed using designated HTTP headers. These headers and other tracing parameters are covered in more detail in the [tracing](tracing.md) section.

