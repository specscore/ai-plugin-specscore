---
captured_by: user
status: queued
---
# Update plan and change-status skills to use specscore plan change-status once the CLI ships it, instead of hand-editing plan Status

Companion to specscore-cli#69. There is no 'specscore plan change-status' verb today, so the plan flow hand-edits the Plan artifact Status (Draft/Under Review/Approved), risking the frontmatter/body mirror drift of #68. Once the CLI adds the verb, update skills/plan/SKILL.md and skills/change-status/SKILL.md (and specstudio-skills/skills/plan) to call it. Until then, if hand-editing is unavoidable, edit BOTH the frontmatter 'status:' and body '**Status:**' together to avoid drift. Also reflect the broader invariant: all artefact status transitions go through the dedicated change-status verb where one exists.
