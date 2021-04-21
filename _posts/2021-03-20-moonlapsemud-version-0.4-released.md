---
layout: post
title: MoonlapseMUD Version 0.4 Released
date: 2021-04-20T11:00:00.000Z
author: saltytaro
---
It's been a while since the last release. 3bolt and I have both been very busy with work and school, but we haven't forgotten about MoonlapseMUD!

This release finally features some gameplay features which we are very excited to share.

---

## Encryption and password security!
All communications are now encrypted in transit, and passwords are hashed at rest.

### In transit
When a new client connects, it exchanges public keys with the server. From then on out, all packets are sent encrypted using a hybrid of public (RSA) and private (AES) key cryptography.

The AES library we are using is offered in [PyCryptodome](https://pypi.org/project/pycryptodome). RSA falls under [this implementation](https://pypi.org/project/rsa).

Here's an example of a packet sent from the client to the server (although the process is identical from server to client):
1. A random, private AES key is generated
2. The packet is encrypted using this private key
3. The private key and the encrypted message are both bundled together and encrypted using the recipient's public RSA key
4. This is sent off

On the receiving end, decryption can take place:
1. The message is decrypted using the private RSA key revealing the AES private key and encrypted packet
2. The AES private key is used to decrypt the packet

In the future, it would be beneficial to only generate an AES key at the beginning of the session, transit that across via RSA, and from then on out, client-serve communications can take place purely off AES encryption. This would provide much more speed.

### At rest
Passwords are hashed and salted (using [hashlib](https://docs.python.org/3/library/hashlib.html)) before they are stored in the database. When a player reconnects to log in again, the password sent (after being decrypted on the server) is hashed and salted again before being compared to what's in the database. If it's a match, open sesame! We can verify you know your password without us even knowing what it is ourselves.

### User interface
Now sensitive text can be censored with the new UI overhaul!

![Login](/assets/images/2021/04/20/moonlapsemud-version-0.4-released/login.gif "Login")

---

## Looking
Players are now able to press `k` to look at the world around them!

![Pickup pickaxe](/assets/images/2021/04/20/moonlapsemud-version-0.4-released/pickup-pickaxe.gif "Pickup pickaxe")

---

## Resource gathering, items and inventory
There are naturally spawning resouces in the world which can be gathered by players with the correct tools.

![Gather iron](/assets/images/2021/04/20/moonlapsemud-version-0.4-released/gather-iron.gif "Gather iron")

![Chop wood](/assets/images/2021/04/20/moonlapsemud-version-0.4-released/chop-wood.gif "Chop wood")

---

## Weather
There is now a system for weather events. Currently only rain is supported. Rain does not pass through the roof (for maps with a roof layer).

![Weather showcase 1](/assets/images/2021/04/20/moonlapsemud-version-0.4-released/weather-showcase-1.gif "Weather showcase 1")

![Weather showcase 2](/assets/images/2021/04/20/moonlapsemud-version-0.4-released/weather-showcase-2.gif "Weather showcase 2")

---

## Easier client installation and dependencies
Our fight against using third-party libraries has come to an end (the libraries won). We realised it is not manageable to write the client in 100% pure, standard, Python so we have compromised by writing a script to check and install dependencies automatically. If the script detects the client's machine already satisfies the requirements, it will do nothing and launch the game. Otherwise, it will setup a virtual environment, install pip inside it, and proceed to collect all the dependencies with pip.

This even means we can install curses for Windows users for them!

![Installation](/assets/images/2021/04/20/moonlapsemud-version-0.4-released/installation.gif "Installation")

---

## Lots of improvements
Lots of refactoring, performance improvements, as always. One noteworthy minor improvement is now the user can save their preferred username so they don't have to type it every time. The system for this is very extensible so it can be used for lots of other preference things in the future.

That's basically it! Thanks for reading, and stay tuned for next time!
