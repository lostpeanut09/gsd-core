---
type: Fixed
pr: 4739
---
OpenCode installs no longer report all 72 installed skills as custom files. The file manifest skipped the skills surface for the opencode runtime only, so every update backed the staged skills up and prompted a restore. The manifest now owns what the installer stages under skills/.
