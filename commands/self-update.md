---
description: Update the specscore CLI to the latest release
---

Invoke the `specscore:self-update` skill. It runs `specscore self-update` to bring the installed CLI to the latest release — self-replacing for manual installs (with sha256 verification) and redirecting to the package manager for Homebrew/Scoop/WinGet installs. For a managed install, surface the printed manager command and let the user run it themselves.
