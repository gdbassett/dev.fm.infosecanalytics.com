---
title: "SPARQL CREATE"
date: 2022-11-04
weight: 2
description: >
  How to create flows using SPARQL queries.
---

Inserting data into Flow Manager is actually relatively easy using SPARQL.  You might find, in fact, that it's easier to write flows in SPARQL than other formats, especially for longer flows.  

The following example is the start (the flow node, first action, and first asset) of a larger flow that highlights how SPARQL can be used for writing flows:

```
PREFIX owl: <http://www.w3.org/2002/07/owl#> 
PREFIX af: <https://attackflow.space/attack-flow#>
PREFIX flows: <urn:absolute:flows#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX veris: <https://attackflow.space/veris#>
INSERT DATA 
{
    flows:example_flow4 rdf:type owl:NamedIndividual, af:attack-flow ;
                      af:created "2022-04-12T13:52:00" ;
                      af:name "example_flow4" .
    flows:action_65f9c664 rdf:type owl:NamedIndividual, af:action,  veris:Use%20of%20stolen%20creds ;
               veris:action.hacking.vector veris:action.hacking.vector.Desktop%20sharing%20software ;
               af:description "actor logs in to a desktop using RDP & stolen creds" ;
               af:logic_operator "AND" ;
               af:name "action1" ;
               af:flow flows:example_flow4 ;
               af:timestamp "1970-01-01T00:00:01" ;
               af:state_change flows:asset_1024b648  .
    flows:asset_1024b648 rdf:type owl:NamedIndividual, af:asset, veris:Desktop ;
               af:flow flows:example_flow4 ;
               af:state_requirement flows:action_e084e1f3, flows:action_3055c8f7-1,  
               						flows:action_3055c8f7-2, lows:action_3055c8f7-3, 
               						flows:action_3055c8f7-4, flows:action_3055c8f7-5, 
               						flows:action_3055c8f7-6, flows:action_3055c8f7-7, 
               						flows:action_a857f6c3 .
}
```

Instead of the `SELECT` query we used to retrieve data, we use `INSERT DATA`.  We can effectively write each node as the edges going from it to other nodes or literals.

```
flows:example_flow4 rdf:type owl:NamedIndividual, af:attack-flow ;
                  af:created "2022-04-12T13:52:00" ;
                  af:name "example_flow4" .
```

For the flow node we can see the first value of the triples is `flows:example_flow4`.  But on the first line we see something curious.  It's not three IRIs long.  Instead we see a `,` between the last two IRIs.  **The `,` effectively means "we're going to have another triple where the first two IRIs are the same."**  The line (and in fact the second line as well) ends with a `;`.  **The  `;` means "we're going to have another  triple where the first IRI is the same."** You can almost think of it as creating a tree (like JSON) that's always three values deep.  Finally once we're done with the triples for this node, we use `.` to indicate the end of the line.  You don't  have to use `,` and `;`.  You could write each triple separately (ending each line in `.`) or repeat the the second and third IRI each time (ending in `;`), but the short hand speeds things up

We then repeat the same process for the next node (flows:action_65f9c664), the following asset (flows:asset_1024b648) and on down the attack graph.

It's fast, easy, and clean.