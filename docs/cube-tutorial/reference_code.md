---
sidebar_position: 2
---

# Source code

This is what the final program should look like

```Javascript

/*
  This is an example program that renders a controllable 3D cube with blot. 
  To be used with the corresponding guide
*/

// Set up the canvas
const CANVAS_SIZE_X = 200;
const CANVAS_SIZE_Y = 200;
setDocDimensions(CANVAS_SIZE_X, CANVAS_SIZE_Y);

// Controls
let pitch = 0.5; // Rotation around X-axis
let yaw = 0.5;   // Rotation around Y-axis
let roll = 0.5;  // Rotation around Z-axis
let CUBE_SIZE = 20;

// This class allows us to define & rotate a vector in 3D space
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

/* 
  This class takes a shape defined by its vertices & edges.
  It provides the methods needed to actually draw the shape
  on the canvas.
*/

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

// Create a cube using our Object class
const Cube = (size) => {
  const halfSize = size / 2;
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

// Create a new instance of our cube
const cube = Cube(CUBE_SIZE);

// Rotate our cube with the specified yaw, pitch, and roll
cube.rotate({ x: pitch, y: yaw, z: roll });

// Render the object
drawLines(cube.renderLines());

```
