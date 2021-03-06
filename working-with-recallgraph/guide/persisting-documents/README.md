---
description: 'Creating, updating and deleting vertices and edges.'
---

# Persisting Documents

RecallGraph can only track history for nodes \(vertices /edges\) that are persisted through its _write_ APIs. This is because it needs to update the event log every time a node is written \(created / updated / deleted\). If a node is written through any means outside of RecallGraph's scope, it has no way of knowing, and hence its event log would fall out of sync with the node's data.

RecallGraph supports 4 write endpoints - one each for create, update, replace and delete operations respectively. These endpoints support both individual as well as bulk node writes, and are purposely kept semantically as close to ArangoDB's core CRUD REST APIs as possible.

## **The Story**

All of RecallGraphs APIs are explored in this guide with the aid of a fictional \(but entirely believable\) organization whose org structure will be tracked using RecallGraph as the backend.

> Eric heads the manufacturing unit of _Acme Corporation_ and has just been informed of a purchase order for 10000 explosive tennis balls by one of their most valuable high-net-worth individual customers - a certain _Wile E. Coyote_. Eric already has a trusty team handling the production unit, headed by a team of joint unit managers - Stan and Kyle. They have, in the past, successfully enabled Acme Inc. to fulfill custom orders of similar size \(usually, anvils\) on time, and Eric places a great deal of faith in their ability to deliver this once as well.
>
> However, explosive tennis balls have a tendency to - well - explode, and so, Eric, out of concern for the safety of his production team \(and to comply with safety regulations\), decides to hire a _Safety Officer_ who specializes in handling hazardous chemicals and flammable substances. After a quick search, they finalize on Kenny who has just recently been relieved of his duties at the last plant he was assigned to, since it had to shut down due to an unexplained fire that, as Kenny claimed, happened while he was on vacation. Eric gives him the benefit of the doubt and brings him on board.

To align with the logical sequence of entering data, the reader is advised to go through the following sections, in order:

{% page-ref page="create.md" %}

{% page-ref page="replace.md" %}

{% page-ref page="update.md" %}

{% page-ref page="delete.md" %}

