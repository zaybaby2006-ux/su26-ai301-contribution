# Contribution 1: Add Support for Initial Content

**Contribution Number:** 1  
**Student:** [Your Name]  
**Issue:** https://github.com/copier-org/copier/issues/2184  
**Status:** Phase II Complete

---

## Why I Chose This Issue

I chose issue #2184 "Add support for initial content" because it addresses a real workflow pain point that affects anyone using Copier templates in team environments. The problem is clear and well-defined: template files with placeholder content get overwritten during updates, even after developers have replaced that content with real implementations. This forces manual conflict resolution every time a team pulls in template updates, which is tedious and error-prone.

This issue is a good fit for several reasons. Copier is written in Python, which aligns with my learning goals. The maintainer (`sisp`) has actively engaged with the proposal, explained how the update algorithm works (a Git 3-way merge), and explicitly invited a design proposal — giving me a clear starting point. A contributor (`lkubb`) also pointed out that file-level exclusion already exists via the `exclude` config, so there's a precedent in the codebase for this kind of skip-on-update behavior that I can study and extend to work at the section level. The issue is labeled `help wanted`, is unassigned, and has no open PR attempting a solution. My contribution will involve first drafting a design proposal for how section-level markers (likely Jinja comments) could integrate with Copier's merge-based update algorithm, followed by the implementation itself. "Done" means users can wrap sections of template files in markers that tell Copier to only generate that content on initial project creation and preserve whatever the developer replaced it with during future updates.

---

## Understanding the Issue

### Problem Description

Copier is a tool that generates projects from templates. When a template is updated to a new version, developers run `copier update` to pull in the latest changes. The update algorithm uses a Git 3-way merge to intelligently combine template changes with the developer's modifications.

The problem: there is no way to mark specific sections of a template file as "initial content only." Template files often contain placeholder text like `TODO: Add setup instructions here` that developers are expected to replace immediately. When the template is updated, Copier's merge process tries to reconcile the template's placeholder with the developer's real content, causing merge conflicts or overwriting their work.

### Expected Behavior

Template authors should be able to mark sections with markers (e.g., Jinja comments like `{# copier:initial #}...{# /copier:initial #}`) that tell Copier: "Only include this content on first copy. During updates, preserve whatever the developer put here instead."

### Current Behavior

Currently, all template content is treated equally during updates. The only workaround is to exclude entire files from updates using the `exclude` config, but there is no way to exclude specific sections within a file. Developers must either manually resolve merge conflicts every update or avoid modifying placeholder sections entirely.

### Affected Components

Based on my reading of the codebase, the key files involved are:

- `copier/_main.py` — Contains the `Worker` class and `run_update()` method (line ~1309) which orchestrates the update process, including the Git 3-way merge
- `copier/_template.py` — Handles template parsing, cleanup, and Jinja rendering
- `copier/_cli.py` — CLI entry points for `copier copy` and `copier update`
- `copier/_tools.py` — Utility functions including file handling

---

## Reproduction Process

### Environment Setup

- **OS:** Windows 10/11
- **Python:** 3.11.9
- **Tools installed:** uv, copier (both dev build via `uv sync` and stable 9.15.2 via pip)
- **Challenges faced:**
  - Git credential mismatch between two GitHub accounts (Kingzayzoom vs zaybaby2006-ux) — resolved by updating the remote URL with the correct username
  - Windows `PermissionError` on temp file cleanup (`copier._vcs.clone.*`) — this is a known Windows issue where Git holds locks on temp directories during cleanup. The dev build of copier crashes during the `_cleanup()` method in `_template.py` line 246. This prevented `.copier-answers.yml` from being written, which in turn prevented `copier update` from finding the template source.
  - Copier's virtual environment uses Python 3.10.20 (via uv) while system Python is 3.11.9

### Steps to Reproduce

This is a **feature request**, not a bug, so reproduction means demonstrating the absence of the feature:

1. Create a template with placeholder content (e.g., `TODO: Add setup instructions here`)
2. Run `copier copy <template> <project>` to generate a project
3. Replace the placeholder text with real content (e.g., actual setup instructions)
4. Commit the changes
5. Update the template to a new version with additional sections
6. Run `copier update` on the project
7. **Expected:** Placeholder sections that the developer already replaced should be preserved
8. **Actual:** Copier's 3-way merge treats the placeholder content the same as any other template content, causing conflicts or overwriting the developer's real content. There is no mechanism to mark sections as "initial only."

### Reproduction Evidence

