# Hugo Blog Setup: Private Repo to Public GitHub Pages

This guide documents how to use a private repository for your Hugo blog source code while deploying to a public GitHub Pages site for free hosting.

## Architecture

```
┌─────────────────────┐         ┌─────────────────────────┐
│  Private Repo       │         │  Public Repo            │
│  (blog source)      │ ──────► │  (jackhale98.github.io) │
│                     │  deploy │                         │
│  - content/         │         │  - Built HTML/CSS/JS    │
│  - themes/          │         │  - GitHub Pages serves  │
│  - hugo.toml        │         │    the site             │
└─────────────────────┘         └─────────────────────────┘
```

## Setup Steps

### 1. Create the Public Repository

Create a new **public** repository on GitHub named `jackhale98.github.io`.

This will be your GitHub Pages site, accessible at `https://jackhale98.github.io/`.

### 2. Create a Personal Access Token (PAT)

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. Click "Generate new token"
3. Configuration:
   - **Name:** `Blog Deploy` (or similar)
   - **Expiration:** Set as needed (recommend 90 days, then rotate)
   - **Repository access:** Select "Only select repositories" → choose `jackhale98.github.io`
   - **Permissions → Repository permissions:**
     - **Contents:** Read and write
     - **Pages:** Read and write (if available)
4. Generate and copy the token immediately (you won't see it again)

### 3. Add the Secret to Your Private Repo

1. Go to your private blog repo → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. **Name:** `DEPLOY_TOKEN`
4. **Value:** Paste your PAT from step 2
5. Click "Add secret"

### 4. Update Hugo Configuration

Edit `hugo.toml` and update the base URL:

```toml
baseURL = 'https://jackhale98.github.io/'
```

### 5. Enable GitHub Pages on Public Repo

1. Go to `jackhale98.github.io` repository → Settings → Pages
2. Under "Build and deployment":
   - **Source:** Deploy from a branch
   - **Branch:** `main`
   - **Folder:** `/ (root)`
3. Save

### 6. Push and Deploy

Commit and push your changes to the private repo's `main` branch. The GitHub Action will automatically:

1. Build your Hugo site
2. Push the built files to the public repo
3. GitHub Pages will serve the site

## Writing Content

### Markdown Posts

```bash
hugo new posts/my-new-post.md
```

Example front matter:

```markdown
+++
title = 'My New Post'
date = 2025-11-26T10:00:00-05:00
draft = false
+++

Your content here...
```

### Org-Mode Posts

```bash
hugo new posts/my-new-post.org
```

Example front matter (org-mode keywords):

```org
#+title: My New Post
#+date: 2025-11-26T10:00:00-05:00
#+draft: false

Your content here...
```

Hugo natively supports both `.md` and `.org` files without additional configuration.

## GitHub Actions Workflow

The workflow (`.github/workflows/hugo.yaml`) runs on every push to `main`:

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source repo
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build site
        run: hugo --minify

      - name: Deploy to GitHub Pages repo
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          token: ${{ secrets.DEPLOY_TOKEN }}
          repository-name: jackhale98/jackhale98.github.io
          branch: main
          folder: public
          single-commit: true
          commit-message: "Deploy from source repo"
```

## Troubleshooting

### Deployment Fails with Permission Error

- Verify the PAT has `Contents: Read and write` permission
- Check that the PAT is scoped to the correct public repository
- Ensure the secret name matches exactly: `DEPLOY_TOKEN`

### Site Shows 404

- Verify GitHub Pages is enabled on the public repo
- Check that the branch is set to `main`
- Ensure `baseURL` in `hugo.toml` matches your GitHub Pages URL

### Org-Mode Not Rendering

- Hugo uses [go-org](https://github.com/niklasfasching/go-org) for org-mode support
- Some advanced org-mode features may not be supported
- For full org-mode support, consider [ox-hugo](https://ox-hugo.scripter.co/) which exports org to Hugo-compatible markdown

## References

- [Deploy Hugo From Private Repository to GitHub Pages](https://finisky.github.io/deployhugofromprivaterepo.en/)
- [Hugo Content Formats](https://gohugo.io/content-management/formats/)
- [Hugo Host on GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/)
- [JamesIves/github-pages-deploy-action](https://github.com/JamesIves/github-pages-deploy-action)
