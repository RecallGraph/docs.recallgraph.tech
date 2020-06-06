---
description: A record of what changed between successive versions
---

# Changelog

## 1.0.0

### Added

1. Post-filter in _read_ endpoints - `log`, `diff`, `show`.
2. _k Shortest Paths_ - Custom-weighted, point-in-time, shortest paths between endpoints.
3. Synchronous skeleton graph updates within commit transactions.
4. [OpenTracing](https://opentracing.io/) instrumentation, using ancillary [collector](https://github.com/RecallGraph/foxx-tracer-collector) foxx service. **Non-intrusive and strictly optional.**
5. Logs can be grouped by type \(vertex/edge\).
6. _Explicit Commits_ to sync event log with writes that occurred outside of RecallGraph's API methods.
7. Providers for all API endpoints to let other dependent Foxx services invoke RecallGraph's service methods directly, using ArangoDB's [service linking](https://www.arangodb.com/docs/stable/foxx-guides-dependencies.html) mechanism.
8. _Purge_ endpoint to remove all history for nodes at a specified path.
9. _Restore_ endpoint to **undelete** nodes that were deleted through RecallGraph's API.
10. Paths are returned in _traverse_ calls, with support for path filters.
11. Support for slicing in _show_ endpoint when aggregation is enabled.
12. **Data:**
    1. Event objects have a `collection` field to identify the collection to which their corresponding document belongs.
    2. Skeleton graph objects have a `collection` field to identify the collection to which their corresponding document belongs.
    3. Origin events have `collection` and `ctime` fields to allow building dense indexes on them.
    4. Migration script to automatically upgrade event and skeleton trees created using older versions.
    5. Migration script to remove stale indices.

### Removed

1. Precision limiters for timestamp parameters.
2. Dedicated `filter` endpoint. All read endpoints now have their own filtering capability.
3. Asynchronous skeleton graph updates in cron jobs.
4. **Breaking:**
   1. `returnCommands` parameter removed from `log`. Anything that depended on this should now use the enhanced `diff` endpoint instead.
   2. `groupSkip` and `groupLimit` removed from `diff` endpoint, as they were deemed _not useful_. The newly available post-filter param can indirectly help make up for the deficit in most cases.
   3. Service collection suffixes are no longer configurable \(extraneous config\).
5. **Bug**:
   1. Sort direction was not being honored in `log` when `groupBy` was specified. This has been fixed.
   2. Hard-coded service collection name in a query for `show` was removed.
   3. `traverse` no longer breaks if starting vertex is not found.
   4. Fixed operation id for the `UPDATE` route, which was incorrectly marked as `replace`.

### Changed

1. _Document_ operations are checked first for compatible last events before executing.
2. **Data:**
   1. Collection origin events have their `origin-for` field renamed to `collection` to remain consistent with field naming convention followed for regular events.
   2. Certain indices have changed - primarily, sparse indexes have been replaced by their dense counterparts.
   3. When replacing/updating an existing document with no event log record, a `created` event is recorded instead of an `updated` event.
3. **Breaking:**
   1. Diffs are enhanced to return some event metadata along with command lists. This is to reduce overloading the `log` function with event+command-related queries.
   2. The `countsOnly` parameter now has an effect even when `groupBy` is `null` for `log` endpoints. It returns the overall total number of events based on the filters provided.
   3. `show` returns an array with total count when `groupBy` is `null`, instead of a bare object. This is for consistent post-filter application semantics.
   4. `traverse` `minDepth` has been changed from `0` to `1`.

