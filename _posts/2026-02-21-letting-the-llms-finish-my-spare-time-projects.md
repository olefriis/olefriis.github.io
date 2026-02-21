---
title: "Letting the LLMs Finish My Spare Time Projects"
tags: 
 - Stunt Car Racer
 - LLMs
excerpt: "Now, with this machine uprising in the form of LLMs, I thought that I would do my part of keeping the machines busy a while finishing the game. You can thank me later."
---
# Letting the LLMs Finish My Spare Time Projects
Like most other software developers these days, I've been blown away by what modern Large Language Models
can do. I don't want to jump into all the apocalyptic talk about them taking "our jobs", since I've got
nothing to add to that, but I want to share what I've been using LLMs for in my spare time recently.

So, I have had this ongoing project for many, many years of
[recreating Stunt Car Racer](/2022/01/16/stunt-car-racer-introduction.md), but it's been lying dormant for
(checking git commit log) almost four years. Problem was, I had done the most interesting technical aspects
of the game, like getting it building with Emscripten and getting the graphics, sound, and input working.
It was a playable game.

The other ideas I had for the game, like making it playable on mobile devices and implementing the original
"tournament mode", are not interesting technical challenges. It's just a lot of fiddling.

Now, with this machine uprising in the form of LLMs, I thought that I would do my part of keeping the machines
busy a while finishing the game. You can thank me later.

So let's do a status of what ~~I've~~Claude has been doing lately.

If you're impatient, like me, just [go ahead and play the game](https://olefriis.github.io/play/) instead of
plowing through this wall of text and images.

## Getting Back on Track
First of all, the world has moved on in four years, and the code would no longer compile with the newest
version of Emscripten. I fought it for a while, but then thought Claude would be better at it. And sure
enough, a couple of minutes later,
[it all compiled nicely](https://github.com/fluffyfreak/stuntcarracer/commit/74acbbc1ee4cc541ec4735e02701cdebefc17f2f).

## Smooth Framerate
The original [fluffyfreak/stuntcarracer](https://github.com/fluffyfreak/stuntcarracer) repository, which I've
forked, has an interesting branch:
[SmoothFrameratePatch](https://github.com/fluffyfreak/stuntcarracer/tree/SmoothFrameratePatch). This has
some quaternion magic that I don't understand - I am not a game developer, and quaternions is an area I've
never dived into.

Anyhow, instead of just applying the branch, I asked Claude to use the branch as inspiration for
implementing smooth framerate on my fork. It took a few turns, where rotations would go the wrong way and
the opponent's shadow would still be janky, etc., but the result is very nice.

I should probably add a video here, but I don't have the software to grab 50fps video... so just
[go and play it yourself](https://olefriis.github.io/play/) to see if you like the results!

## Playable on Mobile!
I think that Stunt Car Racer would have been the perfect mobile game:
* The graphics are pretty simple, so they lend themselves nicely to a small screen.
* Each game is relatively quick (3 minutes?), making it a good way to spend time waiting for the bus.

This requires setting up some new overlays that look good on a mobile device. Yawn. But Claude was ready
for the task. I like it!

![Mobile overlay](/assets/images/letting-the-llms-finish-my-spare-time-projects/mobile-overlay.png)

Also, why not make it a real Progressive Web App? That's another tedious task, but of course Claude was
up for it. So now you can click an "Install" button on Android or add the web page the home screen on iOS,
and you get a beautiful, auto-updating, full-screen experience on mobile!

## Tournament Mode
The original game has the "tournament mode", in which the player races against various opponents in order
to get promoted from Division 4 to Division 1:

![Tournament overview](/assets/images/letting-the-llms-finish-my-spare-time-projects/tournament.png)

Again, this is not a super interesting task to do, so I asked Claude to do this. Claude looked through
the
[original Amiga sources](https://github.com/olefriis/stuntcarracer/blob/emscripten/Reference%20only/StuntCarRacer.s)
to find out how this logic had to work, and now we have a similar mode:

![New tournament overview](/assets/images/letting-the-llms-finish-my-spare-time-projects/new-tournament.png)

It's really impressive that Claude could extract the exact same initial set-up of players and tracks in
each division. As you can see, the new graphics does not really resemble the original. I don't know... I'm
a little afraid of using the original graphics in this remake because it may make people upset, but given
the original game came out in 1989 (37 years ago???), maybe I shouldn't worry too much.

## Lots of Internals
Those are the visible changes made by Claude. In the process, the code has been restructured to revolve
around JavaScript instead of being driven by the C++ code. This has made it easier to add new menus, and
stuff like the tournament logic doesn't have to live in C++. I like these changes!

## Other Projects
The world hasn't stood still for the last four years, and jumping into YouTube etc. again, I can see that
other Stunt Car Racer projects have moved along as well during that time:
* [Stunt Car Racer Remake - Fan based PC remake in the works of one of Geoff Crammonds finest games](https://www.indieretronews.com/2025/12/stunt-car-racer-remake-fan-based-remake.html):
  Seems like an effort to bring back "the real game", based on the same C++ codebase as I am using.
* [ptitSeb's fork is also moving along](https://github.com/ptitSeb/stuntcarremake), with the latest commit
  being only a month old.
* [A 60fps version of the original game](https://www.youtube.com/watch?v=1RVYhO0l3Ts) is also being
  worked on. This looks really amazing!
* Watch [this very interesting walk-through of the game's history and some of its physics](https://www.youtube.com/watch?v=hcmSBwLKVOY).

## More Work to Do!
There's still more to do, and I'm enjoying getting Claude to work on this project in the evenings. Off
the top of my head, I still need to do this:
* Some kind of in-game "car overlay", like in the original game. Making it feel like you're in a real
  "wobbly" stunt car, showing boosts as fire from the cylinders, etc.
* The damage system is not authentic yet. The original game had a notion of damage and "holes". Damage
  would reset between races, but "holes" would persist (and reset between seasons if you were in Division
  4).
* The last video mentioned above talks about a "super league". I never played the original game that far,
  but apparently, after winning the original Division 1, you are promoted to a "super league" with more
  difficult opponents. I don't know how that works, but hopefully YouTube (and the Amiga source code) can
  shed some light on this.
* And the biggie... multi-player support!
