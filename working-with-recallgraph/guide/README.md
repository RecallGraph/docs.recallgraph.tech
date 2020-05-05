---
description: A step-by-step guide to using RecallGraph's features.
---

# Guide

## Before You Begin

Before starting to work with RecallGraph, take a few minutes to go through the background and terminology to set the context for understanding the terms used in this guide. To make things easier, essential concepts and terms used in this guide will be linked to their respective documentation.

## Initial Setup

Before starting with the guide, you need to have RecallGraph installed on a server you can access \(e.g. your `localhost`\). Follow the instructions provided in the link below:

{% page-ref page="../installation.md" %}

After installation, once you login to the web console, you should have your service listed \(possibly among other installed services\) in the _SERVICES_ tab as shown below:

![Service mounted under /recallgraph](../../.gitbook/assets/screenshot_2020-05-05_19-10-37.png)

## Structure of this Guide

RecallGraph works by keeping an internal log of all changes that its tracked documents \(vertices/edges\) have gone through. Based on this log, it supports point-in-time \(i.e. historical\) lookbacks, traversals and even weighted shortest path queries.

To align with the logical sequence of entering data, scanning the logs to identify interesting events and time points, and finally running historical graph and document queries, the guide is structured into the following sections in order:

{% page-ref page="persisting-documents.md" %}

{% page-ref page="analyzing-the-event-log.md" %}

{% page-ref page="navigating-history.md" %}



