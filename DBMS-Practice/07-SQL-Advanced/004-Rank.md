# Rank

## Scenario
Product wants a "top salesperson per region" leaderboard, and ties should share the same rank but the next rank should skip appropriately (standard competition ranking: 1, 1, 3, not 1, 1, 2).

## Solution
```sql
SELECT
    region,
    salesperson,
    total_sales,
    RANK() OVER (PARTITION BY region ORDER BY total_sales DESC) AS sales_rank
FROM sales_summary;
```

If two salespeople in the same region tie for the highest `total_sales`, both get `sales_rank = 1`, and the next distinct value gets `sales_rank = 3` (RANK leaves a gap equal to the number of ties) - this matches how competitions typically rank tied scores.

## Takeaway
`RANK()` gives ties the same rank and **skips** subsequent rank numbers accordingly - use it when "how many people are strictly ahead of me" matters (e.g. leaderboards, competition standings).
