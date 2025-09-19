
# New Docs Site — MkDocs + Material (High-Level Plan)

## Goal
Stand up a new, lightweight docs site with **MkDocs + Material**, collaboratively maintained via Git + PRs, deployed as a static site with preview builds per PR.

---

## Tech Stack
- **Generator:** MkDocs + Material theme (Markdown → static site, built-in search)
- **Repo:** GitHub (public or private)
- **Hosting:** Cloudflare Pages (or Vercel/Netlify)
- **CI/CD:** GitHub Actions (build on PR + main)
- **Cost:** ~$0 (free tiers) + domain you already own

---

## Team Workflow
- Devs clone the repo, run locally, open PRs.
- CI blocks merges if the site fails to build.
- Each PR gets an isolated preview URL.
- Merges to `main` auto-deploy to production.

---

## Project Structure
```
/docs
  index.md
  guide.md
  faq.md
mkdocs.yml
requirements.txt
.github/
  workflows/
    mkdocs-ci.yml
```

**Content rules**
- All pages live in `/docs`.
- Sidebar/nav is defined in `mkdocs.yml` (`nav:` list).
- Use relative links and keep images under `docs/assets/`.

---

## Local Development
```bash
# one-time
pip install -r requirements.txt

# live preview
mkdocs serve       # http://127.0.0.1:8000

# production build
mkdocs build       # outputs to ./site
```

**requirements.txt**
```
mkdocs==1.6.1
mkdocs-material==9.5.26
```

**mkdocs.yml (starter)**
```yaml
site_name: Project Docs
theme:
  name: material
nav:
  - Home: index.md
  - Guide: guide.md
  - FAQ: faq.md
markdown_extensions:
  - admonition
  - toc:
      permalink: true
```

---

## CI/CD (GitHub Actions)
- On every **PR**: install deps → `mkdocs build --strict`
- On **main**: same build → host deploy handled by Pages/Vercel/Netlify

Minimal workflow: `.github/workflows/mkdocs-ci.yml`
```yaml
name: Build Docs
on:
  pull_request: { branches: [ main ] }
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install -r requirements.txt
      - run: mkdocs build --strict
```

---

## Hosting Options
**Cloudflare Pages (recommended)**
- Build: `pip install -r requirements.txt && mkdocs build`
- Output dir: `site`
- Features: free bandwidth, PR preview URLs, easy custom domain
- Privacy: add Cloudflare Access later if needed (SSO/email gate)

**Vercel / Netlify**
- Same build/output settings; enable password/SSO or basic auth if needed.

---

## Versioning / Structure (Optional)
- If you need multiple versions of docs, use separate branches or MkDocs plugins for versioning later.
- Keep sections shallow; add `index.md` in subfolders to make section landing pages.

---

## Content Quality Guidelines
- Short pages with clear headings; first paragraph states the “why”.
- One task per page where possible, with a copy-pasteable “Quickstart”.
- Add code blocks with language hints (```bash, ```js).
- Use admonitions (`!!! note|warning|tip`) for callouts.

---

## Rollout Checklist
1. Create repo and scaffold files.
2. Add initial pages (`index.md`, `guide.md`, `faq.md`).
3. Enable CI; verify PR build is red/green correctly.
4. Connect hosting; confirm PR previews + production.
5. Map custom domain (e.g., `docs.example.com`).
6. Share contributor guide (clone → `mkdocs serve` → PR).
