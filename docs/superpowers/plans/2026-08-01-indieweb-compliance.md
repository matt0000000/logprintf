# IndieWeb Compliance Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add foundations-level IndieWeb support to logprintf.com — microformats2 markup (h-card, h-entry, h-feed), `rel=me` identity, a webmention receiving endpoint, and IndieAuth sign-in — with no visual change except a new h-card on `/about/`.

**Architecture:** Project-level `layouts/` overrides shadow the `themes/etch` submodule via Hugo's template lookup order. Five templates are copied from the theme and annotated with microformats classes. The submodule is never modified. `public/` is rebuilt locally and committed, because Cloudflare Pages serves that directory verbatim with no build command.

**Tech Stack:** Hugo 0.134.2 (standard, non-extended), Go 1.26.5, Go templates, microformats2.

**Spec:** `docs/superpowers/specs/2026-08-01-indieweb-compliance-design.md`

## Global Constraints

- **Never modify `themes/etch/`.** It is a git submodule. All changes go in project-level `layouts/`.
- **Hugo version is exactly 0.134.2.** Install with `go install github.com/gohugoio/hugo@v0.134.2`. The theme uses no SCSS, so the non-extended build is sufficient.
- **Never run `hugo --cleanDestinationDir`.** It deletes files in `public/` that Hugo did not generate.
- **`public/` is a build artifact that MUST be committed.** Cloudflare Pages has no build command and serves the committed directory. Uncommitted `public/` means nothing ships.
- **The umami script must appear in every generated HTML page.** It currently exists only as a hand-edit inside `public/*.html` and in no source template. Losing it is the primary regression risk of this work.
- Identity values, used verbatim everywhere:
  - Name: `mertalp`
  - Site URL: `https://logprintf.com/`
  - rel=me: `https://github.com/matt0000000`
  - Webmention endpoint: `https://webmention.io/logprintf.com/webmention`
  - Authorization endpoint: `https://indieauth.com/auth`
  - umami website id: `b7740ce6-8b64-4398-bcd9-cbb9d3a1a89f`
- **Hiding uses the HTML `hidden` attribute**, never CSS `display:none` and never omission. mf2 parsers read hidden elements from the DOM.
- **Date format string is `"2006-01-02T15:04:05Z07:00"`** everywhere a `datetime` attribute is emitted.
- **Verification is HTML-diff based, not a test framework.** This repo has no test runner. "Test" means: build, then assert on the generated HTML in `public/`.
- **Do not deploy or push.** Commit locally only.

---

### Task 1: Install Hugo and establish a reproducible baseline

Before changing any template, prove that a clean build reproduces the committed
`public/`. Whatever differs is exactly the set of hand-edits that later tasks
must reintroduce through templates. Skipping this makes every later diff
unreadable.

**Files:**
- Create: `/tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/baseline.diff` (scratch, not committed)
- Modify: none

**Interfaces:**
- Consumes: nothing
- Produces: a verified `hugo` binary on `PATH`, and a recorded baseline diff that later tasks compare against

- [ ] **Step 1: Install Hugo 0.134.2**

```bash
go install github.com/gohugoio/hugo@v0.134.2
export PATH="$PATH:$(go env GOPATH)/bin"
hugo version
```

Expected: output contains `hugo v0.134.2`.

- [ ] **Step 2: Confirm the theme submodule is populated**

```bash
cd /home/mertalp/logprintf
git submodule update --init --recursive
ls themes/etch/layouts/_default/single.html
```

Expected: the file exists. If `themes/etch` is empty, every build silently produces bare pages.

- [ ] **Step 3: Build into a throwaway directory and diff against committed `public/`**

Do NOT build over `public/` yet — that would destroy the baseline being measured.

```bash
cd /home/mertalp/logprintf
hugo --destination /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/baseline-public
diff -ru public /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/baseline-public \
  > /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/baseline.diff 2>&1 || true
cat /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/baseline.diff
```

Expected differences, and ONLY these:
1. Every HTML file loses `<script defer src="https://cloud.umami.is/script.js" ...>`
2. `public/index.html` loses `<meta name="generator" content="Hugo 0.134.2">`

- [ ] **Step 4: Judge the baseline**

If the diff contains only the two expected differences, continue to Task 2.

If the diff contains anything else — different CSS hashes, reordered
attributes, changed dates, missing pages — **stop and report the full diff.**
It means the installed Hugo does not reproduce the committed output, and
committing a rebuild would ship unintended changes. Do not proceed.

- [ ] **Step 5: No commit**

This task changes no tracked files. Nothing to commit.

---

