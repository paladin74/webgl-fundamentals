Title: WebGL 3D Basics

This post is a continuation of a series of posts about WebGL.
The first <a href="webgl-fundamentals.html">started with fundamentals</a> and
the previous was about 2d matrices <a href="webgl-2d-matrices.html">about 2D matrices</a>.
If you haven't read those please view them first.

In the last post we went over how 2d matrices worked. We talked
about translation, rotation, scaling, and even projecting from
pixels into clip space can all be done by 1 matrix and some magic
matrix math. To do 3D is only a small step from there.

In our previous 2D examples we had 2D points (x, y) that we multiplied by a 3x3 matrix. To do 3D we need 3D points (x, y, z) and a 4x4 matrix.

Let's take our last example and change it to 3D. We'll use an F again but this time a 3D 'F'.

The first thing we need to do is change the vertex shader to handle 3D. Here's the old shader.

<pre class="prettyprint">
&lt;script id="2d-vertex-shader" type="x-shader/x-vertex"&gt;
attribute vec2 a_position;

uniform mat3 u_matrix;

void main() {
  // Multiply the position by the matrix.
  gl_Position = vec4((u_matrix * vec3(a_position, 1)).xy, 0, 1);
}
&lt;/script&gt;
</pre>

And here's the new one

<pre class="prettyprint">
&lt;script id="2d-vertex-shader" type="x-shader/x-vertex"&gt;
attribute vec4 a_position;

uniform mat4 u_matrix;

void main() {
  // Multiply the position by the matrix.
  gl_Position = u_matrix * a_position;
}
&lt;/script&gt;
</pre>

It got even simpler!

Then we need to provide 3D data.

<pre class="prettyprint">
  ...

  gl.vertexAttribPointer(positionLocation, 3, gl.FLOAT, false, 0, 0);

  ...

// Fill the buffer with the values that define a letter 'F'.
function setGeometry(gl) {
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array([
          // left column
            0,   0,  0,
           30,   0,  0,
            0, 150,  0,
            0, 150,  0,
           30,   0,  0,
           30, 150,  0,

          // top rung
           30,   0,  0,
          100,   0,  0,
           30,  30,  0,
           30,  30,  0,
          100,   0,  0,
          100,  30,  0,

          // middle rung
           30,  60,  0,
           67,  60,  0,
           30,  90,  0,
           30,  90,  0,
           67,  60,  0,
           67,  90,  0]),
      gl.STATIC_DRAW);
}
</pre>

Next we need to change all the matrix functions from 2D to 3D

Here's the 2D (before) versions of makeTranslation, makeRotation and makeScale

<pre class="prettyprint">
function makeTranslation(tx, ty) {
  return [
    1, 0, 0,
    0, 1, 0,
    tx, ty, 1
  ];
}

function makeRotation(angleInRadians) {
  var c = Math.cos(angleInRadians);
  var s = Math.sin(angleInRadians);
  return [
    c,-s, 0,
    s, c, 0,
    0, 0, 1
  ];
}

function makeScale(sx, sy) {
  return [
    sx, 0, 0,
    0, sy, 0,
    0, 0, 1
  ];
}
</pre>

And here's the updated 3D versions.

<pre class="prettyprint">
function makeTranslation(tx, ty, tz) {
  return [
     1,  0,  0,  0,
     0,  1,  0,  0,
     0,  0,  1,  0,
     tx, ty, tz, 1
  ];
}

function makeXRotation(angleInRadians) {
  var c = Math.cos(angleInRadians);
  var s = Math.sin(angleInRadians);

  return [
    1, 0, 0, 0,
    0, c, s, 0,
    0, -s, c, 0,
    0, 0, 0, 1
  ];
};

function makeYRotation(angleInRadians) {
  var c = Math.cos(angleInRadians);
  var s = Math.sin(angleInRadians);

  return [
    c, 0, -s, 0,
    0, 1, 0, 0,
    s, 0, c, 0,
    0, 0, 0, 1
  ];
};

