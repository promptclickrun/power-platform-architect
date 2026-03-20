---
name: architecture-html-dashboard
description: >
  Generates a self-contained, interactive HTML dashboard for Power Platform and
  Copilot Studio architecture documentation. Features pure inline SVG diagrams,
  dark theme, responsive layout, clickable stat cards with slide-in detail drawers,
  and tabbed content. No external dependencies — zero CDN calls.
---

# Skill: Architecture HTML Dashboard Template

> Generates a self-contained, interactive HTML dashboard for Power Platform and Copilot Studio architecture documentation. Features pure inline SVG diagrams (no Mermaid), dark theme, responsive layout, clickable stat cards with slide-in detail drawers, and tabbed content. Fully self-contained — no CDN or external scripts.

---

## When to Use This Skill

Use this skill when generating an architecture analysis as a **single self-contained HTML file**. The template provides:

- Dark-themed responsive dashboard (mobile, tablet, desktop, print)
- Hero section with solution name, badge, and description
- Clickable stat cards → slide-in modal drawers with expandable detail cards
- Five tabbed content panels: Architecture, ERD, Data Flows, Components, Notes
- **Pure inline SVG diagrams** — no Mermaid, no CDN, no external dependencies
- Zoom controls for diagrams
- Full print stylesheet

### Why SVG Instead of Mermaid

Mermaid v11 has critical rendering issues that make it unreliable for production dashboards:

- **Emoji failures:** Emojis in subgraph labels cause parse failures
- **Newline breakage:** `\n` in node/edge labels breaks rendering
- **Colon failures:** Colons in edge labels fail to parse
- **Hidden tab rendering:** `display:none` containers prevent Mermaid rendering entirely
- **CDN fragility:** CDN loading can fail silently, leaving blank diagram panels

Pure inline SVG eliminates all of these issues. Diagrams render instantly, work in hidden tabs, support any Unicode text, and require zero external resources.

---

## Critical Rendering Rules

### Architecture SVG Rules

#### Canvas Setup

Every architecture diagram uses this SVG wrapper:

```xml
<svg viewBox="0 0 1400 1200" xmlns="http://www.w3.org/2000/svg"
     style="width:100%;height:auto;font-family:'Segoe UI',sans-serif;">
```

Adjust the `viewBox` height (1200) based on the number of tiers. Each tier needs ~140-180px including spacing.

#### Required Defs Section

Always include this `<defs>` block as the first child of every architecture SVG. It defines arrowhead markers (one per tier color) and a glow filter for hero components:

```xml
<defs>
    <marker id="ah" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
        <polygon points="0 0,10 3.5,0 7" fill="#64748b"/>
    </marker>
    <marker id="ah-blue" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
        <polygon points="0 0,10 3.5,0 7" fill="#3b82f6"/>
    </marker>
    <marker id="ah-green" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
        <polygon points="0 0,10 3.5,0 7" fill="#10b981"/>
    </marker>
    <marker id="ah-orange" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
        <polygon points="0 0,10 3.5,0 7" fill="#f59e0b"/>
    </marker>
    <marker id="ah-purple" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
        <polygon points="0 0,10 3.5,0 7" fill="#a855f7"/>
    </marker>
    <marker id="ah-cyan" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
        <polygon points="0 0,10 3.5,0 7" fill="#06b6d4"/>
    </marker>
    <filter id="glow">
        <feGaussianBlur stdDeviation="3" result="blur"/>
        <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
</defs>
```

#### Tier Layout System

Architecture diagrams are organized into **horizontal tiers** stacked vertically. Each tier represents a logical layer of the solution (e.g., Intake, Triggers, Agent, External API, Data/AI, Fulfillment).

**Tier Background:**
```xml
<rect x="20" y="[tierY]" width="1360" height="[tierHeight]" rx="14"
      fill="[tier-dark-color]" stroke="[tier-accent]" stroke-width="1.5" opacity="0.5"/>
```

**Tier Label** — positioned at top-left inside the background:
```xml
<text x="40" y="[tierY+22]" fill="[tier-accent]" font-size="11" font-weight="700"
      text-transform="uppercase" letter-spacing="1.5">[TIER NAME]</text>
```

**Component Boxes** — placed inside tier backgrounds:
```xml
<rect x="[boxX]" y="[boxY]" width="[boxW]" height="[boxH]" rx="8"
      fill="[darker-shade]" stroke="[tier-accent]" stroke-width="1.2"/>
<text x="[centerX]" y="[textY]" fill="#e2e8f0" font-size="11" font-weight="600"
      text-anchor="middle">[Component Name]</text>
<text x="[centerX]" y="[textY+15]" fill="[tier-accent]" font-size="9"
      text-anchor="middle">[subtitle or type]</text>
```

**Hero/Primary Components** get a glow effect and thicker stroke:
```xml
<rect ... filter="url(#glow)" stroke-width="2"/>
```

**Vertical Spacing:** Leave 85-110px gaps between tier backgrounds to provide space for connection routing.