### Task 2: Override `head.html` with umami, generator, and IndieWeb link relations

This task both adds the IndieWeb discovery links and closes the umami
regression. Doing them together means the very first rebuild is already safe.

**Files:**
- Create: `layouts/partials/head.html`
- Modify: none

**Interfaces:**
- Consumes: Hugo's template lookup order (project `layouts/` shadows `themes/etch/layouts/`)
- Produces: every page carries `rel=webmention`, `rel=authorization_endpoint`, `rel=me`, the generator meta, and the umami script

- [ ] **Step 1: Write the verification check and watch it fail**

```bash
cd /home/mertalp/logprintf
hugo --destination /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t2
grep -rL 'cloud.umami.is' /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t2 --include='*.html'
```

Expected: lists EVERY html file (all are missing umami). This is the failing state.

- [ ] **Step 2: Create the override**

Create `layouts/partials/head.html` with exactly this content. Everything
outside the `IndieWeb` comment block and the trailing `<script>` is copied
verbatim from `themes/etch/layouts/partials/head.html`.

```go-html-template
<head>
    {{ hugo.Generator }}
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    {{ with .Site.Params.description -}}
    <meta name="description" content="{{ . }}">
    {{ end }}
    {{ printf `<link rel="shortcut icon" href="%s">` ("favicon.ico" | absURL) | safeHTML }}
    {{ with .OutputFormats.Get "rss" -}}
        {{ printf `<link rel="%s" type="%s" href="%s" title="%s">` .Rel .MediaType.Type .Permalink $.Site.Title | safeHTML }}
    {{ end -}}

    {{/* IndieWeb: webmention receiving endpoint, IndieAuth, identity */}}
    <link rel="webmention" href="https://webmention.io/logprintf.com/webmention">
    <link rel="authorization_endpoint" href="https://indieauth.com/auth">
    <link rel="me" href="https://github.com/matt0000000">

    {{ $resources := slice -}}

    {{ $resources = $resources | append (resources.Get "css/main.css") -}}

    {{ $resources = $resources | append (resources.Get "css/min770px.css") -}}

    {{ $dark := .Site.Params.dark | default "auto" -}}
    {{ if not (eq $dark "off") -}}
        {{ $resources = $resources | append (resources.Get "css/dark.css" | resources.ExecuteAsTemplate "dark.css" .) -}}
    {{ end -}}

    {{ if .Site.Params.highlight -}}
        {{ $resources = $resources | append (resources.Get "css/syntax.css") -}}
    {{ end -}}

    {{ $css := $resources | resources.Concat "css/style.css" | minify }}
    {{ printf `<link rel="stylesheet" href="%s">` $css.RelPermalink | safeHTML }}

    <link rel="canonical" href="{{ .Permalink }}" />
    <title>{{ .Title }}</title>
    <script defer src="https://cloud.umami.is/script.js" data-website-id="b7740ce6-8b64-4398-bcd9-cbb9d3a1a89f"></script>
</head>
```

- [ ] **Step 3: Rebuild and verify all four additions on every page**

```bash
cd /home/mertalp/logprintf
rm -rf /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t2
hugo --destination /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t2
for needle in 'cloud.umami.is' 'rel="webmention"' 'rel="authorization_endpoint"' 'github.com/matt0000000'; do
  echo "== missing $needle in:"
  grep -rL "$needle" /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t2 --include='*.html'
done
```

Expected: each `== missing ...` header is followed by no filenames.

- [ ] **Step 4: Confirm the CSS bundle hash did not change**

```bash
grep -o 'css/style\.min\.css' /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t2/index.html
```

Expected: prints `css/style.min.css`. A different filename means the asset
pipeline was disturbed and every page's stylesheet link changed.

- [ ] **Step 5: Commit**

```bash
cd /home/mertalp/logprintf
git add layouts/partials/head.html
git commit -m "feat: add IndieWeb discovery links, move umami into template

Adds rel=webmention, rel=authorization_endpoint and rel=me to every page.

Also moves the umami analytics script and the generator meta tag from
hand-edited public/*.html into the template, so rebuilding no longer
drops analytics."
```

---

### Task 3: Mark up single posts as `h-entry`

**Files:**
- Create: `layouts/_default/single.html`
- Modify: none

**Interfaces:**
- Consumes: `layouts/partials/head.html` from Task 2 (already in place; not referenced directly)
- Produces: every post page contains one `h-entry` with `p-name`, `e-content`, `dt-published`, `u-url`, and a `p-author h-card`

- [ ] **Step 1: Confirm the failing state**

```bash
grep -c 'h-entry' /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t2/kworker-high-cpu-usage-fix/index.html
```

