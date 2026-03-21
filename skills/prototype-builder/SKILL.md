---
name: prototype-builder
description: Generate self-contained clickable HTML prototypes for websites and apps. No external APIs needed — Claude generates the actual HTML/CSS/JS screens and wires them together with navigation. Use when the user wants to prototype a website, app flow, or UI. Triggers: "prototype", "clickable mockup", "clickable prototype", "ui flow", "figma prototype", "website designs".
---

# Prototype Builder Skill

Generate a fully clickable, self-contained HTML prototype — no Figma, no image APIs. Each screen is real HTML/CSS, wired together with JS navigation. Output is a single `prototype.html` file.

## When to use this

- Client wants to click through a flow before anything is built
- Validating UX before writing production code
- Presenting a design concept quickly
- Briefr: showing 5 website design variants to a client

## Workflow

1. **Map the screens** — list every distinct page or state in the flow
2. **Define connections** — which element on which screen goes where
3. **Generate each screen** — full HTML/CSS, matching the brand
4. **Wire navigation** — shared JS `showScreen()` function
5. **Add hotspots** — invisible click overlays on buttons/CTAs
6. **Write `prototype.html`** — single self-contained file, no external dependencies

---

## File structure

Everything in one file:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Project] — Prototype</title>
  <style>
    /* === Global reset === */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; font-family: system-ui, -apple-system, sans-serif; }

    /* === Prototype chrome === */
    #proto-nav {
      position: fixed; top: 0; left: 0; right: 0; z-index: 9999;
      background: #18181b; border-bottom: 1px solid #27272a;
      display: flex; align-items: center; gap: 6px; padding: 8px 16px;
      font-size: 12px;
    }
    #proto-nav .label { color: #71717a; margin-right: 8px; font-weight: 500; letter-spacing: .05em; text-transform: uppercase; }
    #proto-nav button {
      padding: 4px 12px; border-radius: 5px; border: 1px solid #3f3f46;
      background: #27272a; color: #a1a1aa; cursor: pointer; font-size: 12px;
      transition: all 0.1s;
    }
    #proto-nav button:hover { background: #3f3f46; color: #fff; }
    #proto-nav button.active { background: #2563eb; border-color: #2563eb; color: #fff; }
    #proto-content { padding-top: 41px; }

    /* === Screens === */
    .screen { display: none; min-height: calc(100vh - 41px); }
    .screen.active { display: block; }

    /* === Hotspots (invisible click areas over designed elements) === */
    .hs-wrap { position: relative; }
    .hotspot {
      position: absolute; cursor: pointer; border-radius: 4px;
      transition: background 0.15s;
      /* hint mode: add class 'show-hints' to body to reveal hotspots */
    }
    body.show-hints .hotspot {
      background: rgba(37, 99, 235, 0.15);
      outline: 2px dashed rgba(37, 99, 235, 0.5);
    }
    body.show-hints .hotspot::after {
      content: attr(data-label);
      position: absolute; top: 2px; left: 4px;
      font-size: 10px; color: #2563eb; font-weight: 600; white-space: nowrap;
    }

    /* === [Screen-specific styles go here] === */

  </style>
</head>
<body>

<!-- Prototype navigation bar -->
<nav id="proto-nav">
  <span class="label">Prototype</span>
  <button class="active" onclick="go(0, this)">Screen name</button>
  <button onclick="go(1, this)">Screen name</button>
  <!-- add more per screen -->
  <button style="margin-left:auto;background:#1c1c1e;border-color:#3f3f46"
          onclick="document.body.classList.toggle('show-hints')">
    Toggle hotspots
  </button>
</nav>

<div id="proto-content">

  <!-- ── SCREEN 0 ──────────────────────────────── -->
  <div class="screen active" id="s0">
    <div class="hs-wrap" style="position:relative">
      <!-- Full HTML/CSS design for this screen -->

      <!-- Hotspot example: CTA button area → goes to screen 1 -->
      <div class="hotspot" data-label="→ Next screen"
           style="top: 420px; left: 50%; transform: translateX(-50%); width: 180px; height: 48px;"
           onclick="go(1, document.querySelectorAll('#proto-nav button')[1])"></div>
    </div>
  </div>

  <!-- ── SCREEN 1 ──────────────────────────────── -->
  <div class="screen" id="s1">
    <div class="hs-wrap" style="position:relative">
      <!-- Full HTML/CSS design for this screen -->
    </div>
  </div>

</div><!-- /proto-content -->

<script>
function go(index, btn) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'))
  document.querySelectorAll('#proto-nav button:not(:last-child)').forEach(b => b.classList.remove('active'))
  document.getElementById('s' + index).classList.add('active')
  if (btn) btn.classList.add('active')
  window.scrollTo(0, 0)
}
// Keyboard navigation: ← →
document.addEventListener('keydown', e => {
  const screens = document.querySelectorAll('.screen')
  const current = [...screens].findIndex(s => s.classList.contains('active'))
  if (e.key === 'ArrowRight' && current < screens.length - 1) {
    const btn = document.querySelectorAll('#proto-nav button')[current + 1]
    go(current + 1, btn)
  }
  if (e.key === 'ArrowLeft' && current > 0) {
    const btn = document.querySelectorAll('#proto-nav button')[current - 1]
    go(current - 1, btn)
  }
})
</script>
</body>
</html>
```

---

## Screen design guidelines

Each screen is full production-quality HTML/CSS:

- **No external fonts** — use `system-ui` or embed a Google Font with `@import`
- **No external images** — use CSS gradients, inline SVGs, or `https://picsum.photos/[w]/[h]` placeholders
- **No JS frameworks** — plain HTML/CSS only inside each screen
- **Full viewport height** — each screen should fill the browser window at minimum
- **Real copy** — no lorem ipsum, use plausible content matching the brand

### Hotspot positioning

Hotspots are absolutely positioned over the designed element they represent:

```html
<div class="hs-wrap" style="position:relative">
  <!-- your designed button at e.g. y=420, center of page -->
  <button style="position:absolute;top:420px;left:50%;transform:translateX(-50%);
                 padding:14px 32px;background:#2563eb;color:#fff;border:none;
                 border-radius:8px;font-size:16px;cursor:pointer"
          onclick="go(1, ...)">Get started</button>
</div>
```

When the screen uses a normal document flow (not absolute positioning), place the hotspot as a sibling overlay using `position: absolute` on the `.hs-wrap` parent. Use `Toggle hotspots` button to visually debug clickable areas.

---

## Briefr-specific screen list

For Briefr (5 website design concept flow):

| # | Screen | Key interaction |
|---|--------|----------------|
| 0 | Landing | CTA → Questionnaire |
| 1 | Questionnaire (step 1–4) | Next → next step → Designs |
| 2 | Design results (5 cards) | Select card → Quote |
| 3 | Quote / Stripe offer | CTA → confirmation |

For Briefr design variants, generate 5 separate screens (one per style) and let the client toggle between them using the nav bar — simulating the "pick your favourite" step.

---

## Quality checklist

Before delivering the prototype:

- [ ] All nav buttons work
- [ ] Arrow key navigation works
- [ ] "Toggle hotspots" reveals all click areas
- [ ] `window.scrollTo(0,0)` fires on each screen transition
- [ ] No broken external resources (fonts, images)
- [ ] Opens correctly from `file://` (no server needed)
- [ ] Tested in Chrome and Safari
