# RAG: Chat with Claude FAQs
### An AI-Powered Knowledge Base Assistant for Data Engineering

> Built by **Sri Adidam** · Learner Apps Portfolio · March 2026

---

## What This Does

This project is a working **Retrieval-Augmented Generation (RAG)** application that lets you chat with a curated knowledge base — in this case, a custom FAQ document about using Claude for Data Engineering. Ask it anything about pipelines, SQL, Python automation, or best practices, and it retrieves the most relevant content and generates a grounded, accurate response.

This was built as a learning project to understand how RAG systems work end-to-end — from document ingestion and vector embeddings to semantic search and AI-generated answers.

---

## How It Works

The project is split into two N8N workflows that work together:

### Workflow 1 — Load: Ingest the Knowledge Base
Uploads and indexes the FAQ document into a vector database so it can be searched semantically.

![Load RAG Workflow](Load_RAG_with_Claude_FAQs.png)

**What happens in this workflow:**
- A PDF document is submitted via a form trigger
- The document is parsed and chunked by a Default Data Loader
- OpenAI generates vector embeddings for each chunk (1536-dimension vectors)
- All chunks and embeddings are stored in a Supabase vector database

---

### Workflow 2 — Chat: Ask Questions
A conversational interface that retrieves relevant chunks and generates intelligent answers.

![Chat RAG Workflow](RAG-Chat_with_Claude_FAQs.png)

**What happens in this workflow:**
- User types a question in the chat interface
- The AI Agent receives the message and decides to query the knowledge base
- The Supabase Vector Store performs a semantic similarity search
- OpenAI embeddings power the search query
- The AI Agent synthesizes the retrieved content into a clear, conversational answer

---

## Tech Stack

| Layer | Tool |
|---|---|
| Workflow Automation | N8N (self-hosted) |
| Vector Database | Supabase (pgvector extension) |
| Embeddings | OpenAI text-embedding-3-small (1536 dimensions) |
| AI Agent & Chat | OpenAI GPT-4 via N8N AI Agent node |
| Document Parsing | N8N Default Data Loader |
| Knowledge Base | Custom PDF — Claude for Data Engineering FAQ |

---

## The Knowledge Base Document

The FAQ document powering this RAG was purpose-built for this project. It contains **15 questions and answers** across 8 topic areas specifically designed for data engineers:

- Getting Started with Claude
- Pipeline Design & Architecture
- SQL & Data Modeling
- Python & Automation
- Orchestration & DevOps
- Data Quality & Observability
- Latest Trends
- Best Practices

📄 **[Download the FAQ PDF](Claude_for_Data_Engineering_FAQ.pdf)**

---

## Sample Interaction

**User:** How does Claude help with real time streaming pipelines?

**Assistant:** Below is a concise summary of how Claude can help with building real-time streaming pipelines, drawn from the FAQ document.

**What Claude can do:**
- Design event-driven pipelines using Kafka, Apache Flink, Spark Structured Streaming, or cloud services (AWS Kinesis, Google Pub/Sub)
- Write consumer group configurations, windowing logic, and watermark strategies for late data
- Advise on exactly-once delivery guarantees and checkpointing intervals

---

## Reference

- Supabase + LangChain vector store integration: https://supabase.com/docs/guides/ai/langchain?database-method=sql
- N8N AI Agent documentation: https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.agent/

---

## About

Part of the **Learner Apps** portfolio — a collection of hands-on AI and automation projects built for learning and exploration.

**GitHub:** [semantic-sage/Learner-Apps](https://github.com/semantic-sage/Learner-Apps)