#### Tier Color Reference

| Tier | Purpose | Stroke/Accent | Fill (background) | Fill (component box) |
|------|---------|---------------|--------------------|--------------------|
| Intake | Input channels | `#3b82f6` | `#0f1d32` | `#1e3a5f` |
| Triggers | Automation flows | `#8b5cf6` | `#1a1040` | `#2d1b69` |
| Agent | Copilot agents | `#10b981` | `#0d2618` | `#1a4731` |
| External API | Graph, REST calls | `#a855f7` | `#200d28` | `#3b1d3f` |
| Data/AI | Dataverse, AI Builder | `#06b6d4` / `#f59e0b` | `#0a1e2e` / `#261508` | `#1e3a5f` / `#4a2c17` |
| Approval | Workflows, approvals | `#f59e0b` | `#261508` | `#4a2c17` |
| Fulfillment | Output actions | `#10b981` | `#0d2618` | `#1a4731` |

### ERD SVG Rules

#### Canvas Setup

```xml
<svg viewBox="0 0 1200 850" xmlns="http://www.w3.org/2000/svg"
     style="width:100%;height:auto;font-family:'Segoe UI',sans-serif;">
```

Adjust the `viewBox` dimensions based on entity count. Plan ~400px width per entity column, ~200-300px height per entity.

#### Entity Box Structure

Each entity is drawn as a header bar + body rect with field rows:

```xml
<!-- Entity: EntityName -->
<!-- Header bar -->
<rect x="30" y="30" width="370" height="40" rx="10" fill="#1e3a5f"/>
<rect x="30" y="60" width="370" height="2" fill="#3b82f6"/>
<text x="215" y="56" fill="#e2e8f0" font-size="14" font-weight="700"
      text-anchor="middle">EntityName</text>

<!-- Body -->
<rect x="30" y="62" width="370" height="[bodyHeight]" rx="0 0 10 10"
      fill="#111827" stroke="#1e2d4a" stroke-width="1"/>

<!-- Field row (20px line height, start at y=86) -->
<text x="48" y="86" fill="#f59e0b" font-size="10" font-weight="700">PK</text>
<text x="78" y="86" fill="#e2e8f0" font-size="11" font-weight="600">FieldName</text>
<text x="380" y="86" fill="#8896b3" font-size="10" text-anchor="end">string</text>
<line x1="42" y1="95" x2="388" y2="95" stroke="#1e2d4a" stroke-width="1"/>

<!-- Next field row at y=106 -->
<text x="48" y="106" fill="#3b82f6" font-size="10" font-weight="700">FK</text>
<text x="78" y="106" fill="#e2e8f0" font-size="11">RelatedField</text>
<text x="380" y="106" fill="#8896b3" font-size="10" text-anchor="end">guid</text>
<line x1="42" y1="115" x2="388" y2="115" stroke="#1e2d4a" stroke-width="1"/>
```

**Field annotation colors:**
- `PK` → `fill="#f59e0b"` (orange)
- `FK` → `fill="#3b82f6"` (blue)
- Regular fields → no annotation column, text starts at `x="48"`

#### Crow's Foot Relationship Notation

Draw relationships using `<line>` or `<path>` elements with crow's foot markers at endpoints:

**"One" marker** (perpendicular bar):
```xml
<line x1="[x-7]" y1="[y]" x2="[x+7]" y2="[y]" stroke="[color]" stroke-width="2"/>
```

**"Many" marker** (fork lines + circle):
```xml
<line x1="[x-8]" y1="[y-8]" x2="[x]" y2="[y]" stroke="[color]" stroke-width="2"/>
<line x1="[x+8]" y1="[y-8]" x2="[x]" y2="[y]" stroke="[color]" stroke-width="2"/>
<circle cx="[x]" cy="[y-7]" r="4" fill="none" stroke="[color]" stroke-width="1.5"/>
```

**"Zero-or" modifier** — add a small open circle before the bar or fork:
```xml
<circle cx="[x]" cy="[y-12]" r="4" fill="none" stroke="[color]" stroke-width="1.5"/>
```

**Relationship labels** — use opaque pill labels at the midpoint of each connecting line:
```xml
<rect x="[labelX-halfW]" y="[labelY-10]" width="[w]" height="16" rx="4"
      fill="#0a0e1a" opacity="0.92"/>
<text x="[labelCenterX]" y="[labelY+2]" fill="[color]" font-size="9"
      text-anchor="middle">[relationship label]</text>
```

Route relationship lines through gaps between entity columns (typically 130-150px gap between columns).

---

## SVG Design Principles

### Non-Crossing Connection Routing Algorithm

This is the most critical rule for readable architecture diagrams. Connections between tiers MUST NOT cross each other.

**Use L-shaped routes** with SVG `<path>` elements:
```xml
<path d="M x1,y1 V yMid H x2 V y2" fill="none"
      stroke="[color]" stroke-width="1.5" marker-end="url(#ah-[color])"/>
```

