# Technical, mobile UX, accessibility and SEO standards for a top-quality agency marketing site (as of Nov 2026)

Scope note: sources below were fetched this session. Where a topic could only be answered from secondary or no sources it is flagged in Gaps. Several primary pages (web.dev/vitals, MDN reduced-motion, WCAG 2.2.2/1.4.10/2.3.3) were not fetchable, so those items are listed in Gaps, not asserted.

## 1. Core Web Vitals: thresholds, causes of failure, and effects of video, fonts and animation

### Takeaway
Pass requires LCP <= 2.5 s, INP <= 200 ms, CLS <= 0.1, all at the 75th percentile of real page loads. For a no-image, CSS-animated single page the realistic risks are font loading (LCP and CLS), main-thread JS (INP) and any future video or poster.

### Cited Findings
- Thresholds (75th percentile): LCP good <= 2500 ms, poor > 4000 ms; INP good <= 200 ms, poor > 500 ms; CLS good <= 0.1, poor > 0.25. The "needs improvement" band is the range between good and poor — [web.dev: How CWV thresholds were defined](https://web.dev/articles/defining-core-web-vitals-thresholds)
- INP is measured at the 75th percentile of page loads, segmented mobile vs desktop; it has three parts: input delay, processing duration, presentation delay — [web.dev: Optimize INP](https://web.dev/articles/optimize-inp)
- INP fixes: reduce script load/parse work during load, keep event callbacks light, run only render-critical logic in the callback and defer the rest, reduce DOM size, use `content-visibility` for off-screen content, avoid layout thrashing; "yielding indiscriminately is better than not yielding at all" — [web.dev: Optimize INP](https://web.dev/articles/optimize-inp)
- LCP subpart guide (approximate, not absolute ms): TTFB ~40%, resource load delay <10%, resource load duration ~40%, element render delay <10% — [web.dev: Optimize LCP](https://web.dev/articles/optimize-lcp)
- Never lazy-load the LCP image (always causes resource load delay); use `fetchpriority="high"` on the likely LCP image, on one or two images at most; LCP resource should be discoverable in initial HTML (JS-injected, `data-src` lazy and CSS background images are hard to discover) — [web.dev: Optimize LCP](https://web.dev/articles/optimize-lcp)
- Common failure causes (practitioner checklist, secondary): lazy-loaded LCP image, CSS background as LCP, big video as LCP element competing for bandwidth, missing video `poster`, late/unmatched web fonts, too many font weights, long JS tasks, heavy third-party scripts, banners inserted at top of page, images/widgets without reserved space — [corewebvitals.io checklist](https://www.corewebvitals.io/core-web-vitals/utlimate-checklist)
- Fonts and CLS: both invisible text and swap-in text can shift layout. Use `font-display: optional` so the web font only appears if ready at first layout, specify a close fallback, tune fallback with `size-adjust`, `ascent-override`, `descent-override`, `line-gap-override`, and preload critical fonts — [web.dev: Optimize CLS](https://web.dev/articles/optimize-cls)
- Font blocking: Chromium and Firefox block text up to 3 s, Safari indefinitely, before showing fallback; blocking can delay FCP and sometimes LCP — [web.dev: CSS for Web Vitals](https://web.dev/articles/css-web-vitals)
- Animation and CLS: animating `top`, `left`, `margin`, `box-shadow` triggers layout; use `transform`, `opacity`, `filter`. Composited transform animations do not count toward CLS — [web.dev: Optimize CLS](https://web.dev/articles/optimize-cls); [web.dev: CSS for Web Vitals](https://web.dev/articles/css-web-vitals)
- Reserve space with `width`/`height` attributes or CSS `aspect-ratio` for images, video, embeds, iframes; keep aspect ratio consistent across `srcset` variants — [web.dev: Optimize CLS](https://web.dev/articles/optimize-cls)
- Video: always set `poster`; a large in-viewport video can become the LCP element; compress and size to display dimensions — [corewebvitals.io checklist](https://www.corewebvitals.io/core-web-vitals/utlimate-checklist)
- Lab emulation: Lighthouse mobile preset = "Slow 4G", RTT 150 ms, 1.6 Mbps down / 750 Kbps up, 4x CPU slowdown; described as roughly bottom 25% of 4G / top 25% of 3G. Mid-tier mobile reference device is Moto G4 (BenchmarkIndex ~125–800) — [Lighthouse throttling docs](https://raw.githubusercontent.com/GoogleChrome/lighthouse/HEAD/docs/throttling.md)

### Inferences
- Novaframe loads three Google Fonts families (Bricolage Grotesque opsz 500/700, DM Sans 400/500/700, JetBrains Mono 500) with `display=swap` in a render-blocking stylesheet. That is about 6 weights/styles from a third-party origin; with no images the LCP element is almost certainly hero text, so font load time maps directly to LCP and swap shift to CLS. Audit: measure LCP element in DevTools; consider `font-display: optional` or fallback metric overrides, and fewer weights.
- The hero animation `.bar b` uses `transform` with `animation: fill` (seen in file), which is the composited-safe pattern; audit any new scroller code for layout-triggering animation (`scroll-behavior`, `left`, `width`).
- Any future video should carry a `poster`, explicit aspect-ratio, `preload="none"` and not be the LCP element.
- Third-party fonts are the main outside dependency; because the host controls the head, self-inlining fonts as base64 is likely impractical (size), so reduce weights instead.

### Gaps
- web.dev/articles/vitals (canonical thresholds page) was not fetchable; thresholds are confirmed via the "defining thresholds" article instead. I did not verify whether Google has changed any metric set in 2026; no source found indicating a change to the three thresholds.
- No source for the "50 ms long task" figure was found in the fetched INP page (page does not state it).
- No India-specific CrUX/HTTP Archive numbers retrieved (see section 5).

## 2. WCAG 2.2 AA: what single-page sites often miss

### Takeaway
WCAG 2.2 adds AA criteria for Focus Not Obscured (Minimum), Dragging Movements, Target Size (Minimum 24x24 CSS px) and Accessible Authentication (Minimum). Focus Appearance (2.4.13) is AAA, not AA. Sticky headers/CTAs, small icon controls, low-contrast borders and custom focus styles are the typical failures.

### Cited Findings
- WCAG 2.2 adds 9 criteria: 2.4.11 Focus Not Obscured (Minimum) AA; 2.4.12 Enhanced AAA; 2.4.13 Focus Appearance AAA (2 CSS px thick perimeter, 3:1 change of contrast); 2.5.7 Dragging Movements AA; 2.5.8 Target Size (Minimum) AA; 3.2.6 Consistent Help A; 3.3.7 Redundant Entry A; 3.3.8 Accessible Authentication (Minimum) AA; 3.3.9 Enhanced AAA. 4.1.1 Parsing was removed — [W3C: What's New in WCAG 2.2](https://w3.org/WAI/standards-guidelines/wcag/new-in-22/)
- 2.4.11: a focused component must not be entirely hidden by author-created content (e.g. a sticky header or cookie bar) — [W3C: What's New in WCAG 2.2](https://w3.org/WAI/standards-guidelines/wcag/new-in-22/)
- 2.5.8 Target Size (Minimum): pointer targets at least 24x24 CSS px (a 24 px square fits inside the target). Exceptions: spacing (24 px diameter circle centered on each undersized target must not intersect another target or circle), equivalent control elsewhere on page, inline in a sentence/line-height-constrained, user-agent-controlled, essential. 44x44 is the AAA criterion 2.5.5 — [W3C Understanding 2.5.8](https://w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)
- 2.5.7 Dragging Movements: any dragging function needs a single-pointer alternative (relevant to swipe/drag carousels: provide buttons) — [W3C: What's New in WCAG 2.2](https://w3.org/WAI/standards-guidelines/wcag/new-in-22/)
- 1.4.3 Contrast (Minimum): 4.5:1 normal text; 3:1 large text (>= 18 pt or 14 pt bold, ~24 px / ~18.5 px bold). Ratios are not rounded (4.499 fails). Placeholder text and hover/focus text must also meet it. Logotypes and disabled controls exempt. Specifying text color without background (or vice versa) is a failure — [W3C Understanding 1.4.3](https://w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)
- 1.4.11 Non-text Contrast: 3:1 for UI component boundaries needed to identify them, state indicators and focus indicators (unless browser default focus untouched). Examples: #AAA on white (2.3:1) fails; #767676 passes; checkbox border #9D9D9D (2.7:1) fails. Where a border is the only identifier of a text input it must meet 3:1 — [W3C Understanding 1.4.11](https://w3.org/WAI/WCAG22/Understanding/non-text-contrast)
- Form labels: visible `<label for>` association is the baseline pattern (see the Novaframe markup; no spec fetched this session, see Gaps).

### Inferences
- Novaframe has `:focus-visible{outline:2px solid var(--cobalt);outline-offset:3px}` and an input variant — good. Audit that `--cobalt` reaches 3:1 against both light and dark backgrounds, and that the sticky nav (if any) does not cover focused elements when tabbing/anchor-jumping (use `scroll-margin-top` / `scroll-padding-top`).
- Light/dark theming doubles the contrast audit: every token pair must be tested in both themes, including muted/secondary text, JetBrains Mono small labels and input borders.
- A team scroller needs prev/next buttons (2.5.7) at least 24x24 px (2.5.8), ideally 44 px.
- File grep found no `prefers-reduced-motion` rule in novaframe.html; infinite hero animations should be disabled/paused under it (see Gaps for sources).

### Gaps
- Not fetched (pages blocked): WCAG 2.2.2 Pause/Stop/Hide, 2.3.3 Animation from Interactions, 1.4.10 Reflow (320 px), 1.4.4 Resize Text, 3.3.2 Labels or Instructions, 1.3.5 Identify Input Purpose, 2.1.1/2.4.7 Keyboard/Focus Visible, MDN `prefers-reduced-motion`. These are standard AA/AAA criteria but I could not cite them from fetched sources; the report writer should treat them as unverified-in-session and cite w3.org/WAI/WCAG22/Understanding/ if used.
- No source fetched for Safari/Android real-device reduced-motion behavior.

## 3. On-page SEO for a local-service agency

### Takeaway
Google gives no fixed length limits for titles or descriptions (truncation depends on device width); write a distinct, descriptive title and a page-summarizing description. Structured data is optional for ranking; Organization has no required properties, LocalBusiness requires name and address. Because the host controls the head, confirm which tags actually render before relying on them.

### Cited Findings
- Title: include a `<title>` on every page; be descriptive and concise; Google has no length limit but truncates to device width; avoid keyword stuffing; brand concisely (site name on home page); make the main title (first visible `<h1>`, largest text) clear; match page language/script — [Google: title links](https://developers.google.com/search/docs/appearance/title-link)
- Google builds title links from `<title>`, main visual title, headings, `og:title`, anchor text, `WebSite` structured data; it rewrites half-empty titles (e.g. only a site name), inaccurate titles, micro-boilerplate, unclear main title, language mismatch, and duplicated site name; changes take days to weeks to show — [Google: title links](https://developers.google.com/search/docs/appearance/title-link)
- Meta description: Google mostly builds snippets from page content, sometimes uses the meta description; "no limit on how long" but truncated to device width; put important info first; write unique, readable descriptions; avoid keyword lists — [Google: snippets](https://developers.google.com/search/docs/appearance/snippet)
- Canonical: use absolute URLs, include a self-referential canonical on the canonical page, only accepted in `<head>`, choose either HTML link or HTTP header, avoid JS-injected canonical — [Google: consolidate duplicate URLs](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls)
- Favicon: `<link rel="icon">` in the home page head (also accepts `shortcut icon`, `apple-touch-icon`); square 1:1, at least 8x8 px, larger than 48x48 recommended; formats BMP/GIF/ICO/PNG/JPEG/PPM/TIFF; Googlebot and Googlebot-Image must be able to crawl home page and icon (not blocked by robots.txt); stable URL; one per hostname; no guarantee it shows — [Google: favicon in search](https://developers.google.com/search/docs/appearance/favicon-in-search)
- Organization structured data: no required properties; recommended `name`, `alternateName`, `legalName`, `description`, `foundingDate`, `address`, `contactPoint`, `email`, `telephone`, `url`, `logo` (at least 112x112 px, crawlable), `sameAs`, identifiers (`taxID`, `vatID`, `duns` etc.), `numberOfEmployees`. Put on home page or one about page, not every page. Use most specific subtype. No guarantee of visible result; guideline violations can trigger manual action — [Google: Organization structured data](https://developers.google.com/search/docs/appearance/structured-data/logo)
- LocalBusiness structured data: required for rich result eligibility `address` (PostalAddress, as many fields as possible) and `name`; recommended `geo` (5+ decimals), `openingHoursSpecification`, `priceRange` (<=100 chars), `telephone` (with country and area code), `url`, `department`; `aggregateRating`/`review` recommended only for sites aggregating other local businesses. The page does not address service-area businesses — [Google: LocalBusiness structured data](https://developers.google.com/search/docs/appearance/structured-data/local-business)
- Open Graph / Twitter Card sizing guidance (1200x630 og:image, `summary_large_image`) appears only in secondary 2026 guides found in search, not a primary spec fetched — [DEV: OG image sizes 2026 guide](https://dev.to/grabbit/open-graph-image-sizes-and-dimensions-the-complete-2026-guide-1k16) (unverified against ogp.me)

### Inferences
- "Overhyped": meta keywords (not covered by any Google source fetched), exact character counts as ranking factors (Google states no limit; the 50–60 / 150–160 char rules are display heuristics, not Google rules), schema as a ranking boost (Google says no guarantee of visible results). Worth doing: unique title containing brand plus service plus city, one `<h1>`, Organization (or `ProfessionalService`/LocalBusiness subtype) on the home page with `name`, `url`, `logo`, `sameAs`, `address`/`telephone`.
- For a new agency, Google Business Profile and consistent name/address/phone off-site are likely larger local-pack levers than on-page schema (no source fetched; treat as practitioner judgment).
- Current title is just "Novaframe": likely flagged as a half-empty title by Google's rewrite logic; no meta description was found in the file's visible head (host may inject its own).
- Hosted page: sitemap.xml, robots.txt and canonical may be impossible to control; confirm with the host what URL gets indexed and whether the host emits canonical. A `<meta name="claude:viewport-fit-sides">` tag exists in the head, indicating the host recognizes some meta tags.

### Gaps
- No source fetched for robots.txt and sitemap.xml specifics (Google "basic search visibility checklist" and sitemap docs surfaced in search but were not fetched).
- No primary source for Open Graph protocol (ogp.me) or X/Twitter card docs; required OG tags (`og:title`, `og:type`, `og:image`, `og:url`) are standard but not cited here.
- No Service schema documentation fetched; Google has no Service rich result, so `Service` markup would be semantic-only (unverified).
- No source on whether Google AI features/answer engines use structured data differently in 2026.

## 4. Video, and horizontally scrolling team/card carousels

### Takeaway
Video: poster plus `preload="none"` plus lazy loading, muted/inline for any autoplay, and never as LCP. Carousel: a native scroll-snap container with native prev/next buttons, a labelled region, native-focusable content, and no `aria-live` on scroll-paginated lists.

### Cited Findings
- Always set a `poster`; use a poster preloaded with `fetchpriority="high"` if the video is an LCP candidate; `preload="none"` stops fetching media data even in viewport (default `metadata` may still download more than needed); `loading="lazy"` on video where supported, with IntersectionObserver fallback; `autoplay muted loop playsinline` replaces GIFs (`playsinline` needed on iOS); don't lazy-load LCP videos — [web.dev: Lazy loading video](https://web.dev/articles/lazy-loading-video)
- Chrome team accessible carousel guide (Sept 2025): choose the pattern first (single-item, paginated, list-based, multi-item); `scroll-snap-type` on container and `scroll-snap-align` on items; wrap in `role="region"` with descriptive `aria-label`; `aria-live="polite"` only for single-item slideshows — on paginated carousels it can announce every item during slow scrolling; for lists don't use `inert` so item count stays available to screen readers; native CSS `::scroll-marker`/`::scroll-button` shipped in Chrome 135, counters in marker alt text from Chrome 140, other-browser support not covered; "no UI can be guaranteed to be accessible out of the box" — [Chrome for Developers: accessible carousel](https://developer.chrome.com/blog/accessible-carousel)
- W3C APG carousel pattern: container `role="region"` or `group` with `aria-roledescription="carousel"` and an accessible name (without the word "carousel"); each slide `role="group"` with `aria-roledescription="slide"` and name (without "slide", or "3 of 10" if no unique names); previous/next as native buttons that do not move focus; rotation control as native button first in tab order, labelled by action; auto-rotation stops on focus or hover and resumes only on explicit activation; live region `aria-live="off"` while rotating and `"polite"` when not; button pickers use `aria-disabled` not `disabled` for the current slide — [W3C APG: Carousel pattern](https://w3.org/WAI/ARIA/apg/patterns/carousel)
- Dragging alternative (2.5.7): provide single-pointer alternative to swipe/drag — [W3C: What's New in WCAG 2.2](https://w3.org/WAI/standards-guidelines/wcag/new-in-22/)
- Hidden/offscreen carousel items can use `fetchpriority="low"` for images that load early but aren't visible — [web.dev: Optimize LCP](https://web.dev/articles/optimize-lcp)

### Inferences
- Recommended build for a people-card row (not auto-rotating): `<section aria-labelledby="team-h">` containing a `<ul>` with `overflow-x:auto; scroll-snap-type:x mandatory` (or `proximity`), `<li>` cards with `scroll-snap-align:start`, `scroll-padding` for gutters, `tabindex="0"` plus `aria-label` on the scroller only if it contains no focusable content (so keyboard users can arrow-scroll), prev/next native buttons (>= 24 px, ideally 44 px) calling `scrollBy` with `behavior` set to `auto` under `prefers-reduced-motion: reduce`, buttons disabled/`aria-disabled` at ends, a visible partial next card (peek) as affordance, and a text position indicator such as "Showing 2 of 6" in a visually-hidden `aria-live="polite"` element updated on scroll end (debounced), or none if the list semantics already convey count. Because it is not auto-rotating, no pause control is needed.
- Hide scrollbar only if prev/next buttons and a peek exist; otherwise leave scrollbar visible. Horizontal scrolling of a carousel is generally acceptable for the Reflow criterion because content remains reachable via buttons (WCAG 1.4.10 not fetched; see Gaps).
- Avoid `overscroll` trapping vertical page scroll on touch: use `overscroll-behavior-x: contain` and `touch-action: pan-x pan-y` defaults (practitioner judgment, no source).

### Gaps
- No MDN/CSS spec fetched for `scroll-snap` behavior, `scroll-padding`, `scrollend` event support; no Baseline status check for `::scroll-marker`/`scroll-state()` in Safari/Firefox (Chrome article does not cover them), so do not rely on them without a fallback.
- No source on whether screen readers announce position within overflow scrollers without ARIA; test with TalkBack and VoiceOver.
- No NN/g carousel research fetched in this session.

## 5. Mobile UX for the Indian market: what separates polished from average

### Takeaway
Hard evidence retrieved this session is thin for India specifically; the defensible rules are physical touch size (>= 1 cm), WCAG 24 px minimum, test on throttled mid-tier hardware, and keep payload and font count low.

### Cited Findings
- Touch size: make interactive elements at least 1 cm x 1 cm (0.4 in) physical; bigger for primary CTAs and use while moving (Target's primary buttons ~2 cm); average fingertip 1.6–2 cm, thumb impact ~2.5 cm; spacing helps but cannot rescue too-small targets; side-by-side targets are safer than stacked — [NN/g: touch target size](https://www.nngroup.com/articles/touch-target-size/)
- WCAG floor is 24x24 CSS px; 44x44 CSS px is the AAA enhanced level — [W3C Understanding 2.5.8](https://w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)
- Test on a real mid-range Android over mobile data, not office wifi; compress images (WebP), reduce fonts to one family; run PageSpeed Insights with mobile throttling (practitioner guidance, no India data in page) — [echai.ventures: slow 4G in India](https://echai.ventures/startingup/website-landing/how-do-i-make-my-landing-page-load-fast-on-a-slow-4g-phone-in-india)
- Lab proxy for slow Indian mobile: Lighthouse Slow 4G (150 ms RTT, 1.6 Mbps, 4x CPU) and Moto G4 as mid-tier reference — [Lighthouse throttling docs](https://raw.githubusercontent.com/GoogleChrome/lighthouse/HEAD/docs/throttling.md)
- Google's 2015 India-specific initiative (lighter, faster pages for Indian Android users) is historical context only — [Google India blog (2015)](https://india.googleblog.com/2015/06/faster-and-lighter-mobile-web-pages-in.html) (not fetched; title only)

### Inferences
- Mobile checklist to audit (practitioner standards, not individually sourced): body text >= 16 px so iOS Safari does not zoom on input focus; form inputs with correct `type`/`inputmode`/`autocomplete` (Novaframe already uses `autocomplete="name"` and `organization`); `viewport` with `width=device-width, initial-scale=1`, never `user-scalable=no`; `env(safe-area-inset-*)` padding with `viewport-fit=cover` for notched phones and sticky bars (the file has a `claude:viewport-fit-sides` meta, so the host appears to manage this; verify); sticky bottom CTA or WhatsApp/tap-to-call (`tel:`) link, which is common for Indian service businesses (no source); `100dvh` instead of `100vh` for hero; avoid hover-only interactions; test Hindi/regional-language rendering if copy is localized; Data Saver/low-end considerations: avoid autoplay video, defer non-critical JS.
- Sticky CTAs and headers must be checked against 2.4.11 (focus not obscured).

### Gaps
- No retrieved data on Indian device mix, share of Android Go/low-RAM phones, median network speed, or CrUX India LCP distributions in 2026. Searches returned only dated (2015–2018) or low-quality blog sources; do not quote numbers.
- No Google/Material or Apple HIG primary source fetched for 44/48 px targets.
- No evidence fetched on WhatsApp click-to-chat conversion for Indian agencies.

## 6. Quick audit checklist for Novaframe (derived from the above)

### Takeaway
Highest-value checks are title/description, font loading, reduced motion, contrast in both themes, and building the team scroller to the APG/Chrome pattern with buttons.

### Cited Findings
- Thresholds to hit: LCP <= 2.5 s, INP <= 200 ms, CLS <= 0.1 at p75 — [web.dev](https://web.dev/articles/defining-core-web-vitals-thresholds)
- Text contrast 4.5:1 (3:1 large); UI/focus/border 3:1; target >= 24x24 CSS px — [W3C 1.4.3](https://w3.org/WAI/WCAG22/Understanding/contrast-minimum.html); [W3C 1.4.11](https://w3.org/WAI/WCAG22/Understanding/non-text-contrast); [W3C 2.5.8](https://w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)

### Inferences
- Observations from the file (grep only, head partially inspected): `<title>Novaframe</title>` (brand only); no `<meta name="description">`, `rel=canonical`, OG/Twitter, favicon or JSON-LD matched; no `lang` attribute match (host may add); no `prefers-reduced-motion` match; fonts via render-blocking Google Fonts CSS; form uses visible labels with `for` and `autocomplete`, `novalidate` (so custom error messaging must be accessible; verify `aria-describedby`/live announcements).
- Priority order: (1) title and meta description rewrite; (2) reduced-motion guard on infinite animations; (3) contrast and focus verification in both themes; (4) scroller with buttons, peek, label and reduced-motion-aware scrolling; (5) font weight trimming and fallback metrics; (6) JSON-LD Organization if the host allows body-level scripts (JSON-LD may sit in body); (7) OG/Twitter tags only if host passes them through.

### Gaps
- Whether the host strips or allows JSON-LD/OG tags, sets `lang`, or injects canonical is unknown and should be tested by viewing the live page source.
