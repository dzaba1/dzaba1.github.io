# SQL

## Joins

Join joins 2 tables so all columns.

### Inner

Returns rows which match with the comparer.
Examples:
```
select distinct e.FirstName, e.LastName, t.TerritoryDescription
from Employees e
join EmployeeTerritories et on e.EmployeeID = et.EmployeeID
join Territories t on t.TerritoryID = et.TerritoryID

select distinct e.FirstName, e.LastName, t.TerritoryDescription
from Employees e
inner join EmployeeTerritories et on e.EmployeeID = et.EmployeeID
inner join Territories t on t.TerritoryID = et.TerritoryID

select distinct e.FirstName, e.LastName, t.TerritoryDescription
from Employees e, EmployeeTerritories et, Territories t
where e.EmployeeID = et.EmployeeID
	and et.TerritoryID = t.TerritoryID
```

### Left

Returns all left rows with proper right rows. If the comparer doesn't match then NULL's on the right side.
Examples:
```
select e1.FirstName, e1.LastName, e2.FirstName, e2.LastName
from Employees e1
left join Employees e2 on e1.ReportsTo = e2.EmployeeID

select e1.FirstName, e1.LastName, e2.FirstName, e2.LastName
from Employees e1
left outer join Employees e2 on e1.ReportsTo = e2.EmployeeID
```

### Right

Returns all right rows with proper left rows. If the comparer doesn't match then NULL's on the left side.
Examples:
```
select e1.FirstName, e1.LastName, e2.FirstName, e2.LastName
from Employees e1
right join Employees e2 on e1.ReportsTo = e2.EmployeeID

select e1.FirstName, e1.LastName, e2.FirstName, e2.LastName
from Employees e1
right outer join Employees e2 on e1.ReportsTo = e2.EmployeeID
```

### Full

Combination of LEFT and RIGHT. Examples:
```
select e1.FirstName, e1.LastName, e2.FirstName, e2.LastName
from Employees e1
full join Employees e2 on e1.ReportsTo = e2.EmployeeID

select e1.FirstName, e1.LastName, e2.FirstName, e2.LastName
from Employees e1
full outer join Employees e2 on e1.ReportsTo = e2.EmployeeID
```

### Cross

Cartesian product. Joins everything from left to everything from right. Doesn't use any comparer.
Examples:
```
select e1.FirstName, e1.LastName, e2.FirstName, e2.LastName
from Employees e1
cross join Employees e2

select e1.FirstName, e1.LastName, e2.FirstName, e2.LastName
from Employees e1, Employees e2
```

## Primary key (PK)

Primary key identifies every row in some table.

Primary key can be made from multiple columns but at least one is required. There can be only one primary key in the table.

Values can't be NULLs and they have to be unique.

By default when a PK is made then also an unique clustered index is made.

Example:
```
CREATE TABLE Persons
(
ID int NOT NULL PRIMARY KEY,
)

ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY [CLUSTERED] (ID)

ALTER TABLE Persons
DROP CONSTRAINT pk_PersonID
```

## Stored procedures vs. functions

| Stored procedures | User defined functions |
| --- | --- |
| Can be called directly from .NET code. | Can't be called directly from .NET code. |
| Stored procedures can return zero, one or set. | Must return one value (scalar) or a set (table) |
| Stored procedure can have transactions. | Functions can't have transactions. |
| Stored procedures can have input and output parameters. | Functions can have only input parameters. |
| Stored procedure can call functions. | Functions can't call stored procedures. |
| Stored procedure can't be used in select, having, where, etc. | Function can be used in select, having, where, etc. |
| Stored procedure can use try-catch block and error handling. | Function can't use try-catch block and error handling. |
| Stored procedures can have update, insert, delete statements. | Functions can't have update, insert, delete statements. |
| Stored procedures are used to make batches and business logic. | Functions are used for computations. |
| Stored procedures can return everything. | Functions can't return images. |
| Stored procedure can have 21000 parameters. | Function can have 1023 parameters. |
| Stored procedure can use temporary tables. | Function can't use temporary tables. |
| Stored procedure can execute dynamic sql. | Functions can't execute dynamic sql. |
| Stored procedures use Deferred Name Resolution. | Functions don't use Deferred Name Resolution. |
| Stored procedure can use non-deterministic functions. | Functions can't use non-deterministic functions. |

