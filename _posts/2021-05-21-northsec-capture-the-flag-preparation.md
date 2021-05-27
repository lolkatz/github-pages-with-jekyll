I participated in Northsec 2021 CTF. It was super fun and the build up to it kept us pretty busy. I want to tell you about what the weeks were like up to the compettion. First I'll talk about some of features and contents contained in this year badge. Then I'll walk you through CTF warmup on the Discord that culminated in a mock hack on a public IP. After that, I'll highlights some cool things I learned in the excellent CTF 101 workshop that I attended during the conference part of the NorthSec event. Finally, I'll shortly describe my setup to connect to the IPv6 infra and I'll show you where we find our first flag, before the event even started.

## First the badge

The badge was totally optional since the event was remote but there is game inside so I could not resist. Plus it's good looking and has LEDs:

![Northsec 2021 badge](/will-hack-for-coffee/assets/images/nsec2021-badge.png)

So the badge has a screen as well as some buttons to act as a controller. There also two buttons at the back, one acting as a reset. No battery included but there is a connector at the back if you want to carry it around. This badge also feature WIFI and speaker. The game has a retro RPG look, it seems Ubisoft was somewhat involved in that game and it's no surprise because the game looks good. I explored the small map, heard some villagers jokes, looted a chest and visited some building.  One of the buildings allows you to play some songs, while another let you tinker with the LEDs. At the bottom of the screen you can see that I managed to grab 3 of the 10 flags. You can also connect yourself through a USB CLI interface using that command:
````
sudo screen /dev/ttyUSB0 115200
````
In there you will be able to connect to your router and then through the WIFI you will be able to download binaries for a reverse engineering challenges. You could submit flags to the Discord bot and it will give you various vanity role in the forum. I may take another look at it later, after all you can never have too many flags.

## Time to warmup

In the NorthSec Discord there was some warmup for the CTF which made us hunt all over the places for some more flags.

![CTF warmup questions](/will-hack-for-coffee/assets/images/warmup-flag.png)

