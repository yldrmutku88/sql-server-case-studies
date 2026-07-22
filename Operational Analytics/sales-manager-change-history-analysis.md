# Sales Manager Change History Analysis

## Overview

This case study demonstrates how to analyze assignment history by
identifying changes in responsible managers over time.

The solution reconstructs historical ownership information using SQL
Server window functions while preserving the complete change timeline.

> **Note:** All database objects, identifiers and business terminology
> have been anonymized.

------------------------------------------------------------------------

## Business Scenario

Operations teams frequently need to answer:

-   Who was the first assigned manager?
-   Has the manager changed?
-   When did the change occur?
-   How many changes have been made?

This query rebuilds assignment history directly from audit logs.

------------------------------------------------------------------------

## Features

-   Assignment history reconstruction
-   First assignment detection
-   Change tracking
-   Historical comparison
-   Chronological reporting

------------------------------------------------------------------------

## SQL Concepts Used

-   CTE
-   LAG()
-   ROW_NUMBER()
-   Window Functions
-   LEFT JOIN
-   INNER JOIN
-   CASE

------------------------------------------------------------------------

## Sample Query (Anonymized)

``` sql
WITH AssignmentHistory AS (
    SELECT
        LocationId,
        ManagerId,
        EventDate,
        LAG(ManagerId) OVER (
            PARTITION BY LocationId
            ORDER BY EventDate
        ) AS PreviousManager
    FROM AssignmentLog
),
FirstAssignment AS (
    SELECT
        LocationId,
        ManagerId,
        EventDate,
        ROW_NUMBER() OVER (
            PARTITION BY LocationId
            ORDER BY EventDate
        ) AS RowNum
    FROM AssignmentLog
),
AssignmentChanges AS (
    SELECT
        LocationId,
        PreviousManager,
        ManagerId,
        EventDate,
        ROW_NUMBER() OVER (
            PARTITION BY LocationId
            ORDER BY EventDate
        ) AS ChangeNumber
    FROM AssignmentHistory
    WHERE PreviousManager IS NOT NULL
      AND PreviousManager <> ManagerId
)
SELECT
    l.LocationId,
    l.LocationName,
    fa.EventDate,
    oldMgr.FullName,
    newMgr.FullName,
    ac.ChangeNumber
FROM Locations l
LEFT JOIN FirstAssignment fa
    ON fa.LocationId=l.LocationId
LEFT JOIN AssignmentChanges ac
    ON ac.LocationId=l.LocationId;
```

------------------------------------------------------------------------

## Performance Notes

Recommended indexes:

-   LocationId
-   ManagerId
-   EventDate

------------------------------------------------------------------------

## Skills Demonstrated

-   SQL Server
-   Operational Analytics
-   Audit Log Analysis
-   Window Functions
-   Query Optimization

------------------------------------------------------------------------

## Best Practices

-   Preserve historical data.
-   Use LAG() for comparisons.
-   Use ROW_NUMBER() to identify the first assignment.
-   Keep audit reports chronological.

------------------------------------------------------------------------

## What I Learned

Audit trail analysis can be efficiently implemented using SQL Server
window functions to reconstruct historical assignment changes while
maintaining excellent readability.

------------------------------------------------------------------------

## Tags

`SQL Server` `T-SQL` `CTE` `LAG` `ROW_NUMBER` `Operational Analytics`
