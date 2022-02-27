---
tag: Stunt Car Racer
---
# Stunt Car Racer: Starting From Zero
Before starting work on making graphics, sound, and input work in the browser-based
version of Stunt Car Racer, I needed a baseline: A point where everything compiles,
doesn't crash, but otherwise doesn't do much. Getting there required a surprising
amount of grunt work.

This is the second post in a series about recreating the
[Stunt Car Racer](https://en.wikipedia.org/wiki/Stunt_Car_Racer) game for modern
platforms. In [the first post](/2022/01/16/stunt-car-racer-introduction.html) we
went through why I want to do this. I found
[the Stunt Car Racer Remake project on SourceForge](https://sourceforge.net/projects/stuntcarremake/)
which is a C++ code base very specifically targeting Windows and DirectX. I want to
make this run in a browser using the amazing [emscripten](https://emscripten.org)
framework, which, as we will find out, involves implementing a whole lot of Windows
and DirectX APIs using standard C/C++ and OpenGL.

Anyhow, before we get to the exciting stuff like drawing graphics on the screen,
we need to have _something_ compiling. This post is about getting to that point.

Reading time: 10 minutes.
## Initial Status
Well, I started by installing [emscripten](https://emscripten.org) with Homebrew.
As with most things Homebrew, this was straightforward. Now I could run the `emcc`
command, which will compile C++ code and output WebAssembly. It seems like the
`Common` directory contains files that need to be on the include path, so let's
try this (on [this git SHA](https://github.com/olefriis/stuntcarracer/commit/306be6764b4ecee80c0758a0fd9f664c66bbb0d0)):

```
$ emcc StuntCarRacer.cpp -ICommon/            
In file included from StuntCarRacer.cpp:10:
Common/dxstdafx.h:45:10: fatal error: 'windows.h' file not found
#include <windows.h>
         ^~~~~~~~~~~
1 error generated.
```

Good news: There's only one error. But the error also indicates that I may
need to implement a bit of Windows-specific APIs and types, in addition to the
DirectX stuff I was already aware of.

Let's remove the requirement of the `dxstdafx.h` file from the `StuntCarRacer.cpp`
file, to get an initial feel for how much is required, and let's try again (output
pruned a bit):

```
emcc StuntCarRacer.cpp -ICommon/ 
In file included from StuntCarRacer.cpp:13:
./StuntCarRacer.h:40:5: error: unknown type name 'D3DXVECTOR3'
    D3DXVECTOR3 pos;    // The untransformed position for the vertex
    ^
./StuntCarRacer.h:41:5: error: unknown type name 'DWORD'
    DWORD color;                // The vertex diffuse color value
    ^
./StuntCarRacer.h:42:2: error: unknown type name 'FLOAT'
        FLOAT tu,tv;            // The texture co-ordinates
        ^
[...]
In file included from StuntCarRacer.cpp:14:
./3D Engine.h:141:8: error: unknown type name 'HRESULT'
extern HRESULT CreatePolygonVertexBuffer (IDirect3DDevice9 *pd3dDevice);
       ^
./3D Engine.h:141:43: error: unknown type name 'IDirect3DDevice9'
extern HRESULT CreatePolygonVertexBuffer (IDirect3DDevice9 *pd3dDevice);
                                          ^
./3D Engine.h:145:26: error: unknown type name 'POINT'
extern void DrawPolygon( POINT *pptr,
                         ^
[...]
fatal error: too many errors emitted, stopping now [-ferror-limit=]
20 errors generated.
```

Bad news: There's definitely more than one error. I have no experience with C++
programming on Windows, so all these upper-case types (`FLOAT` instead of just
using the C++ `float`, same with `DWORD` and many others) was new to me. Apparently
Windows C++ programmers are not content with the types provided by default by C++.
I con't know why, but probably it has something to do with being explicit about
how many bits are in which types, which types are signed, etc.

There's also a thing called `HRESULT` which seems to be used throughout the Windows
APIs. This defines a set of [standard error codes](https://docs.microsoft.com/en-us/windows/win32/seccrypto/common-hresult-values)
which are luckily well documented.

And of course DirectX is rearing its head a bit here, in the form of `D3DXVECTOR3`
`IDirect3DDevice9`, and `POINT`.

## The Process
I had to implement all of these types. No turning back now - the job was to go through
each and every error with missing types or missing methods, and implement what was
missing. I am not going to write up everything I did in this phase, as it will not be
interesting to anybody. Apart from the unexpected amount of Windows types, it gave me
a good idea of the amount of functionality I would have to implement later to emulate
the DirectX API.

It turned out that DirectX is [amazingly well documented](https://docs.microsoft.com/en-us/windows/win32/direct3d9/dx9-graphics-reference).
I work for Microsoft, so this may seem like a plug, but really, whenever I had to look
up a type, I just ~~Googled~~ Binged it, and in most cases the first search result
pointed to the official documentation, which always turned out to be thorough and
well-written.

While churning through these compile errors, one thing that got me a few times was that
I thought I was done because I had just fixed the last compile error. But unfortunately,
that just meant that `emcc` was able to compile _that_ file, and then it would move on
to the next file, and then a whole new set of compile errors would be shown to me. It
seemed like a never-ending game of "whack-a-mole".

## Hello, emscripten Console
But eventually I got there:
[The glorious commit where everything compiles](https://github.com/olefriis/stuntcarracer/commit/b85aa44109f03e4bc585631331b09ffe5886af59).
You can check it out and run the `./build-and-serve.sh` script. This will compile the
code and start up a web server so you can go to
[http://localhost:8000/source.html](http://localhost:8000/source.html). This will show
you the default emscripten page that looks like this:

![Screenshot of Emscripten console with an exception](/assets/images/stunt-car-racer-starting-from-zero/emscripten-console-with-exception.png)

The big black box with text shows some debug output. Basically that's just a lot of
`puts` lines from all of the stubbed functions I've created. This is not super interesting,
except for the fact that, hey, something's working!!

The black box in the middle of the screen is where, eventually, our OpenGL code will
render beautiful 3D images. Since we only have stub functions, nothing is happening
here for now.

But also, notice the text "Exception thrown, see JavaScript console". Let's look at the
JavaScript console:

![Screenshot of JavaScript console with an error](/assets/images/stunt-car-racer-starting-from-zero/javascript-console-with-error.png)

Something is out of bounds. But what?

## Debugging or Not
I really don't know how to properly debug WebAssembly. Chrome has some plug-ins for
single-stepping WebAssembly bytecode, but I haven't found anything that lets me set
breakpoints and step through the original C++ source code. I may have been looking in
the wrong places.

So... `printf` and `puts` debugging...

I'll spare you most of the details. I had to
[implement resource loading](https://github.com/olefriis/stuntcarracer/commit/9c0fc58989daf46955026df01e4bb5aded138ea8),
otherwise the track reading code would naturally crash. I had to
[implement some other APIs just enough to not make everything crash](https://github.com/olefriis/stuntcarracer/commit/c93d2b157ada6fb45221e61323e654f797cb22b6).
Basically it's "a lot of nothing".

And finally, in [this commit](https://github.com/olefriis/stuntcarracer/commit/31f42967a895ef858a473ba688877c47dcb6652c),
we get this console:

![Screenshot of Emscripten console with no exceptions](/assets/images/stunt-car-racer-starting-from-zero/emscripten-console-without-errors.png)

Yay!

## One More Thing
The above version executes one iteration of the main loop, and then exits. But let's
end this post on a high note: We want the game loop to run over and over again, just
like it's going to do when the game is done. But of course with no graphics, no sound,
and no input. And lots of other stuff missing.

The original code has [this call to a DirectX function](https://github.com/fluffyfreak/stuntcarracer/blob/master/StuntCarRacer.cpp#L1647)
that will run an internal loop until the user quits the game. Then the function will
exit, and the game will clean up all resources:

```C++
    // Start the render loop
    DXUTMainLoop();
```

This doesn't work in a browser. Because of the execution model, we are not allowed to
block the JavaScript thread - JavaScript is single-threaded, so if we block the one
thread there is, nothing else using JavaScript can run in the browser.

Emscripten doesn't magically fix this for us. We need to
[restructure the program to rely on callbacks](https://emscripten.org/docs/porting/emscripten-runtime-environment.html#browser-main-loop)
so that we only do any processing when it is "our turn".

This is no big deal. But it does violate
[my initial principle of not touching the original code at all](/2022/01/16/stunt-car-racer-introduction.html#then-what).
But there's not really anything I can do about it. (Well, theoretically maybe I could
create a C macro to turn the call to `DXUTMainLoop()` into something weird, but I don't
want to go there.)

## Finally
With [this commit](https://github.com/olefriis/stuntcarracer/commit/0dfd94cb8ffb29ffec36936698831fc75084acf5),
which also does a whole lot more stubbing, we have this main loop function:

```C++
// Our "main loop" function. This callback receives the current time as
// reported by the browser, and the user data we provide in the call to
// emscripten_request_animation_frame_loop().
EM_BOOL one_iter(double time, void* userData) {
	// Can render to the screen here, etc.
	puts("One iteration");

	if (frameMoveCallback) {
		frameMoveCallback(DXUTGetD3DDevice(), time, time, null);
	}

	if (frameRenderCallback) {
		frameRenderCallback(DXUTGetD3DDevice(), time, time, null);
	}

	// Return true to keep the loop running.
	//return EM_TRUE;
	// Well, return false right now to allow for some debugging
	puts("Iteration done");
	return EM_FALSE;
}
```

It emulates a bit of DirectX by calling into some existing hooks in the existing
code, and then it returns `EM_FALSE` to signal that we don't want to get another
callback from emscripten. If we change that to `EM_TRUE`, run `./buld-and-serve.sh`
again, and go to [http://localhost:8000/source.html](http://localhost:8000/source.html),
we get... an emscripten console running something that prints a _lot_ of text to
the debug area, and just keeps printing and printing and printing. This is not
super useful for further development, which is why it is turned off in the commit.

But we're there! We have "a walking skeleton" of a game. All of the original logic
is running inside our scaffold of stub functions. It even reads the original Amiga
tracks.

Nice! Now the fun can begin. Next time we'll do some 3D graphics for real.
