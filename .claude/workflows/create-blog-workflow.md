---
description: Technical Blog Creation Workflow
---

# Technical Blog Creation Workflow

## Overview
This document outlines the end-to-end workflow for creating, drafting, and publishing a technical blog post. It hands each phase to a dedicated agent — `blog-researcher`, `blog-writer`, and `blog-reviewer` — which in turn apply the repository's Agent Skills (`tech-research`, `tech-blog-writing`, `tech-doc-review`, `reference-management`). Each topic lives in its own folder under `src/<topic-slug>/`, and a shared `STATE.md` tracks what is done and what comes next.

---

## Phase 0: Scaffold the Topic
Create a dedicated workspace so every artifact for the topic stays together and its progress is tracked.

* **1. Choose a slug:** A kebab-case topic slug (e.g. `caching-strategies-redis`).
* **2. Create the workspace:** Copy `src/_template/` to `src/<topic-slug>/`. This provides `STATE.md`, `article.md`, and `references.md` stubs.
* **3. Fill in the initial state:** In `src/<topic-slug>/STATE.md`, set the `topic`, `title`, `updated` date, and the **Goal** (BLUF + target audience). Leave `status: research`.
* **Human Checkpoint ✅:** Confirm the Goal captures the single most important takeaway and the correct audience before any research begins.

---

## Phase 1: Ideation & Research
Before writing a single word, gather verified technical data and record every source.

* **1. Define the Core Elements** (recorded in `STATE.md` → Goal):
  * **Target Audience:** Who is this for? (e.g., Junior frontend devs, DevOps engineers, CTOs).
  * **The "BLUF" (Bottom Line Up Front):** What is the single most important takeaway?
  * **Target Keywords:** 1-2 primary SEO keywords.
* **2. Execute Technical Research:**
  Hand the topic to the research agent, which gathers specs, examples, and documentation and writes them to `references.md`.
  > **🤖 AI Workflow (Claude Code):**
  > ```bash
  > @blog-researcher Research the topic in src/<topic-slug>/. Find the official documentation for [Technology] and the impact on [Specific Feature]; record sources in references.md using [REF-N] ids.
  > ```
* **3. Create the Outline:**
  Structure your headings (H2, H3) from the synthesis in `references.md`.
* **Human Checkpoint ✅:** Review `references.md` — verify confirmed sources are legitimate, note which `[REF-N] TODO` placeholders still need filling, and confirm `STATE.md` now shows `status: drafting`.

---

## Phase 2: Drafting
Transform the research and outline into a scannable, engaging, and accurate blog post.

* **1. Draft the Content:**
  Hand the draft to the writing agent. It cites claims with `[REF-N]` markers and never fabricates links.
  > **🤖 AI Workflow (Claude Code):**
  > ```bash
  > @blog-writer Using references.md in src/<topic-slug>/, write article.md targeting intermediate Python developers. Include a code example for [Feature].
  > ```
* **2. Add Visuals and Assets:**
  * Generate or capture architecture diagrams, UI screenshots, or flowcharts.
  * Save assets under `src/<topic-slug>/assets/images/` with descriptive alt-text (see the `tech-image-generation` skill for `IMAGE_PROMPTS.md`).
* **3. Format for Scannability:**
  Ensure paragraphs are short (max 3-4 sentences), key terms are **bolded**, code blocks carry language tags, and major `H2` sections are separated by `---`.
* **Human Checkpoint ✅:** Confirm `article.md` has valid frontmatter and that `STATE.md` shows `status: review`.

---

## Phase 3: Review & Refinement
Ensure the draft is technically accurate, grammatically correct, and aligned with the style guide.

* **1. Run the Automated Review:**
  > **🤖 AI Workflow (Claude Code):**
  > ```bash
  > @blog-reviewer Review src/<topic-slug>/article.md for grammar, clarity, style-guide adherence, and reference completeness. Provide Reviewer Notes.
  > ```
* **2. SME (Subject Matter Expert) Review:**
  Send the technical portions and code snippets to an engineer to verify accuracy. (Do not skip this step).
* **3. Final Human Polish:**
  Review the agent and SME feedback. Accept or reject changes and ensure the "voice" sounds natural and empathetic to the reader.
* **Human Checkpoint ✅:** `STATE.md` shows `status: publish` (clean) or `status: revise` with blockers listed. Resolve any remaining `[REF-N] TODO` placeholders before continuing.

---

## Phase 4: Publishing
Finalize the metadata and push the content live.

* **1. Pre-Publish Checks:**
  * [ ] Are the title tag and meta description filled out?
  * [ ] Do all internal and external links work?
  * [ ] Are all code snippets tested and executing properly?
  * [ ] Is there a clear Call to Action (CTA) at the bottom?
  * [ ] **Zero unresolved `[REF-N] TODO` placeholders** — every marker is confirmed and mirrored into `*Sources Consulted:*`.
  * [ ] `STATE.md` set to `status: published`.
* **2. Deployment:**
  * Commit the final `src/<topic-slug>/` folder to the `main` branch (if using Docs-as-Code).
  * Or, copy the rendered Markdown into your CMS (WordPress, Ghost, Webflow).

---

**Next Steps:** Once a post is live, monitor its performance (page views, time on page) and schedule a review date in 6 months to ensure the technical information hasn't deprecated.
