---
title: "ATTACK FLOW"
date: 2022-11-04
weight: 2
description: >
  The basics of what Attack Flow is and how it can be used.
---

[Attack Flow](https://github.com/center-for-threat-informed-defense/attack-flow) is a standard for graphs in information security.  An attack flow is a machine-readable representation of a sequence of actions and assets, plus knowledge properties, joined by relationships.

[](attack_flow1.jpg)

Attack Flow has 5 parts: Actions, Assets, Properties (Objects/Data) & Relationships, all joined through a Flow.
 * Actions are things that happen
 * Assets are things that have state changes
 * Properties are contextual triples (X -[Property of]-> Y) describing either other 'things' or data like a hash
 * Relationships create the causal and contextual connections
 * Flows are the set of Actions, Assets, Properties, and Relationships

[](attack_flow2.png)

When we combine graphs and the 5 parts, we get a flow.

Attack Flow provides the 3 C's: Causality, Context, and Complexity
 * Causality is the causal path from action to asset and asset to action. It's what caused what
 * Context are the properties (both literal properties like description strings and object properties like a 'person' (with a name, job title, permissions, etc)) that describe other nodes
 * Complexity is the ability to represent complex relationships

You’ll notice the causal path goes action->asset and asset->action.  This is intentional.  It forces good data modeling, helps represent complex relationships, and makes visualization easier.

This helps solve several data challenges in information security:

 * Lots of text  ->  Broken up into clear parts (nodes) and relationships (edges)
 * Structured data has steep learning curve  ->  Only learn what you need (node-centric search and namespaces for specialized information)
 * No focus on relationships  ->  Relationships are first-class
 * Little causality  -> Explicit structure for causality
 * No context  ->  Knowledge Graph
