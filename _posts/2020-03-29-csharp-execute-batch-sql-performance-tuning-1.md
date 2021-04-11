---
layout: post
title:  ".NET执行SQL性能优化一： 针对SQL Server批量执行SQL 语句"
date:   2020-03-29 22:32:55
categories: .NET
tags: C# SQL Batch
author: Michael
mathjax: true
---

* content
{:toc}

本文介绍了几种如何使用一个SqlCommand执行多条SQL语句的技术。
* SqlCommand.ExecuteNonQuery执行多条SQL
* 使用Data Reader执行两个SELECT语句
* 将SqlDataAdapter用于多个SELECT语句
* 执行匿名T-SQL块



## 介绍
使用ADO.NET对SQL Server进行数据存储经常被忽略的功能之一是它能够使用单个语句执行多个SQL语句```SqlCommand```。通常，程序分别执行语句和/或调用存储过程来执行更大的语句。当然，使用存储过程是一种首选方法，但是在某些情况下，一次调用执行多个语句是有益的。这可以使用批处理来完成，这基本上意味着一组SQL或T-SQL语句在一起。

## 设置
为了测试功能，让我们有一张数据库表。

创建测试表
```sql
CREATE TABLE MultiStatementTest (
id        int not null identity(1,1),
somevalue int not null
);
```
并用几行填充它。

添加几行
```sql
DECLARE @counter int = 1
BEGIN
WHILE (@counter <= 5) BEGIN
  INSERT INTO MultiStatementTest (somevalue) VALUES (RAND() * 1000);
  SET @counter = @counter + 1;
END;
END;
```
现在数据看起来像：

查询初始数据
```sql
SELECT * FROM MultiStatementTest;

id somevalue

1 854
2 73
3 732
4 546
5 267
```
## 测试程序
该测试程序易于使用。只需为创建测试表的数据库定义正确的连接字符串，即可开始运行测试。

## 执行多个SQL语句
第一个变体用于SqlCommand.ExecuteNonQuery对测试表执行两个单独的SQL语句。第一个将字段更新somevalue一个，第二个更新字段 。该方法如下所示：
```cs
/// <summary>
/// Executes two separate updates against the the connection
/// </summary>
/// <param name="connectionString">Connection string to use</param>
/// <param name="generateError">Should the statement generate an error</param>
/// <returns>True if succesful</returns>
public static bool ExecuteMultipleUpdates(string connectionString, bool generateError = false) {
 System.Data.SqlClient.SqlConnection connection = new System.Data.SqlClient.SqlConnection();
 System.Data.SqlClient.SqlCommand command = new System.Data.SqlClient.SqlCommand();
 int rowsAffected;

 connection.ConnectionString = connectionString;
 command.CommandText = @"
     UPDATE MultiStatementTest SET somevalue = somevalue + 1;
     UPDATE MultiStatementTest SET" + (generateError ? "WONTWORK" : "") + 
                   " somevalue = somevalue + 2;";
 command.CommandType = System.Data.CommandType.Text;
 command.Connection = connection;

 try {
    connection.Open();
    rowsAffected = command.ExecuteNonQuery();
 } catch (System.Exception exception) {
    System.Windows.MessageBox.Show(exception.Message, "Error occurred");
    return false;
 } finally {
    command.Dispose();
    connection.Dispose();
 }
 System.Windows.MessageBox.Show(string.Format("{0} rows updated", 
    rowsAffected, "Operation succesful"));

 return true;
}

```
因此，该CommandText属性包含将在此批处理中执行的所有语句。语句用分号分隔。

执行批处理后，行已更新两次，因此表的内容类似于：
```sql
id somevalue
1857
2 76
3735
4549
5270
```
需要注意的重要一件事是，返回的受影响的行数```ExecuteNonQuery```为10。表中有五行，每行更新了两次，因此更新的总数为10。因此，即使有批次，也可以检查无论哪个语句进行更新，都将更新正确的行数。

