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

### To conclude

I found the Desjardins track unique. It was my first experience of Windows forensics and it deepened my understandings of how Windows works behind the scenes. I hope they participate again next year and that they manage to recruit a couples of skilled hackers! 

[Back to main article](2021-10-09-unitedctf2021.md)





