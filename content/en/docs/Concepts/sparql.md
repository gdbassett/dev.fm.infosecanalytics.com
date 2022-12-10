---
title: "SPARQL"
date: 2022-11-04
weight: 4
description: >
  How to write the SPARQL queries that are important for using Flow Manager.
---

{{% pageinfo %}}
Just a foreword, SPARQL, like any query language, can be picky.  http://www.sparql.org/ has several validators you can use to find errors in your queries.
{{% /pageinfo %}}

So SPARQL is kinda like SQL for graphs.  Frankly, it's kinda old compared to [Cypher](https://opencypher.org/), [Gremlin](https://tinkerpop.apache.org/gremlin.html), or [Storm](https://synapse.docs.vertex.link/en/latest/synapse/userguides/storm_ref_intro.html) bug it has some properties that make it nice for what we're doing.


### Your first SPARQL query

Lets start with something simple and break it down.

```
select * where { 
  ?s ?r ?d .
} limit 10 
```

This query returns 10 triples from the database.
`select * ` - Just return whatever is found.  We could have returned specific variables.  
`where { ... }` - filter out all the triples that are true under all the statements between the curly brackets  
`?s ?r ?d .` - The things with a `?` in front of them are variables. In the [Knowledge Graphs](docs/Concepts/knowledge_graphs) introduction, we talked about how knowledge graphs are composed of triples.  SPARQL centers around triples as well.  We're calling the variables ?s for source, ?r for relationship and ?d for destination here, but they can be anything. You'll often see ?s ?p ?o for subject, predicate, and object.  The `.` at the end means "this is the end of the line".  We'll show what the alternative to everything on one line is a little later.  
`limit 10` - Only return 10 items.

```
{
  "head": {
    "vars": [
      "s",
      "r",
      "d"
    ]
  },
  "results": {
    "bindings": [
      {
        "s": {
          "type": "uri",
          "value": "http://www.w3.org/1999/02/22-rdf-syntax-ns#type"
        },
        "r": {
          "type": "uri",
          "value": "http://www.w3.org/1999/02/22-rdf-syntax-ns#type"
        },
        "d": {
          "type": "uri",
          "value": "http://www.w3.org/1999/02/22-rdf-syntax-ns#Property"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "http://www.w3.org/2000/01/rdf-schema#subPropertyOf"
        },
        "r": {
          "type": "uri",
          "value": "http://www.w3.org/1999/02/22-rdf-syntax-ns#type"
        },
        "d": {
          "type": "uri",
          "value": "http://www.w3.org/1999/02/22-rdf-syntax-ns#Property"
        }
      },
      ...
    ]
  }
}
```

Something to note is what the output is and isn't.  Because SPARQL can return more than just IRIs and triples, the return is lines of text.  Now most tools can convert it back to something more easily parsable such as CSV, JSON etc.  Flow Manager returns the data as a json string, thought not a graph.  The data can then be reassembled to triples (s, r, d) that then can be joined to recreate the graph.

### your first _useful_ SPARQL query

This query is going to produce a table of flows

```
PREFIX af: <https://attackflow.space/attack-flow#>
select DISTINCT ?flow ?name ?description ?author where { 
    ?s af:flow ?flow .
    ?flow af:name ?name .
    ?flow af:description ?description .
    ?flow af:author ?author .
} limit 10
```

