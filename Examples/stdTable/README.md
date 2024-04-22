# stdTable

This example, is kind of a library in-of itself. `stdTable`, found in the `src` folder can be used to perform manipulations on tables in a simple declarative manor which mimics SQL.

```vb
Attribute VB_Name = "Module1"

Sub main()
  Call stdTable.CreateFromTableByName("Table3") _
           .GroupByField("2", "group") _
           .RenameFields("2", "Type") _
           .AddField("Count", stdLambda.Create("$1.group.length")) _
           .AddField("Sum5", stdLambda.Create("$1.group.sum(lambda(""$1.item(""""5"""")""))")) _
           .ToListObject(Sheet3.Range("A1"))
End Sub
```

This interface aims to be similar in style to the functionality in PowerQuery, but instead with a code first approach.

## Requirements

- `stdEnumerator`
- `stdCallback`
- `stdICallable`
- `stdJSON` - For JSON parsing and exporting

`stdLambda` is used in a number of examples. But any class which implements `stdICallable` is valid in their place.

## Constructors

- Constructors
  - `CreateFromListObject` - Create an Excel list object
  - `CreateFromTableByName` - Create an Excel table object by the name of the table
  - `CreateFromRecordSet` - Create a table object from a ADODB record set
- Instance Properties
  - `Get/Let` Name - The name of the table
  - `Get` Headers - The current headers of the table. These are the headers which will be exported when using a To\_\_\_() method.
  - `Get` Rows - Obtain the rows of the table as a `stdEnumerator`.
- Instance Methods
  - `FieldsSelect` - Select the required fields for the table (i.e. remove fields)
  - `RenameFields` - Rename a number of fields to a new name.
  - `AddField` - Add a field, and populate it with some callback.
  - `UpdateField` - Update a field to the result of some callback.
  - `UpdateFieldStatic` - Update a field to a static value.
  - `ForEach` - Execute a callback over each row of the table - Commonly used for updating multiple fields.
  - `Filter` - Filter a table based on some callback.
  - `Join` - Join one table with another table based on some common key.
  - `Concat` - Concatenate 2 tables together.
  - `AddRow` - Add a single row to a table (slow! - use concat for multiple rows)
  - `Reverse` - Reverse the rows in the table.
  - `GroupByField` - Group rows of this table by a particular field.
  - `GroupBy` - Group rows of this table by a key generated from a callback.
  - `Clone` - Clone this table
  - `Unique` - Remove duplicates from a table based key generated by a callback.
  - `ToArray2D` - Convert the table into a 2D array
  - `ToListObject` - Import the table into an Excel ListObject
  - `ToRecordSet` - Create an ADODB recordset based on the data inside the table.
  - `ToCollection` - Creates a Collection of dictionaries from the table.
  - `ToJSON` - Requires `stdJSON` Dump the data to a JSON string.
  - `ToJSONFile` - Requires `stdJSON` Dump the data to a JSON file.

## Currying to increase functionality

One of the concerns that people may have is the difficulty in accessing fields like field `5` above, and ensuring that you have the number of quotes correct. To make this easier we can use currying (functions which return functions) to generate functions like `getter` here, which can be used to get fields for you.

```vb
Attribute VB_Name = "Module1"

Sub main()
  Call stdTable.CreateFromTableByName("Table3") _
           .GroupByField("2", "group") _
           .RenameFields("2", "Type") _
           .AddField("Count", query("$1.group.length")) _
           .AddField("Sum5", query("$1.group.sum(getter(""5""))")) _
           .ToListObject(Sheet3.Range("A1"))
End Sub

'Generate a `stdLambda` instance with additional bound functions
'@param expression - Lambda expression with use of additional functions if needed
'@returns - A lambda object
Public Function query(ByVal expression As String) As stdLambda
  Set query = stdLambda.Create(expression)
  Set query.oFunctExt("lambda") = stdCallback.CreateFromModule("Module1", "query")   'rebind lambda to this function.
  Set query.oFunctExt("getter") = stdCallback.CreateFromModule("Module1", "fieldGetter")
End Function

'Currying - Return a lambda with data bound to it.
'@param field - Field to obtain
'@returns stdLambda<(record: Object<Dictionary<string,variant>>)=>variant>
Public Function fieldGetter(ByVal field As String) As stdLambda
  'Note that bound data appears in 1st argument
  Set fieldGetter = stdLambda.Create("$2.item($1)").bind(field)
End Function
```

This is a fairly advanced technique, but leads to some very powerful functionality. You can find out more about currying [here](https://youtu.be/nuML9SmdbJ4).