---
title: "SPARQL UPDATE"
date: 2022-11-04
weight: 4
description: >
  How to update flows using SPARQL queries.
---

{{% pageinfo %}}
Just a foreword, SPARQL, like any query language, can be picky.  http://www.sparql.org/ has several validators you can use to find errors in your queries.
{{% /pageinfo %}}

Some times you may find the need to update your flow graph.  This can be done with the Flow Manager [Modify API](docs/Tasks/modify_flows.md).  Modifications come in the form of INSERT or DELETE queries.

For example, if we previously added an Unidentified author to a flow, we may want to replace them with a named author.  We may know exactly what we want to change and so fully define the triples to change:
```
INSERT DATA 
{ 
  <urn:absolute:flows#flow--41e0cd93-6fb2-4786-94ab-5adec21960cc> <https://attackflow.space/attack-flow#author> <urn:absolute:flows#person--4c49da73> .
  <urn:absolute:flows#person--4c49da73> a <http://xmlns.com/foaf/0.1/Person>, owl:NamedIndividual;
      <http://xmlns.com/foaf/0.1/firstName> "Gabriel";
      <http://xmlns.com/foaf/0.1/family_name> "Bassett";
      <http://xmlns.com/foaf/0.1/workplaceHomepage> "http://infosecanalytics.com" .
}
```
and
```
DELETE WHERE 
{ 
  <urn:absolute:flows#flow--41e0cd93-6fb2-4786-94ab-5adec21960cc> <https://attackflow.space/attack-flow#author> <urn:absolute:flows#Unspecified> .
}
```


But in many cases, you'll want to replace it in a more general way, using variables.  Note, we're going to change the 'INSERT DATA' to 'INSERT' since we're using variables.

```
DELETE {
      ?flow <https://attackflow.space/attack-flow#author> <urn:absolute:flows#Unspecified> .
}
INSERT { 
  ?flow <https://attackflow.space/attack-flow#author> <urn:absolute:flows#person--4c49da73> .
}
WHERE {
      ?flow <https://attackflow.space/attack-flow#author> <urn:absolute:flows#Unspecified> .

}
```

For more about what you can do with insert and delete commands, see the [W3C Documentation](https://www.w3.org/TR/sparql11-update/).