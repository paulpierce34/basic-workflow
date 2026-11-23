# Syft + Grype Scanning with an Issue-Form GUI

Three-file setup that gives users a form-driven way to kick off a Syft + Grype
SBOM/vulnerability scan without ever touching the Actions tab.

## Files

| Path | Purpose |
|---|---|
| `.github/workflows/sbom-scan.yml` | The scan workflow: Syft SBOM → Grype scan → SARIF to Code Scanning |
| `.github/ISSUE_TEMPLATE/scan-request.yml` | The GUI: an issue form users fill out instead of navigating to Actions |
| `.github/workflows/issue-commands.yml` | Glue: watches issues created from the form, triggers the scan, posts a status comment back on the issue |

## How users trigger it

1. Open an issue on the repo → pick the **"SBOM Scan Request"** template.
2. Fill in target repo (`owner/repo` or full URL) + the critical-failure gate.
3. Submit. The `issue-commands` workflow parses the form, runs `gh workflow run`,
   reacts 🚀 to the issue, and comments a link to the run.
4. Results land in:
   - **Code Scanning** (SARIF, visible on the repo Security tab)
   - **Artifacts** (`sbom-scan-results`: CycloneDX, SPDX, syft-json, grype-json, SARIF)
   - The triggering issue gets a status comment

## Setup steps

1. Copy all three files into your repo's `.github/` directory, preserving paths.
2. (Optional) Add a fine-grained PAT named `GH_SCAN_TOKEN` under repo → Settings →
   Secrets → Actions, with `contents: read` on the target repo. Needed only if
   scanning private/cross-org repos; otherwise `GITHUB_TOKEN` suffices for
   same-org and public targets.
3. Ensure **Code Scanning** is enabled for the repo (Settings → Code security and
   analysis) so SARIF uploads are accepted.
4. Done — no other wiring needed.

## Notes

- The scan workflow normalizes arbitrary user input into `owner/repo` before
  using it, and validates the format, so untrusted text never reaches `git clone`
  or shell expansion directly.
- `fail_on_critical: true` exits non-zero if Grype reports any CRITICAL finding,
  surfacing a visible job failure.
- Grype pulls the latest vulnerability DB on each run (no pinning), so results
  reflect the freshest feed at scan time.
