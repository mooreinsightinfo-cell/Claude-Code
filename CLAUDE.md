# CLAUDE.md — Moore Insight Shopify Theme

## Project Overview

This is the **Moore Insight LLC** Shopify theme (v2.0.0). It is a pure Shopify theme with no build system, no npm, and no framework dependencies — just Liquid templates, one CSS file, and one JS file.

The theme is a branded landing page for Moore Insight LLC, combining Kemetic/ancient African cultural aesthetics with modern AI and technology content.

---

## Repository Structure

```
assets/
  moore-insight.css         — all styles (~12KB)
  moore-insight.js          — 12-line vanilla JS (3 features)
config/
  settings_schema.json      — theme metadata only (name, version, author)
layout/
  theme.liquid              — HTML5 shell: loads fonts, CSS, JS
locales/
  en.default.json           — accessibility string (skip-to-content)
sections/
  moore-insight-landing.liquid — entire landing page HTML + Liquid schema
templates/
  index.json                — wires the homepage to the landing section
  page.json                 — generic page template
README-SHOPIFY-INSTALL.txt  — non-technical upload/customization guide
```

---

## Shopify Liquid Conventions

### Schema-in-section pattern
Every `.liquid` section file ends with a `{% schema %}` JSON block defining all editable settings and block types. This is the Shopify standard — do not move schema to external files.

### Settings access
Use `{{ section.settings.<id> }}` to output a setting value. Strip paragraph wrappers from richtext headings:
```liquid
{{ section.settings.hero_heading | remove: '<p>' | remove: '</p>' }}
```

### Block iteration
Blocks are typed. Always check `block.type` before rendering:
```liquid
{% for block in section.blocks %}
  {% if block.type == 'pillar' %}
    ...{{ block.settings.glyph }}...
  {% endif %}
{% endfor %}
```

### Current block types in the landing section
| Type | Settings |
|------|----------|
| `pillar` | `glyph`, `title`, `text` |
| `product_card` | `sigil`, `tag`, `title`, `text`, `price`, `price_suffix`, `btn_text`, `link` |

### Conditional rendering
Use blank checks to hide optional fields rather than showing empty HTML:
```liquid
{% if section.settings.consulting_li1 != blank %}<li>{{ section.settings.consulting_li1 }}</li>{% endif %}
{% if section.settings.social_youtube != blank %}<a href="{{ section.settings.social_youtube }}">YouTube</a>{% endif %}
```

### Image handling
Always provide a fallback for optional images:
```liquid
{% if section.settings.consulting_image != blank %}
  <img src="{{ section.settings.consulting_image | image_url: width: 800 }}" alt="...">
{% else %}
  <img src="{{ 'queen.png' | asset_url }}" alt="..." onerror="this.style.display='none'">
{% endif %}
```

---

## CSS Conventions

### CSS custom properties (color palette)
All colors are defined in `:root` in `moore-insight.css`:

| Variable | Value | Usage |
|----------|-------|-------|
| `--black` | `#000000` | Page background |
| `--ink` | `#0a0805` | Deep near-black |
| `--gold` | `#d4af37` | Primary gold |
| `--gold-bright` | `#f5d77a` | Light gold highlight |
| `--gold-deep` | `#9a7a1f` | Dark gold shadow |
| `--gold-soft` | `#caa84a` | Mid gold (eyebrow text) |
| `--cream` | `#f4ecd8` | Body text |
| `--muted` | `#b9ac86` | Secondary/muted text |
| `--line` | `rgba(212,175,55,0.18)` | Subtle gold dividers |

### Gold text gradient
Apply the `.gold-text` class for the standard shimmer gradient. Do not hardcode gold colors inline.

### Italic-to-gold pattern (critical)
Italicising a word in any richtext heading (`<em>` tag) automatically renders it in the gold gradient. This is wired in CSS:
```css
h1 em, h2 em, h1 strong, h2 strong {
  font-style: normal;
  background: linear-gradient(135deg, var(--gold-deep) ... var(--gold-bright) ...);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
```
**Never remove or restructure this rule.** It is the core editorial feature of the theme.

### Scroll-reveal animations
Add `.reveal` to any element that should animate in on scroll. The JS automatically observes all `.reveal` elements and adds `.in` once they reach 15% viewport visibility. CSS handles the actual animation via `.reveal.in` rules.

