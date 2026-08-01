# IndieWeb Compliance for logprintf.com

**Date:** 2026-08-01
**Status:** Approved, ready for implementation planning

## Goal

Make logprintf.com IndieWeb compliant at the foundations level: machine-readable
identity and content markup, a webmention receiving endpoint, and IndieAuth
sign-in. No visual change to the site except a new h-card block on `/about/`.

## Context

Hugo 0.134.2 static site using the `etch` theme as a git submodule
(`themes/etch`, pinned at `3286754`). Deployed to Cloudflare Pages.

Two facts discovered during exploration that constrain the design:

1. **Cloudflare Pages has no build command.** It serves the committed `public/`
   directory verbatim. The live homepage is byte-identical to
   `public/index.html`. Therefore `public/` must be rebuilt locally and
   committed, or nothing ships.

2. **`public/` has been hand-edited.** The umami analytics `<script>` and the
   `<meta name="generator">` tag appear only in `public/*.html` and in no source
   template. A naive `hugo` run drops analytics from every page. Moving umami
   into a template is a prerequisite, not an optional cleanup.

## Approach

Project-level `layouts/` overrides. Hugo's template lookup order places
`layouts/` ahead of `themes/etch/layouts/`, so copying the five templates that
need microformats leaves the submodule untouched. `git submodule update` stays
safe and upstream etch changes still reach every template not overridden.

Rejected: forking etch (inherits theme maintenance to add ~15 class attributes);
config-only (etch exposes no microformats hooks).

## Identity

| Field | Value |
|---|---|
| Name | mertalp |
| URL | `https://logprintf.com/` |
| rel=me | `https://github.com/matt0000000` |
| Webmention endpoint | `https://webmention.io/logprintf.com/webmention` |
| Authorization endpoint | `https://indieauth.com/auth` |

`token_endpoint` is deliberately omitted — it exists only to serve Micropub,
which is out of scope.

## Components

### 1. `layouts/partials/head.html`

Copy of the theme partial, with additions:

- `{{ hugo.Generator }}` — replaces the hand-inserted generator meta
- umami `<script defer src="https://cloud.umami.is/script.js"
  data-website-id="b7740ce6-8b64-4398-bcd9-cbb9d3a1a89f">` — moved from
  hand-edited HTML into source so rebuilds preserve it
- `<link rel="webmention" href="https://webmention.io/logprintf.com/webmention">`
- `<link rel="authorization_endpoint" href="https://indieauth.com/auth">`
- `<link rel="me" href="https://github.com/matt0000000">`

Values are hardcoded rather than driven by `config.toml` params. This is a
single-author personal site; indirection through params buys nothing here.

### 2. `layouts/_default/single.html`

The post becomes an `h-entry`:

- `class="h-entry"` on `<article>`
- `class="p-name"` on the `<h1>`
- `class="e-content"` on a `<div>` wrapping `.Content`
- `class="dt-published"` on the visible `<time>`
- `<a class="u-url">` carrying the permalink, using the HTML `hidden` attribute
- a `p-author h-card`, also using the `hidden` attribute

Hiding is done with the HTML `hidden` attribute throughout (never
`display:none` in CSS, and never omission). mf2 parsers walk the DOM and read
hidden elements normally, so this yields correct metadata with no visual
footprint.

**Date semantics.** The theme's `<time>` elements carry no `datetime` attribute,
so no parser can read them. Every `<time>` gains
`datetime="{{ .Date.Format "2006-01-02T15:04:05Z07:00" }}"`.

**Preserving current display.** The theme shows *only* the updated date when
`Lastmod != Date`, and only the created date otherwise. That visible behavior is
preserved exactly. In the `Lastmod != Date` branch the visible `<time>` gets
`dt-updated` and a separate hidden `<time class="dt-published">` is emitted, so
the entry has a published date without changing what a reader sees.

### 3. `layouts/partials/posts.html` and `layouts/_default/li.html`

- `class="h-feed"` on `<ul id="posts">`
- each `<li>` becomes `class="h-entry"` with `u-url` on the anchor, `p-name` on a
  `<span>` around the title, and `dt-published` + `datetime` on the `<time>`

`layouts/_default/list.html` needs no override — it delegates to `posts.html`.

### 4. `layouts/_default/taxonomy.html`

Same `h-feed` class on its `<ul id="posts">`. It renders `li.html`, so entries
are covered by component 3.

### 5. `layouts/index.html`

Homepage: existing `.Content`, then a hidden representative `h-card`, then the
h-feed. The h-card carries `p-name`, `u-url`, and `u-uid` all pointing at
`https://logprintf.com/` — `u-uid` matching `u-url` is what marks it
*representative*, which is what IndieAuth consumers look for.

Rendered output is visually identical to today's homepage.

### 6. `content/about/index.md`

A visible `h-card` block above the existing prose: name linking to
`logprintf.com`, and the GitHub link with `rel="me"`. Goldmark already has
`unsafe = true`, so inline HTML renders.

This is the only user-visible change in the whole design.

### 7. Rebuild and commit `public/`

Install Hugo 0.134.2 to match the version that produced the current output
(`go install github.com/gohugoio/hugo@v0.134.2` — the standard, non-extended
build is sufficient; the theme uses no SCSS). Run `hugo`, commit the result.

## Verification

1. `hugo` builds without error.
2. `grep -rL umami public --include='*.html'` returns nothing — i.e. no HTML
   page is missing the analytics script. This is the regression guard for
   Finding 2.
3. `git diff public/` reviewed — no unintended content or markup changes beyond
   the added microformats.
4. Rendered HTML parsed by an mf2 parser (pin13.net/mf2 or indiewebify.me) to
   confirm the h-card, h-feed, and h-entry structures actually parse, and that
   the homepage h-card is detected as representative.
5. Visual check that `/about/`, the homepage, and one post render correctly —
   specifically that the new `div.e-content` wrapper and the `span.p-name` in
   list items do not disturb etch's CSS.

## Risks

- **`div.e-content` wrapper may affect post styling.** Etch's CSS targets
  `#content` and bare element selectors, so an extra div should be inert, but
  this is the one change with real visual risk. Verification step 5 covers it.
- **Rebuilding `public/` may produce a large diff** if the installed Hugo's
  output differs from what generated the committed files. Any diff outside the
  intended changes must be inspected before committing.

## Manual steps for the site owner

Neither can be done from the repo:

1. Add `https://logprintf.com` to the GitHub profile website field. Without that
   backlink the rel=me relationship is one-directional and IndieAuth
   verification fails.
2. Sign in at webmention.io with `logprintf.com` to activate the endpoint. Until
   then the `rel="webmention"` link points at an inactive address.

## Out of scope

Chosen explicitly during design: displaying received webmentions, Micropub,
POSSE/Bridgy syndication, and a short-form notes post type.
