This year [Northsec CTF](https://youtu.be/wJpInBMBSkg?t=2485) was medieval themed and took place in the land of North Sectoria. In this kingdom, hackers are called magicians. Do you have what it takes to become a wizard?

![Map of North Sectoria](/will-hack-for-coffee/assets/images/north-sectoria-map.png)

I had the opprotunity to play on the CTF with a team called Mary Poppins Shell. This year CTF featured a beginner track called Wizard academy and we finished most of the challenges in this track. The challenges included basic HTML, local file inclusion (LFI), file upload, SQL injection, server side request forgery (SSRF), open redirect and deserialization. 

## Know your basic html

The first set of challenge was called Automatization and was a pun based on the robots text file.

### Automatization 101
The first challenge was quite simple. Looking at the source code of the page, we got our first flag:

````
<!DOCTYPE html>
<html>
    <head>
        <title>Wizard Hackademy</title>
    </head>
    <body>
        <!-- FLAG-c8dd29f9a87f32792f72de7cbe9f9e8c (1/2) -->
        <div class="container">
            Spell is under concoction...
        </div>
    </body>
</html>
````

### Automatization 102
Looking in robots.txt we find a reference to a "secret file":

````
User-agent: *
Disallow: secret-b42677bf3f974f7.txt
````

Browsing to this page we found the next flag.

## Mentalism

This set of challenges involved Local File Inclusion (LFI). The url had a query string: ?page=hackademy.php

![mentalism.png](/will-hack-for-coffee/assets/images/mentalism101.png)

## LFI 101

The first challenge was solved by a  by modifying the query string as such:  
?page=/etc/passwd  

This revealed the linux password file as well as the flag. He also solved the [LFI 103](https://erichogue.ca/2021/05/NorthSec2021WriteupMentalism/#mentalism-103).

## Spell recipe upload

The next set of challenges were classic validation bypass for file upload. 

### File upload 101 

We used a php reverse shell available in kali machine:
/usr/share/webshells/php/php-reverse-shell.php

We then renamed the file to php-reverse-shell.jpg.php to bypass the image extension validation. Uploading this file got us the flag.

### File upload 102

For this challenge, we kept the previous file and uploaded it again. We intercepted the POST request with Burp and send it to the Repeater tab.

We then modified the content type of the request like so to obtain the flag:


![Modifying content type with Burp](/will-hack-for-coffee/assets/images/file-upload102.png)

### File upload 103

For the third challenge, using the same previous technique was not enough. So we decided to add a magic number to the file. Looking at the list of file signature ([List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)):

![JPG file signature](/will-hack-for-coffee/assets/images/jpg-file-signature.png)

With a text editor we added a line with four characters ('AAAA') at the beginning of the file. Then using hexedit we replaced those caracters with the jpg file signature:

![hexedit](/will-hack-for-coffee/assets/images/hexedit.png)

Uploading this file got us another flag.

## What do you mean Spell Query Langage?

This set of challenges involved SQL injection. I tried the most basic injection: "lol ' or '1'='1' -- " but it didn't work. Thinking back I should have tried [SQLmap](https://lolkatz.github.io/will-hack-for-coffee/2021/03/15/full-database-exfiltration-oneliner.html), oh well maybe next time. 

### SQL 101

My teammates found the first flag using UNION based SQL injection:

![Union based SQL injection](/will-hack-for-coffee/assets/images/sql-injection101.png)

[They](https://erichogue.ca/2021/05/NorthSec2021WriteupSpellQueryLanguage/#flag-1) also found the second one, kudos to them!

## Super Spell Requestum Forgerum

Wait a minute, is that Server Side Request Forgery? I was always tought that was pretty advanced stuff so I was thrilled to look at this.

### SSRF 101

The API allows to pass HTTP request along method GET and POST, as well as parameters. But can we request file?

![SSRF 101](/will-hack-for-coffee/assets/images/ssrf-passwd.png)

After this I was wondering which other files I could look up, someone suggested to look for flag obviously, and there it was at the the root of the directory:

``
file:///flag.txt
``

Getting that flag earns you the ultimate wizard certification: the WHRCE (which could be the medieval equivalent of CEH).

## Open redirect

My teammate also solved that challenge, it's really well explained so I'll let you check it out:
[Open Redirect Writeup](https://erichogue.ca/2021/05/NorthSec2021WriteupOpenRedirect/)

## Spellrialize

That's another challenge that was solved by my teammate. I've never done deserialisation so I wish I had time to try it myself but at least you can read about it: [Spellrialize](https://erichogue.ca/2021/05/NorthSec2021WriteupSpellrialize/)

## Bonus points: Hackers trivia

In the CTF, there was also trivia questions about the 1995 movie Hackers starring the lovely Angelina Jolie. The first seven questions could be obtained from (closely) watching the movie but the last two required external resources.

What’s the best brand of toothbrush to wash your teeth after your morning Cereal? 

We will have to ask Twitter about this one:

![Cereal Killer on Twitter](/will-hack-for-coffee/assets/images/ceralKillerToothbrush.png)

Also known as “cyberspirits”, how are these magnificient subway rollerbladers also called?

The folks at HackersCurator will be pleased to answer that one:

[Hackers director interview](https://youtu.be/ZpMVgNNZDPw?t=1431)

I hope you had fun at NorthSec, until next year: Hack the planet!




