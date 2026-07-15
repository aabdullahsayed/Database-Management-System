# Cost-Based Optimization

## Scenario
The same query against `orders WHERE status = 'pending'` uses an index scan when 1% of rows are pending, but switches to a full sequential scan once a data migration bug leaves 80% of rows marked 'pending' - and this is actually the *correct* choice, not a bug.

## Problem
Why does the optimizer abandon the index once most rows match the filter, and is that the right call?

## Solution
Cost-based optimization estimates the cost of each candidate plan using table/column statistics (row counts, value distribution histograms) and picks the cheapest. An index scan for `status = 'pending'` when only 1% of rows match is cheap: jump to the few matching entries via the index, fetch each corresponding row (few bookmark lookups). But when 80% of rows match, using the index means visiting the index AND then randomly jumping back to fetch 80% of the table's rows anyway - at that selectivity, a plain sequential scan (reading the table straight through) is actually **faster** in practice, since it avoids the scattered random-access pattern of repeated index bookmark lookups.

## Takeaway
Indexes aren't always faster - a cost-based optimizer correctly abandons an index once a filter's selectivity gets too low (too many matching rows), because sequential scanning becomes cheaper than the overhead of many random-access index lookups; this is expected behavior, not a bug, though it's often a sign of a data skew problem worth investigating separately.
