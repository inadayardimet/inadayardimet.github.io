# Charity Fundraiser Site — Design & Behavior Spec

Template spec distilled from the İnad Abbasov site redesign (inadayardimet.github.io, July 2026).
Give this file to Claude in any session to apply the same design to another sick child's fundraiser site.
Reference implementation: `index.html` in the inadayardimet.github.io repo (single self-contained HTML file, GitHub Pages, no build step).

## Context & goals

- Audience arrives mostly from TikTok/Instagram/Facebook ads on mobile. Decisions happen in seconds.
- The single conversion action is: **copy a bank card number and transfer money**. Everything serves that.
- Second action: **contact the family on WhatsApp** (trust building).
- Site must be a single `index.html`, no frameworks, no build step, deployable on GitHub Pages.

## Design system

- Style: warm, card-based, "professional charity" look (originally derived from the Ayaz site).
- Fonts: **Lora** (serif, headings) + **Inter** (body), Google Fonts with `display=swap`.
- Color tokens (İnad palette — adjust accent per child if asked, keep the warm-light structure):
  - `--bg: #FAF7F2` (warm cream page background)
  - `--surface: #FFFFFF` (cards)
  - `--ink: #12325B` (headings, navy)
  - `--body: #3D4A5C`, `--muted: #7A8699`
  - `--accent: #E85D2C`, `--accent-dark: #C84A1F`, `--accent-soft: #FDEDE4`, `--accent-border: #F6D5C4`
  - `--green: #25D366` (WhatsApp), `--line: #ECE5DB` (hairlines)
- Shapes: pill buttons (`border-radius: 100px`), cards 20px radius, photos 28px radius with 5-6px white border frame.
- Primary buttons: orange **gradient** `linear-gradient(135deg, #FF7A45, accent 55%, accent-dark)` + soft orange glow shadow.
- Soft layered shadows; hover lift on cards (translateY(-4..6px)).

## Page structure (in order)

