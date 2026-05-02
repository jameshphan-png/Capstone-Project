# SCALING.md

## 📈 Scaling the IT Password Reset Manager

This document outlines how the **IT Help-Desk Password Reset Manager** scales from a local notebook prototype to a production-grade, enterprise-ready system.

The system is designed as a **multi-agent architecture** using:

- **LangGraph** for orchestration  
- **RAG (BM25 / Chroma)** for policy grounding  
- **MCP tools** for system actions (e.g., password reset)  
- **LLMs (OpenAI / fallback API)** for reasoning  

---

## 🧱 Current Architecture (Prototype)

```
User Input
   ↓
Intake Agent
   ↓
Knowledge Agent (RAG lookup)
   ↓
Workflow Agent (decision logic)
   ↓
MCP Tool (password reset simulation)
   ↓
Escalation Agent (fallback / human routing)
```

### Key Characteristics

- Single-user, synchronous execution  
- In-memory state (LangGraph StateGraph)  
- Local BM25 retrieval or optional Chroma  
- Mocked identity + password store  
- No persistence or concurrency handling  

---

## ⚠️ Scaling Challenges

### 1. State Management
- Ephemeral in-memory state  
- No session persistence  

### 2. Concurrency
- Single-threaded execution  
- No support for multiple users  

### 3. Tool Execution
- MCP tools are simulated  
- No real IAM integrations  

### 4. Retrieval Performance
- BM25 limited for large-scale corpora  
- No real-time indexing  

### 5. Security Constraints
- Hardcoded credentials  
- No authentication or audit logs  

---

## 🏗️ Target Scalable Architecture

```
                ┌────────────────────┐
                │   API Gateway      │
                └────────┬───────────┘
                         │
                ┌────────▼─────────┐
                │  Orchestration   │
                │ (LangGraph API)  │
                └────────┬─────────┘
                         │
     ┌────────────┬──────┼────────────┬────────────┐
     ▼            ▼      ▼            ▼            ▼
 Intake       Knowledge  Workflow  Escalation   Tool Layer
 Agent        Agent      Agent     Agent        (MCP APIs)
     │            │         │          │             │
     └────────────┴─────────┴──────────┴─────────────┘
                         │
                ┌────────▼─────────┐
                │ External Systems │
                │ (IAM, MFA, DBs)  │
                └──────────────────┘
```

---

## 🔄 Scaling Strategy

### 1. Stateless API Layer
- FastAPI / Flask  
- Stateless request handling  

### 2. Session & State Persistence

| Component | Technology |
|----------|-----------|
| Session State | Redis |
| Conversations | PostgreSQL / MongoDB |
| Audit Logs | ELK / Datadog |

---

### 3. Agent Execution Scaling
- Async workers (Celery)  
- Kafka / RabbitMQ  

---

### 4. Retrieval Scaling (RAG)
- Pinecone / Weaviate  
- Hybrid search  

---

### 5. Tooling (MCP → Real Systems)
- Active Directory / Okta  
- Duo / Auth0  

---

### 6. LLM Scaling Strategy
- Primary + fallback models  
- Redis caching  

---

### 7. Security & Compliance
- SSO / OAuth  
- MFA verification  
- Audit logging  

---

### 8. Observability
- ELK, Prometheus, OpenTelemetry  

---

## 📊 Load Scaling Scenarios

### Small Team (≤100 users)
- Single API  

### Mid-size Org (1K–10K users)
- Load-balanced APIs  

### Enterprise (50K+ users)
- Multi-region  

---

## 🧾 Summary

| Prototype | Production |
|----------|-----------|
| Notebook | API Service |
| In-memory | Persistent storage |
| Mock tools | Real IAM integrations |
| Single-user | Concurrent users |
| Local RAG | Distributed retrieval |

This system evolves into a **fully automated enterprise-grade IT password reset assistant**.
