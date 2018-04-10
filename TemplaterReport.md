Simple reports can be created using a commercial Rhetos plugin *TemplaterReport*, while complex reports are usually created using Reporting Services
(view and SQL procedures used for those reports are also written in the Rhetos scripts).

The following example contains a report that uses SQL query SqlInvoiceReport as a main data source, and SqlInvoiceProductReport for details.

If a report uses multiple data sources, declare them with `DataSources` concept.

This example uses an external template file "Invoice.doc".

```
Module Demo
{
    TemplaterReport PrintInvoice "Invoice.doc"
    {
        Guid InvoiceID;
                
        DataSources 'SqlInvoiceReport, SqlInvoiceProductReport';
    }
    
     FilterBy SqlInvoiceReport.'Demo.PrintInvoice' '(repository, parameter) =>
        repository.Demo.SqlInvoiceReport.Query()
            .Where(item => item.ID == parameter.InvoiceID.Value).ToArray()';

     FilterBy SqlInvoiceProductReport.'Demo.PrintInvoice' '(repository, parameter) =>
        repository.Demo.SqlInvoiceProductReport.Query()
            .Where(item => item.InvoiceID == parameter.InvoiceID.Value).ToArray()';
}
```

```
SqlQueryable SqlInvoiceReport
    "SELECT
        i.ID,
        i.InvoiceNumber,
        c.VATID,
        Contact = u.LastName + ' ' + u.FirstName,
        i.Total
    FROM
        Demo.Invoice i
        LEFT JOIN Demo.Company c ON c.ID = i.CompanyID
        LEFT JOIN Demo.User u ON u.ID = i.ContactID"
{
    Integer InvoiceNumber;
    ShortString VATID;
    ShortString Contact;
    Money Total;
}
```

```
SqlQueryable SqlInvoiceProductReport
    "SELECT
        ip.ID,
        ip.InvoiceID,
        OrdinalNumber = CAST(ROW_NUMBER() OVER (PARTITION BY ip.InvoiceID ORDER BY ip.CreatedTime) AS VARCHAR(5)) + '.',
        ProductName = p.Name,
        ip.Amount
    FROM
        Demo.InvoiceProduct ip
        LEFT JOIN Demo.Product p ON p.ID = ip.ProcuctID"
{
    Guid InvoiceID;
    ShortString OrdinalNumber;
    ShortString ProductName;
    Decimal Amount;
}
```