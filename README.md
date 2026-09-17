# FearBuster — Build Package

This folder contains the complete FearBuster implementation: a privacy-first, client-side guided practice app for turning toward a fear with support and agency.

## What's Here

- **`fearbuster-dist/`** — Production-ready static build. All assets, no server required.
- **`FEARBUSTER_IMPLEMENTATION.md`** — Complete technical overview, decisions made, testing roadmap, and questions for review.
- **`DEVELOP.md`** — Instructions for local development (below).

## Quick Deploy

### Option 1: GitHub Pages
```bash
# If you have a repo
git clone <repo>
cd fearbuster
npm run build
# Push dist/ to gh-pages branch or configure in repo settings
```

### Option 2: Netlify
```bash
# Drag dist/ into Netlify drop zone
# or connect repo and set build command: npm run build
```

### Option 3: Local Testing via HTTP Server
```bash
# Python 3
cd fearbuster-dist
python -m http.server 8000
# Open http://localhost:8000
```

### Option 4: Docker
```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```

## Local Development

### Setup
```bash
cd fearbuster
npm install
npm run dev
# Opens http://localhost:5173 with hot reload
```

### Make Changes
- **UI/copy:** Edit files in `src/content/` and component `.tsx` files. Changes auto-reload.
- **Design tokens:** `tailwind.config.js` (colors, fonts, spacing).
- **Belief mappings:** `src/content/beliefs.ts` (7 beliefs + support statements).
- **Safety/privacy text:** `src/content/safety.ts` and `src/content/privacy.ts`.

### Build & Test
```bash
npm run build
npm run preview  # Local preview of the production build
```

## What to Test

### Happy Path
1. Click "Begin the practice"
2. Go through all 8 stages, filling in each field
3. Reach the final stage, click "Complete the practice"
4. Click "Export this practice", select sections, download as text or PDF
5. Verify exported content includes your responses

### Session Recovery
1. Start a practice, fill in Stages 1–3
2. Close the browser tab / refresh the page
3. Reload the app — a recovery banner should appear with options to Resume, Review/Export, Start Fresh, or Delete
4. Click "Resume" — you should be back at Stage 4 with all earlier work intact

### Pause & Ground
1. During any stage, click the "Pause & Ground" pill button (bottom-right)
2. Read through the 4-step grounding sequence
3. Click "Return to where I was"
4. Verify you're back at the same stage with no work lost

### Privacy Controls
1. Click "Privacy" (top-right)
2. Toggle autosave off
3. Fill in Stage 1 and refresh — the data should be gone
4. Toggle autosave back on
5. Fill in Stage 1 again and refresh — it should persist

### Mobile Layout
1. Open on a mobile phone or resize desktop browser to ~375px width
2. Stage 4 (the three-column sorting exercise) should stack vertically: realistic → exaggerated → unknown
3. All other stages should be full-width single-column

## File Sizes

- Main bundle: 273 KB (85 KB gzip) — loaded immediately
- jsPDF chunk: 399 KB (130 KB gzip) — loaded only when user exports to PDF
- CSS: 14 KB (4 KB gzip)
- **Total initial load:** ~220 KB gzip (before PDF export)

## Browser Support

- Modern browsers (Chrome, Firefox, Safari, Edge, iOS Safari, Android Chrome)
- Requires ES2023 and DOM APIs (localStorage, crypto.randomUUID, Fetch)
- No polyfills needed for current target market

## Privacy & Data

- **Zero external requests.** All data stays in browser localStorage.
- **No analytics.** No tracking pixels, no server logs.
- **No accounts.** No sign-in required.
- **Delete controls.** Built-in: delete this practice, delete all practices, disable autosave.
- **Export-only strategy.** Users must manually export if they want data outside the browser.

## Customization

### Colors
Edit `tailwind.config.js` → `theme.extend.colors`:
- `slate-deep`: Primary dark (currently `#212B3B`)
- `ember`: Accent (currently `#B8823D`)
- `sage`: Grounding/safety (currently `#5C7A6E`)
- `ink`, `mist`: Text and backgrounds

### Fonts
Edit `tailwind.config.js` → `theme.extend.fontFamily`:
- `serif`: Newsreader (headlines)
- `sans`: Inter (body)

### Copy
Edit `/src/content/`:
- `copy.ts` — All UI text (prompts, button labels, etc.)
- `beliefs.ts` — Belief options and their support statements
- `safety.ts` — Crisis resources text
- `privacy.ts` — Privacy & data control language
- `completion.ts` — Rule-based closing affirmations

After changes, rebuild:
```bash
npm run build
```

## Troubleshooting

**Page blank/white screen:**
- Check browser console for errors (F12)
- Verify all assets in `fearbuster-dist/assets/` are present
- Clear browser cache and reload

**Export not working:**
- Verify localStorage is enabled
- Try a different browser
- Check console for jsPDF import errors

**Pause & Ground doesn't appear:**
- Button is fixed bottom-right; may need scroll
- On mobile, button may be repositioned by keyboard showing
- Check that Radix Dialog is imported in `src/components/PauseGround.tsx`

**Mobile layout not stacking:**
- Check responsive breakpoint at `md:grid-cols-3` in `src/components/stages/Stage4.tsx`
- Verify Tailwind config includes `md` breakpoint (should be 768px by default)

---

## Questions for Rob

See **FEARBUSTER_IMPLEMENTATION.md** for technical questions and testing priorities.

**Key review points:**
1. Do the 7 beliefs (and fallback statement) land correctly?
2. Does the grounding sequence read naturally?
3. Are the closing affirmations hitting the right tone?
4. Mobile layout feels right on a real phone?

---

**Build date:** September 2026  
**Status:** Ready for review, accessibility testing, and deployment

