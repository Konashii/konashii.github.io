
---
layout: ../../layouts/BlogLayout.astro
title: 'agentsudo'
pubDate: 2022-07-01
description: 'This is for the agentsudo CTF on TryHackMe'

tags: ['TryHackMe', 'CTF', 'Easy']
---

First I perform an Nmap scan to see any open ports that may be vulnerable with `nmap -sC -sV -oN nmap {IP}`

Upon accessing the website, there is a message in plain HTML:
```
Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent R 
```

What caught my attention was the wording of the phrase. Using a "codename" as a "user-agent"? Sounds like a header tag when you send a GET request. 

Since *codenames* for agents would comically be a single letter, and the fact that the message was left by "Agent R", I think I should just change my `User-Agent` header to all the 26 letters to get somewhere.

Already, the letter "C" came up with something interesting:
```
Attention chris,

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!

From,
Agent R 
```

Okay, so now I know one of the agent's name is Chris. And he has a weak password? I bet that means it'll be super easy to brute force his credentials.

I remember they had an FTP server so I would try to break into that with `chris` as a username. I run the command `hydra -l chris -P SecLists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt ftp://$IP`

We got a match! The password is `crystal`.

So logging into the FTP server, I check what's in it with `ls` and I see a couple interesting files.

These images might have some interesting text inside it, considering one of the questions to answer is asking about a *steg password*.

The `To_agentJ.txt` file contains the following text:
```
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

So since this seems to be the part where you extract a file or text from a file, I used the tool `steghide` on both `cutie.png` and `cute-alien.jpg`

*I eventually learned that steghide does not even support PNG formats...so cute-alien.jpg is the only file of interest*.

There seems to be a password, but I'm not really sure where I can find this password. I've checked their web application for anything, nothing. So I decided to brute force it using `stegcracker` with the famous rockyou.txt wordlist. 

The password is `Area51`.

The image extracted a file `message.txt`and it contains:
```
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

It's now been discovered that Agent J's real name is James. And he has a login password of `hackerrules!`. This would appear to be the SSH password.

---

### Privilege Escalation

So already logging in as `james`, we can grab the `user.txt`, and the the next question is asking about what incident was the photo taken place, which was `Roswell Alien Autopsy`, as titled by Fox news.

So usually one of the first things I already do is figure out if `james` can run sudo, which you can do `sudo -l` to find out.

The part that's more interesting to me is the `(ALL, !root) /bin/bash`. I have not seen this before so I decided to look it up to find out what it means. Already the first search result is a CVE for it, and how to exploit it.

From my understand of how [CVE-2019-14287](https://www.exploit-db.com/exploits/47502) works:

If the attacker runs `sudo -l` and sees that it outputs: `(ALL, !root) /bin/bash` (which basically means that the attacker *cannot* run `/bin/bash` as root), while also checking the permissions in the `/etc/sudoers` file, technically the attacker can run `/bin/bash` as ANY user as `sudo` does not check for the existence of the specified user ID and executes with the arbitrary user ID with the sudo privileges.

```
sudo -u#-1 /bin/bash
```

The `-u#-1` returns as "0" which is the root's ID. And this will successfully execute with root permission.

---

### Closing

Now after running the exploit, we have now obtained root (AKA having complete control of the system)!