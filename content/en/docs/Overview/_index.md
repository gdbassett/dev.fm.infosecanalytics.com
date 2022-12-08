---
title: "Overview"
linkTitle: "Overview"
weight: 1
description: >
  Find out if Flow Manager is right for you.
---


## What is Flow Manager?

Flow Manager is a graph database and associated API for storing, editing, and retrieving Attack Flows.

## Why do I want it?

While it can be easy to create a flow using [SPARQL](docs/Concepts/sparql) or [Flow Builder](docs/Concepts/flow_builder_v1), it leaves you with a flow, when what you really want are _flows_.  Flow Manager allows you to store all of your flows in a single place and to query them as a dataset.

* **What is it good for?**: The [Tutorials](docs/Tutorials) section of the documentation has several use cases that Attack Flow and Flow Manager are useful for including DFIR, data sharing, detection, attack simulation, executive communication, red teaming, risk assessment, security engineering, and threat intelligence.


* **What is it not good for?**: Graphs do not solve everything.  See Gabriel Bassett's Bluehat 2019 talk [Why Not Graphs?](https://www.youtube.com/watch?v=5xh_HqHtp0Y&t=48s).  There are many situations in which tabular data or non-centralized data is preferable.  In those cases, graphs, and by extension Attack Flow and Flow Manager are not going to be your solution.  Also, Flow Manager specifically is not designed as a general data warehouse, but instead for flows curated by the user.

* **What is it *not yet* good for?**: Currently, Flow Manager is a basic API for storing, modifying, and retrieving flows.  As it expands however, it will add an analysis endpoint to support the various use cases documented in the tutorials.  Future releases will also add a user interface for building, manipulating, and analyzing flows as part of the service and additional self-administration options.  Finally, Flow Builder supports V1 of Attack Flow with V2 coming as STIX implementation of RDF is finalized.

## Where should I go next?

Give your users next steps from the Overview. For example:

* [Getting Started](/docs/getting-started/): Get started with $project