The `d` attribute means: start at (x1,y1), go Vertical to yMid, go Horizontal to x2, go Vertical to y2.

**The Stagger Rule (prevents crossings):**

When multiple connections enter the same tier from above, stagger their landing x-positions from **LEFT to RIGHT**, and stagger their horizontal routing y-positions at incrementing y-values.

Example — 4 connections dropping from Tier 1 into Tier 2:
```
Connection 1: lands at x=150, horizontal at y=328
Connection 2: lands at x=200, horizontal at y=343
Connection 3: lands at x=250, horizontal at y=358
Connection 4: lands at x=300, horizontal at y=373
```

Each subsequent connection lands further right and routes at a lower y-value, so horizontal segments never cross vertical drops from earlier connections.

**Primary flow:** solid lines, `stroke-width="1.5"`
**Secondary/return flow:** `stroke-dasharray="5,3"` or `"6,3"`, `stroke-width="1"`
**Left gutter (x < 60):** route return/feedback paths along the left edge
**Right gutter (x > 1300):** route re-invoke/callback paths along the right edge

### Opaque Pill Labels

Every connection line MUST have a label explaining what flows through it. Labels use an opaque background pill so they remain readable over connection lines and tier backgrounds:

```xml
<!-- Pill background -->
<rect x="[labelX - halfWidth]" y="[labelY - 12]" width="[pillWidth]" height="16"
      rx="4" fill="#0a0e1a" opacity="0.92"/>
<!-- Label text -->
<text x="[labelCenterX]" y="[labelY]" fill="[tier-accent-color]" font-size="9"
      text-anchor="middle">[Label Text]</text>
```

Place labels at the midpoint of the longest segment of each connection path (usually the horizontal segment).

### Step Number Badges

For sequential flows within a tier (e.g., steps 1→2→3 inside an Agent tier), use numbered circle badges:

```xml
<circle cx="[x]" cy="[y]" r="9" fill="#0d2618" stroke="#10b981" stroke-width="1.2"/>
<text x="[x]" y="[y+4]" fill="#10b981" font-size="8" font-weight="700"
      text-anchor="middle">1</text>
```

### Glow Effects for Hero Components

The primary/hero component in the diagram (e.g., the main Copilot agent) should stand out with a glow filter and thicker stroke:

```xml
<rect x="..." y="..." width="..." height="..." rx="10"
      fill="#1a4731" stroke="#10b981" stroke-width="2" filter="url(#glow)"/>
```

### Legend

Include a color-coded legend at the bottom of each diagram panel mapping tier colors to their purpose:

```html
<div class="legend">
    <div class="legend-item"><div class="legend-dot" style="background:#3b82f6"></div>Intake</div>
    <div class="legend-item"><div class="legend-dot" style="background:#8b5cf6"></div>Triggers</div>
    <div class="legend-item"><div class="legend-dot" style="background:#10b981"></div>Agent / Fulfillment</div>
    <div class="legend-item"><div class="legend-dot" style="background:#a855f7"></div>External API</div>
    <div class="legend-item"><div class="legend-dot" style="background:#06b6d4"></div>Data</div>
    <div class="legend-item"><div class="legend-dot" style="background:#f59e0b"></div>AI / Approval</div>
</div>
```

---

## Full HTML Template

Replace all `{{PLACEHOLDER}}` values with real data from your analysis. The template is the structural skeleton — populate every section with solution-specific content.