## Deferred Name Resolution

When creating a stored procedure the SQL Server don't validate if used types or functions exist. It checks that when the stored procedure is invoked.

## How to pass a collection into a stored procedure?

* As some comma separated string
* XML
* As DataTable. Example:
```
CREATE TYPE StringList AS TABLE(
    Item NVARCHAR(MAX) NULL
);

CREATE PROCEDURE Foo
  @List AS StringList READONLY
AS
BEGIN
  SET NOCOUNT ON;

  SELECT Item FROM @List; 
END
GO

static void Main(string[] args)
{
    DataTable tvp = new DataTable();
    // define / populate DataTable from your List here

    using (SqlConnection conn = new SqlConnection("CS"))
    {
        SqlCommand cmd = new SqlCommand("Foo", conn);
        cmd.CommandType = CommandType.StoredProcedure;
        SqlParameter tvparam = cmd.Parameters.AddWithValue("@List", tvp);
        tvparam.SqlDbType = SqlDbType.Structured;
        // execute query, consume results, etc. here
    }
}
```

## What are views?

View can be a virtual table or stored query. A view is defined as a select.

It is used for:
- Security. We can restrict access to rows and columns.
- Comfort. It can make joins with multiple tables but caller sees it as an one table.
- Backward compatibility for tables which were removed.

To modify data in a view, the view has to return columns only from the base table.

## Deterministic and Nondeterministic Functions

Deterministic functions return always the same output for the same input. Examples: SIN, COS

Nondeterministic functions return different output for the same input. Examples: GETDATE

## Triggers

Triggers are kind of special kind of stored procedures, which are triggered automatically if there was some defined action.

Triggers can't be triggered manually.

There are multiple kinds of triggers:
- FOR/AFTER UPDATE, INSERT, DELETE
- INSTEAD OF UPDATE, INSERT, DELETE
- FOR/AFTER CREATE, ALTER, DROP, GRANT, DENY, REVOKE, UPDATE STATISTICS
- FOR/AFTER LOGON

## What is a common table expression (CTE)?

CTE is like a temporary, virtual view. A temporary set with name.

CTE can use itself in recursion.

Example:
```
WITH Moneyz_CTE (EmployeeID, Moneyz)
AS
(
    select e.EmployeeID,
		SUM((od.UnitPrice - od.Discount) * od.Quantity) as Moneyz
	from Employees e
	join Orders o on o.EmployeeID = e.EmployeeID
	join [Order Details] od on od.OrderID = o.OrderID
	group by e.EmployeeID
)

select e.*, mCTE.Moneyz
from Employees e
join Moneyz_CTE mCTE on e.EmployeeID = mCTE.EmployeeID
go
```

## Physical joins

Physical join is an algorithm which is used internally by SQL Server for real joining two tables.

The optimizer decides about the algorithm by checking the parsed query, indexes, statistics and estimated number of rows.

There are 3 algorithms.

### Nested loops

The easiest one to make.

```
var result;
foreach (var row1 in table1)
{
    foreach (var row in table2) // could also use some index from the table2
    {
        if (IsMatch(row1, row2))
        {
            result.Add(row1, row2);
        }
    }
}
```

Complexity is O(NlogM) where N is rows count from the left table and M is rows count from the right table.

The optimizer selects this algorithm when number of left rows is small and right columns which are used for the join are indexed.

Nested loops support every join paradigm.

### Merge join

Rows from both table has to be sorted by the join key.

Supports only the equality paradigm (=) because data has to be sorted in the same way.

The best way for merging a lot of data because there's only one loop.

```
var result;
IEnumerable<object> table1;
IEnumerable<object> table2;
IEnumerator<object> row1 = table1.GetEnumerator();
IEnumerator<object> row2 = table2.GetEnumerator();

while (!row1.IsEnd && !row2.IsEnd)
{
    if (IsMatch(row1.Current, row2.Current))
    {
        result.Add(row1.Current, row2.Current);
        row2.MoveNext();
    }
    else if (row1.Current < row2.Current)
        row1.MoveNext();
    else
        row2.MoveNext();
}
```