function makeZRotation(angleInRadians) {
  var c = Math.cos(angleInRadians);
  var s = Math.sin(angleInRadians);
  return [
     c, s, 0, 0,
    -s, c, 0, 0,
     0, 0, 1, 0,
     0, 0, 0, 1,
  ];
}

function makeScale(sx, sy, sz) {
  return [
    sx, 0,  0,  0,
    0, sy,  0,  0,
    0,  0, sz,  0,
    0,  0,  0,  1,
  ];
}
</pre>

Notice we now have 3 rotations functions. We only needed one in 2D as we were effectively only rotating around the
Z axis. Now though to do 3D we also want to be able to rotate around the x axis and y axis as well.
You can see from looking at them they are all very similar. If we were to work them out
you'd see them simplify just like before

Z rotation

<div class="webgl_center">
<div>newX = x *  c + y * s;</div>
<div>newY = x * -s + y * c;</div>
</div>

Y rotation

<div class="webgl_center">
<div>newX = x *  c + z * s;</div>
<div>newZ = x * -s + z * c;</div>
</div>

X rotation

<div class="webgl_center">
<div>newY = y *  c + z * s;</div>
<div>newZ = y * -s + z * c;</div>
</div>

which gives you these rotations.

<iframe class="external_diagram" src="resources/axis-diagram.html" width="540" height="240"></iframe>

We also need to update the projection function. Here's the old one

<pre class="prettyprint">
function make2DProjection(width, height) {
  // Note: This matrix flips the Y axis so 0 is at the top.
  return [
    2 / width, 0, 0,
    0, -2 / height, 0,
    -1, 1, 1
  ];
}
</pre>

which converted from pixels to clip space. For our first attempt at expanding it to 3D let's try

<pre class="prettyprint">
function make2DProjection(width, height, depth) {
  // Note: This matrix flips the Y axis so 0 is at the top.
  return [
     2 / width, 0, 0, 0,
     0, -2 / height, 0, 0,
     0, 0, 2 / depth, 0,
    -1, 1, 0, 1,
  ];
}
</pre>

Just like for x and y we needed to convert from pixels to clipspace, for Z we need to do the same thing.
In this case I'm making the Z space pixels as well. I'll pass in some value similar to *width* for the
depth so our space will be 0 to width pixels wide, 0 to height pixels tall, but for depth it will be
-depth / 2 to +depth / 2.

Finally we need to to update the code that computes the matrix.

<pre class="prettyprint">
  // Compute the matrices
  var projectionMatrix =
      make2DProjection(canvas.width, canvas.height, canvas.width);
  var translationMatrix =
      makeTranslation(translation[0], translation[1], translation[2]);
  var rotationXMatrix = makeXRotation(rotation[0]);
  var rotationYMatrix = makeYRotation(rotation[1]);
  var rotationZMatrix = makeZRotation(rotation[2]);
  var scaleMatrix = makeScale(scale[0], scale[1], scale[2]);

  // Multiply the matrices.
  var matrix = matrixMultiply(scaleMatrix, rotationZMatrix);
  matrix = matrixMultiply(matrix, rotationYMatrix);
  matrix = matrixMultiply(matrix, rotationXMatrix);
  matrix = matrixMultiply(matrix, translationMatrix);
  matrix = matrixMultiply(matrix, projectionMatrix);

  // Set the matrix.
  gl.uniformMatrix4fv(matrixLocation, false, matrix);
</pre>

And here's that sample.

<iframe class="webgl_example" src="../webgl/webgl-3d-step1.html" width="400" height="300"></iframe>
<a class="webgl_center" href="../webgl/webgl-3d-step1.html" target="_blank">click here to open in a separate window</a>

The first problem we have is that our geometry is a flat F which makes it hard to see any 3D. To fix that
let's expand the geometry to 3D. Our current F is made of 3 rectangles, 2 triangles each. To make it 3D will require a total of 16
rectangles. That's quite a few to list out here. 16 rectangles * 2 triangles per rectangle * 3 vertices per triangle is 96 vertices.
If you want to see all of them view source on the sample.