### Layout utility
`.container` sets `max-width: 1280px` with `padding: 0 28px`. Use it for all full-width section content wrappers.

### Section layout patterns
- `.split` + `.split-text` + `.split-visual` — two-column image/text layout
- `.sec-head` — centered section header with kicker + heading + divider + description
- `.pillar-grid` / `.prod-grid` — responsive card grids

---

## Typography

| Font | CSS family | Usage |
|------|-----------|-------|
| Cinzel | `'Cinzel', serif` | Brand name, nav links, eyebrow text, section kickers |
| Cormorant Garamond | `'Cormorant Garamond', serif` | Hero h1, blockquotes (`.cormorant` class) |
| Jost | `'Jost', sans-serif` | Default body font (weight 300) |

Fonts are loaded from Google Fonts in `layout/theme.liquid`.

---

## JavaScript

`assets/moore-insight.js` (and an identical inline copy at the bottom of the landing section) provides exactly three features:

1. **Nav scroll state** — toggles `.scrolled` on `#header` when `window.scrollY > 40`
2. **Mobile menu toggle** — toggles `.open` on `#navLinks` via `#menuToggle` button; closes on nav link click
3. **Reveal on scroll** — `IntersectionObserver` at 15% threshold adds `.in` to `.reveal` elements

The JS is intentionally duplicated: the asset file covers any non-section pages; the inline script ensures the landing section's DOM is available immediately without waiting for the asset to load.

---

## Adding New Sections

1. Create `sections/your-section-name.liquid`
2. Write HTML/Liquid using the same color variables and class conventions
3. Add a `{% schema %}` block at the bottom with `"name"`, `"settings"`, and optionally `"blocks"` and `"presets"`
4. Reference the new section in a template JSON file (e.g., `templates/index.json`) under `"sections"`
5. Do not create separate CSS files — add styles to `assets/moore-insight.css`

---

## Adding New Block Types

Inside the `{% schema %}` of a section, add to the `"blocks"` array:
```json
{
  "type": "your_block_type",
  "name": "Display Name",
  "settings": [
    { "type": "text", "id": "your_field", "label": "Label", "default": "Default value" }
  ]
}
```
Then iterate in the template with `{% if block.type == 'your_block_type' %}`.

---

## Deployment

### Option A — Zip upload (no CLI needed)
1. Zip the entire theme directory (all folders: assets, config, layout, locales, sections, templates)
2. Shopify Admin › Online Store › Themes › Add theme › Upload zip file

### Option B — Shopify CLI (recommended for active development)
```bash
shopify theme push --store=<store-name>.myshopify.com
```
or for live preview during development:
```bash
shopify theme dev --store=<store-name>.myshopify.com
```

---

## Content Editing (No Code Needed)

All text, images, and links are configurable in **Shopify Admin › Online Store › Themes › Customize**. See `README-SHOPIFY-INSTALL.txt` for a full list of editable fields. Code changes are only needed when:
- Adding or restructuring sections/blocks
- Modifying styles or layout
- Adding new JS behavior

---

## Image Assets

Three optional images are referenced by filename from CSS/Liquid:

| Filename | Used in |
|----------|---------|
| `banner.png` | Hero background (CSS `background-image`) |
| `queen.png` | Consulting section fallback image |
| `king.png` | About section fallback image |

Upload these via **Shopify Admin › Themes › Edit code › Assets** with these exact filenames. If absent, the fallback `onerror` handler hides the image and shows a dark radial gradient background.

---

## What NOT to Do

- Do not run `npm install` or introduce a build step — there is no package.json by design
- Do not create additional CSS or JS files — keep all styles in `moore-insight.css` and JS in `moore-insight.js`
- Do not remove `<em>`/`<strong>` gold gradient rules from CSS — they are an editorial feature
- Do not hardcode color hex values in new HTML/Liquid — use CSS custom properties (`var(--gold)`, etc.)
- Do not strip `{{ block.shopify_attributes }}` from block elements — Shopify needs it for Theme Editor inline editing
- Do not connect product cards directly to Shopify product objects unless intentionally migrating to a Shopify catalog-backed design (current cards are static landing-page content)
