---
layout: post
title: "UnitedCTF 2021 part 2: SQL injection"
date: 2021-10-09
---

I really liked this track, there was nine (relatively) easy challenges on SQL injection. There was a thorough explanation on how to do the first challenge and it allowed everyone to catch up and brush up their skills. Since I've already talked about [SQL injection in a previous blog](/will-hack-for-coffee/_posts/2021-03-15-full-database-exfiltration-oneliner.md) I'll skip right to the chase.

## SQLi 1: What's SQLi?

Each challenge involved a basic login page. Using Burp Suite helped me adjust my payload quickly and offered basic documentation.

![SQLi login](/will-hack-for-coffee/assets/images/unitedctf2021/sqli-login.png)

For the username field I used admin (as with all others SQLi challenges). For the password field I used this payload to get the first flag:
````
password" or 1=1 --
````
Note: I retried the injection on docker and I had to replace the " by ', so you may have to adjust your payload depending on your system. Which is weird since I tought Docker would make it system agnostic, oh well.

## SQLi 2: Only one

The way the challenges were designed you didn't have reflected injection in the page, if you had the correct payload you were forwarded to another page that had the flag. Brute forcing was not allowed but tools like sqlmap probably would not have worked anyway. We were told that the code validated that it returned only one result but that the admin still was the first in the table. So I used this payload instead:
````
password" or id=1 --
````

## SQLi 3: Maybe Burp can find it?

For the next challenge the admin was not in the first row so I used Burp intruder tab and specified id as my payload variable:

![burp-intruder](/will-hack-for-coffee/assets/images/unitedctf2021/burp-intruder.png)

Turned out his id was 2... xD

## SQLi 4: Union attack

There was then a series of challenge based on some variation that needed to use [union attack](https://portswigger.net/web-security/sql-injection/union-attacks). In this challenge the flag is in the flag table in the flag column. First, I found the number of column returned by the table using different column number, this payload didn't provoke errors:

````
" order by 3 --
````
Then I crafted this highly elaborated payload:
````
" UNION SELECT "","",flag from flag -- 
````

## SQLi 5: No union

This challenge is identical to the previous one except the key word UNION is filtered. This one made me smile because I already had solved something similar evading XSS filter. My payload from the previous challenge would be reconstructed after the filter would be applied:
````
" UNUNIONION SELECT "","",flag from flag -- 
```` 

## SQLi 6: What table?

Similar challenge except there is no filter and the table name is unknown. It was mentioned in the first challenge of the series that the database was SQLite3. That is relevant here because we will have to query the table name using the database schema which differs depending on the database. After a little bit of research I crafted this payload to find the name of the table:
````
" UNION SELECT "","",name from sqlite_master where sql like "%flag%" --
````
![table-name](/will-hack-for-coffee/assets/images/unitedctf2021/table-name.png)

With this info I found the flag like so:

````
 " UNION SELECT "","",flag from K953U0Ty4V --
````

## SQLi 7: Where the flag?

For this challenge neither the table or column name is known. So first:

````
" UNION SELECT "","",sql from sqlite_master --
...
CREATE TABLE "Ja93MjakS" (
            "id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
            "Dab93JkA" VARCHAR)
...
````
Resulting in this payload:
````
" UNION SELECT "","",Dab93JkA from Ja93MjakS --
````

## SQLi 8: No flag

So there is a secret table with the column flag but there is a filter on the word flag. First, I needed to find the table name:
````
" UNION SELECT "","",sql from sqlite_master where sql like "%fla%" --
...
CREATE TABLE "cKretTblName" (
            "id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
            "flag" VARCHAR
          )
...
````
So after a couple of trials and errors I found that payload. My union attack returns three columns and that table has only two columns so I returned an empty string instead.
````
" UNION SELECT "",* from cKretTblName --
````
 Would it really work in real life? I don't know but I got my flag.

 ## SQLi 9: No spaces for you

 The good news is that the flag is in the flag table within the flag column. As it was not ridiculous enough the DBA now filters all spaces. I've tried a lot of things for this one but turns out you can use comments instead of spaces:
 ````
"union/*comments*/select/*_*/"","",flag/*_*/from/*_*/flag/*more_comments*/--
 ````
So that was it for the SQL injection track. Next up ReDoS!

