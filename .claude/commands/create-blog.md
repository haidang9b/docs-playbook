---
description: Scaffold and drive a technical blog post through research, drafting, and review.
argument-hint: <topic> — e.g. "cache-aside vs write-through in Redis 7 for intermediate backend devs"
---

You are orchestrating the technical blog creation workflow described in [.claude/workflows/create-blog-workflow.md](../workflows/create-blog-workflow.md), following the conventions in [CLAUDE.md](../../CLAUDE.md). Drive it phase by phase and **stop at every Human Checkpoint** to let the user confirm before continuing — never run all phases unattended.

**Topic:** $ARGUMENTS

If the topic above is empty, ask the user for the topic, target audience (Beginner / Intermediate / Advanced), and the one-line BLUF before doing anything else.

## Phase 0 — Scaffold
1. Derive a kebab-case `<topic-slug>` from the topic.
2. Copy `src/_template/` to `src/<topic-slug>/` (do not overwrite an existing folder — if it exists, read its `STATE.md` and resume from the **Next step** instead).
3. Fill in `src/<topic-slug>/STATE.md`: `topic`, `title`, today's date in `updated`, and the **Goal** (BLUF + audience). Keep `status: research`.
4. **Human Checkpoint ✅** — show the Goal and slug; wait for confirmation.

## Phase 1 — Research
Delegate to the **blog-researcher** agent: research the topic and write `src/<topic-slug>/references.md` with `[REF-N]` sources (confirmed + `TODO` placeholders), then advance `STATE.md` to `status: drafting`.
**Human Checkpoint ✅** — summarize the sources found and any `TODO` gaps; wait for confirmation.

## Phase 2 — Draft
Delegate to the **blog-writer** agent: write `src/<topic-slug>/article.md` from `references.md` with `[REF-N]` markers, SEO frontmatter, and house style, then advance `STATE.md` to `status: review`.
**Human Checkpoint ✅** — report that the draft is ready; wait for confirmation.

## Phase 3 — Review
Delegate to the **blog-reviewer** agent: review `article.md` for structure, style, grammar, and reference completeness; deliver Reviewer Notes; advance `STATE.md` to `status: publish` (clean) or `status: revise` (with blockers).
**Human Checkpoint ✅** — surface the Reviewer Notes and any unresolved `[REF-N] TODO` placeholders. Remind the user to run the SME review and the pre-publish checklist before marking `status: published`.

At each phase, keep `STATE.md` accurate (Progress checklist + Next step). Report which slug you are working in so the user can follow along.
