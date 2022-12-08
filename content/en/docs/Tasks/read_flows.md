---
title: "Read Flows"
date: 2022-11-04
weight: 3
description: >
  How to read flows and context stored in Flow Manager.
---

Reading flows can be challenging.  With graphs, there is not necessarily a specific place to 'start' the way there might be with a JSON tree.  Instead you need to pick a starting place based on the question you want to answer.
 * Do you want to retrieve a specific flow?  Look it up by IRI.  You forgot the IRI? Humm, maybe by name.  The name was something rather generic that you either don't remember or there are multiple of?  Maybe look it up by created time. (The time the flow was created, not the time it was uploaded or it actually happened.)  Still not sure what it was? You're going to have to do some searching, either by properties, or matching text in descriptions and such.
 * Do you want to collect general flows? Maybe try looking for nodes with no parents (i.e. edges that point to them)
 * Do you want to know more about a specific action? start with that node in your graph (maybe https://attackflow.spaces/veris#Phishing)


## GET /flows API

The API is very similar to the [POST API](docs/Tasks/read_flows.md).

```
get_body = {
    "token": TOKEN,
    "version": "0.0.1",
    "graph": "default",
    "flow": query_sparql
}
```

`token` A random string used to identify the user.

`version` is the version of the API.  The current version of the API can be found in the [API Reference](docs/Reference/flow_manager_api.md).

`graph` The graph is the named graph to store into.  Flow Manager supports multiple distinct graph per token, however all tokens have a 'default' graph.

`flow` A SPARQL string query reading data. (INSERT, UPDATE, and DELETE queries will not be accepted.) See the [SPARQL Concepts](docs/Concepts/sparql.md) page for more details on SPARQL queries.

The difference from the POST API is that `builder`, `best_practices`, and `attack_flow_version` will be ignored if provided.

## GET /flows resposne

The returned response will return a JSON response.

```
{'id': 'deee77fe-d73c-4f34-843e-937e5132637e',
 'result': ...,
 'success': True,
 'timestamp': '2022-12-07T22:23:09.098009'}
 ```

All responses will contain a few things:
`id` A unique ID of the query
`timestamp` A timestamp for the query
`success` Logical value on if the query was successful or not
`result` More detail about the result. If successful, it will be the JSON SPARQL response stringified (requiring something such as `json.loads(x.json()['result'])` to read.)

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

## SPARQL responses

Lets assune you've already loaded the json string from the result (`result = json.loads(x.json()['result'])`).

SPARQL does not return a graph.  It can return information that can be formed back into a graph such as with `select DISTINCT ?s ?p ?o ...`, but it can also return more tabular data (for example `select DISTINCT ?flow ?name ?description ?author ...`).

Because of this, it's on the user to parse the response in the way appropriate to what they requested.

```
{
  "head": {
    "vars": [...]
  },
  "results": {
    "bindings": [...]
  }
}
```

in general, sparql JSON results are a dictionary of two lists.  Varialbes is a list of strings representing the variable names in the `SELECT ...` statement (For example `"vars": ["s", "r", "d"]`) while bindings is the actual data.  Each item in "bindings" is an object with keys for the variables and values of objects containing a 'type' and 'value', for example:
```
{
  "s": {
    "type": "uri",
    "value": "urn:absolute:flows#action_653f8cfb-dce0-40b9-89af-1af9c4172340"
  },
  "r": {
    "type": "uri",
    "value": "https://attackflow.space/attack-flow#flow"
  },
  "d": {
    "type": "uri",
    "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
  }
}
```

The above item from the binding list represents the triple (<urn:absolute:flows#action_653f8cfb-dce0-40b9-89af-1af9c4172340>, <https://attackflow.space/attack-flow#flow>, <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97>).

If the SPARQL query returns triples, it can then be used to recreate the graph.