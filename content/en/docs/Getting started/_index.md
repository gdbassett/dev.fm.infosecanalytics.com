---
categories: ["Examples", "Placeholders"]
tags: ["test","docs"] 
title: "Getting Started"
linkTitle: "Getting Started"
weight: 2
description: >
  What you need to get started
---


Information in this section helps your user try your project themselves.

* What do your users need to do to start using your project? This could include downloading/installation instructions, including any prerequisites or system requirements.

* Introductory “Hello World” example, if appropriate. More complex tutorials should live in the Tutorials section.

Consider using the headings below for your getting started page. You can delete any that are not applicable to your project.

## Prerequisites

For beginners, we recommend downloading V1.0.0 of Flow Builder at https://github.com/center-for-threat-informed-defense/attack-flow/releases/tag/v1.0.0 and unzipping it into a directory.  This will provide a local webapp that can used for building flows.

We will also use python to interface with the API, however any tool capable of making HTML requests will do.  If you are using python, please install the [RDFLIB]() package to store the returned graph (`pip3 install rdflib`)

To use Flow Builder you will need an API token.  For trial tokens, please contact us [through our website]](http://infosecanalytics.com/contact/).

## Installation

No software installation is needed to use Flow Builder.

## Setup

There is no setup, but an API key is required.

## Try uploading a flow!

In our python example, we'll start by loading some libraries.
```{p}
# Load Necessary Libraries
import requests # used to interface with the API
import json # used to serialize data
from pprint import pprint # used to print serialized data
from rdflib import Graph, URIRef, Literal # used to store the graph once retrieved from attack flow
```

Next, define static variables.
```
TOKEN =  "<YOUR API TOKEN>"
URL = "https://api.fm.infosecanalytics.com/flows"
```

Load a flow built with Flow Builder.
```
with open("/Users/you/Downloads/attack_flow_2022-11-17_22-37-44.json", 'r') as filehandle:
    flow = json.load(filehandle)
```

Create the query to send to the API.  (For questions about all of the API parameters, see the [API reference](docs/Reference)).
```
# Create the query body
post_body = {
    "token": TOKEN,
    "version": "0.0.1",
    "graph": "default",
    "attack_flow_version": "1",
    "builder": True,
    "best_practices": False,
    "flow": flow
}
```

You can now send the query and print the response.
```
# use the API to store the flow
x = requests.post(URL, json = post_body)

pprint(x.json())
```
Return:
```
{'id': 'deee77fe-d73c-4f34-843e-937e5132637e',
 'result': "35 triples added to graph 'default'",
 'success': True,
 'timestamp': '2022-12-07T22:23:09.098009'}
 ```

## Try retrieving some information about the flows in the graph

We can also retrieve the flows in the graph.

We'll first start by creating the SPARQL query.  (For more information about SPARQL queries, see the [Concepts](docs/Concepts) section.)

```
# create an API query containing a SPARQL query to retrieve the flow
query_sparql = """PREFIX af: <https://attackflow.space/attack-flow#> 
select DISTINCT ?s ?p ?o where { 
    ?s ?p ?o .
    ?s a af:attack-flow .
} limit 10"""
```

Create the API query.

```
get_body = {
    "token": TOKEN,
    "version": "0.0.1",
    "graph": "default",
    "attack_flow_version": "1",
    "builder": True,
    "best_practices": False,
    "flow": query_sparql
}
```

And send it to the API.

```
# Run the GET query to retrieve the flow
x = requests.get(URL, json = get_body)

pprint(x.json())
```

Which gets us the results such as:
```
{'id': 'feb7c29d-0305-475f-bfc7-46079540d28a',
 'result': '{"results": {"bindings": [{"s": {"type": "uri", "value": '
           '"urn:absolute:flows#flow--41e0cd93-6fb2-4786-94ab-5adec21960cc"}, '
           '"p": {"type": "uri", "value": '
           '"https://attackflow.space/attack-flow#created"}, "o": '
           '{"type": "literal", "value": "2022-11-26T22:22:55+00:00", '
           '"datatype": "http://www.w3.org/2001/XMLSchema#dateTime"}}, {"s": '
           '{"type": "uri", "value": '
           '"urn:absolute:flows#flow--5de4fc72-e1b2-47af-8e35-0e0d2b0e425b"}, '
           '"p": {"type": "uri", "value": '
           '"https://attackflow.space/attack-flow#created"}, "o": '
           '{"type": "literal", "value": "2022-11-26T22:22:55+00:00", '
           '"datatype": "http://www.w3.org/2001/XMLSchema#dateTime"}}, {"s": '
           '{"type": "uri", "value": '
           '"urn:absolute:flows#flow--8eff62df-ce9f-4db4-ac37-70d919f589c2"}, '
           '"p": {"type": "uri", "value": '
           '"https://attackflow.space/attack-flow#created"}, "o": '
           '{"type": "literal", "value": "2022-11-26T22:22:55+00:00", '
           '"datatype": "http://www.w3.org/2001/XMLSchema#dateTime"}}, {"s": '
           '{"type": "uri", "value": '
           '"urn:absolute:flows#flow--b7c0e7d8-def1-4fb1-a6d2-07cd3fe55f3c"}, '
           '"p": {"type": "uri", "value": '
           '"https://attackflow.space/attack-flow#created"}, "o": '
           '{"type": "literal", "value": "2022-11-26T22:22:55+00:00", '
           '"datatype": "http://www.w3.org/2001/XMLSchema#dateTime"}}, {"s": '
           '{"type": "uri", "value": '
           '"urn:absolute:flows#flow--41e0cd93-6fb2-4786-94ab-5adec21960cc"}, '
           '"p": {"type": "uri", "value": '
           '"http://www.w3.org/1999/02/22-rdf-syntax-ns#type"}, "o": {"type": '
           '"uri", "value": "http://www.w3.org/2002/07/owl#NamedIndividual"}}, '
           '{"s": {"type": "uri", "value": '
           '"urn:absolute:flows#flow--41e0cd93-6fb2-4786-94ab-5adec21960cc"}, '
           '"p": {"type": "uri", "value": '
           '"http://www.w3.org/1999/02/22-rdf-syntax-ns#type"}, "o": {"type": '
           '"uri", "value": '
           '"https://attackflow.space/attack-flow#attack-flow"}}, {"s": '
           '{"type": "uri", "value": '
           '"urn:absolute:flows#flow--5de4fc72-e1b2-47af-8e35-0e0d2b0e425b"}, '
           '"p": {"type": "uri", "value": '
           '"http://www.w3.org/1999/02/22-rdf-syntax-ns#type"}, "o": {"type": '
           '"uri", "value": "http://www.w3.org/2002/07/owl#NamedIndividual"}}, '
           '{"s": {"type": "uri", "value": '
           '"urn:absolute:flows#flow--5de4fc72-e1b2-47af-8e35-0e0d2b0e425b"}, '
           '"p": {"type": "uri", "value": '
           '"http://www.w3.org/1999/02/22-rdf-syntax-ns#type"}, "o": {"type": '
           '"uri", "value": '
           '"https://attackflow.space/attack-flow#attack-flow"}}, {"s": '
           '{"type": "uri", "value": '
           '"urn:absolute:flows#flow--8eff62df-ce9f-4db4-ac37-70d919f589c2"}, '
           '"p": {"type": "uri", "value": '
           '"http://www.w3.org/1999/02/22-rdf-syntax-ns#type"}, "o": {"type": '
           '"uri", "value": "http://www.w3.org/2002/07/owl#NamedIndividual"}}, '
           '{"s": {"type": "uri", "value": '
           '"urn:absolute:flows#flow--8eff62df-ce9f-4db4-ac37-70d919f589c2"}, '
           '"p": {"type": "uri", "value": '
           '"http://www.w3.org/1999/02/22-rdf-syntax-ns#type"}, "o": {"type": '
           '"uri", "value": '
           '"https://attackflow.space/attack-flow#attack-flow"}}]}, '
           '"head": {"vars": ["s", "p", "o"]}}',
 'success': True,
 'timestamp': '2022-12-07T23:24:56.699994'}
 ```

However, we'd like this in a more managable graph format.  We can parse the SPARQL query response (in JSON) back into a graph:

```
# Create a RDFLIB graph to store the triples
g = Graph()

# NOTE: the result value in the json is a string
# Convert the result back to JSOn
result = json.loads(x.json()['result'])

for record in result['results']['bindings']:
    if record['o']['type'] == 'uri':
        g.add((URIRef(record['s']['value']), URIRef(record['p']['value']), URIRef(record['o']['value'])))
    else:
        g.add((URIRef(record['s']['value']), URIRef(record['p']['value']), Literal(record['o']['value'])))
```

And finally we have our local graph
`print(g.serialize())`
```
@prefix ns1: <https://attackflow.space/attack-flow#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .

<urn:absolute:flows#flow--41e0cd93-6fb2-4786-94ab-5adec21960cc> a owl:NamedIndividual,
        ns1:attack-flow ;
    ns1:created "2022-11-26T22:22:55+00:00" .

<urn:absolute:flows#flow--5de4fc72-e1b2-47af-8e35-0e0d2b0e425b> a owl:NamedIndividual,
        ns1:attack-flow ;
    ns1:created "2022-11-26T22:22:55+00:00" .

<urn:absolute:flows#flow--8eff62df-ce9f-4db4-ac37-70d919f589c2> a owl:NamedIndividual,
        ns1:attack-flow ;
    ns1:created "2022-11-26T22:22:55+00:00" .

<urn:absolute:flows#flow--b7c0e7d8-def1-4fb1-a6d2-07cd3fe55f3c> ns1:created "2022-11-26T22:22:55+00:00" .
```

(Note, this output is truncated since we set a limit of 10 triples.)
