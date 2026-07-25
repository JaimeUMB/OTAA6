# One Thing After Another — Editorial Redesign: Handoff

Context for continuing the editorial (paper-theme) redesign of the **One Thing
After Another** app. Read this top-to-bottom before making changes.

---

## 1. What the project is

- **App**: `index.html` at the repo root — a **single-file vanilla HTML/CSS/JS PWA**
  (~12,800 lines) called *One Thing After Another*. A personal life-tracker with
  sections: **Running, Calisthenics, Reading & Writing** (the three "home" modes),
  plus **Plan** (31-week marathon plan), **Tasks**, **Personal** (finance), **Profile**,
  and **Log Book** (entry forms).
- Persistence: `localStorage` is the source of truth; Firestore sync is optional.
- External CDNs used at runtime: **Google Fonts, Chart.js, Lucide**. (See §7 — these
  are blocked in the build sandbox, and some are now inlined/removed.)
- Repo: **`jaimeumb/otaa6`**, work branch **`claude/file-review-26gy8j`**.
  Served file is **`index.html`** (the user sometimes calls it `index-1.html`).

> Note: GitHub access in this environment is via the `mcp__github__*` tools (load
> with ToolSearch). Do not create a PR unless explicitly asked.

## 2. The design system ("Editorial")

Canonical source of truth = the **Design System Reference** mockup (see §6). It is a
warm cream/paper "editorial magazine" theme. **Single light theme only — dark mode
was removed.**

### Tokens (defined in `:root`, inside the single `<style>`)
```
Surfaces:  --bg #E9E2D2 (screen)  --bg-2 #EDE6D7 (card/panel)  --bg-3 #E6DCC8 (input)
           --canvas #CFC5B1  --field #E3D9C4 (derived/auto)  --hero #221D16 (dark hero)
Ink:       --text #322E27  --text-2 #5b564b  --text-3 #8a8477
           --text-muted #6E685C  --nav-inactive #9a9488  --placeholder #a89f8c
Accents:   --accent #B27A5F (terracotta = links/active/section №)  --accent-hover #8f5f48
           --action #B5461E (PRIMARY CTA / OVERLOAD ONLY)  --action-hover #8f3616  --action-border #983a17
Hairlines: --border rgba(50,46,39,.32) (card)  --border-hair .16 (dividers)
           --border-cell .22 (inner cells)  --border-input .28 (inputs)
Semantic:  --success #7E8A66 (sage — NO green in palette)  --warning/--gold #B08A45  --notification #a0331f (destructive)
Pigment:   --ochre #C4A56A  --sage #7E8A66  --sage-soft #A6AC96  --blush #C99C89
           --terra-tan #B98A72  --dblue #6E8390  --blue-soft #9EAAB2
Geometry:  --radius 0  --radius-sm 0  (sharp panels)
Type:      --font-display 'Playfair Display',Georgia,serif   (numerals/headlines/wordmark)
           --font-ui 'Space Grotesk','Geist',sans-serif      (UI/labels)
           --font-mono 'JetBrains Mono',monospace            (data annotations)
Shadow:    --shadow-inset inset 0 1px 0 rgba(255,255,255,.5)  --shadow-lg none
```

### HARD CONSTRAINTS (non-negotiable — from the reference's Do/Don't)
1. **No rounded corners** except the inset top-highlight; pills/dots/chips keep their
   own `border-radius` (99px/999px/50%). Panels/cards/inputs/buttons are sharp.
2. **No drop shadows** beyond `--shadow-inset`.
3. **No colours/gradients outside the palette**, except the effort-zone gradient
   `linear-gradient(90deg,#7E8A66,#C4A56A 45%,#B08A45 65%,#B5461E)`.
4. **Playfair Display for all large numerals/headlines** — never sans-serif.
5. **Red `#B5461E` (`--action`) is reserved** for primary CTAs / overload warnings only.
   Interactive accent is terracotta `--accent`.
6. **Micro-labels are always UPPERCASE, letter-spaced, Space Grotesk** — never sentence case.
7. **Keep the paper-grain overlay and two-line wordmark** in the app shell.

The `<head>` loads Playfair Display + Space Grotesk + JetBrains Mono via Google Fonts.
Paper grain is a `body::after` fixed fractal-noise overlay. Wordmark is two-line in the
sidebar (`.sb-logo-text` → "One Thing<br>After Another").

