# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project
Personal portfolio for **Sasha Zherdeva**, a communications and short-form video storyteller with a 3D animation/VFX background, targeting social, content and communications roles. Its job is to get hiring managers to (1) read a case study and (2) click email or download the résumé. The site is also a writing sample: copy quality matters as much as code quality. Video is the primary medium; one Three.js hero scene is the signature moment.

Product and design decisions live in `docs/site-plan.md`. Follow it for page structure, components and build order.

## Stack
- **Astro** (static output only; no SSR, no server endpoints)
- **TypeScript** (strict)
- **Plain CSS** with custom properties (no CSS framework unless asked)
- **Content collections** (Markdown/MDX) for case studies and writing samples
- **GitHub Pages**, deployed by GitHub Actions on push to `main`
- Animation: CSS first; Astro View Transitions; **Three.js** for the single hero scene only (pre-approved dependency), loaded in an island. No other animation libraries without asking.

If an Astro API below differs from the installed version, follow the current Astro docs and the version in `package.json`.

## Commands
```bash
npm install
npm run dev       # local dev server
npm run build     # production build to dist/ — must pass before committing
npm run preview   # serve the built site
npx astro check   # type + content schema checks — must pass
```

## Structure
```
src/
  content.config.ts        # collection schemas
  content/
    work/                  # one .md/.mdx per case study
    writing/               # writing samples (press releases, articles, newsletters)
  components/              # small, reusable .astro components
    HeroScene.astro        # wrapper: poster image + mounts the Three.js island
    VideoFacade.astro      # click-to-play video/embed with poster
  scripts/
    hero-scene.ts          # Three.js scene (dynamically imported)
  layouts/
    BaseLayout.astro       # <head>, SEO/meta, nav, footer, ViewTransitions
    CaseStudyLayout.astro
  pages/
    index.astro            # home
    work/index.astro       # all case studies
    work/[slug].astro      # case study page
    writing.astro
    about.astro
    404.astro
  styles/
    global.css             # tokens (colour, type, spacing), resets, motion rules
  assets/                  # images processed by astro:assets
public/
  resume.pdf
  og-default.png           # 1200×630 share image
  CNAME                    # custom domain
  favicon.svg
```

## Content model
Case study front matter (enforce with a Zod schema in `src/content.config.ts`):

```ts
title: string            // "League of Mighty: monthly-giving campaign"
summary: string          // one line: what, for whom, outcome
role: string             // honest title/contribution, e.g. "Communications assistant"
org: string
period: string           // "May–Aug 2024"
tags: string[]           // e.g. ["campaign", "social", "media relations"]
cover: image()           // via astro:assets
coverAlt: string
metric?: { value: string; label: string }   // "+304%", "revenue growth YoY"
featured: boolean        // show on home (max 3)
order: number            // sort order
draft: boolean           // exclude from build when true
```

Body sections, in order: **Challenge · My role · Approach · Work samples · Results · What I learned** (last is optional).

### Content rules for agents
- **Never invent** metrics, quotes, testimonials, clients, or outcomes. Use `[TODO: …]` placeholders and list them in your summary.
- Don't change the owner's job titles or overstate contribution.
- Flag anything that looks confidential or shows identifiable private individuals.
- Canadian spelling. Active voice. No filler adjectives.

## Design rules
- Mobile-first. Test at 375px, 768px, 1280px. No horizontal scroll.
- Readable body text: ≥ 18px, line length 60–75ch, strong contrast (WCAG AA minimum).
- Consistent spacing/type scale from tokens in `global.css`; no one-off magic numbers.
- Every page's top section must state who the owner is and what they do within 5 seconds of reading.
- Primary CTAs (email, résumé) visible on home without scrolling and in every footer.

## Animation rules
- Motion must support reading, never block it. Content is visible and usable with JS disabled.
- Allowed: subtle scroll reveals (opacity/translate ≤ 24px, 300–600ms), hover/focus states, View Transitions between cards and case study headers, the Three.js hero scene (see below), and one stat counter.
- Animate only `transform` and `opacity`. Use `IntersectionObserver` or CSS scroll-driven animations; no scroll listeners doing layout work.
- **Not allowed:** loading screens, scroll-jacking, custom cursors replacing the pointer, autoplaying audio, text that only appears after long animations.
- Always honour reduced motion:
  ```css
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
  }
  ```
  JS animations must check `matchMedia('(prefers-reduced-motion: reduce)')` and skip.
- Keep client JS small: prefer zero-JS components; hydrate islands with `client:visible`.

