# Convalt Energy — 3D Website · Phase 0 Audit & Plan

Status: Phase 0 complete, waiting for direction approval and assets. No code has been written and nothing is deployed.
Date: 2026-10-06

---

## 0. What was actually inspected

| Source | Result |
|---|---|
| SOW PDF (DigiCollect, Draft v1.0, 5 Sep 2026, 4 pp.) | Read in full. |
| Repository `khalidvvvven/3d` | Empty: one `README.md` and an initial commit. **No client assets, logos, copy, imagery, video, PDFs or brand guide.** |
| www.convalt.com | **Not reachable.** The build environment's network policy blocks the host (it returns 403 at the proxy), and the site isn't indexed under `site:` search. |
| mont-fort.com | **Not reachable** for the same reason. I haven't studied the benchmark directly. |
| Public third-party sources (web search) | Gave a few company facts, listed in §A.3. They are **reference only, not approved copy.** |
| Installed tools/skills | No Three.js/R3F/GLSL/GLTF/GSAP/UI-UX/Motion/21st.dev/Nano Banana skills or MCPs are installed in this environment. Available and relevant: Playwright plus pre-installed Chromium (screenshots and QA), npm registry access, and the `code-review`/`simplify` skills for later review passes. |
| Hardware | 4 vCPU, 15 GB RAM, **no GPU**. Headless Chromium renders WebGL through SwiftShader (a software renderer). |

The last point matters. I can measure draw calls, triangles, texture memory, JS cost, CPU frame time, CLS and LCP here. I **can't** honestly measure real GPU frame rate. A 60 fps claim needs profiling on real hardware (your laptop, an iPhone, a mid-range Android) or a GPU CI runner, and I won't claim it otherwise.

---

## A. Project understanding

### A.1 What the SOW actually requires
1. **Landing page = full 3D/WebGL.** The SOW calls it "the single most important deliverable". It must be cinematic and scroll-driven, built on Convalt's business (solar manufacturing, power generation, data centres, recycling) "reimagined in 3D rather than stock photography". It needs a hero moment, then a guided reveal of value propositions.
2. **Lightweight fallback** for low-power devices. The SOW explicitly accepts a static hero image or CSS animation.
3. **Nav, logo and CTAs** must stay legible and accessible over the 3D background.
4. **Internal pages are 2D:** Projects, Team, Media, Press Releases, Resources, Contact, plus project detail pages. They use the same theme, the same IA as the current site (improvements need approval), responsive layouts, a contact form and basic SEO.
5. **Stack** is the developer's choice but must be justified, current, maintainable, with no proprietary or abandoned frameworks, and must support current and previous Chrome/Safari/Edge/Firefox. The stack must be **documented before development**. This document does that.
6. **Deliverables:** source in Git, asset library (models, textures, images), design files if a non-code tool is used, and a staging/live URL. Under your rule, deployment waits for explicit approval.
7. **Timeline in the SOW:** 3 days design + 4 days development, marked "indicative". See risk R6.
8. **Out of scope:** hosting and maintenance, and photorealistic 3D beyond what the landing page needs.

### A.2 Positioning
Convalt presents itself as a **vertically integrated** energy company. That gives the creative a strong premise: the homepage is *one connected system*, not four business lines placed side by side. The narrative device below is built around that.

### A.3 Company facts found publicly (to be verified by the client; not for use as copy yet)
- Founded 2011. Founder, Chairman and CEO: Hari Achuthan. *(SEC 8-K exhibit, 2022)*
- Describes itself as a vertically integrated renewable company: panel manufacturing, owned generation, and engineering and construction including panel recycling.
- 2021: acquired SunPower manufacturing assets and moved automated lines from Hillsboro, OR to **Watertown, NY**. The stated factory capacity was "up to 900 MW", with a 2023 target date that needs status confirmation.
- Recycling facility under development in **East Millinocket, Maine**. Community and utility-scale projects in Maine and New York.
- 2026: purchase and sale agreement with Gallup Land Partners for an **advanced manufacturing campus in Gallup, New Mexico** (cells, modules, solar glass) with **up to 1 GW behind-the-meter generation**.
- A press item mentions a solar project in Laos (nny360), which needs confirmation.
- Listed on datacentermap.com as a data centre company. I couldn't fetch the details.

Some of these sources are four years old. **None of these numbers will appear on the site until Convalt confirms them.**

