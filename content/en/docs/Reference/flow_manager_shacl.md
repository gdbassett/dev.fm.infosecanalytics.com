+++
title = "Flow Manager SHACL"
type = "swagger"
weight = 1
description = "Reference for the Flow Manager SHACL"
+++

Flow Builder uses SHACL constraints to validate the ingested graph before adding it to the full graph.  [SHACL](https://shacl.org/playground/) is kind of like json-schema, but for graphs.  Graph folks tend to call constraints 'shapes' (kind of like the 'shape' af a node and it's properties).  The shapes below help ensure an Attack Flow graph stays consistent. 

If you haven't seen this file format, it's called Turtle.  It's a really common way to write knowledge graphs.  If you look at the [SPARQL](docs/Concepts/sparql.md) concepts page, you'll see a lot of similarities between SPARQL (the query language) and turtle (the method for writing out graphs).

```
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix af: <https://attackflow.shape/attack-flow#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix schema: <https://schema.org/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .



af:FlowShape a sh:NodeShape ;
  sh:targetClass af:attack-flow;
  sh:class owl:NamedIndividual ;
  sh:property [
    sh:path af:name;
    sh:datatype xsd:string;
    sh:minCount 1;
  ] ;
  sh:property [
    sh:path af:created;
    sh:datatype xsd:dateTime;
    sh:minCount 1;
  ] ;
  sh:property [
    sh:path af:author;
    sh:class foaf:Person;
    sh:minCount 0;
  ] ;
  sh:property [
    sh:path af:description;
    sh:datatype xsd:string;
    sh:minCount 0;
  ] .


af:ActionShape a sh:NodeShape ;
  sh:targetClass af:action ;
  sh:class owl:NamedIndividual ;
  sh:property [
    sh:path af:name;
    sh:datatype xsd:string;
    sh:minCount 0;
  ] ;
  sh:property [
    sh:path af:description;
    sh:datatype xsd:string;
    sh:minCount 0;
  ] ;
  sh:property [
    sh:path af:timestamp;
#    sh:datatype xsd:decimal;
    sh:minCount 0;
    sh:maxCount 1;
    sh:minInclusive 0;
  ] ;
  sh:property [
    sh:path af:reference;
    sh:or (
        [ sh:nodeKind sh:IRI ]
        [ sh:datatype xsd:string ]
    );
    sh:minCount 0;
  ] ;
  sh:property [
    sh:path af:succeeded;
#    sh:datatype xsd:decimal;
    sh:minCount 0;
    sh:maxCount 1;
    sh:minInclusive 0;
    sh:maxInclusive 1;
  ] ;
  sh:property [
    sh:path af:confidence;
#    sh:datatype xsd:decimal;
    sh:minCount 0;
    sh:maxCount 1;
    sh:minInclusive 0;
    sh:maxInclusive 1;
  ] ;
  sh:property [
    sh:path af:logic_operator_language;
    sh:datatype xsd:string;
    sh:minCount 0;
    sh:maxCount 1;
  ] ;
  sh:property [
    sh:path af:logic_operator;
    sh:datatype xsd:string;
    sh:minCount 1;
    sh:maxCount 1;
  ] ;
  sh:property [
    sh:path af:state_change;
    sh:class af:asset
  ] ;
  sh:property [
    sh:path af:flow;
    sh:class af:attack-flow;
    sh:minCount 1;
    sh:maxCount 1;
  ] ;
  sh:sparql [
    a sh:SPARQLConstraint ;   # This triple is optional
    sh:message "Action has property of action." ;
    sh:prefixes af: ;
    sh:select """
      select * where { 
        $this ?path ?value .
          ?value a af:action .
      } limit 100 
      """ ;
  ] .

af:AssetShape a sh:NodeShape ;
  sh:targetClass af:asset ;
  sh:class owl:NamedIndividual ;
  sh:property [
    sh:path af:description;
    sh:datatype xsd:string;
    sh:minCount 0;
  ] ;
  sh:property [
    sh:path af:state;
    sh:or (
        [ sh:nodeKind sh:IRI ]
        [ sh:datatype xsd:string ]
    );
    sh:minCount 0; # was 1, but 0 in schema
  ] ;
  sh:property [
    sh:path af:state_requirement;
    sh:class af:action
  ] ;
  sh:property [
    sh:path af:flow;
    sh:class af:attack-flow;
    sh:minCount 1;
    sh:maxCount 1;
  ] ;
  sh:sparql [
    a sh:SPARQLConstraint ;   # This triple is optional
    sh:message "Asset has property of asset." ;
    sh:prefixes af: ;
    sh:select """
      select * where { 
        $this ?path ?value .
          ?value a af:asset .
      } limit 100 
      """ ;
  ] .

```