Expected: `0`.

- [ ] **Step 2: Create the override**

Create `layouts/_default/single.html`:

```go-html-template
{{ define "main" }}
<article class="h-entry">
    <header id="post-header">
        <h1 class="p-name">{{ .Title }}</h1>
        <div>
        {{- if isset .Params "date" -}}
            {{ if eq .Lastmod .Date }}
                <time class="dt-published" datetime="{{ .Date.Format "2006-01-02T15:04:05Z07:00" }}">{{ .Date | time.Format (i18n "post.created") }}</time>
            {{ else }}
                <time class="dt-published" datetime="{{ .Date.Format "2006-01-02T15:04:05Z07:00" }}" hidden></time>
                <time class="dt-updated" datetime="{{ .Lastmod.Format "2006-01-02T15:04:05Z07:00" }}">{{ .Lastmod | time.Format (i18n "post.updated") }}</time>
            {{ end }}
        {{- end -}}
        </div>
    </header>
    <a class="u-url" href="{{ .Permalink }}" hidden></a>
    <div class="p-author h-card" hidden>
        <a class="p-name u-url" href="{{ site.Home.Permalink }}">mertalp</a>
        <a class="u-url" rel="me" href="https://github.com/matt0000000">github.com/matt0000000</a>
    </div>
    <div class="e-content">
    {{- .Content -}}
    </div>
</article>
{{ end }}
```

Note on the `else` branch: the theme displays *only* the updated date when
`Lastmod != Date`. That visible behavior is preserved exactly; the published
date is carried by an empty `hidden` `<time>` so the entry still has one.

- [ ] **Step 3: Rebuild and verify the markup**

```bash
cd /home/mertalp/logprintf
rm -rf /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t3
hugo --destination /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t3
P=/tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t3/kworker-high-cpu-usage-fix/index.html
for needle in 'class="h-entry"' 'class="p-name"' 'class="e-content"' 'class="dt-published"' 'datetime="2024-09-01' 'p-author h-card'; do
  printf '%s -> %s\n' "$needle" "$(grep -c "$needle" "$P")"
done
```

Expected: every line ends in a count of `1` or greater.

- [ ] **Step 4: Verify no post page lost its content**

```bash
for f in /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t3/*/index.html; do
  printf '%s %s\n' "$(wc -c < "$f")" "$f"
done
```

Expected: no post page is drastically smaller than its counterpart under
`public/`. Compare against `wc -c public/*/index.html` if unsure.

- [ ] **Step 5: Commit**

```bash
cd /home/mertalp/logprintf
git add layouts/_default/single.html
git commit -m "feat: mark up posts as h-entry

Adds p-name, e-content, dt-published, u-url and a hidden p-author h-card.
Also adds machine-readable datetime attributes, which the theme's <time>
elements previously lacked entirely."
```

---

### Task 4: Mark up post listings as `h-feed`

Covers the homepage list, `/posts/`, and every tag and category page. All three
render through `li.html`, so the entry markup is written once.

**Files:**
- Create: `layouts/_default/li.html`
- Create: `layouts/partials/posts.html`
- Create: `layouts/_default/taxonomy.html`
- Modify: none

**Interfaces:**
- Consumes: nothing from earlier tasks
- Produces: `<ul id="posts" class="h-feed">` containing `li.h-entry` items with `u-url`, `p-name`, `dt-published`

`layouts/_default/list.html` deliberately gets no override — the theme version
already delegates to `posts.html`.

- [ ] **Step 1: Confirm the failing state**

```bash
grep -c 'h-feed' /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t3/index.html
```

Expected: `0`.

- [ ] **Step 2: Create `layouts/_default/li.html`**

```go-html-template
<li class="h-entry">
    <a class="u-url" href="{{ .Permalink }}">
        <span class="p-name">{{ .Title }}</span>
        <small><time class="dt-published" datetime="{{ .Date.Format "2006-01-02T15:04:05Z07:00" }}">{{ .Date | time.Format (i18n "posts.date") }}</time></small>
    </a>
</li>
```

- [ ] **Step 3: Create `layouts/partials/posts.html`**

```go-html-template
<h3>{{ i18n "posts.title" }}</h3>
<ul id="posts" class="h-feed">
{{- range where site.RegularPages "Type" "in" site.Params.mainSections }}
    {{ .Render "li" }}
{{- end }}
</ul>
```

- [ ] **Step 4: Create `layouts/_default/taxonomy.html`**

```go-html-template
{{ define "main" }}
<h3>{{ .Title }}</h3>
<ul id="posts" class="h-feed">
{{- range .Pages }}
    {{ .Render "li" }}
{{- end }}
</ul>
{{ end }}
```