## 3. Work completed (commits on `claude/file-review-26gy8j`, oldest→newest)

1. `cb5d755` **Migrate to Editorial design system; remove dark mode** — rewrote `:root`
   tokens; global remap of every foreign hex (violet/blue/red/amber/teal/green/orange)
   → pigment set across CSS **and** JS/chart configs; zeroed panel radii; neutralised
   drop-shadows; re-capitalised the 35 micro-labels the old "Notion" `#pg-home` override
   layer had de-capped; added paper grain + two-line wordmark; **removed dark mode**
   (11 `[data-theme="dark"]` CSS blocks + base dark palette deleted; `applyTheme()`
   locks to light; `toggleTheme()` is a no-op; sidebar theme button + Profile
   Appearance/Theme row removed; `ui.theme` init forced `'light'`).
2. `ac8b0d5` **Rebuild Home → Running tab** to the editorial mockup (see §5).
3. `e55d2ac` **Running fixes**: removed `⬇️` emoji from `computeACWR()` Undertraining
   label + fixed its `var(--error)`→`var(--notification)`; rebuilt "This week" to the
   mockup structure (range + "N done · M to go" header, tag pills, dots on the progress
   line, labels); verified `Running · No. X` == total logged runs.
4. `fbf8e87` **Inline navigation icons** so they render without the Lucide CDN.
5. `e08e646` **Match nav icons to the design-system shell** (Log = list-lines,
   Personal = person; Home/Plan/Tasks unchanged).
6. `8a201fd` **Rebuild Home → Calisthenics tab** to the editorial mockup (see §5).

## 4. Key code locations (use function names — line numbers drift)

- **`:root`** — top of the single `<style>` (~L40–90).
- **`#pg-home` override layer** (~L1250–5560) — a prior "Notion" restyle scoped to the
  home page. Mostly **inert now** for Running/Calisthenics because those branches were
  fully replaced. Still styles Reading & Writing home (not yet rebuilt).
- **`renderHome(el)`** — builds the home page. Computes ALL data at the top (available to
  every mode branch): `wd`, `wActs`, `moKm`, `moLng`, `wkSteps`, `wkKcal`, `runStreak`,
  `acwr` (`.ratio/.label/.color`), `ctlToday`, `ctlTgt`, `rdPct`, `tsbLabel`, `thisWeekKm`,
  `planWeekKm`, `kmPct`, `avg4`, `wkRem`, `spkD/spkL/spkAvg8/spkYMax`, `recentRuns`,
  `_sessByDate`, `dLbls`, `kcalMethod`, cal* vars (`calPullups`, `calDips`, `calStreak`,
  `ppDone`, `prehabDone`, `prehabWarn`, …), and `_sliderHtml` (the mode switcher).
  - **Calisthenics branch**: `if(AppState.ui.homeMode==='calisthenics'){ … return; }` — ✅ REBUILT (editorial).
  - **Reading branch**: `if(AppState.ui.homeMode==='reading'){renderReadingHome(el,_sliderHtml);return;}` — ⛔ NOT rebuilt.
  - **Running** = the fallthrough after the reading return — ✅ REBUILT (editorial).
- **`renderReadingHome(el, sliderHtml)`** — Reading & Writing home; ⛔ still the old
  token-only skin (kpi-row / rd-* / Chart.js donut). **Next to rebuild.**
- **`buildSidebar()`** + **`NAV_ICON_SVG`** map — desktop sidebar nav (inline SVG icons).
- **`<nav id="bot-nav">`** static HTML — mobile bottom nav (inline SVG icons).
- Hero image constants **`_HERO_RUN_IMG`**, **`_HERO_CAL_IMG`** (base64 JPEG data URIs)
  are defined just before `function renderHome`.
- Data helpers: `computeACWR()`, `computePMC()`, `computeExpectedPeakCTL()`,
  `computePaceZones()`, `generatePlan()`, `getWeekDates()`, `getActsForDate/Week()`,
  `fmtPace()`, `fmtDateSh()`, `iso()`, `isoToday()`. `window.APP.SCHEDULE` = 31-week plan.

### Data model
- **Activity** `{type:'run'|'calisthenics'|'reading', date, …}`
  - run: `distance_km, duration_sec, avg_pace_sec_per_km, run_type, avg_hr, step_count, load_score`
  - calisthenics: `workout_type ('Pull & Push'|'Prehab'|…), pullups, dips, duration_min, rpe,
    prehab_complete, exercises:[{name, type:'sets'|time, sets_or_min, reps, load, kg}]`
  - reading: `activity_type:'reading'|'writing'|'editing', pages, words, book_name, duration_min`
