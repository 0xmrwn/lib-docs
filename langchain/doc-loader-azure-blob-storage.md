# Azure Blob Storage Container

> [Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) is Microsoft's object storage solution for the cloud. Blob Storage is optimized for storing massive amounts of unstructured data. Unstructured data is data that doesn't adhere to a particular data model or definition, such as text or binary data.

`Azure Blob Storage` is designed for:

- Serving images or documents directly to a browser.
- Storing files for distributed access.
- Streaming video and audio.
- Writing to log files.
- Storing data for backup and restore, disaster recovery, and archiving.
- Storing data for analysis by an on-premises or Azure-hosted service.

This notebook covers how to load document objects from a container on `Azure Blob Storage`.

```codeBlockLines_e6Vv
%pip install --upgrade --quiet  azure-storage-blob

```

```codeBlockLines_e6Vv
from langchain_community.document_loaders import AzureBlobStorageContainerLoader

```

**API Reference:** [AzureBlobStorageContainerLoader](https://python.langchain.com/api_reference/community/document_loaders/langchain_community.document_loaders.azure_blob_storage_container.AzureBlobStorageContainerLoader.html)

```codeBlockLines_e6Vv
loader = AzureBlobStorageContainerLoader(conn_str="<conn_str>", container="<container>")

```

```codeBlockLines_e6Vv
loader.load()

```

```codeBlockLines_e6Vv
[Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': '/var/folders/y6/8_bzdg295ld6s1_97_12m4lr0000gn/T/tmpaa9xl6ch/fake.docx'}, lookup_index=0)]

```

## Specifying a prefix [​](https://python.langchain.com/docs/integrations/document_loaders/azure_blob_storage_container/\#specifying-a-prefix "Direct link to Specifying a prefix")

You can also specify a prefix for more fine-grained control over what files to load.

```codeBlockLines_e6Vv
loader = AzureBlobStorageContainerLoader(
    conn_str="<conn_str>", container="<container>", prefix="<prefix>"
)

```

```codeBlockLines_e6Vv
loader.load()

```

```codeBlockLines_e6Vv
[Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': '/var/folders/y6/8_bzdg295ld6s1_97_12m4lr0000gn/T/tmpujbkzf_l/fake.docx'}, lookup_index=0)]

```

## Related [​](https://python.langchain.com/docs/integrations/document_loaders/azure_blob_storage_container/\#related "Direct link to Related")

- Document loader [conceptual guide](https://python.langchain.com/docs/concepts/document_loaders/)
- Document loader [how-to guides](https://python.langchain.com/docs/how_to/#document-loaders)

- [Specifying a prefix](https://python.langchain.com/docs/integrations/document_loaders/azure_blob_storage_container/#specifying-a-prefix)
- [Related](https://python.langchain.com/docs/integrations/document_loaders/azure_blob_storage_container/#related)