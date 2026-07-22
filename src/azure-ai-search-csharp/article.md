---
title: "Getting Started with Azure AI Search in C#"
description: "Learn how to use Azure AI Search in C# with the Azure.Search.Documents SDK: create an index, upload documents, and run keyword queries with filters."
focus_keyphrase: "Azure AI Search in C#"
keywords:
  - "Azure AI Search in C#"
  - "Azure.Search.Documents"
  - ".NET search SDK"
  - "keyword search C#"
  - "SearchClient"
author: ""
date: "2026-07-22"
slug: "azure-ai-search-csharp"
---

# Getting Started with Azure AI Search in C#

Building search that actually works is deceptively hard. Users expect typo tolerance, relevance ranking, filtering, and sub-second results — and rolling your own on top of `LIKE '%term%'` queries falls apart the moment your data grows. **Azure AI Search in C#** gives you a fully managed search engine you can populate and query from a few dozen lines of .NET code, so you can spend your time on your product instead of on an inverted index.

This guide walks intermediate .NET developers through the essentials: connecting to a search service, creating an index, uploading documents, and running keyword (full-text) queries with filters, ordering, and pagination — all with the current `Azure.Search.Documents` SDK.

---

## What is Azure AI Search?

**Azure AI Search** is Microsoft's managed, cloud-based information-retrieval platform for building rich search experiences over your own content [REF-1]. It handles the indexing, tokenization, relevance scoring, and query infrastructure that you would otherwise have to build and operate yourself.

One naming note trips up almost everyone: the product was **formerly named Azure Cognitive Search** and was renamed to **Azure AI Search on November 15, 2023**. The rename brought **no breaking API or SDK changes and no pricing changes** [REF-1][REF-5][REF-23]. So when you find an older tutorial under the "Cognitive Search" name, the concepts still apply — but watch the SDK package (more on that below).

### Where it fits

