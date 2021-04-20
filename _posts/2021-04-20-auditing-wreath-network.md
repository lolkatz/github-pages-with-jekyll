---
title: "Auditing Wreath network"
date: 2021-04-20
---

A new room has opened on Tryhackme to learn about network penetration testing: [Wreath network](https://tryhackme.com/room/wreath). It's a free room but you need to have a 7-day streak, meaning  solving a question every day for seven days, to access it. The writeup for this room need to be in a different format than usual. You needed to write a penetration testing report as you would have to do if it was a real job. It was the first time for me so I had to learn more in details the classification of vulnerabilities and I had to put more toughts into potential remediation. My writeup got accepted so you can read it here: [Penetration test report for Wreath network](https://lolkatz.github.io/will-hack-for-coffee/writeups/penetrationTestReportWreathNetwork.pdf). In the rest of this post I'll just highlights some of the things I found interesting.

![I hacked Wreath network](/will-hack-for-coffee/assets/images/hackedWreathNetworkMeme.png)

The scenario is that you have been tasked by one of your old classmate, Thomas Wreath, to audit his personal network consisting of a public facing webserver, a git server and his personal computer. It's an introduction to network penetration testing where you can learn and practice multiple pivoting techniques, among them Socat (a beefed up version of netcat) relay, a Chisel forward socks proxy and sshuttle a very useful encrypted tunnel using ssh that act has a kind of VPN. You will also have to use Foxyproxy which is a must have extension for Firefox if you ever used Burp Suite. Finally, you will also get some experiences dealing with firewall and antivirus system.

I've done the Throwback network before but I'll highly recommend doing Wreath network first, as it is an introduction to network penetration testing. When I was navigating [Throwback network](https://tryhackme.com/room/throwback) I only knew about proxychains and I even had to fall back to chaining remote desktop to reach the final machine. Wreath network made me more comfortable pivoting and teach me some useful techniques like Pass the hash with EvilWinRM or stealing the private key to have an SSH access. I also practiced shell stabilisation a technique that I recently learned reading a [Watcher room writeup](https://erichogue.ca/2021/03/TryHackMe-Walkthrough-Watcher/). It's also explained in this excellent room [What the shell](https://erichogue.ca/2021/03/introtoshells/) (but it's for subscriber only). Thinking back there is a couple of times I could have used that technique to spare me some frustration.

I also learned several new ways of sharing data between the attacking machine and the target. [EvilWinRM](https://github.com/Hackplayers/evil-winrm) has easy command to upload and download files but you can also specify a local directory on your machine so you will have access to your tools and script without copying files to the target. And if you really want to live off the land, why not use a temporary SMB server? That's what you will do for the exfiltration part. There was also an interesting moment where you downloaded the git repository on a private server and analyzed the source code of the website to find a point of entry (a file upload vulnerability in ths case).

There was a couple of tools that helped me find vulnerabilities. There is searchsploit, very useful to quickly find CVE from the command line. There is also [WinPEAS (Windows Privilege Escalation Awesome Scripts)](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS), if you ran it in windows it will highlight all the misconfigurations that will allow you to elevate your privilege. And I also learned a new tricks with nmap:
````
sudo nmap -sV -script vuln <IP>
````
At first I omitted the -sV flag and it only gives me one vulnerability but adding the flag it gave me almost forty. There is bound to have false positive or CVE that cannot be exploited but it gives you options.

There was a section of the room that was about Empire exploitation framework, it didn't work for me. From what I've heard there was a bug linked to the pivoting so maybe it will be fixed when you try it or you could downgrade you version. It's entirely optional so don't worry about it.

The Wreath network was quite an experience for me and I can't wait for the next Tryhackme network. Until then have fun and hack responsibly!