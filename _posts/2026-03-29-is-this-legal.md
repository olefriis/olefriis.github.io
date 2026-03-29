---
title: "Is This Legal?"
tags: 
 - Stunt Car Racer
 - LLMs
excerpt: "I recently thought \"aw, screw it!\" regarding incorporating the original artwork of Stunt Car Racer into the remake. Thus, if you play it today, you'll see that it looks a lot more like the original. I'll walk you through the recent changes."
---

# Is This Legal?
Reviving old games is almost by definition a gray area, legal-wise. I guess. A few weeks
ago I naively thought that as long as a game is more than 20 years old, it was past some
legal threshold, and then you could do whatever with the assets. Chatting a bit with one
of my lawyer friends, it seems like this is not necessarily the case.

So I guess everything in Stunt Car Racer is still copyrighted and shouldn't be meddled
with, without the consent of the authors or whomever has the rights. Just like I
shouldn't start publishing my own drawings of a certain mouse called Mickey whose
rights are held by a very large American entertainment company.

But... it's just that there are so many other projects similar to mine. Both regarding
Stunt Car Racer and a lot of other retro games. You cannot even buy these games
anywhere anymore, so nobody is set to lose anything from these projects.

Therefore, I recently thought "aw, screw it!" regarding incorporating some of the
original artwork of Stunt Car Racer into the remake. Thus, if you play it today, you'll
see that it looks a lot more like the original. I'll walk you through the recent
changes and how they were done.

And just to be clear: I have **not** done my "due diligence" at all regarding copyright
laws, so ripping the artwork from the original game may be super bad. I'm just thinking
that when so many others are doing something similar, it's no big deal. Open an issue in
[the repository](https://github.com/olefriis/stuntcarracer) and tell me otherwise if you
want.

## Cockpit and Boost Flames
Let's do the easy one first: The cockpit. All I had to do was start up the game in an
Amiga emulator and take a screenshot:

![Screenshot of the in-game experience](/assets/images/is-this-legal/screenshot-with-cockpit.png)

Opening this up in [Paintbrush](https://paintbrush.sourceforge.io), I could manually
remove everything from the image that was "not the cockpit", make that "not the cockpit"
part transparent, and get this:

![Everything except the cockpit removed](/assets/images/is-this-legal/cockpit.png)

Just putting this on top of the in-game experience and moving the HUD shown in
[the previous post](/2026/03/28/stunt-car-racer-gameplay-complete) to the designated
areas at the bottom looked pretty good:

![Game experience, now with cockpit!](/assets/images/is-this-legal/in-game-with-cockpit.png)

The flames that appear during boosting were a bit more challenging: It's an animation
with three different overlays, cycling through these. So I took a whole lot of
screenshots and afterwards sorted them so that I could find three different flame
images. For each of these, I manually opened them and removed anything that was "not
a flame". It was a bit of work, but cycling through these looks really good.

![Boost animation](/assets/images/is-this-legal/boost.gif)

## Wheels
But if you look at the screenshot I took of the original game (the uppermost image),
you'll notice something crucial missing: The wheels!

It turns out that these are also made up of three images for each side, looping to
give a sense of rotation. But even more annoyingly, you can only see them properly
when the car is pushed towards the track, e.g. when landing hard after a jump.

There were a few options:
* Do my best to capture the different wheel images in screenshots, then manually
  extract the wheels themselves. This would be tedious.
* "Steal" from other Stunt Car Racer remake projects. I know that some of them have
  succeeded in getting the wheel graphics somehow.
* Or the long shot: Ask Claude to extract the artwork from an ADF (Amiga Disk File)
  with the original game.

I didn't really believe in the last option, but when a few friends also brought
this up, I thought "Why not?". I found an ADF file from
[Amiga 500 Archive](https://www.amiga500archive.com/items.php?search=stunt+car+racer)
and basically told Claude to look at the Amiga source code and use that to find a
clever way of extracting the artwork from the ADF file. It wasn't an easy task for
Claude, but eventually it worked!

If you want all of the details of how Claude did it, I asked it to
[write up a guide that you can read](https://github.com/olefriis/stuntcarracer/blob/emscripten/Reference%20only/extracting-images.md).
If you want the artwork, you'll have to run
[the script](https://github.com/olefriis/stuntcarracer/blob/emscripten/Reference%20only/extract_images.py)
yourself - just clone the repository, get an ADF, and run the tool.

Getting the wheels properly incorporated took a bit of work, but I think it looks
really nice. This GIF capture doesn't do it justice as the frame rate is cut down,
but you hopefully get the idea:

![Wheel animation](/assets/images/is-this-legal/functioning-wheels.gif)

## Chains
I also made the chain effect work almost like in the original game. This was the version
before:

![First version of chains](/assets/images/is-this-legal/first-version-of-chains.png)

After incorporating the real graphics, it looks more legit:

![Final version of chains](/assets/images/is-this-legal/final-version-of-chains.gif)

## What Now?
Part of my intent with doing the Stunt Car Racer remake is to modernize the game for
the mobile and web era, so I don't _have_ to make it look exactly like the original,
but it's just a lot of fun. The cockpit, the boost flames, and the wheels are just
crucial to making the remake recognizable. Am I going too far? Don't hesitate reaching
out through e.g. [a GitHub issue](https://github.com/olefriis/stuntcarracer/issues).

Just for context, you can play the whole original game online on
[The Internet Archive](https://archive.org/details/Stunt_Car_Racer_1989_Micro_Style_a2).

Legal issues or not, if you want to check out the latest changes shown above, you can
[go and play the game](https://olefriis.github.io/play). It works for both mobile and
desktop!
