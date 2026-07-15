# Dense Rank

## Scenario
Same leaderboard as before, but this time product wants NO gaps in the rank numbers even when there are ties - e.g. tiers "Gold=1, Silver=2, Bronze=3" shouldn't skip a tier number just because two people tied for Gold.

## Solution
```sql
SELECT
    region,
    salesperson,
    total_sales,
    DENSE_RANK() OVER (PARTITION BY region ORDER BY total_sales DESC) AS sales_tier
FROM sales_summary;
```

With `DENSE_RANK()`, two tied salespeople both get rank 1, and the very next distinct value gets rank **2** (no gap), unlike `RANK()` which would jump to 3.

## Takeaway
Use `RANK()` when gaps after ties are meaningful (true competitive standing); use `DENSE_RANK()` when you want a compact, gapless tier/level number regardless of how many ties occurred.