The first few are pretty easy involving browsing in the [NorthSec website](https://nsec.io/) and looking at the clues given in Discord (like the _American psycho_ classic scene shown at the bottom of the picture). Question number 6 involved runic alphabet and the caracters would be recognized by Google when copied into. Question 7 involved looking at the code representing the various emoticons, I used that site:
[Full emoji List](https://unicode.org/emoji/charts/full-emoji-list.html)

And with that I was done with the warmup... or was I? Soon after, there was a video about bagel and mate posted on Discord containing a flag:

![More warmup](/will-hack-for-coffee/assets/images/more-warmup.png)

## And now I beg to see you hack just one more time

And then a few days before the CTF began, those who had done all the warmup received a link to a video. It featured the videoclip of the song _Dance Monkey_ from _Tones and I_, which is quite catchy and funny. Frome times to times you'll see some of the Northsec crew dancing along the music. But there is also subtitles that are added and they are slightly different then the lyrics.

![Hack one more time](/will-hack-for-coffee/assets/images/hack-one-more-time.png) 

So here is some of the lyrics:

````
They say oh my god I see the way you scan
Take n-map, my dear, and trace with sT param
You know to stop right after port 4999
And now I beg to see you hack just one more time

Ooh I see you, see you, see you in Solarwinds
And oh my god, what's that, is that a flag?
````
There is also those lyrics that are kinda intriguing:
````
Hack for me, hack for me, hack for me, oh, oh
I've never seen anybody find the flags you did before
They say, 
move for me, move for me, move la-te-r-a-a-lly
And when you're done I'll make you pop a shell again
````
The lyrics also mentions Wannacry, APT and malware, all that good hacker stuff. And there is a word uttered throughout the videoclip: Success. If you pay attention you'll also see a number appears in the bottom righ corner:

![Hidden IP](/will-hack-for-coffee/assets/images/hidden-ip.png).

There is also another text at the end that says: The numbers are an ipv4 address. So I used that -sT parameter with nmap and I found nothing, apparently it was a joke and by doing so the firewall will detect the scan. So I then used a more classic scan techniques, stopping right after port 4999 like they told me:

````
sudo nmap -sS -p-5000 174.138.62.119                                                        
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-25 15:28 EDT
Nmap scan report for 174.138.62.119
Host is up (0.0070s latency).
Not shown: 4999 filtered ports
PORT     STATE SERVICE
4999/tcp open  hfcs-manager

Nmap done: 1 IP address (1 host up) scanned in 14.84 seconds
````

I wasn't entirely surprised they were talking about popping a shell, so I tried to connect with telnet:

````
telnet 174.138.62.119 4999
Trying 174.138.62.119...
Connected to 174.138.62.119.
Escape character is '^]'.
Ooh, ooh, what's the name of your malware?
Wannacry
I see you see you everytime! Take a 10 seconds break!
Connection closed by foreign host.
````

I'm not a big buff on malware but I googled Solarwinds attack and stumbled on that site: [SolarWinds attack explained](https://www.csoonline.com/article/3601508/solarwinds-supply-chain-attack-explained-why-organizations-were-not-prepared.html). Apparently the customised backdoor used in the SolarWinds attack was codenamed SUNBURST. So I used that instead:

````
And oh my god, what's that, is that a flag? 
Send lyrics from the song to FLAGBOT to get a monkey vanity role!
$ help
Available commands: ip, telnet, exit
$ ip
External IP: 174.138.62.119
Internal IP: 10.10.10.3
Gateway: 10.10.10.1
$ telnet 10.10.10.2
I've never seen anybody find this flag before: WARMUP-BONUS-d0763edaa9d9bd2a9516280e9044d885
(THIS IS A WARM UP BONUS FLAG ONLY - NOT FOR THE OFFICIAL CTF)
Connection closed by foreign host.
````
They where talking about moving lateraly so that's why I tried to connect to a machine between mine and the gateway. Hurry up if you want to try it yourself, even if the competition is over that host is till up!

## Warmup some more with CTF 101

I also attended [CTF 101 workshop](https://www.youtube.com/watch?v=wh7v_W27fbg) which aired during the conference parth of the NorthSec conference, it was hosted by Olivier Bilodeau who is a very funny guy. It was well structured, very interesting and I was glad I attended.

![CTF 101](/will-hack-for-coffee/assets/images/capture-the-flag-101.png)

### The basics
It began by using a private key to connect to a machine then shows how to use an SSH tunnel to do lateral movement in a network. Essential stuff and I liked how they inboarded us so quickly. With that out of the way we moved to basic html, SQL injection and XSS, again classic stuff. But they turn it up a notch for XSS and set us up with a server so we can effectively steal cookies from an user. Until now I had only tried reflected and stored xss on myself which is ok but not too fancy, so I liked that. Then we moved to forensics and some useful tools for CTF: strings and Wireshark. But then again they went up another notch and we were shown python scripting to reassemble flags and we got to script with tshark too, the terminal equivalent of wireshark.

### Reverse engineering
Next section was reversing engineering. I don't know much about reverse engineering and I consider it more advanced stuff but it's something you need to learn if you want to progress in CTF and hacking. First of all, he advised us to never run unknown binary on our computer. That being said he also told us that this binary was not malicious and since I trust them I gave it a try:

````
└─$ ./crackme
Performing intense computation...
FLAG-zsh: segmentation fault  ./crackme
````
So it crash just before printing the content of the flag. We ran Ghidra (a tool developed by the NSA!) for static analysis of the crackme file:

![Hydra](/will-hack-for-coffee/assets/images/hydra-crackme.png)

Then we switched to gdb for dynamic analysis. What I like is that he shown us how to set up GDB in a much more useful ways using the [pwngdb extension](https://github.com/pwndbg/pwndbg). It allows you to see the register, the stack and the assembly code all at once. Since we suspect by examining the source code earlier with Ghidra that the raise call was making the program crash we can set the index pointer register to the next adress, skipping that line entirely.

![pawngdb](/will-hack-for-coffee/assets/images/pawngdb.png)

Well that was interesting!

### Binary exploit

Again binary exploit is more advanced stuff, useful to know for CTF, but also essential to understand if you want to obtain the OSCP certification. This time we have the source code so there is one more feature that I can show you with pawngdb but you need to add **-g** to the suggested command to compile the source code:
````
gcc -m32 -I./ -fno-stack-protector service.c -g -o service
````
This will allow to see source code in GDB:

![Pawngdb with source code](/will-hack-for-coffee/assets/images/gdb-with-source-code.png)

I also suggest that you modify the source code to remove the fork call so you won't have to constantly kill and restart service:

![Modified service source code](/will-hack-for-coffee/assets/images/modified-service-source-code.png)

I did not complete the challenge by myself but using the same string as the instructor did I managed to grab the flag. Usually buffer overflow exploit involve rewriting the Index Pointer register so that it point to a part of memory you overflowed with shellcode so you can pop a shell with the privilege associated to the binary. That was not the case for that challenge and I'll have look at it again to understand it.

The best introduction to binary exploit I saw yet is on [Hack that box academy](https://academy.hackthebox.eu/course/preview/stack-based-buffer-overflows-on-linux-x86), it's free so take a look if you want to learn more.

## Setting up and grabbing our first flag

The NorthSec infra for the CTF is entirely IPv6. The VPN gave us access to a 9000::/16 subnets, I can't tell you how many IPs it represents but it's a pretty big network (hint: I don't know how to count up to that number). We obtained a certificate and a OpenVPN configuration file by mail with [instructions](https://nsec.io/vpn/). Some of us got our file attachment blocked and others found Northsec mail in the spam folder but nothing that couldn't be overcome. I use Firefox for browser so there was nothing special apart from installing the certificate but there was some additionnal difficulties for Chrome user. The steps to install the VPN were given for a Ubuntu machine but it was fairly straighforward for Kali:

![Kali OpenVPN configuration](/will-hack-for-coffee/assets/images/kali-openvpn-config.png)

I did had to search a little for the Routes options tough because I didn't had internet access on my machine at first:

![Kali OpenVPN configuration - routes](/will-hack-for-coffee/assets/images/kali-openvpn-config2.png)

With that done, we were let loose on the network, like beasts looking for blood. And there it was, in the wild, our first official flag!

![Kali OpenVPN configuration - routes](/will-hack-for-coffee/assets/images/first-blood.png)

The NorthSec did an awesome job, they really did build up the hype before the CTF event and I liked every minutes of it. You can see some of the stuff my team did in this other post: [Northsec Wizard Academy](https://lolkatz.github.io/will-hack-for-coffee/2021/05/24/northsec-wizard-academy.html). Until then have fun and hack responsibly!