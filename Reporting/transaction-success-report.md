# Transaction Success Report

## Overview

This document demonstrates a generic SQL Server reporting query that
retrieves successful payment transactions by combining customer, card,
merchant, currency, and transaction information.

> **Note:** All object names have been anonymized. The business logic is
> preserved while removing organization-specific details.

------------------------------------------------------------------------

## Business Scenario

Support and operations teams frequently need a single report to
investigate successful financial transactions.

This query demonstrates how multiple data sources can be combined into
one report while keeping the query readable and maintainable.

------------------------------------------------------------------------

## Features

-   Transaction reporting
-   Customer information
-   Card information
-   Merchant information
-   Currency mapping
-   Status mapping using `CASE`
-   JSON parsing
-   Date range filtering
-   Ordered reporting

------------------------------------------------------------------------

## SQL Concepts Used

-   `LEFT JOIN`
-   `CASE`
-   `TRY_CAST`
-   `JSON_VALUE`
-   `ISJSON`
-   `DATEADD`
-   `ORDER BY`

------------------------------------------------------------------------

# Sample Query

``` sql
SELECT
    c.CustomerNumber,
    c.FullName,
    t.TransactionId,
    t.CardNumber,
    t.TransactionDate,
    t.ReferenceNumber,
    t.AuthorizationCode,

    CASE
        WHEN card.Status = 'A' THEN 'Active'
        WHEN card.Status = 'B' THEN 'Pending'
        WHEN card.Status = 'F' THEN 'Fraud'
        WHEN card.Status = 'I' THEN 'Cancelled'
        ELSE 'Unknown'
    END AS CardStatus,

    m.MerchantName,
    m.CategoryCode,

    t.Amount,
    cur.CurrencyCode,

    CASE
        WHEN ISJSON(t.ResultJson) = 1
        THEN JSON_VALUE(t.ResultJson,'$.ResponseCode')
    END AS ResponseCode,

    CASE
        WHEN ISJSON(t.ResultJson) = 1
        THEN JSON_VALUE(t.ResultJson,'$.ResponseMessage')
    END AS ResponseMessage

FROM Transactions t

LEFT JOIN Customers c
    ON c.CustomerId = t.CustomerId

LEFT JOIN Cards card
    ON card.CardId = t.CardId

LEFT JOIN Merchants m
    ON m.MerchantId = t.MerchantId

LEFT JOIN Currency cur
    ON cur.CurrencyId = t.CurrencyId

WHERE
    t.TransactionDate >= '2025-01-01'
    AND t.Status = 'Success'

ORDER BY
    t.TransactionDate DESC;
```

------------------------------------------------------------------------

## Performance Notes

Recommended indexes:

-   TransactionDate
-   Status
-   CustomerId
-   MerchantId

Apply filtering before JSON parsing to reduce execution cost.

------------------------------------------------------------------------

## Expected Output

  Customer   Transaction     Amount Merchant   Status
  ---------- ------------- -------- ---------- ---------
  Example    TX-100001       100.00 Store A    Success

------------------------------------------------------------------------

## Skills Demonstrated

-   SQL Server
-   Reporting
-   Data Analysis
-   JSON Parsing
-   Query Optimization
-   Financial Transaction Analysis

------------------------------------------------------------------------

## Best Practices

-   Avoid exposing production database names.
-   Replace business-specific identifiers with generic names.
-   Keep reporting queries readable.
-   Use explicit aliases.
-   Validate JSON before parsing.
-   Filter early for better performance.

------------------------------------------------------------------------

## What I Learned

Building consolidated operational reports requires balancing
readability, performance, and maintainability. Generic SQL patterns like
joins, CASE expressions, and JSON parsing can be reused across many
reporting scenarios without exposing proprietary database structures.
