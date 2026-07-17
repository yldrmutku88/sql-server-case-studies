# Transaction Processing Time Analysis

## Overview

This document presents an anonymized SQL Server analysis for measuring
the time spent between different stages of a transaction lifecycle.

The original database objects have been generalized while preserving the
analytical logic.

------------------------------------------------------------------------

## Business Scenario

Operations teams often need to measure how long transactions spend in
each processing stage to identify bottlenecks and monitor service
performance.

This report calculates the elapsed time between status changes and
identifies the users responsible for key workflow steps.

------------------------------------------------------------------------

## Features

-   Transaction lifecycle analysis
-   Processing time calculation
-   Workflow stage comparison
-   First status occurrence detection
-   User attribution
-   SLA monitoring

------------------------------------------------------------------------

## SQL Concepts Used

-   Common joins
-   Window Functions (`ROW_NUMBER`)
-   Conditional aggregation
-   `DATEDIFF`
-   `MIN`
-   `CASE`
-   `COALESCE`

------------------------------------------------------------------------

## Sample Query

``` sql
SELECT
    t.TransactionId,
    YEAR(t.TransactionDate) AS TransactionYear,
    MONTH(t.TransactionDate) AS TransactionMonth,

    h.CreatedDate,
    h.ReviewDate,
    h.CompletedDate,

    DATEDIFF(
        SECOND,
        h.CreatedDate,
        h.ReviewDate
    ) AS ReviewDurationSeconds,

    DATEDIFF(
        SECOND,
        COALESCE(h.CreatedDate, h.ReviewDate),
        h.CompletedDate
    ) AS CompletionDurationSeconds,

    reviewUser.UserName,
    completeUser.UserName

FROM Transactions t

LEFT JOIN TransactionHistorySummary h
    ON h.TransactionId = t.TransactionId

LEFT JOIN Users reviewUser
    ON reviewUser.UserId = h.CreatedBy

LEFT JOIN Users completeUser
    ON completeUser.UserId = h.CompletedBy;
```

------------------------------------------------------------------------

## Performance Notes

Recommended indexes:

-   TransactionId
-   StatusId
-   TransactionDate
-   EventDate

Window functions should operate on indexed transaction identifiers to
improve performance on large history tables.

------------------------------------------------------------------------

## Skills Demonstrated

-   SQL Server
-   SLA Reporting
-   Operational Analytics
-   Window Functions
-   Performance Measurement
-   Data Analysis

------------------------------------------------------------------------

## Best Practices

-   Capture the first occurrence of workflow events.
-   Use `COALESCE` to handle missing intermediate stages.
-   Keep duration calculations in seconds for consistent reporting.
-   Index history tables used in window functions.

------------------------------------------------------------------------

## What I Learned

Measuring stage-to-stage processing times provides valuable operational
insights. Combining window functions with conditional aggregation
enables accurate workflow reporting while keeping SQL queries
maintainable.
