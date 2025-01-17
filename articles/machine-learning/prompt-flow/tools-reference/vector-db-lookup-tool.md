---
title: Vector DB Lookup tool in Azure Machine Learning prompt flow
titleSuffix: Azure Machine Learning
description: Vector DB Lookup is a vector search tool that allows users to search top k similar vectors from vector database. This tool is a wrapper for multiple third-party vector databases. The list of current supported databases is as follows.
services: machine-learning
ms.service: machine-learning
ms.subservice: prompt-flow
ms.custom:
  - ignite-2023
ms.topic: reference
author: likebupt
ms.author: keli19
ms.reviewer: lagayhar
ms.date: 11/02/2023
---

# Vector DB Lookup tool

Vector DB Lookup is a vector search tool that allows users to search top k similar vectors from vector database. This tool is a wrapper for multiple third-party vector databases. The list of current supported databases is as follows.

| Name | Description |
| --- | --- |
| Azure AI Search (formerly Cognitive Search) | Microsoft's cloud search service with built-in AI capabilities that enrich all types of information to help identify and explore relevant content at scale. |
| Qdrant | Qdrant is a vector similarity search engine that provides a production-ready service with a convenient API to store, search and manage points (i.e. vectors) with an additional payload. |
| Weaviate | Weaviate is an open source vector database that stores both objects and vectors. This allows for combining vector search with structured filtering. |

This tool will support more vector databases.

## Prerequisites

