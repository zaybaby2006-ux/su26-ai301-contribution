# Contribution 1: Add Support for Initial Content

**Contribution Number:** 1  
**Student:** [Your Name]  
**Issue:** https://github.com/copier-org/copier/issues/2184  
**Status:** Phase I Complete

---

## Why I Chose This Issue

I'll use issue #2184 "Add support for initial content" for this write-up because it describes a workflow that all Copier template users will encounter in a team environment at some point․ The issue is straightforward: template files with initial content cannot be updated when developers make their implementation overwrite what was generated․ This forces manual conflict resolution every time a team pulls in template updates‚ making this a tedious and error-prone process․

This is a good problem‚ for several reasons: one‚ it gives me a chance to try Python‚ a language I'm interested in learning․ Another is that the maintainer (sisp) has actually engaged with the proposal‚ has explained the update algorithm (Git 3-way merge)‚ and has asked for a design proposal․ That's a good place to start․ (Another contributor‚ lkubb‚ patiently pointed out that file-level exclusion already works via the exclude config‚ so I can just look at that code and adapt it to work at the section level․) The issue is tagged help wanted‚ unassigned‚ and there is no open PR for it․ I would like to write a design doc describing how section-level markers (probably Jinja comments) would work with Copier's merge-based update algorithm‚ then implement it․ "Done" also allows wrapping sections of template files in markup‚ so Copier only renders that section when the project is first generated‚ not when the project is later updated with the content the developer replaced it with․

---

## Understanding the Issue

### Problem Description

The downside to this is that updating the project by running the copier update command will overwrite all files‚ including files where the Copier template authors replaced placeholder/dummy content with their real implementation․ At this time‚ there is no way to mark template file sections as "initial content only"․ This means the work the developers put in those sections is always overwritten when the templates are pulled‚ causing conflicts․

### Expected Behavior

[To be completed in Phase II]

### Current Behavior

[To be completed in Phase II]

### Affected Components

[To be completed in Phase II]

---

## Reproduction Process

### Environment Setup

[To be completed in Phase II]

### Steps to Reproduce

[To be completed in Phase II]

### Reproduction Evidence

[To be completed in Phase II]

---

## Solution Approach

### Analysis

[To be completed in Phase II]

### Proposed Solution

[To be completed in Phase II]

### Implementation Plan

[To be completed in Phase II]

---

## Testing Strategy

[To be completed in Phase III]

---

## Implementation Notes

[To be completed in Phase III]

---

## Pull Request

[To be completed in Phase IV]

---

## Learnings & Reflections

[To be completed in Phase IV]

---

## Resources Used

- [Copier issue #2184](https://github.com/copier-org/copier/issues/2184)
- [Copier documentation — exclude config](https://copier.readthedocs.io/en/stable/configuring/#exclude)
- [Copier CONTRIBUTING.md](https://github.com/copier-org/copier/blob/master/CONTRIBUTING.md)