1. **Sticky nav** — heart logo circle + child's name, anchor links (Hekayə / Şəkillər / Sənədlər), decorative lang pills (AZ TR RU EN), primary button "Kart nömrələri". Solid `rgba(bg, .96)` background — **NO backdrop-filter blur** (janks scroll on cheap Androids).
2. **Hero** — two columns (text left, photo right in rounded white frame with floating badge "Hər dəstək bir ümiddir").
   - Badge pill "Xeyriyyə kampaniyası", serif H1 ("<Ad>ın dəstəyinizə ehtiyacı var"), 2-3 sentence subtitle (age, diagnosis, what's urgently needed).
   - Exactly 3 compact buttons: **"Kart nömrələrinə bax"** (primary, card icon), **"Sənədlərə bax"** (white, → #senedler), **"Şübhən var? Ailədən soruş"** (accent-tinted `btn-ask`, → #elaqe).
   - NO chips/tags row (removed as clutter). NO WhatsApp button here (floating one exists).
   - **Mobile: text ABOVE photo**, and hero must be compact enough that the photo top peeks into the first viewport (~670px from top at 412px width). Compact mobile sizes: h1 33px, sub 16px, buttons 11px/18px padding.
3. **Story** (`#hekaye`) — white band; left: 4 short paragraphs (diagnosis, what family sold/sacrificed, parents' jobs, hospital + needed treatment + amount in bold); right: sticky quote card with mother's quote, her name, green "Doğrulanıb" pill.
4. **Gallery** (`#sekiller`) — "İnadın gözündən" style title; 3 photos, rounded white-framed cards. **Mobile: 3 columns side-by-side (small but recognizable), NOT stacked** (keeps page short).
5. **Documents** (`#senedler`) — "Hər söz sənədlə təsdiqlənir" + "İnanmaq üçün sözümüzə yox, sənədlərə baxın."; 3 document cards (hospital treatment plan, bank requisites with stamp, epicrisis), each with green check label. Desktop image aspect 1/1, mobile 4/3 (cropped to top of document — keeps page short, lightbox shows full).
6. **Trust/contact card** (`#elaqe`, `scroll-margin-top: 58px`) — THE anti-scam section, placed **immediately before bank cards** (kills last doubt right before the money moment):
   - Avatar circle with mother's initials, title "Şübhəniz var? Birbaşa ailədən soruşun", subtitle with mother's full name.
   - Text: answers questions personally + video call offer + **"Ailə hazırda Türkiyədə olduğu üçün zəng yalnız WhatsApp üzərindən mümkündür."** (adapt: if family is abroad, ALL phone contact is WhatsApp-only — see rule below).
   - Buttons: "WhatsApp-da soruş" (green) + "WhatsApp zəngi et" (white, phone icon, wa.me link).
   - Anchor position must show this card AND the top of the bank-cards section together in one viewport.
7. **Donation** (`#donation`) — dark navy rounded block (32px radius, inset from page edges):
   - Title "Bank kartları", subtitle naming the cardholder (parent) — all cards must be in one parent's name.
   - 3 bank cards in glass-morphism style (`rgba(255,255,255,.07)` + border), each: colored **bank badge** (Kapital=red "KB", ABB=blue "ABB", Leo=yellow "leo"), monospace card number, **large full-width "Kopyala" button** (16px font, 15px padding), holder line.
   - Copy button: clipboard API with `execCommand` fallback, turns green "Kopyalandı ✓" for 2s, fires Meta Pixel `AddToCart` event if `fbq` exists.
   - Below: "Anaya birbaşa yaz" (WhatsApp green) + phone number button labeled "`<number>` · yalnız WhatsApp" linking to wa.me (NOT `tel:`).
8. **Share** — WhatsApp share (prefilled text), Facebook sharer, copy-link button.
9. **Footer** — serif italic line ("Hər paylaşım ... yaşama şansını artırır" / dua-dəstək-ümid), copyright + WhatsApp number.

## Floating / overlay elements

- **Floating WhatsApp button**: fixed bottom-right, 64px green circle, **pulse ring animation** (scaling ::before, 2s loop, disabled under `prefers-reduced-motion`). On mobile it slides UP (`bottom: calc(92px + safe-area)`) when the sticky bar is shown (`body.bar-shown`).
- **Sticky bottom CTA (mobile only)**: hidden on load; after **300px scroll** a single centered pill "Kart nömrələrinə bax" slides up. **Transparent container — no white bar background, no border, no blur**; `pointer-events: none` on container, `auto` on button. Hides again at top.
- **Callout bubble**: navy bubble "Sualın var? Buradan soruş" attached BELOW the `btn-ask` hero button (arrow pointing up). Appears 1.5s after load, stays 5s, fades; also hides on button tap. Positioned below so it **never covers the other buttons**. Pure CSS transition + 2 setTimeouts — zero perf cost.
- **Lightbox**: tapping any gallery photo or document opens fullscreen dark overlay. Close via ✕, backdrop tap, Escape, and — critically — **device back button**: `history.pushState` on open, `popstate` closes the lightbox instead of leaving the site; non-popstate closes call `history.back()` to clean state.

## UX copy rules

- CTA labels must say literally what happens: "Kart nömrələrinə bax", not "Dəstək ol" / "Kömək et".
- Keep the martyr-family / family-sacrifice angle in hero + story if applicable to the child.
- Trust language: "Doğrulanıb", "İnanmaq üçün sözümüzə yox, sənədlərə baxın."
- If family is abroad (e.g., in Turkey for treatment): **never use `tel:` links anywhere** — every call/contact is a `wa.me` link, and say so explicitly ("yalnız WhatsApp").

## Performance rules (site must open fast, scroll smooth)

- Single HTML file; no JS libraries at all. All JS is small vanilla IIFE.
- `<link rel="preload" as="image" href="<hero>.jpg" fetchpriority="high">`; all other images `loading="lazy" decoding="async"` with width/height attrs.
- Compress all JPGs before shipping (target: hero < ~150KB, others < ~100KB).
- Tracking pixels (Meta + TikTok) **deferred**: load 500ms after `window.load`, with `dns-prefetch` hints and `<noscript>` fallback. Keep per-site pixel IDs.
- **No `backdrop-filter` anywhere** (scroll jank on low-end Android/WebViews).
- Scroll listener: single `requestAnimationFrame`-throttled handler that only touches classList when state actually flips.
- Reveal-on-scroll animations: elements **visible by default**; motion only added when JS adds `html.anim` class + IntersectionObserver. **Skip entirely for in-app browsers** — UA regex `/musical_ly|Bytedance|TikTok|ByteLocale|Instagram|FBAN|FBAV|FB_IAB|Snapchat/i` (these WebViews break animations/viewport units → blank screens).
- Compact page: section padding 68px desktop / 44px mobile, no dead sections. (Stats-counter and donation-impact-amounts sections were tried and REMOVED — don't add them.)

## `<head>` requirements (per child)

- Title pattern: "`<Ad>`a Kömək Et — `<Diaqnoz>` ilə Mübarizə | ...".
- Full SEO set: description, keywords, author, robots; Open Graph (og:image = hero photo, 1200×630, absolute URL); Twitter card; `theme-color` = page bg.
- Inline SVG favicon (data URI) with child's name/initial in brand colors.
- Meta Pixel + TikTok Pixel IDs are per-campaign — ask the user for them (İnad's TikTok: D84Q4V3C77U73K7PGLE0; Meta was placeholder YOUR_PIXEL_ID_HERE).

## Things to collect from the user per child

1. Name, age, diagnosis, years fighting, needed treatment + amount + currency.
2. Story facts (family situation, sacrifices, parents' work, hospital/city).
3. Mother's (or parent's) name + quote; WhatsApp number (confirm if calls are WhatsApp-only).
4. Photos: 1 hero + 3 gallery; 3 document scans. Compress them.
5. Bank cards (bank, number, holder) — badges/colors per bank.
6. Pixel IDs (Meta, TikTok), GitHub Pages repo/URL.

## Known open items on the İnad site (as of this spec)

- Language switcher (AZ/TR/RU/EN) is decorative — the old site's `data-i18n` + TRANSLATIONS dictionary system was NOT ported to the new design (translations for the old copy exist in `index.html.backup-pre-ayaz-style`).
- Accent color change was discussed (teal #0F8A6D recommended as alternative to orange) but not applied.
