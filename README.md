---
description: RecallGraph - A versioning data store for time-variant graph data.
---

# Introduction

{% embed url="https://github.com/RecallGraph/RecallGraph" caption="Project Source @ Github" %}

[RecallGraph](https://github.com/RecallGraph/RecallGraph) is a _versioned-graph_ data store - it retains all changes that its data \(vertices and edges\) have gone through to reach their current state. It supports _point-in-time_ graph traversals, letting the user query any past state of the graph just as easily as the present.

It is a [Foxx Microservice](https://www.arangodb.com/why-arangodb/foxx/) for [ArangoDB](https://www.arangodb.com/) that features _VCS-like_ semantics in many parts of its interface, and is backed by a transactional event tracker. It is currently being developed and tested on ArangoDB v3.5 and v3.6, with support for v3.7 in the pipeline.

[![Build Status](https://travis-ci.org/RecallGraph/RecallGraph.svg?branch=development)](https://travis-ci.org/RecallGraph/RecallGraph) [![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=adityamukho_evstore&metric=alert_status)](https://sonarcloud.io/dashboard?id=adityamukho_evstore) [![Coverage](https://sonarcloud.io/api/project_badges/measure?project=adityamukho_evstore&metric=coverage)](https://sonarcloud.io/component_measures?id=adityamukho_evstore&metric=coverage) [![Maintainability Rating](https://sonarcloud.io/api/project_badges/measure?project=adityamukho_evstore&metric=sqale_rating)](https://sonarcloud.io/dashboard?id=adityamukho_evstore) [![Reliability Rating](https://sonarcloud.io/api/project_badges/measure?project=adityamukho_evstore&metric=reliability_rating)](https://sonarcloud.io/dashboard?id=adityamukho_evstore) [![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=adityamukho_evstore&metric=security_rating)](https://sonarcloud.io/dashboard?id=adityamukho_evstore)

## Do I Need a 'Versioned Graph' Database?

To get an idea of where such a data store might be used, see:

1. [The Case for Versioned Graph Databases](https://adityamukho.com/the-case-for-versioned-graph-databases/),
2. [Illustrative Problems in Dynamic Network Analysis](https://en.wikipedia.org/wiki/Dynamic_network_analysis#Illustrative_problems_that_people_in_the_DNA_area_work_on)

Also check out the recording below:

{% embed url="https://www.youtube.com/watch?v=UP2KDQ\_kL4I" caption="RecallGraph presented @ ArangoDB Online Meetup" %}

{% embed url="https://docs.google.com/presentation/d/1FHNfMxNnBiR4dXdqVJqInTiXdmX-9dSEtUrHMsw-O0E/edit\#slide=id.p" caption="The associated slide deck" %}

**TL;DR:** RecallGraph is a potential fit for scenarios where data is best represented as a network of vertices and edges \(i.e., a graph\) having the following characteristics:

1. Both vertices and edges can hold properties in the form of attribute/value pairs \(equivalent to JSON objects\).
2. Documents \(vertices/edges\) mutate within their lifespan \(both in their individual attributes/values and in their relations with each other\).
3. Past states of documents are as important as their present, necessitating retention and queryability of their change history.

## API Features

RecallGraph's API is split into 3 top-level categories:

### Document

* **Create** - Create single/multiple documents \(vertices/edges\).
* **Replace** - Replace entire single/multiple documents with new content.
* **Delete** - Delete single/multiple documents.
* **Update** - Add/Update specific fields in single/multiple documents.
* **\(Planned\) Explicit Commits** - Commit a document's changes separately, after it has been written to DB via other means \(AQL / Core REST API / Client\).
* **\(Planned\) CQRS/ES Operation Mode** - Async implicit commits.

### Event

* **Log** - Fetch a log of events \(commits\) for a given path pattern \(path determines scope of documents to pick\). The log can be optionally grouped/sorted/sliced within a specified time interval.
* **Diff** - Fetch a list of forward or reverse commands \(diffs\) between commits for specified documents.
* **\(Planned\) Branch/Tag** - Create parallel versions of history, branching off from a specific event point of the main timeline. Also, tag specific points in branch+time for convenient future reference.
* **\(Planned\) Materialization** - Point-in-time checkouts.

### History

* **Show** - Fetch a set of documents, optionally grouped/sorted/sliced, that match a given path pattern, at a given point in time.
* **Filter** - In addition to a path pattern like in **'Show'**, apply an expression-based, simple/compound post-filter on the retrieved documents.
* **Traverse** - A point-in-time traversal \(walk\) of a past version of the graph, with the option to apply additional post-filters to the result.
* **k Shortest Paths** - Point-in-time, weighted, shortest paths between two endpoints.
* **\(Planned\) Purge** - Delete all history for specified nodes.

## Limitations

1. Although the test cases are quite extensive and have good coverage, this service has only been tested on single-instance DB deployments, and **not on clusters**.
2. As of version 3.6, ArangoDB does not support ACID transactions for multi-document/collection writes in [cluster mode](https://www.arangodb.com/docs/3.6/transactions-limitations.html#in-clusters). Transactional ACIDity is not guaranteed for such deployments.

## Development Roadmap

1. Support for absolute/relative revision-based queries on individual documents \(in addition to the timestamp-based queries supported currently\),
2. Branching/tag support,
3. Support for the _valid time_ dimension in addition to the currently implemented _transaction time_ dimension \([https://www.researchgate.net/publication/221212735\_A\_Taxonomy\_of\_Time\_in\_Databases](https://www.researchgate.net/publication/221212735_A_Taxonomy_of_Time_in_Databases)\),
4. Support for ArangoDB v3.7,
5. Multiple, simultaneous materialized checkouts \(a la `git`\) of selectable sections of the database \(entire DB, named graph, named collection, document list, document pattern\), with eventual branch-level specificity,
6. CQRS/ES operation mode \(async implicit commits\),
7. Explicit commits,
8. Support for ArangoDB clusters \(limited at present by lack of support for multi-document ACID transactions in clusters\).
9. Multiple authentication and authorization mechanisms.

## Get in Touch

* Raise an issue or PR on the [project repository](https://github.com/RecallGraph/RecallGraph), or
* Mail me \(![Email Link](http://safemail.justlikeed.net/e/aa7232bbfc22c7580ae7a4b561562e0b.png)\), or
* Join the [Gitter channel](https://gitter.im/RecallGraph/community).

{% hint style="warning" %}
## 🚫Disclaimer

The authors and maintainers of RecallGraph are not liable for damages or indemnity \(express or implied\) for loss of any kind incurred directly or indirectly as a result of using this software.
{% endhint %}

## 

