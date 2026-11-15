# Telugu / English / Tinglish local-business website: i18n, fonts, SEO (Andhra Pradesh and Telangana, India)

Research date: 2026-11-15. Tool-call budget limited the depth; several items are flagged as gaps.

## 1. Single page client-side toggle vs separate URLs (Google docs: hreflang, lang, duplicate content)

### Takeaway
Use separate, crawlable URLs per language (subdirectories such as /te/ and /en/) linked with reciprocal hreflang, plus a visible hyperlink language switcher. Google explicitly recommends different URLs per language over cookie/browser-setting switching and warns against auto-redirecting by IP or guessed language.

### Cited Findings
- Google recommends "using different URLs for each language version of a page" rather than cookies or browser settings, with hreflang annotations to link them — [Google Search Central: Managing multi-regional and multilingual sites](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites)
- If content changes by language setting, Google might not find and crawl all variations; Googlebot usually crawls from the USA and sends no Accept-Language header — [same](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites)
- Do not redirect users between language versions based on guessed language or IP; add hyperlinks to other versions as a language selector — [same](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites)
- URL structure: subdirectories on a gTLD (example.com/te/) are "easy to set up and low maintenance" (single server location); subdomains are easy but users may not tell whether a code means language or country; ccTLDs are expensive and single-country; URL parameters (?loc=de) are not recommended — [same](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites)
- Google determines page language from visible content, not from lang attributes or URL. Keep content and navigation in one language per page and avoid side-by-side translations — [same](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites)
- Translating only boilerplate while leaving main content in one language creates a poor experience in results — [same](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites)
- "Localized versions of a page are only considered duplicates if the main content of the page remains untranslated." So fully translated Telugu/English pages are not duplicate content — [Google: Localized versions](https://developers.google.com/search/docs/specialty/international/localized-versions)
- hreflang rules: each version lists itself and all others (identical sets); fully qualified URLs with https; reciprocal links or tags are ignored; use x-default for fallback (e.g., language selector); codes are ISO 639-1 plus optional ISO 3166-1 region (a country code alone is invalid); script codes like zh-Hant allowed. Delivered via link tags, HTTP headers or XML sitemap (all treated equally) — [Google: Localized versions](https://developers.google.com/search/docs/specialty/international/localized-versions)
- Google does not use hreflang or the HTML lang attribute to detect page language — [Google: Localized versions](https://developers.google.com/search/docs/specialty/international/localized-versions)
- W3C: use a script subtag only when it makes a necessary distinction; keep tags as short as possible; script subtags come from ISO 15924, Latn conventionally capitalised; lang is inherited by children unless overridden — [W3C: Language tags in HTML and XML](https://www.w3.org/International/articles/language-tags/index.var)

### Inferences
- Recommended structure: /te/ (Telugu script), /en/ (English), optionally /te-latn/ or Tinglish handled as sections on the te page rather than a third full language tree (see section on search behaviour). hreflang set: te, en, x-default (pointing to the chosen default or a selector page), each self-referencing and reciprocal. Using `en-IN` and `te-IN` is valid but region is optional; Google does not require it.
- `<html lang="te">` on Telugu pages, `lang="en"` on English pages; mark inline English phrases in Telugu pages with `<span lang="en">` (helps screen readers and font fallback). For Tinglish text, `lang="te-Latn"` is a valid BCP 47 tag (language + ISO 15924 script) — this is my inference from the W3C tag structure, W3C gave no transliteration example. Since Google ignores lang, this is for accessibility/browsers only.
- Client-side JS toggle that swaps text on one URL means only one language is indexable; avoid it for anything you want ranked. A progressive enhancement is fine: real links to /te/ and /en/, JS only remembers the preference (never auto-redirect crawlers).
- Persisting preference: store choice in localStorage/cookie and offer a suggestion banner, not a forced redirect.

### Gaps
- Google's pages fetched do not cover Telugu-specific hreflang behaviour or mixed-script (Tinglish) page handling; no official guidance found on Tinglish pages.
- w3.org "qa-html-language-declarations" article was not on the fetch allowlist; lang attribute detail relies on the language-tags article.

## 2. Telugu web fonts, weight cost, subsetting

### Takeaway
Noto Sans Telugu (variable, OFL) is the safe default; full variable TTF is 868 KB, so self-host WOFF2 with Telugu + Latin subsetting, limit weights, and use font-display and preload. Specific WOFF2 byte sizes for Hind Guntur and Mandali could not be measured (CDN blocked).

### Cited Findings
- Noto Sans Telugu on Fontsource: variable file with wght 100–900 and wdth 62.5–100 axes; 958 glyphs; covers Telugu, Basic Latin, General Punctuation, Devanagari; SIL OFL 1.1; `NotoSansTelugu[wdth,wght].ttf` is 868 KB (uncompressed TTF); last updated Mar 12 2026 — [Fontsource](https://fontsource.org/fonts/noto-sans-telugu/about)
- WOFF2 uses Brotli and is up to ~30% smaller than WOFF; serve WOFF2 first; EOT/TTF unnecessary — [web.dev: Reduce web font size](https://web.dev/articles/reduce-webfont-size)
- unicode-range lets browsers download only needed subsets; matters most for Asian-script fonts where a full font can be megabytes. Subset by script, not per page, to preserve cache reuse; use pyftsubset; Google Fonts subsets automatically — [web.dev](https://web.dev/articles/reduce-webfont-size)
- font-display: `swap` can cause layout shifts hurting CLS; `optional` or `fallback` suggested; preload critical fonts — [web.dev](https://web.dev/articles/reduce-webfont-size) (article last updated Aug 2019, some details dated)
- Other Google Fonts Telugu families exist, including Ramabhadra (listed on Google Fonts) and Hind Guntur (also on Adobe Fonts) — [Google Fonts: Ramabhadra](https://fonts.google.com/specimen/Ramabhadra/about), [Adobe Fonts: Hind Guntur](https://fonts.adobe.com/fonts/hind-guntur)

### Inferences
- Practical standard: Noto Sans Telugu for body/UI (broadest, consistent with Noto Sans Latin pairing); Hind Guntur is a lighter-looking alternative with matching Latin, and Mandali/Ramabhadra/other display faces are best reserved for headings only. Pick at most 2 weights (400 and 600/700), or the variable font if 3+ weights are needed.
- Telugu conjuncts rely on OpenType shaping (GSUB); aggressive glyph-level subsetting can break conjunct rendering, so subset by Unicode block (Telugu U+0C00–0C7F, ZWJ/ZWNJ U+200C/200D, Latin, punctuation, ₹ U+20B9) rather than by the page's used characters.
- Keep the Telugu font off the critical path for English pages (load via unicode-range or only on /te/), and define a system fallback stack (Noto Sans Telugu, Nirmala UI, Gautami, sans-serif); Android and Windows ship Telugu system fonts, so a fallback is acceptable on slow networks (font-display: swap/optional with size-adjust to limit CLS).
- Telugu needs more line-height (roughly 1.6–1.8) than Latin because of stacked vowel signs; avoid tight letter-spacing and uppercase transforms.

### Gaps
- Could not measure actual WOFF2 sizes for Noto Sans Telugu / Hind Guntur / Mandali (fonts.googleapis.com and jsDelivr returned proxy 403). Report writer should say "measure with Google Fonts CSS2 API or pyftsubset" rather than quote numbers.
- No credible source found for the line-height figure; it is typographic practice, not cited.

## 3. Telugu users: search, video, WhatsApp; credible stats

### Takeaway
Indian-language usage is the growth driver (IAMAI-Kantar), short video reaches 61% of users, and YouTube is the dominant platform among Indian-language news consumers; but no source found gives Telugu-specific user counts or Tinglish query share.

### Cited Findings
- IAMAI-Kantar "Internet in India 2024": 886M users, rural 488M (about 55%), 98% of users accessed Indic-language content; 57% of urban users prefer regional-language content; Telugu named with Tamil and Malayalam among most popular Indic languages "due to extensive availability" (no Telugu user numbers) — [The Week/PTI](https://www.theweek.in/wire-updates/business/2025/01/16/dcm51-biz-iamai-internet-usage-report.html)
- IAMAI "Internet in India 2025": 958M active internet users, rural 57% (about 548M), 588M (61%) consumed short-video content, 44% used AI features such as voice search, 18% use someone else's phone (about 80% of those rural) — [Telangana Today/PTI](https://telanganatoday.com/indias-internet-user-base-crosses-950-million-in-2025-iamai-report)
- Google News Initiative–Kantar study of Indian-language online news consumers (May 2023, eight languages incl. Telugu): 93% use YouTube for news, 88% social media, 82% chat apps (WhatsApp not named), 45% publisher sites/apps; average 5.05 platforms; no Telugu-specific figures in the coverage — [Telangana Today/IANS](https://telanganatoday.com/youtube-most-popular-platform-for-indian-language-news-consumers-report)
- Google (Dec 2020): typing in non-English languages is a challenge so many users search in English even when preferring local-language results; Google added Telugu results display; MuRIL model supports transliterated (Roman-script) Hindi and beats earlier model by 27% on transliterated text (Hindi only stated) — [TechCrunch](https://techcrunch.com/2020/12/16/google-expands-languages-push-in-india-to-serve-non-english-speakers/amp)
- Think with Google "App and website localization for Indian users": 63% go online in an Indian language (alone or with English); users read colloquial language, literal translations of complex words are hard; transliterate technical terms; keep numerals in English; use ₹ and lakh/crore; keep each text in a single script, with exceptions for brand names and English technical terms with meaning in brackets; users type transliterated text so autocomplete should handle it; local-language search returns fewer results; typing Indian scripts is hard even for fluent users; prioritise voice. HDFC MF Hindi case: 40% lower CPA on Discovery — [Think with Google PDF](https://thinkwithgoogle.com/_qs/documents/13326/App_and_website_localization_for_Indian_users.pdf) (undated; the "98% prefer exclusively regional" and "63%" figures had ambiguous labels in extraction, so treat with caution)

### Inferences
- Because typing Telugu script is hard, Latin-script (Tinglish) and English queries likely coexist with Telugu-script queries for local-service intent (e.g., "website design Vijayawada", "Hyderabad lo digital marketing"). This is supported only indirectly (Google 2020; Think with Google) and not quantified for Telugu.
- Practical: target English commercial keywords on /en/ pages; Telugu-script pages for Telugu-language intent and voice search; include a few Tinglish phrasings naturally in headings/FAQ/alt text or WhatsApp/Reels captions rather than building a standalone Tinglish page set. Verify with Search Console queries and Google Keyword Planner / Trends before committing.
- WhatsApp click-to-chat (wa.me) CTA with prefilled Telugu or English message, plus click-to-call, aligns with chat-app dominance, shared-device and voice-first behaviour. Embed or link YouTube/Reels and keep captions in Telugu script + Tinglish hashtags (inference; no source on Reels caption behaviour).

### Gaps
- No source found on Telugu-specific internet users, Telugu share of searches, share of Tinglish/Latin-script Telugu queries, Instagram Reels or WhatsApp usage among Telugu users, or state-level (AP/Telangana) language data. Google-KPMG 2017 "Indian Languages: Defining India's Internet" appeared in results (KPMG page) but was not fetched; its figures are dated anyway.

## 4. Local SEO and structured data for a two-state agency without a storefront

### Takeaway
Use a Google Business Profile as a service-area business (hide address) if there is no staffed premises, mark up the site with a ProfessionalService/LocalBusiness JSON-LD with areaServed listing both states as AdministrativeArea, and keep NAP identical across site, GBP and directories.

### Cited Findings
- Google Business Profile guidelines: businesses without a storefront that travel to customers may have one service-area profile; a "virtual" office can't be listed unless staffed in business hours; service-area businesses should hide their address; boundaries generally should not extend more than about two hours' drive from the base (larger may suit some businesses); no maximum number of service areas stated; business name must match real-world representation — [Google Business Profile Help](https://support.google.com/business/answer/3038177?hl=en)
- Google's LocalBusiness structured data: required `address` and `name`; recommended `geo` (5+ decimals), `openingHoursSpecification`, `telephone` (with country code), `url`, `priceRange`; use the most specific subtype; list multiple types as an array; JSON-LD in a script tag; validate with Rich Results Test. The page does not cover areaServed, service-area businesses, sameAs or no-storefront businesses — [Google: LocalBusiness structured data](https://developers.google.com/search/docs/appearance/structured-data/local-business)
- schema.org: `areaServed` expects AdministrativeArea, GeoShape, Place or Text; defined as the geographic area where a service is provided (supersedes serviceArea); ProfessionalService is a LocalBusiness subtype — [schema.org LocalBusiness](https://schema.org/LocalBusiness)

### Inferences
- Example pattern: `@type: ["ProfessionalService"]` (or LocalBusiness), `name`, `url`, `telephone` (+91), `email`, `areaServed: [{"@type":"State","name":"Andhra Pradesh","sameAs":"<Wikipedia/Wikidata URL>"},{"@type":"State","name":"Telangana", ...}]`, plus a handful of City entries (Hyderabad, Vijayawada, Visakhapatnam, Guntur, Tirupati, Warangal) if you serve them specifically; `State` is a schema.org subtype of AdministrativeArea (I did not fetch the State page; it is well known but unverified here). `inLanguage`/`availableLanguage` can express Telugu, English via `knowsLanguage` or ContactPoint `availableLanguage: ["Telugu","English"]`.
- For a no-storefront business, include a postal address only if it is a real operating base you are willing to show; Google's own structured-data page requires address for LocalBusiness, so if you will not publish an address, use `Organization`/`Service` markup with areaServed instead and accept no local-pack rich markup. That is a tension to flag; resolve by choice of base city (and GBP service-area).
- Two states exceed the ~two-hour guideline from any single base (e.g., Hyderabad to Visakhapatnam is far longer than two hours by road), so GBP will realistically cover only the base metro; reach the rest via location landing pages with genuine content (not doorway pages; see Google scaled-content policy), citations and Telugu content. This is judgment based on the guideline.
- NAP: Google's page does not explicitly address NAP consistency for SABs; best practice (industry, not cited here) is exact same business name, phone format (+91) and address string on site footer, schema, GBP and directories (Justdial, Sulekha, IndiaMART etc.), including the Telugu-script name variant as an alternate name rather than a replacement.
- Add a Telugu name/description on GBP as additional content, but GBP category, primary name and reviews should remain consistent; bilingual reviews replies are good.

### Gaps
- No fetched source on GBP language handling (bilingual profile fields), on schema `State` type, or on Indian citation directories; hybrid recommendations above are uncited.
- Rich-result eligibility for Service/ProfessionalService without address was not verified.

## 5. Telugu content quality pitfalls

### Takeaway
Avoid raw machine translation and literal translations; write in colloquial (spoken-register) Telugu with English loanwords for technical terms, keep one script per text unit, and have a native reviewer. Google's spam policy targets scaled low-value or translated content.

### Cited Findings
- Google spam policy: scaled content abuse includes generating many pages mainly to manipulate rankings, including via "synonymizing, translating, or other obfuscation" with little user value; the page has no separate rule on machine translation as such — [Google: Spam policies](https://developers.google.com/search/docs/essentials/spam-policies)
- Think with Google guidance: use colloquial language, transliterate technical terms where no everyday equivalent, keep numerals in English, localise currency (₹, lakh/crore, no "15k"), build a translation glossary, translate all essential UI including errors, FAQs, CTAs; keep one script per text with brand-name/acronym exceptions — [Think with Google PDF](https://thinkwithgoogle.com/_qs/documents/13326/App_and_website_localization_for_Indian_users.pdf)
- Google: do not mix translated boilerplate with untranslated main content, and do not present side-by-side translations on one page — [Google: Managing multi-regional sites](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites)

### Inferences
- Common pitfalls (practitioner knowledge, no source found): formal literary (granthika) Telugu that reads as stiff vs spoken (vyavaharika) register; machine-translated marketing terms such as "SEO" or "brand" rendered as awkward Telugu coinages; dialect differences between Telangana and coastal Andhra / Rayalaseema vocabulary (use neutral vocabulary or note a regional variant); broken conjunct rendering due to missing fonts or Unicode normalization issues; legacy non-Unicode fonts (Anu, etc.) used in copy-paste sources; mixed scripts in one sentence; English-only meta titles on Telugu pages; Telugu slugs percent-encoded (prefer Latin transliterated slugs, e.g. /te/seva/website-design/, with Telugu H1/title).
- Tinglish should be a deliberate tone/keyword layer (captions, WhatsApp copy, FAQ phrasing), always with accurate Telugu-script main copy on /te/ pages.
- Run native review for each page; track Telugu-specific Core Web Vitals on mobile mid-range Android over 4G.

### Gaps
- No credible source found for dialect or register guidance, or for Telugu slug/URL-encoding practice; these items are uncited practitioner judgment.