- [ ] **Step 5: Rebuild and verify feeds on all three page kinds**

```bash
cd /home/mertalp/logprintf
rm -rf /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t4
hugo --destination /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t4
T=/tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t4
for f in "$T/index.html" "$T/posts/index.html" "$T/tags/debian/index.html"; do
  printf '%s h-feed=%s h-entry=%s\n' "$f" "$(grep -c 'h-feed' "$f")" "$(grep -c 'h-entry' "$f")"
done
```

Expected: `h-feed=1` on each, and `h-entry` greater than 0 on each.

- [ ] **Step 6: Verify the homepage still lists all 8 posts**

```bash
grep -c '<li class="h-entry">' /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t4/index.html
```

Expected: `8`.

- [ ] **Step 7: Commit**

```bash
cd /home/mertalp/logprintf
git add layouts/_default/li.html layouts/partials/posts.html layouts/_default/taxonomy.html
git commit -m "feat: mark up post listings as h-feed

Homepage, /posts/ and taxonomy pages become h-feeds of h-entry items
with u-url, p-name and machine-readable dt-published."
```

---

### Task 5: Add the representative `h-card` to the homepage

An h-card is *representative* of a page when its `u-uid` and `u-url` both equal
the page's own URL. That is what IndieAuth consumers look for when they resolve
`logprintf.com` to a person.

**Files:**
- Create: `layouts/index.html`
- Modify: none

**Interfaces:**
- Consumes: `layouts/partials/posts.html` from Task 4
- Produces: homepage contains exactly one representative `h-card`, hidden, with no visual change

- [ ] **Step 1: Confirm the failing state**

```bash
grep -c 'h-card' /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t4/index.html
```

Expected: `0`.

- [ ] **Step 2: Create `layouts/index.html`**

```go-html-template
{{ define "main" }}
{{ .Content }}
<div class="h-card" hidden>
    <a class="p-name u-url u-uid" href="{{ site.Home.Permalink }}">mertalp</a>
    <a class="u-url" rel="me" href="https://github.com/matt0000000">github.com/matt0000000</a>
</div>
{{- partial "posts.html" . -}}
{{ end }}
```

- [ ] **Step 3: Rebuild and verify**

```bash
cd /home/mertalp/logprintf
rm -rf /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t5
hugo --destination /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t5
grep -o 'class="p-name u-url u-uid" href="[^"]*"' /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t5/index.html
```

Expected exactly: `class="p-name u-url u-uid" href="https://logprintf.com/"`

The href MUST end in a trailing slash and MUST match the canonical URL on the
same page, or the h-card is not representative.

- [ ] **Step 4: Verify the h-card is hidden and the page is otherwise unchanged**

```bash
diff <(sed 's/<div class="h-card" hidden>.*<\/div>//' /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t5/index.html) \
     /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t4/index.html
```

Expected: differences confined to the h-card block only.

- [ ] **Step 5: Commit**

```bash
cd /home/mertalp/logprintf
git add layouts/index.html
git commit -m "feat: add representative h-card to homepage

Hidden h-card with matching u-url and u-uid, which is what marks it
representative for IndieAuth. Homepage renders identically."
```

---

### Task 6: Add the visible `h-card` to the About page

The only user-visible change in this plan.

**Files:**
- Modify: `content/about/index.md`

**Interfaces:**
- Consumes: nothing
- Produces: `/about/` renders a visible h-card above the existing prose

- [ ] **Step 1: Confirm the failing state**

```bash
grep -c 'h-card' /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t5/about/index.html
```

Expected: `0`.

- [ ] **Step 2: Rewrite `content/about/index.md`**

Goldmark already has `unsafe = true` in `config.toml`, so inline HTML renders.
The blank line after `</div>` is required — without it Goldmark treats the
following prose as part of the HTML block.

```markdown
+++
title = "About"
+++
<div class="h-card">
  <p><a class="p-name u-url u-uid" href="https://logprintf.com/" rel="me">mertalp</a></p>
  <p><a class="u-url" rel="me" href="https://github.com/matt0000000">github.com/matt0000000</a></p>
</div>

This website is powered by [Hugo](https://github.com/gohugoio/hugo). Theme being used is [Etch by Lukas Joswiak](https://github.com/LukasJoswiak/etch). It is deployed on [Cloudflare Pages](https://pages.cloudflare.com).
```

- [ ] **Step 3: Rebuild and verify both the h-card and the surviving prose**

