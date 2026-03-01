---
title: "The Gameplay is Now Complete!"
tags: 
 - Stunt Car Racer
 - LLMs
excerpt: "I believe that by now, all gameplay elements in Stunt Car Racer are complete. I'll do a run-through of the last missing bits here."
---

# The Gameplay is Now Complete!
I believe that by now, all gameplay elements in Stunt Car Racer are complete. I'll do a
run-through of the last missing bits here.

Just to be clear, I am not saying that everything is perfect. While I think all elements
are there, a lot will probably still need tweaking. Especially I have a feeling that the
overall speed of the game during the races is a bit high compared to the original
version. Also, I'd love to do an overhaul of all the menu and overlay graphics.

## Thanks God it's Soon March!
I've really been having fun using Copilot (with Claude Opus) recently. Apparently so
much that I've spent all of this month's request budget:

![Premium Requests budget spent!](/assets/images/stunt-car-racer-gameplay-complete/almost-used-premium-requests.png)

But tomorrow it's March, and I can start all over. Phew!

## Tournament Mode
While I previously wrote about
[adding tournament mode](/2026/02/21/letting-the-llms-finish-my-spare-time-projects), I
had read somewhere on the internet that the original game did not simply have four
divisions, as the UI shows: Once you've progressed to Division 1 and won that, you will
now be promoted to the "Super League". This is essentially the original Division 1-4
with the same opponents again, except:
* The cars go faster.
* Boost will be burnt faster.
* The tracks change color a bit.

Being in the Super League also affects Practise mode. Additionally, I've made the
decision that when doing two-player mode, the "Super League or not" state is
inherited from the player that hosts the game.

Claude investigated the original Amiga source code and came up with these additional
metrics for "Super League":
* Engine power goes from 240 to 320 (in whatever units we use).
* There's "road cushion", which means the car has to smash harder at the track before
  taking damage. Otherwise, with 50% more engine power in the Super League, cars would
  be wrecked too soon.
* "Boost Unit Value" goes from 16 to 12, meaning boost burns 25% faster.

## More Information On Screen
The original Stunt Car Racer shows more information than I was showing on the
screen:
* Damage (which I've partially shown, but see below).
* Speed.
* Distance to opponent.
* Current lap.
* Current lap time.
* Best lap time.

So that's added now! It's all shown in a rather boring box to the left (except damage,
see below), but that's the best I could come up with now - I don't want it to clutter
the simple mobile controls. Maybe we'll get to something better later.

![New heads-up display](/assets/images/stunt-car-racer-gameplay-complete/hud.png)


## Saving of Progress
As we now have the Super League and all, I thought it would make sense to properly
save the progress, and also allow the player to pause and resume a racing season.
You can also choose to reset the current season (going back to how the start of the
season was set up) or the whole season state (going back to Division 4).

## Damage System
Until now, "my" version of Stunt Car Racer only had a one-dimensional damage system:
As you're hitting the track hard, the "damage meter" increases, and when it reaches
the end, you are "wrecked". But the Amiga version also had a notion of "holes". Unlike
"normal damage", holes persist between races. You get a hole when you hit your car
particularly hard.

Letting Claude examine the Amiga source code, it found these details:
* Regular damage goes from 0 to 240. When reaching 240, the car is wrecked, and you
  lose the game.
* Holes are purely visual. You can have between 0 and 10 holes. When you have all
  10 holes, nothing happens if you would otherwise get another hole.
* As such, the only point with holes is that it hides part of your "damage meter",
  making it difficult to see how much you are still allowed to damage your car.
* Between seasons, holes are repaired based on your position in the division: The
  1st place gets 3 holes repaired, the 2nd place gets 2 holes repaired, 3rd place
  gets 1 hole repaired, and the last place keeps all holes.
* When you are promoted to the Super League, you get a full repair.

The way that holes are currently visualized is not great - it just "blacks out"
parts of the damage bar, so you cannot see how much you are damaged. I want it to
be a bit more clear, but that'll have to wait until I have a new budget for
Copilot tomorrow!

![Super leage and a hole](/assets/images/stunt-car-racer-gameplay-complete/super-league-and-holes.png)

## Play It!
As usual, you can [go and play the game](https://olefriis.github.io/play). It works
for both mobile and desktop!
