---
layout: post
title: "UnitedCTF 2021 part 4: QR and cats"
date: 2021-10-09
---

Crypto and steganography are definitely not my specialty but I checked them out to see if there was some low hanging fruits. I checked the checked who were not worth a lot of points but I also looked at the number of peoples who had solved the challenge. Here are some that I've done:

## Encodage

The challenge explained the difference between encoding and encryption. They gave you a string to decode and told you how important it was to recognize popular encoding scheme like base64, rot13 and hexadecimal. They instruct you to use [Cyberchef](https://gchq.github.io/CyberChef/) which is a great website. I discovered the magic function which gives you suggestion of transformation that could be interesting. Here's my solution:

![decoding with cyberchef](/will-hack-for-coffee/assets/images/unitedctf2021/cyberchef-encodage.png)

## Spectacles

So you have a weird audio file. After some googling I found out that you can hide informatio in a audio file via the spectrogram view. I used the Audacity software but turns out the name of the challenge was a reference to the spek tool. Well that reference was lost on me.

![hidden in audio file](/will-hack-for-coffee/assets/images/unitedctf2021/audacity.png)

## Lost and found 1

Some (unethical) hacker corrupted a bunch of cat pictures and you need to restore them. So you have a png file but it cannot be opened. There is a link to [some documentation about png](http://www.libpng.org/pub/png/spec/1.2/PNG-Structure.html). In one of the first paragraphs they talk about file signature, so the first eight bytes (also called magic bytes) must contain a signature which is indicated in decimal. For my file signature need I always go to [wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures) and as bonus the signature are in hexadecimal which is more convenient for the next step. So I went to this site that allows modification of file: https://hexed.it/. I loaded my file and modified the headers like so:

![Hexed it](/will-hack-for-coffee/assets/images/unitedctf2021/lost-found-hexed-it.png)

Opening the file again you can see the flag:

![Cat 1](/will-hack-for-coffee/assets/images/unitedctf2021/chat1.png)

That technique is useful in other case, like if a file upload validation restricts by MIME type you can try to bypass it in a similar manner. By the way, do you think that kitten look evil? No wonder the hacker tried to destroy those images.

## Lost and found 2

Again I looked at the file with an hexadecimal editor. Looking at the end of the files, I see the magic bytes but it looked reversed... 

![Reversed magic bytes](/will-hack-for-coffee/assets/images/unitedctf2021/lost-found-reversed-hex.png)

So I uploaded the picture into Cyberchef and used the reverse function. You can then redownload the picture, here you go another demonic kitten!

![Cat 2](/will-hack-for-coffee/assets/images/unitedctf2021/chat2.png)

Disturbed by those pictures of weird kittens I went on to another challenge.

## rqbp-ed

So the title of the challenge didn't helped me much but the description said: "Can you find the image within the text file?". I heard a lot about QR code this year, most of them related to vaccinal passport. I looked at the text file and recognized one. So this was one of the challenge involving QR code! 

![Text QR code](/will-hack-for-coffee/assets/images/unitedctf2021/text-qr-code.png)

I did a little bit of googling and found some code to transform text into a black and white image, here is my python script:

````python
import numpy as np
from PIL import Image

# before that step I replaced the '0' and '1' by '0 ' and '1 ' so they were delimited by spaces
matrix = np.loadtxt('file2.txt', usecols=range(200))

# it's 0 and 1 but I want higher contrast
matrix = np.where(matrix == 1, 255, matrix)

# I put the same value in RGB channel so it will be black and white
img = np.zeros([200,200,3],dtype=np.uint8)
img[...,0] = matrix
img[...,1] = matrix
img[...,2] = matrix

im = Image.fromarray(img) 
im.save('codeQR.png')
````

![Text QR code](/will-hack-for-coffee/assets/images/unitedctf2021/qr-code.png)

So I had a QR code but when I scanned it using this [site](https://webqr.com/index.html) I was puzzled by the answer:

0199-133-102 1+ rz yynP

I tought my QR was corrupted and since the CTF had just ended, I let it go. But someone told me on the Discord that the string needed to be ROT13 and reverse, at first I discarded that as way too convoluted but I should have been paying more attention to the title:

![rqbp-ed](/will-hack-for-coffee/assets/images/unitedctf2021/rqbp-ed.png)

And then you had to call that number and it was a voice message with the flag, doh! I got confused by the title because I was focused on the rq part which is QR backward but the rest of the title didn't make sense, turns out I was headed in the wrong direction.

## Won't be next: Reverse engineering

I won't do writeups for reverse engineering but I've done Bootme and Exports, which were the two easiest. If you never done reverse engineering, you'll have to install and use a decompiler for the first challenge, which you will need to familiarize yourself with anyway if you want to improve in CTF. For the second challenge you'll need to use a tool to emulate processor which allows to run application compiled for a different architecture.

## Next up: Web challenge

I plan to do another post on the web tracks which features delicious cookies, file uploads, local file inclusion, remote command execution, insecure object reference as well as epic log forging. I will also include the Vaxinull challenge which I tought fit well with another healthcare-themed challenge: Plus bas soumissionnaire. Hope you liked what you saw and until have fun hacking with dockers!