- **Working branch:** https://github.com/zaybaby2006-ux/copier/tree/fix-issue-2184
- **My findings:** The core issue is in the update algorithm in `_main.py`. The `run_update()` method performs a Git 3-way merge between the old template output, the new template output, and the developer's project. There is no concept of "initial content" sections — all content is merged uniformly. The `exclude` config in `copier.yml` only works at the file level (documented in the configuring docs), confirming there is no section-level exclusion mechanism.

---

## Solution Approach

### Analysis

The root cause is architectural: Copier's update algorithm treats all template content identically during the Git 3-way merge. There is no metadata or marker system to differentiate between content that should always be synced with the template and content that should only be generated once.

The existing `exclude` config proves the codebase already supports skipping content during updates — but only at the file level. The challenge is extending this concept to work at the section level within files, which requires intervening in the merge process.

### Proposed Solution

Implement a Jinja comment-based marker system that template authors can use to wrap "initial content only" sections. During `copier update`, a pre-processing step would detect these markers and modify the merge behavior for those sections.

### Implementation Plan

Using UMPIRE framework:

**Understand:** Template files need a way to mark sections as "generate once, then preserve developer changes." Currently all content is merged uniformly during updates, with no section-level control.

**Match:** The `exclude` config in `copier.yml` already implements file-level skip-on-update logic. The Jinja templating engine already supports custom comment syntax. The merge logic in `_main.py` `run_update()` handles the 3-way merge where this intervention would occur.

**Plan:**
1. Define a marker syntax using Jinja comments: `{# copier:initial #}` and `{# /copier:initial #}`
2. Add a pre-processing step in the template rendering pipeline (`_template.py`) that detects and tracks these markers
3. Modify the merge logic in `_main.py` `run_update()` to preserve the developer's version of marked sections instead of merging them
4. Add configuration options in `copier.yml` for enabling/disabling this feature
5. Write tests covering: first copy includes initial content, update preserves developer changes, update still applies changes outside marked sections
6. Update documentation

**Implement:** Work will be done on branch: https://github.com/zaybaby2006-ux/copier/tree/fix-issue-2184

**Review:** Will self-review against Copier's CONTRIBUTING.md guidelines, including Conventional Commits format for commit messages and ensuring all existing tests still pass.

**Evaluate:** 
- Run `uv run poe test` to verify no regressions
- Manual test: create template with initial markers → copy → modify marked sections → update template → verify marked sections are preserved while other changes come through
- Write new unit tests for the marker detection and merge modification logic

---

## Testing Strategy

### Unit Tests

- [ ] Test that `{# copier:initial #}` markers are correctly detected during template parsing
- [ ] Test that on first `copier copy`, initial content sections render normally
- [ ] Test that on `copier update`, content within initial markers is preserved from the developer's version
- [ ] Test that content outside initial markers still updates normally
- [ ] Test that nested or malformed markers produce clear error messages

### Integration Tests

- [ ] End-to-end test: copy → modify → update cycle with initial content markers
- [ ] Test interaction with existing `exclude` config

### Manual Testing

[To be completed in Phase III]

---

## Implementation Notes

### Week 1 Progress

- Cloned fork and set up development environment on Windows with `uv sync`
- Created working branch `fix-issue-2184` and pushed to fork
- Read through `_main.py`, `_template.py`, and `_cli.py` to understand the update flow
- Identified the Git 3-way merge in `run_update()` as the key intervention point
- Encountered Windows-specific `PermissionError` during temp file cleanup — this is a pre-existing issue in the dev build, not related to my feature
- Studied the existing `exclude` config as a pattern for section-level exclusion

---

## Pull Request

**PR Link:** [To be completed in Phase IV]

**PR Description:** [To be completed in Phase IV]

**Maintainer Feedback:**
- [To be completed in Phase IV]

**Status:** Not yet submitted

---

## Learnings & Reflections

[To be completed in Phase IV]

---

## Resources Used

- [Copier issue #2184 — Add support for initial content](https://github.com/copier-org/copier/issues/2184)
- [Copier documentation — exclude config](https://copier.readthedocs.io/en/stable/configuring/#exclude)
- [Copier CONTRIBUTING.md](https://github.com/copier-org/copier/blob/master/CONTRIBUTING.md)
- [Copier source — _main.py (update logic)](https://github.com/copier-org/copier/blob/master/copier/_main.py)
- [Copier source — _template.py (template handling)](https://github.com/copier-org/copier/blob/master/copier/_template.py)
