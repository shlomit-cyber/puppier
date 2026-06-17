# Workfully Landing Page - Design Guidelines

## DO
- Light, minimal, clean, modern design — shadcn aesthetic
- Neutral color palette: whites, grays
- Primary color: green #309C8D
- Secondary colors: pink #C43E66, orange #DB8A16
- Good typography hierarchy, lots of whitespace
- Font: Inter from Google Fonts
- Mobile responsive
- Subtle motion: scroll animations via IntersectionObserver, smooth transitions, subtle hover
- Use Lucide icons (no emoji)

## DON'T
- No bright/saturated gradients or gradient text
- No emoji (use Lucide icons)
- No marketing copy like "revolutionary", "game-changer"
- No heavy shadows or busy patterns
- No dark mode
- No border-radius larger than rounded-2xl
- No tight spacing or fonts smaller than 14px

## Project structure & deploy

Static HTML site — **no build step, no framework**. Tailwind via CDN (`cdn.tailwindcss.com`, config inline in each file's `<head>`), Lucide icons via CDN, fonts: PauzaBook/PauzaLight (local `fonts/`) with Inter fallback. All persistent config (colors) is duplicated per-page in the inline `tailwind.config`.

- **Repo:** `github.com/shlomit-cyber/puppier` (remote `origin`). A second remote `workfully` points elsewhere — push to `origin` only.
- **Host:** Netlify site **`workfully-app`** (live at `workfully-app.netlify.app`; siteId `d1f66801-446f-4cdb-a66d-6fe6261c7547`). No custom domain.
- **Deploy:** Netlify CLI from the folder. `netlify deploy` = draft preview URL (use to review before going live); `netlify deploy --prod` = live. `publish = "."` (whole folder). The user is already authed (team `workfully`). Install once: `npm i -g netlify-cli`.

### Pages
- `index.html` — Hebrew, **RTL** (`dir="rtl"`). The canonical page.
- `en.html` — English, **LTR**. Content-parallel to `index.html`; keep the two in sync when changing copy or structure.
- `contact.html` — contact form. Others: `companies.html`, `terms.html`, `privacy.html`, `one-pager.html`, `dana-intro.html`.

### Contact form (contact.html)
Uses **Netlify Forms** (hidden detection form `name="contact"`; JS submits via `fetch('/')`). Submissions email **efrat@workfully.ai** via a `submission_created` email hook configured in the Netlify dashboard. Notes: form detection must be enabled in the dashboard (it was, + a redeploy registered the form); Netlify notification emails are **delayed several minutes** (expected). The JS uses `new FormData(form)` (not `form.name.value` — `name` collides with the form element's built-in property) and shows the error state on failure (no false "thank you").

## Interactive features (index.html + en.html, kept in sync)

Both pages share the same JS patterns; **RTL vs LTR differences are deliberate** — don't "fix" them by LTR intuition.

- **Scroll reveal + stagger** — `.fade-in` (opacity + translateY) revealed by an `IntersectionObserver`. Cards sharing a parent get incremental `transition-delay` (90ms × index) so grids cascade in. `prefers-reduced-motion` disables all of this.
- **Interactive timeline** ("How it works") — two numbered `.timeline-step` nodes connected by `.timeline-line`. On scroll the line **draws** (CSS `clip-path` reveal) and nodes light up (`.is-active`, teal then pink) staggered. The line is positioned **edge-to-edge between the circle perimeters** by JS (`positionTimelineLine()`, re-runs on resize) — not center-to-center. The line gradient (teal→pink) is an **intentional, approved exception** to the "no gradients" rule: it's a thin 3px connector in brand colors representing step 1→2. RTL draws right→left (`to left`, `clip-path: inset(0 0 0 100%)`); LTR draws left→right (`to right`, `inset(0 100% 0 0)`).
- **Count-up numbers** — `[data-count]` spans animate 0→target when scrolled into view (`animateCount`, easeOutCubic). `data-decimals="1"` for ratings (4.5), none → integer with thousands separator (11,000). Used in the methodology banner and the "results" band.
- **Results band** ("התוצאות מדברות" / "The Results Speak for Themselves") — pilot metrics after the methodology banner: career-clarity 2.7→4.5 (arrow nudged down `translate-y-[3px]` to sit on the digits' midline), 4.7/5 satisfaction, 4.6/5 recommend. Numbers are real pilot data.
- **Subtle parallax** — `[data-parallax="<speed>"]` translateY on scroll (rAF, reduced-motion-aware). Applied to the hero mockup. Put it on a non-`.fade-in` element to avoid transform conflicts.
- **Micro-interactions** — cards lift + shadow + icon scale on hover (`group`/`group-hover`); CTA buttons slide their arrow + `active:scale-95`; nav links have a `.nav-link` underline that grows (from right in RTL, left in LTR).
- **Animated chat mockup (en.html hero only)** — replaces the old static SVG. A chat window with Dana's avatar (`dana.png`, copied from the career app) that loops: Dana "types" (bouncing dots) → message appears → user types → reply, then restarts (`initChatDemo`, conversation array in the script). The Hebrew hero keeps its real video (`demo.mp4`) instead. Demo text is English; edit the `convo` array to change it.