## 使用Data Reader执行两个SELECT语句
下一个测试是执行两个不同的SELECT语句，并使用一个SqlDataReader类读取结果。方法是：
```cs
/// <summary>
/// Executes two separate select statements against the the connection using data reader
/// </summary>
/// <param name="connectionString">Connection string to use</param>
/// <param name="generateError">Should the statement generate an error</param>
/// <returns>True if succesful</returns>
public static bool ExecuteReader(string connectionString, bool generateError = false) {
 System.Data.SqlClient.SqlConnection connection = new System.Data.SqlClient.SqlConnection();
 System.Data.SqlClient.SqlCommand command = new System.Data.SqlClient.SqlCommand();
 System.Data.SqlClient.SqlDataReader dataReader;
 System.Text.StringBuilder stringBuilder;
 bool loopResult = true;

 connection.ConnectionString = connectionString;
 command = new System.Data.SqlClient.SqlCommand();
 command.CommandText = @"
    SELECT somevalue FROM MultiStatementTest WHERE somevalue%2 = 1;
    SELECT somevalue FROM MultiStatementTest " + (generateError ? "WONTWORK" : "WHERE") + 
               " somevalue%2 = 0;";
 command.CommandType = System.Data.CommandType.Text;
 command.Connection = connection;

 try {
    connection.Open();
    dataReader = command.ExecuteReader();
    while (loopResult) {
       stringBuilder = new System.Text.StringBuilder();
       while (dataReader.Read()) {
          stringBuilder.AppendLine(dataReader.GetInt32(0).ToString());
       }
       System.Windows.MessageBox.Show(stringBuilder.ToString(), "Data from the result set");
       loopResult = dataReader.NextResult();
    }
 } catch (System.Exception exception) {
    System.Windows.MessageBox.Show(exception.Message, "Error occurred");
    return false;
 } finally {
    command.Dispose();
    connection.Dispose();
 }

 return true;
}

```
批处理中的想法是相同的，两个语句之间用分号分隔。在此示例中，将行分为两个结果集，具体取决于数字是奇数还是偶数。当```ExecuteReader```被调用时，第一个结果集是自动可用。该方法遍历各行并显示结果：

857
735
549
为了获得下一个结果，必须指示读者使用该```NextResult```方法前进到下一个结果集。此后，第二组值可以再次循环通过。第二组结果：

76
270

## 将SqlDataAdapter用于多个SELECT语句
SqlDataReader如果要将结果存储在中，通常使用```DataSet```。对于下一个测试，让我们使用SqlDataAdapter 来填充数据集。代码如下：
```cs
/// <summary>
/// Executes two separate select statements against the the connection
/// </summary>
/// <param name="connectionString">Connection string to use</param>
/// <param name="generateError">Should the statement generate an error</param>
/// <returns>True if succesful</returns>
public static bool ExecuteMultipleSelects(string connectionString, bool generateError = false) {
 System.Data.SqlClient.SqlConnection connection = new System.Data.SqlClient.SqlConnection();
 System.Data.SqlClient.SqlCommand command = new System.Data.SqlClient.SqlCommand();
 System.Data.SqlClient.SqlDataAdapter adapter = new System.Data.SqlClient.SqlDataAdapter();
 System.Data.DataSet dataset = new System.Data.DataSet();

 connection.ConnectionString = connectionString;
 command = new System.Data.SqlClient.SqlCommand();
 command.CommandText = @"
     SELECT * FROM MultiStatementTest WHERE somevalue%2 = 1;
     SELECT " + (generateError ? "WONTWORK" : "*") + 
       " FROM MultiStatementTest WHERE somevalue%2 = 0;";
 command.CommandType = System.Data.CommandType.Text;
 command.Connection = connection;

 try {
    connection.Open();
    adapter.SelectCommand = command;
    adapter.Fill(dataset);
 } catch (System.Exception exception) {
    System.Windows.MessageBox.Show(exception.Message, "Error occurred");
    return false;
 } finally {
    command.Dispose();
    connection.Dispose();
 }
 System.Windows.MessageBox.Show(string.Format(
    "Dataset contains {0} tables, {1} rows in table 1 and {2} rows in table 2", 
    dataset.Tables.Count, 
    dataset.Tables[0].Rows.Count, 
    dataset.Tables[1].Rows.Count, 
    "Operation succesful"));

 return true;
}

```
现在，以这种方式获取数据非常容易。该代码仅调用Fill适配器的方法，并将```DataSet```参数作为参数传递。适配器会自动```DataTable```在数据集中创建两个单独的对象，然后填充它们。在我的测试场景中，第一张表包含三行，第二张表包含两行。