---

## B. Sitemap (proposed, mirroring the SOW list; to be reconciled with the live site once accessible)

```
/                         Home — 3D experience (+ full semantic DOM copy)
/about                    About / Company   (if present on the current site; likely)
/projects                 Project index (filter: category · status · location)
/projects/[slug]          Project detail
/team                     Leadership / team
/media                    Media hub (news coverage, video, gallery)
/press                    Press releases (dated list → /press/[slug] if releases are hosted)
/resources                Downloads (PDFs, fact sheets, reports)
/contact                  Contact form + offices
/privacy, /terms          Legal (if present on current site)
/404
```
Possible IA improvement, for your approval: merge **Media** and **Press** into one "Newsroom" with filters, and keep `/press` as a filtered URL. I'll only do this if the current site's content is thin.

---

## C. Asset inventory

| Asset | Status | Needed for |
|---|---|---|
| Logo (SVG, mono and colour) | **Missing** | Loader, nav, favicon, OG images |
| Brand guide (colours, type) | **Missing** | Design tokens |
| Copy (all pages) | **Missing** | Every page |
| Project data (name, location, category, status, capacity, summary, images) | **Missing** | Projects and detail pages |
| Team (names, titles, bios, portraits) | **Missing** | Team |
| Press releases / media items | **Missing** (some are findable publicly) | Press / Media |
| Downloadable resources (PDFs) | **Missing** | Resources |
| Photography / video of real facilities | **Missing** | 2D pages and the 3D fallback poster |
| Contact details / form destination | **Missing** | Contact |
| 3D models | Not needed from the client. See §F.4: the homepage is mostly procedural. | — |

---

## D. Proposed stack

| Layer | Choice | Why |
|---|---|---|
| Site framework | **Astro 7** (static output) | Multi-page static HTML is right for SEO and the 2D pages. Internal pages ship close to zero JS. Content collections fit projects, press, team and resources (typed Markdown/JSON, which makes editing easy). Astro uses Vite underneath. It's widely used and well maintained. |
| Homepage 3D | **React 19 + React Three Fiber 9 + Drei 10 + three r186**, mounted as one Astro island | Declarative scene graph that's easy to hand over. Drei supplies `PerformanceMonitor`, `AdaptiveDpr`, `Instances`, `useGLTF`, and KTX2/Draco/Meshopt loaders. |
| Timeline | **GSAP 3.15 + ScrollTrigger**, with **Lenis** for smooth scroll | One master timeline driven by normalised scroll progress. GSAP is now free for commercial use. Lenis gives consistent wheel and trackpad feel and steps aside for reduced motion. |
| Post | `@react-three/postprocessing` (tone mapping, very light bloom, vignette): **HIGH tier only** | Every effect has a cost, so the lower tiers skip it. |
| GPU tiering | `detect-gpu` (initial guess) + Drei `PerformanceMonitor` (runtime correction) | Picks tiers from GPU capability rather than screen width. |
| DOM motion on 2D pages | CSS + small GSAP utilities | No second animation system (Motion) unless a real need appears. |
| Styling | CSS custom-property tokens + scoped Astro styles | No UI kit, to avoid a template look. |
| QA | Playwright 1.63 + @axe-core/playwright, Lighthouse CI | Screenshots at timeline checkpoints, plus accessibility and SEO checks. |
| Forms | Astro static form posting to a configurable endpoint (env var), e.g. Formspree, Netlify Forms or the client's CRM | No fake success state. Documented. |
| Fonts | Self-hosted WOFF2 subsets. Brand fonts once supplied; interim proposal: a neo-grotesk display face (e.g. Inter Display / Inter Tight, OFL) plus a mono for labels (IBM Plex Mono, OFL) | Licences need to be safe for handover. |

Alternative considered: Next.js. It's rejected because a server runtime and hydration everywhere give no benefit for a mostly static corporate site.

---

## E. Proposed 3D narrative — "One Line"

**Device:** a single thin conductor line runs through the whole experience. It starts as a **busbar** printed on a solar cell. It becomes the **grid line** leaving an array, then the **power and data path** into a compute hall, then the **material-recovery path** in recycling, and it closes the loop back into a new wafer. The line is the brand idea made visible: *one integrated system*. It's the only "effect" in the piece, and it's drawn, not glowing.