- **Seed**: `init()` seeds **runs only** (`_SEED_RUNS`); reading has `_RD_SEED` (tracker).
  **No calisthenics is seeded**, so the Calisthenics home legitimately shows 0/"—".
- `AppState.ui`: `homeMode`, `theme:'light'` (locked), `tab`, `logMode`, `tskView`.

## 5. The editorial home-tab recipe (how Running & Calisthenics were built)

Each rebuilt branch sets `el.innerHTML = _sliderHtml + \`…editorial markup…\`` using
full-width panels (the `.page` container supplies the gutter — do NOT copy the mockup's
`22px` side padding). Standard sections, top → bottom:
1. **Eyebrow date row** — uppercase terracotta label · hairline · long date.
2. **Duotone hero** — `background:var(--hero)`, photo on right ~58–60% (`object-fit:cover`),
   left→right scrim `linear-gradient(95deg,var(--hero) 27%, …, transparent 84%)`, a vertical
   `writing-mode:vertical-rl` "… · No. N" label, ochre eyebrow, **giant Playfair numeral**,
   delta line. Sharp corners (no radius).
3. **3-up specimen metrics grid** — bordered `--bg-2` panel, three cells with 8.5px
   uppercase label + 7px colour dot, big Playfair value, top-bordered 8px unit footer.
4. **Full-width index row** — Run Streak (Running) / **action-red Workout-Streak banner**
   (Calisthenics).
5. **Readiness / status panel** (Running: CTL / Week Volume / Load Ratio; Calisthenics:
   two planned status cards).
6. **"This week" strip** — panel with `range` + `N done · M to go` header, tag-pill row,
   dots ON the progress line (faint track + gold fill to today; today = ringed gold dot),
   day/date labels. Days call `showDayPopover(iso,this)`.
7. **Numbered section headers** (`01/02/03…` terracotta + Playfair title + hairline + meta).
8. **Recent-activity rows** — coloured 4px left mark, name, `date · … · zone`, big Playfair value.
9. **CSS bar chart** (Running weekly volume — replaces Chart.js) / **Max-set table**
   (Calisthenics, computed from `exercises[]`).
10. **Personal Bests / Max set table** — grid `Distance|Pace|Date`, Playfair values, "—" fallback.

Wire everything to the real data vars (do not hardcode mockup placeholder numbers). End
the branch with `requestAnimationFrame(()=>{ if(typeof _createIcons==='function')_createIcons(); });`.

## 6. The mockup source files (uploads)

Location: `/root/.claude/uploads/ce5995f1-05f3-558e-9c4d-2fb9060beaca/`. These are
Claude-Artifact **"bundled page" exports** (huge, self-contained). Files: Design System
Reference, Profile, Plan, Tasks, Running, Calisthenics, Reading & Writing, and Log Book
(Running/Calisthenics/Reading&Writing). Each renders a 390×844 phone frame.

**Extracting a mockup's HTML/images** (they don't read as plain HTML):
```python
import json, base64, io
from PIL import Image, ImageOps
raw = open(FILE).read().split('\n')
# find the line with <script type="__bundler/template"> ; the NEXT line is the payload
tmpl = json.loads(payload_line)          # -> a STRING = the mockup's real HTML
# manifest line similarly: json.loads(...) -> { uuid: {mime, compressed, data(base64)} }
img = ImageOps.exif_transpose(Image.open(io.BytesIO(base64.b64decode(m[uuid]['data']))))
```
Decoded copies of every mockup already exist in the scratchpad
(`…/scratchpad/*_template.html`) from earlier turns.

**Hero images** (embedded in the bundles; extracted + recompressed with PIL):
- Running = **El Ángel Caído statue** — uuid `25c4565b-…` in the Running bundle → `_HERO_RUN_IMG` (~90 KB).
- Calisthenics = **surreal facade** — uuid `5255eba3-…` in the Calisthenics bundle → `_HERO_CAL_IMG` (~49 KB).
- Reading & Writing = **tiled-room** photo — uuid `ed1bf5a4-…` in the Reading bundle (NOT yet used);
  plus a writing-time side-plate `7c9cd870-…`.
