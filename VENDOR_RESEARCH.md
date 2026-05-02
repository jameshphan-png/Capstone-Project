# 🔍 Industry Vendor Research — Password Reset Agent

## 📌 Overview

This document provides an in-depth industry vendor analysis for the **IT Help Desk Password Reset Agent**, a system designed to automate secure password reset workflows using:

- Retrieval-Augmented Generation (RAG)
- Multi-agent orchestration (LangGraph)
- LLM-based reasoning
- Policy-aware decision making

In addition to evaluating core technology vendors, this research includes **real-world enterprise implementations** that closely mirror this system’s architecture.

---

## 🧠 System Context

The Password Reset Agent is designed to:

- Verify user identity
- Retrieve IT policies dynamically
- Guide users through password reset workflows
- Execute or simulate reset actions securely

### Core Architecture

| Layer | Function |
|------|--------|
| LLM | Conversational reasoning |
| RAG | Policy retrieval |
| Orchestration | Workflow control |
| Tools | Authentication + reset logic |
| Memory | Context persistence |

---

# 🏢 Vendor Landscape

---

## 1️⃣ Large Language Model (LLM) Providers

### 🔹 OpenAI

**Role:** Primary reasoning engine

📖 OpenAI models demonstrate strong reasoning and instruction-following capabilities suitable for enterprise agents [1].

---

### 🔹 llm7.io (Fallback LLM)

**Role:** Backup LLM provider

---

## 2️⃣ Orchestration Frameworks

### 🔹 LangGraph

📖 Graph-based orchestration improves reliability in multi-step AI workflows [2].

---

### 🔹 LangChain

📖 LangChain enables composable LLM applications with external integrations [3].

---

## 3️⃣ Retrieval Systems (RAG)

### 🔹 BM25

📖 BM25 remains a strong baseline for efficient information retrieval [4].

---

### 🔹 Chroma (Vector DB)

📖 Vector databases power semantic retrieval in RAG systems [5].

---

# 🏢 Real-World Industry Vendor Examples

---

# 🔐 1. Okta — Identity & Password Reset Automation

📖 Okta delivers secure identity management and adaptive authentication [6].

📖 SSPR reduces IT workload while maintaining security [7].

---

# 🏢 2. ServiceNow — IT Help Desk Automation

📖 ServiceNow enables automated IT workflows and AI-powered support [8].

📖 Virtual agents reduce support costs and improve efficiency [9].

---

# 🔗 Architecture

User → LLM → LangGraph → Tools → RAG → Policies  
                     ↓  
                Okta (Identity)  
                     ↓  
             ServiceNow (ITSM)

---

# 📚 References

[1] https://platform.openai.com/docs  
[2] https://docs.langchain.com  
[3] https://www.langchain.com  
[4] Robertson & Zaragoza (2009)  
[5] Lewis et al. (2020)  
[6] https://www.okta.com  
[7] https://help.okta.com  
[8] https://www.servicenow.com  
[9] https://www.servicenow.com/products/virtual-agent.html  
