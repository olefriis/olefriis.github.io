---
tag: Stunt Car Racer
---
# My Stunt Car Racer Obsession
This is the first post in a series about some work I've done trying to bring the game Stunt Car Racer
to modern platforms. Before going technical in the following posts, let me try to explain why I have
decided to spend time on this.

Reading time: 5 minutes.

## What is Stunt Car Racer?
You can of course read up on it yourself on [the Wikipedia page](https://en.wikipedia.org/wiki/Stunt_Car_Racer),
but in short:

* It's a 3D racing game where you drive on ramps.
* It has a pretty realistic physics engine.
* It was released in 1989 on a host of platforms: Amiga, Amstrad CPC, Atari ST, Commodore 64, MS-DOS, and ZX Spectrum.

You can get an idea of what the game is like by watching this video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/wWCZ7lP1u6Q" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Why Rebuild Stunt Car Racer?
When I originally played the game, I was amazed and fascinated. I loved the 3D aestetics and the whole
premise of the game. So after I moved away from the Commodore 64 and later the Amiga, I wanted some way to
relive this experience.

Surely you can spin up a Commodore 64 emulator and play the game that way. You can
[run the emulator in your browser](https://c64online.com/c64-games/stunt-car-racer/). But this will
give you the old game. That was enough to fascinate me back in 1989, but nowadays, to provide the same
_experience_, we need higher frame rates, higher resolution, and, well, generally an experience that
fits the device you are using.

For the last 25 years, I've done a number of partial implementations of Stunt Car Racer for different
platforms: Java with my own 3D engine, Java with OpenGL, Android, and iOS (using SceneKit). I think I've got
pretty close to a good gaming experience, but I'm still far from finishing my iOS version. So at least for
now, my own "from scratch" approaches are stuck.

## Stunt Car Remake
Some years ago, I stumbled upon the [Stunt Car Racer Remake project on SourceForge](https://sourceforge.net/projects/stuntcarremake/):

![Screenshot of Stunt Car Racer Remake on SourceForge](/assets/images/stunt-car-racer-introduction/stunt-car-racer-remake.png)

There are also a few uploads on GitHub with some additional work done on e.g. supporting the XBox controller,
e.g. [fluffyfreak/stuntcarracer](https://github.com/fluffyfreak/stuntcarracer).

As you can see from the image above, this project is based on the original Amiga version. Yes, the author actually
took [27410 lines of Motorola 68000 assembly code](https://github.com/fluffyfreak/stuntcarracer/blob/master/Reference%20only/StuntCarRacer.s)
and converted it to C++ and DirectX/Direct3D! This means the game utilizes the original physics engine and contains
the original track data.

Unfortunately, this is Windows-only. And not only Windows-only, it also requires you to find some old DirectX DLLs
(which I can only find on shady sites...) in order to work. I want to play this game on my Mac, on my phone, on my
son's Chromebook, and everywhere else.

But we have the code! Now it's just a matter of converting it to other platforms.

## Then What?
There's this amazing project called [emscripten](https://emscripten.org):

![Screenshot of emscripten.org front page](/assets/images/stunt-car-racer-introduction/emscripten.png)

As the frontpage of the project site outlines, it allows you to compile C++ code to something that can
run in your browser. It converts OpenGL into WebGL, meaning we can use the graphics card to draw our 3D
graphics. And since it compiles into WebAssembly, the result is super fast.

This is almost all we need! Just some little details, like having OpenGL at our disposal instead of Direct3D,
are off. But it seems doable.

So I went ahead and forked the Stunt Car Racer Remake project on GitHub:
[olefriis/stuntcarracer](https://github.com/olefriis/stuntcarracer)

But how should I approach this conversion? This can be done in a lot of ways.

I could go through the C++ code, rip out the DirectX/Direct3D code and everything else referencing
Windows-specific APIs, and replace it with stuff that works in emscripten.

Or I could keep the C++ code intact and instead implement the DirectX/Direct3D and other Windows-specific
APIs used by the code using the frameworks supported by emscripten.

I went for the latter, and I decided to be pretty strict about not altering the C++ code. This was not
_entirely_ possible (we'll probably get back to that in a later post), but almost. I believe this
approach is the one that would give me the quickest path from "nothing compiles" to "I can draw a
single triangle on screen". But if you have an idea for a better approach, I'm always interested
(you can [hit me up on Twitter](https://twitter.com/olefriis)).

## Current Status
You can [go and play the game right now if you want to](https://olefriis.github.io/play/). The game
works in a desktop browser. As there are no touch controls, playing it on your phone won't work.
As in the Stunt Car Racer Remake project, there's no tournament mode. It's not a "progressive web
app", so it doesn't feel "native". And there's probably a ton of other things missing. Maybe we'll
get there.

It's just a spare-time project, and I'm not a real game developer, so everything takes a lot of time for
me. Plus, I have a track record of abandoning spare-time projects, so maybe I'll abandon this at some
time too. But I'll try to write a few blog posts about how the game ended up where it is right now,
including some linear algebra, and then hopefully after that I'll be able to implement new stuff and
blog about it in lockstep. Let's see. No promises.
