---
layout: post
title: How to run your own Moonlapse server
date: 2020-08-17T22:07:57.096Z
author: saltytaro
---
Interested in playing by your own rules? You can download the code to run your very own MoonlapseMUD server.

To follow along with this tutorial, you will need a machine running an operating system with access to [Advanced Package Tool (or APT)](https://en.wikipedia.org/wiki/APT_(Package_Manager)).

### Prerequisites

1. Firstly you will need Python 3.6 which should be installed by default on most Debian-based systems.
2. Install [PIP, the Package Installer for Python](https://pypi.org/project/pip/), which is what we will use to obtain the rest of our prerequisite Python libraries\
   `sudo apt install python3-pip`
3. Install the PostgreSQL, the database server the game will need to read from and write to.\
   `sudo apt install postgresql`
4. Next, the drivers required to let Python and PostgreSQL talk to each other.\
   `sudo apt install python-psycopg2`\
   `sudo apt install libpq-dev`\
   `sudo pip3 install psycopg2`\
   *Note for Mac users, you can instead run the following command (assuming you have homebrew installed):\
   `brew install postgresql`\
   `sudo pip3 install psycopg2`\
   If you get a C compilation error such as:\
   "library not found for -lssl"\
   then run this command and try the pip3 install again.\
   `export LIBRARY_PATH=$LIBRARY_PATH:/usr/local/opt/openssl/lib/`
5. Install git and clone the MoonlapseMUD source from the current repository\
   `sudo apt install git`\
   `git clone https://github.com/trithagoras/MoonlapseMUD`
6. Set a password for the operating system user `postgres`, which is created when you install postgresql by default. This will be used to create your database.\
   `sudo -u postgres psql postgres
   \password posrgres`

   You'll see:\
   `Enter new password:
   Enter it again:`
7. From your terminal, create a new user to run the Moonlapse database and create the new database itself.\
   `su - postgres
   createuser --interactive --pwprompt
   Enter name of role to add: MoonlapseAdmin
   Enter password for new role:
   Enter it again:
   Shall the new role be a superuser? (y/n) y`\
   `createdb Moonlapse`
8. When you're running `psql` as the `postgres` user, create the database required for the game. Then run the script you downloaded earlier to create the structure for the game database.\
   `wget https://raw.githubusercontent.com/trithagoras/MoonlapseMUD/master/Moonlapse.sql`\
   \
   `psql Moonlapse`\
   `\i Moonlapse.sql`
9. Create the database connection string and place it in the `MoonlapseMUD/server` directory\
   `nano MoonlapseMUD/server/connectionstrings.json`

   `{`\
   `"user": "MoonlapseAdmin",`\
   `"password": "myPassword",`\
   `"host": "localhost",`\
   `"port": "5432",`\
   `"database": "Moonlapse"`\
   `}`
10. `python3 MoonlapseMUD/server`