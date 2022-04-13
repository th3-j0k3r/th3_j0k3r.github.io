---
title:  "SQL-injection"
mathjax: true
layout: post
categories: SQLi
---

Note: This was written back in my collegage days, hosting it now for personal/other's reference. 

According to [OWASP](https://owasp.org/www-community/attacks/SQL_Injection), SQL injection is an attack wherein the attacker can insert SQL query into the application via user input. A successful SQL injection can result in reading of sensitive data, modification of data in the database and so on.

# What is informaion_schema?
[Information_schema](https://dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html) is a database which comes by default with MYSQL installation that consists of different tables which holds the metadata information about all the databases and tables in the MYSQL server. Information_schema will come in handy while exploiting a SQLi. We will discuss how to use information_shema for exploiting SQLi later.

# Why SQL injection happens?

The core problem of SQL injection is that the developers just dynamically concatenate user input without any sanitization to the application. So there is no separation of code from data. Thus, the attacker injected SQL code will get executed by the application.

# What is the impact?
A SQL injection can have a high impact on the application ranging from reading, modifying or even deleting data in the database to remote code execution under some situations.

# Identification of the vulnerability
While testing for SQL injection, check every field in the application where the application asks for user input. Once we find all the user input fields, start enumerating and fuzzing those fields and see if the application throws some error, have different response. Or even check for the time taken to execute the command. It cannot be said that SQLi can be there in login forms, or where the application asks for user input. At times, application saves some cookie values or some headers. So injection might be possible in those fields too. Always understand the application flow and think out of the box for finding an injection points.

So how do we enumerate or fuzz the user input?

See what the application is expecting. Suppose if the application is expecting an integer input. Try supplying various numbers, negative number, special characters or even a string and see how the application behaves. 

Usually, developers enclose the user input inside a single quote or a double quote. While testing, supply these characters and see if there are any change occurs in the application.

Sometimes, the application will throw errors while passing single quote or double quote. The error occurs because there will be unbalance of single quotes or double quotes. In order to balance the query, we can add SQL comments after the single quote or double quote. So that the rest of the original SQL command will be commented out. Once the we have balanced the query, we will have a perfect injection point ie. between the single/ double quotes and before the comment. Here, We can inject our own [SQL commands](https://dev.mysql.com/doc/refman/8.0/en/comments.html) in order for further exploitation which is discussed next.  

```https://example.com/?id=1'<injection point>--+```

# Exploitation of the vulnerability
1. **Identification of the Database:** Once you have a valid injection point, first think to look for is to extract the database version. You can use  `@@version`  to retrieve the database name and it’s version.

2. **Extracting the database name:** After the database and it’s version is identified, the next step is to find the database that is used by the application. We can get that using the inbuilt function database(). 

3. **Extracting the table names:** Once the database name is found, the next step will be to extract each table names in that database. All the table names are stored inside the table information_schema.tables.  `SELECT table_name from information_schema.tables where table_schema="<database_name>";`

4. **Extracting the column names:** We can dump the column names similar way we did for finding table names.  `SELECT table_name from information_schema.tables where table_schema="<database_name>";`
Dumping the data: The final step is to dump the data from the database. 

This flow remains the same for most of the exploitation part of SQL injection.
# Types of SQL injection
## 	Error based SQL injection
Error based SQL injection happens because of improper error handling at the application level or the attacker intentionally make a SQL query to return a error and the attacker dumps the data in the database into the error. There are mainly two types of Error based SQL injection:
* Union based injection
* Double query based injection 


# Union based injection
A [Union](https://dev.mysql.com/doc/refman/8.0/en/union.html) operator is used to combine the results of multiple SELECT statements in a SQL query. The attacker make use of this functionality to add a additional SELECT statement which he controls along with making the first SQL query (which is the actual SQL query used by the application in the backend) to return false. Thus, the attacker execute his query and dump the data from the database.

Let’s see an example:
We will use of SQLi labs made by Audi-1 for demonstration.
[https://github.com/Audi-1/sqli-labs/tree/master/Less-1](https://github.com/Audi-1/sqli-labs/tree/master/Less-1 
) 

```$id=$_GET['id'];
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);
    if($row){
      echo 'Your Login name:'. $row['username'];
      echo "<br>";
      echo 'Your Password:' .$row['password'];
      }
    else{
    print_r(mysql_error());
    }
}
    else { echo "Please input the ID as parameter with numeric value";}
```

Here, we can see that there are no input sanitization so whatever user input is supplied to the application will be appended. So the attacker can know the database name by  the following payload:

`-1' union select 1,2,database()--+`


There are no records with the id ` -1 `. So the first SELECT query will become a false statement and returns nothing. The attacker will inject another SELECT statement in order to dump the data from the database. One thing to note here is that while using union statement the number of columns selected in first SELECT statement should be equal to the second SELECT statement.
### Finding the table_name
Now that we know the database name, next step is to find the table names. We can make use of the information_schema which we learned earlier for further exploitation. We can get the table name from the information_schema.tables using the payload given below:

`-1' union select 1,2,table_name from information_schema.tables where table_schema=database() limit 0,1--+`


# Finding the column_name  

After we dump all the table names, the next step is to find all the column names of each tables that we found. We can get the column names using the following payload:

`-1' union select 1,2,column_name from information_schema.columns where table_name='users' limit 0,1--+`


After this step we will have all the necessary information to dump the data from the database. The following SQL query can be used to dump all the data in the database.

`-1' union select 1,2,<column_name> from <table_name> limit 0,1--+`


# Exercise:
Write an automation script for dumping the database name, table names, column names for the same challenge with your language of choice.

# Double query injection

Before diving deep into Double query injection, let’s first understand what does double query is in SQL.

Technically, a double query or subquery is a query inside another query. So there will be a two queries, one is an  inner query and other is the outer query. The inner query will get executed first and it’s output will be sent to the outer query.  We will see an example for further clarity. 

`select * from employee_details where username in (select ename from department_details where designation='engineer');`


So here the select statement inside the brackets is the inner query or the sub query. The inner query get all the ename from the table according to the condition and outer query gets all the employee details matching those enames. Hope now you have a clear idea about how double query works. Now let’s get to the exploitation part. 

Double query injection is an error based injection where the attacker will dump the data from the database though errors. We will see how that is being done with the help of [Less-5](https://github.com/Audi-1/sqli-labs/tree/master/Less-5) of SQLi labs for explanation of these type of injection. 

Before going to exploitation, you need to be familiar with some SQL functions which will be needed while exploiting double query injection.

* [rand()](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_rand) returns a floating point number between 0 and <1.0.
* [floor()](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_floor) returns the nearest integer which is less than or equal to the number given into the function.  
* [concat()](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_concat) joins the given strings to the function.
* [count(*)](https://dev.mysql.com/doc/refman/8.0/en/counting-rows.html) counts the number of rows in a table.

Now that we have learned the functions that are needed, lets move to the exploitation. As discussed before, now we need to invoke error and dump the database through the error. So how will we generate an error along with the data that we need to dump?

Some great researchers have found a method to dump the data from the database through error, we will use their work for exploitation. 
 
`select count(*), concat((select version()),floor(rand(0)*2)) a from information_schema.tables group by a;`


So when we analyse the query, the error is invoked because

* [https://security.stackexchange.com/questions/89341/sql-injection-explain-this-query](https://security.stackexchange.com/questions/89341/sql-injection-explain-this-query) 
* [https://security.stackexchange.com/questions/92137/can-anybody-explain-error-based-sql-injection-attacks](https://security.stackexchange.com/questions/89341/sql-injection-explain-this-query)

# Blind SQL injection

A blind SQL injection is when the attacker input SQL query is being executed by the application but the response of it will not be displayed to the attacker. The attacker need to customize his payload accordingly to dump the data from the database. As there are no errors on being displayed in the response, the attacker need to find difference in time taken to respond or the difference in the content of the html response to exploit these type of SQL injection. 

# Boolean based
In boolean based injection, there will be a response when the attacker injection SQL code return TRUE and  a different response when the statement is FALSE. The 

Lets see an example. We will use [level-8](https://github.com/Audi-1/sqli-labs/tree/master/Less-8) from SQLi labs for explanation.

```
$id=$_GET['id'];
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);
    if($row){
      echo 'You are in...........';
      }
    else{   
    echo '<font size="5" color="#FFFF00">';
    echo "</br></font>";    
    echo '<font color= "#0000ff" font size= 3>';    
  }
}
    else { echo "Please input the ID as parameter with numeric value";}

```


So from the above example, it can be understood that if a user passes an id which exits in the database, the html response will contain ` You are in…` else the response will be empty. Now, let’s see how we can do SQL injection in this case:

The first step is to confirm if injection is possible or not. We already know that there is an id `1` that exits in the database. So if we add the following SQL code `1' and 1--+`  the application should return           ` Your are in ..` because both the operands are TRUE. And if we pass  `1' and 0--+` , the application returns nothing. So now we can confirm that injection can be possible here.

Next Step will be to find the Database name. But here comes the next challenge. We can’t just dump the database name the way we did for Error based injection. In blind SQLi, we can only check by character by character. So how is it done?

First we will find the length of the database name. We will use MYSQL function [length()](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_length) to get the database length.

`1' and length(database())=<n>--+`


If we breakdown the payload, what we have done is that we pass the database() to the length() function. Now the length function return the length of the current database. We don’t get to see the length of the database. So the only way is to enumerate until the length is equal to ` n `. When the correct database length is passed, we will receive the html response ` You are in …`. From this response, we can confirm that the length of the database.

Now that we know the database length, we can start dumping the database name. We need to do this the same way that we did for finding the database length but we need to use some other functions as we need to dump a character at a time.  

`1' and ascii(substr(database(),1,1))=<n>--+`


So here,  we have used [substr()](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_substr) and [ascii()](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_ascii) to determine the database name. substr(database(), 1,1) will return the first character of the database. It is then passed to ascii() to get the ascii value of the character that we got. Finally, it is checked  that the value returned by the ascii() is equal to ` n ` where n is an integer that represent the ascii number.

The method remains the same for finding the table names and column names. Doing blind injection manually can take a long time. So it is recommended to write a automation script while doing blind injection.

Exercise:

Find other methods that can be used instead of ascii().

# Time based 
In time based injection, once the injection is confirmed, if the attacker injected SQL code returns TRUE, it will take certain time to respond and if the statement returns FALSE it will take some more or less time to respond when compared to TRUE statement. By knowing the time difference, the attacker can extract the data from the database. 

Let’s use [level-9](https://github.com/Audi-1/sqli-labs/tree/master/Less-9) of SQLi labs for the explanation:

```
$id=$_GET['id'];
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);
    if($row){
      echo 'You are in...........';
      }
    else{
    echo 'You are in...........';  
    }
}
    else { echo "Please input the ID as parameter with numeric value";}
}
```


From the above code, it can be understood that we will have the same response when the id exists or not in the database. We can use [sleep()](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_sleep) function to confirm weather an injection is possible or not.

`1' and sleep(10)--+`


When we pass the above payload, the application will take 10 seconds to return the html response and when we pass some id that doesn’t exist in the db, the application will return the html response without any delay. From that we know that injection is possible because while using an and operator both the operands need to be TRUE, So the whole statement will be TRUE. Now let’s see how can we extract data from the database by only knowing the time difference.

Before moving to further exploitation let’s learn a new function called if(). In if() function, there are three expression, if the first expression is TRUE, then it returns the second expression and if the first expression if FALSE, it returns third expression.

Now, let’s see how we can create a payload to extract the database name in time based injection. In the first expression, we can have the ascii value checking that we have done before. If the that statement is TRUE, then we can call the sleep() function with the time that we specify. So that we know that it is a TRUE statement.  If the first expression returns FALSE, we will give the third expression as `0` which means FALSE. So the application will respond within no time. So the final payload will look like this: 

`1' and if(ascii(substr(database(),1,1))=<n>,sleep(10),0)--+`


Similarly, we can extract table names and column names then dump the data in the database. 

# Second order injection
 
Till now we have learned about the first order injection which means the attacker injected payload get executed immediately after the payload is sent to the application. So how does second order works?

In a second order injection, the attacker injected payload gets stored into the database after sanitization. So the developer assumes that this data will be safe to use in other parts of application as it went through the sanitization function. This payload is later used in the application inside a SQL statement where there are no sanitization of data. Now let’s see an example of this type of injection. We will use [Less-24](https://github.com/Audi-1/sqli-labs/tree/master/Less-24) of SQLi labs.

Upon visiting the challenge, we are presented with a login page. In the same page, we have option to register new account. Once we have signed up and logged in, there is a option to reset the password of the logged in user. This is the flow of the application. Now if we look the [code](https://github.com/Audi-1/sqli-labs/blob/master/Less-24/login_create.php) for the sign up page, it can be seen that the user name is passed to a sanitization function, [mysql_escape_string()](https://www.php.net/manual/en/function.mysql-escape-string.php). So the attacker payload will be properly escaped by the sanitization function. Now let’s take a look at the password reset code. We can see that the username is set from the session variable `$_SESSION["username"]` that has been set from the database, you can see the code [here](https://github.com/Audi-1/sqli-labs/blob/master/Less-24/login.php). The username is fetched from the database and concatenated with SQL update command without sanitization.

Let’s assume that there is an user called admin in the database. So the attacker will try to sign up with payload  `admin'#`. As there are proper sanitization on that endpoint, single quote will be escaped. So there will not be SQLi on that endpoint. But when the payload is stored into the database, it will be in the same form as it was passed by the attacker.

Now the attacker will login with the username `admin'#`  and resets the password, the user `admin` password will be changed to the password set by the attacker. This happens because the username is fetched from the database and passed to the SQL statement without sanitization. As there are no sanitization, the attacker payload will get executed and the attacker can takeover other user’s account.

`$sql = "UPDATE users SET PASSWORD='$pass' where username='$username' and password='$curr_pass' ";`



# Out of band injection

To:do


# Patching SQL injection

# Prepared statements with Parameterized Queries
Prepared statements is a feature used to execute same or similar database statements repeatedly with high efficiency. 

Now let’s see how prepared statements works. The whole process is divided into the following phases:
1. Parsing and Normalization phase
2. Compilation phase
3. Query Optimization Plan
4. Cache
5. Execution Phase

![](https://3.bp.blogspot.com/-Ru6lCV80PTE/Vfb44kaVfvI/AAAAAAAAAWI/6lLFVMsbe3E/s1600/QueryExecutionPhases.png)
Image taken from http://javabypatel.blogspot.com/2017/06/how-prepared-statement-works-internally-java.html 

# Parsing & Normalization Phase 

This is the initial phase, here checking for syntax, semantics and the existence of given table name and column names of the query are done.  

# Compilation phase
The SQL keywords are converted into code which database server understands is done in this phase.

# Query Optimization Plan
In this phase, a decision tree is created for number of ways the query can be executed. The decision tree finds all possible ways in which the query can be executed along with its cost. The SQL server engine chooses a best plan for executing a query from it.

# Cache
The best plan for executing the query will be now stored in the cache. So when the same query is comes in, the pre-compiled query in the cache is picked up and executed by the SQL server. 

# Execution Phase
Finally, the given query will be executed and the data is returned to the user.


Let’s see an example:


`SELECT username from users where id=?`


The above incomplete query with the placeholder will be parsed, complied, optimized and stored in the cache. When the user input is passed, the placeholders are replaced with the user input in the pre-compiled query and then executed.

So how does prepared statements prevent SQL injection?

The query with placeholders which can be called as the templated will be sent to the SQL server engine  and the query is compiled and stored in the cache. Now when the user input is passed, the pre-compiled query is picked from the cached and the placeholders are replaced with the user data. As the query is already pre-compile and saved in the cache, even when an attacker tries to inject SQL code, it will be treated as data. This is because there is no compilation after of the query after the user input is replaced with the place holders. So this is how the separation of code and data is done by using Prepared statements.

# Stored Procedure

Stored Procedure is similar to functions in programming languages. The stored procedure holds group of  SQL statements and are stored in the SQL server. The stored procedure can be invoked with the SQL [CALL](https://dev.mysql.com/doc/refman/8.0/en/call.html) keyword from the application.

Let’s see an example code to understand how Stored Procedure works:

```
 DELIMITER //
CREATE PROCEDURE country_hos
(IN con CHAR(20))
BEGIN
  SELECT Name, HeadOfState FROM Country
  WHERE Continent = con;
END //
DELIMITER ;
```

Code taken from https://dev.mysql.com/doc/connector-net/en/connector-net-tutorials-stored-procedures.html 

So the above is a code is an example how stored procedure are created at server side.

Stored procedures which is not implemented properly will still be vulnerable to SQL injection. 




# Whitelist input validation

In prepared statements values  like table name, column names or even SQL keywords cannot used to bound parameters. Only literals values in a SQL statements can be used in a prepared statements to bound parameters. So suppose if your application need to bound a SQL keyword from user input in a SQL statement, how can you make it safe from SQL injection in cases where escaping and parameterization will not work.

Here is a unsafe code where parameterization is not possible:  

```
<?php 
 $sortorder = $_GET["order"];
  $direction = $_GET["dir"];
 $sql = "SELECT * FROM Bugs   ORDER BY {$sortorder} {$direction}"; 
 $stmt = $pdo->query($sql);
?>
```

So how can make the code secure in this condition?

By whitelisting the user input we can make the code more secure here. So we need to have two array with whitelisted sortorders and directions and compare it with user supplied value. Thus, we can avoid SQL injection under this scenarios. 

```

$sortorders = array( "DEFAULT" => "bug_id",  "status" => "status",  "date" => "date_reported" ); 
$directions = array( "DEFAULT" => "ASC",  "up" => "ASC",  "down" => "DESC" );
 ```



# References and Credits:
* [https://github.com/Audi-1](https://github.com/Audi-1  - SQLi labs)
* [https://en.wikipedia.org/wiki/Prepared_statement](https://en.wikipedia.org/wiki/Prepared_statement) 
* [http://php.net/manual/en/mysqli.quickstart.prepared-statements.php](http://php.net/manual/en/mysqli.quickstart.prepared-statements.php)
* [http://javabypatel.blogspot.com/2017/06/how-prepared-statement-works-internally-java.html](http://javabypatel.blogspot.com/2017/06/how-prepared-statement-works-internally-java.html)
* [https://www.slideshare.net/billkarwin/sql-injection-myths-and-fallacies](https://www.slideshare.net/billkarwin/sql-injection-myths-and-fallacies)  
* [http://mysqlserverteam.com/re-factoring-some-internals-of-prepared-statements-in-5-7/](http://mysqlserverteam.com/re-factoring-some-internals-of-prepared-statements-in-5-7/) 
* [https://www.vividcortex.com/blog/2014/07/10/4-things-to-know-about-mysql-prepared-statements/](https://www.vividcortex.com/blog/2014/07/10/4-things-to-know-about-mysql-prepared-statements/) 
* [https://dev.mysql.com/doc/internals/en/com-stmt-prepare.html](https://dev.mysql.com/doc/internals/en/com-stmt-prepare.html) 
* [https://www.vividcortex.com/blog/2014/07/25/prepared-statement-samples/](https://www.vividcortex.com/blog/2014/07/25/prepared-statement-samples/) 
* [https://www.percona.com/blog/2006/08/02/mysql-prepared-statements/](https://www.percona.com/blog/2006/08/02/mysql-prepared-statements/)