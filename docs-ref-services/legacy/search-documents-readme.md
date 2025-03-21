---
title: Azure Cognitive Search client library for Python
keywords: Azure, python, SDK, API, azure-search-documents, search
ms.date: 07/07/2020
ms.topic: reference
ms.devlang: python
ms.service: search
---
# Azure Cognitive Search client library for Python - version 11.0.0 


[Azure Cognitive Search](https://docs.microsoft.com/azure/search/) is a
search-as-a-service cloud solution that gives developers APIs and tools
for adding a rich search experience over private, heterogeneous content
in web, mobile, and enterprise applications.

The Azure Cognitive Search service is well suited for the following
 application scenarios:

* Consolidate varied content types into a single searchable index.
  To populate an index, you can push JSON documents that contain your content,
  or if your data is already in Azure, create an indexer to pull in data
  automatically.
* Attach skillsets to an indexer to create searchable content from images
  and large text documents. A skillset leverages AI from Cognitive Services
  for built-in OCR, entity recognition, key phrase extraction, language
  detection, text translation, and sentiment analysis. You can also add
  custom skills to integrate external processing of your content during
  data ingestion.
* In a search client application, implement query logic and user experiences
  similar to commercial web search engines.

Use the Azure.Search.Documents client library to:

* Submit queries for simple and advanced query forms that include fuzzy
  search, wildcard search, regular expressions.
* Implement filtered queries for faceted navigation, geospatial search,
  or to narrow results based on filter criteria.
* Create and manage search indexes.
* Upload and update documents in the search index.
* Create and manage indexers that pull data from Azure into an index.
* Create and manage skillsets that add AI enrichment to data ingestion.
* Create and manage analyzers for advanced text analysis or multi-lingual content.
* Optimize results through scoring profiles to factor in business logic or freshness.

[Source code](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents) |
[Package (PyPI)](https://pypi.org/project/azure-search-documents/) |
[API reference documentation](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-search-documents/latest/index.html) |
[Product documentation](https://docs.microsoft.com/azure/search/search-what-is-azure-search) |
[Samples](https://github.com/Azure/azure-sdk-for-python/tree/7cd31ac01fed9c790cec71de438af9c45cb45821/sdk/search/azure-search-documents/samples)


## Getting started

### Install the package

Install the Azure Cognitive Search client library for Python - version 11.0.0 
 with [pip](https://pypi.org/project/pip/):

```bash
pip install azure-search-documents --pre
```

### Prerequisites

* Python 2.7, or 3.5 or later is required to use this package.
* You need an [Azure subscription][azure_sub] and a
[Azure Cognitive Search service][search_resource] to use this package.

To create a new search service, you can use the [Azure portal][create_search_service_docs], [Azure PowerShell][create_search_service_ps], or the [Azure CLI][create_search_service_cli].

```Powershell
az search service create --name <mysearch> --resource-group <mysearch-rg> --sku free --location westus
```

See [choosing a pricing tier](https://docs.microsoft.com/azure/search/search-sku-tier)
 for more information about available options.

### Authenticate the client

All requests to a search service need an api-key that was generated specifically
for your service. [The api-key is the sole mechanism for authenticating access to
your search service endpoint.](https://docs.microsoft.com/azure/search/search-security-api-keys)
You can obtain your api-key from the
[Azure portal](https://portal.azure.com/) or via the Azure CLI:

```Powershell
az search admin-key show --service-name <mysearch> --resource-group <mysearch-rg>
```

There are two types of keys used to access your search service: **admin**
*(read-write)* and **query** *(read-only)* keys.  Restricting access and
operations in client apps is essential to safeguarding the search assets on your
service.  Always use a query key rather than an admin key for any query
originating from a client app.

*Note: The example Azure CLI snippet above retrieves an admin key so it's easier
to get started exploring APIs, but it should be managed carefully.*

We can use the api-key to create a new `SearchClient`.

```python
import os
from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient

index_name = "nycjobs";
# Get the service endpoint and API key from the environment
endpoint = os.environ["SEARCH_ENDPOINT"]
key = os.environ["SEARCH_API_KEY"]

# Create a client
credential = AzureKeyCredential(key)
client = SearchClient(endpoint=endpoint,
                      index_name=index_name,
                      credential=credential)
```


### Send your first search request

To get running immediately, we're going to connect to a well known sandbox
Search service provided by Microsoft.  This means you do not need an Azure
subscription or Azure Cognitive Search service to try out this query.


```python
from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient

# We'll connect to the Azure Cognitive Search public sandbox and send a
# query to its "nycjobs" index built from a public dataset of available jobs
# in New York.
service_name = "azs-playground"
index_name = "nycjobs"
api_key = "252044BE3886FE4A8E3BAA4F595114BB"

# Create a SearchClient to send queries
endpoint = "https://{}.search.windows.net/".format(service_name)
credential = AzureKeyCredential(api_key)
client = SearchClient(endpoint=endpoint,
                      index_name=index_name,
                      credential=credential)

# Let's get the top 5 jobs related to Microsoft
results = client.search(search_text="Microsoft", top=5)

for result in results:
    # Print out the title and job description
    print("{}\n{}\n)".format(result["business_title"], result["job_description"]))
```

## Key concepts

An Azure Cognitive Search service contains one or more indexes that provide
persistent storage of searchable data in the form of JSON documents.  _(If
you're brand new to search, you can make a very rough analogy between
indexes and database tables.)_  The Azure.Search.Documents client library
exposes operations on these resources through two main client types.

* `SearchClient` helps with:
  * [Searching](https://docs.microsoft.com/azure/search/search-lucene-query-architecture)
    your indexed documents using
    [rich queries](https://docs.microsoft.com/azure/search/search-query-overview)
    and [powerful data shaping](https://docs.microsoft.com/azure/search/search-filters)
  * [Autocompleting](https://docs.microsoft.com/rest/api/searchservice/autocomplete)
    partially typed search terms based on documents in the index
  * [Suggesting](https://docs.microsoft.com/rest/api/searchservice/suggestions)
    the most likely matching text in documents as a user types
  * [Adding, Updating or Deleting Documents](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents)
    documents from an index

* `SearchIndexClient` allows you to:
  * [Create, delete, update, or configure a search index](https://docs.microsoft.com/rest/api/searchservice/index-operations)
  * [Declare custom synonym maps to expand or rewrite queries](https://docs.microsoft.com/rest/api/searchservice/synonym-map-operations)
  * Most of the `SearchServiceClient` functionality is not yet available in our current preview

* `SearchIndexerClient` allows you to:
  * [Start indexers to automatically crawl data sources](https://docs.microsoft.com/rest/api/searchservice/indexer-operations)
  * [Define AI powered Skillsets to transform and enrich your data](https://docs.microsoft.com/rest/api/searchservice/skillset-operations)

_The `Azure.Search.Documents` client library (v1) is a brand new offering for
Python developers who want to use search technology in their applications.  There
is an older, fully featured `Microsoft.Azure.Search` client library (v10) with
many similar looking APIs, so please be careful to avoid confusion when
exploring online resources._

## Examples

The following examples all use a simple [Hotel data set](https://docs.microsoft.com/samples/azure-samples/azure-search-sample-data/azure-search-sample-data/)
that you can [import into your own index from the Azure portal.](https://docs.microsoft.com/azure/search/search-get-started-portal#step-1---start-the-import-data-wizard-and-create-a-data-source)
These are just a few of the basics - please [check out our Samples](https://github.com/Azure/azure-sdk-for-python/tree/7cd31ac01fed9c790cec71de438af9c45cb45821/sdk/search/azure-search-documents/samples) for
much more.


* [Querying](#querying)
* [Creating an index](#creating-an-index)
* [Adding documents to your index](#adding-documents-to-your-index)
* [Retrieve a specific document from an index](#retrieve-a-specific-document-from-an-index)
* [Async APIs](#async-apis)


### Querying

Let's start by importing our namespaces.

```python
import os
from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient
```

We'll then create a `SearchClient` to access our hotels search index.

```python
index_name = "hotels"
# Get the service endpoint and API key from the environment
endpoint = os.environ["SEARCH_ENDPOINT"]
key = os.environ["SEARCH_API_KEY"]

# Create a client
credential = AzureKeyCredential(key)
client = SearchClient(endpoint=endpoint,
                      index_name=index_name,
                      credential=credential)
```

Let's search for a "luxury" hotel.

```python
results = client.search(search_text="luxury")

for result in results:
    print("{}: {})".format(result["hotelId"], result["hotelName"]))
```


### Creating an index

You can use the `SearchIndexClient` to create a search index. Fields can be
defined using convenient `SimpleField`, `SearchableField`, or `ComplexField`
models. Indexes can also define suggesters, lexical analyzers, and more.

```python
import os
from azure.core.credentials import AzureKeyCredential
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import ( 
    ComplexField, 
    CorsOptions, 
    SearchIndex, 
    ScoringProfile, 
    SearchFieldDataType, 
    SimpleField, 
    SearchableField 
)

endpoint = os.environ["SEARCH_ENDPOINT"]
key = os.environ["SEARCH_API_KEY"]

# Create a service client
client = SearchIndexClient(endpoint, AzureKeyCredential(key))

# Create the index
name = "hotels"
fields = [
        SimpleField(name="hotelId", type=SearchFieldDataType.String, key=True),
        SimpleField(name="baseRate", type=SearchFieldDataType.Double),
        SearchableField(name="description", type=SearchFieldDataType.String),
        ComplexField(name="address", fields=[
            SimpleField(name="streetAddress", type=SearchFieldDataType.String),
            SimpleField(name="city", type=SearchFieldDataType.String),
        ])
    ]
cors_options = CorsOptions(allowed_origins=["*"], max_age_in_seconds=60)
scoring_profiles = []

index = SearchIndex(
    name=name,
    fields=fields,
    scoring_profiles=scoring_profiles,
    cors_options=cors_options)

result = client.create_index(index)
```


### Adding documents to your index

You can `Upload`, `Merge`, `MergeOrUpload`, and `Delete` multiple documents from
an index in a single batched request.  There are
[a few special rules for merging](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents#document-actions)
to be aware of.

```python
import os
from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient

index_name = "hotels"
endpoint = os.environ["SEARCH_ENDPOINT"]
key = os.environ["SEARCH_API_KEY"]

DOCUMENT = {
    'Category': 'Hotel',
    'HotelId': '1000',
    'Rating': 4.0,
    'Rooms': [],
    'HotelName': 'Azure Inn',
}

result = client.upload_documents(documents=[DOCUMENT])

print("Upload of new document succeeded: {}".format(result[0].succeeded))
```


### Retrieve a specific document from an index

In addition to querying for documents using keywords and optional filters,
you can retrieve a specific document from your index if you already know the
key. You could get the key from a query, for example, and want to show more
information about it or navigate your customer to that document.

```python
import os
from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient

index_name = "hotels"
endpoint = os.environ["SEARCH_ENDPOINT"]
key = os.environ["SEARCH_API_KEY"]

client = SearchClient(endpoint, index_name, AzureKeyCredential(key))

result = client.get_document(key="1")

print("Details for hotel '1' are:")
print("        Name: {}".format(result["HotelName"]))
print("      Rating: {}".format(result["Rating"]))
print("    Category: {}".format(result["Category"]))
```


### Async APIs
This library includes a complete async API supported on Python 3.5+. To use it, you must
first install an async transport, such as [aiohttp](https://pypi.org/project/aiohttp/).
See
[azure-core documentation](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/core/azure-core/README.md#transport)
for more information.


```py
from azure.core.credentials import AzureKeyCredential
from azure.search.documents.aio import SearchClient

client = SearchClient(endpoint, index_name, AzureKeyCredential(api_key))

async with client:
  results = await client.search(search_text="hotel")
  async for result in results:
    print("{}: {})".format(result["hotelId"], result["hotelName"]))
...


## Troubleshooting

### General

The Azure Cognitive Search client will raise exceptions defined in [Azure Core][azure_core].

### Logging

This library uses the standard [logging][python_logging] library for logging.
Basic information about HTTP sessions (URLs, headers, etc.) is logged at INFO
level.

Detailed DEBUG level logging, including request/response bodies and unredacted
headers, can be enabled on a client with the `logging_enable` keyword argument:
```python
import sys
import logging
from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient

# Create a logger for the 'azure' SDK
logger = logging.getLogger('azure')
logger.setLevel(logging.DEBUG)

# Configure a console output
handler = logging.StreamHandler(stream=sys.stdout)
logger.addHandler(handler)

# This client will log detailed information about its HTTP sessions, at DEBUG level
client = SearchClient("<service endpoint>", "<index_name>", AzureKeyCredential("<api key>"), logging_enable=True)

```

Similarly, `logging_enable` can enable detailed logging for a single operation,
even when it isn't enabled for the client:
```python
result =  client.search(search_text="spa", logging_enable=True)
```

## Next steps

* [Go further with Azure.Search.Documents and our samples](https://github.com/Azure/azure-sdk-for-python/tree/7cd31ac01fed9c790cec71de438af9c45cb45821/sdk/search/azure-search-documents/samples)
* [Watch a demo or deep dive video](https://azure.microsoft.com/resources/videos/index/?services=search)
* [Read more about the Azure Cognitive Search service](https://docs.microsoft.com/azure/search/search-what-is-azure-search)

## Contributing

This project welcomes contributions and suggestions.  Most contributions require
you to agree to a Contributor License Agreement (CLA) declaring that you have
the right to, and actually do, grant us the rights to use your contribution. For
details, visit [cla.microsoft.com][cla].

This project has adopted the [Microsoft Open Source Code of Conduct][code_of_conduct].
For more information see the [Code of Conduct FAQ][coc_faq]
or contact [opencode@microsoft.com][coc_contact] with any
additional questions or comments.



## Related projects

* [Microsoft Azure SDK for Python](https://github.com/Azure/azure-sdk-for-python)

<!-- LINKS -->



[azure_cli]: https://docs.microsoft.com/cli/azure
[azure_core]: https://github.com/Azure/azure-sdk-for-python/tree/7cd31ac01fed9c790cec71de438af9c45cb45821/sdk/core/azure-core/README.md
[azure_sub]: https://azure.microsoft.com/free/
[search_resource]: https://docs.microsoft.com/azure/search/search-create-service-portal
[azure_portal]: https://portal.azure.com

[create_search_service_docs]: https://docs.microsoft.com/azure/search/search-create-service-portal
[create_search_service_ps]: https://docs.microsoft.com/azure/search/search-manage-powershell#create-or-delete-a-service
[create_search_service_cli]: https://docs.microsoft.com/cli/azure/search/service?view=azure-cli-latest#az-search-service-create
[search_contrib]: ../CONTRIBUTING.md
[python_logging]: https://docs.python.org/3.5/library/logging.html

[cla]: https://cla.microsoft.com
[code_of_conduct]: https://opensource.microsoft.com/codeofconduct/
[coc_faq]: https://opensource.microsoft.com/codeofconduct/faq/
[coc_contact]: mailto:opencode@microsoft.com