由于在此示例中，表是即时创建的，因此会自动为其命名Table1，Table2因此，如果使用名称来引用表，则将名称更改为更具描述性的名称是明智的。

## 执行匿名T-SQL块
尽管存储过程非常出色，但是有时T-SQL代码在本质上可能非常动态。在这种情况下，可能很难创建存储过程。批处理还可以用于执行一堆T-SQL语句。在这种方法中，数据库中没有命名对象，但批处理的执行方式与本来的执行方式相同，例如，可以从SQL Server Management Studio中执行。

测试代码如下：
```cs
/// <summary>
/// Executes an anonymous T-SQL batch against the the connection
/// </summary>
/// <param name="connectionString">Connection string to use</param>
/// <param name="generateError">Should the statement generate an error</param>
/// <returns>True if succesful</returns>
public static bool ExecuteAnonymousTSql(string connectionString, bool generateError = false) {
 System.Data.SqlClient.SqlConnection connection = new System.Data.SqlClient.SqlConnection();
 System.Data.SqlClient.SqlCommand command = new System.Data.SqlClient.SqlCommand();
 int rowsAffected;

 connection.ConnectionString = connectionString;
 command.CommandText = @"
    DECLARE @counter int = 1
    BEGIN
     WHILE (@counter <= 5) BEGIN
     INSERT INTO MultiStatementTest (somevalue) VALUES (RAND() * 100000);
     SET @counter = @counter + 1;
     " + (generateError ? "WONTWORK" : "") + @"
     END;
    END;";
 command.CommandType = System.Data.CommandType.Text;
 command.Connection = connection;

 try {
    connection.Open();
    rowsAffected = command.ExecuteNonQuery();
 } catch (System.Exception exception) {
    System.Windows.MessageBox.Show(exception.Message, "Error occurred");
    return false;
 } finally {
    command.Dispose();
    connection.Dispose();
 }
 System.Windows.MessageBox.Show(string.Format("{0} rows inserted", 
    rowsAffected, 
    "Operation succesful"));

 return true;
}

```
现在，在此示例中，使用了与开始创建几个测试行相同的脚本。如您所见，变量声明，循环等是包含在批处理中的完全有效的语句。

运行该命令时，会将另外五行添加到表中。还要注意，由于NOCOUNT默认情况下处于关闭状态，因此该```ExecuteNonQuery```方法将返回正确数量的行插入到批处理中。如果NOCOUNT设置为on，则受影响的行数为-1。

## 错误处理
如果在批处理中执行单个语句时发生错误怎么办？我使用了一些语法错误来对此进行测试。该批处理将作为一个整体进行分析，因此即使以后的语句包含语法错误，该批处理也不会起作用。例如，如果UPDATE批处理中包含错误的 语句，则表的状态不会更改。

如果错误不是语法错误而是在执行过程中发生，则情况会有所不同。考虑例如当错误值出现时检测到的外键错误。在这种情况下，先前的语句可能已经更改了数据库状态（取决于语句），因此建议像往常一样使用适当的事务。

## 结论
虽然批处理的方式不会（也不应该）代替良好的旧存储过程等，但如果使用得当，它们将很有用。只要调用之间不需要客户端逻辑，它们就可以用于创建非常动态的操作，而无需进行多次往返。

本文翻译自文章： [Executing multiple SQL statements as one against SQL Server](https://www.codeproject.com/articles/306722/executing-multiple-sql-statements-as-one-against-s)