```bash
cd /home/mertalp/logprintf
rm -rf /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t6
hugo --destination /tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t6
A=/tmp/claude-1000/-home-mertalp-logprintf/5ffb2ab5-d4ad-4130-aeaf-afc7872298b6/scratchpad/t6/about/index.html
grep -c 'class="h-card"' "$A"
grep -c 'gohugoio/hugo' "$A"
grep -c 'pages.cloudflare.com' "$A"
```

Expected: `1`, `1`, `1`. If the Hugo/Cloudflare links are missing, the blank
line after `</div>` was omitted and Goldmark swallowed the prose.

- [ ] **Step 4: Confirm the markdown links rendered as anchors, not literal text**

```bash
grep -o '<a href="https://github.com/gohugoio/hugo">Hugo</a>' "$A"
```

Expected: prints the anchor.

- [ ] **Step 5: Commit**

```bash
cd /home/mertalp/logprintf
git add content/about/index.md
git commit -m "feat: add visible h-card to about page"
```

---

### Task 7: Rebuild `public/` and commit the deployable output

Nothing shipped until this task. Cloudflare Pages serves the committed
`public/` directory verbatim.

**Files:**
- Modify: every file under `public/`

**Interfaces:**
- Consumes: all templates from Tasks 2-6
- Produces: a committed `public/` carrying all IndieWeb markup

- [ ] **Step 1: Build over the real `public/` directory**

Without `--cleanDestinationDir`. See Global Constraints.

```bash
cd /home/mertalp/logprintf
hugo
```

Expected: no errors.

- [ ] **Step 2: Run the umami regression guard**

This is the check that protects the site's analytics.

```bash
cd /home/mertalp/logprintf
grep -rL 'cloud.umami.is' public --include='*.html'
```

Expected: no output at all. Any filename listed is a page that lost analytics —
stop and fix before committing.

- [ ] **Step 3: Run the full IndieWeb assertion sweep**

```bash
cd /home/mertalp/logprintf
for needle in 'rel="webmention"' 'rel="authorization_endpoint"' 'rel="me"'; do
  printf '%s missing from: %s\n' "$needle" "$(grep -rL "$needle" public --include='*.html' | tr '\n' ' ')"
done
printf 'homepage h-card: %s\n' "$(grep -c 'class="h-card"' public/index.html)"
printf 'homepage h-feed: %s\n' "$(grep -c 'h-feed' public/index.html)"
printf 'homepage entries: %s\n' "$(grep -c '<li class="h-entry">' public/index.html)"
printf 'post h-entry: %s\n' "$(grep -c 'class="h-entry"' public/kworker-high-cpu-usage-fix/index.html)"
printf 'post e-content: %s\n' "$(grep -c 'class="e-content"' public/kworker-high-cpu-usage-fix/index.html)"
printf 'about h-card: %s\n' "$(grep -c 'class="h-card"' public/about/index.html)"
```

Expected: the three `missing from:` lines are empty after the colon; then
`1`, `1`, `8`, `1`, `1`, `1`.

- [ ] **Step 4: Review the diff for unintended changes**

```bash
cd /home/mertalp/logprintf
git diff --stat public | tail -5
git diff public/kworker-high-cpu-usage-fix/index.html
```

Expected: changes limited to added microformats classes, `datetime`
attributes, the IndieWeb `<link>` tags, and the e-content wrapper. Report
anything else rather than committing it.

- [ ] **Step 5: Commit the build output**

```bash
cd /home/mertalp/logprintf
git add public
git commit -m "build: regenerate public/ with IndieWeb markup

Cloudflare Pages serves this directory verbatim, so the build output
must be committed for the changes to reach the live site."
```

---

## Definition of Done

- [ ] `layouts/` contains 6 override files; `themes/etch/` is untouched (`git status` shows no submodule change)
- [ ] `grep -rL 'cloud.umami.is' public --include='*.html'` prints nothing
- [ ] Homepage has a representative h-card (`u-url` == `u-uid` == `https://logprintf.com/`) and an h-feed of 8 h-entries
- [ ] Every post is an h-entry with `p-name`, `e-content`, `dt-published` + `datetime`, `u-url`, `p-author`
- [ ] `/about/` shows a visible h-card and retains its three original links
- [ ] `public/` is rebuilt and committed
- [ ] Nothing pushed, nothing deployed

## Out of Scope

Displaying received webmentions, Micropub, POSSE/Bridgy, notes post type. Also
out of scope, though noticed during planning: the site has no `favicon.ico`
despite linking one, the IndexNow key file was deleted from git in `02790d8`,
and Cloudflare returns HTTP 200 with the homepage for unknown paths.
