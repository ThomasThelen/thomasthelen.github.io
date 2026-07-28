---
title: Neo4j Cheat Sheet
date: 2025-07-30
slug: neo4j-cheat-sheet
tags: neo4j
description: A cheat sheet of Neo4j operations: bolt vs neo4j protocols, read replicas, server/client routing, LDAP, GDS clusters, upgrades, backups, and query tuning.
draft: true
---
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

Neo4j handles LDAP by mapping LDAP roles to Neo4j *roles*. Because this is configuration driven - the bulk of the work is done within the Neo4j config file where the LDAP server details are filled out with the role mappings. This can be helpful when multiple teams are interacting with the graph and you want to restrict access to different labels or write permissions. For example, allow the data engineering team to write to the graph but not modify the schema (leave that to the knowledge engineer team).

**What are the default ldap groups in Neo4j?**

Trick question? I wasn't sure how to answer this one because there aren't any default groups to my knowledge.

**Explain the architecture of a cluster with GDS**

Clusters are supposed to have 2n nodes in the cluster because of the reliance on the Raft protocol. GDS nodes don't participate in node consensus and therefore don't contribute to the . For this reason, a minimum cluster with GDS would consist of 3 nodes in the consensus with the fourth node acting as the GDS node.

**How do you restart Neo4j?**

Neo4j typically runs as a service, which means a simple `systemctl restart neo4j` should do the trick. It's not possible to restart Neo4j without downtime unless there's a deployment system on top, such as Kubernetes.

**How do you upgrade Neo4j? Is there downtime?**

Upgrading Neo4j can be done by replacing the neo4j binaries on the host system. Personally, I prefer to install the target version on the host and use the `neo4j-admin` tool to manually migrate the database to the fresh install.

*There will always be downtime when upgrading major versions of Neo4j*. Minor versions can be updated without downtime so long that Neo4j is running as a cluster. Otherwise, there will always be downtime when upgrading standalone instances.

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

**What are several ways to visualize graph data?**

1. The main Neo4j query interface displays CYPHER query results. I've found that I hit performance issues when visualizing large number of nodes and the visualization window is fairly small.
2. Neo4j offers Neodash, which is a dashboard application built on top of Neo4j. It allows for quick control creation (pie charts, bar charts, etc) from CYPHER queries. It's a great way to show summaries of data, but not for the overall network structure.
3. Neo4j also has a Gephi plugin that allows for sending data from a CYPHER query directly to Gephi where it can be visualized. Gephi is able to render large graphs and has a breadth of ways to tweak the visualizations. This is my go-to for visualizing large graphs however, the setup process can be cumbersome and I don't ask others to replicate the process.
4. Non-free tools like G.V()
