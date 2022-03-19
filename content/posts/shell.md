---
title: "Shell"
date: 2022-03-19T01:09:19Z
draft: false
---

So it took three days, but `shell` is finally running. What the hell have I been going with the time?
Nothing much. I've created a Digital Ocean droplet three times, run some basic commands on the machine
and have logged off, only to discover later that I'd misconfigured something. Through a small list of
commands, I somehow managed to screw up some very basic settings. Most of this seemed to impact around
`sudo`, `visudo` and once, running `pfctl` for a firewall that I didn't really need.

These are the commands I've been running:

```
pkg update
pkg install sudo
adduser
visudo
vim /etc/ssh/sshd_config
```

Nothing much there, is there? It's too bad that someone who is supposed to know what they're doing can make
such of a meal of something as simple as trying to ban an SSH user from using the service, create a second
account, add that account to wheel and then configure wheel with password permission to allow the user to
access `root` level options if needed.

Now that we're done with this little adventure, I have a machine configured that I can at least use.

Usually at this point, I'd dive into making other modifications to the system to try and improve things.
Let's however dig into some of the things that I want to investigate before we start making any unncessary
changes.

```
$ nmap --script ssh2-enum-algos shell.pufferfish.cc
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-18 23:09 GMT
Nmap scan report for shell.pufferfish.cc (138.68.184.126)
Host is up (0.0093s latency).
Not shown: 994 closed tcp ports (conn-refused)
PORT     STATE    SERVICE
22/tcp   open     ssh
| ssh2-enum-algos:
|   kex_algorithms: (10)
|       curve25519-sha256
|       curve25519-sha256@libssh.org
|       ecdh-sha2-nistp256
|       ecdh-sha2-nistp384
|       ecdh-sha2-nistp521
|       diffie-hellman-group-exchange-sha256
|       diffie-hellman-group16-sha512
|       diffie-hellman-group18-sha512
|       diffie-hellman-group14-sha256
|       diffie-hellman-group14-sha1
|   server_host_key_algorithms: (5)
|       rsa-sha2-512
|       rsa-sha2-256
|       ssh-rsa
|       ecdsa-sha2-nistp256
|       ssh-ed25519
|   encryption_algorithms: (9)
|       chacha20-poly1305@openssh.com
|       aes128-ctr
|       aes192-ctr
|       aes256-ctr
|       aes128-gcm@openssh.com
|       aes256-gcm@openssh.com
|       aes128-cbc
|       aes192-cbc
|       aes256-cbc
|   mac_algorithms: (10)
|       umac-64-etm@openssh.com
|       umac-128-etm@openssh.com
|       hmac-sha2-256-etm@openssh.com
|       hmac-sha2-512-etm@openssh.com
|       hmac-sha1-etm@openssh.com
|       umac-64@openssh.com
|       umac-128@openssh.com
|       hmac-sha2-256
|       hmac-sha2-512
|       hmac-sha1
|   compression_algorithms: (2)
|       none
|_      zlib@openssh.com
25/tcp   filtered smtp
80/tcp   open     http
443/tcp  open     https
5060/tcp open     sip
8080/tcp open     http-proxy
```

When compared to [Mozilla's InfoSec page on SSH security][1], there are a few holes that we can fix via
`/etc/ssh/sshd_config`:

```
$ nmap --script ssh2-enum-algos shell.pufferfish.cc
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-18 23:44 GMT
Nmap scan report for shell.pufferfish.cc (138.68.184.126)
Host is up (0.0076s latency).
Not shown: 994 closed tcp ports (conn-refused)
PORT     STATE    SERVICE
22/tcp   open     ssh
| ssh2-enum-algos:
|   kex_algorithms: (5)
|       curve25519-sha256@libssh.org
|       ecdh-sha2-nistp521
|       ecdh-sha2-nistp384
|       ecdh-sha2-nistp256
|       diffie-hellman-group-exchange-sha256
|   server_host_key_algorithms: (5)
|       rsa-sha2-512
|       rsa-sha2-256
|       ssh-rsa
|       ecdsa-sha2-nistp256
|       ssh-ed25519
|   encryption_algorithms: (6)
|       chacha20-poly1305@openssh.com
|       aes256-gcm@openssh.com
|       aes128-gcm@openssh.com
|       aes256-ctr
|       aes192-ctr
|       aes128-ctr
|   mac_algorithms: (6)
|       hmac-sha2-512-etm@openssh.com
|       hmac-sha2-256-etm@openssh.com
|       umac-128-etm@openssh.com
|       hmac-sha2-512
|       hmac-sha2-256
|       umac-128@openssh.com
|   compression_algorithms: (2)
|       none
|_      zlib@openssh.com
25/tcp   filtered smtp
80/tcp   open     http
443/tcp  open     https
5060/tcp open     sip
8080/tcp open     http-proxy

Nmap done: 1 IP address (1 host up) scanned in 80.07 seconds
```

That's just plugging the following into `/etc/ssh/sshd_config`:

```
KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
```

I also made some small modifications to the `/etc/ssh/sshd_config` file:

```
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

LoginGraceTime 2m
StrictModes yes
MaxAuthTries 6
MaxSessions 10

PermitRootLogin No
AllowUsers <redacted>
```

`visudo` contains another modification:

```
%wheel ALL=(ALL) ALL
```

This is all very nice and good. SSH access is limited to me alone. I have a pretty secure OpenSSH server. `sudo` is configured
to do the right thing insofar as asking for a password prompt is concerned.

Because it's fun, let's increase our security config a little. [yubikey-agent][2] exists to help us use a Yubikey device to
secure logins against unauthorized access. To get this rolling, run the following:

```
brew install yubikey-agent
brew services start yubikey-agent
yubikey-agent -setup
```

and copy your new SSH key to your remote machine. What we have now is a system that *should* lock anyone out who can't provide
a 2FA code from our Yubikey device. Does it really matter though? I have to say no, no it doesn't matter. By default, a key
configured with `0400` on your local hard drive is going to be pretty safe. The keys on your FreeBSD are going to be pretty safe,
as they're public. Yubikey and [yubikey-agent][2] exist to allow you the benefit of touch-based authentication as well as well as
the ability to use the Yubikey to log on between devices.

Next objectives for this little shell server of mine are going to be setting up Docker in a jail and working on some source code to
implement a couple of API's on that Docker instance. Experimentation should hopefully happen over the weekend.


[1]: https://infosec.mozilla.org/guidelines/openssh
[2]: https://github.com/FiloSottile/yubikey-agent