Persisting the computed data is done by developing the data source that computed the data (usually an `SqlQueryable`),
and saving the result into the database table.

Rhetos contains concepts that help automate the implementation:
`KeepSynchronized`, `ChangesOnLinkedItems`, `ChangesOnChangedItems`, `ChangesOnBaseItem` and `ChangesOnReferenced`.

Example:

**Rhetos:**
```
Module Demo
{
    SqlQueryable SubjectDs
    "
        SELECT
            s.ID,
            NrlCode = g.Code + '.' + CAST(s.NrlCode AS NVARCHAR(100)),
            subjectLastVersion.RnkAsc,
            subjectLastVersion.RnkDesc,
            IsFirstVersion = CAST(CASE WHEN subjectLastVersion.RnkAsc = 1 THEN 1 ELSE 0 END AS BIT),
            IsLastVersion = CAST(CASE WHEN subjectLastVersion.RnkDesc = 1 THEN 1 ELSE 0 END AS BIT),
            StatusSubjectID = ss.ID
        FROM
            Demo.Subject s
            LEFT JOIN
            (
                SELECT
                    s.ID,
                    RnkAsc = ROW_NUMBER() OVER (PARTITION BY s.NrlCode ORDER BY s.Version ASC),
                    RnkDesc = ROW_NUMBER() OVER (PARTITION BY s.NrlCode ORDER BY s.Version DESC),
                    StatusProductID =
                        (SELECT TOP 1
                            ssh.StatusSubjectID
                        FROM
                            Demo.StatusSubjectHistory ssh
                        WHERE
                            s.ID = ssh.SubjectID AND ssh.EntityType = 'Subject'
                        ORDER BY
                            ssh.SetupTime DESC)
                FROM
                    Demo.Subject s
            ) subjectLastVersion ON subjectLastVersion.ID = s.ID
            LEFT JOIN Demo.Group g ON s.GroupID = g.ID
            LEFT JOIN Demo.StatusProduct ss
                WITH (INDEX(IX_StatusProduct_ID_Includes))
                ON ss.ID = subjectLastVersion.StatusProductID
            LEFT JOIN Demo.NrlCountry country ON country.ID = s.NrlCountryID
    "
    {
        Extends Demo.Subject;

        ChangesOnReferenced 'Base.Group';
        
        ChangesOnLinkedItems Demo.StatusSubjectHistory.Subject;

        ChangesOnChangedItems Demo.Subject 'Guid[]' ' changedItems =>
        {
            var changedItemIds = changedItems.Select(item => item.ID).ToList();
            var nrlCodes = _domRepository.Demo.Subject.Query()
                .Where(s => changedItemIds.Contains(s.ID))
                .Select(s => s.NrlCode)
                .Distinct()
                .ToList();

            return _domRepository.Demo.Subject.Query()
                .Where(item => nrlCodes.Contains(item.NrlCode))
                .Select(item => item.ID)
                .ToArray();
        }';
        
        ShortString NrlCode;
        Integer RnkAsc;
        Integer RnkDesc;
        Bool IsFirstVersion;
        Bool IsLastVersion;
        Reference StatusSubject Demo.StatusProduct;
    }
}
```

```
Module Demo
{
    Entity SubjectPersisted
    {
        ComputedFrom Demo.SubjectDs
        {
            AllProperties;
            KeepSynchronized;
        }
    }
}
```
