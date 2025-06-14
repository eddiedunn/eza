# Remove Unused GitHub Actions Workflows

To keep your repo clean and focused on Darwin (macOS) only, you should delete the following workflow YAML files from `.github/workflows/`:

- `apt.yml`  
- `unit-tests.yml`  
- `winget.yml`

These files are for Linux package builds, upstream unit tests, and Windows/Winget automation, none of which are needed for a Darwin-only fork.

## How to Remove

You can safely delete these files:

```
rm .github/workflows/apt.yml
rm .github/workflows/unit-tests.yml
rm .github/workflows/winget.yml
```

Or delete them from the GitHub web UI under your repo's `.github/workflows/` directory.

---

**After cleanup, only keep:**
- `build-darwin.yml` (Darwin binary build)
- `auto-release.yml` (auto release on tag)
- `upstream-sync.yml` (sync upstream tags)

This ensures your Actions tab and repo are clean and only run workflows relevant to macOS releases.
