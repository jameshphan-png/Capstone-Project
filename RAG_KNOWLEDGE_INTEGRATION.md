# RAG Knowledge Management Integration

## Overview

This document explains how **embeddings, vector databases, and retrieval are used for context grounding** in the **Capstone Project** implementation found in `Password Reset Agent.ipynb`.

Repository: https://github.com/jameshphan-png/Capstone-Project

This project implements an **IT Password Reset Manager** using:

- **RAG** for policy-grounded answers
- **LangGraph** for multi-agent orchestration
- **MCP (Model Context Protocol)** for tool execution
- **OpenAI + Chroma** as the primary semantic retrieval path
- **BM25** as the fallback retrieval path for low-cost or offline-friendly use

The knowledge layer is designed to ground responses in real IT policy documents such as:

- `account_lockout_policy.md`
- `mfa_help_guide.md`
- `password_reset_policy.md`

---

## 1. How RAG is Implemented in This Project

In this notebook, RAG is used to support the **Knowledge Agent** and parts of the **Workflow Agent** by retrieving relevant policy content before generating or rendering a response.

The retrieval pipeline in the code follows this sequence:

1. Load policy documents from the policy directory
2. Split documents into smaller chunks
3. Build a retriever over those chunks
4. Query the retriever with the user's request
5. Return the top matching chunks
6. Use those chunks as grounding context for the agent response

This is implemented around:

- `load_policy_documents(...)`
- `RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)`
- `Chroma.from_documents(...)` when `USE_OPENAI = True`
- `BM25Retriever.from_documents(...)` when OpenAI is not enabled
- `_retrieve_policy_docs(query: str)` inside the Knowledge Agent

---

## 2. Document Loading and Chunking

The notebook first loads policy files from disk using LangChain document loaders:

- `TextLoader` for `.md` and `.txt`
- `PyPDFLoader` for `.pdf`

Each loaded document is enriched with metadata such as:

- `title`
- `source`

After loading, the documents are chunked using:

```python
RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
```

### Why this matters

Chunking improves retrieval quality because the system does not search entire documents at once. Instead, it searches smaller, more focused pieces of text. The overlap of 50 characters helps preserve context between adjacent chunks, which reduces the chance of losing important information at chunk boundaries.

---

## 3. Embeddings in This Project

Embeddings are used in the **primary production retrieval path** when `USE_OPENAI = True`.

In that mode, the notebook imports:

```python
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
```

The role of embeddings here is to convert each text chunk into a dense numerical vector so that semantically similar content can be matched even when the wording is different.

### How embeddings are used

- Each chunked policy document is transformed into an embedding
- A user query is also transformed into an embedding at retrieval time
- The system compares the query vector against stored document vectors
- The most semantically similar chunks are returned

### Why embeddings help

This allows the system to retrieve meaning-based matches rather than only exact keyword matches. For example, a user might ask about being locked out of an account without using the exact phrasing found in the policy document. Embeddings help bridge that wording gap.

### Important implementation note

Your notebook is intentionally built with two modes:

- **OpenAI mode** → semantic retrieval using embeddings + Chroma
- **Fallback mode** → lexical retrieval using BM25 without embeddings

So embeddings are part of the main architecture, but the system is resilient even when embeddings are unavailable.

---

## 4. Vector Database in This Project

The vector database used in the primary semantic path is **Chroma**.

This is created in the notebook with code equivalent to:

```python
vectorstore = Chroma.from_documents(
    documents=split_policy_docs,
    embedding=embeddings,
    persist_directory=CHROMA_DIR,
    collection_name="it_policy_knowledge_base",
)
```

### What Chroma stores

Chroma stores:

- the embedding vector for each chunk
- the raw text content of each chunk
- metadata such as source file and title

### Why Chroma is important

Chroma acts as the semantic memory layer for the policy knowledge base. Instead of manually scanning files, the system can search the vector store for the closest-matching chunks in milliseconds.

### Retrieval configuration

The retriever is created as:

```python
kb_retriever = vectorstore.as_retriever(search_kwargs={"k": 3})
```

This means the system retrieves the **top 3 matching chunks** for a query.

---

## 5. Retrieval in This Project

Retrieval is handled through a shared knowledge retriever called `kb_retriever`.

The notebook supports two retrieval strategies:

### A. Semantic retrieval with Chroma
Used when `USE_OPENAI = True`

- document chunks are embedded
- query is embedded
- Chroma retrieves the nearest vectors
- top 3 chunks are returned

### B. Lexical retrieval with BM25
Used when OpenAI is disabled

```python
kb_retriever = BM25Retriever.from_documents(split_policy_docs)
kb_retriever.k = 3
```

This path does not use embeddings or a vector database. Instead, it ranks chunks using keyword relevance. This is useful for:

- development
- demos
- lower-cost execution
- fallback when API-based embeddings are unavailable

### Why this dual approach is strong

This design gives the project both:

- **semantic power** in production mode
- **practical robustness** in low-cost or offline-friendly mode

