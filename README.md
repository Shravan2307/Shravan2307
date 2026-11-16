
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Shravan Patel | Cat Playground</title>

<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;700&display=swap" rel="stylesheet">

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg: #08070d;
  --purple: #a78bfa;
  --text: #f5f0ff;
  --muted: #9990b5;
}

body {
  background:
    radial-gradient(ellipse at top, #20133b 0%, #08070d 60%);
  color: var(--text);
  font-family: "Space Grotesk", sans-serif;
  min-height: 100vh;
  overflow-x: hidden;
}

main {
  max-width: 1000px;
  margin: auto;
  padding: 80px 24px;
}

.hero {
  animation: reveal 1.5s ease both;
}

h1 {
  font-size: clamp(3.5rem, 12vw, 9rem);
  line-height: .9;
  letter-spacing: -8px;
  font-weight: 700;
  background: linear-gradient(
    120deg,
    #ffffff,
    #d8b4fe,
    #8b5cf6
  );
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 0 30px #8b5cf633);
}

.subtitle {
  margin-top: 22px;
  color: var(--muted);
  font-size: 1.1rem;
}

.badges {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  margin-top: 25px;
}

.badge {
  padding: 8px 16px;
  border: 1px solid #33264d;
  border-radius: 50px;
  background: #a78bfa0d;
  font-size: .85rem;
  color: #d8b4fe;
}

a {
  color: #c4b5fd;
  text-decoration: none;
}

.playground {
  margin-top: 60px;
  border: 1px solid #33264d;
  border-radius: 24px;
  overflow: hidden;
  background: #0c0914;
  box-shadow: 0 0 70px #8b5cf610;
  animation: reveal 2s ease both;
}

canvas {
  display: block;
  width: 100%;
  height: 430px;
  cursor: crosshair;
  touch-action: none;
}

.hint {
  text-align: center;
  color: #77708d;
  margin-top: 18px;
  font-size: .85rem;
}

@keyframes reveal {
  from {
    opacity: 0;
    transform: translateY(35px);
    filter: blur(12px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
    filter: blur(0);
  }
}

@media(max-width:600px) {
  main {
    padding: 60px 16px;
  }

  h1 {
    letter-spacing: -3px;
  }

  canvas {
    height: 350px;
  }
}

@media(prefers-reduced-motion:reduce) {
  *, *::before, *::after {
    animation: none !important;
  }
}
</style>
</head>

<body>

<main>

  <section class="hero">

    <h1>SHRAVAN<br>PATEL</h1>

    <p class="subtitle">
      building cool things, one commit at a time.
    </p>

    <div class="badges">
      <span class="badge">GSSoC'26 · #536</span>
      <span class="badge">TOP 2%</span>
      <span class="badge">42 PRs</span>
      <span class="badge">
        <a href="https://shravan-portfolio-one.vercel.app/">
          Portfolio ↗
        </a>
      </span>
    </div>

  </section>

  <section class="playground">
    <canvas id="game"></canvas>
  </section>

  <p class="hint">
    move your cursor · play with the cat · click the glowing orb ✦
  </p>

</main>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let W, H;
let time = 0;

const mouse = {
  x: 0,
  y: 0,
  active: false
};

const orb = {
  x: 0,
  y: 0,
  tx: 0,
  ty: 0
};

const cat = {
  x: 0,
  y: 0,
  face: 1,
  jump: 0,
  happy: 0
};

const hero = {
  x: 0
};

const particles = [];

function resize() {
  const dpr = Math.min(window.devicePixelRatio || 1, 2);

  W = canvas.clientWidth;
  H = canvas.clientHeight;

  canvas.width = W * dpr;
  canvas.height = H * dpr;

  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);

  hero.x = W * 0.25;

  cat.x ||= W * 0.65;
  cat.y = H * 0.76;

  orb.x ||= W * 0.7;
  orb.y ||= H * 0.6;

  orb.tx ||= W * 0.7;
  orb.ty ||= H * 0.6;
}

window.addEventListener("resize", resize);

function pointerMove(e) {
  const rect = canvas.getBoundingClientRect();

  mouse.x = e.clientX - rect.left;
  mouse.y = e.clientY - rect.top;
  mouse.active = true;

  orb.tx = mouse.x;
  orb.ty = mouse.y;
}

canvas.addEventListener("pointermove", pointerMove);

canvas.addEventListener("pointerdown", e => {
  pointerMove(e);
  burst(orb.x, orb.y);
  cat.happy = 1.5;
});

function burst(x, y) {
  for (let i = 0; i < 25; i++) {
    const angle = Math.random() * Math.PI * 2;
    const speed = Math.random() * 3 + 1;

    particles.push({
      x, y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      life: 1
    });
  }
}

function glow(x, y, radius, color) {
  const g = ctx.createRadialGradient(
    x, y, 0,
    x, y, radius
  );

  g.addColorStop(0, color);
  g.addColorStop(1, "transparent");

  ctx.fillStyle = g;
  ctx.beginPath();
  ctx.arc(x, y, radius, 0, Math.PI * 2);
  ctx.fill();
}

function ellipse(x, y, rx, ry, color) {
  ctx.fillStyle = color;

  ctx.beginPath();
  ctx.ellipse(x, y, rx, ry, 0, 0, Math.PI * 2);
  ctx.fill();
}

function drawBackground() {
  ctx.clearRect(0, 0, W, H);

  // Background stars
  for (let i = 0; i < 50; i++) {
    const x = (i * 97) % W;
    const y = (i * 53) % (H * 0.6);

    ctx.fillStyle = "#bba5ff";
    ctx.globalAlpha = 0.3 + Math.sin(time + i) * 0.2;

    ctx.fillRect(x, y, 2, 2);
  }

  ctx.globalAlpha = 1;

  // Triangle constellation
  const points = [];

  for (let row = 0; row < 4; row++) {
    for (let col = 0; col <= row; col++) {
      points.push({
        x: W / 2 + (col - row / 2) * 45,
        y: 35 + row * 45
      });
    }
  }

  ctx.strokeStyle = "#a78bfa33";
  ctx.lineWidth = 1;

  ctx.beginPath();

  points.forEach((p, i) => {
    if (i < points.length - 1) {
      ctx.moveTo(p.x, p.y);
      ctx.lineTo(points[i + 1].x, points[i + 1].y);
    }
  });

  ctx.stroke();

  points.forEach(p => {
    glow(p.x, p.y, 12, "#a78bfa");
  });

  // Floor
  ctx.strokeStyle = "#a78bfa33";
  ctx.beginPath();
  ctx.moveTo(0, H * 0.78);
  ctx.lineTo(W, H * 0.78);
  ctx.stroke();
}

function drawHero() {
  const x = hero.x;
  const y = H * 0.76;

  const lookX = (orb.x - x) / 100;

  ctx.save();
  ctx.translate(x, y);

  // Shadow
  ellipse(0, 0, 45, 12, "#201534");

  // Body hoodie
  ellipse(0, -42, 25, 32, "#7c3aed");

  // Head
  ellipse(0, -90, 25, 27, "#f4cdb9");

  // Hair
  ellipse(0, -108, 26, 15, "#171020");

  ctx.fillStyle = "#171020";

  ctx.beginPath();
  ctx.moveTo(-25, -100);
  ctx.lineTo(-20, -125);
  ctx.lineTo(-5, -115);
  ctx.lineTo(10, -128);
  ctx.lineTo(25, -105);
  ctx.fill();

  // Eyes follow the orb
  ellipse(-9 + lookX * 2, -88, 4, 5, "#201534");
  ellipse(9 + lookX * 2, -88, 4, 5, "#201534");

  // Headphones
  ctx.strokeStyle = "#c4b5fd";
  ctx.lineWidth = 4;

  ctx.beginPath();
  ctx.arc(0, -90, 29, Math.PI, 0);
  ctx.stroke();

  ellipse(-28, -88, 5, 10, "#a78bfa");
  ellipse(28, -88, 5, 10, "#a78bfa");

  // Arm reaching toward cat
  ctx.strokeStyle = "#7c3aed";
  ctx.lineWidth = 9;
  ctx.lineCap = "round";

  ctx.beginPath();
  ctx.moveTo(15, -55);
  ctx.lineTo(40, -25);
  ctx.stroke();

  ellipse(40, -25, 6, 6, "#f4cdb9");

  ctx.restore();
}

function drawCat() {
  const x = cat.x;
  const y = cat.y - cat.jump;

  ctx.save();
  ctx.translate(x, y);
  ctx.scale(cat.face, 1);

  // Tail
  ctx.strokeStyle = "#e9d5ff";
  ctx.lineWidth = 7;
  ctx.lineCap = "round";

  ctx.beginPath();
  ctx.moveTo(-20, -20);
  ctx.quadraticCurveTo(-45, -50, -25, -60);
  ctx.stroke();

  // Body
  ellipse(0, -20, 25, 17, "#e9d5ff");

  // Head
  ellipse(25, -42, 17, 16, "#e9d5ff");

  // Ears
  ctx.fillStyle = "#e9d5ff";

  ctx.beginPath();
  ctx.moveTo(12, -50);
  ctx.lineTo(13, -72);
  ctx.lineTo(27, -55);
  ctx.fill();

  ctx.beginPath();
  ctx.moveTo(30, -55);
  ctx.lineTo(43, -72);
  ctx.lineTo(42, -48);
  ctx.fill();

  // Eyes
  ellipse(21, -42, 3, 5, "#24133d");
  ellipse(32, -42, 3, 5, "#24133d");

  // Nose
  ellipse(27, -35, 3, 2, "#ff9ec4");

  // Legs
  ellipse(-12, -5, 7, 10, "#e9d5ff");
  ellipse(12, -5, 7, 10, "#e9d5ff");

  ctx.restore();
}

function drawOrb() {
  glow(orb.x, orb.y, 45, "#8b5cf6");
  glow(orb.x, orb.y, 15, "#c4b5fd");

  ellipse(orb.x, orb.y, 5, 5, "#ffffff");
}

function update(dt) {
  time += dt;

  // Orb eases toward cursor
  orb.x += (orb.tx - orb.x) * 0.08;
  orb.y += (orb.ty - orb.y) * 0.08;

  // Cat chases orb
  const dx = orb.x - cat.x;
  const distance = Math.abs(dx);

  if (distance > 10) {
    cat.face = dx > 0 ? 1 : -1;

    cat.x += cat.face * Math.min(distance, 180 * dt);
  }

  // Cat jump near the orb
  if (distance < 35) {
    cat.jump = Math.sin(time * 10) * 10;
    cat.happy = 0.5;
  } else {
    cat.jump *= 0.9;
  }

  cat.happy = Math.max(0, cat.happy - dt);

  // Particle physics
  particles.forEach(p => {
    p.x += p.vx;
    p.y += p.vy;
    p.life -= 0.02;
  });

  for (let i = particles.length - 1; i >= 0; i--) {
    if (particles[i].life <= 0) {
      particles.splice(i, 1);
    }
  }
}

function drawParticles() {
  particles.forEach(p => {
    ctx.globalAlpha = p.life;
    glow(p.x, p.y, 5, "#c4b5fd");
  });

  ctx.globalAlpha = 1;
}

let last = 0;

function animate(timestamp) {
  const dt = Math.min((timestamp - last) / 1000, 0.04);
  last = timestamp;

  update(dt);
  drawBackground();
  drawHero();
  drawCat();
  drawOrb();
  drawParticles();

  requestAnimationFrame(animate);
}

resize();

orb.x = W * 0.7;
orb.y = H * 0.6;
orb.tx = orb.x;
orb.ty = orb.y;

cat.x = W * 0.65;

requestAnimationFrame(animate);
</script>

</body>
</html>
