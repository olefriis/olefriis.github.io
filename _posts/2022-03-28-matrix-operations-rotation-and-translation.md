---
title: "Matrix Operations: Rotation and Translation"
tags: 
 - Stunt Car Racer
 - Math
excerpt: "Let's dive into some of the actual math required to make a 3D game work. This time we will look at some
matrix algebra that enables us to do rotation and translation of our 3D objects."
---
# Matrix Operations: Rotation and Translation
Let's dive into some of the actual math required to make a 3D game work. This time we will look at some
matrix algebra that enables us to do rotation and translation of our 3D objects.

This is the fourth post in a series about recreating the
[Stunt Car Racer](https://en.wikipedia.org/wiki/Stunt_Car_Racer) game for modern platforms. You can find
the previous posts by looking the [Stunt Car Racer tag](/tags). A very quick recap: We have some existing
C++ code which uses the Direct3D API to render the graphics, but since that is Windows-specific, we want
to convert it to OpenGL. This involves understanding some of the matrix math going on, since it works
_slightly_ differently in Direct3D and OpenGL - the order of matrix multiplications are switched around,
and some of the coordinate systems are a bit different.

In the [previous post](/2022/02/27/stunt-car-racer-draw-something-anything) we quickly skipped past the
matrix math required to do rotation, translation, and perspective. Let's dive into that now, since it's
really interesting. In this post we will do rotation and translation, and then we will look at perspective
in the next post.

Reading time: 10 minutes

## The Vertex Shader
As you may remember, our vertex shader - the function that transforms our initial 3D points to 2D points
that can be rendered on our 2D screens - looks something like this:

```
attribute vec3 vPosition;
uniform mat4 projectionMatrix;
uniform mat4 viewMatrix;
uniform mat4 worldMatrix;

void main() {
  gl_Position = vPosition * worldMatrix * viewMatrix * projectionMatrix;
}
```

This is running as part of the OpenGL rendering pipeline, to which we supply an array of 3D points.
These points will eventually be mapped to the 2D space of our monitor. OpenGL tears apart our array of 3D points
and feeds them to individual calls to the vertex shader through the `vPosition` variable. We also supply three
matrices: A projection matrix, a view matrix, and a world matrix.

You may have noticed that the one variable that changes on each invocation of the vertex shader is prefixed
with `attribute`, while the variables that stay constant during the rendering of our array of 3D points are
prefixed with `uniform`. We won't get into those specifics in this post, but it's something you need to keep
in mind when writing your own vertex shader.

Why supply three matrices? This is a convention in 3D graphics libraries.

* The world matrix will transform points from "model space" to wherever in the world and in whatever rotation
  your model is placed.
* The view matrix will transform points from "world space" to "view space", which means the points will be moved
  to adjust for the camera position and rotation. After applying this matrix, the camera will be at the (0, 0, 0)
  point and looking in a certain, well-defined direction.
* The projection matrix ensures that points far away from the camera are drawn closer together than points close
  to the camera, so it looks like 3D.

I'll provide some examples in a moment. We _could_ just multiply all these matrices before invoking the OpenGL
pipeline and have a simpler vertex shader, but I want to stay true to what the current C++ Stunt Car Racer
code is doing. Besides, there are some benefits to this way of doing it - for example, the view matrix will
only change once on each new frame we render, while the projection matrix will only change when the player
resizes the window.

You may be wondering why we are using 4x4 matrices (the `mat4` type in the shader code above) when we want to
render 3D objects, not 4D objects. This will become apparent soon.

## Dial Down on Dimensions
But before we proceed, allow me to simplify all the math involved. Instead of taking 3D objects, putting them
in a 3D world, and rendering them on a 2D screen, let's take 2D objects, put them in a 2D world, and render
them on a 1D screen. (I know, not a lot of blockbuster games transfer nicely to a 1D display...) This allows
us to more easily show - in this blog post, on your 2D screen - what is going on. And luckily, the linear
algebra we use as a foundation scales easily from 2 to 3 dimensions. Just trust me 😅

If that's OK with you, then consider this example: We have two objects (paper planes?) that we first move to
their "world positions". Then we move the world around so that the camera is at a certain place and looking
upwards, and finally we "flatten" the world so we can draw it on our 1-dimensional screen.

![Translating two model objects to world, view, and perspective](/assets/images/matrix-operations-rotation-and-translation/transformations.png)

## Refresher: Multiply a Vector and a Matrix
In the rest of this article, we'll need to multiply vectors and matrices. You can look up the rules, but I
will spare you the trouble. Multiplying a two-dimensional vector and a 2x2 matrix results in a two-dimensional
vector, and it goes like this:

<math>
  <mrow>
    <mrow>
      <mo>[</mo>
      <mi>x</mi>
      <mo>,</mo>
      <mi>y</mi>
      <mo>]</mo>
    </mrow>
    <mo>&sdot;</mo>
    <mrow>
      <mo>[</mo>
      <mtable>
        <mtr>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>1</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>2</mn>
              </mrow>
            </msub>
          </mtd>
        </mtr>
        <mtr>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>1</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>2</mn>
              </mrow>
            </msub>
          </mtd>
        </mtr>
      </mtable>
      <mo>]</mo>
    </mrow>
    <mo>=</mo>
    <mrow>
      <mo>[</mo>
      <mtable>
        <mtr>
          <mtd>
            <mi>x</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>1</mn>
              </mrow>
            </msub>
            <mo>+</mo>
            <mi>y</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>1</mn>
              </mrow>
            </msub>
            <mo>,</mo>
          </mtd>
          <mtd>
            <mi>x</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>2</mn>
              </mrow>
            </msub>
            <mo>+</mo>
            <mi>y</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>2</mn>
              </mrow>
            </msub>
          </mtd>
        </mtr>
      </mtable>
      <mo>]</mo>
    </mrow>
  </mrow>
</math>

Similarly, multiplying a three-dimensional vector with a 3x3 matrix goes like this:

<math>
  <mrow>
    <mrow>
      <mo>[</mo>
      <mi>x</mi>
      <mo>,</mo>
      <mi>y</mi>
      <mo>,</mo>
      <mi>z</mi>
      <mo>]</mo>
    </mrow>
    <mo>&sdot;</mo>
    <mrow>
      <mo>[</mo>
      <mtable>
        <mtr>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>1</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>2</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>3</mn>
              </mrow>
            </msub>
          </mtd>
        </mtr>
        <mtr>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>1</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>2</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>3</mn>
              </mrow>
            </msub>
          </mtd>
        </mtr>
        <mtr>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>3</mn>
                <mn>1</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>3</mn>
                <mn>2</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>3</mn>
                <mn>3</mn>
              </mrow>
            </msub>
          </mtd>
        </mtr>
      </mtable>
      <mo>]</mo>
    </mrow>
    <mo>=</mo>
    <mrow>
      <mo>[</mo>
      <mtable>
        <mtr>
          <mtd>
            <mi>x</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>1</mn>
              </mrow>
            </msub>
            <mo>+</mo>
            <mi>y</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>1</mn>
              </mrow>
            </msub>
            <mo>+</mo>
            <mi>z</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>3</mn>
                <mn>1</mn>
              </mrow>
            </msub>
            <mo>,</mo>
          </mtd>
          <mtd>
            <mi>x</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>2</mn>
              </mrow>
            </msub>
            <mo>+</mo>
            <mi>y</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>2</mn>
              </mrow>
            </msub>
            <mo>+</mo>
            <mi>z</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>3</mn>
                <mn>2</mn>
              </mrow>
            </msub>
            <mo>,</mo>
          </mtd>
          <mtd>
            <mi>x</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>3</mn>
              </mrow>
            </msub>
            <mo>+</mo>
            <mi>y</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>3</mn>
              </mrow>
            </msub>
            <mo>+</mo>
            <mi>z</mi>
            <mo>&sdot;</mo>
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>3</mn>
                <mn>3</mn>
              </mrow>
            </msub>
          </mtd>
        </mtr>
      </mtable>
      <mo>]</mo>
    </mrow>
  </mrow>
</math>

You can also multiply matrices with matrices, but you'll have to look that up by yourself. (I'm
using [MathML](https://www.w3.org/Math/) here, and it's just too painful to write this up.)

## Rotation
Let's say we have a point
<math>
  <mrow>
    <mo>[</mo>
    <msub>
      <mi>x</mi>
      <mn>1</mn>
    </msub>
    <mo>,</mo>
    <msub>
      <mi>y</mi>
      <mn>1</mn>
    </msub>
    <mo>]</mo>
  </mrow>
</math>
that we want to rotate at angle ß around
<math>
  <mrow>
    <mo>[</mo>
    <mn>0</mn>
    <mo>,</mo>
    <mn>0</mn>
    <mo>]</mo>
  </mrow>
</math>
so we end up at point
<math>
  <mrow>
    <mo>[</mo>
    <msub>
      <mi>x</mi>
      <mn>2</mn>
    </msub>
    <mo>,</mo>
    <msub>
      <mi>y</mi>
      <mn>2</mn>
    </msub>
    <mo>]</mo>
  </mrow>
</math>
![Rotating a point angle ß around 0,0](/assets/images/matrix-operations-rotation-and-translation/rotation.png)

How do we do that? You can [look it up](https://matthew-brett.github.io/teaching/rotation_2d.html) and arrive
at this formula:
<math>
  <mtable>
    <mtr>
      <mrow>
        <msub>
          <mi>x</mi>
          <mn>2</mn>
        </msub>
        <mo>=</mo>
        <msub>
          <mi>x</mi>
          <mn>1</mn>
        </msub>
        <mo>&sdot;</mo>
        <mi>cos</mi>
        <mo>&ApplyFunction;</mo>
        <mi>(</mi>
        <mi>ß</mi>
        <mi>)</mi>
        <mo>-</mo>
        <msub>
          <mi>y</mi>
          <mn>1</mn>
        </msub>
        <mo>&sdot;</mo>
        <mi>sin</mi>
        <mo>&ApplyFunction;</mo>
        <mi>(</mi>
        <mi>ß</mi>
        <mi>)</mi>
      </mrow>
    </mtr>
    <mtr>
      <mrow>
        <msub>
          <mi>y</mi>
          <mn>2</mn>
        </msub>
        <mo>=</mo>
        <msub>
          <mi>x</mi>
          <mn>1</mn>
        </msub>
        <mo>&sdot;</mo>
        <mi>sin</mi>
        <mo>&ApplyFunction;</mo>
        <mi>(</mi>
        <mi>ß</mi>
        <mi>)</mi>
        <mo>+</mo>
        <msub>
          <mi>y</mi>
          <mn>1</mn>
        </msub>
        <mo>&sdot;</mo>
        <mi>cos</mi>
        <mo>&ApplyFunction;</mo>
        <mi>(</mi>
        <mi>ß</mi>
        <mi>)</mi>
      </mrow>
    </mtr>
  </mtable>
</math>

You may ask what this has to do with matrix multiplication. It just turns out that this is exactly the
same as writing it like this:
<math>
  <mrow>
    <mo>[</mo>
    <msub>
      <mi>x</mi>
      <mn>2</mn>
    </msub>
    <mo>,</mo>
    <msub>
      <mi>y</mi>
      <mn>2</mn>
    </msub>
    <mo>]</mo>
    <mo>=</mo>
    <mo>[</mo>
    <msub>
      <mi>x</mi>
      <mn>1</mn>
    </msub>
    <mo>,</mo>
    <msub>
      <mi>y</mi>
      <mn>1</mn>
    </msub>
    <mo>]</mo>
    <mo>&sdot;</mo>
  </mrow>
  <mrow>
    <mo>[</mo>
    <mtable>
      <mtr>
        <mtd columnalign="center">
          <mi>cos</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
        <mtd columnalign="center">
          <mi>sin</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
      </mtr>
      <mtr>
        <mtd columnalign="center">
          <mo>-</mo>
          <mi>sin</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
        <mtd columnalign="center">
          <mi>cos</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
      </mtr>
    </mtable>
    <mo>]</mo>
  </mrow>
</math>

Just check with the rules for multiplying a vector and a matrix above. In other words: By multiplying
our point with the matrix
<math>
  <mrow>
    <mo>[</mo>
    <mtable>
      <mtr>
        <mtd columnalign="center">
          <mi>cos</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
        <mtd columnalign="center">
          <mi>sin</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
      </mtr>
      <mtr>
        <mtd columnalign="center">
          <mo>-</mo>
          <mi>sin</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
        <mtd columnalign="center">
          <mi>cos</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
      </mtr>
    </mtable>
    <mo>]</mo>
  </mrow>
</math>

we get our rotation, so this matrix is our "rotation matrix".

## Translation
One down! We now know how to rotate points. But we also need to know how to use matrix algebra
to move points in certain directions. Say we have a point
<math>
  <mrow>
    <mo>[</mo>
    <msub>
      <mi>x</mi>
      <mn>1</mn>
    </msub>
    <mo>,</mo>
    <msub>
      <mi>y</mi>
      <mn>1</mn>
    </msub>
    <mo>]</mo>
  </mrow>
</math>
that we want to move in a direction and distance defined by the vector
<math>
  <mrow>
    <mo>[</mo>
    <mi>a</mi>
    <mo>,</mo>
    <mi>b</mi>
    <mo>]</mo>
  </mrow>
</math>
so we end up at point
<math>
  <mrow>
    <mo>[</mo>
    <msub>
      <mi>x</mi>
      <mn>2</mn>
    </msub>
    <mo>,</mo>
    <msub>
      <mi>y</mi>
      <mn>2</mn>
    </msub>
    <mo>]</mo>
    <mo>=</mo>
    <mo>[</mo>
    <msub>
      <mi>x</mi>
      <mn>1</mn>
    </msub>
    <mo>+</mo>
    <mi>a</mi>
    <mo>,</mo>
    <msub>
      <mi>y</mi>
      <mn>1</mn>
    </msub>
    <mo>+</mo>
    <mi>b</mi>
    <mo>]</mo>
  </mrow>
</math>
![Moving a point along vector a,b](/assets/images/matrix-operations-rotation-and-translation/translation.png)

Our first reaction is of course to see if there is any way we can create a 2x2 matrix such that
<math>
  <mrow>
    <mrow>
      <mo>[</mo>
      <msub>
        <mi>x</mi>
        <mn>1</mn>
      </msub>
      <mo>,</mo>
      <msub>
        <mi>y</mi>
        <mn>1</mn>
      </msub>
      <mo>]</mo>
    </mrow>
    <mo>&sdot;</mo>
    <mrow>
      <mo>[</mo>
      <mtable>
        <mtr>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>1</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>1</mn>
                <mn>2</mn>
              </mrow>
            </msub>
          </mtd>
        </mtr>
        <mtr>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>1</mn>
              </mrow>
            </msub>
          </mtd>
          <mtd columnalign="center">
            <msub>
              <mi>m</mi>
              <mrow>
                <mn>2</mn>
                <mn>2</mn>
              </mrow>
            </msub>
          </mtd>
        </mtr>
      </mtable>
      <mo>]</mo>
    </mrow>
    <mo>=</mo>
    <mrow>
      <mo>[</mo>
      <msub>
        <mi>x</mi>
        <mn>1</mn>
      </msub>
      <mo>+</mo>
      <mi>a</mi>
      <mo>,</mo>
      <msub>
        <mi>y</mi>
        <mn>1</mn>
      </msub>
      <mo>+</mo>
      <mi>b</mi>
      <mo>]</mo>
    </mrow>
  </mrow>
</math>
But that's just not possible! No matter what you try, an x, y, or a multiplication gets in the way of just
adding a constant to the initial coordinates. What to do?

## Let's Add a Dimension!
The solution may not seem very straightforward, but what if we add another dimension to our 2D vector and
start using 3D vectors and 3x3 matrices? If we always put a 1 at the third coordinate in our vector, then
we have a well-defined constant that our matrix multiplication can make use of:
<math>
  <mrow>
    <mrow>
      <mo>[</mo>
      <msub>
        <mi>x</mi>
        <mn>1</mn>
      </msub>
      <mo>,</mo>
      <msub>
        <mi>y</mi>
        <mn>1</mn>
      </msub>
      <mo>,</mo>
      <mn>1</mn>
      <mo>]</mo>
    </mrow>
    <mo>&sdot;</mo>
    <mrow>
      <mo>[</mo>
      <mtable>
        <mtr>
          <mtd columnalign="center">
            <mn>1</mn>
          </mtd>
          <mtd columnalign="center">
            <mn>0</mn>
          </mtd>
          <mtd columnalign="center">
            <mn>0</mn>
          </mtd>
        </mtr>
        <mtr>
          <mtd columnalign="center">
            <mn>0</mn>
          </mtd>
          <mtd columnalign="center">
            <mn>1</mn>
          </mtd>
          <mtd columnalign="center">
            <mn>0</mn>
          </mtd>
        </mtr>
        <mtr>
          <mtd columnalign="center">
            <mi>a</mi>
          </mtd>
          <mtd columnalign="center">
            <mi>b</mi>
          </mtd>
          <mtd columnalign="center">
            <mn>1</mn>
          </mtd>
        </mtr>
      </mtable>
      <mo>]</mo>
    </mrow>
    <mo>=</mo>
    <mrow>
      <mo>[</mo>
      <mtable>
        <mtr>
          <mtd>
            <msub>
              <mi>x</mi>
              <mn>1</mn>
            </msub>
            <mo>+</mo>
            <mi>a</mi>
            <mo>,</mo>
          </mtd>
          <mtd>
            <msub>
              <mi>y</mi>
              <mn>1</mn>
            </msub>
            <mo>+</mo>
            <mi>b</mi>
            <mo>,</mo>
          </mtd>
          <mtd>
            <mn>1</mn>
          </mtd>
        </mtr>
      </mtable>
      <mo>]</mo>
    </mrow>
    <mo>=</mo>
    <mrow>
      <mo>[</mo>
      <mtable>
        <mtr>
          <mtd>
            <msub>
              <mi>x</mi>
              <mn>2</mn>
            </msub>
            <mo>,</mo>
          </mtd>
          <mtd>
            <msub>
              <mi>y</mi>
              <mn>2</mn>
            </msub>
            <mo>,</mo>
          </mtd>
          <mtd>
            <mn>1</mn>
          </mtd>
        </mtr>
      </mtable>
      <mo>]</mo>
    </mrow>
  </mrow>
</math>
Try to do the math yourself and see if you arrive at the same result.

This is nice! We just add a third component, always set it to 1, and when multiplying our vector
with our translation matrix, we get a new vector with the third component set to 1, which means
we can keep multiplying translation matrices, and it will keep working nicely. That's a nice
property of matrix multiplication: It is easy to compose. So if you want to translate first in one
direction, then another direction, and then a third direction, you can create translation matrices
for each of these translations, multiply them all together, and the result will be one "translation
matrix to rule them all" which represents the combination of all the translations.

So this explains why, in our original 3D example, we need to use 4D vectors and 4x4 matrices.

## Revisiting Rotation
But we're not done yet, because our rotation matrix is still 2x2. To be able to multiply all of
our matrices without considering whether it's a rotation matrix, a translation matrix, or any
combination, we need a 3x3 rotation matrix with the same properties as our 3x3 translation matrix.

Luckily, the solution is very straightforward:
<math>
  <mrow>
    <mo>[</mo>
    <mtable>
      <mtr>
        <mtd columnalign="center">
          <mi>cos</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
        <mtd columnalign="center">
          <mi>sin</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
        <mtd columnalign="center">
          <mn>0</mn>
        </mtd>
      </mtr>
      <mtr>
        <mtd columnalign="center">
          <mo>-</mo>
          <mi>sin</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
        <mtd columnalign="center">
          <mi>cos</mi>
          <mo>&ApplyFunction;</mo>
          <mi>(</mi>
          <mi>ß</mi>
          <mi>)</mi>
        </mtd>
        <mtd columnalign="center">
          <mn>0</mn>
        </mtd>
      </mtr>
      <mtr>
        <mtd columnalign="center">
          <mn>0</mn>
        </mtd>
        <mtd columnalign="center">
          <mn>0</mn>
        </mtd>
        <mtd columnalign="center">
          <mn>1</mn>
        </mtd>
      </mtr>
    </mtable>
    <mo>]</mo>
  </mrow>
</math>
This will make sure that we preserve the 1 in our 3rd vector coordinate, while resulting in the
same results for x and y. Try to do the math if you don't believe me.

## Generalizing From 2D to 3D
All of the above works on 2D models, a 2D world, and a 1D screen. As promised in the beginning of this
post, it really does work almost exactly the same way if we use 3D models, a 3D world, and a 2D screen.

Translation is pretty straightforward - basically just add another component to your vector, but still
keep the last component set to 1. The same principle goes for the matrix.

Rotation is a tiny bit more complex, but not much. When you rotate, what you normally will do is decide
whether to rotate around the X, Y, or Z axis, so instead of one "rotation matrix", you will need to
construct one for each of the three dimensions. Again I will leave this exercise to you.

## Getting it Right Ain't Easy
Now that we know what's going on, it should be easy to get it all right. But even with the help of
[GitHub Copilot](https://copilot.github.com), my first attempt didn't work out quite the way it should:

![example where track is rotated opposite the horizon](/assets/images/matrix-operations-rotation-and-translation/wrong-rotations.gif)

(There are other things wrong with this capture, but we'll get back to that in the next post.)

After fidding with `sin`, `cos`, adding random `-`s etc., it eventually worked:

![now the track is rotated correctly](/assets/images/matrix-operations-rotation-and-translation/correct-rotations.gif)

## Adding some Perspective
We still need to go through the last step, which is "flattening" our 2D world into the one-dimensional screen
(or rather, projecting our 3D world in Stunt Car Racer to your 2D computer screen). For this, we need to apply
another trick. But that'll have to wait for the next blog post.
