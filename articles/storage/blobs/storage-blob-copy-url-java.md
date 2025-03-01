---
title: Copy a blob from a source object URL with Java
titleSuffix: Azure Storage
description: Learn how to copy a blob from a source object URL in Azure Storage by using the Java client library.
author: pauljewellmsft

ms.author: pauljewell
ms.date: 04/18/2023
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.devlang: java
ms.custom: devx-track-java, devguide-java
---

# Copy a blob from a source object URL with Java

This article shows how to copy a blob from a source object URL using the [Azure Storage client library for Java](/java/api/overview/azure/storage-blob-readme). You can copy a blob from a source within the same storage account, from a source in a different storage account, or from any accessible object retrieved via HTTP GET request on a given URL.

The client library methods covered in this article use the [Put Blob From URL](/rest/api/storageservices/put-blob-from-url) and [Put Block From URL](/rest/api/storageservices/put-block-from-url) REST API operations. These methods are preferred for copy scenarios where you want to move data into a storage account and have a URL for the source object. For copy operations where you want asynchronous scheduling, see [Copy a blob with asynchronous scheduling using Java](storage-blob-copy-async-java.md).

## Prerequisites

To work with the code examples in this article, make sure you have:

- An authorized client object to connect to Blob Storage data resources. To learn more, see [Create and manage client objects that interact with data resources](storage-blob-client-management.md).
- Permissions to perform a copy operation. To learn more, see the authorization guidance for the following REST API operations:
    - [Put Blob From URL](/rest/api/storageservices/put-blob-from-url#authorization)
    - [Put Block From URL](/rest/api/storageservices/put-block-from-url#authorization)
- Packages installed to your project directory. These examples use **azure-storage-blob**. If you're using `DefaultAzureCredential` for authorization, you also need **azure-identity**. To learn more about setting up your project, see [Get Started with Azure Storage and Java](storage-blob-java-get-started.md#set-up-your-project). To see the necessary `using` directives, see [Code samples](#code-samples).

## About copying a blob from a source object URL

The `Put Blob From URL` operation creates a new block blob where the contents of the blob are read from a given URL. The operation completes synchronously.

The source can be any object retrievable via a standard HTTP GET request on the given URL. This includes block blobs, append blobs, page blobs, blob snapshots, blob versions, or any accessible object inside or outside Azure.

When the source object is a block blob, all committed blob content is copied. The content of the destination blob is identical to the content of the source, but the committed block list isn't preserved and uncommitted blocks aren't copied.

The destination is always a block blob, either an existing block blob, or a new block blob created by the operation. The contents of an existing blob are overwritten with the contents of the new blob.

The `Put Blob From URL` operation always copies the entire source blob. Copying a range of bytes or set of blocks isn't supported. To perform partial updates to a block blob’s contents by using a source URL, use the [Put Block From URL](/rest/api/storageservices/put-block-from-url) API along with [Put Block List](/rest/api/storageservices/put-block-list).

To learn more about the `Put Blob From URL` operation, including blob size limitations and billing considerations, see [Put Blob From URL remarks](/rest/api/storageservices/put-blob-from-url#remarks).

## Copy a blob from a source object URL

This section gives an overview of methods provided by the Azure Storage client library for Java to perform a copy operation from a source object URL.

The following methods wrap the [Put Blob From URL](/rest/api/storageservices/put-blob-from-url) REST API operation, and create a new block blob where the contents of the blob are read from a given URL:

- [uploadFromUrl](/java/api/com.azure.storage.blob.specialized.blockblobclient#method-details)
- [uploadFromUrlWithResponse](/java/api/com.azure.storage.blob.specialized.blockblobclient#method-details)

These methods are preferred for scenarios where you want to move data into a storage account and have a URL for the source object.

For large objects, you can work with individual blocks. The following method wraps the [Put Block From URL](/rest/api/storageservices/put-block-from-url) REST API operation. This method creates a new block to be committed as part of a blob where the contents are read from a source URL:

- [stageBlockFromUrl](/java/api/com.azure.storage.blob.specialized.blockblobclient#method-details)

## Copy a blob from a source within Azure

If you're copying a blob from a source within Azure, access to the source blob can be authorized via Azure Active Directory (Azure AD), a shared access signature (SAS), or an account key. 

The following example shows a scenario for copying a source blob within Azure. The [uploadFromUrl](/java/api/com.azure.storage.blob.specialized.blockblobclient#method-details) method can optionally accept a Boolean parameter to indicate whether an existing blob should be overwritten, as shown in the example.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/blob-devguide/blob-devguide-blobs/src/main/java/com/blobs/devguide/blobs/BlobCopy.java" id="Snippet_CopyFromAzure_PutBlobFromURL":::

The [uploadFromUrlWithResponse](/java/api/com.azure.storage.blob.specialized.blockblobclient#method-details) method can also accept a [BlobUploadFromUrlOptions](/java/api/com.azure.storage.blob.options.blobuploadfromurloptions) parameter to specify further options for the operation.

## Copy a blob from an external source

You can perform a copy operation on any source object that can be retrieved via HTTP GET request on a given URL, including accessible objects outside of Azure. The following example shows a scenario for copying a blob from an accessible source object URL.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/blob-devguide/blob-devguide-blobs/src/main/java/com/blobs/devguide/blobs/BlobCopy.java" id="Snippet_CopyFromExternalSource_PutBlobFromURL":::

## Resources

To learn more about copying blobs using the Azure Blob Storage client library for Java, see the following resources.

### REST API operations

The Azure SDK for Java contains libraries that build on top of the Azure REST API, allowing you to interact with REST API operations through familiar Java paradigms. The client library methods covered in this article use the following REST API operations:

- [Put Blob From URL](/rest/api/storageservices/put-blob-from-url) (REST API)
- [Put Block From URL](/rest/api/storageservices/put-block-from-url) (REST API)

### Code samples

- [View code samples from this article (GitHub)](https://github.com/Azure-Samples/AzureStorageSnippets/blob/master/blobs/howto/Java/blob-devguide/blob-devguide-blobs/src/main/java/com/blobs/devguide/blobs/BlobCopy.java)

[!INCLUDE [storage-dev-guide-resources-java](../../../includes/storage-dev-guides/storage-dev-guide-resources-java.md)]