We have to draw more vertices so

<pre class="prettyprint">
    // Draw the geometry.
    gl.drawArrays(gl.TRIANGLES, 0, 16 * 6);
</pre>

And here's that version

<iframe class="webgl_example" src="../webgl/webgl-3d-step2.html" width="400" height="300"></iframe>
<a class="webgl_center" href="../webgl/webgl-3d-step2.html" target="_blank">click here to open in a separate window</a>

Moving the sliders it's pretty hard to tell that it's 3D. Let's try coloring each rectangle a different color.
To do this we will add another attribute to our vertex shader and a varying to pass it to the fragment
shader.

Here's the new vertex shader

<pre class="prettyprint">
&lt;script id="2d-vertex-shader" type="x-shader/x-vertex"&gt;
attribute vec4 a_position;
attribute vec4 a_color;

uniform mat4 u_matrix;

varying vec4 v_color;

void main() {
  // Multiply the position by the matrix.
  gl_Position = u_matrix * a_position;

  // Pass the color to the fragment shader.
  v_color = a_color;
}
&lt;/script&gt;
</pre>

And we need use that color in the fragment shader

<pre class="prettyprint">
&lt;script id="2d-vertex-shader" type="x-shader/x-fragment"&gt;
precision mediump float;

// Passed in from the vertex shader.
varying vec4 v_color;

void main() {
   gl_FragColor = v_color;
}
&lt;/script&gt;
</pre>

We need to lookup the location to supply the colors, then setup
another buffer and attribute to give it the colors.

<pre class="prettyprint">
  ...
  var colorLocation = gl.getAttribLocation(program, "a_color");

  ...
  // Create a buffer for colors.
  var buffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
  gl.enableVertexAttribArray(colorLocation);

  // We'll supply RGB as bytes.
  gl.vertexAttribPointer(colorLocation, 3, gl.UNSIGNED_BYTE, true, 0, 0);

  // Set Colors.
  setColors(gl);

  ...
// Fill the buffer with colors for the 'F'.

