# Portfolio Identity Kit (Week 03 Deliverable)

> **Pass/Revise Criteria Verification**:
> - [x] **1–2 Free Fonts Named**: Space Grotesk (Headings) + Inter (Body)
> - [x] **Tight 4-Color Palette with Hex Codes**: `#F8F9FA`, `#111215`, `#2B2F38`, `#0D52D6`
> - [x] **Simple Logo & Favicon**: Geometric vector monogram "AD" + wordmark
> - [x] **Two-Line Style Note**: Ready for Claude Project / Antigravity instructions
> - [x] **Single-Page Deliverable**: Complete live HTML interactive kit built at [index.html](file:///C:/Users/ameyd/.gemini/antigravity/scratch/identity-kit/index.html)

---

## 1. Typography Selection

| Role | Font Family | Source | Specimen / Usage |
| :--- | :--- | :--- | :--- |
| **Heading Font** | **Space Grotesk** (Weight: 600) | [Google Fonts](https://fonts.google.com/specimen/Space+Grotesk) | Display headlines, section titles, case study titles. Proportional grotesque with sharp architectural ink-traps that communicate technical precision. |
| **Body Font** | **Inter** (Weights: 400, 500) | [Google Fonts](https://fonts.google.com/specimen/Inter) | Explanatory prose, telemetry, tabular data, and UI controls. Neutral grotesque construction optimized for high legibility at micro and macro scales. |

---

## 2. Tight 4-Color Palette

Designed with strict restraint so that your engineering work, screenshots, and benchmarks remain the loudest visual element on the page.

| Swatch | Color Name | Hex Code | RGB | Role & Accessibility Rationale |
| :---: | :--- | :--- | :--- | :--- |
| <span style="background-color:#F8F9FA; border:1px solid #ccc; display:inline-block; width:28px; height:28px; border-radius:4px;"></span> | **Near-White Background** | `#F8F9FA` | `rgb(248, 249, 250)` | Soft chalk white. Eliminates high-contrast screen glare while providing an optical baseline for images. |
| <span style="background-color:#111215; display:inline-block; width:28px; height:28px; border-radius:4px;"></span> | **Near-Black Text** | `#111215` | `rgb(17, 18, 21)` | Deep graphite ink. 16.5:1 AAA contrast against `#F8F9FA`. Avoids harsh pitch-black clipping. |
| <span style="background-color:#2B2F38; display:inline-block; width:28px; height:28px; border-radius:4px;"></span> | **Main Structure** | `#2B2F38` | `rgb(43, 47, 56)` | Slate steel for architectural dividers, tabular headers, borders, and secondary metadata. |
| <span style="background-color:#0D52D6; display:inline-block; width:28px; height:28px; border-radius:4px;"></span> | **Calm Accent** | `#0D52D6` | `rgb(13, 82, 214)` | Signal Cobalt Blue. Calm, authoritative, and reserved strictly for interactive states, key focal points, and active filters. |

---

## 3. Logo & Favicon

### Monogram & Wordmark Mark
- **Vector Construction**: Precision architectural grid monogram intertwining "A" and "D" with a calibrated accent focus node.
- **Wordmark**: `AMEY DONGRE` set in uppercase Space Grotesk (tracked +0.02em) with subtitle `ENGINEERING & SYSTEMS`.

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64" width="64" height="64">
  <rect width="64" height="64" rx="14" fill="#111215"/>
  <path d="M18 46 L26 18 L34 46 M20.5 37 L31.5 37" stroke="#F8F9FA" stroke-width="3.5" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
  <path d="M36 18 L43 18 C48.5 18 52 22 52 32 C52 42 48.5 46 43 46 L36 46 Z" stroke="#F8F9FA" stroke-width="3.5" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
  <circle cx="43" cy="32" r="3" fill="#0D52D6"/>
</svg>
```

- Standalone asset saved at: [favicon.svg](file:///C:/Users/ameyd/.gemini/antigravity/scratch/identity-kit/favicon.svg)

---

## 4. Two-Line Style Note (Add to Claude Project)

Copy and paste this exact snippet into your Claude Project or agent custom instructions:

```text
Fonts: Space Grotesk (headings, 600) & Inter (body, 400/500). Palette: #F8F9FA (bg), #111215 (text), #2B2F38 (structure), #0D52D6 (accent).
Mood: Swiss industrial precision and rigorous restraint — unadorned structural clarity where the case studies and data stand out without visual competition.
```

---

## 5. Ready-to-Submit Files

1. **Interactive Deliverable Webpage**: [index.html](file:///C:/Users/ameyd/.gemini/antigravity/scratch/identity-kit/index.html)  
   - Includes live archetype switcher (Swiss Modernist, Editorial Brutalist, Spatial, Warm Artisan).
   - One-click copy for the style note.
   - Built-in `Print / Save as PDF` stylesheet optimized for assignment upload.
2. **Vector Favicon**: [favicon.svg](file:///C:/Users/ameyd/.gemini/antigravity/scratch/identity-kit/favicon.svg)