**Geometry continuity:** the **cell** is the atom of the whole film. One instanced cell geometry becomes the module, the array, the data-hall rack grid (same instancing, re-targeted transforms) and the recycled stack. Viewers see a transformation instead of cuts, and the GPU sees a handful of draw calls.

| # | Range (initial) | Scene | Camera | Light state | Copy (placeholder, intent only) |
|---|---|---|---|---|---|
| 01 | 0.00–0.14 | **Origin.** Macro silicon surface: wafer texture, a fine busbar line catching a slow raking light. Logo, one line of copy. | Locked macro, very slow push-in, slight pointer parallax | Low warm raking key, off-white fog | "Energy, engineered end to end." *(to be confirmed)* |
| 02 | 0.14–0.32 | **Manufacturing.** Cell → module: cells tile into a module, glass, encapsulant and frame layers settle on top (assembly as choreography), then modules move along a minimal line. | Rise from macro to a three-quarter view; one reveal through the frame edge | Controlled, even industrial top light | Manufacturing: label + one sentence |
| 03 | 0.32–0.50 | **Generation.** Module → array → landscape. Hold on scale. The busbar line exits the array as a grid line to a minimal substation volume. | Biggest move: pull-back and rise, the "breath" of the film | Open daylight, long soft shadows | Generation |
| 04 | 0.50–0.68 | **Data infrastructure.** The grid line enters a modular hall. Array rows re-form as rack rows (same instances). Cooling volumes as simple architecture. | Follow the line at a low angle into the hall, then lateral dolly down an aisle | Cooler neutral, structured light. **No blue glow.** | Data centres |
| 05 | 0.68–0.85 | **Recycling.** A module separates into labelled layers (glass, aluminium, silicon, copper, polymer) in clean diagrammatic space, then the materials re-stack. | Static frame, objects move instead: a calm counterpoint | Brightest, most spacious | Recycling / circularity |
| 06 | 0.85–1.00 | **System.** Pull back to an abstract campus: factory, array, hall and recovery facility joined by the one line, which closes into a loop. | Slow high pull-back, then rest | Balanced brand environment | Short close + 3 CTAs: Projects · About · Contact |

Every range lives in one config file (`experience.config.ts`). Copy lives in `content/home.ts`, so it's never hard-coded in a scene.

**Palette (until the brand guide arrives):** paper `#F1F0EC`, mist `#DCDDD9`, steel `#8C9298`, graphite `#2A2D30`, ink `#141618`, plus **one** brand accent, taken from the Convalt logo once supplied and used only on the line and CTAs. Light-led, not dark sci-fi.

---

## F. Technical architecture

### F.1 Structure
```
src/
  pages/                 index.astro, projects/[slug].astro, team.astro, …
  layouts/               BaseLayout (SEO head, header, footer)
  components/ui/         Header, Footer, Button, SectionHeading, ProjectCard, PressItem, ResourceRow, Form…
  content/               projects/, press/, team/, resources/  (Astro content collections, typed)
  data/                  home.ts (copy), navigation.ts, site.ts
  experience/            ← homepage only, loaded as a client island
    Experience.tsx       Canvas, quality tier, context-loss handling
    config/              experience.config.ts (scene ranges, camera keyframes, light states), quality.ts
    timeline/            useScrollProgress, masterTimeline.ts, range helpers (remap, smoothstep)
    rig/                 CameraRig.tsx (keyframed path + pointer parallax), Lighting.tsx
    scenes/              Origin/, Manufacturing/, Generation/, Data/, Recycling/, System/
    objects/             CellInstances.tsx (shared atom), ConductorLine.tsx, ModuleLayers.tsx
    materials/           shared material registry (pvGlass, brushedAluminium, silicon, matte…)
    shaders/             only where justified (e.g. wafer surface, line reveal), documented
    fallback/            Poster + CSS animation, WebGL detection
    debug/               dev-only HUD (fps, draw calls, tris, memory); stripped from prod
public/
  assets/ models/ textures/ hdr/ fonts/ images/
assets-src/              editable originals (.blend, PSD/Figma exports), not shipped
docs/                    this audit, ASSET-REGISTER.md, README sections
```

