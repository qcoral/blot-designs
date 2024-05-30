---
sidebar_position: 1
---

# Draw a cube with blot

This is a basic guide that will walk you through the process of drawing a 3D object (in this case a cube) with Blot. The math we're dealing with here is called **Linear Algebra**, but prior knowledge is NOT necessary!

This document is meant to be viewed alongside the source code.

If you just want to view the final product, go ahead and copy-paste the source code into Blot.

## Setup

As with any blot program, we need to set up our scene with some basic variables first:

```Javascript
// Set up the canvas
const CANVAS_SIZE_X = 200;
const CANVAS_SIZE_Y = 200;
setDocDimensions(CANVAS_SIZE_X, CANVAS_SIZE_Y);

// Controls
let pitch = 0.5; // Rotation around X-axis
let yaw = 0.5;   // Rotation around Y-axis
let roll = 0.5;  // Rotation around Z-axis
let CUBE_SIZE = 20;
```

## Creating a 3D object

### Defining a point

First, let's define a class `Vector` that allows us to define a point in 3D space. It will have coordinates for x, y, and z respectively

```Javascript
class Vector {
  constructor(x = 0, y = 0, z = 0) {
    this.x = x;
    this.y = y;
    this.z = z;
  }
}
```

Next, let's create an `Object` class that can store these edges & vertices. It'll look a little empty for now; we'll add more to this later!

```Javascript
class Object {
  constructor(vertices, edges) {
    this.vertices = vertices;
    this.edges = edges;
  }
}
```

### Defining our cube

When creating 3D shapes, it helps to think of them as a bunch of points in 3D space. For example, a square pyramid has 4 points at its base, and one point at the top. Likewise, a cube is essentially two squares connected to each other, which means there's 8 vertices in total.

Using our `Vector` class from earlier, we can define vertices & edges and pass it through a new instance of our `Object` class. To do this, we'll create a `Cube` class.

```Javascript
// Create a cube using our Object class
const Cube = (size) => {
  const halfSize = size / 2;
  // These are all our points
  const vertices = [
    new Vector(-halfSize, -halfSize, -halfSize),
    new Vector(halfSize, -halfSize, -halfSize),
    new Vector(halfSize, halfSize, -halfSize),
    new Vector(-halfSize, halfSize, -halfSize),
    new Vector(-halfSize, -halfSize, halfSize),
    new Vector(halfSize, -halfSize, halfSize),
    new Vector(halfSize, halfSize, halfSize),
    new Vector(-halfSize, halfSize, halfSize),
  ];

  /* 
    This array tells us which vectors/points are attached to each other, like so:
    7-------6
    /|      /|
  3-------2 |
  | |     | |
  | 4-----|-5
  |/      |/
  0-------1
 */
  const edges = [
    [0, 1], [1, 2], [2, 3], [3, 0], // Front Face
    [4, 5], [5, 6], [6, 7], [7, 4], // Back Face
    [0, 4], [1, 5], [2, 6], [3, 7]  // Connecting edges
  ];

  return new Object(vertices, edges);
};
```

We now have a cube we can work with! Next, we'll see how we can rotate this cube in 3D space


## Rotating our object

### Rotation Matrices

This is where the bulk of the our difficulty lies. Our task here is to take the x, y, and z coordinates from each point in our cube and somehow rotate them in 3D space. This is where we can use something called a **Rotation Matrix**, which is a handy tool that allows us to take a given vector $V$, rotate it along any axes, and give us the position of that new vector.

For example, if we wanted to rotate a point by angle $\theta$ about the x-axis, our rotation matrix would look something like this:

$$
\begin{bmatrix}
x \\
y \\
z
\end{bmatrix}
\cdot
\begin{bmatrix}
1 & 0 & 0 \\
0 & \cos\theta & -\sin\theta \\
0 & \sin\theta & \cos\theta
\end{bmatrix}
=
\begin{bmatrix}
x \\
y\cos\theta-z\sin\theta \\
y\sin\theta+z\cos\theta
\end{bmatrix}
$$

Where the right side represents our new x, y, and z coordinates from top to bottom. If we do this for all three axes of rotation, we should be able to account for any possible rotation.

<details>

<summary>How does this actually work?</summary>

