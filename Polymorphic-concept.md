Table of contents:

* [Essential info](#essential-info)
* [Usage](#usage)
* [Referencing or extending a polymorphic entity](#referencing-or-extending-a-polymorphic-entity)
* [Multiple interface implementations](#multiple-interface-implementations)
* [Specific implementation with SQL query](#specific-implementation-with-sql-query)
* [Writing efficient queries from client application](#writing-efficient-queries-from-client-application)

## Essential info

Polymorphic concept is intended for implementing the application design pattern where multiple entities share common interface for reading data (defined as a list of common properties).

Similar features:
* Polymorphic concept is similar to object-oriented concept of class inheritance, but class inheritance is not used here.
* Polymorphic concept can sometimes be used as an alternative to entity Extension.

## Usage

For example, `MoneyTransaction` data structure can have multiple forms: `BorrowMoney` and `LendMoney` (see the example below).

* In this example, `MoneyTransaction` can be seen as an *interface* that is *implemented* by the concrete subtypes `BorrowMoney` and `LendMoney`.
* `MoneyTransaction` is readable data structure, it returns **union of records** from  `BorrowMoney` and `LendMoney`.

Example:

	Module Demo
	{
		Polymorphic MoneyTransaction
		{
			DateTime EventDate;
			Money Amount;
		}
		
		Entity BorrowMoney
		{
			ShortString FromWhom;
			Money Amount;
			
			Is Demo.MoneyTransaction;
			// EventDate will be automatically added.
		}
		
		Entity LendMoney
		{
			ShortString ToWhom;
			Money Amount;
			
			Is Demo.MoneyTransaction
			{
				Implements Demo.MoneyTransaction.Amount '-Amount';
				// Amount in the MoneyTransaction related to the LendMoney record will have negative value.  
			}
		}
	}

## Referencing or extending a polymorphic entity

A polymorphic data structure may be referenced or extended by other entites (Browse is also an extension).
Rhetos will check the **reference constraint** on data modifications.

	Module Demo
	{
		Entity TransactionComment
		{
			Reference MoneyTransaction;
			LongString Comment;
		}
	}

Internally, the reference is implemented as a foreign key in database: from the `TransactionComment` table to the automatically generated `MoneyTransaction_Key` table.
The `MoneyTransaction_Key` table contains union of IDs from all MoneyTransaction subtypes: `BorrowMoney` and `LendMoney`.
The redundant IDs are automatically updated when inserting or deleting the subtype entities' records.

## Multiple interface implementations

An entity may implement multiple interfaces.
It can even implement the same interface multiple times, using additional string parameter ImplementationName to distinguish the implementations.

For example, the MoneyTransfer record may be implemented as two money transactions: subtracting from one account and adding to the other account.

## Specific implementation with SQL query

Instead of using property implementations (`Implements` keyword), a specific SQL query may be provided to implement the mapping between the subtype and the polymorphic entity.

This example is an alternative implementation of the `LendMoney` subtype (see the original implementation above):

    Module Demo
    {
        Entity LendMoney
        {
            ShortString ToWhom;
            Money Amount;
            
            Is Demo.MoneyTransaction
            {
                SqlImplementation "SELECT lm.ID, lm.EventDate, Amount = -lm.Amount FROM Demo.LendMoney lm"
                {
                    AutoDetectSqlDependencies;
                }
            }
        }
    }

`SqlImplementation` may cover more complex scenarios than per-property implementations (`Implementats`), but it lacks readability and extensibility such as adding a custom property to the subtype from another package.

## Writing efficient queries from client application

When reading a polymorphic entity, filtering by subtype can be evaluated efficiently in SQL Server if the filter is based on a subtype reference property.
For example, when reading records from `MoneyTransaction` of subtype `LendMoney` through REST API, the following filter should be used:

	http://localhost/Rhetos/rest/Demo/Perf/?filters=[{"Property":"LendMoney","Operation":"NotEqual","Value":null}]

The request above will be translated to an SQL query similar to `SELECT * FROM Demo.MoneyTransaction WHERE LendMoneyID IS NOT NULL`.
SQL Server will optimize the query's execution plan so that only `Demo.LendMoney` table will be scanned.

A similar optimization is done in database when filtering by subtype: `SELECT * FROM Demo.MoneyTransaction WHERE Subtype = 'Demo.LendMoney'`.
Unfortunately, if such filter is defined in LINQ query, it will generate a different WHERE part:
`WHERE Subtype = @p0`, and the LINQ query will cause reading both `Demo.LendMoney` and `Demo.BorrowMoney` tables.
