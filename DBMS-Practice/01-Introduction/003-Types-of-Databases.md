# Types of Databases

## Scenario
You are designing the backend for a new product. Three candidate features:
1. Order and payment records for an e-commerce checkout (must be consistent, has clear schema: orders, items, payments).
2. A product catalog where each product category has wildly different attributes (books have ISBN/author, TVs have screen-size/resolution).
3. A "people you may know" feature that needs to traverse friend-of-a-friend relationships quickly.

## Problem
For each of the three features, pick a database type (relational, document, graph, key-value, column-family) and justify it.

## Solution
1. Orders/payments -> **Relational (SQL)**. Strong schema, foreign keys, and ACID transactions matter a lot when money is involved (e.g. an order and its payment must commit together).
2. Product catalog with variable attributes -> **Document store** (e.g. MongoDB). Each product can have a different shape of JSON document without forcing a rigid column-per-attribute schema or dozens of nullable columns.
3. Friend-of-a-friend traversal -> **Graph database** (e.g. Neo4j). Relational JOINs across a "friends" table get expensive at 3+ hops; graph databases traverse edges in near-constant time regardless of depth.

## Takeaway
"Which database should I use" is not one answer - it depends on the shape of your data and your access patterns. Relational is the safe default for transactional, structured data; reach for NoSQL variants when your data or query pattern doesn't fit rows and columns well.