The essence of what we're doing here is called a **linear transformation** 

I highly recommend watching [this](https://www.youtube.com/watch?v=kYB8IZa5AuE) video by 3Blue1Brown to get a better understanding.

Essentially, we are changing the basis vectors of each axis to its trigonmetric function output.

Read more at https://en.wikipedia.org/wiki/Rotation_matrix

</details>

### Implementing it in code

If we add this to our `Vector` class, it should look something like this:

```Javascript
class Vector {
  constructor(x = 0, y = 0, z = 0) {
    this.x = x;
    this.y = y;
    this.z = z;
  }

  rotateX(angle) {
    const cosTheta = Math.cos(angle);
    const sinTheta = Math.sin(angle);
    return new Vector(
      this.x,
      this.y * cosTheta - this.z * sinTheta,
      this.y * sinTheta + this.z * cosTheta
    );
  }

  rotateY(angle) {
    const cosTheta = Math.cos(angle);
    const sinTheta = Math.sin(angle);
    return new Vector(
      this.x * cosTheta + this.z * sinTheta,
      this.y,
      this.z * cosTheta - this.x * sinTheta
    );
  }

  rotateZ(angle) {
    const cosTheta = Math.cos(angle);
    const sinTheta = Math.sin(angle);
    return new Vector(
      this.x * cosTheta - this.y * sinTheta,
      this.x * sinTheta + this.y * cosTheta,
      this.z
    );
  }
}
```

Let's also extend this method to our `Object` class, so that we can rotate not just a single point, but all of them at once:

```Javascript
class Object {
  constructor(vertices, edges) {
    this.vertices = vertices;
    this.edges = edges;
  }

  // Use the mthods in the Vector class to rotate all our points
  rotate(angles) {
    this.vertices = this.vertices.map((vertex) =>
      vertex.rotateX(angles.x).rotateY(angles.y).rotateZ(angles.z)
    );
  }
}
```

Using the power of Rotation Matrices, we now have a set of methods that allows us to rotate a given vector in 3D space. By extending it to our `Object` class, we are able to rotate entire *shapes* however we want!

## Drawing our cube

After transforming all our points, we need to actually draw them. To do this, we'll add two methods to our `Object` class in order 

The first one, `project()`, simply centers our shape; we can do more with it, guide for that is coming soon

The second one, `renderLines()`, iterates through our array of edges & vertices and returns the lines that blot needs to draw. 

Altogether, our `Object` class should look something like this.

```Javascript
class Object {
  constructor(vertices, edges) {
    this.vertices = vertices;
    this.edges = edges;
  }

  // Use the methods in the Vector class to rotate all our points
  rotate(angles) {
    this.vertices = this.vertices.map((vertex) =>
      vertex.rotateX(angles.x).rotateY(angles.y).rotateZ(angles.z)
    );
  }

  project(vertex) {
    // For orthographic projection, we can ignore the z-coordinate for now
    return new Vector(
      vertex.x + CANVAS_SIZE_X / 2, // center points relative to X axis
      vertex.y + CANVAS_SIZE_Y / 2  // center points relative to Y axis
    );
  }

  renderLines() {
    return this.edges.map(([start, end]) => {
      const startVertex = this.project(this.vertices[start]);
      const endVertex = this.project(this.vertices[end]);
      return [
        [startVertex.x, startVertex.y],
        [endVertex.x, endVertex.y],
      ];
    });
  }
}
```

Lastly, we need to actually execute the code we just wrote. Add the following lines to the end:

```Javascript
// Create a new instance of our cube
const cube = Cube(CUBE_SIZE);

// Rotate our cube with the specified yaw, pitch, and roll
cube.rotate({ x: pitch, y: yaw, z: roll });

// Render the object
drawLines(cube.renderLines());
```

If all went well, you should see a cube appear on the screen! If not, check with the source code.

## Next steps

Congrats! You have officially drawn a 3D object with blot. Some things to try:

- Drawing multiple objects
- Creating different shapes
- Create 4D objects instead

Feedback is HIGHLY appreciated!! You can find me on slack @Alex Ren

This guide was partly based on the [3D Game of Life](https://github.com/hackclub/blot/tree/main/art/3DGameOfLife) submission by Ethan Standafer. Many thanks.