Complexity is O(N+M)

### Hash match

Selected when the data is not sorted by the key and there aren't any indexes.

Supports only the equality paradigm (=) because there has to be the same hash algorithm.

Requires some additional memory for building the hash table. If there aren't any memory then SQL Server will make the hash table on the TEMPDB on a disc.

```
var result;
IEnumerable<object> table1;
IEnumerable<object> table2;

IEnumerable<object> buildTable;
IEnumerable<object> probeTable;
Dictionary<int, List<object>> hashTable = new Dictionary<int, List<object>>();

if (table1.Count() <= table2.Count())
{
    buildTable = table1;
    probeTable = table2;
}
else
{
    buildTable = table2;
    probeTable = table1;
}

foreach (var row in buildTable)
{
    int hash = row.GetHashCode();
    hashTable.AddRow(hash, row);
}

foreach (var row in probeTable)
{
    int hash = row.GetHashCode();
    foreach (var row2 in hashTable.GetRows(hash))
    {
        if (IsMatch(row, row2))
            result.Add(row, row2);
    }
}
```

Complexity is `O(N*hc + M*hm)` where hc is a building hash table complexity and hm is hash function complexity.

## Indexes

Physical representation of an index is a B-Tree. Logically an index is some value copy. Two types: clustered and non-clustered.

When there are a lot of indexed then data modification is slower. And when there are really a lot of indexes then queries are slower because the optimizer will start to check a lot of possibilities for running the query.

### Clustered

Can be only one in a table. It makes then the physical table data representation. In leafs, there are data, not pointers to data.

The index can be made from multiple columns.

In root and in the middle layers of the B-Tree pages have pointers to next pages in a form of doubled linked list. 

Data are always sorted.

Clustered indexes have one row in sys.partitions with index_id = 1 for every partition. When an index has multiple partitions then every partition is a B-Tree with data from that partition.

In the table sys.system_internals_allocation_units the column root_page specifies the root for that partition.

### Non-clustered

The physical structure is the same as in the clustered index. But:
- Leafs don't contain data but pointers to data.
- Data is not sorted

When a table doesn't contain the clustered index then leaf pointers point to data on the heap. When clustered index exist then leaf pointers point to data on the clustered index leafs.

Non-clustered indexes have one row in sys.partitions with index_id > 0 for every partition. When an index has multiple partitions then every parton is a B-Tree with data from that partition.

In a table there can be max 999 non-clustered indexes.

## What’s the index rebuild and recreation?

When there are updates, inserts, deletes then SQL Server tries to reorganize indexes.
After some time there can be the fragmentation phenomena. It happens when pages in indexes which are sorted logically don't match physical data layout on a disc.
When fragmentation increases then queries are slower because the disc head has more work to do.

Index rebuilding means deleting the index, collecting the free space and making the index again. When option ALL specified then all indexes are rebuild in one transaction.

Index recreation means data defragmentation to the logical data layout. It consumes less resources then rebuilding.

To select the way first we can check sys.dm_db_index_physical_stats. One column returns fragmentation in %. When it's between 5% and 30% then we can recreate the index.
If more then 30% then it's better to rebuild.

## Truncate

Deleted every row from a table. It doesn't log anything and also it doesn't trigger triggers.

So it's like a DELETE without WHERE and it's faster.

## NOLOCK

`WITH (NOLOCK)` means that data are 'read uncommited'. There can be some dirty reading when some rollback happens in parallel.

## Transactions isolations levels

TODO

### Read Uncommited

Makes that we can read rows which are not yet commited by some other transaction.

So locks are not checked at all. There can be some dirty reading when some rollback happens in parallel.

### Read Commited

TODO

## Collaction

It specified the encoding.

TODO.

## How to get number of rows affected by the statement?

We can use `@@ROWCOUNT` which returns number rows affected by the last statement. But when there are more than 2 billions of rows then we must use `@@ROWCOUNT_BIG`.