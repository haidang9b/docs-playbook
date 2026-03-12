---
description: Technical Blog Creation Workflow
---

# Technical Blog Creation Workflow

## Overview
This document outlines the end-to-end workflow for creating, drafting, and publishing a technical blog post. It integrates human expertise with our repository's custom AI Agent Skills (`tech-research`, `tech-blog-writing`, and `tech-doc-review`) to accelerate the writing process and maintain high-quality standards.

---

## Phase 1: Ideation & Research
Before writing a single word, define the purpose of the post and gather verified technical data.

* **1. Define the Core Elements:**
  * **Target Audience:** Who is this for? (e.g., Junior frontend devs, DevOps engineers, CTOs).
  * **The "BLUF" (Bottom Line Up Front):** What is the single most important takeaway?
  * **Target Keywords:** 1-2 primary SEO keywords.
* **2. Execute Technical Research:**
  Gather specifications, codebase examples, and documentation.
  > **🤖 AI Workflow (Claude Code):**
  > Run the research agent to gather factual data.
  > ```bash
  > /tech-research Find the official documentation for the latest breaking changes in [Technology] and summarize the impact on [Specific Feature].
  > ```
* **3. Create the Outline:**
  Structure your headings (H2, H3) based on the research provided by the agent.

---

## Phase 2: Drafting
Transform the research and outline into a scannable, engaging, and accurate blog post.

* **1. Draft the Content:**
  Focus on getting ideas down. Prioritize code accuracy, a strong narrative hook, and clear transitions between technical concepts.
  > **🤖 AI Workflow (Claude Code):**
  > You can generate the first draft or specific sections using the writing agent.
  > ```bash
  > /tech-blog-writing Using the outline in `drafts/outline.md`, write a technical blog post targeting intermediate Python developers. Include a code example for [Feature].
  > ```
* **2. Add Visuals and Assets:**
  * Generate or capture architecture diagrams, UI screenshots, or flowcharts.
  * Save all image assets in the `/assets/images/` directory with descriptive alt-text.
* **3. Format for Scannability:**
  Ensure paragraphs are short (max 3-4 sentences), key terms are **bolded**, and code blocks contain correct language syntax tags (e.g., ````javascript ````).

---

## Phase 3: Review & Refinement
Ensure the draft is technically accurate, grammatically correct, and aligns with the company style guide.

* **1. Run the Automated Review:**
  Use the AI to catch passive voice, formatting errors, and style guide violations before passing it to a human.
  > **🤖 AI Workflow (Claude Code):**
  > ```bash
  > /tech-doc-review Review `drafts/my-new-post.md` for grammar, clarity, and adherence to our corporate style guide. Provide a list of recommended edits.
  > ```
* **2. SME (Subject Matter Expert) Review:**
  Send the technical portions and code snippets to an engineer to verify accuracy. (Do not skip this step).
* **3. Final Human Polish:**
  Review the AI and SME feedback. Accept or reject changes and ensure the "voice" sounds natural and empathetic to the reader.

---

## Phase 4: Publishing
Finalize the metadata and push the content live.

* **1. Pre-Publish Checks:**
  * [ ] Are the title tag and meta description filled out?
  * [ ] Do all internal and external links work?
  * [ ] Are all code snippets tested and executing properly?
  * [ ] Is there a clear Call to Action (CTA) at the bottom?
* **2. Deployment:**
  * Commit the final `.md` file to the `main` branch (if using Docs-as-Code).
  * Or, copy the rendered Markdown into your Content Management System (CMS) like WordPress, Ghost, or Webflow.

---

**Next Steps:** Once a post is live, monitor its performance (page views, time on page) and schedule a review date in 6 months to ensure the technical information hasn't deprecated.