# 🍉 Watermelon Jelly

An interactive, jiggly watermelon slice you can drag and poke, built with plain HTML/CSS/JS
(single file, no build step, no dependencies).

**Live demo: https://githubhjs.github.io/watermelon-jelly/**

## What it does

- A soft-body ("jelly") physics simulation renders a watermelon wedge as a ring of spring-connected
  points — drag it and it stretches toward your finger/cursor; let go and it wobbles back into
  shape. Tap/click without dragging to poke it and watch it bounce.
- Two smaller watermelon slices wobble gently in the background and react to pokes too.
- A tiny synthesized "boing" sound effect (Web Audio oscillator, no audio files) plays on poke —
  toggle with the sound button.
- Fully responsive (desktop + touch), no external assets or libraries.

## Running locally

It's a single static file — just open `index.html` in a browser, or serve the folder with any
static file server, e.g.:

```bash
python3 -m http.server 8080
```

## How the jelly effect works

Each point on the watermelon's outline has a fixed "home" position (where it rests). Every frame,
a spring pulls each point back toward its home, with damping, plus a weaker spring between
neighboring points for cohesion. Dragging applies an extra proportional pull toward the pointer
for every point within range; poking applies a one-off velocity impulse. Nothing exotic — just a
small mass-spring-damper system drawn as a smooth closed curve through the points each frame.