**IMPORTANT:** This template has **no external dependencies**. No `<script src="...">` tags, no CDN calls. Diagrams are pure inline SVG embedded directly in the HTML.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{SOLUTION_NAME}} — Architecture & ERD</title>
    <!-- NO external scripts — diagrams are pure inline SVG -->
    <style>
        /* === RESET & VARIABLES === */
        *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
        :root {
            --bg: #0a0e1a;
            --surface: #111827;
            --surface-alt: #1a2236;
            --border: #1e2d4a;
            --border-hover: #3b5998;
            --text: #e2e8f0;
            --text-muted: #8896b3;
            --accent-blue: #3b82f6;
            --accent-purple: #8b5cf6;
            --accent-green: #10b981;
            --accent-orange: #f59e0b;
            --accent-red: #ef4444;
            --accent-cyan: #06b6d4;
            --glow-blue: rgba(59, 130, 246, 0.15);
            --glow-purple: rgba(139, 92, 246, 0.15);
        }

        body {
            font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif;
            background: var(--bg); color: var(--text); line-height: 1.6; overflow-x: hidden;
        }

        body::before {
            content: ''; position: fixed; top: 0; left: 0; right: 0; bottom: 0;
            background:
                radial-gradient(ellipse 80% 50% at 20% 20%, rgba(59,130,246,0.06) 0%, transparent 60%),
                radial-gradient(ellipse 60% 40% at 80% 80%, rgba(139,92,246,0.06) 0%, transparent 60%);
            pointer-events: none; z-index: 0;
        }

        .container { max-width: 1400px; margin: 0 auto; padding: 2rem 1.5rem; position: relative; z-index: 1; }

        /* === HERO === */
        .hero { text-align: center; padding: 3rem 1rem 2rem; margin-bottom: 2rem; }
        .hero-badge {
            display: inline-flex; align-items: center; gap: 0.5rem;
            padding: 0.4rem 1rem; background: linear-gradient(135deg, var(--glow-blue), var(--glow-purple));
            border: 1px solid var(--border); border-radius: 999px;
            font-size: 0.8rem; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.1em; margin-bottom: 1.5rem;
        }
        .hero-badge .dot { width: 6px; height: 6px; background: var(--accent-green); border-radius: 50%; animation: pulse 2s ease-in-out infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }
        .hero h1 {
            font-size: clamp(2rem, 5vw, 3.2rem); font-weight: 700;
            background: linear-gradient(135deg, #fff 0%, #94a3b8 100%);
            -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; margin-bottom: 0.75rem;
        }
        .hero p { font-size: 1.1rem; color: var(--text-muted); max-width: 700px; margin: 0 auto; }

        /* === STAT CARDS (clickable) === */
        .stats-row { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 1rem; margin-bottom: 2.5rem; }
        .stat-card {
            background: var(--surface); border: 1px solid var(--border); border-radius: 12px;
            padding: 1.25rem; text-align: center; transition: all 0.3s; cursor: pointer; position: relative; overflow: hidden;
        }
        .stat-card:hover { border-color: var(--border-hover); transform: translateY(-3px); box-shadow: 0 8px 24px rgba(0,0,0,0.3); }
        .stat-card::after {
            content: 'Click for details →'; position: absolute; bottom: 0; left: 0; right: 0;
            padding: 0.3rem; font-size: 0.65rem; color: var(--text-muted); background: var(--surface-alt);
            opacity: 0; transform: translateY(100%); transition: all 0.25s;
        }
        .stat-card:hover::after { opacity: 1; transform: translateY(0); }
        .stat-value { font-size: 2rem; font-weight: 700; line-height: 1; margin-bottom: 0.35rem; }
        .stat-label { font-size: 0.78rem; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.05em; }

        /* Color each stat card's value */
        .stat-card:nth-child(1) .stat-value { color: var(--accent-blue); }
        .stat-card:nth-child(2) .stat-value { color: var(--accent-purple); }
        .stat-card:nth-child(3) .stat-value { color: var(--accent-green); }
        .stat-card:nth-child(4) .stat-value { color: var(--accent-orange); }
        .stat-card:nth-child(5) .stat-value { color: var(--accent-cyan); }
        .stat-card:nth-child(6) .stat-value { color: var(--accent-red); }

        /* === MODAL DRAWER === */
        .modal-overlay {
            position: fixed; top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(0,0,0,0.65); backdrop-filter: blur(4px);
            z-index: 1000; opacity: 0; pointer-events: none; transition: opacity 0.3s;
        }
        .modal-overlay.open { opacity: 1; pointer-events: all; }
        .modal-drawer {
            position: fixed; top: 0; right: 0; bottom: 0; width: min(560px, 90vw);
            background: var(--surface); border-left: 1px solid var(--border); z-index: 1001;
            transform: translateX(100%); transition: transform 0.35s cubic-bezier(0.16, 1, 0.3, 1);
            display: flex; flex-direction: column; box-shadow: -8px 0 40px rgba(0,0,0,0.4);
        }
        .modal-drawer.open { transform: translateX(0); }
        .modal-header {
            display: flex; align-items: center; gap: 0.75rem;
            padding: 1.25rem 1.5rem; border-bottom: 1px solid var(--border); background: var(--surface-alt); flex-shrink: 0;
        }
        .modal-header-icon { width: 40px; height: 40px; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 1.3rem; flex-shrink: 0; }
        .modal-header h2 { font-size: 1.1rem; font-weight: 600; flex: 1; }
        .modal-header .modal-count { padding: 0.2rem 0.65rem; border-radius: 999px; font-size: 0.75rem; font-weight: 600; }
        .modal-close {
            width: 32px; height: 32px; display: flex; align-items: center; justify-content: center;
            background: transparent; border: 1px solid var(--border); border-radius: 6px;
            color: var(--text-muted); cursor: pointer; font-size: 1.1rem; transition: all 0.2s; font-family: inherit;
        }
        .modal-close:hover { background: rgba(239,68,68,0.15); border-color: var(--accent-red); color: var(--accent-red); }
        .modal-body { flex: 1; overflow-y: auto; padding: 1.5rem; }
        .modal-body::-webkit-scrollbar { width: 6px; }
        .modal-body::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }

        /* Detail cards inside modal */
        .detail-card { background: var(--bg); border: 1px solid var(--border); border-radius: 12px; margin-bottom: 1rem; overflow: hidden; transition: border-color 0.2s; }
        .detail-card:hover { border-color: var(--border-hover); }
        .detail-card-header { display: flex; align-items: center; gap: 0.75rem; padding: 1rem 1.25rem; cursor: pointer; transition: background 0.2s; }
        .detail-card-header:hover { background: rgba(255,255,255,0.02); }
        .detail-card-icon { width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 0.95rem; flex-shrink: 0; }
        .detail-card-header h4 { font-size: 0.9rem; font-weight: 600; flex: 1; }
        .detail-card-header .detail-tag { padding: 0.15rem 0.55rem; border-radius: 999px; font-size: 0.65rem; font-weight: 500; text-transform: uppercase; letter-spacing: 0.04em; }
        .detail-card-chevron { color: var(--text-muted); transition: transform 0.2s; font-size: 0.8rem; }
        .detail-card.expanded .detail-card-chevron { transform: rotate(90deg); }
        .detail-card-body { max-height: 0; overflow: hidden; transition: max-height 0.35s ease, padding 0.35s ease; padding: 0 1.25rem; }
        .detail-card.expanded .detail-card-body { max-height: 600px; padding: 0 1.25rem 1.25rem; border-top: 1px solid var(--border); }
        .detail-row { display: flex; padding: 0.5rem 0; font-size: 0.83rem; border-bottom: 1px solid rgba(255,255,255,0.04); }
        .detail-row:last-child { border-bottom: none; }
        .detail-row-label { color: var(--text-muted); min-width: 120px; flex-shrink: 0; }
        .detail-row-value { color: var(--text); }
        .detail-row-value code { background: rgba(59,130,246,0.12); color: var(--accent-blue); padding: 0.1rem 0.4rem; border-radius: 4px; font-size: 0.78rem; font-family: 'Cascadia Code', 'Fira Code', monospace; }

        /* === TABS === */
        .tabs { display: flex; gap: 0.25rem; margin-bottom: 0; border-bottom: 1px solid var(--border); overflow-x: auto; -webkit-overflow-scrolling: touch; }
        .tab-btn {
            padding: 0.75rem 1.5rem; background: transparent; border: none; color: var(--text-muted);
            font-size: 0.9rem; font-weight: 500; cursor: pointer; border-bottom: 2px solid transparent;
            transition: all 0.25s; white-space: nowrap; font-family: inherit;
        }
        .tab-btn:hover { color: var(--text); background: rgba(255,255,255,0.03); }
        .tab-btn.active { color: var(--accent-blue); border-bottom-color: var(--accent-blue); }
        .tab-content { animation: fadeIn 0.3s ease; }
        .tab-content.hidden { display: none; }
        .tab-content.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }

        /* === DIAGRAM PANELS === */
        .diagram-panel { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; margin-top: 1.5rem; overflow: hidden; }
        .diagram-panel-header { display: flex; align-items: center; justify-content: space-between; padding: 1rem 1.5rem; border-bottom: 1px solid var(--border); background: var(--surface-alt); }
        .diagram-panel-header h3 { font-size: 1rem; font-weight: 600; display: flex; align-items: center; gap: 0.5rem; }
        .diagram-panel-body { padding: 2rem; overflow-x: auto; min-height: 400px; display: flex; align-items: center; justify-content: center; }
        .diagram-panel-body svg { width: 100%; height: auto; }
        .legend { display: flex; flex-wrap: wrap; gap: 1rem; padding: 1rem 1.5rem; border-top: 1px solid var(--border); background: var(--surface-alt); }
        .legend-item { display: flex; align-items: center; gap: 0.4rem; font-size: 0.78rem; color: var(--text-muted); }
        .legend-dot { width: 10px; height: 10px; border-radius: 3px; }

        /* Zoom controls */
        .zoom-controls { display: flex; gap: 0.25rem; }
        .zoom-btn {
            width: 32px; height: 32px; display: flex; align-items: center; justify-content: center;
            background: var(--surface); border: 1px solid var(--border); border-radius: 6px;
            color: var(--text-muted); cursor: pointer; font-size: 1rem; font-family: inherit; transition: all 0.2s;
        }
        .zoom-btn:hover { border-color: var(--border-hover); color: var(--text); }

        /* === INVENTORY CARDS === */
        .inventory-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(340px, 1fr)); gap: 1.25rem; margin-top: 1.5rem; }
        .inv-card { background: var(--surface); border: 1px solid var(--border); border-radius: 12px; overflow: hidden; transition: border-color 0.3s, box-shadow 0.3s; }
        .inv-card:hover { border-color: var(--border-hover); box-shadow: 0 4px 24px rgba(0,0,0,0.2); }
        .inv-card-header { padding: 1rem 1.25rem; border-bottom: 1px solid var(--border); display: flex; align-items: center; gap: 0.75rem; }
        .inv-icon { width: 36px; height: 36px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 1.1rem; flex-shrink: 0; }
        .inv-card-header h4 { font-size: 0.95rem; font-weight: 600; }
        .inv-card-header .inv-tag { margin-left: auto; padding: 0.15rem 0.6rem; border-radius: 999px; font-size: 0.7rem; font-weight: 500; text-transform: uppercase; letter-spacing: 0.04em; }
        .inv-card-body { padding: 1rem 1.25rem; }
        .inv-card-body ul { list-style: none; }
        .inv-card-body li { padding: 0.4rem 0; font-size: 0.85rem; color: var(--text-muted); border-bottom: 1px solid rgba(255,255,255,0.04); display: flex; align-items: flex-start; gap: 0.5rem; }
        .inv-card-body li:last-child { border-bottom: none; }
        .inv-card-body li::before { content: '›'; color: var(--accent-blue); font-weight: 700; flex-shrink: 0; margin-top: -1px; }
        .inv-card-body li strong { color: var(--text); font-weight: 500; }

        /* === DATA FLOW === */
        .flow-section { margin-top: 1.5rem; }
        .flow-card { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 2rem; margin-bottom: 1.5rem; }
        .flow-card h3 { font-size: 1.1rem; margin-bottom: 1.25rem; display: flex; align-items: center; gap: 0.5rem; }
        .flow-steps { position: relative; padding-left: 2rem; }
        .flow-steps::before { content: ''; position: absolute; left: 11px; top: 0; bottom: 0; width: 2px; background: linear-gradient(180deg, var(--accent-blue), var(--accent-purple), var(--accent-green)); border-radius: 1px; }
        .flow-step { position: relative; padding: 0.6rem 0 0.6rem 1rem; font-size: 0.9rem; color: var(--text-muted); }
        .flow-step::before { content: ''; position: absolute; left: -1.97rem; top: 0.85rem; width: 10px; height: 10px; border-radius: 50%; border: 2px solid var(--accent-blue); background: var(--bg); }
        .flow-step.decision::before { border-color: var(--accent-orange); background: var(--accent-orange); }
        .flow-step.success::before { border-color: var(--accent-green); background: var(--accent-green); }
        .flow-step.error::before { border-color: var(--accent-red); background: var(--accent-red); }
        .flow-step strong { color: var(--text); }
        .flow-branch { margin: 0.5rem 0 0.5rem 1rem; padding: 0.75rem 1rem; border-radius: 8px; font-size: 0.85rem; }
        .flow-branch.success-branch { background: rgba(16,185,129,0.08); border-left: 3px solid var(--accent-green); color: #6ee7b7; }
        .flow-branch.error-branch { background: rgba(239,68,68,0.08); border-left: 3px solid var(--accent-red); color: #fca5a5; }

        /* === NOTES === */
        .notes-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 1.25rem; margin-top: 1.5rem; }
        .note-card { background: var(--surface); border: 1px solid var(--border); border-radius: 12px; padding: 1.5rem; }
        .note-card h4 { font-size: 0.95rem; margin-bottom: 0.75rem; display: flex; align-items: center; gap: 0.5rem; }
        .note-card ul { list-style: none; }
        .note-card li { padding: 0.35rem 0; font-size: 0.85rem; color: var(--text-muted); line-height: 1.5; }

        /* === RESPONSIVE === */
        @media (max-width: 768px) {
            .container { padding: 1rem; }
            .hero { padding: 2rem 0.5rem 1rem; }
            .stats-row { grid-template-columns: repeat(3, 1fr); gap: 0.5rem; }
            .stat-card { padding: 0.75rem; }
            .stat-value { font-size: 1.5rem; }
            .stat-card::after { display: none; }
            .inventory-grid { grid-template-columns: 1fr; }
            .diagram-panel-body { padding: 1rem; }
            .tabs { gap: 0; }
            .tab-btn { padding: 0.65rem 1rem; font-size: 0.82rem; }
        }
        @media (max-width: 480px) {
            .stats-row { grid-template-columns: repeat(2, 1fr); }
            .legend { gap: 0.5rem; }
            .notes-grid { grid-template-columns: 1fr; }
            .modal-drawer { width: 100vw; }
        }
        @media print {
            body { background: #fff; color: #000; }
            .tabs, .zoom-controls { display: none; }
            .tab-content { display: block !important; page-break-inside: avoid; }
            .diagram-panel { border: 1px solid #ccc; }
            .modal-overlay, .modal-drawer { display: none !important; }
        }
    </style>
</head>
<body>

<!-- Modal elements -->
<div class="modal-overlay" id="modalOverlay" onclick="closeModal()"></div>
<div class="modal-drawer" id="modalDrawer">
    <div class="modal-header" id="modalHeader"></div>
    <div class="modal-body" id="modalBody"></div>
</div>

<div class="container">
    <div class="hero">
        <div class="hero-badge"><span class="dot"></span> Production Architecture</div>
        <h1>{{SOLUTION_NAME}}</h1>
        <p>{{SOLUTION_DESCRIPTION}}</p>
    </div>

    <div class="stats-row">
        <!-- Each stat card calls openModal('key') -->
        <div class="stat-card" onclick="openModal('{{KEY}}')">
            <div class="stat-value">{{VALUE}}</div>
            <div class="stat-label">{{LABEL}}</div>
        </div>
        <!-- Repeat for each stat... -->
    </div>

    <div class="tabs" role="tablist">
        <button class="tab-btn active" data-tab="arch" role="tab">Architecture</button>
        <button class="tab-btn" data-tab="erd" role="tab">ERD</button>
        <button class="tab-btn" data-tab="flow" role="tab">Data Flows</button>
        <button class="tab-btn" data-tab="inventory" role="tab">Components</button>
        <button class="tab-btn" data-tab="notes" role="tab">Notes</button>
    </div>

    <!-- Architecture Tab — inline SVG, no library needed -->
    <div class="tab-content active" id="tab-arch">
        <div class="diagram-panel">
            <div class="diagram-panel-header">
                <h3>Solution Architecture</h3>
                <div class="zoom-controls">
                    <button class="zoom-btn" onclick="zoom('archDiagram', -0.15)">−</button>
                    <button class="zoom-btn" onclick="zoom('archDiagram', 0.15)">+</button>
                </div>
            </div>
            <div class="diagram-panel-body">
                <div id="archDiagram" style="width:100%;">
                    <svg viewBox="0 0 1400 1200" xmlns="http://www.w3.org/2000/svg"
                         style="width:100%;height:auto;font-family:'Segoe UI',sans-serif;">
                        <!-- {{ARCHITECTURE_SVG}} -->
                        <!-- Include <defs> with markers and glow filter -->
                        <!-- Draw tier backgrounds, component boxes, connections, pill labels -->
                    </svg>
                </div>
            </div>
            <div class="legend">
                <!-- Color-coded legend items matching tier colors -->
                <div class="legend-item"><div class="legend-dot" style="background:#3b82f6"></div>Intake</div>
                <div class="legend-item"><div class="legend-dot" style="background:#8b5cf6"></div>Triggers</div>
                <div class="legend-item"><div class="legend-dot" style="background:#10b981"></div>Agent / Fulfillment</div>
                <div class="legend-item"><div class="legend-dot" style="background:#a855f7"></div>External API</div>
                <div class="legend-item"><div class="legend-dot" style="background:#06b6d4"></div>Data</div>
                <div class="legend-item"><div class="legend-dot" style="background:#f59e0b"></div>AI / Approval</div>
            </div>
        </div>
    </div>

    <!-- ERD Tab — inline SVG with crow's foot notation -->
    <div class="tab-content hidden" id="tab-erd">
        <div class="diagram-panel">
            <div class="diagram-panel-header">
                <h3>Entity Relationship Diagram</h3>
                <div class="zoom-controls">
                    <button class="zoom-btn" onclick="zoom('erdDiagram', -0.15)">−</button>
                    <button class="zoom-btn" onclick="zoom('erdDiagram', 0.15)">+</button>
                </div>
            </div>
            <div class="diagram-panel-body">
                <div id="erdDiagram" style="width:100%;">
                    <svg viewBox="0 0 1200 850" xmlns="http://www.w3.org/2000/svg"
                         style="width:100%;height:auto;font-family:'Segoe UI',sans-serif;">
                        <!-- {{ERD_SVG}} -->
                        <!-- Draw entity boxes with header/body/fields -->
                        <!-- Draw relationship lines with crow's foot markers and pill labels -->
                    </svg>
                </div>
            </div>
            <div class="legend">
                <div class="legend-item"><div class="legend-dot" style="background:#f59e0b"></div>Primary Key</div>
                <div class="legend-item"><div class="legend-dot" style="background:#3b82f6"></div>Foreign Key</div>
                <div class="legend-item"><div class="legend-dot" style="background:#8896b3"></div>Attribute</div>
            </div>
        </div>
    </div>

    <div class="tab-content hidden" id="tab-flow">
        <!-- Data flow cards with timeline steps and decision branches -->
    </div>
    <div class="tab-content hidden" id="tab-inventory">
        <!-- Inventory grid cards -->
    </div>
    <div class="tab-content hidden" id="tab-notes">
        <!-- Notes grid: Strengths, Considerations, Dependencies, Security, Env Vars, ROI -->
    </div>
</div>

<script>
    // Modal data object — one key per stat card
    const modalData = {
        // Each key contains: icon, iconBg, iconColor, title, count, countBg, countColor, items[]
        // Each item: icon, iconBg, iconColor, name, tag, tagBg, tagColor, details[]
        // Each detail: { label, value } where value can contain <code> and <strong> HTML
    };

    function openModal(key) {
        const data = modalData[key]; if (!data) return;
        document.getElementById('modalHeader').innerHTML = `
            <div class="modal-header-icon" style="background:${data.iconBg}; color:${data.iconColor}">${data.icon}</div>
            <h2>${data.title}</h2>
            <span class="modal-count" style="background:${data.countBg}; color:${data.countColor}">${data.count}</span>
            <button class="modal-close" onclick="closeModal()" title="Close">✕</button>`;
        document.getElementById('modalBody').innerHTML = data.items.map((item, i) => `
            <div class="detail-card ${i === 0 ? 'expanded' : ''}" onclick="toggleCard(this)">
                <div class="detail-card-header">
                    <div class="detail-card-icon" style="background:${item.iconBg}; color:${item.iconColor}">${item.icon}</div>
                    <h4>${item.name}</h4>
                    <span class="detail-tag" style="background:${item.tagBg}; color:${item.tagColor}">${item.tag}</span>
                    <span class="detail-card-chevron">▶</span>
                </div>
                <div class="detail-card-body">
                    ${item.details.map(d => `<div class="detail-row"><span class="detail-row-label">${d.label}</span><span class="detail-row-value">${d.value}</span></div>`).join('')}
                </div>
            </div>`).join('');
        document.getElementById('modalOverlay').classList.add('open');
        document.getElementById('modalDrawer').classList.add('open');
    }
    function closeModal() {
        document.getElementById('modalOverlay').classList.remove('open');
        document.getElementById('modalDrawer').classList.remove('open');
    }
    function toggleCard(el) { el.classList.toggle('expanded'); }
    document.addEventListener('keydown', e => { if (e.key === 'Escape') closeModal(); });

    // Tab switching
    document.querySelectorAll('.tab-btn').forEach(btn => {
        btn.addEventListener('click', () => switchTab(btn.dataset.tab));
    });
    function switchTab(tabName) {
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.toggle('active', b.dataset.tab === tabName));
        document.querySelectorAll('.tab-content').forEach(tc => {
            const id = tc.id.replace('tab-', '');
            tc.classList.toggle('hidden', id !== tabName);
            tc.classList.toggle('active', id === tabName);
        });
    }

    // Zoom
    const zoomLevels = {};
    function zoom(id, delta) {
        const el = document.getElementById(id);
        if (!el) return;
        zoomLevels[id] = Math.min(2.5, Math.max(0.3, (zoomLevels[id] || 1) + delta));
        el.style.transform = `scale(${zoomLevels[id]})`;
        el.style.transformOrigin = 'top center';
    }

    // No diagram library needed — SVGs are inline
    // Initialize default tab immediately
    switchTab('arch');
</script>
</body>
</html>
```

---

## Template Placeholders Reference

| Placeholder | Description | Example |
|---|---|---|
| `{{SOLUTION_NAME}}` | Solution display name | `Manufacturing HR Copilot` |
| `{{SOLUTION_DESCRIPTION}}` | 1-2 sentence overview | `AI-powered intake processing...` |
| `{{KEY}}` | Modal data key for stat card | `agents`, `aiModels`, `flows` |
| `{{VALUE}}` | Stat card numeric value | `2`, `6`, `13` |
| `{{LABEL}}` | Stat card label text | `AI Models`, `Cloud Flows` |
| `{{ARCHITECTURE_SVG}}` | Full SVG content for architecture diagram | Tier backgrounds, boxes, connections |
| `{{ERD_SVG}}` | Full SVG content for ERD diagram | Entity boxes, fields, relationships |

---

## Content Population Guide

### Stat Cards
Create one `<div class="stat-card">` per major metric. Typical cards: Agents, AI Models, Flows, MCPs/Connectors, Channels, Data Entities. Each must have a corresponding `modalData` key.

### modalData Structure
```javascript
const modalData = {
    agents: {
        icon: '🤖', iconBg: 'rgba(59,130,246,0.15)', iconColor: '#3b82f6',
        title: 'Copilot Studio Agents', count: '2 agents',
        countBg: 'rgba(59,130,246,0.15)', countColor: '#3b82f6',
        items: [
            {
                icon: '🎯', iconBg: 'rgba(139,92,246,0.15)', iconColor: '#8b5cf6',
                name: 'Agent Name', tag: 'PRIMARY', tagBg: 'rgba(139,92,246,0.15)', tagColor: '#8b5cf6',
                details: [
                    { label: 'Schema', value: '<code>cr665_agent</code>' },
                    { label: 'Purpose', value: 'Routes incoming requests...' }
                ]
            }
        ]
    }
};
```

### Tabs
- **Architecture**: Inline SVG with tier layout inside a `.diagram-panel`. Use `viewBox="0 0 1400 1200"`. Include `<defs>` with arrowhead markers and glow filter. Draw tier backgrounds, component boxes, L-shaped connection paths with pill labels, and step badges.
- **ERD**: Inline SVG with entity boxes and crow's foot relationships inside a `.diagram-panel`. Use `viewBox="0 0 1200 850"`. Draw entity header bars, body rects, field rows (PK/FK/regular), and relationship lines with crow's foot markers and pill labels.
- **Data Flows**: `.flow-card` elements with `.flow-steps` timeline
- **Components**: `.inventory-grid` with `.inv-card` elements per category
- **Notes**: `.notes-grid` with `.note-card` elements (Strengths, Considerations, Dependencies, Security, ROI)