This article focuses on **keyword / full-text search**, the bread-and-butter capability: a user types words, and the service returns the most relevant documents ranked by an Apache Lucene-based **BM25** scoring algorithm [REF-17]. Azure AI Search also supports **vector** and **semantic** search for AI-powered relevance, but those are out of scope here — see [Next Steps](#next-steps) for where to go [REF-5].

---

## Core concepts

Before writing code, it helps to get four terms straight [REF-1][REF-2][REF-5]:

- **Search service** — the top-level resource you provision in Azure. It hosts your indexes, indexers, data sources, and other objects.
- **Index** — persistent storage of searchable JSON **documents**. This is what you query.
- **Document** — a single searchable item (a hotel, a product, an article).
- **Field** — a named, typed property on a document. Each field carries **attributes** that control behavior: `key`, `searchable`, `filterable`, `sortable`, `facetable`, and `retrievable` [REF-2][REF-3].

If you come from a relational background, a rough (and imperfect) analogy helps: an **index is like a table**, a **document is like a row**, and a **field is like a column** [REF-2]. Text fields are processed by **analyzers**, which tokenize and normalize text so full-text search can match variations of a word [REF-2].

---

## Setting up

You need a search service and a .NET project.

### Create a service

You can create a search service in the [Azure portal](https://learn.microsoft.com/en-us/azure/search/search-create-service-portal) in a few clicks [REF-24]. For learning and prototyping, the **Free tier** works — but note its limits before you commit to it (see [Pricing & tiers](#pricing--tiers)) [REF-21][REF-22].

### Install the SDK

Add the current data-plane package and, for keyless auth, the identity package:

```bash
dotnet add package Azure.Search.Documents
dotnet add package Azure.Identity
```

The current stable NuGet package is **`Azure.Search.Documents` v12.0.0** (released 2026-05-01; latest prerelease `12.1.0-beta.1`) [REF-6]. One important caveat: the NuGet **major version is 12**, while the docs sometimes still refer to the "v11" data-plane library lineage. The key point is that `Azure.Search.Documents` (namespace `Azure.Search.Documents`) is the **current** library; the older **`Microsoft.Azure.Search` (v10) package is retired** — do not mix samples from the two [REF-5][REF-6].

---

## Connecting from C#

The SDK exposes two clients you will use in this article [REF-5][REF-8]:

- **`SearchIndexClient`** — creates and manages indexes (namespace `Azure.Search.Documents.Indexes`).
- **`SearchClient`** — uploads/merges/deletes documents and runs queries (namespace `Azure.Search.Documents`).

*(There is also a `SearchIndexerClient` for indexers and skillsets, which is out of scope here [REF-5].)*

### Authentication: keyless (recommended) vs API key

You have two options [REF-5][REF-9][REF-10][REF-11]:

1. **Microsoft Entra ID (RBAC)** via `DefaultAzureCredential` from `Azure.Identity` — the recommended modern default. It uses role assignments such as **Search Index Data Reader** (for queries) and **Search Index Data Contributor** (for writes), so there are no secrets in your code [REF-9][REF-10].
2. **API key** via `AzureKeyCredential` — the quickest way to get going. Keys come in two flavors: an **admin key** (read-write) and a **query key** (read-only). Always prefer a **query key** in client-facing apps [REF-11].

The recommended keyless approach:

```csharp
using Azure.Identity;
using Azure.Search.Documents;
using Azure.Search.Documents.Indexes;

var endpoint = new Uri("https://<your-service>.search.windows.net");
var credential = new DefaultAzureCredential();

// Manages indexes
var indexClient = new SearchIndexClient(endpoint, credential);

// Queries and document operations against a specific index
var searchClient = new SearchClient(endpoint, "hotels", credential);
```

The API-key alternative simply swaps the credential type:

```csharp
using Azure;
using Azure.Search.Documents;

var endpoint = new Uri("https://<your-service>.search.windows.net");
var credential = new AzureKeyCredential("<query-or-admin-key>");

var searchClient = new SearchClient(endpoint, "hotels", credential);
```

---

## Creating an index in C#

Let's define a simple `Hotel` domain model. The attribute-driven approach lets you decorate a POCO with field attributes and generate the index schema with `FieldBuilder` [REF-5][REF-3].

```csharp
using Azure.Search.Documents.Indexes;
using Azure.Search.Documents.Indexes.Models;

public class Hotel
{
    // Every index needs exactly one string key field.
    [SimpleField(IsKey = true)]
    public string HotelId { get; set; }

    // Full-text searchable; also sortable for name-based ordering.
    [SearchableField(IsSortable = true)]
    public string HotelName { get; set; }

    // Full-text searchable description with an English analyzer.
    [SearchableField(AnalyzerName = "en.microsoft")]
    public string Description { get; set; }

    // Not full-text searchable, but filterable/sortable/facetable.
    [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
    public double Rating { get; set; }

    [SimpleField(IsFilterable = true, IsFacetable = true)]
    public string Category { get; set; }
}
```

A few rules worth internalizing [REF-5][REF-3]:

- **Exactly one key field** — marked `IsKey = true`, and it must be a `string`.
- **`SimpleField`** — stores data that is *not* full-text searchable; use it for values you filter, sort, or facet on (numbers, dates, keywords).
- **`SearchableField`** — full-text searchable text; you can also set an **`AnalyzerName`** to control tokenization.

Now build the fields and create the index:

```csharp
using Azure.Search.Documents.Indexes;
using Azure.Search.Documents.Indexes.Models;

var fieldBuilder = new FieldBuilder();
var fields = fieldBuilder.Build(typeof(Hotel));

var definition = new SearchIndex("hotels")
{
    Fields = fields
};

indexClient.CreateIndex(definition);
```

If you prefer explicit control, you can construct `SimpleField`, `SearchableField`, and `ComplexField` objects directly instead of using attributes — both patterns are supported [REF-5].

---

## Uploading documents

Documents are indexed in **batches** using `IndexDocumentsBatch` and the client's `IndexDocuments` method [REF-5][REF-4]. Each document in a batch is wrapped in an **action** that tells the service what to do:

- **`Upload`** — inserts or fully replaces a document (an upsert of the whole document).
- **`MergeOrUpload`** — partially updates an existing document, or inserts it if absent.
- **`Merge`** — partially updates an existing document; it **fails if the document does not already exist** [REF-12].
- **`Delete`** — removes a document.

```csharp
using System.Linq;
using Azure.Search.Documents;
using Azure.Search.Documents.Models;

var hotels = new[]
{
    new Hotel { HotelId = "1", HotelName = "Stay-Kay City", Description = "Downtown boutique hotel.", Rating = 4.5, Category = "Boutique" },
    new Hotel { HotelId = "2", HotelName = "Old Century Hotel", Description = "Renovated historic landmark.", Rating = 3.8, Category = "Historic" }
};

var batch = IndexDocumentsBatch.Create(
    hotels.Select(h => IndexDocumentsAction.Upload(h)).ToArray());

var result = searchClient.IndexDocuments(batch);
```

### Handle partial failures

A batch can **partially succeed** — some documents may fail while others index cleanly [REF-5][REF-4]. You have two choices:

- Set `IndexDocumentsOptions { ThrowOnAnyError = true }` to fail loudly on any error.
- Or inspect the returned `IndexDocumentsResult` per action to see which keys succeeded.

```csharp
var options = new IndexDocumentsOptions { ThrowOnAnyError = true };
var result = searchClient.IndexDocuments(batch, options);
```

For updates to existing documents, prefer `MergeOrUpload` so you don't have to resend every field:

```csharp
var patch = IndexDocumentsBatch.Create(
    IndexDocumentsAction.MergeOrUpload(
        new Hotel { HotelId = "1", Rating = 4.7 }));

searchClient.IndexDocuments(patch);
```

---

## Running keyword queries

To query, call **`Search<T>`** with your search text and a `SearchOptions` object; it returns `SearchResults<T>` that you iterate via `GetResults()` (or `SearchAsync` + `GetResultsAsync` for async code) [REF-5][REF-13].

```csharp
using Azure.Search.Documents;
using Azure.Search.Documents.Models;

SearchResults<Hotel> response = searchClient.Search<Hotel>("boutique downtown");

foreach (SearchResult<Hotel> result in response.GetResults())
{
    Hotel hotel = result.Document;
    Console.WriteLine($"{hotel.HotelName} (score: {result.Score})");
}
```

By default, the **simple query parser** applies — it supports `AND`/`OR`/`NOT`, phrase search, and prefix matching with `*` [REF-15]. If you need fuzzy search, proximity, or regex, switch to the **full Lucene** parser with `queryType=full` [REF-16]. Either way, results are ranked by **BM25** relevance scoring [REF-17].

### Shape results with SearchOptions

`SearchOptions` is where the real power lives [REF-5][REF-13]:

| Option | Purpose |
| --- | --- |
| **`Filter`** | OData `$filter` expression to narrow results (e.g., `Rating ge 4`) [REF-19] |
| **`OrderBy`** | Sort expression, e.g. `"Rating desc"` [REF-20b] |
| **`Select`** | Which fields to return |
| **`Size`** | Number of results to return (page size) [REF-5] |
| **`SearchMode`** | `Any` (default) vs `All` — controls Boolean precision/recall |

Build filters safely with **`SearchFilter.Create`**, which handles OData escaping (single quotes in string literals are escaped by doubling them, e.g. `'Alice''s'`) [REF-19][REF-27]:

```csharp
var options = new SearchOptions
{
    Filter = SearchFilter.Create($"Rating ge {4} and Category eq {"Boutique"}"),
    OrderBy = { "Rating desc" },
    Select = { "HotelId", "HotelName", "Rating" },
    Size = 10
};

SearchResults<Hotel> response = searchClient.Search<Hotel>("downtown", options);

foreach (var result in response.GetResults())
{
    Console.WriteLine(result.Document.HotelName);
}
```

### Pagination

For paging, set `Size` for the page size and `Skip` to move to later pages; `IncludeTotalCount` requests an approximate total match count so you can render page controls [REF-28][REF-30]. The returned `TotalCount` is `null` unless you request it, and — because it's approximate — it may exceed the number of results in the current page [REF-31]. Note that `Skip` cannot exceed **100,000**; to page beyond that, sort on a unique key and use a range filter instead [REF-28].

```csharp
var page2 = new SearchOptions
{
    Size = 10,
    Skip = 10,                 // second page of 10
    IncludeTotalCount = true
};

SearchResults<Hotel> response = searchClient.Search<Hotel>("*", page2);
Console.WriteLine($"Approx. total matches: {response.TotalCount}");
```

---

## Pricing & tiers

Azure AI Search is billed by **service tier**, with capacity measured in **Search Units (SUs)** [REF-21]. Tiers include **Free**, **Basic**, **Standard (S1–S3, S3 HD)**, and **Storage Optimized (L1–L2)**, plus a consumption-based **Serverless (preview)** model [REF-21].

The **Free tier** is great for tutorials but has real caveats you should know before building anything you care about [REF-21][REF-22]:

- **One free service per subscription**, running on **shared infrastructure**.
- Roughly **50 MB** of storage.
- **No scale-up** — you cannot upgrade a Free service in place; you must create a new one on a paid tier.

Always check the [official pricing page](https://azure.microsoft.com/en-us/pricing/details/search/) for current numbers before you provision [REF-29].

---

## Next Steps

You now have the full loop: connect, create an index, upload documents, and run keyword queries with filters, ordering, and paging — all in C# with `Azure.Search.Documents` [REF-5].

Where to go from here:

- **Add AI relevance.** Explore **vector search** and **semantic ranking**, both supported by the SDK, to move beyond keyword matching [REF-5].
- **Browse working samples.** Clone the official [`Azure.Search.Documents` v12 samples](https://github.com/Azure/azure-sdk-for-net/tree/Azure.Search.Documents_12.0.0/sdk/search/Azure.Search.Documents/samples) for end-to-end code [REF-25].
- **Go keyless in production.** Wire up `DefaultAzureCredential` with the right RBAC roles instead of API keys [REF-9][REF-10].

Provision a Free service, paste in the snippets above, and you'll have searchable data in minutes.

---

*Sources Consulted:*
- [REF-1] Introduction to Azure AI Search — https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search
- [REF-2] Search index overview (indexes, documents, fields, attributes) — https://learn.microsoft.com/en-us/azure/search/search-what-is-an-index
- [REF-3] Create a search index — https://learn.microsoft.com/en-us/azure/search/search-how-to-create-search-index
- [REF-4] Load an index (add/update documents) — https://learn.microsoft.com/en-us/azure/search/search-how-to-load-search-index
- [REF-5] Azure AI Search client library for .NET (README) — https://learn.microsoft.com/en-us/dotnet/api/overview/azure/search.documents-readme?view=azure-dotnet
- [REF-6] NuGet — Azure.Search.Documents — https://www.nuget.org/packages/Azure.Search.Documents/
- [REF-8] SearchIndexClient class API reference — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.indexes.searchindexclient?view=azure-dotnet
- [REF-9] Connect to Azure AI Search using Azure roles (RBAC) — https://learn.microsoft.com/en-us/azure/search/search-security-rbac
- [REF-10] Use keyless connections (DefaultAzureCredential / Entra ID) — https://learn.microsoft.com/en-us/azure/search/search-howto-aad
- [REF-11] Connect using API keys (admin vs query keys) — https://learn.microsoft.com/en-us/azure/search/search-security-api-keys
- [REF-12] Add, update, or delete documents (merge rules) — https://learn.microsoft.com/en-us/rest/api/searchservice/addupdate-or-delete-documents
- [REF-13] Create a full-text query — https://learn.microsoft.com/en-us/azure/search/search-query-create
- [REF-15] Simple query syntax — https://learn.microsoft.com/en-us/azure/search/query-simple-syntax
- [REF-16] Full Lucene query syntax — https://learn.microsoft.com/en-us/azure/search/query-lucene-syntax
- [REF-17] Full-text search: Lucene query architecture & BM25 scoring — https://learn.microsoft.com/en-us/azure/search/search-lucene-query-architecture
- [REF-19] OData $filter reference — https://learn.microsoft.com/en-us/azure/search/search-query-odata-filter
- [REF-20b] OData language overview ($filter / $orderby syntax) — https://learn.microsoft.com/en-us/azure/search/query-odata-filter-orderby-syntax
- [REF-21] Choose a pricing model and service tier — https://learn.microsoft.com/en-us/azure/search/search-sku-tier
- [REF-22] Service limits for tiers and SKUs — https://learn.microsoft.com/en-us/azure/search/search-limits-quotas-capacity
- [REF-23] What's new in Azure AI Search (rename from Azure Cognitive Search) — https://learn.microsoft.com/en-us/azure/search/whats-new
- [REF-24] Create an Azure AI Search service in the portal — https://learn.microsoft.com/en-us/azure/search/search-create-service-portal
- [REF-25] Azure.Search.Documents v12.0.0 samples (GitHub) — https://github.com/Azure/azure-sdk-for-net/tree/Azure.Search.Documents_12.0.0/sdk/search/Azure.Search.Documents/samples
- [REF-27] SearchOptions.Filter property API reference — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.searchoptions.filter?view=azure-dotnet
- [REF-28] SearchOptions.Skip property API reference — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.searchoptions.skip?view=azure-dotnet
- [REF-29] Azure AI Search pricing (product pricing page) — https://azure.microsoft.com/en-us/pricing/details/search/
- [REF-30] SearchOptions.IncludeTotalCount property API reference — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.searchoptions.includetotalcount?view=azure-dotnet
- [REF-31] SearchResults<T>.TotalCount property API reference — https://learn.microsoft.com/en-us/dotnet/api/azure.search.documents.models.searchresults-1.totalcount?view=azure-dotnet
