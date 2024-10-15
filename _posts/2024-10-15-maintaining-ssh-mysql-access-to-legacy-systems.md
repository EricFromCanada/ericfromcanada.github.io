---
title: Maintaining SSH and MySQL access to legacy systems
---

My line of work has some legacy system installations that I need to maintain access to, which abruptly became a problem when I finally migrated to an ARM-based Mac running Sonoma as my daily driver.

## SSH client

Anyone running macOS Ventura or later may have noticed that its `ssh` client can no longer get into servers running CentOS 6 or earlier with e.g. "No matching host key type found. Their offer: ssh-rsa,ssh-dss", or that it asks for a password when a public key used to work. This is because OpenSSH 8.8 disabled the use of RSA signatures using the SHA-1 hash algorithm by default (Ventura shipped with OpenSSH 9.0p1, and Sonoma with OpenSSH 9.7p1).

Until those servers are replaced or upgraded, you can force your local SSH client to selectively enable older algorithms for certain servers by including this flag when connecting by including this in your `ssh` invocation:

    -o HostKeyAlgorithms=+ssh-rsa

This will let you connect, but it'll always ask a password. To also force it to offer your older RSA key (which is also being deprecated — you DO have an [ED25519 public key created](https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54), right?), add this:

    -o PubkeyAcceptedKeyTypes=+ssh-rsa

If the server is REALLY old (think CentOS 5), you'll also need to add this:

    -o KexAlgorithms=+diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1

(These are all internal-only systems with no public internet presence, right? Oh good; just checking.)

## MySQL client

The [last nightly build of Sequel Pro](https://sequelpro.com/test-builds) still seems to run in Sonoma under Rosetta 2, but I wanted to see if [Sequel Ace](https://sequel-ace.com/) was up to the task. I quickly found out that the current version does not support MySQL Server versions earlier than 5.6, which were dropped as part of their [adding support for MySQL 8](https://github.com/Sequel-Ace/Sequel-Ace/issues/199). In my case, it would still connect to 5.5 servers, but 5.1 and earlier would fail with "MySQL said: Bad handshake".

The workaround I came up with was to hack an earlier version the Sequel Ace application itself:

1. Download [Sequel Ace v2.0.2](https://github.com/Sequel-Ace/Sequel-Ace/releases/tag/2.0.2), the last version with pre-5.6 support
	- after moving to `/Applications`, rename to something like "Sequel Ace for old MySQL"
2. Within the app bundle, edit `Contents/Resources/ssh_config` to add:

           HostKeyAlgorithms +ssh-rsa
           PubkeyAcceptedKeyTypes +ssh-rsa
           KexAlgorithms +diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1

3. Replace `Contents/MacOS/SequelAceTunnelAssistant` with the file from same path within the current version of Sequel Ace
    - this binary is ARM-native, allowing it to use keychain passwords

With this modified app you should be able to connect to pre-5.6 MySQL databases. It'll share bookmarks with the current version, so be sure to only run one of them at a time!
