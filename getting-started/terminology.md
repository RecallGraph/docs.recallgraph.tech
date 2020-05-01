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
5. Distance from last [snapshot](terminology.md#snapshot).
6. Distance to next [snapshot](terminology.md#snapshot) \(which may or may not exist yet\).

### Diff

A reversible delta between two JSON objects, computed using the [JSON Patch \(RFC6902\)](https://tools.ietf.org/html/rfc6902) standard. This delta itself is represented as a JSON array of objects. For more information on its schema/format, refer to the aforementioned link.

### Command

A container for a single \(forward\) [Diff](terminology.md#diff) array. This is the storage unit for the difference between two successive [events](terminology.md#event) that occur on a single document \(vertex/edge\). The command also contains a reference to the `_id` of the document whose change it records. Every event is associated with a corresponding command that records the changes caused by that event.

#### Special Cases

1. When a document is first created, its command diff is computed from an empty JSON object \(`{}`\) to the document's JSON representation.
2. When a document is deleted, its command diff is computed from the document's last JSON representation to an empty JSON object \(`{}`\).

### Snapshot

After every configurable number of write [events](terminology.md#event) on a document, a snapshot of the current state of the document is recorded in its entirety. This can later be used to quickly rebuild a document's state at a particular point in the past, without having to replay all the events starting from the creation of the document up to that instant.

Instead, if a nearby snapshot is found \(before or after the instant\), it can be used as a starting point from where to replay the events, thus saving some computation time. In case the snapshot is in the future w.r.t. the instant asked for, the events are replayed in reverse chronological order, with their corresponding [diffs](terminology.md#diff) being first reversed before being applied.

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

This scope restricts the query to only look within the collections that belong to one or more \(named\) graphs whose names match a specified [glob pattern](terminology.md#glob-pattern) \(barring _RecallGraph_'s own service graphs\). The pattern is passed as a query parameter.

### Collection Scope

This scope restricts the query to only look within collections whose names match a specified [glob pattern](terminology.md#glob-pattern) \(barring _RecallGraph_'s own service collections\). The pattern is passed as a query parameter.

### Node-Glob Scope

This scope restricts the query to only look within those documents \(vertex/edge\) whose `_id` matches a specified [glob pattern](terminology.md#glob-pattern) \(barring documents within _RecallGraph_'s own service collections\). The pattern is passed as a query parameter.

### Node-Brace Scope

This scope restricts the query to only look within those documents \(vertex/edge\) whose `_id` matches a specified [brace pattern](terminology.md#brace-pattern) \(barring documents within _RecallGraph_'s own service collections\). The pattern is passed as a query parameter.

## Path

A pattern used to select a particular scope to scan. This is not to be confused with a [graph traversal path](https://www.arangodb.com/docs/stable/aql/graphs-traversals-explained.html). In this context, _path_ refers to the UNIX directory-like navigation pointers that the patterns resemble. These are defined as follows:

1. `/` - **The root path**. This tells the log builder to basically pick everything - the entire database for logging - every user-defined collection and every document \(vertex/edge\) \(existing and deleted\) therein. This is essentially the [Database Scope](terminology.md#database-scope).
2. `/g/<glob pattern>` - **Named Graph**. A path that starts with `/g/` followed by a valid [glob pattern](terminology.md#glob-pattern). This is essentially the [Graph Scope](terminology.md#graph-scope).
3. `/c/<glob pattern>` - **Collection**. A path that starts with `/c/` followed by a valid [glob pattern](terminology.md#glob-pattern). This is essentially the [Collection Scope](terminology.md#collection-scope).
4. `/ng/<glob pattern>` - **Node Glob**. A path that starts with `/ng/` followed by a valid [glob pattern](terminology.md#glob-pattern). This is essentially the [Node-Glob Scope](terminology.md#node-glob-scope).
5. `/n/<brace pattern>` - **Node Brace**. A path that starts with `/n/` followed by a valid [brace pattern](terminology.md#brace-pattern). This is essentially the [Node-Brace Scope](terminology.md#node-brace-scope). This is much faster than a node-glob scan, and should be the preferred document selection pattern wherever possible.

## Filter

In many queries that the _RecallGraph_ API supports, filters can be applied to restrict the results that are returned. These filters are classified according to whether they apply within a running DB query, or during post-processing steps on the query results \(defined in the service code, running in a V8 context\).

### Pre-Filter

A pre-filter is a filter that is applied at the time of running a DB query.

Pre-filters supported by _RecallGraph_ include the [path]() parameter and the time interval bounds that several API endpoints accept.

### Post-Filter

Once a query returns some results, a post-filter can be applied on them to further restrict the number of matching results that are returned. These filters are executed within service code executed in a V8 context.

The only type of post-filter supported by _RecallGraph_ is a string containing any Javascript-like expression that is supported by [jsep](http://jsep.from.so/), with a few extensions defined below.

**Important:** _jsep_ does not support object literals, only array literals. This may be fixed in a fork, maintained by _RecallGraph_, in the future.

#### _jsep_ Extensions

**Operators**

1. `=~` - **Regex Match**. Filters on a regex pattern. The left operand must [match the regex pattern](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test) given in the right operand for the filter to pass.
2. `=*` - **Glob Match**. Filters on a [glob pattern](). The left operand must match the glob pattern given in the right operand for the filter to pass.
3. `in` - **Array Search**. Filters on an array search. The left operand must be present in the array provided in the right operand. **Deep comparison** is performed for each element of the right operand against the left operand.
4. `**` - **Exponentiation**. Returns the left operand raised to the power right operand.

**Special Handling for ==, ===, != and !==**

1. `==, ===` - **Deep Equality**. Filters on deep equality. The left operand must be **deeply equal** to the right operand. Both `==` and `===` behave identically.
2. `!=, !==` - **Deep Inequality**. Filters on deep inequality. The left operand must **NOT** be **deeply equal** to the right operand. Both `!=` and `!==` behave identically.

**Functions**

**Binary Functions**

1. `eq(left, right)` - **Deep Equality**. Filters on deep equality. See [https://lodash.com/docs/4.17.15\#isEqual](https://lodash.com/docs/4.17.15#isEqual).
2. `lt(left, right)` - **'Less Than' Comparison**. Filters on strict inequality. The `left` operand must be strictly less than the `right` operand. See [https://lodash.com/docs/4.17.15\#lt](https://lodash.com/docs/4.17.15#lt).
3. `gt(left, right)` - **'Greater Than' Comparison**. Filters on strict inequality. The `left` operand must be strictly greater than the `right` operand. See [https://lodash.com/docs/4.17.15\#gt](https://lodash.com/docs/4.17.15#gt).
4. `lte(left, right)` - **'Less Than or Equals' Comparison**. Filters on non-strict inequality. The `left` operand must be less than or equal to the `right` operand. See [https://lodash.com/docs/4.17.15\#lte](https://lodash.com/docs/4.17.15#lte).
5. `gte(left, right)` - **'Greater Than or Equals' Comparison**. Filters on non-strict inequality. The `left` operand must be greater than or equal to the `right` operand. See [https://lodash.com/docs/4.17.15\#gte](https://lodash.com/docs/4.17.15#gte).
6. `typeof(val)` - **Type Identifier**. Returns the type of `val` as per the specification defined at [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof).
7. `in(needle, haystack)` - **Array Search**. Filters on an array search. The `needle` operand must be present in the array provided in the `haystack` operand. **Deep comparison** is performed for each element of `haystack` against `needle`.
8. `glob(string, pattern)` - **Glob Match**. Filters on a [glob pattern](). The `string` operand must match the glob pattern given in the `pattern` operand for the filter to pass.
9. `regx(string, pattern)` - **Regex Match**. Filters on a regex pattern. The `string` operand must [match the regex pattern](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test) given in the `pattern` operand for the filter to pass.

**Ternary Functions**

1. `all(op, array, value)` - **Array 'All Match'**. Filters on an array 'all match'. Every element `el` in `array` must satisfy the `op(el, value)` function filter, where `op` is one of the above [binary function filters]().
2. `any(op, array, value)` - **Array 'Any Match'**. Filters on an array 'any match'. At least one element `el` in `array` must satisfy the `op(el, value)` function filter, where `op` is one of the above [binary function filters]().

**Special Handling for Math.\* - EXPERIMENTAL!**

All exposed properties and methods of the built-in [Math](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math) object are correctly recognized and processed.

A consequent **CAVEAT** is that in the intermediate result array \(the DB query results\), any object with field/s \(top-level or nested\) having the name `Math` will **NOT** be processed normally. This behaviour is experimental, and subject to change in future versions.