The tool searches data from a third-party vector database. To use it, you should create resources in advance and establish connection between the tool and the resource.

  - **Azure AI Search:**
    - Create resource [Azure AI Search](../../../search/search-create-service-portal.md).
    - Add "Cognitive search" connection. Fill "API key" field with "Primary admin key" from "Keys" section of created resource, and fill "API base" field with the URL, the URL format is `https://{your_serive_name}.search.windows.net`.

  - **Qdrant:**
    - Follow the [installation](https://qdrant.tech/documentation/quick-start/) to deploy Qdrant to a self-maintained cloud server.
    - Add "Qdrant" connection. Fill "API base" with your self-maintained cloud server address and fill "API key" field.

  - **Weaviate:**
    - Follow the [installation](https://weaviate.io/developers/weaviate/installation) to deploy Weaviate to a self-maintained instance.
    - Add "Weaviate" connection. Fill "API base" with your self-maintained instance address and fill "API key" field.

> [!NOTE]
> When legacy tools switching to code first mode, if you encounter "'embeddingstore.tool.vector_db_lookup.search' is not found" error, please refer to the [Troubleshoot Guidance](./troubleshoot-guidance.md).

## Inputs

The tool accepts the following inputs:
- **Azure AI Search:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | connection | CognitiveSearchConnection | The created connection for accessing to Azure AI Search endpoint. | Yes |
  | index_name | string | The index name created in Azure AI Search resource. | Yes |
  | text_field | string | The text field name. The returned text field will populate the text of output. | No |
  | vector_field | string | The vector field name. The target vector is searched in this vector field. | Yes |
  | search_params | dict | The search parameters. It's key-value pairs. Except for parameters in the tool input list mentioned above, additional search parameters can be formed into a JSON object as search_params. For example, use `{"select": ""}` as search_params to select the returned fields, use `{"search": ""}` to perform a [hybrid search](../../../search/search-get-started-vector.md#hybrid-search). | No |
  | search_filters | dict | The search filters. It's key-value pairs, the input format is like `{"filter": ""}` | No |
  | vector | list | The target vector to be queried, which can be generated by Embedding tool. | Yes |
  | top_k | int | The count of top-scored entities to return. Default value is 3 | No |

- **Qdrant:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | connection | QdrantConnection | The created connection for accessing to Qdrant server. | Yes |
  | collection_name | string | The collection name created in self-maintained cloud server. | Yes |
  | text_field | string | The text field name. The returned text field will populate the text of output. | No |
  | search_params | dict | The search parameters can be formed into a JSON object as search_params. For example, use `{"params": {"hnsw_ef": 0, "exact": false, "quantization": null}}` to set search_params. | No |
  | search_filters | dict | The search filters. It's key-value pairs, the input format is like `{"filter": {"should": [{"key": "", "match": {"value": ""}}]}}` | No |
  | vector | list | The target vector to be queried, which can be generated by Embedding tool. | Yes |
  | top_k | int | The count of top-scored entities to return. Default value is 3 | No |

- **Weaviate:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | connection | WeaviateConnection | The created connection for accessing to Weaviate. | Yes |
  | class_name | string | The class name. | Yes |
  | text_field | string | The text field name. The returned text field will populate the text of output. | No |
  | vector | list | The target vector to be queried, which can be generated by Embedding tool. | Yes |
  | top_k | int | The count of top-scored entities to return. Default value is 3 | No |

## Outputs

The following is an example JSON format response returned by the tool, which includes the top-k scored entities. The entity follows a generic schema of vector search result provided by promptflow-vectordb SDK. 
- **Azure AI Search:**

  For Azure AI Search, the following fields are populated:

  | Field Name | Type | Description |
  | ---- | ---- | ----------- |
  | original_entity | dict | the original response json from search REST API|
  | score | float |  @search.score from the original entity, which evaluates the similarity between the entity and the query vector |
  | text | string | text of the entity|
  | vector | list | vector of the entity|

  <details>
    <summary>Output</summary>
    
  ```json
  [
    {
      "metadata": null,
      "original_entity": {
        "@search.score": 0.5099789,
        "id": "",
        "your_text_filed_name": "sample text1",
        "your_vector_filed_name": [-0.40517663431890405, 0.5856996257406859, -0.1593078462266455, -0.9776269170785785, -0.6145604369828972],
        "your_additional_field_name": ""
      },
      "score": 0.5099789,
      "text": "sample text1",
      "vector": [-0.40517663431890405, 0.5856996257406859, -0.1593078462266455, -0.9776269170785785, -0.6145604369828972]
    }
  ]
  ```
  </details>

- **Qdrant:**

  For Qdrant, the following fields are populated:

  | Field Name | Type | Description |
  | ---- | ---- | ----------- |
  | original_entity | dict | the original response json from search REST API|
  | metadata | dict | payload from the original entity|
  | score | float | score from the original entity, which evaluates the similarity between the entity and the query vector|
  | text | string | text of the payload|
  | vector | list | vector of the entity|

  <details>
    <summary>Output</summary>
    
  ```json
  [
    {
      "metadata": {
        "text": "sample text1"
      },
      "original_entity": {
        "id": 1,
        "payload": {
          "text": "sample text1"
        },
        "score": 1,
        "vector": [0.18257418, 0.36514837, 0.5477226, 0.73029673],
        "version": 0
      },
      "score": 1,
      "text": "sample text1",
      "vector": [0.18257418, 0.36514837, 0.5477226, 0.73029673]
    }
  ]
  ```
  </details>

- **Weaviate:**

  For Weaviate, the following fields are populated:
  
  | Field Name | Type | Description |
  | ---- | ---- | ----------- |
  | original_entity | dict | the original response json from search REST API|
  | score | float | certainty from the original entity, which evaluates the similarity between the entity and the query vector|
  | text | string | text in the original entity|
  | vector | list | vector of the entity|

  <details>
    <summary>Output</summary>
    
  ```json
  [
    {
      "metadata": null,
      "original_entity": {
        "_additional": {
          "certainty": 1,
          "distance": 0,
          "vector": [
            0.58,
            0.59,
            0.6,
            0.61,
            0.62
          ]
        },
        "text": "sample text1."
      },
      "score": 1,
      "text": "sample text1.",
      "vector": [
        0.58,
        0.59,
        0.6,
        0.61,
        0.62
      ]
    }
  ]
  ```
  </details>
