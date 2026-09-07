---
name: learning-workflow
description: >-
  Rules and procedures for the user's AI engineering curriculum. Use whenever taking study notes,
  updating learning/README.md progress, or determining what to learn next.
---

# Learning Workflow & Note-Taking System

## 1. Directory Structure
- `Projects/learning/README.md`: Master curriculum roadmap and source of truth for progress.
- `Projects/learning/notes/`: Storage for all study notes (e.g., `notes/intro-to-llms.md`).
- `Projects/<project-name>/`: Standalone implementation codebases (e.g., `mini-vllm/`, `micrograd/`).

## 2. Note-Taking Rules
- **Audience**: The user alone. Write strictly for personal recall.
- **Format**: Flat bullet points only (10–15 bullets max).
- **No Titles / Section Headers**: Avoid markdown H1/H2/H3 headers within the notes body.
- **Zero Filler Words**: Be dense, punchy, and direct. Omit fluffy phrases, elaborate introductions, or redundant prose.
- **Preserve Concepts**: Keep personal mental models and references to past projects (`micrograd`, `makemore`).
- **Source Link**: Always include the source URL at the top of the notes.
- **File Location**: Save under `Projects/learning/notes/<topic-name>.md`.

## 3. Progress Tracking & Check-in
- When a video, paper, or milestone is finished:
  1. Update `Projects/learning/README.md`: change `[ ]` to `[x]` and hyperlink to the note file (`notes/<name>.md`) or code repository.
  2. Commit and push the changes to GitHub (`git commit` and `git push origin main`).

## 4. Answering "What to learn next?"
When the user asks what to work on or learn next:
1. Open and inspect `Projects/learning/README.md`.
2. Locate the earliest uncompleted item (first item marked `- [ ]`, whether a pre-text video, paper, or milestone).
3. Provide that exact single next action immediately:
   - If a pre-text video/reading: provide the title, duration, and link.
   - If a hands-on milestone: cite the milestone number, what needs to be implemented, and which project directory under `Projects/` to work in.
