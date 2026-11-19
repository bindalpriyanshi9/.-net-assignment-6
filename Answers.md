# Lab Assignment 6 - ADO.NET (SQL Server) Solutions

## 1. Database setup (do this first)
1. Open **SQL Server Management Studio** (or Visual Studio > SQL Server Object Explorer) and run `StudentDB_Setup.sql`.
   Command-line alternative:
   ```
   sqlcmd -S "(localdb)\MSSQLLocalDB" -i StudentDB_Setup.sql
   ```
   It creates database `StudentDB` and table `Student` (StudentId INT PRIMARY KEY, Name, Course, Age, Email) with 3 sample rows.
2. Every project has the connection string in its `App.config`:
   ```xml
   <add name="StudentDB" providerName="System.Data.SqlClient"
        connectionString="Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=StudentDB;Integrated Security=True" />
   ```
   If you use SQL Server Express change `Data Source` to `.\SQLEXPRESS` (or `localhost`, or your server name).
   For SQL authentication use: `Data Source=SERVER;Initial Catalog=StudentDB;User ID=sa;Password=yourPassword`.
3. Open `LabAssignment6.slnx` in Visual Studio, right-click a project > *Set as Startup Project* > **Ctrl+F5**.
   All projects reference `System.Data` and `System.Configuration` (for `ConfigurationManager`).

## 2. Project list
| # | Project | Concept |
|---|---------|---------|
| 1 | `Q1_ReadStudents_Reader` | SqlConnection + SqlCommand + SqlDataReader (display all) |
| 2 | `Q2_InsertStudent_Connected` | INSERT with parameterised SqlCommand.ExecuteNonQuery |
| 3 | `Q3_UpdateStudent_Connected` | UPDATE with SqlCommand |
| 4 | `Q4_DeleteStudent_Connected` | DELETE by Student ID |
| 5 | `Q5_DisplayStudents_DataSet` | SqlDataAdapter.Fill(DataSet) |
| 6 | `Q6_InsertStudent_DataSet` | DataRow added to DataSet + adapter.InsertCommand + Update() |
| 7 | `Q7_UpdateStudent_DataSet` | Edit DataRow + adapter.UpdateCommand + Update() |
| 8 | `Q8_DeleteStudent_DataSet` | row.Delete() + adapter.DeleteCommand + Update() |
| 9 | `Q9_CommandBuilder_CRUD` | SqlCommandBuilder auto-generates INSERT/UPDATE/DELETE (menu-driven) |
| 10 | `Q10_Connected_vs_Disconnected` | Full CRUD with both architectures + comparison (menu option 3) |

All SQL uses **parameters** (not string concatenation) to prevent SQL injection, and `using` blocks so connections are always closed.

## 3. Q10 - Comparison of Connected vs Disconnected ADO.NET

| Feature | Connected | Disconnected |
|---|---|---|
| Key classes | `SqlConnection`, `SqlCommand`, `SqlDataReader` | `SqlDataAdapter`, `DataSet`/`DataTable`, `SqlCommandBuilder` |
| Connection | Must stay open while data is read/written | Opened only inside `Fill()` / `Update()`, then closed |
| Where data lives | Streamed from the server row by row | Cached in memory in the DataSet |
| Navigation | Forward-only, read-only (DataReader) | Move in any direction, edit, sort, filter |
| Performance / memory | Fast, very low memory | Slightly slower to load, uses more memory |
| Data freshness | Always latest data | Can become stale; call `Refresh()` |
| Scalability | Each user holds a connection (limits users) | Releases connections quickly - scales better |
| Offline use | Not possible | Possible (DataSet can be saved as XML) |
| Concurrency | Each statement runs immediately (last write wins) | Optimistic concurrency: `Update()` detects conflicts |
| Coding style | Write a SQL statement per operation | Change rows in the DataSet, call `Update()` once; builder generates the SQL |
| Multiple tables / relations | Manual joins | DataSet supports several tables + `DataRelation` |
| Best suited for | Reports, quick lookups, single operations | Data-entry forms/grids, batch updates, mobile/offline apps |

**Implementation difference in this project**
- *Connected* - `ConnectedCrud`: each method creates a connection, opens it, runs one `SqlCommand` (`ExecuteReader` for read, `ExecuteNonQuery` for create/update/delete) and closes it.
- *Disconnected* - `DisconnectedCrud`: the table is loaded once with `da.Fill`; insert = `NewRow` + `Rows.Add`, update = change a `DataRow`, delete = `row.Delete()`; then `da.Update(ds, "Student")` sends only the changed rows (the `SqlCommandBuilder` supplies the INSERT/UPDATE/DELETE commands).
- Both classes implement the same `IStudentCrud` interface, so the same menu code drives both and the differences are easy to see.

## 4. Notes
- `SqlCommandBuilder` works only when the SELECT returns the primary key of a single table - that is why `MissingSchemaAction.AddWithKey` is used.
- For a deleted row the key parameter uses `DataRowVersion.Original` (a deleted row no longer has a *Current* version).
- Error 2627 (primary key violation) is caught to show "Student ID already exists".
