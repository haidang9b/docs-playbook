---
name: tech-blog-writing
description: Drafts engaging, SEO-optimized technical blog posts. Use when the user requests a blog post, article, or tutorial intended for developer or engineering audiences.
---
# Technical Blog Writer

## Instructions
When drafting a technical blog post, adhere to the following workflow:
1. **Audience Empathy:** Establish the target audience (Beginner, Intermediate, Advanced) based on the prompt. Match the technical depth and jargon to this audience.
2. **The Hook:** Write an introduction that immediately establishes the pain point and the value the reader will get from the post. 
3. **Scannable Formatting:** - Keep paragraphs under 4 sentences.
   - Liberally use `H2` and `H3` headers to break up text.
   - Use bold text for key terms and bulleted lists for requirements.
4. **Code Integration:** If the post involves code, provide clean, well-commented code blocks with the correct language tags for syntax highlighting.
5. **SEO Optimization:** Naturally weave the topic's primary keywords into the `H1`, at least one `H2`, and the first 100 words.
6. **Call to Action (CTA):** Conclude every post with a clear next step (e.g., "Check out the official docs," "Clone the repository").
7. **Citations:** Follow the `reference-management` skill — cite every factual claim with an inline `[REF-N]` marker matching an entry in the topic's `references.md`. Never fabricate a URL; if a source is missing, cite the `[REF-N] TODO` placeholder as-is.
8. **Output location & frontmatter:** Write the article to `src/<topic-slug>/article.md`. Include YAML frontmatter with `title`, `description`, `focus_keyphrase`, `keywords` (array whose first element equals `focus_keyphrase`), `author: ""`, `date` (`YYYY-MM-DD`), and `slug`. Separate major `H2` sections with `---`, and end with a `*Sources Consulted:*` block mirroring the confirmed references.

## Examples
**User Request:** "Write a blog post about how to use the fetch API in JavaScript."
**Skill Execution:** The agent generates an article with a catchy title, a clear introductory hook explaining *why* `fetch` is better than `XMLHttpRequest`, scannable H2 sections, commented code blocks showing a basic GET request, and a CTA to try it in the console.