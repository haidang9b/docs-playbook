# CLAUDE.md — Technical Blog Playbook

Project instructions for writing technical blog posts with AI agents. This is a docs-as-code repository; there is no build or test system — everything is Markdown.

## Purpose
Produce accurate, scannable, SEO-optimized technical blog articles through a research → draft → review pipeline, with every source tracked and every step recorded.

## Architecture: agents → skills → workflows
- **Agents** (`.claude/agents/*.md`) — the three personas you hand a job to. Each owns one phase and calls skills:
  - `blog-researcher` — internet research + `references.md` (verified sources and `[REF-N]` placeholders).
  - `blog-writer` — drafts `article.md` with citations and SEO frontmatter.
  - `blog-reviewer` — edits for style, grammar, and reference completeness.
- **Skills** (`.claude/skills/*/SKILL.md`) — reusable capabilities the agents apply: `tech-research`, `tech-blog-writing`, `tech-doc-review`, `tech-summarization`, `tech-image-generation`, and `reference-management`. (`dotnet-*` skills serve a separate .NET workflow.)
- **Workflows** (`.claude/workflows/*.md`) — human-in-the-loop SOPs that chain the agents with `Human Checkpoint ✅` gates. Start from `create-blog-workflow.md`.

## Topic workspace
Each article is a self-contained folder under `src/`:

```
src/<topic-slug>/
├── STATE.md                  # progress tracker (the shared "memory")
├── article.md                # the article (SEO frontmatter + house style)
├── references.md             # confirmed sources + [REF-N] placeholders
└── assets/images/            # image prompts / diagrams
```

Copy `src/_template/` to start a new topic. Legacy articles remain in `drafts/` and are not the target for new work.

## The STATE.md contract
`STATE.md` is the per-topic source of truth for progress. **Every agent reads it first and updates it last** — mark its step done, set `status` and the single **Next step**, bump `updated`, and log open questions. `status` moves through: `research → drafting → review → revise → published`.

## Article frontmatter schema
```yaml
title: "…"                    # mirror the H1
description: "…"              # meta description, includes the focus keyphrase
focus_keyphrase: "…"
keywords: ["<focus_keyphrase>", "…"]   # first element == focus_keyphrase
author: ""                    # filled at publish time
date: "YYYY-MM-DD"
slug: "<topic-slug>"
```

## House style
- Open with a **hook** stating the reader's pain point; weave the focus keyphrase into the first 100 words.
- Short paragraphs (max 3-4 sentences); liberal `H2`/`H3`; **bold** key terms; lists and tables for scannability.
- Language-tagged code blocks. Separate major `H2` sections with `---`.
- End the body with a **Next Steps** CTA, then a `*Sources Consulted:*` block.

## References convention
Follow the `reference-management` skill. Cite claims inline with `[REF-N]`; each maps to an entry in `references.md`. Sources are either **confirmed** (`[REF-N] Title — https://url`) or **placeholder** (`[REF-N] TODO: verify — <need>`). **Never fabricate a URL.** No topic is `published` while any `[REF-N] TODO` remains.
