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

## Reference constraints

A polymorphic data structure may be referenced from other entities and the Rhetos business object model will check the **reference constraint** when inserting or deleting data.     

	Module Demo
	{
		Entity TransactionComment
		{
			Reference MoneyTransaction;
			LongString Comment;
		}
	}

Internally, the reference is implemented as a foreign key in database, from the `TransactionComment` table to the `MoneyTransaction_Key` table. `MoneyTransaction_Key` table contains union of IDs from all MoneyTransaction subtypes: `BorrowMoney` and `LendMoney`. The redundant IDs are automatically updates when inserting or deleting the subtype entities' records.

## Multiple interfaces

An entity may implement multiple interfaces; it can even implement the same interface multiple times.

For example, some kind of money transfer record may be implemented as two money transactions: subtracting from one account and adding to the other account.

## Efficient queries

When reading a polymorphic entity, filtering by subtype can be evaluated efficiently in SQL Server if the filter is based on a subtype reference property. For example, when reading records from `MoneyTransaction` of subtype `LendMoney` through REST API, the following filter should be used:

	http://localhost/Rhetos/rest/Demo/Perf/?filters=[{"Property":"LendMoney","Operation":"NotEqual","Value":null}]

The request above will be translated to an SQL query similar to `SELECT * FROM Demo.MoneyTransaction WHERE LendMoneyID IS NOT NULL`. SQL Server will optimize the query's execution plan so that only `Demo.LendMoney` table will be scanned.

A similar optimization is done in database when filtering by subtype: `SELECT * FROM Demo.MoneyTransaction WHERE Subtype = 'Demo.LendMoney'`. Unfortunately, if such filter is defined in LINQ query, it will generate a different WHERE part: `WHERE Subtype = @p0`, and the LINQ query will cause reading both `Demo.LendMoney` and `Demo.BorrowMoney` tables.