function setColors(gl) {
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Uint8Array([
          // left column front
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,

          // top rung front
        200,  70, 120,
        200,  70, 120,
        ...
        ...
      gl.STATIC_DRAW);
}
</pre>

Now we get this.

<iframe class="webgl_example" src="../webgl/webgl-3d-step3.html" width="400" height="300"></iframe>
<a class="webgl_center" href="../webgl/webgl-3d-step3.html" target="_blank">click here to open in a separate window</a>

Uh oh, what's that's mess? Well, it turns out all the various parts of that 3D 'F', front, back, sides, etc
get drawn in the order they appear in our geometry. That doesn't give us quite the desired results.

Triangles in WebGL have the concept of front facing and back facing. A front facing triangle has its
vertices go in a clockwise direction. A back facing triangle has its vertices go in a counter clockwise
direction

<img src="resources/triangle-winding.svg" class="webgl_center" width="400" />

WebGL has the ability to draw only forward facing or back facing triangles. We can turn that feature
on with

<pre class="prettyprint">
  gl.enable(gl.CULL_FACE);
</pre>

which we do just once, right at the start of our program. With that feature turned on, WebGL defaults
to "culling" back facing triangles. "Culling" in this case is a fancy word for "not drawing".

Note that as far as WebGL is conserned, whether or not a triangle is considered to be going clockwise or
counter clockwise depends on the vertices of that triangle in clipspace. In other words, WebGL
figures out whether a triangle is front or back AFTER you've applied math to the vertices
in the vertex shader. That means for example a clockwise triangle that is scaled in X by -1 becomes
a counter clockwise triangle. Because we had CULL_FACE disabled we can see both sides of each
triangle. Now that we've turned it on, any time a front facing triangle flips around either because
of scaling or rotation or for whatever reason, WebGL won't draw it. That's a good thing since as your turn something around in 3D
you generally want which ever side the triangles are facing you to be considered front facing.

With CULL_FACE turned on this is what we get

<iframe class="webgl_example" src="../webgl/webgl-3d-step4.html" width="400" height="300"></iframe>
<a class="webgl_center" href="../webgl/webgl-3d-step4.html" target="_blank">click here to open in a separate window</a>

Hey! Where did all the triangles go? It turns out, many of them are facing the wrong way. Rotate it and you'll
see them appear when you look at the other side. Fortunately it's easy to fix. We just look at which ones are
backward and exchange 2 of their vertices. For example if one backward triangle is

<pre class="prettyprint">
           0,   0,   0,
          30,   0,   0,
           0, 150,   0,
</pre>

we just flip the last 2 vertices to make it forward.

<pre class="prettyprint">
           0,   0,   0,
           0, 150,   0,
          30,   0,   0,
</pre>

Going through and fixing all the backward triangles gets us to this

<iframe class="webgl_example" src="../webgl/webgl-3d-step5.html" width="400" height="300"></iframe>
<a class="webgl_center" href="../webgl/webgl-3d-step5.html" target="_blank">click here to open in a separate window</a>

That's closer but there's still one more problem. Even with all the triangles facing in the correct direction and
with the back facing ones being culled we still have places where triangles that should be in back
are being drawn in over triangles that should be in front.

Enter the DEPTH BUFFER.

A Depth buffer, sometimes called a Z-Buffer, is a rectangle of *depth* pixels, one depth pixel for each
color pixel used to make the image. As WebGL draws each color pixel it can also draw a depth pixel.
It does this based on the values we return from the vertex shader for Z. Just like we had to convert
to clip space for X and Y, so to Z is in clip space or (-1 to +1). That value is then converted into
a depth space value (0 to +1). Before WebGL draws a color pixel it will check the corresponding
depth pixel. If the depth value for the pixel it's about to draw is greater than the value of the
corresponding depth pixel then WebGL does not draw the new color pixel. Otherwise it draws both
the new color pixel with the color from your fragment shader AND it draws the depth pixel with
the new depth value. This means, pixels that are behind other pixels won't get drawn.

We can turn on this feature nearly as simply as we turned on culling with

<pre class="prettyprint">
  gl.enable(gl.DEPTH_TEST);
</pre>

We also need to clear the depth buffer back to 1.0 before we start drawing.

<pre class="prettyprint">
  // Draw the scene.
  function drawScene() {
    // Clear the canvas AND the depth buffer.
    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
    ...
</pre>

And now we get

<iframe class="webgl_example" src="../webgl/webgl-3d-step6.html" width="400" height="300"></iframe>
<a class="webgl_center" href="../webgl/webgl-3d-step6.html" target="_blank">click here to open in a separate window</a>

which is 3D!

In the next post I'll go over <a href="webgl-3d-perspective-camera.html">how to make it have perspective</a>.

<div class="webgl_bottombar">
<h3>Why is the attribute vec4 but glVertexAttribPointer size 3</h3>
<p>
For those of you who are detail oriented you might have noticed we defined our 2 attributes as
</p>
<pre class="prettyprint">
attribute vec4 a_position;
attribute vec4 a_color;
</pre>
<p>both of which are 'vec4' but when we tell WebGL how to take data out of our buffers we used</p>
<pre class="prettyprint">
  gl.vertexAttribPointer(positionLocation, 3, gl.FLOAT, false, 0, 0);
  gl.vertexAttribPointer(colorLocation, 3, gl.UNSIGNED_BYTE, true, 0, 0);
</pre>
<p>
That '3' in each of those says only to pull out 3 values per attribute. This works because WebGL
supplies defaults for those you don't supply. The defaults are 0, 0, 0, 1 where x = 0, y = 0, z = 0
and w = 1. This is why in our old 2D vertex shader we had to explicitly supply the 1. We were passing
in x and y and we needed a 1 for z but because the default for z is 0 we had to explicitly supply
a 1. For 3D though, even though we don't supply a 'w' it defaults to 1 which is what we need for
the matrix math to work.
</p>
</div>

