# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static AngularJS 1.3 app — the classic "PhoneCat" tutorial app (phone/tablet catalog with a list view and a detail view). There is no build tool, package manager manifest, or test runner checked into this repo (no `package.json`, `bower.json`, or Karma config) — `bower_components/` is vendored directly, and `js/`, `css/`, `partials/`, `phones/` are plain static assets served as-is.

## Running the app

The app fetches `phones/*.json` via `$resource`/XHR, so it must be served over HTTP — opening `index.html` directly via `file://` will fail to load phone data. Serve the repo root with any static file server, e.g.:

```
python3 -m http.server 8000
```

then open `http://localhost:8000/index.html`.

There is no configured lint, build, or test command in this repo.

## Deployment

`.deployment` + `deploy.cmd` configure an Azure/Kudu deployment: the site is deployed as-is (static files), with `.bowerrc` pointing bower installs at `app/bower_components` when `bower install` is run from a parent `app/` directory (not applicable to local dev in this checkout, where `bower_components` already exists at the repo root).

## Architecture

Standard Angular 1.x module structure, wired together in `js/app.js`:

- **`phonecatApp`** (`js/app.js`) — root module; declares two routes via `ngRoute`:
  - `/phones` → `phone-list.html` + `PhoneListCtrl`
  - `/phones/:phoneId` → `phone-detail.html` + `PhoneDetailCtrl`
- **`phonecatControllers`** (`js/controllers.js`) — `PhoneListCtrl` loads all phones via the `Phone` service; `PhoneDetailCtrl` loads a single phone by `$routeParams.phoneId` and tracks the selected main image.
- **`phonecatServices`** (`js/services.js`) — `Phone` is an `$resource` wrapping `phones/:phoneId.json`, with a custom `query` action that fetches `phones/phones.json` (the catalog index) for the list view, and per-phone `phones/<id>.json` files for detail views.
- **`phonecatFilters`** (`js/filters.js`) — `checkmark` filter renders booleans as ✓/✗ in the spec table.
- **`phonecatAnimations`** (`js/animations.js`) — CSS/JS-driven slide transitions for `.phone` image swaps in the detail view, built on `ngAnimate` + jQuery `.animate()`.
- **`js/directives.js`** — currently empty, present as a module placeholder.

Data flow: `phones/phones.json` is a lightweight index (id, name, snippet, thumbnail, age for sort order) used by the list view; each `phones/<phone-id>.json` file holds the full spec sheet used by the detail view. Adding a new phone means adding both an entry in `phones/phones.json` and a corresponding `phones/<id>.json` detail file, plus images under `img/phones/`.

There is also an unrelated PDF (`Arhitektura_korporativnih_programmnih_prilojzeniy_[torrents.ru]/`) checked into the repo root — not part of the app.
