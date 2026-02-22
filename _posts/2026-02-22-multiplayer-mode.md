---
title: "Multiplayer Mode? No Problem!"
tags: 
 - Stunt Car Racer
 - LLMs
excerpt: "As always, Claude set out with high energy, and after about 15 minutes, it had something ready."
---
# Multiplayer Mode!
Nowadays, many (if not most?) games have multiplayer modes where you can challenge your
friends online, but when I was a child, there was no internet. Or, well, theoretially
ARPAnet existed, but just wasn't available to everybody. The way we would mostly play
together was sitting around the same gaming console or computer and cheer on each
other. Some games would offer split-screen modes so several players could play at once.
And then, a small number of games would offer the closest we would get to playing on
the internet: The ability to connect two computers through a serial cable.

This was a main feature of Stunt Car Racer, and as mentioned in
[my previous post](/2026/02/21/letting-the-llms-finish-my-spare-time-projects),
one of the big, remaining tasks for the Stunt Car Racer remake was multi-player mode.

I don't know why, but sometimes I just need "a breather" before I start up the LLMs on
the next big task, even though in principle it's not me doing the hard work, so I was
planning on waiting until next weekend to dive into this along with Claude. But today
I decided to just give it a go. My prompt was this:

> Let's see if we can tackle the next - rather big - feature!
> 
> We want multiplayer support. Meaning, two players in their own browser should be able
> to connect and play against each other.
> 
> I don't think the C++ code has any support for two-player games, but the Amiga source
> code certainly does. Have a look there how it's done.
> 
> To communicate, we'll use WebRTC. This means the browsers need to somehow be able to
> connect, and as I understand, this requires an online service they both connect to. The
> first browser will connect, get e.g. a 4-character code (and other information), and
> the second browser can connect to the service as well, provide the same 4-character
> code, and when the two browsers have exchanged information through the online service,
> they can connect to each other. If possible, for debug purposes to begin with, I want a
> more local solution. Can the two browsers exchange something else that I can just
> copy-paste between browsers? Or maybe just a local Ruby service (written in Sinatra)?
> 
> Apart from connecting to each other, there are considerations in the game. The two
> browsers need to run the game at exactly the same pace - the frame rate can differ (and
> should, so that each browser provides the best experience for the respective players),
> but the game logic speed has to be in sync.
> 
> Since sending messages through WebRTC will of course incur a small delay, we might have
> to delay local feedback as well - when the player hits a button, we may have to send
> this information to the other browser, and also get the other browser's input, and only
> when this has been exchanged, both browsers can apply the input. We may also need to
> exchange car position and speed, so the two browsers don't drift regarding game state.
> If you can come up with a better way to solve this, please do.
> 
> Initiating a two-player game happens on the main menu, where we will add a new button:
> Two-player game. The player then selects between being the part that selects the track
> and provides the connection information, or the one that simply enters the connection
> information and then plays the track that the other player has selected.

I didn't have high hopes, but as always, Claude set out with high energy, and after
about 15 minutes, it had something ready.

It wasn't great on first try: The opponent would be driving up in the sky because of
some scaling details of the height of each wheel, and after fixing that, the opponent
would look like it was positioned a little more to the right than it should be (and
sometimes hovering to the right of the track). And a few other details. But after I
pointed these issues out one by one, what is left is a really nicely working 2-player
mode.

Here's a Safari window and an iOS simulator playing the same game:

![Two players competing](/assets/images/multiplayer-mode/two-players.png)

## Technicalities
So how does it work? Well, how would I know? I just asked the LLM to do it for me! So
frankly I'm a bit fuzzy on the details, but judging from my quick screening of the code
and Claude's thinking, this is how I understand it:

Browsers cannot just connect to each other, so they need some service to exchange stuff
like IP addresses. For this, Claude made me a "signaling server" in Sinatra:
[server.rb](https://github.com/olefriis/stuntcarracer/blob/emscripten/signaling/server.rb).
That's what caused the most work for me, as I had to deploy this somewhere on the
internet. It's currently set up on [fly.io](https://fly.io). I haven't even checked
yet how much I need to pay after my 7-day free trial period, but it's probably worth
it...

Anyhow, this simple "signaling" service has an in-memory cache of four-character,
randomly generated "rooms". The "hosting" browser creates a new room and shows the
"room name", which is entered into the "joining" browser, and now the two can exchange
messages through the service.

Once the browsers have found each other, they communicate through a WebRTC session.
There's a reliable channel to handle control instructions, like starting and ending a
race, and an unreliable (/fast) channel used during the race, to which each player
just sends its coordinates continuously to each other, and then each game instance puts
the opponent at that last known location, handling collisions etc. that way.

You can mis-use this protocol to cheat and do strange things, but as this is used for a
two-player mode in which you know whom the opponent is, its... fine! And works
surprisingly well on a local network, giving you the same experience as when connecting
two Amigas with a serial cable.

In other words, it may not be perfect by a lot of measures, but it works, and was just
the last thing missing. I love it!
