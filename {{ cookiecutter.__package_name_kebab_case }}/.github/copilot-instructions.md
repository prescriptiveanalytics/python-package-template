---
applyTo: '**/*'
---

## Instruction File Router - ALWAYS READ FIRST

### Convention Definitions
- *Do:* required/recommended actions
- *Don't:* prohibitions
- *Avoid:* things to steer clear of
- *Must:* mandatory requirements
- *Prefer:* preferences between alternatives
- *Tolerate:* acceptable exceptions

### Always Read (Universal)
- `coding_guidelines.instructions.md` - **MUST READ** for any Python code task

### Context-Based Reading
- `data_packages.instructions.md` - Read when:
  - Working with data package classes (`*DataPackage*`)
  - Heavy preprocessing, imputation, or schema validation code
  - Preprocessing or validation tasks
  - File paths containing `/data_package/` or `/data/`
  - Mentions of: "regenerate", "poe regenerate"
- `review.instructions.md` - Read when:
  - User asks to "review" code
  - User mentions "code review" or "audit"
  - User wants to check for issues or best practices compliance
  - Preparing for a release or major changes

### Available Commands (poe tasks)
**Code Quality & Formatting:**
- `poe precommit` - Format, sort imports, and lint code
- `poe check` - Verify code formatting and licensing
- `poe test` - Run pytest test suite

**Development:**
- `poe docs` - Serve documentation locally
- `poe check_licenses` - Validate dependency licenses

**Check the local `pyproject.toml` file for additional project-specific `poe` tasks**