Lets break this one down.
`PREFIX af: <https://attackflow.space/attack-flow#>` This is something new.  We discussed namespaces on the [Knowledge Graphs](docs/Concepts/knowledge_graphs) page.  Namespaces get used over and over so to simplify them, we can create a PREFIX command at the beginning of the SPARQL query and only use the short string (in this case 'af') in our query.  (P.S. You can have as many PREFIX lines as you want.  It's usual to to have several common PREFIX lines such as RDF, RDFS, OWL, etc)
`select DISTINCT ?flow ?name ?description ?author` Return four values per row.  (Note the **DISTINCT** keyword. It does the obvious thing: only returns unique lines.)
`?s af:flow ?flow .` This requires that a node (?s) points to the node ?flow with an af:flow edge.  (We did it this way so there'd only be one prefix, but if you were OK with two prefixes, this line could easily be `?flow RDF:type af:flow .`)
```
    ?flow af:name ?name .
    ?flow af:description ?description .
    ?flow af:author ?author .
```
THe next three lines identify multiple variables per flow (?name, ?description, and ?author).  NOTE! This does not mean you'll get one line per flow.  If a flow happened to have multiple names, authors, etc it could potentially be on multiple lines.

So what does it return?

```
{
  "head": {
    "vars": [
      "flow",
      "name",
      "description",
      "author"
    ]
  },
  "results": {
    "bindings": [
      {
        "flow": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        },
        "name": {
          "type": "literal",
          "value": "Test Flow"
        },
        "description": {
          "type": "literal",
          "value": "A test flow"
        },
        "author": {
          "type": "literal",
          "value": "Gabriel Bassett"
        }
      },
      {
        "flow": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_6f471cb3-1fc0-4fd6-8d9b-7725af56b442"
        },
        "name": {
          "type": "literal",
          "value": "DDoS Template Flow"
        },
        "description": {
          "type": "literal",
          "value": "A prototypical DDoS Attack"
        },
        "author": {
          "type": "literal",
          "value": "Gabriel. Bassett"
        }
      }
    ]
  }
}
```

### Finding the Actions and Assets in a Flow

Because all actions and assets in a flow point to the flow node, it serves as an easy way to query for them:

```
PREFIX af: <https://attackflow.space/attack-flow#>
select DISTINCT ?s ?o ?p where { 
    ?s ?o ?p .
    FILTER (?o = af:flow)
    FILTER (?p = <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97)
} 
ORDER BY ?s 
LIMIT 100 
```
resulting in
```
{
  "head": {
    "vars": [
      "s",
      "o",
      "p"
    ]
  },
  "results": {
    "bindings": [
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#action_653f8cfb-dce0-40b9-89af-1af9c4172340"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#flow"
        },
        "p": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#Asset_fda2b7a6-3904-4f78-ab1d-0ad7b1b6a1ed"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#flow"
        },
        "p": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        }
      }
    ]
  }
}
```


You probably have got most of the lines of this at this point so lets skip to the interesting part:
`?s ?o ?p .` As you now know, this selects all triples
`FILTER (?o = af:flow)` Here we we say we only want flows where ?o is af:flow, so `?s af:flow ?p`
`FILTER (?p = <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97)` adds another filter, resulting in something like `?s af:flow <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97>`.
`order by ?s` just arranges the results

In fact, you can create that query:
```
PREFIX af: <https://attackflow.space/attack-flow#>
select DISTINCT ?s where { 
    ?s af:flow <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97> .
} limit 100 
```
Except because only ?s is a variable now it's the only thing returned:
```
{
  "head": {
    "vars": [
      "s"
    ]
  },
  "results": {
    "bindings": [
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#action_653f8cfb-dce0-40b9-89af-1af9c4172340"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#Asset_fda2b7a6-3904-4f78-ab1d-0ad7b1b6a1ed"
        }
      }
    ]
  }
```

### Adding the relationships between actions/assets to the actions, assets, and flows

We now have two triples, one that points from an action to a flow and one from an asset to the flow.  But we don't have the relationships between them.  Lets add that:

```
PREFIX af: <https://attackflow.space/attack-flow#>
select DISTINCT ?s ?o ?p where { 
    {
        ?s ?o ?p .
        FILTER (?o = af:flow)
        FILTER (?p = <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97>)
    } UNION {
        ?s ?o ?p .
        ?s af:flow <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97> .
        ?p af:flow <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97> .
    }
} limit 100 
```

The only real addition here is the `UNION` line.  We've created two sets of filters, one that gets us (action/asset af:flow the_flow) and one that gets us (action/asset relationship action/asset).  (The second filter does this by starting with all triples, then filtering to only the ones where the source and destination of the triple point to our specific flow of interest with a `af:flow` edge.)

Now in addition to the triples from before, we have a triple from our action to our asset.
```
{
  "head": {
    "vars": [
      "s",
      "o",
      "p"
    ]
  },
  "results": {
    "bindings": [
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#action_653f8cfb-dce0-40b9-89af-1af9c4172340"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#flow"
        },
        "p": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#Asset_fda2b7a6-3904-4f78-ab1d-0ad7b1b6a1ed"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#flow"
        },
        "p": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#action_653f8cfb-dce0-40b9-89af-1af9c4172340"
        },
        "o": {
          "type": "uri",
          "value": "http://attackflow.space/veris#Availability"
        },
        "p": {
          "type": "uri",
          "value": "urn:absolute:flows#Asset_fda2b7a6-3904-4f78-ab1d-0ad7b1b6a1ed"
        }
      }
    ]
  }
}
```

> Lets get a bit meta.  There's a sort of [Grammar for manipulating data](https://dplyr.tidyverse.org/).  Things like "select", "filter", "modify", "arrange", "summarize", and "join" and it applies to graphs too.  They form a sort of command progression: start with the dataset -> filter stuff out -> select just the variables we need from what's left -> arrange it.  (In fact this command progression is ridiculously common across all types of data.)  
>  
> In SPARQL we start with our full dataset. We use each line in 'where' to filter until we have just the parts we need (filter).  our SELECT line determines which variables we return.  and our ORDER BY determines the order they are returned in.  
>  
> It's also common to add in UNIONs to build groupings which don't logically result from a single filter.  
>  
> While we didn't cover other things like modify, SPARQL supports it.  It's just not something you're likely to use right away, except maybe to remove some data unintentionally stored.  More than likely if you need to modify data, you'll run a query, create the updates outside the database (such as by enriching the data) and adding the new edges back in using a (create)[docs/Concepts/sparql_create.md] statement.  


We can put this all together in a query to retrieve an entire flow and all of it's properties:

```
PREFIX af: <https://attackflow.space/attack-flow#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
select DISTINCT ?s ?o ?p where { 
    {
        ?s ?o ?p  . 
        ?s af:flow <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97> .
    } UNION {
        ?s ?o ?p .
        FILTER (?s = <urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97>)
    }
}
```

The first  WHERE block gives us all triples that start with a node in our flow (our actions and assets) including their properties.  The second where block gives us all triples starting in the flow node itself (giving us the flow's properties).

Together this returns:

```
{
  "head": {
    "vars": [
      "s",
      "o",
      "p"
    ]
  },
  "results": {
    "bindings": [
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#action_653f8cfb-dce0-40b9-89af-1af9c4172340"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#flow"
        },
        "p": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#action_653f8cfb-dce0-40b9-89af-1af9c4172340"
        },
        "o": {
          "type": "uri",
          "value": "http://attackflow.space/veris#Availability"
        },
        "p": {
          "type": "uri",
          "value": "urn:absolute:flows#Asset_fda2b7a6-3904-4f78-ab1d-0ad7b1b6a1ed"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#Asset_fda2b7a6-3904-4f78-ab1d-0ad7b1b6a1ed"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#flow"
        },
        "p": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        },
        "o": {
          "type": "uri",
          "value": "http://www.w3.org/1999/02/22-rdf-syntax-ns#type"
        },
        "p": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#flow"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#created"
        },
        "p": {
          "type": "literal",
          "value": "2022-08-25T17:02:45.423Z"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#name"
        },
        "p": {
          "type": "literal",
          "value": "Test Flow"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#description"
        },
        "p": {
          "type": "literal",
          "value": "A test flow"
        }
      },
      {
        "s": {
          "type": "uri",
          "value": "urn:absolute:flows#Flow_605eeda8-2628-4897-b68c-e54b144d4e97"
        },
        "o": {
          "type": "uri",
          "value": "https://attackflow.space/attack-flow#author"
        },
        "p": {
          "type": "literal",
          "value": "Gabriel Bassett"
        }
      }
    ]
  }
}
```

This is certainly not a comprehensive look at SPARQL.  For that there's [the documentation](https://www.w3.org/TR/2013/REC-sparql11-overview-20130321/).  But hopefully this is enough to get you started as you look to retrieve data from Flow Manager.