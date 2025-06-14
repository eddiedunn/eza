# Upstream Sync & Automated Darwin (macOS) Binary Releases

> **This fork and its automation are 100% focused on Darwin (macOS) builds.**
> - No Windows or Linux binaries are built or supported.
> - All workflows and documentation here are for macOS users and maintainers only.
> - You may safely ignore or remove any non-Darwin workflows (such as Windows/Winget or Linux jobs).

This guide explains how to set up your fork to automatically sync releases from upstream (`eza-community/eza`), build Darwin (macOS) binaries, and attach them to releases using GitHub Actions.

---

## 1. Forking and Initial Setup

1. **Fork the upstream repository**
   - Go to https://github.com/eza-community/eza
   - Click **Fork** (top-right) and create your own copy.

2. **Clone your fork**
   ```sh
   git clone https://github.com/<your-username>/eza.git
   cd eza
   ```

3. **Add the upstream remote**
   ```sh
   git remote add upstream https://github.com/eza-community/eza.git
   ```

---

## 2. Syncing Tags from Upstream

To make sure your fork has all upstream tags (required for matching releases):

```sh
git fetch upstream --tags
git push origin --tags
```

---

## 3. Setting Up GitHub Actions for Automated Builds

### 3.1. Create a Personal Access Token (PAT)
- Go to [GitHub Tokens](https://github.com/settings/tokens)
- Create a classic token with `repo` scope **OR** a fine-grained token with `Contents: Read and write` for your fork
- Copy the token

### 3.2. Add the PAT as a Secret
- Go to your fork on GitHub
- Click **Settings** → **Secrets and variables** → **Actions**
- Click **New repository secret**
- Name: `PAT_PUSH`
- Value: (paste your PAT)

---

## 4. Required Workflow YAML Files

Place these files in `.github/workflows/` in your fork.

### 4.1. `.github/workflows/upstream-sync.yml`

```yaml
name: Sync tags from upstream
on:
  schedule:
    - cron: '0 * * * *'  # every hour
  workflow_dispatch:
jobs:
  sync-upstream-tags:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout your fork
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Add upstream remote
        run: git remote add upstream https://github.com/eza-community/eza.git
      - name: Fetch upstream tags
        run: git fetch upstream --tags
      - name: Push new tags to origin
        env:
          GIT_ASKPASS: /dev/null
          GITHUB_TOKEN: ${{ secrets.PAT_PUSH }}
        run: |
          set -e
          git tag -l | xargs -n 1 -I {} git push https://x-access-token:${{ secrets.PAT_PUSH }}@github.com/${{ github.repository }}.git refs/tags/{}
```

### 4.2. `.github/workflows/auto-release.yml`

```yaml
name: Auto Create Release from Tag
on:
  push:
    tags:
      - '*'
jobs:
  create-release:
    runs-on: ubuntu-latest
    steps:
      - name: Create GitHub Release for new tag
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ github.ref_name }}
          name: ${{ github.ref_name }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 4.3. `.github/workflows/build-darwin.yml`

```yaml
name: Build Darwin Binary
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  release:
    types: [published]
  workflow_dispatch:
jobs:
  build-darwin:
    runs-on: macos-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      - name: Install Rust toolchain
        uses: dtolnay/rust-toolchain@stable
      - name: Build release binary
        run: cargo build --release
      - name: Upload Darwin binary artifact
        uses: actions/upload-artifact@v4
        with:
          name: eza-darwin-binary
          path: target/release/eza
          if-no-files-found: error
      - name: Upload Darwin binary to release
        if: github.event_name == 'release'
        uses: softprops/action-gh-release@v2
        with:
          files: target/release/eza
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 5. How the Automation Works

1. **Tag Sync**: Every hour, `upstream-sync.yml` fetches tags from upstream and pushes them to your fork.
2. **Auto Release**: When a new tag appears, `auto-release.yml` creates a GitHub Release for it.
3. **Darwin Build**: On every release, `build-darwin.yml` builds and attaches the Darwin binary.

---

## 6. Building Binaries for Older Releases

1. Ensure the tag exists in your fork (see section 2).
2. Go to your fork’s **Releases** tab on GitHub.
3. Click **Draft a new release**.
4. Select the tag you want.
5. Publish the release.
6. The workflow will build and attach the Darwin binary.

---

## 7. Troubleshooting

- **No release created?**
  - Check that `auto-release.yml` is present and enabled.
  - Check the Actions tab for failed workflow runs.
- **No Darwin binary attached?**
  - Check that `build-darwin.yml` is present and completed successfully.
- **PAT issues?**
  - Make sure the secret is named exactly `PAT_PUSH` and has correct permissions.

---

## 8. FAQ

**Q: How often does the sync run?**
A: Every hour by default, or manually via the Actions tab.

**Q: Can I sync release notes from upstream?**
A: This setup does not copy release notes. If you want this, additional scripting is required.

**Q: What if I want to build binaries for all historical tags?**
A: Manually draft releases for each tag, or script the process.

---

For questions or advanced automation, open an issue or ask for help!
