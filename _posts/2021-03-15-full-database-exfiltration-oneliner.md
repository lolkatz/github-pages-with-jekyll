Now that I have your attention I must warn you, this post is about SQL injection. It's been there for awhile and it's widely known how to protect against them. But even then it's still the top vulnerability in the [OWASP top ten](https://owasp.org/www-project-top-ten/) (even tough it been merged with other type of injection). You probably have seen this joke before (thanks to [xkcd](https://xkcd.com)):

![Bobby Tables cartoon](/will-hack-for-coffee/assets/images/little-bobby-tables.png)\
Figure 1: Little Bobby Tables

## My own tale

Well it happened to me. Once a senior developer came to my desk:\
Senior developer: You got SQL injection on your site.\
Me: No way...\
Senior developer: You got to fix it.\
Me: _Oh la boulette..._

Turn out a user asked for a way to search a list using a keyword. It was a small features and a update was quickly pushed. Except we forgot to use parametrized query for the request. When I was informed of this I tried to exploit SQL injection but I wasn't able to. Since I was kind of hard pressed to fix it, I just added parametrized query and forgot about it. In the next section I will show you a basic SQL injection and after I'll talk about a tool that could have demonstrated the vulnerability: SQLMap.

## SQL injection 101

So the classic example is a login form with the following SQL statement in the backend to verify if the user correctly typed the password:
````

SELECT * FROM users WHERE username = ' + _user_ + ' AND password = '  + _password_ + ';' 

````

If a user is selected, the application would allow login. But if instead of the user you submitted:

``' OR '1'='1' --``

The resulting request sent to the database would be:

````
SELECT * FROM users WHERE username = '' OR '1'='1' -- ' + _user_ + ' AND password = '  + _password_ + ';'_ 
````

Everything after the -- is considered a comment so will be ignored by the database and you will bypass login. What's even funnier is depending how it's handled by the backend you could be logged as admin since it's usually the first user added to the table. Hopefully login are not handled that way in modern website. However, the consequence of having any  SQL injection vulnerability on your website could be a disaster as I will show you.

## Some manual SQL injection

 The demo I'll use is from the Web Security Academy by [Portswigger](https://portswigger.net/web-security), the company that own the tool Burp Suite. The lab is called _SQL injection attack, listing the database contents on Oracle_  and the goal is to find the administrator password. Let's take a look at the site and refine the search by clicking on a category:

![A simple shopping site](/will-hack-for-coffee/assets/images/sqli-demo1.png)\
Figure 2: A simple shopping site

Notice the query string, that's where I'll try to inject by adding a string at the end of the url. On this site I can use UNION SQL injection and it will show me the results of my query. But there is other techniques like blind SQL injection that doesn't require you to see the output on the website, it's out of scope of this post but just know that it exist. First we need to determine the nuber of colums. So I will try to union a row with the same number of column. Why selecting a null? It's because null can be inserted in columns of any datatype:

``'+union+select+null+from+dual--``

Browser respond with Internal server error, that's interesting. If I try this:

``'+union+select+null,null+from+dual--``

I got no error but I can't see the row I've added, well it's was null value after all. I could select two constant string and see it output on the website. But instead I'll output the names of all the tables in the schema:

``'+union+select+'TableName',table_name+from+all_tables--``

![Shopping site displaying internal details about the database](/will-hack-for-coffee/assets/images/sqli-demo2.png)\
Figure 3: Shopping site displaying internal details about the database

I knew from the start it was an Oracle database management system so I knew which syntax to use but otherwise it's a bit of trials and errors. The following line would have output the DBMS for that particular website but it would have been different if it had been PostreSQL or else:

``'+union+select+'version',banner from v$version--``

Likewise you can query the column name of a particular table I found earlier:

``'+union+select+'column_name',column_name+from+all_tab_columns+where+table_name='EMPLOYEES'--``

But even using tools like Burp Suite it can be pretty tedious. So we could switch to the terminal and let SQLMap do all the heavy lifting.

## Bring out the big guns

Take note, this section will be entirely theoretical since the challenge are designed for Burp Suite and using SQLMap instead wouldn't be nice. So this is the only output you will see in this section:

````
└─$ sqlmap
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.5.2#stable}                                                           
|_ -| . [.]     | .'| . |                                                                          
|___|_  [)]_|_|_|__,|  _|                                                                          
      |_|V...       |_|   http://sqlmap.org                                                        

Usage: python3 sqlmap [options]

sqlmap: error: missing a mandatory option (-d, -u, -l, -m, -r, -g, -c, --list-tampers, --wizard, --update, --purge or --dependencies). Use -h for basic and -hh for advanced help

````
A word of caution, the following command could dump the content of a database. It could disclose sensitive data and you could be in serious problems if you do this without asking permission first. So here is the oneliner that I could have tried:

``sqlmap -u 'https://web-security-academy.net/filter?category=Lifestyle' --cookie="session=yourSessionCookieHere" --current-db --dump --technique=U`` 

So _u_ is the url and there also the option to set your cookie. Then there is _--current-db_, why? SQLMap can dump not only the schema that the website access but **all other schemas in the DBMS as well**. Next flag is _--dump_ and it will create a folder named _Output_ filled with all the data of the table it found, conveniently stored in csv format. The last flag is --technique and I specified _U_ for Union. By default, SQLMap will run all techniques but some like blind SQL injection can take a long time.

Before I launch that command I might begin by grabbing the banner (that is the information the service share about the software it use) with _-b_. Since SQLMap store the information it get about a target, it will remember that it's an Oracle DBMS.

The documentation on the command line is a little bit parse but the tool is open source and there is excellent documentation on their github site:
[SQLMap github documentation](https://github.com/sqlmapproject/sqlmap/wiki/Features). By default SQLMap will use "safe" settings but you can specify riskier setting that could result in update to the database and that wouldn't be good.

## Conclusion

 I've only touched a fraction of what SQLMap can do. It can do dictionnary attack on password stored in the database and can even allow to take over the server where the database is hosted. Now that you know the consequence of SQL injection vulnerability I hope you'll help getting rid of them for good. Until next time keep hacking but be nice to the DBAs out there.