### F.2 Timeline model
`scroll (Lenis) → progress p ∈ [0,1] → master GSAP timeline (paused, scrubbed) → each scene gets local t = remap(p, start, end)`. The camera is one keyframed path (position, target, fov) authored in config. Each scene exports `update(t)` and reads no global state. **The scroll loop doesn't trigger React re-renders.** All per-frame work runs through refs in `useFrame`. DOM copy panels are positioned by the same timeline, so text and camera stay in sync.

### F.3 Scene lifecycle
- One `Canvas`, one renderer, one camera.
- A scene is **mounted** within `[start − preload, end + margin]`, **visible** only inside its range, and unmounted (with disposal) outside it, except for shared resources.
- Shared geometry and materials are created once in a registry and never disposed per scene.
- Scene 01 assets block the first frame. Scenes 02–06 load during idle time after the first frame. A scene whose assets aren't ready holds the previous composition instead of showing a pop.

### F.4 Asset strategy
- **Procedural first:** wafers, cells, modules, frames, arrays, racks, building volumes and the line are all built from instanced primitives and extrusions. That's sharp, tiny and fully art-directable.
- **GLB only if needed** (e.g. a substation silhouette or cooling units): modelled low-poly in Blender, exported to glTF with Meshopt compression and KTX2 textures. Originals go in `assets-src/`.
- **Textures:** few and small. One wafer normal/roughness set (≤1K, KTX2), one small studio HDR (≤1K, compressed), baked AO for static volumes where it helps.

---

## G. Performance strategy & budgets (homepage)

| Metric | HIGH | MEDIUM | LOW / mobile |
|---|---|---|---|
| DPR cap | 2.0 | 1.5 | 1.25 |
| Draw calls per frame | ≤ 120 | ≤ 80 | ≤ 50 |
| Triangles visible | ≤ 600k | ≤ 300k | ≤ 120k |
| Shadows | 1 shadow map, 2048 | 1024 | baked only |
| Post | tone map + light bloom + vignette | tone map only | none |
| Texture memory | ≤ 64 MB | ≤ 40 MB | ≤ 24 MB |
| 3D JS chunk (gzip) | ≤ 300 KB (three + r3f + gsap), lazy-loaded after the DOM hero paints | | |

- **LCP is DOM text** (headline plus logo), not the canvas, so first meaningful paint is fast even on a slow GPU.
- No per-frame allocations, no React state in the loop, instancing everywhere, and minimal transparency (layered glass uses one sorted pass).
- Runtime: `PerformanceMonitor` steps the tier down if fps stays below the threshold. After repeated decline it moves to the poster fallback.
- Profiling happens **every phase**, not at the end: draw calls and triangles via `renderer.info`, Chrome Performance traces, and real-device checks (see risk R3).

---

## H. Mobile & fallback strategy
- **Designed separately, not scaled down:** a portrait camera track per keyframe (wider fov, re-framed targets), copy at the top or bottom third with scene composition kept clear behind it, fewer instances (smaller array, fewer racks), and no post.
- **Tier selection:** `detect-gpu` tier, `deviceMemory`/`hardwareConcurrency` hints, `saveData`, and the WebGL2 capability check. Screen width only affects composition, never the quality tier.
- **Fallback** (no WebGL, context lost twice, persistent low fps, or user choice): stills **rendered from the real scenes** at each chapter, with a slow CSS pan/crossfade and the same DOM copy. The narrative stays intact.
- **Reduced motion:** Lenis is disabled, the camera **cuts** with crossfades between chapter compositions instead of flying, there's no parallax, and all copy remains.
- **Context loss:** listen for `webglcontextlost`/`webglcontextrestored`, try one restore, then fall back gracefully.

---

## I. Risks & missing inputs

| # | Risk | Impact | Mitigation / what I need |
|---|---|---|---|
| R1 | **convalt.com and mont-fort.com are blocked** by this environment's network policy | IA, copy and brand can't be audited; the benchmark can't be studied | Allow `convalt.com`, `www.convalt.com` and `mont-fort.com` in the environment's network settings, **or** send a content export, screenshots, or a screen recording of Mont-Fort. |
| R2 | No brand assets (logo, colours, fonts) | The prototype uses a neutral interim palette and a text wordmark | Supply the logo SVG and brand guide. |
| R3 | No GPU in the build container | Real fps can't be verified here | Prototype includes a dev HUD. Please test on your machine/phone. Optionally use a GPU CI runner later. |
| R4 | Public facts may be outdated (2022 SPAC filing; Watertown status) | Wrong claims on a corporate site | No figures go into copy without client confirmation. |
| R5 | Data-centre business line is the least documented | Scene 04 copy is thin | Need a short description of Convalt's data-centre offering. |
| R6 | SOW timeline (3 + 4 days) vs this level of quality | Schedule slip | Phase gates as planned: approve the Phase 1 prototype before full scope. Re-baseline the schedule after Phase 1. |
| R7 | Contact form backend unknown | Form can't submit | Need the endpoint/provider choice. Until then the form shows a clear "not configured" error, never a fake success. |
| R8 | Safari/iOS WebGL quirks (memory limits, DPR) | Crashes on iPhone | Conservative mobile budget, KTX2, test on real iOS early in Phase 2. |

