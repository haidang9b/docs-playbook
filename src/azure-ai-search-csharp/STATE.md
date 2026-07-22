---
topic: azure-ai-search-csharp
title: "Getting Started with Azure AI Search in C#"
status: publish
updated: 2026-07-22
---

## Goal
Introduce Azure AI Search to intermediate .NET developers and show them how to create, populate, and query an index from C# using the Azure.Search.Documents SDK. BLUF: You can stand up a production-grade search service and query it from C# in a few dozen lines of code.

## Progress
- [x] Research — references.md created (28 confirmed, 0 TODO placeholders)
- [x] Draft — article.md v1 written
- [x] Review — Reviewer Notes applied, references resolved (2026-07-22)
- [ ] Publish

## Next step
**Publish.** Review passed clean: 0 TODO placeholders, all inline `[REF-N]` markers map to `references.md`, Sources Consulted mirrors cited refs. Fill in `author:` in frontmatter at publish time.

## Notes / open questions
- CONFIRMED audience: intermediate .NET developers.
- CONFIRMED code scope: core CRUD + keyword search (create index, upload docs, keyword/full-text queries with filters). Vector/semantic search out of scope for the sample; may mention as "next steps."
- CONFIRMED SDK: `Azure.Search.Documents` stable **v12.0.0** (2026-05-01); prerelease 12.1.0-beta.1. Namespace is `Azure.Search.Documents`; the older `Microsoft.Azure.Search` (v10) is **retired** — do not mix samples.
- BRANDING: use "Azure AI Search (formerly Azure Cognitive Search)"; renamed 2023-11-15, no API/pricing breaking changes.
- RESOLVED [REF-28] (2026-07-22): pagination confirmed against dedicated official API reference pages — [REF-28] SearchOptions.Skip (max 100,000), [REF-30] SearchOptions.IncludeTotalCount (default false, approximate, perf impact), [REF-31] SearchResults<T>.TotalCount (null unless requested). Blockquote caveat removed; article now cites [REF-28][REF-30][REF-31]. **0 TODO placeholders remaining.**
- RECOMMENDATION: present `DefaultAzureCredential` (Entra ID / keyless) as the default auth path, API key as the quick-start alternative. (Applied in draft.)

## Draft notes (blog-writer, 2026-07-22)
- article.md v1 written per the recommended H2/H3 outline; focus keyphrase "Azure AI Search in C#" in H1, description, and first 100 words.
- Cited [REF-1..6, 8..13, 15..17, 19, 20b, 21..25, 27, 29] inline; Sources Consulted mirrors these confirmed refs.
- Refs NOT cited (available if reviewer wants them): [REF-7] dotnet SDK how-to, [REF-14] query types overview, [REF-18] simple syntax examples, [REF-26] text query filters. Not errors — just unused.
- [REF-28] pagination gap resolved 2026-07-22 (see Notes above); no TODO placeholders remain.

## Reviewer notes (blog-reviewer, 2026-07-22) — status: publish
Edits applied:
- Hook: "rolling that yourself" → "rolling your own" (natural phrasing).
- Query snippets: changed two `var response = searchClient.Search<Hotel>(...)` to explicit `SearchResults<Hotel> response = ...`. `Search<T>` returns `Response<SearchResults<T>>`; `.GetResults()` / `.TotalCount` are members of `SearchResults<T>`, not `Response<T>`. Explicit typing triggers the `Response<T>`→`T` implicit conversion and matches the already-correct first query example. Fixes a likely compile error.

Checks passed:
- References: all inline [REF-N] (1,2,3,4,5,6,8,9,10,11,12,13,15,16,17,19,20b,21,22,23,24,25,27,28,29,30,31) map to references.md; Sources Consulted mirrors exactly; **0 TODO placeholders**. Refs 7/14/18/26 defined but uncited (intentional, not errors).
- Frontmatter valid: title mirrors H1, description ~148 chars w/ focus keyphrase, keywords[0]==focus_keyphrase, date/slug correct, author blank.
- House style: hook + focus keyphrase in first 100 words; short paras; H2/H3; bold terms; language-tagged code; H2s separated by `---`; Next Steps CTA + Sources Consulted present. Anchor links (#next-steps, #pricing--tiers) resolve.
- C# snippets internally consistent and plausible for v12 (SearchIndexClient/SearchClient, FieldBuilder, IndexDocumentsBatch, Search<T>, SearchOptions, SearchFilter.Create).

No blockers. SME/author question (non-blocking): confirm sample hotel name "Stay-Kay City" is intentional; fill author at publish.
