---
layout: post
title: How to run your own Moonlapse server [Arch/Manjaro]
date: 2020-10-12T08:28:38.060Z
author: saltytaro
---
Interested in playing by your own rules? You can download the code to run your very own MoonlapseMUD server.

To follow along with this tutorial, you will need a machine running an operating system with access to [Arch Linux official repositories](https://wiki.archlinux.org/index.php/Official_repositories). In this example, we will be using [pacman](https://wiki.archlinux.org/index.php/pacman) which is standard in most Arch-based distributions.

### Prerequisites

1. Firstly you will need Python 3.6 which should be installed by default on most Arch-based systems.
2. Update the package list using the following command:

   ```shell
   sudo pacman -Syyu
   ```
3. Install [PIP, the Package Installer for Python](https://pypi.org/project/pip/), which is what we will use to obtain the rest of our prerequisite Python libraries\

   ```shell
   sudo pacman -S python-pip
   ```
4. Install PostgreSQL the database server the game will need to read from and write to.

   ```shell
   sudo pacman -S postgresql
   ```
5. Next, the drivers required to let Python and PostgreSQL talk to each other.

   ```shell
   sudo pacman -S python-psycopg2
   sudo pip install psycopg2
   ```
6. Install `git` and clone the MoonlapseMUD source from the current repository

   ```shell
   sudo pacman -S git
   git clone https://github.com/trithagoras/MoonlapseMUD
   sudo cp MoonlapseMUD/Moonlapse.sql /var/lib/postgres
   ```
7. Initialise the database cluster.

   ```shell
   sudo -iu postgres
   initdb -D /var/lib/postgres/data
   exit
   ```
8. Enable and start the PostgreSQL service.

   ```shell
   sudo systemctl enable postgresql
   sudo systemctl start postgresql
   ```
9. Set a password for the operating system user `postgres`, which is created when you install postgresql by default. This will be used to create your database.\

   ```shell
   sudo -u postgres psql postgres
   \password postgres
   \q
   ```

   You'll see:

   ```shell
   Enter new password:
   Enter it again:
   ```
10. From your terminal, create a new user to run the Moonlapse database and create the new database itself.\

```shell
sudo -iu postgres
createuser --interactive --pwprompt
Enter name of role to add: MoonlapseAdmin
Enter password for new role:
Enter it again:
Shall the new role be a superuser? (y/n) y
createdb Moonlapse
```

11. Install Twisted, the networking library for the server.\

```shell
sudo pip install twisted
```

12. Install Django, the library used for the ORM.\

```shell
sudo pip install django
```

13. Create the database connection string and place it in the `MoonlapseMUD/server` directory.\
    `MoonlapseMUD/server/connectionstrings.json`

```json
{
"user": "MoonlapseAdmin",
"password": "myPassword",
"host": "localhost",
"port": "5432",
"database": "Moonlapse"
}
```

14. Initialise the database.\

```shell
python MoonlapseMUD/server/manage.py migrate
```

14. Run the server!\

```shell
python MoonlapseMUD/server
```