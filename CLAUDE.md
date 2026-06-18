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

- **Repo / two remotes:** the local folder is git-linked to **two** remotes — `origin` = `github.com/shlomit-cyber/puppier` (feeds the Netlify preview) and `workfully` = `github.com/uidb-dev/workfully.co.il.git` (feeds the LIVE public site, managed by an external dev). Their `main` branches are normally identical — **push to both**.
- **The live public site is `workfully.co.il`** (nginx, deployed from the `workfully` remote). A `git push workfully main` **auto-deploys it in ~90 seconds**. `workfully-app.netlify.app` (siteId `d1f66801-446f-4cdb-a66d-6fe6261c7547`, no custom domain) is **only a preview/parallel copy** fed by `origin` + `netlify deploy` — NOT the public site. `workfully.ai` is a THIRD thing entirely: the actual Workfully product app (Laravel/nginx, login/intro/today) — it never shows landing-page changes.
- **To ship a change:** iterate with `netlify deploy` (draft preview URL, no `--prod`), then go live with **`git push workfully main`** (+ `git push origin main` to keep the preview/repo in sync; `netlify deploy --prod` optional, only updates the preview). `publish = "."`. User is already authed (Netlify team `workfully`; `npm i -g netlify-cli` once). **Pushing to Netlify/origin alone leaves workfully.co.il unchanged — this caused real confusion once.**

### Pages
- `index.html` — Hebrew, **RTL** (`dir="rtl"`). The canonical page.
- `en.html` — English, **LTR**. Content-parallel to `index.html`; keep the two in sync when changing copy or structure.
- `contact.html` — contact form. Others: `companies.html`, `terms.html`, `privacy.html`, `one-pager.html`, `dana-intro.html`.

### Contact form (contact.html)
Uses **Web3Forms** — the JS POSTs `new FormData(form)` (+ `access_key`, `subject`, `from_name`) to `https://api.web3forms.com/submit` and emails **efrat@workfully.ai** (the address the Web3Forms account/key is tied to). **Why not Netlify Forms:** Netlify Forms only works on Netlify-hosted sites; the live site is nginx (`workfully.co.il`), so Netlify Forms silently dropped every real submission — only the Netlify preview ever captured anything. Web3Forms is host-agnostic, so it works on the live site. Notes: the access key is public-by-design (client-side). Web3Forms free tier accepts **browser submissions only** (Cloudflare-protected) — you can't test it with server-side curl; test from a real browser and check efrat's inbox (incl. spam). `new FormData(form)` avoids the `form.name` collision; failure shows the error state (no false "thank you").

## Interactive features (index.html + en.html, kept in sync)

Both pages share the same JS patterns; **RTL vs LTR differences are deliberate** — don't "fix" them by LTR intuition.

- **Scroll reveal + stagger** — `.fade-in` (opacity + translateY) revealed by an `IntersectionObserver`. Cards sharing a parent get incremental `transition-delay` (90ms × index) so grids cascade in. `prefers-reduced-motion` disables all of this.
- **Interactive timeline** ("How it works") — two numbered `.timeline-step` nodes connected by `.timeline-line`. On scroll the line **draws** (CSS `clip-path` reveal) and nodes light up (`.is-active`, teal then pink) staggered. The line is positioned **edge-to-edge between the circle perimeters** by JS (`positionTimelineLine()`, re-runs on resize) — not center-to-center. The line gradient (teal→pink) is an **intentional, approved exception** to the "no gradients" rule: it's a thin 3px connector in brand colors representing step 1→2. RTL draws right→left (`to left`, `clip-path: inset(0 0 0 100%)`); LTR draws left→right (`to right`, `inset(0 100% 0 0)`).
- **Count-up numbers** — `[data-count]` spans animate 0→target when scrolled into view (`animateCount`, easeOutCubic). `data-decimals="1"` for ratings (4.5), none → integer with thousands separator (11,000). Used in the methodology banner and the "results" band.
- **Results band** ("התוצאות מדברות" / "The Results Speak for Themselves") — pilot metrics after the methodology banner: career-clarity 2.7→4.5 (arrow nudged down `translate-y-[3px]` to sit on the digits' midline), 4.7/5 satisfaction, 4.6/5 recommend. Numbers are real pilot data.
- **Subtle parallax** — `[data-parallax="<speed>"]` translateY on scroll (rAF, reduced-motion-aware). Applied to the hero mockup. Put it on a non-`.fade-in` element to avoid transform conflicts.
- **Micro-interactions** — cards lift + shadow + icon scale on hover (`group`/`group-hover`); CTA buttons slide their arrow + `active:scale-95`; nav links have a `.nav-link` underline that grows (from right in RTL, left in LTR).
- **Animated chat mockup (both heroes)** — replaces the old hero visuals (en.html's static SVG and index.html's `demo.mp4` laptop video — `demo.mp4` is now unused). A chat window with Dana's avatar (`dana.png`, copied from the career app) that loops: Dana "types" (bouncing dots) → message appears → user types → reply, then restarts (`initChatDemo`, `convo` array in each page's script — edit it to change the text). RTL vs LTR differ: bubble tails mirror (Dana right in HE / left in EN) and the `convo` text is Hebrew vs English. `prefers-reduced-motion` shows the full conversation statically.
