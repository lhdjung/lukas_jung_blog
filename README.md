# Lukas Jung's blog

Personal site and blog, built with Quarto and deployed on Netlify. Still under construction — all posts in `blog/posts/` are currently `draft: true`.

## Structure

- `index.qmd` — the "About" page (landing page, Publications list). This is **not** the blog.
- `blog/index.qmd` — the actual blog listing page. Currently just a "coming soon" placeholder; the `listing:` config that pulls in `blog/posts/` was removed until there's something real to publish.
- `blog/posts/*/index.qmd` — individual posts, all currently drafts.
- `theme.scss` — navbar, headings, and listing typography overrides. Body font/highlighting theme are otherwise Quarto/Bootstrap defaults.
- `_freeze/` — cached R chunk execution results, so re-renders don't have to re-execute unchanged code. Committed to git.
- `_site/` — build output. **This is what gets deployed** (see below) and is committed to git.

## Rendering locally

`quarto` isn't on `PATH` by default in this setup — use RStudio's bundled copy:

```sh
export PATH="/Applications/RStudio.app/Contents/Resources/app/quarto/bin:$PATH"
quarto render          # full project render
quarto preview          # live-reloading local preview
```

Or just use RStudio's Render/Preview buttons, which use the same bundled binary.

## Deploying

Netlify has **no build step** — `netlify.toml` just points `publish` at `_site/`, and Netlify serves whatever's currently committed there, as-is. This means:

- Editing `.qmd` source files and pushing does **nothing** to the live site by itself. You have to render locally and commit the resulting `_site/` changes too.
- Always render with a clean slate before committing: `rm -rf _site && quarto render`. A non-clean re-render can leave orphaned files behind (old content-hashed CSS/JS from a previous render, stale assets no page references anymore) that then get committed alongside the real changes.
- We tried the alternative — a Netlify build step running `quarto render` remotely — and reverted it after two failed production deploys. Worth knowing why, so nobody re-attempts it blind:
  - Netlify's build image has no R installed, and any post whose `_freeze/` cache gets invalidated (e.g. by editing its frontmatter) falls back to actually executing R chunks, which then fails.
  - Downloading and extracting Quarto's own release tarball into the repo directory made `quarto render` discover Quarto's *own* bundled example templates as if they were project files, and choke trying to render one.
  - If this route is revisited, it'd need R installed on Netlify too (e.g. via a buildpack), not just Quarto.

## Footguns

- **Two `index.qmd` files, easy to mix up.** `index.qmd` (root) is the About page; `blog/index.qmd` is the blog. Rendering/previewing the root one will never show blog posts — they're unrelated pages that happen to share a filename.

- **A source change isn't live until you render and commit `_site/` too.** See "Deploying" above — this is the single easiest way to think a fix shipped when it didn't.

- **Rendering a single draft file leaks all drafts into listings.** Running `quarto render some/draft/index.qmd` directly (or letting `quarto preview`'s file-watcher do an incremental rebuild of a draft page) puts Quarto into a mode where *all* draft posts become visible in any listing rendered in that pass — not just the one you're editing. This looks like drafts "escaped" but nothing is actually wrong with `draft: true` in the source files. Fix: do a full, clean `quarto render` (no target file) and check the listing output again before assuming something's broken.

- **Editing a post's frontmatter invalidates its `_freeze/` cache.** Freeze is keyed off a hash of the whole source file, not just the code chunks. Removing/changing a YAML field is enough to force R to re-execute on the next render — harmless locally (R is installed), but it's the reason a Netlify-side build isn't viable without R there too.

- **Hand-edited YAML files can get silently mangled.** `_quarto.yml`, `blog/posts/_metadata.yml`, and similar have been observed coming back with all whitespace/indentation stripped (e.g. `title:"Lukas Jung"` instead of `title: "Lukas Jung"`) after being saved through certain tool integrations. If a render suddenly fails with a YAML parse error right after an edit, re-check the file's actual formatting before assuming the content itself is wrong.
