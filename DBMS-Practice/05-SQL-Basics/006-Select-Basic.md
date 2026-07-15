# Select Basic

## Scenario
Support asks for a quick list of all "premium" plan customers to check if a bug affected them specifically.

## Problem
Write a query returning `id`, `name`, and `email` for customers on the `'premium'` plan.

## Solution
```sql
SELECT id, name, email
FROM customers
WHERE plan = 'premium';
```

## Takeaway
Avoid `SELECT *` in application code and ad-hoc queries you'll reuse - naming exact columns is more efficient (less data transferred), more resilient to schema changes (adding a column doesn't silently change your result shape), and self-documents what the query actually needs.
