---
title: "Use character format to import & export data"
description: Character format uses character data format for all columns. This is useful working with other programs or copying to an instance from another database vendor.
author: rwestMSFT
ms.author: randolphwest
ms.date: "09/29/2016"
ms.service: sql
ms.subservice: data-movement
ms.topic: conceptual
ms.custom: seo-lt-2019
helpviewer_keywords:
  - "data formats [SQL Server], character"
  - "character formats [SQL Server]"
monikerRange: ">=aps-pdw-2016||=azuresqldb-current||=azure-sqldw-latest||>=sql-server-2016||>=sql-server-linux-2017||=azuresqldb-mi-current"
---
# Use character format to import or export data (SQL Server)
[!INCLUDE[SQL Server Azure SQL Database Synapse Analytics PDW ](../../includes/applies-to-version/sql-asdb-asdbmi-asa-pdw.md)]
Character format is recommended when you bulk export data to a text file that is to be used in another program or when you bulk import data from a text file that is generated by another program.  

Character format uses the character data format for all columns. Storing information in character format is useful when the data is used with another program, such as a spreadsheet, or when the data needs to be copied into an instance of [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] from another database vendor such as Oracle.  
  
> [!NOTE]
>  When you bulk transfer data between instances of [!INCLUDE[msCoName](../../includes/msconame-md.md)] [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] and the data file contains Unicode character data but not any extended or DBCS characters, use the Unicode character format. For more information, see [Use Unicode Character Format to Import or Export Data &#40;SQL Server&#41;](../../relational-databases/import-export/use-unicode-character-format-to-import-or-export-data-sql-server.md).
  
|In this Topic:|
|---|
|[Considerations for Using Character Format](#considerations)|
|[Command Options for Character Format](#command_options)|
|[Example Test Conditions](#etc)<br />&emsp;&#9679;&emsp;[Sample Table](#sample_table)<br />&emsp;&#9679;&emsp;[Sample Non-XML Format File](#nonxml_format_file)<br />|
|[Examples](#examples)<br />&emsp;&#9679;&emsp;[Using bcp and Character Format to Export Data](#bcp_char_export)<br />&emsp;&#9679;&emsp;[Using bcp and Character Format to Import Data without a Format File](#bcp_char_import)<br />&emsp;&#9679;&emsp;[Using bcp and Character Format to Import Data with a Non-XML Format File](#bcp_char_import_fmt)<br />&emsp;&#9679;&emsp;[Using BULK INSERT and Character Format without a Format File](#bulk_char)<br />&emsp;&#9679;&emsp;[Using BULK INSERT and Character Format with a Non-XML Format File](#bulk_char_fmt)<br />&emsp;&#9679;&emsp;[Using OPENROWSET and Character Format with a Non-XML Format File](#openrowset_char_fmt)|
|[Related Tasks](#RelatedTasks)<p>                                                                                                                                                                                                                  </p>|


  
## Considerations for Using Character Format<a name="considerations"></a>
When using character format, consider the following:  
  
-   By default, the [bcp utility](../../tools/bcp-utility.md) separates the character-data fields with the tab character and terminates the records with the newline character.  For information about how to specify alternative terminators, see [Specify Field and Row Terminators &#40;SQL Server&#41;](../../relational-databases/import-export/specify-field-and-row-terminators-sql-server.md).  
  
-   By default, before the bulk export or import of character-mode data, the following conversions are performed:  
  
    |Direction of bulk operation|Conversion|  
    |---------------------------------|----------------|  
    |Export|Converts data to character representation. If explicitly requested, the data is converted to the requested code page for character columns. If no code page is specified, the character data is converted by using the OEM code page of the client computer.|  
    |Import|Converts character data to native representation, when necessary, and translates the character data from the client's code page to the code page of the target column(s).|  
  
-   To prevent loss of extended characters during conversion, either use Unicode character format or specify a code page.  
  
-   Any [sql_variant](../../t-sql/data-types/sql-variant-transact-sql.md) data that is stored in a character-format file is stored without metadata. Each data value is converted to [char](../../t-sql/data-types/char-and-varchar-transact-sql.md) format, according to the rules of implicit data conversion. When imported into a [sql_variant](../../t-sql/data-types/sql-variant-transact-sql.md) column, the data is imported as [char](../../t-sql/data-types/char-and-varchar-transact-sql.md). When imported into a column with a data type other than [sql_variant](../../t-sql/data-types/sql-variant-transact-sql.md), the data is converted from [char](../../t-sql/data-types/char-and-varchar-transact-sql.md) by using implicit conversion. For more information about data conversion, see [Data Type Conversion &#40;Database Engine&#41;](../../t-sql/data-types/data-type-conversion-database-engine.md).  
  
-   The [bcp utility](../../tools/bcp-utility.md) exports [money](../../t-sql/data-types/money-and-smallmoney-transact-sql.md) values as character-format data files with four digits after the decimal point and without any digit-grouping symbols such as comma separators. For example, a [money](../../t-sql/data-types/money-and-smallmoney-transact-sql.md) column that contains the value 1,234,567.123456 is bulk exported to a data file as the character string 1234567.1235.  
  
## Command Options for Character Format<a name="command_options"></a>  
You can import character format data into a table using [bcp](../../tools/bcp-utility.md), [BULK INSERT](../../t-sql/statements/bulk-insert-transact-sql.md) or [INSERT ... SELECT * FROM OPENROWSET(BULK...)](../../t-sql/functions/openrowset-transact-sql.md). For a [bcp](../../tools/bcp-utility.md) command or [BULK INSERT](../../t-sql/statements/bulk-insert-transact-sql.md) statement, you can specify the data format in the statement.  For an [INSERT ... SELECT * FROM OPENROWSET(BULK...)](../../t-sql/functions/openrowset-transact-sql.md) statement, you must specify the data format in a format file.  
  
Character format is supported by the following command options:  
  
|Command|Option|Description|  
|-------------|------------|-----------------|  
|bcp|**-c**|Causes the bcp utility to use character data.\*|  
|BULK INSERT|DATAFILETYPE **='char'**|Use character format when bulk importing data.|  
|OPENROWSET|N/A|Must use a format file|
  
 \*To load character (**-c**) data to a format compatible with earlier versions of [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] clients, use the **-V** switch. For more information, see [Import Native and Character Format Data from Earlier Versions of SQL Server](../../relational-databases/import-export/import-native-and-character-format-data-from-earlier-versions-of-sql-server.md).  
   
> [!NOTE]
>  Alternatively, you can specify formatting on a per-field basis in a format file. For more information, see [Format Files for Importing or Exporting Data &#40;SQL Server&#41;](../../relational-databases/import-export/format-files-for-importing-or-exporting-data-sql-server.md).

## Example Test Conditions<a name="etc"></a>  
The examples in this topic are based on the table, and format file defined below.

### **Sample Table**<a name="sample_table"></a>
The script below creates a test database, a table named `myChar` and populates the table with some initial values.  Execute the following Transact-SQL in Microsoft [!INCLUDE[ssManStudioFull](../../includes/ssmanstudiofull-md.md)] (SSMS):

```sql
CREATE DATABASE TestDatabase;
GO

USE TestDatabase;
CREATE TABLE dbo.myChar ( 
   PersonID smallint NOT NULL,
   FirstName varchar(25) NOT NULL,
   LastName varchar(30) NOT NULL,
   BirthDate date,
   AnnualSalary money
   );

-- Populate table
INSERT TestDatabase.dbo.myChar
VALUES 
(1, 'Anthony', 'Grosse', '1980-02-23', 65000.00),
(2, 'Alica', 'Fatnowna', '1963-11-14', 45000.00),
(3, 'Stella', 'Rossenhain', '1992-03-02', 120000.00);

-- Review Data
SELECT * FROM TestDatabase.dbo.myChar;
```

### **Sample Non-XML Format File**<a name="nonxml_format_file"></a>
SQL Server support two types of format file: non-XML format and XML format.  The non-XML format is the original format that is supported by earlier versions of SQL Server.  Please review [Non-XML Format Files (SQL Server)](../../relational-databases/import-export/non-xml-format-files-sql-server.md) for detailed information.  The following command will use the [bcp utility](../../tools/bcp-utility.md) to generate a non-xml format file, `myChar.fmt`, based on the schema of `myChar`.  To use a [bcp](../../tools/bcp-utility.md) command to create a format file, specify the **format** argument and use **nul** instead of a data-file path.  The format option also requires the **-f** option.  In addition, for this example, the qualifier **c** is used to specify character data, and **T** is used to specify a trusted connection using integrated security.  At a command prompt, enter the following command:

```cmd
bcp TestDatabase.dbo.myChar format nul -f D:\BCP\myChar.fmt -T -c 

REM Review file
Notepad D:\BCP\myChar.fmt
```

> [!IMPORTANT]
> Ensure your non-XML format file ends with a carriage return\line feed.  Otherwise you will likely receive the following error message:
> 
> `SQLState = S1000, NativeError = 0`  
> `Error = [Microsoft][ODBC Driver 13 for SQL Server]I/O error while reading BCP format file`

## Examples<a name="examples"></a>
The examples below use the database, and format files created above.

### **Using bcp and Character Format to Export Data**<a name="bcp_char_export"></a>
**-c** switch and **OUT** command.  Note: the data file created in this example will be used in all subsequent examples.  At a command prompt, enter the following command:

```cmd
bcp TestDatabase.dbo.myChar OUT D:\BCP\myChar.bcp -T -c

REM Review results
NOTEPAD D:\BCP\myChar.bcp
```

### **Using bcp and Character Format to Import Data without a Format File**<a name="bcp_char_import"></a>
**-c** switch and **IN** command.  At a command prompt, enter the following command:

```cmd
REM Truncate table (for testing)
SQLCMD -Q "TRUNCATE TABLE TestDatabase.dbo.myChar;"

REM Import data
bcp TestDatabase.dbo.myChar IN D:\BCP\myChar.bcp -T -c

REM Review results
SQLCMD -Q "SELECT * FROM TestDatabase.dbo.myChar;"
```

### **Using bcp and Character Format to Import Data with a Non-XML Format File**<a name="bcp_char_import_fmt"></a>
**-c** and **-f** switches and **IN** command.  At a command prompt, enter the following command:

```cmd
REM Truncate table (for testing)
SQLCMD -Q "TRUNCATE TABLE TestDatabase.dbo.myChar;"

REM Import data
bcp TestDatabase.dbo.myChar IN D:\BCP\myChar.bcp -f D:\BCP\myChar.fmt -T

REM Review results
SQLCMD -Q "SELECT * FROM TestDatabase.dbo.myChar;"
```

### **Using BULK INSERT and Character Format without a Format File**<a name="bulk_char"></a>
**DATAFILETYPE** argument.  Execute the following Transact-SQL in Microsoft [!INCLUDE[ssManStudioFull](../../includes/ssmanstudiofull-md.md)] (SSMS):

```sql
TRUNCATE TABLE TestDatabase.dbo.myChar; -- for testing
BULK INSERT TestDatabase.dbo.myChar
	FROM 'D:\BCP\myChar.bcp'
	WITH (
		DATAFILETYPE = 'Char'
		);

-- review results
SELECT * FROM TestDatabase.dbo.myChar;
```

### **Using BULK INSERT and Character Format with a Non-XML Format File**<a name="bulk_char_fmt"></a>
**FORMATFILE** argument.  Execute the following Transact-SQL in Microsoft [!INCLUDE[ssManStudioFull](../../includes/ssmanstudiofull-md.md)] (SSMS):

```sql
TRUNCATE TABLE TestDatabase.dbo.myChar; -- for testing
BULK INSERT TestDatabase.dbo.myChar
   FROM 'D:\BCP\myChar.bcp'
   WITH (
		FORMATFILE = 'D:\BCP\myChar.fmt'
		);

-- review results
SELECT * FROM TestDatabase.dbo.myChar;
```

### **Using OPENROWSET and Character Format with a Non-XML Format File**<a name="openrowset_char_fmt"></a>
**FORMATFILE** argument.  Execute the following Transact-SQL in Microsoft [!INCLUDE[ssManStudioFull](../../includes/ssmanstudiofull-md.md)] (SSMS):

```sql
TRUNCATE TABLE TestDatabase.dbo.myChar;  -- for testing
INSERT INTO TestDatabase.dbo.myChar
	SELECT *
	FROM OPENROWSET (
		BULK 'D:\BCP\myChar.bcp', 
		FORMATFILE = 'D:\BCP\myChar.fmt'  
		) AS t1;

-- review results
SELECT * FROM TestDatabase.dbo.myChar;
```
  
## Related Tasks<a name="RelatedTasks"></a>  
To use data formats for bulk import or bulk export 
  
-   [Import Native and Character Format Data from Earlier Versions of SQL Server](../../relational-databases/import-export/import-native-and-character-format-data-from-earlier-versions-of-sql-server.md)  
  
-   [Use Native Format to Import or Export Data &#40;SQL Server&#41;](../../relational-databases/import-export/use-native-format-to-import-or-export-data-sql-server.md)  
  
-   [Use Unicode Character Format to Import or Export Data &#40;SQL Server&#41;](../../relational-databases/import-export/use-unicode-character-format-to-import-or-export-data-sql-server.md)  
  
-   [Use Unicode Native Format to Import or Export Data &#40;SQL Server&#41;](../../relational-databases/import-export/use-unicode-native-format-to-import-or-export-data-sql-server.md)  
  
## See Also  
 [bcp Utility](../../tools/bcp-utility.md)   
 [BULK INSERT &#40;Transact-SQL&#41;](../../t-sql/statements/bulk-insert-transact-sql.md)   
 [OPENROWSET &#40;Transact-SQL&#41;](../../t-sql/functions/openrowset-transact-sql.md)   
 [Data Types &#40;Transact-SQL&#41;](../../t-sql/data-types/data-types-transact-sql.md)   
 [Import Native and Character Format Data from Earlier Versions of SQL Server](../../relational-databases/import-export/import-native-and-character-format-data-from-earlier-versions-of-sql-server.md)  
  
  