---

## J. Implementation plan

| Phase | Scope | Gate |
|---|---|---|
| **1 — Visual prototype** | Scene 01 complete plus the transition into Scene 02, with loader, nav, quality tiers, fallback poster, reduced motion and mobile framing | **Your approval** |
| 2 — Homepage | Scenes 02–06, continuous transitions, asset optimisation, mobile track, fallback stills, QA | Your approval |
| 3 — Internal 2D site | Components and content collections derived from the approved homepage system, all pages, form, SEO | — |
| 4 — Final QA | Cross-browser, axe, Lighthouse, link check, console, production build, docs, asset register | Your approval → staging |

### Exact Phase 1 prototype plan
1. **Scaffold:** Astro + React island, TypeScript strict, ESLint/Prettier, design tokens (CSS vars), self-hosted interim fonts, base layout with SEO head, header (logo placeholder wordmark, nav, CTA), and a footer stub.
2. **Experience core:** `Experience.tsx` with WebGL2 detection, quality tier (`detect-gpu` + `PerformanceMonitor`), context-loss handling, Lenis + GSAP ScrollTrigger master timeline, `experience.config.ts` with ranges 01 and 02, and the CameraRig keyframe path plus damped pointer parallax.
3. **Scene 01 — Origin:** a procedural wafer/cell surface (instanced cells, a small shader for the silicon crystalline sheen, justified because textures alone alias badly at macro scale), the ConductorLine busbar with a drawn reveal, raking light state, and a minimal hero typography lock-up composed with the camera.
4. **Transition 01 → 02:** the camera rises, cells tile into a module, then glass, encapsulant and frame layers settle, ending on the first manufacturing composition. This proves the "same atom, transformed" continuity.
5. **Loader:** logo plus a real asset-progress line (driven by the loading manager, no fake percentages) that fades into the first frame. The DOM headline is visible before the canvas.
6. **Fallback and reduced motion:** a poster rendered from the scene plus a CSS treatment; reduced-motion cut/crossfade mode. Both reachable with `?quality=fallback` / `?motion=reduced` for review.
7. **Mobile:** a portrait camera track and copy placement for 390/430 widths.
8. **Measure:** dev HUD (draw calls, tris, textures, fps), Playwright screenshots at p = 0, 0.05, 0.10, 0.14, 0.20, 0.28 at 1440×900, 1920×1080, 1280×720 (short laptop), 768×1024 and 390×844, plus axe on the page. I'll report numbers, clearly separating SwiftShader results from real-GPU expectations.
9. **Self-review against §45 of the brief, fix, then stop and report.**

Nothing is deployed at any point until you explicitly approve.

---

*Sources for §A.3: [SEC 8-K Ex-99.1 (2022)](https://www.sec.gov/Archives/edgar/data/1848795/000119312522233466/d385074dex991.htm), [Gallup EDC press release](https://www.gallupedc.com/media/press-releases/p/item/67990/convalt-energy-enters-into-purchase-and-sale-agreement-with-gallup-land-partners-for-advanced-manufacturing-campus-in-new-mexico-and-up-to-1gw-of-behindthemeter-power-generation), [Solar Power World (Jun 2026)](https://www.solarpowerworldonline.com/2026/06/convalt-energy-agrees-to-buy-new-mexico-site-for-solar-manufacturing-campus/), [Data Center Map](https://www.datacentermap.com/c/convalt-energy/), [NNY360](https://www.nny360.com/communitynews/business/convalt-energy-lands-250-million-solar-project-in-laos/article_3aecb8e6-b1da-5bc6-9bc5-c7e2382e0d43.html).*
