<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Fireworks with Sound</title>
<style>
  html, body {
    margin: 0;
    padding: 0;
    overflow: hidden;
    background: black;
  }
  canvas {
    display: block;
  }
</style>
</head>
<body>
<canvas id="canvas"></canvas>

<script>
// ====== Canvas Setup ======
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
let cw = window.innerWidth;
let ch = window.innerHeight;
canvas.width = cw;
canvas.height = ch;

window.addEventListener('resize', () => {
  cw = window.innerWidth;
  ch = window.innerHeight;
  canvas.width = cw;
  canvas.height = ch;
});

// ====== Sounds ======
const launchSound = new Audio("https://actions.google.com/sounds/v1/ambiences/rocket_whoosh.ogg");
const explosionSounds = [
  "https://actions.google.com/sounds/v1/explosions/explosion.ogg",
  "https://actions.google.com/sounds/v1/explosions/explosion_large.ogg",
  "https://actions.google.com/sounds/v1/explosions/fireworks_burst.ogg"
];

// ====== Utilities ======
function random(min, max) {
  return Math.random() * (max - min) + min;
}

function calculateDistance(sx, sy, tx, ty) {
  const xDistance = tx - sx;
  const yDistance = ty - sy;
  return Math.sqrt(xDistance * xDistance + yDistance * yDistance);
}

// ====== Classes ======
let fireworks = [];
let particles = [];

class Firework {
  constructor(sx, sy, tx, ty) {
    this.x = sx;
    this.y = sy;
    this.sx = sx;
    this.sy = sy;
    this.tx = tx;
    this.ty = ty;
    this.distanceToTarget = calculateDistance(sx, sy, tx, ty);
    this.distanceTraveled = 0;
    this.coordinates = [];
    this.coordinateCount = 3;
    while (this.coordinateCount--) this.coordinates.push([this.x, this.y]);
    this.angle = Math.atan2(ty - sy, tx - sx);
    this.speed = 3;
    this.acceleration = 1.05;
    this.brightness = random(50, 70);
    this.targetRadius = 2;

    // Play launch sound
    launchSound.currentTime = 0;
    launchSound.play().catch(() => {});
  }

  update(index) {
    this.coordinates.pop();
    this.coordinates.unshift([this.x, this.y]);

    if (this.targetRadius < 8) this.targetRadius += 0.3;
    else this.targetRadius = 1;

    this.speed *= this.acceleration;
    const vx = Math.cos(this.angle) * this.speed;
    const vy = Math.sin(this.angle) * this.speed;
    this.distanceTraveled = calculateDistance(this.sx, this.sy, this.x + vx, this.y + vy);

    if (this.distanceTraveled >= this.distanceToTarget) {
      createParticles(this.tx, this.ty);
      fireworks.splice(index, 1);
    } else {
      this.x += vx;
      this.y += vy;
    }
  }

  draw() {
    ctx.beginPath();
    ctx.moveTo(this.coordinates[this.coordinates.length - 1][0],
               this.coordinates[this.coordinates.length - 1][1]);
    ctx.lineTo(this.x, this.y);
    ctx.strokeStyle = `hsl(${random(0, 360)}, 100%, ${this.brightness}%)`;
    ctx.stroke();

    ctx.beginPath();
    ctx.arc(this.tx, this.ty, this.targetRadius, 0, Math.PI * 2);
    ctx.stroke();
  }
}

class Particle {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.coordinates = [];
    this.coordinateCount = 5;
    while (this.coordinateCount--) this.coordinates.push([this.x, this.y]);
    this.angle = random(0, Math.PI * 2);
    this.speed = random(1, 10);
    this.friction = 0.95;
    this.gravity = 1;
    this.hue = random(0, 360);
    this.brightness = random(50, 80);
    this.alpha = 1;
    this.decay = random(0.015, 0.03);
  }

  update(index) {
    this.coordinates.pop();
    this.coordinates.unshift([this.x, this.y]);
    this.speed *= this.friction;
    this.x += Math.cos(this.angle) * this.speed;
    this.y += Math.sin(this.angle) * this.speed + this.gravity;
    this.alpha -= this.decay;

    if (this.alpha <= this.decay) particles.splice(index, 1);
  }

  draw() {
    ctx.beginPath();
    ctx.moveTo(this.coordinates[this.coordinates.length - 1][0],
               this.coordinates[this.coordinates.length - 1][1]);
    ctx.lineTo(this.x, this.y);
    ctx.strokeStyle = `hsla(${this.hue}, 100%, ${this.brightness}%, ${this.alpha})`;
    ctx.stroke();
  }
}

// ====== Particle Explosion ======
function createParticles(x, y) {
  let count = 100;
  while (count--) particles.push(new Particle(x, y));

  // Play random explosion sound
  const boom = new Audio(explosionSounds[Math.floor(Math.random() * explosionSounds.length)]);
  boom.volume = 0.3;
  boom.play().catch(() => {});
}

// ====== Main Animation Loop ======
function loop() {
  requestAnimationFrame(loop);
  ctx.globalCompositeOperation = 'destination-out';
  ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
  ctx.fillRect(0, 0, cw, ch);
  ctx.globalCompositeOperation = 'lighter';

  fireworks.forEach((firework, index) => {
    firework.draw();
    firework.update(index);
  });

  particles.forEach((particle, index) => {
    particle.draw();
    particle.update(index);
  });

  if (Math.random() < 0.05) {
    fireworks.push(new Firework(cw / 2, ch, random(0, cw), random(0, ch / 2)));
  }
}

// ====== Click to Launch ======
canvas.addEventListener('click', (e) => {
  fireworks.push(new Firework(cw / 2, ch, e.clientX, e.clientY));
});

loop();
</script>
</body>
</html>
