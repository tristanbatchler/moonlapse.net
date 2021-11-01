---
layout: post
title: How to run your own Moonlapse server [Windows]
date: 2021-11-01T08:59:14.411Z
author: saltytaro
---
Interested in playing by your own rules? You can download the code to run your very own MoonlapseMUD server.

To follow along with this tutorial, you will need a machine running a Windows operating system. Do note this has only been tested with Windows 10.

### Prerequisites

1. Firstly you will need at least Python 3.6. You should follow the [official Microsoft instructions](https://docs.microsoft.com/en-us/windows/python/beginners) to obtain this.
1. Download pgAdmin 4, a PostgreSQL database management software tool. Visit (their download page for Windows)[https://www.pgadmin.org/download/pgadmin-4-windows] and select the most recent release.
1. Install `git`. Follow the instructions over at [the Git downloads page for Windows](https://git-scm.com/download/win).
1. Clone the MoonlapseMUD source from the current repository. Run the following in Powershell.

   ```shell
   git clone https://github.com/trithagoras/MoonlapseMUD
   ```
1. Initialise the database cluster. Open pgAdmin 4 and create a new user named `MoonlapseAdmin` and set/remember the password for this new user.
1. Create a new database called `Moonlapse`.
1. Create the database connection string and place it in the `MoonlapseMUD/server` directory.

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
1. Install dependencies. Open Powershell and run the following code, waiting for the dependencies to automatically install and for you to receive a database error.
    
    ```shell
    python MoonlapseMUD/server
    ```
1. Initialise the database.

    ```shell
    python MoonlapseMUD/server/manage.py makemigrations server
    python MoonlapseMUD/server/manage.py migrate
    python MoonlapseMUD/server/manage.py flush
    python MoonlapseMUD/server/loaddata.py
    ```
1. Run the server again!

    ```shell
    python MoonlapseMUD/server
    ```