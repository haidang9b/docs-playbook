---
name: reference-management
description: Defines the shared citation and placeholder-reference convention for blog topics. Use when researching sources, inserting citations into a draft, or checking that every reference is resolved before publishing.
---
# Reference Management

## Instructions
All research, drafting, and review work shares one citation system so sources never get lost or fabricated. Follow it exactly:

1. **One references file per topic:** Every topic keeps its sources in `src/<topic-slug>/references.md`, alongside `article.md`. Never scatter citations elsewhere.
2. **Stable reference ids:** Assign each source a stable id `[REF-1]`, `[REF-2]`, … in the order it is first needed. Ids never change once assigned, even if sources are reordered — the writer relies on them as anchors.
3. **Two reference states:**
   - **Confirmed** — a verified source. Format: `[REF-1] Source Title — https://full-url`.
   - **Placeholder** — a source that is still needed or unverified. Format: `[REF-2] TODO: verify — <what is needed, e.g. "official docs for feature X">`.
4. **Never fabricate URLs:** If a claim needs a citation you cannot verify, leave it as a `TODO` placeholder. A placeholder is a valid, honest state; an invented link is not.
5. **Inline markers mirror the file:** In prose, cite as `[REF-N]` immediately after the claim. Every `[REF-N]` used in `article.md` must have a matching entry in `references.md`, and vice versa.
6. **Publish gate:** Before a topic is marked *published*, every `[REF-N]` must be in the *confirmed* state — zero `TODO` placeholders remain. Unresolved placeholders block publishing and are logged in `STATE.md` under **Notes / open questions**.
7. **Sources Consulted:** At publish time, mirror the confirmed references into a `*Sources Consulted:*` block at the bottom of `article.md`, favoring official/primary documentation.

## Examples
**User Request:** "I found the official Git submodules docs while researching — record it and cite it in the intro."
**Skill Execution:** The agent adds `[REF-1] Git Submodules Documentation — https://git-scm.com/book/en/v2/Git-Tools-Submodules` to `references.md` as a confirmed source, then places `[REF-1]` after the relevant sentence in `article.md`.

**User Request:** "The draft claims submodules improve CI speed but I have no source yet."
**Skill Execution:** The agent inserts `[REF-2]` in the prose and adds `[REF-2] TODO: verify — need a benchmark or primary source for CI speed claim` to `references.md`, then notes the open item in `STATE.md`. It does not invent a link.
