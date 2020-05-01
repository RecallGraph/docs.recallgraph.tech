---
description: >-
  A glossary of terms defined in the context of RecallGraph. Understanding these
  terms is a prerequisite to following the API docs and using the service
  effectively.
---

# Terminology

## Event Log

In order to keep track of changes to all documents \(vertices/edges\), _RecallGraph_ maintains a log of all write events \(create/update/delete\) that occur on each document through its interface. This log is built along the lines of the [event sourcing pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing).

It is comprised of the following components:

### Event

An event is a record of a write operation pertaining to a single document \(vertex/edge\) that takes place through _RecallGraph_'s API. The write operation is one of _create_, _update_, or _delete_. An event object contains information on a number of parameters related to the write op:

1. The `_id`, `_key` and `_rev` of the document which was written.
2. The _type_ of event - one of `created`, `updated` or `deleted`.
3. The UNIX timestamp \(in seconds\) of the instant when the write happened. This is also referred to as the **transaction time**. Decimal precision - 5 decimal points, ie., 0.1μs.
4. The sequence number of the write operation - this tracks the number of times this document was written, including creates, updates and deletes.
5. Distance from last [snapshot](./#snapshot).
6. Distance to next [snapshot](./#snapshot) \(which may or may not exist yet\).

### Diff

A reversible delta between two JSON objects, computed using the [JSON Patch \(RFC6902\)](https://tools.ietf.org/html/rfc6902) standard. This delta itself is represented as a JSON array of objects. For more information on its schema/format, refer to the aforementioned link.

### Command

A container for a single \(forward\) [Diff](./#diff) array. This is the storage unit for the difference between two successive [events](./#event) that occur on a single document \(vertex/edge\). The command also contains a reference to the `_id` of the document whose change it records. Every event is associated with a corresponding command that records the changes caused by that event.

#### Special Cases

1. When a document is first created, its command diff is computed from an empty JSON object \(`{}`\) to the document's JSON representation.
2. When a document is deleted, its command diff is computed from the document's last JSON representation to an empty JSON object \(`{}`\).

### Snapshot

After every configurable number of write [events](./#event) on a document, a snapshot of the current state of the document is recorded in its entirety. This can later be used to quickly rebuild a document's state at a particular point in the past, without having to replay all the events starting from the creation of the document up to that instant.

Instead, if a nearby snapshot is found \(before or after the instant\), it can be used as a starting point from where to replay the events, thus saving some computation time. In case the snapshot is in the future w.r.t. the instant asked for, the events are replayed in reverse chronological order, with their corresponding [diffs](./#diff) being first reversed before being applied.

## Patterns

Pattern strings are used in several places throughout the API to match against a set of resources. Other than the commonplace [regex pattern](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions), _RecallGraph_'s API heavily relies on two other patterns:

### Glob Pattern

A string containing a pattern that is accepted by [minimatch](https://realguess.net/2014/07/05/extended-pattern-matching/) \(extglob, globstar, brace expansion and any valid combinations thereof\).

### Brace Pattern

A string containing a valid [brace expansion pattern](https://www.npmjs.com/package/brace-expansion).

## Scopes

Many of _RecallGraph_'s queries are executed within the confines of a _scope_. A _scope_ defines a **restriction boundary** on the data that should considered for a query. There are 5 types of scope.

### Database Scope

This is essentially an unrestricted scope, i.e. it allows the entire database to be scanned during the query \(barring _RecallGraph_'s own service collections\).

### Graph Scope

This scope restricts the query to only look within the collections that belong to one or more \(named\) graphs whose names match a specified [glob pattern](./#glob-pattern) \(barring _RecallGraph_'s own service graphs\). The pattern is passed as a query parameter.

### Collection Scope

This scope restricts the query to only look within collections whose names match a specified [glob pattern](./#glob-pattern) \(barring _RecallGraph_'s own service collections\). The pattern is passed as a query parameter.

### Node-Glob Scope

This scope restricts the query to only look within those documents \(vertex/edge\) whose `_id` matches a specified [glob pattern](./#glob-pattern) \(barring documents within _RecallGraph_'s own service collections\). The pattern is passed as a query parameter.

### Node-Brace Scope

This scope restricts the query to only look within those documents \(vertex/edge\) whose `_id` matches a specified [brace pattern](./#brace-pattern) \(barring documents within _RecallGraph_'s own service collections\). The pattern is passed as a query parameter.

## Path

A pattern used to select a particular scope to scan. This is not to be confused with a [graph traversal path](https://www.arangodb.com/docs/stable/aql/graphs-traversals-explained.html). In this context, _path_ refers to the UNIX directory-like navigation pointers that the patterns resemble. These are defined as follows:

1. `/` - **The root path**. This tells the log builder to basically pick everything - the entire database for logging - every user-defined collection and every document \(vertex/edge\) \(existing and deleted\) therein. This is essentially the [Database Scope](./#database-scope).
2. `/g/<glob pattern>` - **Named Graph**. A path that starts with `/g/` followed by a valid [glob pattern](./#glob-pattern). This is essentially the [Graph Scope](./#graph-scope).
3. `/c/<glob pattern>` - **Collection**. A path that starts with `/c/` followed by a valid [glob pattern](./#glob-pattern). This is essentially the [Collection Scope](./#collection-scope).
4. `/ng/<glob pattern>` - **Node Glob**. A path that starts with `/ng/` followed by a valid [glob pattern](./#glob-pattern). This is essentially the [Node-Glob Scope](./#node-glob-scope).
5. `/n/<brace pattern>` - **Node Brace**. A path that starts with `/n/` followed by a valid [brace pattern](./#brace-pattern). This is essentially the [Node-Brace Scope](./#node-brace-scope). This is much faster than a node-glob scan, and should be the preferred document selection pattern wherever possible.

## Operations

In many queries that the _RecallGraph_ API supports, operations can be applied to transform the results before they are returned. These operations can execute within a running DB query, or during post-processing steps on the query results \(defined in the service code, running in a V8 context\).

The order of operations that are applied to a result set with a request are as follows:

1. **Pre-Filter** \(Applied to individual entries \(events/diffs/nodes\)\)
   1. [Scope](./#scopes) \(Determined by [path](./#path) parameter\)
   2. Transaction-Time Bounds \(Since/Until\)
2. **Group**
   1. Group Sort \(Sorting withing a group\)
   2. Group Slice \(Skip/Limit within a group\)
3. **Sort** \(Grouped/Ungrouped, driven by grouping parameter\)
4. **Slice** \(Skip/Limit\) \(Applied on groups, if grouping parameter present\)
5. **Post-Filter** \(Filter on the array result obtained at the end of step 4\)

All operations are explained in detail in the sub-pages that follow.



