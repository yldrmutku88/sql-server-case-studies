# Active Service Point Report

## Overview

This document presents an anonymized SQL reporting example that
retrieves agency and service point information together with regional
assignments and activation details.

All database objects and business-specific identifiers have been
generalized to protect proprietary information while preserving the
reporting approach.

------------------------------------------------------------------------

## Business Scenario

Operations teams may need to identify service points that have been
activated but have not yet processed any completed transactions.

The query combines organizational hierarchy, location metadata,
activation history, and transaction validation into a single report.

------------------------------------------------------------------------

## Features

-   Agency and location reporting
-   Status mapping
-   Manager hierarchy
-   Activation date extraction
-   First activation detection using window functions
-   Exclusion of locations with completed transactions

------------------------------------------------------------------------

## SQL Concepts Used

-   LEFT JOIN
-   ROW_NUMBER()
-   Window Functions
-   CASE
-   Subqueries
-   YEAR / MONTH
-   NOT IN

------------------------------------------------------------------------

## Sample Query

``` sql
SELECT
    a.AgencyCode,
    a.AgencyName,
    l.LocationId,
    l.LocationName,
    mgr.ManagerName,
    area.AreaManagerName,
    act.ActivationDate,
    YEAR(act.ActivationDate) AS ActivationYear,
    MONTH(act.ActivationDate) AS ActivationMonth,
    region.RegionName,
    sector.SectorName

FROM Locations l

LEFT JOIN Agencies a
    ON a.AgencyId = l.AgencyId

LEFT JOIN Managers mgr
    ON mgr.ManagerId = l.ManagerId

LEFT JOIN AreaManagers area
    ON area.ManagerId = l.AreaManagerId

LEFT JOIN (
    SELECT
        LocationId,
        ActivationDate
    FROM (
        SELECT
            LocationId,
            ActivationDate,
            ROW_NUMBER() OVER (
                PARTITION BY LocationId
                ORDER BY ActivationDate
            ) AS rn
        FROM LocationHistory
        WHERE Status = 'Active'
    ) x
    WHERE rn = 1
) act
    ON act.LocationId = l.LocationId

WHERE l.LocationId NOT IN (
    SELECT LocationId
    FROM Transactions
    WHERE Status = 'Completed'
);
```

------------------------------------------------------------------------

## Performance Notes

Recommended indexes:

-   LocationId
-   AgencyId
-   Status
-   ActivationDate

For large datasets, consider using `NOT EXISTS` instead of `NOT IN` if
NULL handling or execution plans become a concern.

------------------------------------------------------------------------

## Skills Demonstrated

-   SQL Server
-   Reporting
-   Window Functions
-   Operational Analytics
-   Data Validation
-   Query Optimization

------------------------------------------------------------------------

## Best Practices

-   Use descriptive aliases.
-   Filter historical records before joining.
-   Prefer window functions to identify first or latest records.
-   Review execution plans when using exclusion queries.

------------------------------------------------------------------------

## What I Learned

Operational reporting often requires combining master data, historical
records, and transactional data. Window functions and carefully
structured joins provide a reliable way to produce accurate business
reports while remaining performant.
