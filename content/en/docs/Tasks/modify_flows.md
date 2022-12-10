---
title: "Modify Flows"
date: 2022-11-04
weight: 2
description: >
  How to modify flows and context stored in Flow Manager.
---

{{% pageinfo %}}
Just a foreword, SPARQL, like any query language, can be picky.  http://www.sparql.org/ has several validators you can use to find errors in your queries.
{{% /pageinfo %}}


The Modify API is for making changes to the flow graph.  It can be used to add, change, or remove from the graph.

{{% pageinfo %}}
**WARNING** The modify API is more like a SQL interface than the write API.  It is possible with the modify API to do general things that SPARQL allows.  This includes destructive changes.  Be careful when using!
{{% /pageinfo %}}

The modify API is very similar to the other APIs with the difference that it uses a PUT request.

## PUT /flows API

To store a flow, you'll need two things:
 * A valid SPARQL query
 * an API token and associated graph

 The key to the modify API is the SPARQL request.  It will likely take the form of an INSERT clause to add data to the graph or a DELETE clause to remove data from the graph.  The modify API will not accept SELECT clauses. See the [SPARQL Updates](docs/Concepts/sparql_update.md) page for ideas on how to update the graph.

 {{% pageinfo %}}
**NOTE** Due to a platform limitiation, PREFIX statements do not work in the modify API.  Instead, use the `initNs` parameter to list namespaces that will be replaced in the query body itself.
{{% /pageinfo %}}

 Queries are passed as JSON in the body of a PUT request.

```
{
    "token": TOKEN,
    "version": "0.0.1",
    "graph": "default",
    "flow": sparql,
    "initNs": {"foaf": "http://xmlns.com/foaf/0.1/"}
}
```

`token` A random string used to identify the user.

`version` is the version of the API.  The current version of the API can be found in the [API Reference](docs/Reference/flow_manager_api.md).

`graph` The graph is the named graph to store into.  Flow Manager supports multiple distinct graph per token, however all tokens have a 'default' graph.

`initNs` Optionally, rather than specify PREFIXes in the SPARQL query, a dictionary of namespaces can be supplied to replace in the SELECT statement itself.  The dictionary takes the form `{"foaf": "http://xmlns.com/foaf/0.1/"}` for example, then allowing `foaf:workplaceHomepage \"http://infosecanalytics.com\"` to be used in the query.  If not using, do not include in the query.

The difference from the POST API is that `builder`, `best_practices`, and `attack_flow_version` will be ignored if provided.


## PUT /flows Responses

The returned response will return a JSON response.

```
{
    "id": "aec7ccd5-d4cb-48a1-882f-23fa765c400c",
    "result": "Update succeeded.",
    "success": true,
    "timestamp": "2022-12-10T02:28:42.268548"
}
 ```

All responses will contain a few things:
`id` A unique ID of the query
`timestamp` A timestamp for the query
`success` Logical value on if the query was successful or not
`result` More detail about the result.  

```
{'error': {'message': 'You did something wrong formulating either the URI or '
                      'your SPARQL query',
           'type': 'query_error'},
 'id': '9ab51e53-97de-408f-ae62-88bbe3c0a2f2',
 'result': 'error',
 'success': False,
 'timestamp': '2022-12-08T20:57:17.330628'}
 ```

In case of an error, the return will include an `error` property containing an object with both the error message and the type of error.

## EXAMPLE

Mirroring the example on the [Update Concepts](docs/Concepts/sparql_update.md) page, we can implement similar queries in the Flow Manager API to replace Unspecified authors with a named one.

First we'll insert the author:
```
{
    "token": TOKEN,
    "version": "0.0.1",
    "graph": "default",
    "flow": "INSERT DATA \
{ \
  flows:flow--41e0cd93-6fb2-4786-94ab-5adec21960cc af#author flows:person--4c49da73 .\
  <urn:absolute:flows#person--4c49da73> a foaf:Person, owl:NamedIndividual;\
      foaf:firstName "Gabriel";\
      foaf:family_name "Bassett";\
      foaf:workplaceHomepage "http://infosecanalytics.com" .\
}",
    "initNs": {"foaf": "http://xmlns.com/foaf/0.1/", 
               "flows": "urn:absolute:flows#",
               "owl": "http://www.w3.org/2002/07/owl#",
               "af": "https://attackflow.space/attack-flow#"}
}
```

Note that we don't use PREFIX clauses, but instead initNs's.  We can then DELETE the previous Unidentified authors and connect the flows to the new author.
```
{
    "token": TOKEN,
    "version": "0.0.1",
    "graph": "default",
    "flow": "DELETE {\
      ?flow flows:author flows:Unspecified .\
} \
INSERT { \
  ?flow flows:author flows"person--4c49da73 .\
} \
WHERE {\
      ?flow flows:author flows:Unspecified .\
}",
    "initNs": {"foaf": "http://xmlns.com/foaf/0.1/", 
               "flows": "urn:absolute:flows#",
               "owl": "http://www.w3.org/2002/07/owl#",
               "af": "https://attackflow.space/attack-flow#"}
}
```