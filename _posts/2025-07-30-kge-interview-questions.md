---
title: 'Knowledge Graph Engineer Interview Questions'
date: 2025-07-30
permalink: /posts/kge-interview/
tags:
  - knowledge-graph
---

# Knowledge Graph Engineer Interview Questions

Over the past several years I've been been on both ends of interviews for knowledge graph engineers. The gist of these interviews is that there's usually a soft portion centered around success (or failure) stories of getting graph systems in production. There's also usually a technical round that involves questions specific to graph and less so around typical DSA questions. Below are a number of questions I've asked and been asked. Most of these can be answered by regurgitating the output of an LLM, so try and add a touch of personal experience with each one to set yourself aside.


# RDF

The RDF space is pretty wide: questions can range from data modeling, graph databases, and knowledge engineering. Fortunately, there's no silver bullet database which cuts down on questions around DBA.

**What do rdfs:domain and rdfs:range do?**

This is a common question that tries to trick you with a common misconception around data modeling. The wrong answer would be that `rdfs:domain` and `rdfs:range` place restrictions on the data however, these annotations are actually for the logic reasoner. For restrictions, SHACL would need to be utilized. For example, the following ontology snippet states that the domain and range of the relation "tt:knows" are `foaf:Person`.

```

```

The misconception arises as a bug when the t-box is inserted into the graph and the relation `<tt:SparkyTheDog> tt:knows <tt:Python>` materializes, which is clearly incorrect.

```

```

**What is the difference between RDF, RDFS, and OWL?**

These three technologies form a hierarchy of expressibility, starting with RDF as the base, RDFS as the middleground, and OWL building on top of the two for maximum expressivity.

1. RDF: The base graph data model that is utilized by RDFS and OWL. No one really thinks of this when working with RDF (second nature).
2. RDFS: Builds on RDF by providing a basic vocabulary for building other vocabularies. It supports fairly basic reasoning around class hierarchy and properties. I almost always have some sort of RDFS going on in my ontologies.
3. OWL: Provides additional vocabulary for building more complex ontologies that support enhanced reasoning capabilities (think class intersection, cardinality, etc). This usually enters the picture when some sort of reasoning on the data needs to happen to fill the gap between the source data and the target queries.

**What kinds of reasoning can be done in OWL?**

Within the OWL ecosystem, there are several _profiles_ that have guarantees around decidability and supported reasoning strategies. Picking the appropriate profile can be a matter of looking at the types of target queries and the source data - the OWL profile provides the glue between the two.

1. The most common reasoning strategy is RDFS which is guaranteed to be decidable however, it lacks expressive reasoning (mostly subclass reasoning). Because it relies solely on RDFS, it's not an OWL profile. One strategy is to choose RDFS reasoning and then add rules on top of it, if the graph database supports this type of functionality (for example, Jena and GraphDB)

3. OWL Full: The full OWL superset that includes all features in OWL. I've never seen this profile used in production, likely due to it being undecidable for some axioms (although I don't think I've ever seen undecidability in practice).

2. OWL DL: The first subset of OWL which I've also never seen, likely due to exponential complexity guarantees: O(2^polynomial). It also includes a number of OWL features that usually aren't essential.

2. OWL EL: Provides existential quantifiers on class axioms however it lacks support for features such as negation, unions, and cardinality restrictions. It guarantees polynomial time complexity which is better than OWL DL.

3. OWL QL: Guarantees logarithmic scaling w.r.t axioms and is focused more on query-intensive applications (hence QL for query language). Like EL, it lacks cardinality restrictions and quantifiers. I've seen this in use in several production systems.

3. OWL RL: This profile also guarantees polynomial time complexity for reasoning tasks, and places an emphasis on systems that heavily leverage reasoning. From what I've seen - most applications place an emphasis on queryability and less on advanced reasoning. This is probably the reason why I've never seen it used in production systems.

![](/images/posts/kg-interviews/owl.svg)

**What are the major triplestore vendors?**

1. Apache Jena: My go-to for RDF work. It's been around for 20+ years, is open source, supports enterprise features, and is free.

2. RDF4J: Another open source RDF solution that's had years of community development. I use this for embedded applications however, it lacks enterprise features.

3. Ontotext GraphDB: This used to be my go-to until the recent licensing changes. It's a great out-of-the-box solution that includes an intuitive interface. The enterprise version supports additional features like HA clusters.

4. Stardog: Another closed source graph database that provides an out-of-the-box experience. It's locked behind hefty licensing which makes evaluation difficult.

5. QLever: Less so a graph database than a query interface for RDF data. One of the benefits is that it's lightweight, easy to deploy, and comes with a UI. 

**Why are graph databases called triplestores, how do they compare to quadstores?**

Graph databases that support RDF are sometimes referred to as _triplestores_ because they store RDF triples (s, p, o). Most triplestores are actually quadstores, which support named graphs (s,p,o,g). For historical reasons, most people just say _triplestore_ when referring to quadstores.

**When would you use the turtle format instead of n-triples?**

There are a couple of times when using turtle has advantages over n-triples:

1. Human readability is better with turtle
2. Data compression is often better with turtle because of the prefix manifest. When transferring or storing data this could be the optimal format.
3. Some graph databases import data faster as turtle instead of n-triples. Be careful though - others operate faster on n-triples, presumably because the prefix manifest doesn't need to be parsed.

A similar question is comparing trix vs n-quads, which effectively has the same answer.

**How does SPARQL compare to SQL?**

