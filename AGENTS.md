# AGENTS.md — sebastian-just.com

Primary personal site. The site is a resume generated from `resume.json` (JSON Resume
schema) using [resume-cli](https://github.com/jsonresume/resume-cli) with the
`macchiato` theme.

## Repo layout
- `resume.json` — the single source of truth for the site content.
- `package.json` — pins `resume-cli` + `jsonresume-theme-macchiato`; defines the scripts below.
- `.github/workflows/validate.yml` — runs on PRs to `main`.
- `.github/workflows/deploy.yml` — runs on push to `main` (+ manual dispatch).
- `CNAME`, `favicon.ico` — static assets copied into the published site.
- `profile.jpeg` — present but NOT used by the macchiato theme (it ignores `basics.image`).
- `public/`, `node_modules/` — generated/installed, git-ignored.

## Commands
- Install: `PUPPETEER_SKIP_DOWNLOAD=true npm install` (see note below).
- Validate: `npm run validate` → `resume validate resume.json`.
- Build:    `npm run build` → `mkdir -p public && resume export public/index.html --format html --theme macchiato`.
  - `resume export` does NOT create parent dirs — always `mkdir -p` first.

## Important gotchas
- **PDF export is intentionally not used.** `resume-cli` pulls in full `puppeteer`,
  whose postinstall downloads Chromium. We only export HTML, so set
  `PUPPETEER_SKIP_DOWNLOAD=true` for installs (the workflows set it as a job `env`).
  The bundled Chromium is also broken on Apple Silicon locally (spawn error -88).
- **Node 24.** Use `node-version: 24` and node24-runtime actions
  (`actions/checkout@v7`, `actions/setup-node@v7`); node20-runtime actions are deprecated.

## Deployment (GitHub Pages)
- On merge to `main`, `deploy.yml` builds and publishes `./public` to the **`gh-pages`**
  branch via `peaceiris/actions-gh-pages@v4` (force_orphan), copying `CNAME`/`favicon.ico` in.
- GitHub Pages source = `gh-pages` branch, path `/`. Custom domain `sebastian-just.com`
  is kept alive by the `CNAME` file in the published output.
- `sebastian-just.de` and `sebastianjust.de` are separate repos that redirect here.

## Branch protection on `main`
- PRs required before merge; the **`validate`** status check must pass (strict / up-to-date).
- `enforce_admins = false` — repo admins (derjust) can bypass when needed.
- Direct force-pushes and branch deletion are disabled.
- The required check name is `validate` (the job id in `validate.yml`) — keep them in sync.
