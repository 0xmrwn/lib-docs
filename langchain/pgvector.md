# PGVector

> An implementation of LangChain vectorstore abstraction using `postgres` as the backend and utilizing the `pgvector` extension.

The code lives in an integration package called: [langchain\_postgres](https://github.com/langchain-ai/langchain-postgres/).

## Status [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#status "Direct link to Status")

This code has been ported over from `langchain_community` into a dedicated package called `langchain-postgres`. The following changes have been made:

- langchain\_postgres works only with psycopg3. Please update your connnecion strings from `postgresql+psycopg2://...` to `postgresql+psycopg://langchain:langchain@...` (yes, it's the driver name is `psycopg` not `psycopg3`, but it'll use `psycopg3`.
- The schema of the embedding store and collection have been changed to make add\_documents work correctly with user specified ids.
- One has to pass an explicit connection object now.

Currently, there is **no mechanism** that supports easy data migration on schema changes. So any schema changes in the vectorstore will require the user to recreate the tables and re-add the documents.
If this is a concern, please use a different vectorstore. If not, this implementation should be fine for your use case.

## Setup [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#setup "Direct link to Setup")

First donwload the partner package:

```codeBlockLines_e6Vv
pip install -qU langchain_postgres

```

You can run the following command to spin up a a postgres container with the `pgvector` extension:

```codeBlockLines_e6Vv
%docker run --name pgvector-container -e POSTGRES_USER=langchain -e POSTGRES_PASSWORD=langchain -e POSTGRES_DB=langchain -p 6024:5432 -d pgvector/pgvector:pg16

```

### Credentials [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#credentials "Direct link to Credentials")

There are no credentials needed to run this notebook, just make sure you downloaded the `langchain_postgres` package and correctly started the postgres container.

If you want to get best in-class automated tracing of your model calls you can also set your [LangSmith](https://docs.smith.langchain.com/) API key by uncommenting below:

```codeBlockLines_e6Vv
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"

```

## Instantiation [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#instantiation "Direct link to Instantiation")

Select [embeddings model](https://python.langchain.com/docs/integrations/text_embedding/):

OpenAI▾

[OpenAI](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[Azure](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[Google Gemini](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[Google Vertex](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[AWS](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[HuggingFace](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[Ollama](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[Cohere](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[MistralAI](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[Nomic](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[NVIDIA](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[Voyage AI](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[IBM watsonx](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)
[Fake](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#)

```codeBlockLines_e6Vv
pip install -qU langchain-openai

```

```codeBlockLines_e6Vv
import getpass
import os

if not os.environ.get("OPENAI_API_KEY"):
  os.environ["OPENAI_API_KEY"] = getpass.getpass("Enter API key for OpenAI: ")

from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-large")

```

```codeBlockLines_e6Vv
from langchain_postgres import PGVector

# See docker command above to launch a postgres instance with pgvector enabled.
connection = "postgresql+psycopg://langchain:langchain@localhost:6024/langchain"  # Uses psycopg3!
collection_name = "my_docs"

vector_store = PGVector(
    embeddings=embeddings,
    collection_name=collection_name,
    connection=connection,
    use_jsonb=True,
)

```

## Manage vector store [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#manage-vector-store "Direct link to Manage vector store")

### Add items to vector store [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#add-items-to-vector-store "Direct link to Add items to vector store")

Note that adding documents by ID will over-write any existing documents that match that ID.

```codeBlockLines_e6Vv
from langchain_core.documents import Document

docs = [\
    Document(\
        page_content="there are cats in the pond",\
        metadata={"id": 1, "location": "pond", "topic": "animals"},\
    ),\
    Document(\
        page_content="ducks are also found in the pond",\
        metadata={"id": 2, "location": "pond", "topic": "animals"},\
    ),\
    Document(\
        page_content="fresh apples are available at the market",\
        metadata={"id": 3, "location": "market", "topic": "food"},\
    ),\
    Document(\
        page_content="the market also sells fresh oranges",\
        metadata={"id": 4, "location": "market", "topic": "food"},\
    ),\
    Document(\
        page_content="the new art exhibit is fascinating",\
        metadata={"id": 5, "location": "museum", "topic": "art"},\
    ),\
    Document(\
        page_content="a sculpture exhibit is also at the museum",\
        metadata={"id": 6, "location": "museum", "topic": "art"},\
    ),\
    Document(\
        page_content="a new coffee shop opened on Main Street",\
        metadata={"id": 7, "location": "Main Street", "topic": "food"},\
    ),\
    Document(\
        page_content="the book club meets at the library",\
        metadata={"id": 8, "location": "library", "topic": "reading"},\
    ),\
    Document(\
        page_content="the library hosts a weekly story time for kids",\
        metadata={"id": 9, "location": "library", "topic": "reading"},\
    ),\
    Document(\
        page_content="a cooking class for beginners is offered at the community center",\
        metadata={"id": 10, "location": "community center", "topic": "classes"},\
    ),\
]

vector_store.add_documents(docs, ids=[doc.metadata["id"] for doc in docs])

```

**API Reference:** [Document](https://python.langchain.com/api_reference/core/documents/langchain_core.documents.base.Document.html)

```codeBlockLines_e6Vv
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

```

### Delete items from vector store [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#delete-items-from-vector-store "Direct link to Delete items from vector store")

```codeBlockLines_e6Vv
vector_store.delete(ids=["3"])

```

## Query vector store [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#query-vector-store "Direct link to Query vector store")

Once your vector store has been created and the relevant documents have been added you will most likely wish to query it during the running of your chain or agent.

### Filtering Support [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#filtering-support "Direct link to Filtering Support")

The vectorstore supports a set of filters that can be applied against the metadata fields of the documents.

| Operator | Meaning/Category |
| --- | --- |
| $eq | Equality (==) |
| $ne | Inequality (!=) |
| $lt | Less than (<) |
| $lte | Less than or equal (<=) |
| $gt | Greater than (>) |
| $gte | Greater than or equal (>=) |
| $in | Special Cased (in) |
| $nin | Special Cased (not in) |
| $between | Special Cased (between) |
| $like | Text (like) |
| $ilike | Text (case-insensitive like) |
| $and | Logical (and) |
| $or | Logical (or) |

### Query directly [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#query-directly "Direct link to Query directly")

Performing a simple similarity search can be done as follows:

```codeBlockLines_e6Vv
results = vector_store.similarity_search(
    "kitty", k=10, filter={"id": {"$in": [1, 5, 2, 9]}}
)
for doc in results:
    print(f"* {doc.page_content} [{doc.metadata}]")

```

```codeBlockLines_e6Vv
* there are cats in the pond [{'id': 1, 'topic': 'animals', 'location': 'pond'}]
* the library hosts a weekly story time for kids [{'id': 9, 'topic': 'reading', 'location': 'library'}]
* ducks are also found in the pond [{'id': 2, 'topic': 'animals', 'location': 'pond'}]
* the new art exhibit is fascinating [{'id': 5, 'topic': 'art', 'location': 'museum'}]

```

If you provide a dict with multiple fields, but no operators, the top level will be interpreted as a logical **AND** filter

```codeBlockLines_e6Vv
vector_store.similarity_search(
    "ducks",
    k=10,
    filter={"id": {"$in": [1, 5, 2, 9]}, "location": {"$in": ["pond", "market"]}},
)

```

```codeBlockLines_e6Vv
[Document(metadata={'id': 1, 'topic': 'animals', 'location': 'pond'}, page_content='there are cats in the pond'),\
 Document(metadata={'id': 2, 'topic': 'animals', 'location': 'pond'}, page_content='ducks are also found in the pond')]

```

```codeBlockLines_e6Vv
vector_store.similarity_search(
    "ducks",
    k=10,
    filter={
        "$and": [\
            {"id": {"$in": [1, 5, 2, 9]}},\
            {"location": {"$in": ["pond", "market"]}},\
        ]
    },
)

```

```codeBlockLines_e6Vv
[Document(metadata={'id': 1, 'topic': 'animals', 'location': 'pond'}, page_content='there are cats in the pond'),\
 Document(metadata={'id': 2, 'topic': 'animals', 'location': 'pond'}, page_content='ducks are also found in the pond')]

```

If you want to execute a similarity search and receive the corresponding scores you can run:

```codeBlockLines_e6Vv
results = vector_store.similarity_search_with_score(query="cats", k=1)
for doc, score in results:
    print(f"* [SIM={score:3f}] {doc.page_content} [{doc.metadata}]")

```

```codeBlockLines_e6Vv
* [SIM=0.763449] there are cats in the pond [{'id': 1, 'topic': 'animals', 'location': 'pond'}]

```

For a full list of the different searches you can execute on a `PGVector` vector store, please refer to the [API reference](https://python.langchain.com/api_reference/postgres/vectorstores/langchain_postgres.vectorstores.PGVector.html).

### Query by turning into retriever [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#query-by-turning-into-retriever "Direct link to Query by turning into retriever")

You can also transform the vector store into a retriever for easier usage in your chains.

```codeBlockLines_e6Vv
retriever = vector_store.as_retriever(search_type="mmr", search_kwargs={"k": 1})
retriever.invoke("kitty")

```

```codeBlockLines_e6Vv
[Document(metadata={'id': 1, 'topic': 'animals', 'location': 'pond'}, page_content='there are cats in the pond')]

```

## Usage for retrieval-augmented generation [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#usage-for-retrieval-augmented-generation "Direct link to Usage for retrieval-augmented generation")

For guides on how to use this vector store for retrieval-augmented generation (RAG), see the following sections:

- [Tutorials](https://python.langchain.com/docs/tutorials/)
- [How-to: Question and answer with RAG](https://python.langchain.com/docs/how_to/#qa-with-rag)
- [Retrieval conceptual docs](https://python.langchain.com/docs/concepts/retrieval)

## API reference [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#api-reference "Direct link to API reference")

For detailed documentation of all \_\_ModuleName\_\_VectorStore features and configurations head to the API reference: [https://python.langchain.com/api\_reference/postgres/vectorstores/langchain\_postgres.vectorstores.PGVector.html](https://python.langchain.com/api_reference/postgres/vectorstores/langchain_postgres.vectorstores.PGVector.html)

## Related [​](https://python.langchain.com/docs/integrations/vectorstores/pgvector/\#related "Direct link to Related")

- Vector store [conceptual guide](https://python.langchain.com/docs/concepts/vectorstores/)
- Vector store [how-to guides](https://python.langchain.com/docs/how_to/#vector-stores)

- [Status](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#status)
- [Setup](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#setup)
  - [Credentials](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#credentials)
- [Instantiation](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#instantiation)
- [Manage vector store](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#manage-vector-store)
  - [Add items to vector store](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#add-items-to-vector-store)
  - [Delete items from vector store](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#delete-items-from-vector-store)
- [Query vector store](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#query-vector-store)
  - [Filtering Support](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#filtering-support)
  - [Query directly](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#query-directly)
  - [Query by turning into retriever](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#query-by-turning-into-retriever)
- [Usage for retrieval-augmented generation](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#usage-for-retrieval-augmented-generation)
- [API reference](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#api-reference)
- [Related](https://python.langchain.com/docs/integrations/vectorstores/pgvector/#related)