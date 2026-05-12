---
name: add-project
description: Add or refine a Project entry on the CaiKong website. Use when the user says "添加项目", "新增作品", "添加到 projects", "add project", or wants to publish a product/research/interactive HTML item under content/projects.
---

# Add Project

Add a project to the CaiKong site's Projects page. A project is usually a small frontmatter-only Markdown file in `content/projects/`; research projects may also include an interactive HTML file in `static/html/` and a thumbnail in `static/images/`.

## Workflow

### Step 1: Classify the Project

Use the existing two-track structure:

| Category | Use for | Existing tone |
|---|---|---|
| `product` | shipped or in-development game/product work | role, genre, engine, team/time, concise responsibility bullets |
| `research` | design research, specs, framework maps, interactive documents | distilled scope, covered topics, medium/interface, optional embedded HTML |

Default to `product` for portfolio work experience and `research` for knowledge documents, specs, GDC/GMTK compendiums, methodology pages, or interactive codex pages.

### Step 2: Gather Required Fields

Every file needs YAML frontmatter:

```yaml
---
title: "Project Title"
slug: "project-kebab-slug"
category: "product"
year: "2026"
type: "Role or Research Type, Genre/Domain, Engine/Format"
description: "Card/detail description. HTML line breaks and bullets are allowed."
team: "City · Company | YYYY.MM - YYYY.MM"
thumbnail: "static/images/project-thumb.png"
card_title: "Short Card Title"
card_meta: "Product &bull; 2026"
media:
  - { type: "html", url: "static/html/project-doc.html", title: "Embedded Document Title" }
---
```

Required fields: `title`, `slug`, `category`, `year`, `type`, `description`, `thumbnail`, `card_title`, `card_meta`.

Optional fields:
- `team`: product team/company/date or research context.
- `media`: one or more visual/detail assets. Use for interactive HTML, images, or video.
- `quotes`, `stats`: supported by `build.js`, but only add when the project has strong evidence for them.

### Step 3: Write the Description

For `product` entries:

- First sentence: product genre, perspective, platform, or core promise.
- Then `<br><br>` and 3-4 bullet lines using `•`.
- Bullets should describe actual design ownership: combat core, 3C, boss, role/skill design, level/event work, tool/system/pipeline design, milestone delivery.
- Keep the tone concrete and resume-like, not marketing-heavy.

Example shape:

```yaml
description: "一款修仙题材的ARPG。项目旨在PC端实现主机级的高品质战斗体验。<br><br>• 负责战斗核心玩法框架的设计与落地。<br>• 主导剧情BOSS副本的玩法设计与落地。<br>• 推进系统化功能开发，完成战斗底层核心功能系统建设。"
```

For `research` entries:

- One paragraph is usually enough.
- State what the interactive document distills, what topics it covers, and how it is presented.
- If it came from an Obsidian/GameDev methodology note, preserve its original role: comparison map, spec, framework reference, or codex. Do not rewrite it into a broad tutorial unless asked.

### Step 4: Name Files and Assets

Project Markdown:

```text
content/projects/<project-slug-without-project-prefix>.md
```

Slug convention:

```text
slug: "project-<same-kebab-name>"
```

Examples:

| File | Slug |
|---|---|
| `ak3.md` | `project-ak3` |
| `combat-table-spec.md` | `project-combat-table-spec` |
| `combat-framework-cross-reference.md` | `project-combat-framework-cross-reference` |

Static assets:

```text
static/html/<same-kebab-name>.html
static/images/<same-kebab-name>-thumb.png
static/images/<same-kebab-name>-thumb.svg
```

Use local thumbnail paths for research projects when possible. Existing product projects can use hosted `https://img.kongcai.cc/...` thumbnails.

### Step 5: Generate Thumbnail from HTML (HTML Projects Only)

If the project includes an HTML file in `static/html/`, **prioritize taking a Playwright screenshot of the hero section** (first viewport) as the thumbnail instead of manually sourcing one.

**Default — Playwright CLI (PowerShell):**

