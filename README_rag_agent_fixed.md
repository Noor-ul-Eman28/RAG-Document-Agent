# Document RAG Assistant 📚🔍

An automated n8n workflow that builds a searchable knowledge base from documents in Google Drive and lets users ask questions about them through a chat interface, using Retrieval-Augmented Generation (RAG).

## Problem Statement

Finding specific information across a growing collection of documents is time-consuming. This workflow automatically ingests any new document dropped into a Google Drive folder, converts it into a searchable vector database, and lets users query it in natural language — with answers grounded strictly in the uploaded content.

## How It Works

### 1. Document Ingestion

- **Google Drive Trigger** watches a specific folder and fires whenever a new file is added.
- **Download file** retrieves the new document.
- The document is split into chunks using a **Recursive Character Text Splitter** (200-character overlap) via the **Default Data Loader**, which also attaches metadata (file name, file ID, timestamp).
- **Embeddings (OpenAI)** converts each chunk into a vector representation.
- The chunks and their embeddings are stored in an **In-Memory Vector Store**, building the knowledge base automatically as new files arrive.

### 2. Chat-Based Retrieval

- **Chat Trigger** listens for incoming user questions.
- The **AI Agent** (GPT-4o) is configured as a Document Q&A Assistant with strict grounding rules:
  - It must search the knowledge base before answering any content question
  - It only states facts found in the retrieved passages — no answers from general knowledge
  - If the answer isn't in the documents, it replies that it couldn't be found, rather than guessing
  - Every answer cites the source file it came from
- The **Vector Store (retrieve-as-tool)** is exposed to the AI Agent as a semantic search tool, letting the agent decide when to search and with what query.
- **Simple Memory** (buffer window, last 10 messages) gives the agent short-term conversational context, so follow-up questions can be understood correctly.

## Key Design Choices

- **Grounded answers only** — the agent is explicitly instructed to avoid hallucination and to say when information isn't available, rather than filling gaps with assumptions.
- **Automatic re-indexing** — any file added to the watched Drive folder is ingested without manual steps.
- **Tool-based retrieval** — the vector store is wired in as a callable tool rather than a fixed pipeline step, so the agent only searches when it decides a question needs it.

## Technologies Used

- n8n (workflow automation)
- Google Drive API (trigger + file storage)
- OpenAI GPT-4o
- OpenAI Embeddings
- LangChain (Vector Store, Text Splitter, Document Loader, Buffer Memory, Agent)

## Workflow Diagram

![Workflow Diagram](workflow-diagram.png)

## Files in This Repo

- `RAG_agent.json` — full exported n8n workflow, importable directly into any n8n instance
- `workflow-diagram.png` — visual overview of the automation

## Results

This project helped me practice building a Retrieval-Augmented Generation pipeline from scratch: document ingestion, chunking, embeddings, vector search, and grounding an AI agent's responses strictly in retrieved content rather than its own general knowledge.