- iPhone photos need `ImageOps.exif_transpose`; recompress to ~760–900px, JPEG q72–74.

## 7. Environment & verification workflow

- **CDNs are 403-blocked** in this sandbox (unpkg, jsdelivr, googleapis/gstatic). So
  in-sandbox renders show **serif/sans font fallback** and **no Chart.js/Lucide**. In a
  real browser they load. What's already CDN-independent: **all nav icons** (inline SVG),
  the **Running & Calisthenics heroes** (embedded), and those two home tabs use **no
  Chart.js** (CSS bars instead).
- **JS syntax check** (run after every edit):
  ```bash
  python3 -c "import re;s=open('index.html').read();open('/tmp/app.js','w').write(max(re.findall(r'<script>(.*?)</script>',s,re.S),key=len))"
  node --check /tmp/app.js
  ```
  (There are 4 inline `<script>` blocks; the largest is the app.)
- **Screenshot** (Playwright, Chromium preinstalled):
  ```js
  import pkg from '/opt/node22/lib/node_modules/playwright/index.js'; const {chromium}=pkg;
  const b=await chromium.launch({executablePath:'/opt/pw-browsers/chromium-1194/chrome-linux/chrome'});
  const pg=await b.newPage({viewport:{width:402,height:900,deviceScaleFactor:2},isMobile:true});
  await pg.route('**unpkg.com/**', r=>r.abort());   // simulate CDN block
  await pg.goto('file:///home/user/OTAA6/index.html',{waitUntil:'load'});
  await pg.waitForTimeout(1800);
  await pg.evaluate(()=>{navigate('#home');setHomeMode('calisthenics');});
  await pg.screenshot({path:'out.png',fullPage:true});
  ```
- **PIL/Pillow** is installed (`pip install pillow` already done).
- **Editing large JS templates**: write the new block to a file via a quoted heredoc
  (`cat > f <<'EOF'`) so `$`/backticks stay literal, then insert with a small Python
  script that splices by unique string anchors. This avoids escaping hell. (Embedded
  base64 hero consts are inserted the same way.)

## 8. Remaining work / TODO

1. **Reading & Writing home** — rebuild `renderReadingHome()` to the editorial mockup:
   words-written hero (extract tiled-room image `ed1bf5a4`), Pages/Edit-time/Books grid,
   reading & writing weekly-goal bars, writing-time breakdown (segmented bar + side-plate
   image `7c9cd870`), recent-sessions rows, reading-tracker table. It currently uses a
   Chart.js donut — replace with editorial markup like the other tabs. **This is the last
   home tab still on the old skin.**
2. **Plan / Tasks / Profile pages** — tokens are migrated (colours/fonts/radius), but the
   component markup is NOT yet rebuilt to the mockup recipes (specimen tiles, editorial
   calendar, task rows). Optional deeper pass; mockups exist for all three.
3. **In-content Lucide icons** — the Log form (+/×), Recipes (chef-hat), import/upload,
   activity, chevrons, etc. still use `<i data-lucide>` and go blank under strict CSP.
   Convert to inline SVG (couldn't bundle the Lucide lib — CDN 403 in sandbox). The nav
   pattern (`NAV_ICON_SVG` map + inline SVG) is the template to follow.
4. **Plan-page emoji / undefined var** — `getPlanHealth()` still emits `Needs Adjustment 🔁`
   and uses `var(--error)` (undefined → no colour); a help-docs table also shows
   `Undertraining ⬇️`. Clean these if the user wants (readiness section was already fixed).
5. **Full offline** — Fonts / Chart.js / (remaining) Lucide are still CDN. For a truly
   self-contained PWA, inline them (blocked here by the 403 proxy; do when network allows).
6. **Filename** — served as `index.html`; confirm with user if they expect `index-1.html`.

## 9. Conventions

- Work on **`claude/file-review-26gy8j`**; commit + push each logical change
  (`git push -u origin claude/file-review-26gy8j`, retry w/ backoff on network errors).
- Commit trailers used in this project:
  ```
  Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01V2hWZyoVsQJvpCGLmWtuS2
  ```
- **Do not change app logic / calculations / state / event handlers / ids / classes the JS
  queries** — this is a visual migration. (The one authorised exception already done:
  removing the dark-mode toggle.)
- After any change: `node --check`, then screenshot the affected screen with the CDN
  blocked to confirm it renders self-contained.
- Verify against the **Design System Reference** first, individual screen mockups second.