```powershell
# Replace <name> with the kebab slug
$html = (Resolve-Path "static/html/<name>.html").Path
npx playwright screenshot --browser chromium --viewport-size=1280,720 "file:///$html" "static/images/<name>-thumb.png"
```

If the page has a loading animation or deferred render, add `--wait-for-timeout`:

```powershell
npx playwright screenshot --browser chromium --viewport-size=1280,720 --wait-for-timeout=2000 "file:///$html" "static/images/<name>-thumb.png"
```

**Alternative — inline Node script (if CLI fails on file:// paths):**

```js
// run once: node screenshot.mjs
import { chromium } from 'playwright';
import { resolve } from 'path';

const name = '<name>'; // ← set slug
const browser = await chromium.launch();
const page = await browser.newPage();
await page.setViewportSize({ width: 1280, height: 720 });
await page.goto(`file:///${resolve(`static/html/${name}.html`)}`);
await page.screenshot({ path: `static/images/${name}-thumb.png` });
await browser.close();
```

**Post-screenshot checklist:**
- Open the PNG and confirm the hero is fully visible and not cropped awkwardly. Adjust `--viewport-size` height (640 / 720 / 800) to match the visual weight of the hero.
- Set the screenshot path as `thumbnail` in frontmatter:

  ```yaml
  thumbnail: "static/images/<name>-thumb.png"
  ```

- Only fall back to a hand-crafted thumbnail when the HTML hero is mostly blank, canvas-only, or requires user interaction before it renders meaningfully.

### Step 6: Attach Media

For an interactive HTML document, copy or create the HTML under `static/html/`, then add:

```yaml
media:
  - { type: "html", url: "static/html/<name>.html", title: "Readable Embed Title" }
```

Important:
- `build.js` renders HTML media in an iframe with `src="./static/html/..."`.
- The iframe sandbox allows scripts and same-origin, so self-contained interactive HTML works.
- Check embedded HTML for external fonts/assets. If it references local relative assets, copy those assets too or rewrite paths.

For image or video media, follow `build.js` support:

```yaml
media:
  - { type: "image", url: "static/images/example.png" }
  - { type: "video", url: "static/video/example.mp4" }
```

### Step 7: Build and Verify

Run:

```bash
npm run build
```

Then verify:

- `dist/index.html` contains the new `slug`.
- The project card image path appears and loads.
- Detail page contains the title, description, metadata, and media iframe/image/video.
- If a dev preview is needed, run `npm run dev`. If port 3000 is occupied, use PowerShell:

```powershell
$env:PORT='3001'; npm run dev
```

### Step 8: Commit Scope

When committing an added project, stage only the relevant files unless the user asks for all changes:

```bash
git add content/projects/<name>.md static/html/<name>.html static/images/<name>-thumb.*
git commit -m "feat(projects): add <project title>"
```

## File Locations

| File | Purpose |
|---|---|
| `content/projects/` | Project frontmatter entries |
| `static/html/` | Embedded interactive documents |
| `static/images/` | Project thumbnails and supporting images |
| `build.js` -> `readContentDir('projects')` | Reads projects |
| `build.js` -> `genProjectCard()` | Renders grid cards |
| `build.js` -> `genProjectDetail()` | Renders detail pages |
| `build.js` -> `genMediaItem()` | Renders image/video/html media |

## Existing Project Patterns

Use these as tone references:

| Project | Category | Emphasis |
|---|---|---|
| `ak3.md` | `product` | ARPG combat, multi-character switching, giant boss, role pipeline |
| `xiaoyaoyou.md` | `product` | lead combat design, core combat framework, boss encounters, version planning |
| `dawn.md` | `product` | executive lead design, TPS 3C, skill system, production decomposition |
| `gdc-combat-design.md` | `research` | GDC/GMTK combat design compendium, interactive web presentation |
| `combat-table-spec.md` | `research` | GAS skill data spec, layered definitions, validation rules, interactive spec |
| `combat-framework-cross-reference.md` | `research` | combat framework mapping, external vocabulary, interactive codex |
