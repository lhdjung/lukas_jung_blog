# Lukas Jung's blog

Personal site and blog, built with Quarto and deployed on Netlify. Still under construction — all posts in `blog/posts/` are currently `draft: true`.

## Structure

- `index.qmd` — the "About" page (landing page, Publications list). This is **not** the blog.
- `blog/index.qmd` — the actual blog listing page. Currently just a "coming soon" placeholder; the `listing:` config that pulls in `blog/posts/` was removed until there's something real to publish.
- `blog/posts/*/index.qmd` — individual posts, all currently drafts.
- `theme.scss` — navbar, headings, and listing typography overrides. Body font/highlighting theme are otherwise Quarto/Bootstrap defaults.
- `_freeze/` — cached R chunk execution results (see freeze caveat below). Committed to git on purpose.
- `_site/` — build output. Gitignored; Netlify builds it fresh on every deploy (see below).

## Rendering locally

`quarto` isn't on `PATH` by default in this setup — use RStudio's bundled copy:

```sh
export PATH="/Applications/RStudio.app/Contents/Resources/app/quarto/bin:$PATH"
quarto render          # full project render
quarto preview          # live-reloading local preview
```

Or just use RStudio's Render/Preview buttons, which use the same bundled binary.

## Deploying

Netlify builds from `netlify.toml`'s `command`, which downloads Quarto and runs `quarto render`, then publishes `_site/`. Nothing needs to be pre-rendered or committed for a deploy to pick up changes — push to `master` and Netlify does the rest.

## Caveats

- **Two `index.qmd` files, easy to mix up.** `index.qmd` (root) is the About page; `blog/index.qmd` is the blog. Rendering/previewing the root one will never show blog posts — they're unrelated pages that happen to share a filename.

- **Hardcoded Quarto version in `netlify.toml`.** The build command downloads a *pinned* Quarto version (currently `1.9.38`, matching the RStudio-bundled version used locally). If you upgrade Quarto locally, update the version in `netlify.toml` too — otherwise local renders and the deployed build can silently diverge, or the deploy can start failing if that version ever gets pulled from GitHub releases.

- **Rendering a single draft file leaks all drafts into listings.** Running `quarto render some/draft/index.qmd` directly (or letting `quarto preview`'s file-watcher do an incremental rebuild of a draft page) puts Quarto into a mode where *all* draft posts become visible in any listing rendered in that pass — not just the one you're editing. This looks like drafts "escaped" but nothing is actually wrong with `draft: true` in the source files. Fix: do a full `quarto render` (no target file) or restart `quarto preview`, and check the listing output again before assuming something's broken.

- **R chunks need `_freeze/` cache, or a re-render before pushing.** Netlify's build image doesn't have R installed. `freeze: true` is set at the project level in `_quarto.yml`, so `quarto render` reuses cached results from `_freeze/` instead of executing R — but that cache only updates when you render locally (where R *is* available). If you add or change an R code chunk anywhere, render locally first and commit the updated `_freeze/*.json` before pushing, or the Netlify build will either fail or silently serve stale output.

- **`_site/` is gitignored now — don't hand-commit it.** This used to be the deploy mechanism (Netlify served whatever `_site/` was committed, with no build step), which is how a stale `favicon.png` and a pile of duplicate hashed `bootstrap-*.min.css` files once ended up permanently in git history. Now Netlify builds `_site/` itself; the local copy is disposable and shouldn't be staged.

- **Hand-edited YAML files can get silently mangled.** `_quarto.yml`, `blog/posts/_metadata.yml`, and similar have been observed coming back with all whitespace/indentation stripped (e.g. `title:"Lukas Jung"` instead of `title: "Lukas Jung"`) after being saved through certain tool integrations. If a render suddenly fails with a YAML parse error right after an edit, re-check the file's actual formatting before assuming the content itself is wrong.
