---
title: "Write Flows"
date: 2022-11-04
weight: 1
description: >
  How to upload a flow, written in json-schema, RDF, or SPARQL, to Flow Manager.
---

The first task you'll take in Flow Manager is to store a flow.

## POST /flows API

To store a flow, you'll need two things:
 * A flow
 * an API token and associated graph

The flow can take three forms:
 * JSON that conforms to the Attack Flow json-schema.  (Currently, Flow Manager supports V1 of the attack flow schema.)  You can find a detailed explanation of how to use Flow Builder V1 to generate a flow in [the Concepts section](docs/Concepts/flow_builder_v1).
 * A RDF graph in json-ld format that conforms to the Attack Flow [SHACL](docs/Reference/flow_manager_shacl.md) constraints.
 * A SPARQL query that creates a graph that conforms to the SHACL constraints.

The API itself is rather simple. Queries are passed as JSON in the body of a POST request.

```
{
    "token": TOKEN,
    "version": "0.0.1",
    "graph": "default",
    "attack_flow_version": "1",
    "builder": True,
    "best_practices": False,
    "flow": flow
}
```

`token` A random string used to identify the user.

`version` is the version of the API.  The current version of the API can be found in the [API Reference](docs/Reference/flow_manager_api.md).

`graph` The graph is the named graph to store into.  Flow Manager supports multiple distinct graph per token, however all tokens have a 'default' graph.

`attack_flow_version` This is the version of the Attack Flow schema used for the flow file.  It is only applicable if `flow` is an Attack Flow json-schema object (as is created by Flow Builder).  Supported versions are listed in the [API Reference](docs/Reference/flow_manager_api.md).

`builder` This indicates if the flow was generated in Flow Builder.  Taken in combination with `attack_flow_version`, Flow Manager will make modifications to the flow such as correcting the flow namespace and giving actions and assets unique names.

`best_practices` (Currently not used)  Best practices will attempt to apply best practices (such as using UUIDs in NamedIndividual IRIs). If `builder` is true, `best_practices` are applied regardless of this setting as Flow Builder V1 creates flows with some, we'll say, issues.  In general, `best practices` should be on, however it will be provided an an option for those who wish to write flows exactly as they are provided to the API.

`flow` The flow. For RDF and json-schema flows, a JSON object (not a string).  For SPARQL queries, the SPARQL query string.

`initNs` Optionally, rather than specify PREFIXes in the SPARQL query, a dictionary of namespaces can be supplied to replace in the SELECT statement itself.  The dictionary takes the form `{"foaf": "http://xmlns.com/foaf/0.1/"}` for example, then allowing `foaf:workplaceHomepage \"http://infosecanalytics.com\"` to be used in the query.  If not using, do not include in the query.


## POST /flows Responses

The returned response will return a JSON response.

```
{'id': 'deee77fe-d73c-4f34-843e-937e5132637e',
 'result': "35 triples added to graph 'default'",
 'success': True,
 'timestamp': '2022-12-07T22:23:09.098009'}
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