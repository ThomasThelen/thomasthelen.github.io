---
title: Changes to RDF in 1.2
date: 2025-03-06
slug: learning-content-hamiltonian-path
tags: knowledge-graph
description: Using Hamiltonian paths to model course sequencing and learning-path recommendations.
draft: true
---
I recently read a post by Clara Nguyễn, titled [Graph Theory Applications in Video Games](https://utk.claranguyen.me/talks.php?id=videogames). The big idea was using .

This got me thinking about a previous customer I was consulting for, who was building a system to upskill internal talent by building career paths which were supplemented by learning paths. For example, if an employee was a .

When reading Clara's post, I noticed that there are a few similarities between the graphical representation of mario kart and navigating career paths/learning content recommendations: You only want to visit each node (representing a course, or a job) once, you must visit them all, and the last-reached node isn't the one started at.

## 

The question becomes: Given a set of courses along with information on their pre-requisites, can we generate a graph whose hamilton path matches the desired learning path.
