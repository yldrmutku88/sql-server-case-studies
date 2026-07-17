# Post-Restriction Transaction Analysis

## Overview

This document demonstrates an anonymized SQL analysis used to identify
customers who completed transactions after a restriction or review
record was created.

All table names, database names, and business-specific identifiers have
been generalized while preserving the analytical approach.

------------------------------------------------------------------------

## Business Scenario

Operations and risk teams may need to verify whether customers completed
transactions after a restriction event was recorded.

The report correlates customer restrictions with completed transactions
to help review operational processes and identify potential exceptions.

------------------------------------------------------------------------

## Features

-   Customer-based transaction analysis
-   Restriction timeline comparison
-   Completed transaction filtering
-   Transaction counting
-   Operational investigation support

------------------------------------------------------------------------

## SQL Concepts Used

-   INNER JOIN
-   LEFT JOIN
-   Aggregate Functions (`COUNT`)
-   GROUP BY
-   Date comparison
-   Conditional aggregation

------------------------------------------------------------------------

## Sample Query

``` sql
SELECT
    t.CustomerId,
    c.FullName,
    r.RestrictionCreatedDate,
    t.TransactionDate,

    COUNT(
        CASE
            WHEN ts.StatusGroup = 'Completed'
            THEN 1
        END
    ) AS CompletedTransactionCount,

    t.TransactionId,
    r.AlertMessage,
    r.Notes

FROM Transactions t

INNER JOIN TransactionStatus ts
    ON ts.StatusId = t.StatusId

INNER JOIN CustomerRestrictions r
    ON r.CustomerId = t.CustomerId
   AND r.RestrictionType = 'Risk'
   AND r.IsActive = 0
   AND t.TransactionDate >= r.RestrictionCreatedDate

LEFT JOIN Customers c
    ON c.CustomerId = t.CustomerId

WHERE
    t.TransactionType IN ('Payment','Transfer')
    AND ts.StatusGroup = 'Completed'
    AND r.AlertMessage NOT LIKE '%Excluded%'
    AND t.TransactionDate < '2026-06-12'

GROUP BY
    t.CustomerId,
    c.FullName,
    r.RestrictionCreatedDate,
    t.TransactionDate,
    t.TransactionId,
    r.AlertMessage,
    r.Notes

ORDER BY
    t.CustomerId;
```

------------------------------------------------------------------------

## Performance Notes

Recommended indexes:

-   CustomerId
-   TransactionDate
-   StatusId
-   RestrictionCreatedDate

Filtering transactions before aggregation reduces the workload on the
query optimizer.

------------------------------------------------------------------------

## Skills Demonstrated

-   SQL Server
-   Risk & Operations Analytics
-   Timeline Analysis
-   Conditional Aggregation
-   Data Validation
-   Reporting

------------------------------------------------------------------------

## Best Practices

-   Compare event dates explicitly.
-   Keep business filters readable.
-   Aggregate only after applying filters.
-   Use meaningful aliases.

------------------------------------------------------------------------

## What I Learned

Timeline-based analyses are valuable for validating operational
controls. By joining restriction events with completed transactions,
analysts can quickly identify cases requiring further investigation
while maintaining efficient SQL patterns.