---

## 6. Context Grounding in the Knowledge Agent

The clearest example of context grounding appears in the **Knowledge Agent**.

The function:

```python
def _retrieve_policy_docs(query: str) -> List[RetrievedDoc]:
```

invokes the retriever and converts retrieved LangChain documents into structured `RetrievedDoc` objects containing:

- `title`
- `snippet`
- `source`

These retrieved chunks are then used as grounded context for the response.

### In OpenAI mode
The retrieved policy chunks are intended to support an LLM-generated answer grounded in policy text.

### In BM25 mode
The notebook uses `_render_bm25_answer(...)` to produce a deterministic answer based on the retrieved snippets, without needing an LLM.

This is an important design choice: even when the full semantic + LLM path is unavailable, the system still grounds answers in retrieved policy text rather than inventing answers.

---

## 7. Context Grounding in the Workflow Agent

RAG is not only used for direct policy Q&A. It also supports workflow execution.

In the **Workflow Agent**, the code retrieves relevant policy context before taking actions for intents such as:

- `forgot_password`
- `account_locked`

This means the system does not treat workflow automation as completely separate from knowledge retrieval. Instead, policy retrieval helps ensure that operational actions remain aligned with documented IT rules.

For example, before or during a password reset or account unlock flow, the system can reference the correct policy context for:

- lockout thresholds
- temporary password handling
- MFA guidance
- escalation conditions

This improves consistency between what the system says and what it does.

---

## 8. Retrieved Context as Shared Agent State

The project stores retrieved knowledge in the LangGraph shared state model:

```python
retrieved_context: List[RetrievedDoc]
```

inside `GraphState`.

This is significant because retrieval is not treated as an isolated step. The retrieved context becomes part of the shared blackboard used by the multi-agent system.

That means:

- the Knowledge Agent can answer with grounded context
- the Workflow Agent can combine policy retrieval with tool execution
- the Escalation Agent can inherit a richer case context if needed

This is a strong knowledge management pattern because retrieved information becomes reusable state across the workflow.

---

## 9. Why This Counts as Knowledge Management Integration

This project is not just using RAG as a search feature. It is integrating retrieval into a broader **knowledge management architecture**.

### Knowledge sources
The policy files act as the managed knowledge base.

### Knowledge transformation
The raw documents are loaded, normalized, chunked, and enriched with metadata.

### Knowledge indexing
In OpenAI mode, chunks are embedded and indexed in Chroma.

### Knowledge access
Agents query the retriever to access the most relevant policy knowledge.

### Knowledge application
The system uses retrieved knowledge to:

- answer user questions
- guide password reset workflows
- support account lockout handling
- reduce hallucinations
- improve auditability and consistency

This makes the RAG layer a core part of operational decision support, not just an optional add-on.

---

## 10. End-to-End Flow in Your Notebook

Here is the project’s actual RAG flow in practical terms:

1. Policy files are loaded from the local policy directory
2. Files are parsed using `TextLoader` or `PyPDFLoader`
3. Documents are split into 500-character chunks with 50-character overlap
4. If OpenAI mode is enabled:
   - `OpenAIEmbeddings` converts chunks into dense vectors
   - `Chroma` stores vectors, raw text, and metadata
5. If OpenAI mode is disabled:
   - `BM25Retriever` indexes the chunked documents lexically
6. A user query is passed into `kb_retriever`
7. The retriever returns the top 3 relevant chunks
8. `_retrieve_policy_docs(...)` converts results into structured `RetrievedDoc` records
9. The retrieved context is stored in `GraphState`
10. The Knowledge Agent or Workflow Agent uses that grounded context to produce a response or support an action

---

## 11. Why This Design Is Effective

This implementation is effective because it balances **capability, cost, and reliability**.

### Strengths of the design

- Uses real policy documents as the grounding source
- Supports both semantic and lexical retrieval
- Keeps retrieved context in structured shared state
- Connects knowledge retrieval to multi-agent orchestration
- Improves consistency between answers and IT policy
- Avoids total dependency on one provider or one retrieval method

### Practical benefit

For an IT help-desk use case, this means the assistant can provide answers that are:

- more accurate
- more policy-compliant
- more explainable
- more useful for operational workflows

---

## Conclusion

In this Capstone Project, embeddings, vector databases, and retrieval are used as the knowledge grounding layer for an IT password reset assistant.

Specifically:

- **Embeddings** are used in the OpenAI-enabled path to represent policy chunks and user queries semantically
- **Chroma** serves as the vector database that stores and retrieves those semantic representations
- **BM25** provides a strong fallback retrieval path when embeddings are unavailable
- **Retrieval** returns the most relevant policy chunks
- **Context grounding** happens when those retrieved chunks are injected into the agent workflow and used to produce policy-aligned answers and actions

Overall, the notebook demonstrates a well-structured **RAG Knowledge Management Integration** by connecting document ingestion, chunking, retrieval, agent state, and workflow automation into one grounded support system.
