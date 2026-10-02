---
title: "Relationship trust mischeck: authorizing only one side of a graph or join relationship"
description: Object graphs where org, team, or tenant membership is not joined when accessing a child or related record.
keywords:
  - cross tenant
  - org membership
  - team access
  - BOLA
---

# Relationship mischeck

## Context

B2B apps model org, team, project, and role edges. A subject may be in org A and still request an object that is only linked to org B through a relationship table. Missing membership joins on that edge is a BOLA variant on the relationship, not on the single primary key table alone.

## Theory

The tell in code review is a query that checks `user.org_id` but not `object.org_id` or that checks `project_id` on the child without verifying the project is under the org. API batch endpoints that return “all projects for user” and then take a project id from a second call are easy to misalign.

## Practice

### Cross-tenant id handoff in a lab

- Create the same object type in org A and org B with two test users. Call the read or update for B’s object while authenticated as a user who should only see A, using paths that include A’s org or team context in the URL.

## Tools

- **Burp Suite**
- **curl**
