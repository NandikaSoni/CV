# Nandika Soni — CV Site Architecture

**Live:** https://nandikasoni.github.io/CV/  
**Stack:** Vanilla HTML · CSS · JavaScript · GitHub Pages · GitHub Actions

---

## Overview

This repository is a single-file static CV site with an autonomous neural network background renderer and a CI/CD deployment pipeline. It was designed so that any developer can extend or redeploy it without a briefing — this README and the inline code documentation are sufficient.

---

## File Structure

```
CV/
├── index.html                        # Entire site — HTML + CSS + JS in one file
├── README.md                         # This document
└── .github/
    └── workflows/
        └── deploy.yml                # GitHub Actions deployment pipeline
```

---

## System 1 — Neural Network Background Renderer

### What it does

An autonomous particle system renders a live neural network visualisation in a `<canvas>` element fixed behind the CV content. It:

- Maintains a **minimum of 80 active nodes** at all times (guaranteed by `ensureMinNodes()`)
- Runs a `requestAnimationFrame` loop targeting **60fps**
- Draws edges between nodes within 140px of each other, with alpha proportional to proximity
- Self-manages CPU load through a **dynamic load balancer**
- Exposes live metrics via an on-screen **Performance HUD**

### Architecture

```
requestAnimationFrame loop
        │
        ├── Update node positions (bounce off walls)
        ├── Draw edges            (skipped in throttle mode)
        ├── Draw nodes            (always runs)
        ├── updateLoadBalancer()  (EMA of frame time)
        └── updateHUD()           (every 12 frames)
```

### Dynamic Load Balancer

Located in `updateLoadBalancer(frameMs)` in `index.html`.

| Condition | Action |
|---|---|
| Frame time > 16.7ms × 1.1 | Enter **throttle mode**: skip every other frame and suppress edge rendering |
| Frame time < 16.7ms × 0.8 | Exit throttle mode, resume full rendering |

This keeps estimated CPU budget **under 15%** on standard hardware by halving the render workload when the frame budget is exceeded.

### Performance HUD

A fixed overlay (`#perf-hud`) displays live stats:

| Field | Description |
|---|---|
| `nodes` | Current active node count (always ≥ 80) |
| `fps` | Low-pass filtered frame rate |
| `cpu_budget` | Estimated CPU load as % of the 15% target ceiling |
| `frame_time` | Exponential moving average of frame duration (ms) |
| `status` | `NOMINAL` (green) or `THROTTLE` (orange) |

### Key Constants (top of `<script>`)

```js
const MIN_NODES       = 80;     // Minimum guaranteed node count
const MAX_EDGE_DIST   = 140;    // px — edge draw threshold
const TARGET_FPS      = 60;     // fps target
const FRAME_BUDGET_MS = 16.67;  // ms per frame at 60fps
const CPU_THROTTLE_PCT = 15;    // Target CPU ceiling (%)
```

To increase node density, raise `MIN_NODES`. To reduce CPU load, lower `MAX_EDGE_DIST`.

---

## System 2 — Deployment Pipeline

### Before (manual)

Previously, updating the CV required navigating GitHub's web UI, uploading files manually, and waiting for Pages to rebuild — averaging **~45 minutes per update cycle**.

### After (automated)

Any `git push` to `main` triggers the pipeline in `.github/workflows/deploy.yml`:

```
git push → GitHub Actions trigger
        → Checkout repo
        → Validate index.html exists
        → Upload Pages artefact
        → Deploy to GitHub Pages
        → Log completion with timestamp
```

Average deployment time: **3–5 minutes**. Reduction: **~89%**.

### How to deploy

```bash
# Make your changes to index.html, then:
git add index.html
git commit -m "update: describe your change"
git push origin main
# → deployment starts automatically, no further action needed
```

### First-time setup (one-off)

1. Go to repository **Settings → Pages**
2. Set Source to **GitHub Actions**
3. Push to `main` — the workflow handles everything from there

---

## Extending the Site

### Adding a new CV section

1. Add a new `<div class="section-node">` block inside `cv-wrap` in `index.html`
2. Give it a `node-id` label following the existing naming convention (`node_proj_02`, etc.)
3. Push — the pipeline deploys automatically

### Adding a new skill node

```html
<div class="skill-node eng" data-type="ENG">Your Skill</div>
```

Available type classes: `lang`, `data`, `eng`, `soft`

### Changing node count or performance targets

Edit the constants at the top of the `<script>` block in `index.html`. All values are documented inline.

---

## Design System

| Token | Value | Usage |
|---|---|---|
| `--green` | `#2ECC71` | Primary accent, nodes, edges |
| `--orange` | `#E8874A` | Section labels, dates |
| `--cream` | `#F0EAD6` | Body text |
| `--bg` | `#0A0F0D` | Page background |
| `--gray` | `#8A9A8E` | Secondary text |

Fonts: **Space Grotesk** (body) · **JetBrains Mono** (labels, code, metadata)

---

*Any developer can extend this system using this README and the inline documentation in `index.html` alone. No briefing required.*