## Three.js hero scene
One scene, on the home page hero only. It decorates; it never carries content.
- **Poster first:** render a static image (`astro:assets`) in the hero's space at the final size, so there's no layout shift. The canvas fades in over it once ready.
- **Load late:** dynamically `import('three')` inside the island after the page is idle or visible (`client:visible` / `requestIdleCallback`). Import only what's used from `three` (tree-shakeable named imports); no full examples bundles.
- **Skip entirely** (keep the poster) when: `prefers-reduced-motion: reduce`, WebGL is unavailable, `navigator.connection?.saveData` is true, or the device looks low-power (e.g., `navigator.hardwareConcurrency <= 4` on narrow screens). Never show an error.
- **Budgets:** added JS ≤ ~200 KB gzipped for the hero island; models/textures ≤ 500 KB total (compressed glTF/Draco or procedural geometry); steady 60 fps on a mid-range phone; renderer pixel ratio capped at `Math.min(devicePixelRatio, 2)` (1.5 on mobile).
- **Be a good citizen:** pause the render loop when the canvas is off-screen (`IntersectionObserver`) or the tab is hidden (`visibilitychange`); resize on container resize, not every frame.
- **Clean up on navigation:** with View Transitions, dispose renderer, geometries, materials and textures and cancel the animation frame on `astro:before-swap`; re-init on `astro:page-load` only when the hero exists.
- **Accessibility:** canvas gets `aria-hidden="true"`; headline, pitch and CTAs are real HTML layered above it with sufficient contrast; any pointer interaction is optional flair, never required.
- Interaction (e.g., gentle cursor/scroll parallax) is fine; no click-to-explore mechanics that hide content.

## Video
- Short-form Reels are 9:16: use a dedicated vertical layout (e.g., max-width ~360px, grid of 2–3 on desktop, swipeable row on mobile). Landscape video uses 16:9.
- Use `VideoFacade.astro`: poster image + play button; load the real `<video>` or YouTube/Vimeo iframe only on click. Never autoplay with sound.
- Self-hosted clips: H.264 MP4 (plus WebM if smaller), ≤ ~5 MB each, `preload="none"`, `playsinline`, captions (`<track kind="captions">`) where speech is present.
- Every video has a text caption stating what it is and its result (e.g., "Reel · 12 shares after hook re-edit").

## Accessibility
- Semantic HTML: one `<h1>` per page, logical heading order, `<nav>`, `<main>`, `<footer>`.
- All images need meaningful `alt` (decorative images: `alt=""`).
- Visible focus styles; everything reachable by keyboard; link text makes sense out of context.
- Embedded video (YouTube) via lightweight facade or `loading="lazy"` iframe with a `title`.

## SEO & sharing
`BaseLayout` accepts `title`, `description`, `image` and must output:
- Unique `<title>` in the format `Page — Sasha Zherdeva, Communications`
- `<meta name="description">`, canonical URL, Open Graph + Twitter card tags, share image
- `sitemap` via `@astrojs/sitemap`; `robots.txt` allowing indexing
- Never add `noindex` to production pages.

## Performance
- All raster images through `astro:assets` (`<Image />` / `<Picture />`), with width/height set, lazy-loaded below the fold, AVIF/WebP output.
- Self-host fonts or use at most two font families; `font-display: swap`.
- Target Lighthouse ≥ 90 for Performance, Accessibility, Best Practices, and SEO.

## Deployment
- `astro.config.mjs`: set `site: 'https://<custom-domain>'`. Only set `base` if deploying to a `username.github.io/repo` path without a custom domain; if you do, build all internal links and asset paths with `import.meta.env.BASE_URL`.
- Deploy with the official Astro GitHub Pages workflow (`.github/workflows/deploy.yml` using `withastro/action`); Pages source = GitHub Actions.
- Keep `public/CNAME` containing the custom domain.

## Definition of done (every change)
1. `npm run build` and `npx astro check` pass with no errors or warnings.
2. Pages render correctly on mobile and desktop widths; no console errors.
3. Reduced-motion mode verified for any new animation; the hero works (poster only) with JS disabled and with WebGL unavailable.
4. New pages have title, description, and share image.
5. No broken internal links; external links open normally (no forced `target="_blank"` unless opening files).
6. Summarize what changed, files touched, and any `[TODO]` placeholders left.

## Working style
- Make small, focused changes; don't refactor unrelated code.
- Ask before adding dependencies (Three.js and `@astrojs/sitemap` are pre-approved). Prefer built-in Astro features.
- Build order: layout + SEO → case study pages → video components → home → Three.js hero last.
- Don't edit `src/content/**` copy beyond what was asked; propose copy changes separately.
- Commit messages: imperative, short (e.g., "Add case study layout").