SPARQL queries can generally be transformed to SQL through algebraic processes and differs by placing an emphasis on traversing graph paths. Joins are completed by the processing engine rather than being explicitly stated, as done in SQL. This is true for most graph query languages, including ones for LPG.

**What is the underlying data structure for most graph databases?**

Most graph databases utilize a table structure and place heavy reliance on indexes. Common indexes seen in RDF are permutations of the SPO structure. These are SPO, PSO, OSP. When named graphs are present the permutations expand to include the graph: GSPO, etc. 

The heavy reliance on indexing creates issues scaling large knowledge graphs w.r.t hardware usage. Additionally, the lack of support from triplestores for data sharding compounds this issue.

**What's the difference between RDF and LPG?**

The main difference is that they aim to solve different problems. RDF arose as a solution to tagging data on the web, while LPG's original intent was to support network science at scale. The databases that we see today tell enough of the story - most triplestores don't have out-of-the-box data science platforms while most LPG's do.

I make the decision between the two based on the product requirements. Is the goal to share information, leverage the semantic web, or utilize complex logical reasoning? RDF. Is data science being done in a way that requires ACID transactions and high availability? LPG.

# LPG


## Neo4j

At the moment, Neo4j is the most popular LPG and I've found that most interview questions revolve around modeling data and managing Neo4j instances.

**Explain the difference between the bolt and Neo4j Protocols**

If you've ever programmatically interacted with Neo4j, you've most likely connected to the graph through `bolt://` or `neo4j://`.

`neo4j://` handles routing to different nodes within Neo4j clusters - it's not possible to connect to a specific instance. It's a good catch-all for letting the Neo4j proxy dictate which node your query gets sent to.

`bolt://` allows you to connect to a single node in a cluster. This is used when communicating with a GDS node or when targeting specific read replicas.

**What are the uses for a read replica?**

The great thing about read replicas is that they can be individually scaled to support queries, easing up resources on nodes within the Raft consensus. They can also be used to perform graph backups, data science operations, and QA. The gist is that these use cases all stem from the fact that they don't participate in node consensus, so these operations don't interfere with other graph operations. It's also possible to vertically scale these nodes without touching other cluster nodes.

**What is Server and Client Side Routing?**

These both refer to how your connection is routed to different nodes in a cluster.

Server side routing: Routing from your application to a node is done on Neo4j's side by its load balancer/proxy. The advantage is that Neo4j is always aware of its cluster topology and can handle routing. Rather than needing to connect to a single node, whose location can change - server side routing allows you to connect to a single endpoint and handles the rest.

Client side routing is performed by your own application (usually the Neo4j driver you're using). It needs to maintain routing tables of the node topology; it allows for finer grain control over which node you're accessing. One problem here is that the driver needs to stay up to date with the cluster topology; if it gets out of date you may be connecting to the wrong node (or one that doesn't exist).

**How do you enable LDAP in neo4j?**

Neo4j handles LDAP by mapping LDAP roles to Neo4j _roles_. Because this is configuration driven - the bulk of the work is done within the Neo4j config file where the LDAP server details are filled out with the role mappings. This can be helpful when multiple teams are interacting with the graph and you want to restrict access to different labels or write permissions. For example, allow the data engineering team to write to the graph but not modify the schema (leave that to the knowledge engineer team).

**What are the default ldap groups in Neo4j?**

Trick question? I wasn't sure how to answer this one because there aren't any default groups to my knowledge.

**Explain the architecture of a cluster with GDS**

Clusters are supposed to have 2n nodes in the cluster because of the reliance on the Raft protocol. GDS nodes don't participate in node consensus and therefore don't contribute to the . For this reason, a minimum cluster with GDS would consist of 3 nodes in the consensus with the fourth node acting as the GDS node.

**How do you restart Neo4j?**

Neo4j typically runs as a service, which means a simple `systemctl restart neo4j` should do the trick. It's not possible to restart Neo4j without downtime unless there's a deployment system on top, such as Kubernetes.

**How do you upgrade Neo4j? Is there downtime?**

Upgrading Neo4j can be done by replacing the neo4j binaries on the host system. Personally, I prefer to install the target version on the host and use the `neo4j-admin` tool to manually migrate the database to the fresh install.

_There will always be downtime when upgrading major versions of Neo4j_. Minor versions can be updated without downtime so long that Neo4j is running as a cluster. Otherwise, there will always be downtime when upgrading standalone instances.

**How does SSL work in Neo4j?**

SSL is configured through the node's Neo4j configuration file. Certificates should be either self signed or done through a CA. These need to be placed on each node, along with the proper config values to both enable ad point to the certificates.

**How do you back up and restore data?**

There are a couple of ways to back up data. The `neo4j-admin` tool can be used for both online and offline backups with the `database dump` and `database backup` commands. Alternatively, the entire database folder can be copied.

Restoring can also be done with the `neo4j-admin` tool or by copying the database backup from disk onto the new instance.

**Why would you cycle primary and secondary nodes?**

This one was a bit tough - the only reasons I could think of are to clear the caches.

**How do you speed up a query?**

1. The easiest is to profile the query and look for any steps than can be removed, for example filtering on labels
2. Increase RAM
3. Write a stored procedure

**Why would you write custom stored procedures?**
    
Stored procedures should be used to improve query performance by manually leveraging the Java transaction API and when loading bulk data. By leveraging the transaction API the query planner is bypassed, giving finer control of how the data is navigated to answer the query. Note that stored procedures for query answering are usually one-off and targeted for specific queries.