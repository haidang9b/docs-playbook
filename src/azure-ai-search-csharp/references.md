# References — azure-ai-search-csharp

## Synthesis

**What the article can assert (all confirmed against official Microsoft sources):**

- **Product & branding.** Azure AI Search is Microsoft's managed, AI-powered information-retrieval (search) platform. It was **formerly named "Azure Cognitive Search"** and was renamed to **Azure AI Search on November 15, 2023**; the rename brought **no breaking API/SDK changes and no pricing changes** [REF-1][REF-5][REF-20]. The writer should introduce it as "Azure AI Search (formerly Azure Cognitive Search)" once, then use the current name throughout. This is the most important caveat for an intermediate-dev audience who will find older tutorials under the old name and the retired `Microsoft.Azure.Search` (v10) package.

- **Core concepts** to define for the audience: a **search service** hosts top-level objects (indexes, indexers, data sources, skillsets, synonym maps); an **index** is persistent storage of searchable JSON **documents**; each **field** has a name, data type, and **attributes** (`key`, `searchable`, `filterable`, `sortable`, `facetable`, `retrievable`) that control behavior; **analyzers** tokenize text for full-text search. A rough (imperfect) analogy: index ≈ table, document ≈ row, field ≈ column [REF-1][REF-2][REF-3][REF-5].

