---
title: "UnitedCTF 2021 part 1: Desjardins track (and more)"
date: 2021-10-09
---

[UnitedCTF](https://www.unitedctf.ca/) is a Quebec university two week hacking competition that is open to all. It was the first time I participated and I was joined by a couple of my friends from Hackfest. I found it very beginner friendly, there was huge number of references and hints to help with the CTF and I managed to learn a couple of very interesting techniques. 

## Crypto as a welcoming gift from United CTF

One of the first challenge puzzled a lot of players myself included. The challenge was the second text in one of the Discord channels:

![Discord flag](/will-hack-for-coffee/assets/images/unitedctf2021/discord-flag.png)

I struggled at first, was it base64, a binary, some hex dump? I tried to concatenate the four strings but to no avail. I came back to it a week later and I thought maybe it's some hash? A [wise man](https://www.mindkind.org/index.php) told me later that I could have used the hash-identifier tool in Linux, which is conveniently installed in kali. At that time tough I simply navigated to [Crack station](https://crackstation.net/) and got the flag:

![Crack station working it magic](/will-hack-for-coffee/assets/images/unitedctf2021/crack-station-magic.png)

Very reliable site for CTF but I would not recommend using it for sensitive hash tough (that you got from work or home network), since once they are on the internet your hash won't provide much protection to your password (as you can see).

## Join the fun

All the docker file to run those challenges are available on [United CTF git page](https://github.com/UnitedCTF/UnitedCTF-2021) so I was able to revisit them and solved some more using the writeups provided by the authors. Here is what I did to run one of my favorite challenge on my windows machine:
- Install [docker](https://www.docker.com/get-started): There is a nice little tutorial you can do once you installed it that helped me getting started.
- Clone the repository: Git was already installed on my machine so I just ran this line in the PowerShell window:
````
git clone https://github.com/UnitedCTF/UnitedCTF-2021.git
````
- Build:
````
 docker build -t plusbassoumissionnaire .
````
- Run: I used port 80 for this one but you might use other ports depending on the challenge (80:80 means your machine port and the docker port).
````
docker run -dp 80:80 plusbassoumissionnaire
````
- You can then access the challenge using localhost:80 or for my Kali VM I used the IP:port of the machine I was running docker from.

Depending on the challenge, like with redos, I used that command and it configured the port for me:
````
docker-compose up
````

Since you can do the challenge even after it's over, you should try them and get back here if you want to know how I tackled them. The authors writeups are also included in the git repository.

## Desjardins Windows Forensics track

[Desjardins](https://github.com/UnitedCTF/UnitedCTF-2021/blob/main/challenges/desjardins/Looking%20for%20interns-Recherchons%20des%20stagiaires.md) wants to recruit young talented individuals so they made a very interesting track about Windows Forensics for the CTF. You had to download an OVF image (a VMWare format) which hosted a Windows 10 machine containing four flags. There were forensics tools already installed so it made it a breeeze. They also linked a Youtube channel by [13Cubed](https://www.youtube.com/playlist?list=PLlv3b9B16ZadqDQH0lTRO4kqn2P1g9Mve) on Windows Forensics that taught you all you needed to know to solve the challenges. I've checked the links on the GitHub pages for the image and it seems to be still working but I'd hurry if you want to give this challenge a try.

### Windows Forensic 1: RDP cache

So the user of that machine connected via[Remote Desktop on another machine and looked at a flag. Windows keep screenshots of what the user saw through [remote desktop as cache](https://www.youtube.com/watch?v=NnEOk5-Dstw), they are stored in this folder:
````
C:\Users\user\AppData\Local\Microsoft\Terminal Server Client\Cache
```` 
Using the python scripts already installed on the machine you can extract those pictures (-s source -d destination):
````
./bmc-tools/bmc-tools.py -s bcache24.bmc -d ./bcache24.bmc
````
Not very friendly to search through but here is the flag:

![RDP cache flag](/will-hack-for-coffee/assets/images/unitedctf2021/rdp-cache-flag.png)

### Windows Forensic 2: Deleted folder

So the user cleaned up his desktop and emptied his bin to make sure nobody could find what he did. Turns out you can see the folders that were deleted using [ShellBags Explorer](https://www.youtube.com/watch?v=YvVemshnpKQ), which was also conveniently installed on the desktop. You only had to select File -> Load active registry and there you have it:

![Shellbag explorer](/will-hack-for-coffee/assets/images/unitedctf2021/shellbag-explorer.png)

### Windows Forensic 3: Deleted file

And now what was the name of the file inside of this folder? Turns out Windows is also storing that info somewhere as shortcuts or [LNK file](https://www.youtube.com/watch?v=YvVemshnpKQ):

![lnk](/will-hack-for-coffee/assets/images/unitedctf2021/lnk.png)

### Windows Forensic 4: And now when was it deleted?

The last flag was the the date that file was deleted. It was very straightforward to do since the NTFS journal was already extracted on the machine and the tools to read it was also already installed: ANJP. But just look at the video called [NTFS Journal Forensics](https://www.youtube.com/watch?v=1mwiShxREm8) if you want to see more in depth. So here is the info we are looking for:

![ntfs-journal](/will-hack-for-coffee/assets/images/unitedctf2021/ntfs-journal.png)

### FAT32: Did someone stick an USB in this machine?

The last challenge on the Desjardins track involved a malicious actor that used an USB stick to copy files on a machine. You need to find that file and all you are told is that the USB key is formatted in FAT32 (commonly used by manufacturer since it work on most system). But first you need to install the filesystem, just download it from GitHub and double cllick it in Windows. Inside the Desktop folder there was huge folder with many files each containing what could be a flag but no way (at first) to see which file is the correct one.  I did not solve it but I read the author writeup and I found it interesting. The way to solve it is that the NTFS filesystem (common in windows) store the time the file was modified with a nanosecond precision, while the FAT32 has a 2 second precision. So you list the files with the time of modification and the FAT32 files will stand out. I wasn't able to show that information using PowerShell but the author showed how to do this using a python script.

### More forensics: LibreOffice

There was a LibreOffice file which was describing a real life vulnerability affecting particular DrayTek device. We were told flags were hidden in the file, I found two of them. The description of the challenge gave a couple of clues of how to find hidden text in Libre Office files like making font invisible and such. By changing the color of the background I found a first flag (the document is several page long so I'm just showing the first part of the flag):

![libre office hidden fonts](/will-hack-for-coffee/assets/images/unitedctf2021/libre-office-hidden-fonts.png)

For the second flag a wise man told from Hackfest told me that odt (and docx) file format are basically zip file. I had forgotten about this so I unzipped the files and browsing through the picture folders I found this:

![libre office out of place picture](/will-hack-for-coffee/assets/images/unitedctf2021/libre-office-007.png)

## To conclude

I found the Desjardins track unique. It was my first experience of Windows forensics and it deepened my understandings of how Windows works behind the scenes. I hope they participate again next year and that they manage to recruit a couples of skilled hackers! 

Next up: SQL injection track


