# Jeju trip board — notes for Claude

A static site for a 11/17–11/21/2026 Jeju road trip, served by GitHub Pages from `main`
(https://aimj0120-ai.github.io/jeju-trip/). The owner edits it from an iPhone while travelling.

## How to talk to the owner
- Reply in Traditional Chinese (繁體中文) only, no English mixed into prose.
- The owner checks results on iPhone Safari. GitHub Pages caches ~10 minutes: after a deploy,
  tell them to wait and reload if they don't see the change.

## Files
- `index.html` — the trip board (one file, vanilla JS, Leaflet map, OSM tiles).
- `food.html` — food list page. Data array `DAYS` inside; when meals change in `index.html`, update this too.
- `pay.html` — Korea payment guide (LINE Pay / 悠遊付 / 全支付 promos, Lotte Mart and convenience-store
  shopping lists). Organized by card, never by person; no product photos (copyright).

## The one rule that matters: saved data lives on the phone
The board stores the itinerary in the phone's localStorage (`jeju-trip-v1`, `tripId:'jeju-2026-11'`).
Editing the default `TRIP` object alone does NOT change what the owner sees.
Any change to existing stops must ship as a one-time migration:

```js
if(S.tripId===TRIP_ID&&!S.patches.includes('my-new-id')){
  /* find the day by its stops (e.g. d.stops.some(s=>s.ko==='...')), never by title */
  S.patches.push('my-new-id');
}
```
- Put new patches just before the line `S.days.forEach(d=>d.stops.forEach(s=>{const c=CONTACT[s.ko];`
  so they run after every older patch.
- Stops are keyed by Korean name `s.ko`; side data maps use the same key:
  `CONTACT`, `REVIEWS`, `PARK`, `LINKS`, `ICON_OF`, `MUSTEAT`, `RESERVE`, `KAKAO_ID`.
- `alt:true` marks a rain backup that replaces the main stop just before it.

## Facts sources
- Places, phone, hours, reviews: Kakao Map (`search.map.kakao.com/mapsearch/map.daum?q=`,
  `place-api.map.kakao.com/places/panel3/{id}` with headers `Referer: https://place.map.kakao.com/`, `pf: web`).
- Drive times: `router.project-osrm.org`.
- NAVER blocks automated access; use NAVER search links (`https://map.naver.com/p/search/...`).

## Before pushing
- Syntax-check the inline script:
  `awk '/^<script>$/{f=1;next}/^<\/script>$/{f=0}f' index.html > /tmp/chk.js && node --check /tmp/chk.js`
  (same for `food.html`).
- Keep the layout working at 375 px width (iPhone). iOS gotchas: no `var()` inside `@keyframes`;
  use `overflow-x:clip` (not `hidden`) on html/body; no overflow clip on the header.
- Deploy = merge to `main`. If the session can only push a branch, open a PR so the owner can merge it from the phone.

## Privacy
Never put booking numbers, membership/coupon barcodes, emails or other personal data on the site — it is public.
