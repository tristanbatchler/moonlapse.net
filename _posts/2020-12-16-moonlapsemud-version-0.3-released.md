---
layout: post
title: MoonlapseMUD Version 0.3 Released
date: 2020-12-15T21:19:00.000Z
author: saltytaro
---
There have been some very exciting changes since version 0.2, both to the code quality/infrastructure and to the game elements.

Here is a rundown of the most noteworthy changes.

---

## Server rewrite using [Twisted](https://twistedmatrix.com/)
To improve on stability, code quality, power, and efficiency, the entire server-side code was re-written early on in development. 

What used to be a clumsy, multi-threaded, pure Python sockets driven application is now a full-fledged Twisted protocol factory.

The new server has some cool benefits:
* It was easy to implement different server "rooms" which are, in themselves, isolated "sub-servers". It's not possible to send or receive information to/from other players across rooms which cuts down on unnecessary network interference. In the future, a global communication channel will need to be implemented.
* Twisted protocols act like state machines so it is much better at handling packets thrown at it. It is designed to only accept certain packet types when they are expected.
* Speaking of packets, a new networking module was introduced which includes a special packet class, written in pure Python, so they can be shared between the server and the client. Here's an example:

    From the client
    ```python
    lp = packet.LoginPacket("username", "password")
    server_socket.send(loginpacket.tobytes())
    ```

    From the server, Twisted has a method called `stringReceived` which is fired whenever bytes a received over the network.
    ```python
    @override
    def stringReceived(self, string: bytes) -> None:
        p: packet.Packet = packet.frombytes(string)
        self._debug(f"Received packet from my client {p}")
        self.state(p)
    ```

    `self.state` is a Callable class member. For instance, if `self.state` is equal to `self._GETENTRY`, the following method is called.
    ```python
    def _GETENTRY(self, p: Union[packet.LoginPacket, packet.RegisterPacket]) -> None:
        if isinstance(p, packet.LoginPacket):
            self._login_user(p)
        elif isinstance(p, packet.RegisterPacket):
            self._register_user(p)
    ```
Notice the `Packet.frombytes` method magically returns the packet of the correct type. This is what allows the server (and client) to check the packet type is what is expected first before processing it and offers more control and security. This is the power of working with Twisted, sharing a packet class between server and client, and using state machines to control the logic of the server.

---

## Using [Django](https://www.djangoproject.com) ORM
Our old database tables were completely decoupled from our models and the gap was bridged shoddily with some hand-written SQL queries which were prone to SQL injection, inefficient, difficult to maintain, and, frankly, just terrible all-round.

Django is originally a web framework for Python, but what we're interested in is its extremely powerful object relational mapping (ORM) framework. Benefits:
* Models as Python objects are easily saved and pushed to the database without the need for handwritten SQL
* Superior security - can rely on Django sanitising models before inserting them to the database
* Superior efficiency - so the game can run faster!
* Migrations!

When the game is updated, e.g. a new skill is introduced or an entity model needs a new attribute, Django (mostly) automatically propagates these changes to the database using its powerful migrations system.

Although it was a huge slog, the switch to using both Django and Twisted was a wise move early in development, otherwise it would have been much more difficult. These two powerful frameworks will aid in much easier development for future released.

---

## Enhanced maps
The map system was rewritten so that the following can occur:
* We can rapidly create maps of differing terrain and static objects and save them as files to be used by the server
* These map files are compressable and, importanly, **uncompressable** without the need of a third party library.
    * This is because one of our goals and challenges with MoonlapseMUD is to create a highly compatible, accessible, and customisable game client which requires almost no setup. This means it has to be written with pure Python.
* The maps contain a collision mask of sorts, in fact, it ended up containing three layers - ground terrain, solid objects, and roof terrain.

> ### What we came up with is the following:
> 1. Under `maps/layouts/`, create a new directory with the name of your map. This must be unique. E.g. `maps/layouts/forest/`.
> 2. Inside this new folder, create three files.
>       * ground.data
>       * roof.data
>       * solid.data
> 
> These files, respectively, represent terrain for the ground level, roof level, and things between which are collidable.
>
> These files must be formatted according to the Moonlapse map file specification.
>
> ### Moonlapse map file specification?
> Yeah, it's a long story. Basically something that looks like this:
> ```
> 64$
> 63$}
> 24G6(34}
> ```
> In this made up context, what this means is
> ```
> A row of 64x "$" terrain tiles
> A row of 63x "$" terrain tiles followed by a "}" terrain tile
> A row of 24x "G" terrain tiles followed by 6x "(" terrain tiles followed by 34 "}" terrain tiles
> ```
>
> Whatever `$`, `}`, `G`, `{`, etc. represents is up to you and defined in `maps/__init__.py`.
>
> ### How do I make one of these files?
> You can draw a bitmap file and use the `maps/bmp2ml.py` script to spit out a properly formatted file. The script defines a colour palette for you, so your bitmaps need to have those exact colours loaded.

In the future, we will likely add a proper map editor GUI with the names of the terrain materials but please bear with the project in its early stages.

Here are some example bitmaps and what they look like as compressed data.

<img src="/assets/images/2020/12/16/moonlapsemud-version-0.3-released/ground.bmp" alt="Tavern ground map" title="Tavern ground map" width="32" />

`maps/layouts/tavern/ground.data`
```
32z
6B18z8B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
```

<img src="/assets/images/2020/12/16/moonlapsemud-version-0.3-released/solid.bmp" alt="Tavern solid map" title="Tavern solid map" width="32" />

`maps/layouts/tavern/solid.data`
```
32i
5iB18iB7i
5i20B7i
5i20B7i
7iB3iB3iB6iB9i
17iB14i
26iB5i
3iB28i
3i2B20i2B5i
2i4BiB17i2BiB3i
Bi4B26i
3i2B27i
32i
3iB28i
32i
32i
```

<img src="/assets/images/2020/12/16/moonlapsemud-version-0.3-released/roof.bmp" alt="Tavern roof map" title="Tavern roof map" width="32" />

`maps/layouts/tavern/roof.data`
```
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
32B
```

And here is how it looks in-game
![Tavern screebshot](/assets/images/2020/12/16/moonlapsemud-version-0.3-released/tavern-screenshot.png "Tavern screenshot")

---

## Portals
It is now possible to teleport to any location in the game with portals! This includes across different rooms.
![Portals demo](/assets/images/2020/12/16/moonlapsemud-version-0.3-released/portals.gif "Portals demo")


Conceptually, a portal is just an entity which has a position in the world, as well as another "linked" position. When a player moves into the portal, instead of occupying the same space as that portal, they immediately assume the portal's linked position.

---

Aaaaand that's it! Thanks for reading, we are really excited about releasing version 0.3 of MoonlapseMUD!