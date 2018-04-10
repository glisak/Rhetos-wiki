Use the `History` concept to manage temporal data in an entity.
The the concept is used on an entity, the last version of each record is written in the main table,
while older versions are archived in the "\_Changes" table.
The *ActiveSince* property is automatically added to the entity, each version of the record is effective from that time.

The following objects are created in the database:

* "ActiveSince" column in the entity's table.
* "\_Changes" table - A copy of the entity's table, contains EntityID reference to the ID of the main table.
* "\_History" view - Union of the current record and the archived records.
* "\_ChangesActiveUntil" view - Computed the active range for the records in "\_Changes" table
* "\_AtTime(@ContextTime DATETIME)" function - Returns the record that was active at the specified time.

**Rhetos:**

```
Module Demo
{
    Entity ContractStatus
    {
        History { AllProperties; }
        ShortString Name { Required; Unique; LookupVisible; }
        Bool Active { Required; }
    }
}
```

**SQL:**

``` sql
CREATE TABLE [Demo].[ContractStatus] (
    [ID] [uniqueidentifier] NOT NULL,
    [ActiveSince] [datetime] NULL,
    [Active] [bit] NULL,
    [Name] [nvarchar](256) NULL,
    CONSTRAINT [PK_ContractStatus] PRIMARY KEY NONCLUSTERED ([ID] ASC)
)
GO
ALTER TABLE [Demo].[ContractStatus] ADD  CONSTRAINT [DF_ContractStatus_ID]  DEFAULT (newid()) FOR [ID]
GO
```

``` sql
CREATE TABLE [Demo].[ContractStatus_Changes] (
    [ID] [uniqueidentifier] NOT NULL,
    [EntityID] [uniqueidentifier] NULL,
    [ActiveSince] [datetime] NULL,
    [Active] [bit] NULL,
    [Name] [nvarchar](256) NULL,
    CONSTRAINT [PK_ContractStatus_Changes] PRIMARY KEY NONCLUSTERED ([ID] ASC)
) ON [PRIMARY]
GO
ALTER TABLE [Demo].[ContractStatus_Changes] ADD  CONSTRAINT [DF_ContractStatus_Changes_ID]  DEFAULT (newid()) FOR [ID]
GO
ALTER TABLE [Demo].[ContractStatus_Changes]  WITH CHECK ADD  CONSTRAINT [FK_ContractStatus_Changes_ContractStatus_EntityID] FOREIGN KEY([EntityID])
REFERENCES [Demo].[ContractStatus] ([ID])
ON DELETE CASCADE
GO
ALTER TABLE [Demo].[ContractStatus_Changes] CHECK CONSTRAINT [FK_ContractStatus_Changes_ContractStatus_EntityID]
GO
```

``` sql
CREATE VIEW [Demo].[ContractStatus_History] AS
SELECT
    ID = entity.ID,
    EntityID = entity.ID,
    ActiveUntil = CAST(NULL AS DateTime),
    ActiveSince = entity.ActiveSince,
    Active = entity.Active,
    Name = entity.Name/*EntityHistoryInfo SelectEntityProperties Demo.ContractStatus*/
FROM
    Demo.ContractStatus entity

UNION ALL

SELECT
    ID = history.ID,
    EntityID = history.EntityID,
    au.ActiveUntil,
    ActiveSince = history.ActiveSince,
    Active = history.Active,
    Name = history.Name/*EntityHistoryInfo SelectHistoryProperties Demo.ContractStatus*/
FROM
    Demo.ContractStatus_Changes history
    LEFT JOIN Demo.ContractStatus_ChangesActiveUntil au ON au.ID = history.ID
GO
```

``` sql
ALTER VIEW [Demo].[ContractStatus_ChangesActiveUntil] AS
SELECT
    history.ID,
    ActiveUntil = COALESCE(MIN(newerVersion.ActiveSince), MIN(currentItem.ActiveSince))
FROM
    Demo.ContractStatus_Changes history
    LEFT JOIN Demo.ContractStatus_Changes newerVersion ON newerVersion.EntityID = history.EntityID AND newerVersion.ActiveSince > history.ActiveSince
    INNER JOIN Demo.ContractStatus currentItem ON currentItem.ID = history.EntityID
GROUP BY
    history.ID
GO
```

``` sql
CREATE FUNCTION [Demo].[ContractStatus_AtTime] (@ContextTime DATETIME)
RETURNS TABLE
AS
RETURN
SELECT
    ID = history.EntityID,
    ActiveUntil,
    EntityID = history.EntityID,
    ActiveSince = history.ActiveSince,
    Active = history.Active,
    Name = history.Name/*EntityHistoryInfo SelectHistoryProperties Demo.ContractStatus*/
FROM
    Demo.ContractStatus_History history
    INNER JOIN
    (
        SELECT EntityID, Max_ActiveSince = MAX(ActiveSince)
        FROM Demo.ContractStatus_History
        WHERE ActiveSince <= @ContextTime
        GROUP BY EntityID
    ) last ON last.EntityID = history.EntityID AND last.Max_ActiveSince = history.ActiveSince
GO
```
