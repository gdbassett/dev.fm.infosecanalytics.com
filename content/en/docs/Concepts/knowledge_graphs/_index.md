---
title: "Knowledge Graphs"
date: 2022-11-04
weight: 1
description: >
  Get some basic information about what knowledge graphs are and how to use them.
---

Attack Flow and Flow Manager are designed to work with knowledge graphs.  Which begs the question, "What's a knowledge graph?" And possibly, "What's a graph?"

![](Picture1.png)

Graphs are pretty simple.  They are made of two things: nodes (dots) and edges (lines).  Edges always start and end in a node.  That's about it for rules for graphs.

Knowledge graphs are a specific type of graph.  It's mostly documented in the [Resource Description Framework (RDF)](https://www.w3.org/RDF/) spec at W3, but lets not start there.  Instead there's three important things to know. In knowledge graphs ...

 * all nodes and edges are IRIs (except the nodes that are Literals)
 * everything is a triple
 * namespaces are used a lot to organize and simplify triples

Lets go into each of these in a bit of detail.


### Internationalized Resource Identifier (IRI)

While there's more to it, [IRIs](https://en.wikipedia.org/wiki/Internationalized_Resource_Identifier) are basically web addresses, so <https://some.namespace.org#the_node_or_edge>.  They can also be things like <urn:absolute:namespace#the_node_or_edge>.  The main point is they have two parts, the namespace (https://some.namespace.org or urn:absolute:namespace in our examples) and a node/edge.  (Ok, so it's more complicated than that.  But by the time you need to know the rest of the stuff, you'll be comfortable with it and you won't be looking for it here.)

The exception that proves the rule: Sometimes triples (see below) point to literals.  These can be a string, a number, really any XSD (see namespaces below) data type.  These are like the value of a property of knowledge graph nodes (since the note below about properties).

### Everything is a triple
A triple is 3 things (surprising! I know!):

 * <urn:absolute:flows#Action1> <rdf:type> <https://attackflow.space/veris#Phishing>
 * Subject Predicate Object
 * Source Relationship Destination
 * Beginning Middle End

There are a few important implications of this:

 * Knowledge graphs are directed. <urn:absolute:flows#Action1> <rdf:type> <https://attackflow.space/veris#Phishing> is not the same as <https://attackflow.space/veris#Phishing> <rdf:type> <urn:absolute:flows#Action1>.
 * Nothing has properties.  There are other graphs where a node or even an edge might have a dictionary of properties, but not knowledge graphs.  That said, it's easy to represent properties.  Instead of something like Node1 {'a': 1, 'b': 2}, you just create triples: Node1 - a - 1, Node1 - b - 2.

 ### Namespaces

 Namespaces are a critical part of knowledge graphs.  They help group like data and explain _who_ is doing the defining.  You can find the V1 knowledge graph version of the attack flow schema at attackflow.space. For example, in the previous section, we mentioned "veris.attackflow.space".  This is a namespace to represent [VERIS](https://github.com/vz-risk/veris) as a knowledge graph.  An organization might store all of it's flows as "urn:absolute:flows".

 Additionally, namespaces let you use other folks work rather than reinventing the wheel.  There are a huge number of namespaces structuring all types of data about the world.  You can read about some of the most popular namespaces at http://richard.cyganiak.de/blog/2011/02/top-100-most-popular-rdf-namespace-prefixes/ but one thing to keep in mind is most of the time a namespace defines a TON of stuff when you probably only need one or two things from it.   The following namespaces are useful or used in attack flow, but usually only for one or two things:

 * RDF (Resource Description Framework) - type (defines what type a node is)
 * RDFS (Resource Description Framework Schema) - label (the human readable 'name' of a node or edge)
 * OWL (Web Ontology Language) - SameAs (two nodes are the same thing), ObjectProperty (a Predicate that needs to point to another node), DatatypeProperty (a Predicate that needs to point to a Literal), NamedIndividual (a node is an instance of a thing, kinda like an object in a program is an instance of a class, or you're an instance of a person)
 * DC (Dublin Core) - description (helpful for creating a description of something)
 * TIME - timestamps
 * XSD (XML Schema) - data types

It's actually kinda important that you don't go looking too much into these different schemas until you're ready.  They can contain LOTS of information that won't really help you and becomes really confusing really quickly.  Take OWL for example:

 ### A note about OWL

 Frankly, OWL (Web Ontology Language) may be the biggest hindrance to using knowledge graphs.  When people search for RDF or knowledge graphs, OWL often comes up.  OWL provides some useful things (see the list above), but it also provides a LOT of other stuff. Specifically, OWL provides the ability to **reason** over the knowledge graph.  This means things like: "if animal:dog rdf:type animal:mammal and family:Sadie owl:NamedIndividual animal:dog, then family:sadie rdf:type animal:mammal".  It's kinda cool, but the problem is it gets _really_ complicated _really_ quick.  We don't need that.  (And when we do we'll just hit the 'reason' button in our software.) Because of the confusion, skip the OWL documentation until _much_ later.  Just use it for the things (object types, named individuals, etc) that it's useful for now.

