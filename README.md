# Koch-Curve-Particles

This code is an interactive **Koch Snowflake** fractal generator using **p5.js**. It displays a series of iterated line segments forming a Koch fractal, and when a user touches or clicks on the fractal, colorful particles are emitted from the touched segment.

The script includes:
- The recursive **Koch fractal** generation algorithm.
- **Interactive touch detection** to emit particles.
- **Particle system** for visual effects.
- **Responsive resizing** for a dynamic experience.

---

## **HTML & Head Section**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.min.js"></script>
    <style>
      html, body {
        margin: 0;
        padding: 0;
        overflow: hidden;
      }
      canvas {
        display: block;
      }
    </style>
</head>
```
- **Includes p5.js** from a CDN.
- **Removes default margins & padding** for full-screen rendering.
- **Ensures canvas fills the entire viewport** and hides scrollbars.


## **Main Variables**
```js
let segments = [];
let particles = [];
let iterations = 5;
```
- `segments`: Stores the fractal's line segments.
- `particles`: Stores the animated particles.
- `iterations`: Determines how many times the fractal is subdivided.


## **Setup Function**
```js
function setup() {
    createCanvas(windowWidth, windowHeight);
    let start = createVector(width * 0.2, height / 2);
    let end = createVector(width * 0.8, height / 2);
    segments = [new KochLine(start, end)];

    for (let i = 0; i < iterations; i++) {
      generate();
    }
}
```
- **Creates the canvas** that fills the screen.
- **Defines the initial Koch line** from left to right.
- **Performs recursive subdivision** using `generate()`.


## **Draw Function**
```js
function draw() {
    background(0);
    for (let segment of segments) {
        segment.show();
    }
    
    for (let i = particles.length - 1; i >= 0; i--) {
        particles[i].update();
        particles[i].show();
        if (particles[i].finished()) {
            particles.splice(i, 1);
        }
    }
}
```
- **Clears the background** each frame (`background(0)`).
- **Draws all Koch fractal segments**.
- **Updates & displays particles**, removing finished ones.


## **Window Resize Handling**
```js
function windowResized() {
    setup();
}
```
- Recalculates everything when the window is resized.
- Calls `setup()` again to adjust dimensions dynamically.


## **Fractal Generation**
```js
function generate() {
    let next = [];
    for (let segment of segments) {
        let [a, b, c, d, e] = segment.kochPoints();
        next.push(new KochLine(a, b));
        next.push(new KochLine(b, c));
        next.push(new KochLine(c, d));
        next.push(new KochLine(d, e));
    }
    segments = next;
}
```
- **Subdivides each existing segment into 4 new segments**.
- Uses `kochPoints()` to calculate the new points for Koch fractal growth.


## **User Interaction**
```js
function touchStarted() {
    for (let segment of segments) {
        if (segment.isTouched(mouseX, mouseY)) {
            segment.emitParticles();
        }
    }
    return false;
}
```
- Checks if a segment is touched (or clicked).
- If touched, it emits particles from that segment.


## **KochLine Class**
```js
class KochLine {
    constructor(a, b) {
        this.start = a.copy();
        this.end = b.copy();
        this.color = color(random(100, 255), random(100, 255), random(100, 255));
    }

    show() {
        stroke(this.color);
        strokeWeight(9);
        line(this.start.x, this.start.y, this.end.x, this.end.y);
    }

    isTouched(x, y) {
        let d1 = dist(x, y, this.start.x, this.start.y);
        let d2 = dist(x, y, this.end.x, this.end.y);
        let lineLen = dist(this.start.x, this.start.y, this.end.x, this.end.y);
        return abs(d1 + d2 - lineLen) < 5;
    }

    emitParticles() {
        let mid = p5.Vector.lerp(this.start, this.end, random());
        for (let i = 0; i < 20; i++) {
            particles.push(new Particle(mid.x, mid.y));
        }
    }

    kochPoints() {
        let a = this.start.copy();
        let e = this.end.copy();
        let v = p5.Vector.sub(this.end, this.start);
        v.mult(1 / 3);
        let b = p5.Vector.add(a, v);
        let d = p5.Vector.add(b, v);
        v.rotate(-PI / 3);
        let c = p5.Vector.add(b, v);
        return [a, b, c, d, e];
    }
}
```
### **What This Class Does**
1. **Defines a line segment** with two endpoints `start` and `end`.
2. **Draws the segment** in a random color.
3. **Detects user touch/click** to check if the segment is being interacted with.
4. **Emits particles** from a random point on the segment.
5. **Calculates the next iteration's fractal points** to form a Koch curve.


## **Particle Class**
```js
class Particle {
    constructor(x, y) {
        this.pos = createVector(x, y);
        this.vel = p5.Vector.random2D().mult(random(1, 3));
        this.acc = createVector(0, 0.05);
        this.lifetime = 255;
        this.color = color(random(255), random(255), random(255));
    }

    update() {
        this.vel.add(this.acc);
        this.pos.add(this.vel);
        this.lifetime -= 5;
    }

    show() {
        noStroke();
        fill(this.color, this.lifetime);
        ellipse(this.pos.x, this.pos.y, 1);
    }

    finished() {
        return this.lifetime < 0;
    }
}
```
### **What This Class Does**
1. **Defines a particle** with position, velocity, acceleration, lifetime, and color.
2. **Moves the particle** each frame using acceleration.
3. **Fades the particle** over time (`lifetime` decreases).
4. **Draws the particle** as a tiny fading circle.
5. **Removes dead particles** when their lifetime reaches zero.


## **Summarizing**
1. **Generates a Koch fractal** with `iterations = 5`.
2. **Divides each line segment into smaller parts recursively**.
3. **Displays the fractal** with colored strokes.
4. **Detects user interaction**, emitting colorful particles when touched.
5. **Animates particles** as they fade and disappear.
6. **Supports resizing dynamically** for a full-screen interactive experience.
