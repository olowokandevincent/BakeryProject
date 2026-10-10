# AGENTS.md — running this repo in the Base44 sandbox

## What this project is

A **static, client-side single-page site** (Bakerz Bite). No build step, no package
manager, no backend, no database, no secrets.

- `index.html` — the SPA
- `css/style.css`, `js/main.js`
- `data/products.json` — product/gallery/offer/FAQ data
- Third-party libraries (Bootstrap 5, Font Awesome, AOS, Google Fonts) load from CDNs
  at runtime; no local install is required.

## Non-obvious setup requirements

- It **must be served over HTTP**. `js/main.js` does `fetch('data/products.json')`
  with a relative path, so opening the file directly (`file://`) fails with a CORS
  error and the page renders with empty product/gallery sections.
- It must be served **from the repository root**, so the relative `data/`,
  `css/`, `js/` and `assets/` paths resolve.

## How it runs here

`docker-compose.base44.yml` runs one service (`web`): a `python:3.12-alpine`
container serving the bind-mounted repository root on port 3000.
Repository changes are visible on the next page load; there is no file watcher, so
after editing a file use `reload_preview` (or refresh) to see it.

```bash
docker compose -f docker-compose.base44.yml up -d --build
docker compose -f docker-compose.base44.yml ps
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:3000/
```

## How to verify it works

1. `curl -s http://localhost:3000/` returns the HTML shell (`200`).
2. `curl -s http://localhost:3000/data/products.json` returns JSON (not 404) —
   this is what the product grid depends on.
3. In the preview: the preloader clears and the **Product Catalogue** grid is
   populated with cards from `products.json` (not the "No products found" message).

## Things that are expected (not bugs)

- Feedback/contact/newsletter forms are client-side only — they show a toast and do
  not send anything (no endpoint).
- The visitor counter, ticker and geolocation use browser APIs; geolocation may be
  denied in the preview, which is handled gracefully.
- Google Maps iframe / WhatsApp links point at external services.
