# Portfolio Site Plan (v1, 2026-09-29)

## Design direction
- **Look:** Luna (lunatemplate.framer.website): calm, big type, lots of white space, 3-item nav.
- **Case study structure:** Bohdana (portfolio-2026-six-tan.vercel.app): fact panel, "How might we" challenge, constraints, a pivot, reflection. Add a results strip, which she lacks.
- **Palette:** off-white background, near-black ink, one bright accent (moss green or lake blue), one warm pop for stickers and highlights. Dark mode is optional and comes later.
- **Type:** one bold grotesk for display and a clean sans for body, both self-hosted. Display sizes are huge; body text is at least 18px (per AGENTS.md).
- **Motion rule:** one "fun" element per page. Home gets the hero video loops. Case studies get a stat count-up. Everything else is a subtle fade. Honour `prefers-reduced-motion` everywhere.

## Sitemap
| URL | Purpose |
|---|---|
| `/` | Home |
| `/work/` | All case studies (3–5) |
| `/work/swan-lake/`, `/work/uvic/`, `/work/prince-rupert-port/`, `/work/bc-parks-foundation/` | Case studies |
| `/writing/` | Writing samples |
| `/motion/` | Motion & 3D |
| `/about/` | About |
| `/resume.pdf` | Résumé |
| `/404` | Friendly 404 |

Nav: **Work · Writing · About** plus an **Email me** button. Motion & 3D is linked from Home and the footer.

## Home (top to bottom)
1. Hero sentence with inline 9:16 video loops. Posters show when reduced motion is on.
2. Proof line (Swan Lake watch time) plus Email and Résumé buttons.
3. Three featured case cards, each showing name, one outcome line and a poster.
4. Marquee: "Where my work has run". Plain-text org names, no logos without permission.
5. Motion & 3D teaser: one strip with 2–3 stills or loops, linking to /motion.
6. Footer CTA: "Get in touch" marquee (mailto), LinkedIn, Instagram, résumé.

## Case study template
1. Title plus one-line outcome
2. Fact panel: Organisation · My role · Team · Tools · Timeline
3. Results strip: the top 3 numbers
4. Challenge: one "How might we…" line, then the audience
5. Constraints
6. Approach: decisions, plus one change of course
7. Work samples: Reel facades, before/after pairs, excerpts
8. What I learned: two sentences
9. Next case study link

## Other pages
- **Writing:** 6–9 short excerpts grouped as Social captions / Blog & web / Internal comms, each with context and a link.
- **Motion & 3D:** a small grid (ICON Creative Studio, UCalgary lab, VFS). One line per piece on how the skill serves comms.
- **About:** photo, 3 short paragraphs, "Currently: co-op at Mellenger Interactive", contact.

## Tech architecture (Astro, static, GitHub Pages)
- Content collection `work` (MDX), using the schema in AGENTS.md (`draft: true` keeps uncleared work out of the build). Proposed additions for the fact panel and results strip: `team?: string`, `tools?: string[]`, `stats?: {value, label}[]` (up to 3).
- Collection `writing` (MD): `title, type, org, excerpt, link, date`.
- Components: `BaseLayout` (title/description/OG), `Nav`, `Footer`, `HeroSentence`, `InlineLoop`, `CaseCard`, `FactPanel`, `StatStrip`, `VideoFacade` (poster, click to load), `ReelGrid` (9:16), `BeforeAfter`, `Marquee`.
- Video: self-hosted short muted loops (webm + mp4, under 500 KB each) for the hero. Reels play through a click-to-play facade.
- View Transitions between card and case study title.
- Three.js hero: phase 7 only, lazy-loaded, static fallback.

## Build phases
0. **Content & permissions:** Swan Lake interview, asset gathering, permission checks.
1. **Skeleton:** repo, deploy pipeline, design tokens, layout, nav/footer, SEO basics.
2. **Swan Lake end-to-end:** proves the template and components.
3. **Home:** hero, cards, marquee, footer CTA.
4. **Remaining case studies:** UVic, Prince Rupert Port Authority, BC Parks Foundation.
5. **Writing, Motion & 3D, About, Résumé**
6. **QA:** mobile, reduced motion, alt text, links, titles/descriptions, Lighthouse, typos.
7. **Three.js hero**
8. **Mellenger case study:** only once cleared.

## Permissions checklist
- [ ] Swan Lake Indigenous-centred content: confirm it can be shown
- [ ] Identifiable members of the public in any footage (hero loops especially)
- [ ] Logos: use plain text names unless approved
- [ ] Mellenger: NDA, hold until cleared
- [ ] PRPA SharePoint blog: internal, so confirm excerpts can be shared or paraphrase them

## To reconcile with AGENTS.md
- **Hero:** this plan puts video loops in the hero sentence; AGENTS.md reserves the hero for the Three.js scene. Options: loops now, Three.js replaces them in phase 7; or loops elsewhere on Home.
- **Marquees:** not on the AGENTS.md list of allowed motion. Either add "one slow CSS marquee, static under reduced motion" to the rules, or make them static rows.
- **Motion & 3D page:** add `pages/motion.astro` to the AGENTS.md structure.

## Open decisions
- Accent colour (moss vs lake blue)
- Hero loop layout (inline between words vs one larger phone frame)
- Domain name
