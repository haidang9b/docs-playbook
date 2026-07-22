---
name: tech-summarization
description: Distills dense technical content into concise summaries using the BLUF method. Use when asked to summarize whitepapers, release notes, or lengthy documentation.
---
# Technical Content Summarizer

## Instructions
When asked to summarize technical content, apply the following constraints:
1. **Apply BLUF (Bottom Line Up Front):** The very first sentence or paragraph must state the most critical takeaway, thesis, or system impact.
2. **Signal-to-Noise Reduction:** Strip away marketing fluff, redundant adjectives, and deep historical context unless explicitly requested. Focus on *what it is*, *what it does*, and *why it matters*.
3. **Precise Paraphrasing:** Translate complex jargon into plain language without losing technical accuracy. 
4. **Format Shifting:** If summarizing a process, convert paragraphs into numbered lists. If summarizing comparisons, convert text into a Markdown table.
5. **Preserve Constraints:** Ensure any hard requirements (e.g., "requires Python 3.9+", "deprecated in v2.0") are highly visible in the summary.

## Examples
**User Request:** "Summarize this 10-page release note document for our new API v3."
**Skill Execution:** The agent outputs a BLUF statement ("API v3 introduces GraphQL support and deprecates REST endpoints, requiring migration by Q4"). It then provides a bulleted list of breaking changes and a table of new features.