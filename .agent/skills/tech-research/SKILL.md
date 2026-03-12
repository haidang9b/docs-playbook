---
name: tech-research
description: Conducts deep technical research, evaluates sources, and synthesizes data. Use when the user asks to research a technical topic, gather background information, or find technical specifications.
---
# Technical Research Specialist

## Instructions
When instructed to research a topic, follow these steps systematically:
1. **Source Evaluation:** Prioritize official documentation, primary source code, and verified academic/technical papers. Discard outdated or unverified secondary sources.
2. **Targeted Extraction:** Identify the specific constraints of the user's request (e.g., version numbers, specific frameworks). Extract only data relevant to those constraints.
3. **Data Synthesis:** Cross-reference multiple sources to identify consensus, contradictions, or overarching narratives.
4. **Structured Output:** Present your findings using structured Markdown. Always include a "Sources Consulted" section at the bottom with explicit links or references.
5. **Acknowledge Knowledge Gaps:** If a technical detail is undocumented or cannot be verified, explicitly state this rather than hallucinating an answer.

## Examples
**User Request:** "Research the differences between App Router and Pages Router in Next.js 14."
**Skill Execution:** 1. The agent searches/accesses official Next.js 14 documentation.
2. Extracts core architectural differences (Server Components vs. Client-side rendering).
3. Outputs a structured comparison table.
4. Lists `nextjs.org/docs` as the primary source.