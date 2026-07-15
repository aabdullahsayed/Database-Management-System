# Lead

## Scenario
For a subscription analytics table, you need each customer's "time until next renewal" by comparing the current row's date to the NEXT row's date, per customer.

## Solution
```sql
SELECT
    customer_id,
    renewal_date,
    LEAD(renewal_date) OVER (
        PARTITION BY customer_id ORDER BY renewal_date
    ) - renewal_date AS days_until_next_renewal
FROM renewals;
```

`LEAD(renewal_date)` looks **forward** to the next row's value within the same partition (mirror image of `LAG`) - for a customer's most recent renewal (no future row exists yet), `LEAD` returns `NULL`, correctly signaling "we don't know the next renewal date yet."

## Takeaway
`LAG` looks backward, `LEAD` looks forward - both avoid self-joins for "compare this row to the neighboring row" patterns, common in time-series and sequence analysis.