- **SDK & version.** The current stable NuGet package is **`Azure.Search.Documents` v12.0.0** (released 2026-05-01; latest prerelease 12.1.0-beta.1) — note the NuGet package major is 12 while docs/APIs still speak of the "v11" data-plane library lineage that superseded the retired `Microsoft.Azure.Search` v10 [REF-6][REF-5]. Three main clients: **`SearchClient`** (query + upload/merge/delete documents), **`SearchIndexClient`** (create/manage indexes), **`SearchIndexerClient`** (indexers/skillsets — out of scope for this article's sample) [REF-5][REF-8].

- **Authentication.** Two approaches: **API key** via `AzureKeyCredential` (admin key = read-write, query key = read-only; always prefer a query key in client apps), or **Microsoft Entra ID (RBAC)** via `DefaultAzureCredential` from the `Azure.Identity` package (recommended; requires role assignments such as *Search Index Data Reader* for queries, *Search Index Data Contributor* for writes) [REF-5][REF-9][REF-10][REF-11]. Recommend Entra ID / keyless as the modern default, showing API key as the quick-start path.

- **Create an index (C#).** Two patterns, both confirmed in the SDK README: (a) attribute-driven — decorate a model class with `[SimpleField]`, `[SearchableField]`, `[SimpleField(IsKey = true)]` etc. and build fields with `new FieldBuilder().Build(typeof(Hotel))`; (b) explicit — construct `SimpleField`, `SearchableField`, `ComplexField` objects directly. Then `SearchIndexClient.CreateIndex(new SearchIndex("name") { Fields = ... })`. Key rules: exactly one **key field** (`IsKey = true`, string); `SimpleField` = non-full-text (filter/sort/facet only); `SearchableField` = full-text searchable and can also set `AnalyzerName` [REF-5][REF-3].

- **Upload/index documents (C#).** Batch actions via `IndexDocumentsBatch.Create(IndexDocumentsAction.Upload(...), IndexDocumentsAction.Merge(...), ...)` then `SearchClient.IndexDocuments(batch)`. Actions: `Upload`, `Merge`, `MergeOrUpload`, `Delete`. `MergeOrUpload` inserts or partially updates. Use `IndexDocumentsOptions { ThrowOnAnyError = true }`; otherwise inspect `IndexDocumentsResult` per-action (a batch can partially succeed). Merge has special rules (e.g., cannot merge into a nonexistent document) [REF-5][REF-4][REF-12-merge].

- **Query (C#).** `SearchClient.Search<T>("searchText", SearchOptions)` returns `SearchResults<T>`; iterate `response.GetResults()` (async: `SearchAsync` + `GetResultsAsync`). Use a typed model or `SearchDocument` dictionary. `SearchOptions` controls behavior: **`Filter`** (OData `$filter`, e.g. `Rating ge 4`; build safely with `SearchFilter.Create($"...")`), **`OrderBy`** (e.g. `"Rating desc"`), **`Size`**/**`Skip`** (pagination), `Select`, `SearchFields`, `SearchMode`, `IncludeTotalCount`, `Facets` [REF-5][REF-13][REF-14]. Full-text syntax: **simple** query parser (default; AND/OR/NOT, phrase, prefix `*`) vs **full Lucene** (`queryType=full`: fuzzy, proximity, regex, wildcards); scoring uses Apache Lucene + **BM25** [REF-15][REF-16][REF-17][REF-18]. `searchMode` (`any` default vs `all`) controls Boolean precision/recall. OData string literals escape single quotes by doubling them (`'Alice''s'`) [REF-19][REF-20b].

- **Pricing / free tier (brief mention).** Tiers: **Free** (1 per subscription, shared infra, ~50 MB storage, no scale-up, for dev/tutorials), **Basic**, **Standard (S1–S3, S3 HD)**, **Storage Optimized (L1–L2)**; capacity measured in **Search Units (SUs)**. You cannot upgrade a Free service in place. There is a consumption-based **Serverless (preview)** model. Point readers to the pricing page for current numbers [REF-21][REF-22][REF-23].

**Contradictions / caveats to flag for the writer:**
- Package **major version 12** vs docs' "v11 data-plane library" language — clarify that `Azure.Search.Documents` (namespace `Azure.Search.Documents`) is the current library and `Microsoft.Azure.Search` (v10) is **retired**; don't mix samples.
- Vector/semantic search is **out of scope for sample code** — mention only as a "Next Steps" pointer (the SDK supports it via vector queries / `SemanticSearch`) [REF-5].
- Free-tier storage limit (~50 MB) and "may be deleted after inactivity" caveats should be stated so readers don't build production demos on Free.

**Recommended H2/H3 outline:**
1. **H2: What is Azure AI Search?** (hook: building search is hard; managed service solves it) — includes the "formerly Azure Cognitive Search" note. [REF-1][REF-23]
   - H3: Where it fits (keyword/full-text now; vector/semantic later). [REF-1][REF-5]
2. **H2: Core concepts** — service, index, documents, fields & attributes, analyzers (with the index≈table analogy). [REF-1][REF-2][REF-5]
3. **H2: Setting up** — create a service (portal/CLI free tier), install `Azure.Search.Documents` + `Azure.Identity`, note version. [REF-6][REF-7][REF-24][REF-21]
4. **H2: Connecting from C#** — `SearchIndexClient` vs `SearchClient`; API key (`AzureKeyCredential`) vs `DefaultAzureCredential` (recommended). [REF-5][REF-9][REF-10][REF-11]
5. **H2: Creating an index in C#** — model class + attributes + `FieldBuilder`; explain `SimpleField`/`SearchableField`, key/filterable/sortable/facetable. [REF-5][REF-3]
6. **H2: Uploading documents** — `IndexDocumentsBatch`, `Upload` vs `MergeOrUpload`, batching & partial-failure handling. [REF-5][REF-4]
7. **H2: Running keyword queries** — `Search<T>`, iterate results, `SearchOptions` (Filter/OrderBy/Select/Size/Skip), OData filters, simple vs Lucene syntax, `searchMode`. [REF-5][REF-13][REF-15][REF-17][REF-19]
8. **H2: Pricing & tiers (brief)** — Free vs paid, SUs. [REF-21][REF-22]
9. **H2: Next Steps** — vector & semantic ranking pointer; full samples repo. [REF-5][REF-25]

Counts: **25 confirmed, 1 TODO placeholder.**

## References
<!-- Confirmed:   [REF-N] Title — https://url -->
<!-- Placeholder: [REF-N] TODO: verify — what is still needed -->

### Overview & concepts
- [REF-1] Introduction to Azure AI Search — https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search
- [REF-2] Search index overview (indexes, documents, fields, attributes) — https://learn.microsoft.com/en-us/azure/search/search-what-is-an-index
- [REF-3] Create a search index — https://learn.microsoft.com/en-us/azure/search/search-how-to-create-search-index
- [REF-4] Load an index (add/update documents) — https://learn.microsoft.com/en-us/azure/search/search-how-to-load-search-index

### .NET SDK
- [REF-5] Azure AI Search client library for .NET (README: clients, auth, FieldBuilder/SimpleField/SearchableField, IndexDocumentsBatch/MergeOrUpload, Search<T>, SearchOptions; documents v12.0.0; notes "formerly Azure Cognitive Search" and retired Microsoft.Azure.Search v10) — https://learn.microsoft.com/en-us/dotnet/api/overview/azure/search.documents-readme?view=azure-dotnet
- [REF-6] NuGet — Azure.Search.Documents (stable v12.0.0, 2026-05-01; latest prerelease 12.1.0-beta.1) — https://www.nuget.org/packages/Azure.Search.Documents/
- [REF-7] Use Azure.Search.Documents in .NET (how-to) — https://learn.microsoft.com/en-us/azure/search/search-how-to-dotnet-sdk
- [REF-8] SearchIndexClient class (Azure.Search.Documents.Indexes) API reference — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.indexes.searchindexclient?view=azure-dotnet
- [REF-25] Azure.Search.Documents v12.0.0 samples (GitHub) — https://github.com/Azure/azure-sdk-for-net/tree/Azure.Search.Documents_12.0.0/sdk/search/Azure.Search.Documents/samples

### Authentication
- [REF-9] Connect to Azure AI Search using Azure roles (RBAC) — https://learn.microsoft.com/en-us/azure/search/search-security-rbac
- [REF-10] Use keyless connections (DefaultAzureCredential / Entra ID) in search apps — https://learn.microsoft.com/en-us/azure/search/search-howto-aad
- [REF-11] Connect using API keys (admin vs query keys) — https://learn.microsoft.com/en-us/azure/search/search-security-api-keys

### Querying & filters
- [REF-12] (merge rules) Add, update, or delete documents (REST — special merge rules) — https://learn.microsoft.com/en-us/rest/api/searchservice/addupdate-or-delete-documents
- [REF-13] Create a full-text query — https://learn.microsoft.com/en-us/azure/search/search-query-create
- [REF-14] Query types in Azure AI Search — https://learn.microsoft.com/en-us/azure/search/search-query-overview
- [REF-15] Simple query syntax — https://learn.microsoft.com/en-us/azure/search/query-simple-syntax
- [REF-16] Full Lucene query syntax — https://learn.microsoft.com/en-us/azure/search/query-lucene-syntax
- [REF-17] Full-text search: Lucene query architecture & BM25 scoring — https://learn.microsoft.com/en-us/azure/search/search-lucene-query-architecture
- [REF-18] Examples of simple query syntax — https://learn.microsoft.com/en-us/azure/search/search-query-simple-examples
- [REF-19] OData $filter reference (eq/ne/gt/lt/ge/le, quote escaping) — https://learn.microsoft.com/en-us/azure/search/search-query-odata-filter
- [REF-20b] OData language overview ($filter / $orderby syntax) — https://learn.microsoft.com/en-us/azure/search/query-odata-filter-orderby-syntax
- [REF-26] Text query filters (search.ismatch, combining filters with full-text) — https://learn.microsoft.com/en-us/azure/search/search-filters
- [REF-27] SearchOptions.Filter property (Azure.Search.Documents) API reference — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.searchoptions.filter?view=azure-dotnet
- [REF-28] SearchOptions.Skip property (Azure.Search.Documents) API reference (paging; max 100,000) — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.searchoptions.skip?view=azure-dotnet
- [REF-30] SearchOptions.IncludeTotalCount property API reference (default false; approximate count; perf impact) — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.searchoptions.includetotalcount?view=azure-dotnet
- [REF-31] SearchResults<T>.TotalCount property API reference (null unless requested; may exceed page size) — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.models.searchresults-1.totalcount?view=azure-dotnet

### Pricing & limits
- [REF-21] Choose a pricing model and service tier (Free/Basic/Standard/Storage Optimized/Serverless, Search Units) — https://learn.microsoft.com/en-us/azure/search/search-sku-tier
- [REF-22] Service limits for tiers and SKUs (Free tier ~50 MB, one per subscription) — https://learn.microsoft.com/en-us/azure/search/search-limits-quotas-capacity
- [REF-29] Azure AI Search pricing (product pricing page) — https://azure.microsoft.com/en-us/pricing/details/search/

### Branding & setup
- [REF-23] What's new in Azure AI Search (rename from Azure Cognitive Search, Nov 15 2023) — https://learn.microsoft.com/en-us/azure/search/whats-new
- [REF-24] Create an Azure AI Search service in the portal — https://learn.microsoft.com/en-us/azure/search/search-create-service-portal
