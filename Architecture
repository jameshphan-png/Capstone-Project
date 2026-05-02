# IT Password Reset Manager - System Architecture

## High-Level Overview

```text
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│                                      USER INTERFACE                                         │
│                         Jupyter / Colab Demo + Gradio Web Interface                         │
│  ┌────────────────────────────────────┐    ┌──────────────────────────────────────────────┐  │
│  │         Helpdesk Chat / FAQ        │    │       Identity & Password Reset Panels      │  │
│  │  • General IT help questions       │    │  • Username / employee verification         │  │
│  │  • Policy and MFA guidance         │    │  • Second-factor validation                 │  │
│  │  • Agent route / audit feedback    │    │  • Temporary password + password change     │  │
│  │  • Trace + MCP tool summaries      │    │  • Ticket / escalation visibility           │  │
│  └────────────────────────────────────┘    └──────────────────────────────────────────────┘  │
└──────────────────────────────────────────────┬───────────────────────────────────────────────┘
                                               │
                                               ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│                                 LANGGRAPH ORCHESTRATION                                     │
│                          Directed state machine over a shared GraphState                    │
└──────────────────────────────────────────────┬───────────────────────────────────────────────┘
                                               │
                                               ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│                                   MULTI-AGENT PIPELINE                                      │
│                                                                                              │
│   ┌────────────────────┐                                                                     │
│   │    INTAKE AGENT    │  Entry point for every request                                      │
│   │                    │  • Classifies top-level intent                                      │
│   │                    │  • Assigns fine-grained label                                       │
│   │                    │  • Returns confidence score                                         │
│   └─────────┬──────────┘                                                                     │
│             │                                                                                │
│   ┌─────────┴───────────────────────────────┬───────────────────────────────┐               │
│   │                                         │                               │               │
│   ▼                                         ▼                               ▼               │
│ ┌────────────────────┐           ┌────────────────────┐          ┌──────────────────────┐   │
│ │  KNOWLEDGE AGENT   │           │   WORKFLOW AGENT   │          │   ESCALATION AGENT   │   │
│ ├────────────────────┤           ├────────────────────┤          ├──────────────────────┤   │
│ │ • RAG retrieval    │           │ • Executes MCP     │          │ • Final fallback for  │   │
│ │ • Policy Q&A       │           │   automation       │          │   failed / unclear /  │   │
│ │ • MFA guidance     │           │ • Runs log analysis│          │   out-of-scope cases  │   │
│ │ • Grounded answers │           │ • Issues temp pwd  │          │ • Creates ticket ID   │   │
│ │ • Source-backed    │           │ • Unlocks account  │          │ • Returns IT handoff  │   │
│ │   responses        │           │ • Flags escalation │          │   response            │   │
│ └─────────┬──────────┘           └─────────┬──────────┘          └──────────┬───────────┘   │
│           │                                │                                │               │
└───────────┼────────────────────────────────┼────────────────────────────────┼───────────────┘
            │                                │                                │
            ▼                                ▼                                ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│                                   DATA, TOOLS & SERVICES                                    │
│                                                                                              │
│  ┌────────────────────────┐  ┌────────────────────────┐  ┌────────────────────────────────┐ │
│  │   Policy Knowledge     │  │      MCP Tool Layer    │  │          LLM / Retrieval       │ │
│  │   Base (.md / .pdf)    │  │                        │  │                                │ │
│  ├────────────────────────┤  ├────────────────────────┤  ├────────────────────────────────┤ │
│  │ • password_reset_policy│  │ • reset_password       │  │ • OpenAI GPT-4o-mini           │ │
│  │ • account_lockout      │  │ • unlock_account       │  │   + Chroma (primary)           │ │
│  │ • mfa_help_guide       │  │ • create_ticket        │  │ • llm7.io GPT-4o compatible    │ │
│  │ • chunked into context │  │ • log_analysis         │  │   + BM25 (backup)              │ │
│  │ • top-k retrieval      │  │ • audited tool calls   │  │ • BM25 offline fallback        │ │
│  └────────────────────────┘  └────────────────────────┘  └────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Core Architectural Idea

The system is a **LangGraph-managed multi-agent IT help-desk workflow** for password-reset and account-access support. Every user request enters through a single **Intake Agent**, which writes a structured `Intent` object into a shared `GraphState`. LangGraph then routes the request to the correct specialist agent:

- **Knowledge Agent** for policy and MFA guidance questions.
- **Workflow Agent** for automatable IT actions such as password reset and account unlock.
- **Escalation Agent** for failed automation, unclear requests, or out-of-scope issues.

This design separates **decision-making**, **knowledge retrieval**, **tool execution**, and **human handoff**, which keeps each agent narrow in responsibility and easier to debug, explain, and extend.

## Agent Roles

### 1) Intake Agent

**Purpose:** Act as the routing and classification gate for every request.

**Responsibilities**
- Read the user query and optional session flags such as `forgot_password_selected`.
- Classify the request into a top-level route:
  - `knowledge`
  - `workflow`
  - `escalation`
- Assign a fine-grained label:
  - `forgot_password`
  - `account_locked`
  - `mfa_help`
  - `policy_question`
  - `escalate`
- Produce a confidence score.
- Append the routing decision to the execution trace.

**Why this role exists**
- Keeps downstream agents focused on execution rather than interpretation.
- Prevents all agents from having to re-parse the user’s intent.
- Makes the workflow explainable because routing is explicit in `GraphState.intent`.

**Typical routing rules**
- Forgot or reset password → `workflow / forgot_password`
- Account locked and needs action → `workflow / account_locked`
- MFA or lost device questions → `knowledge / mfa_help`
- Policy / rule / process questions → `knowledge / policy_question`
- Unclear or unsupported requests → `escalation / escalate`

---

### 2) Knowledge Agent

**Purpose:** Answer policy and guidance questions using retrieved policy context.

**Responsibilities**
- Query the retriever for the most relevant policy chunks.
- Convert retrieved chunks into structured `RetrievedDoc` records.
- Generate a grounded answer using:
  - OpenAI + prompt context when available, or
  - a deterministic BM25 fallback answer path.
- Return the answer in `final_response`.
- Store the retrieved evidence in `retrieved_context`.
- Append retrieval activity to the trace.

**What it should handle**
- Temporary password usage rules.
- Account lockout policy questions.
- MFA reset / lost device guidance.
- Password-policy or help-desk process questions.

**Why this role exists**
- Keeps policy interpretation isolated from operational actions.
- Reduces hallucination by grounding answers in policy documents.
- Makes the assistant useful even when no tool execution is needed.

---

### 3) Workflow Agent

**Purpose:** Perform operational IT actions through MCP tools.

**Responsibilities**
- Handle automatable support intents only.
- Always collect supporting audit information through `log_analysis` for workflow cases.
- For `forgot_password`:
  - retrieve relevant reset-policy context,
  - call `reset_password`,
  - store the issued temporary password,
  - return a successful reset response.
- For `account_locked`:
  - retrieve lockout-policy context,
  - call `unlock_account`,
  - mark access restored,
  - return an unlock confirmation.
- When a tool fails or the request is unsupported:
  - set `escalation_needed = True`,
  - record `escalation_reason`,
  - allow LangGraph to hand off to the Escalation Agent.

**Why this role exists**
- Separates secure operational actions from general knowledge responses.
- Centralises MCP tool usage and audit logging.
- Creates a clean boundary between automation success and human handoff.

**Tools used by this role**
- `log_analysis`
- `reset_password`
- `unlock_account`

---

### 4) Escalation Agent

**Purpose:** Serve as the final safety net when automation should not continue.

**Responsibilities**
- Receive cases that are:
  - unclear,
  - out of scope,
  - or failed during automated workflow execution.
- Generate a ticket identifier for follow-up.
- Return a handoff response indicating IT support escalation.
- Record escalation details in the trace.

**Why this role exists**
- Ensures there is always a safe terminal path.
- Prevents silent workflow failure.
- Preserves a realistic service-desk model with human support continuity.

**Typical escalation triggers**
- Password reset service failure.
- Account unlock service failure.
- Vague requests that cannot be classified confidently.
- Issues unrelated to the supported password / account-access scope.

## Shared State Structure

The agents collaborate through a typed shared object called `GraphState`.

```text
GraphState
├── user_query
├── user_id
├── forgot_password_selected
├── intent
│   ├── top_level
│   ├── label
│   └── confidence
├── retrieved_context[]
├── mcp_calls[]
├── temp_password_issued
├── login_access_restored
├── escalation_needed
├── escalation_reason
├── ticket_id
├── final_response
└── trace[]
```

### Why the shared state matters
- It acts as a **single source of truth** across all nodes.
- It preserves auditability by storing intent, retrieved context, tool calls, and trace history.
- It supports deterministic routing after each node completes.
- It allows one agent to hand off structured work to another without re-deriving context.

## Request Flow

```text
User Request
     │
     ▼
┌───────────────┐
│ Intake Agent  │
│ classify      │
│ + confidence  │
└───────┬───────┘
        │
        ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ route_from_intake(state.intent.top_level)                                │
└───────────────┬─────────────────────────────┬─────────────────────────────┘
                │                             │
                │                             │
                ▼                             ▼
      ┌──────────────────┐          ┌──────────────────┐
      │ Knowledge Agent  │          │ Workflow Agent   │
      │ 1. Retrieve docs │          │ 1. log_analysis  │
      │ 2. Build context │          │ 2. reset/unlock  │
      │ 3. Generate      │          │ 3. success?      │
      │    grounded      │          │ 4. else escalate │
      │    answer        │          └────────┬─────────┘
      └────────┬─────────┘                   │
               │                             │
               └──────────────┬──────────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │ Escalation Agent   │
                    │ create ticket /    │
                    │ return handoff     │
                    └─────────┬──────────┘
                              │
                              ▼
                         Final Response
```

## LangGraph Routing Logic

```text
[START]
   │
[intake]
   │
   ├── if intent.top_level == knowledge  ─────────▶ [knowledge]  ─▶ [END]
   ├── if intent.top_level == workflow   ─────────▶ [workflow]
   │                                              │
   │                                              ├── if escalation_needed == false ─▶ [END]
   │                                              └── if escalation_needed == true  ─▶ [escalation] ─▶ [END]
   └── if intent.top_level == escalation ────────▶ [escalation] ─▶ [END]
```

## Data and Service Flow

### Knowledge Path
```text
Policy Documents
   │
   ▼
Text Loader / PDF Loader
   │
   ▼
Text Splitter (chunk_size=500, overlap=50)
   │
   ├── OpenAI mode  ─▶ Embeddings ─▶ Chroma Vector Store ─▶ Retriever (top-k)
   └── Fallback     ─▶ BM25 Retriever
                                   │
                                   ▼
                           Knowledge Agent
```

### Workflow Path
```text
Workflow Agent
   │
   ├── log_analysis(user_id, log_type="login_failures")
   ├── reset_password(user_id)     for forgot_password
   ├── unlock_account(user_id)     for account_locked
   └── create escalation flags     when automation fails
```

### Escalation Path
```text
Escalation Agent
   │
   ├── create ticket ID
   ├── persist escalation outcome in state
   └── return handoff message to the user
```

## MCP Tool Layer

The MCP layer abstracts backend IT services behind standard tool interfaces.

| Tool | Purpose | Used By |
|------|---------|---------|
| `reset_password` | Issue a temporary password | Workflow Agent |
| `unlock_account` | Restore access to a locked account | Workflow Agent |
| `create_ticket` | Open an IT support ticket | Architectural support primitive; escalation-related service |
| `log_analysis` | Retrieve audit / security log entries | Workflow Agent |

## Security and Operational Guardrails

```text
User requests password reset
        │
        ▼
Identity verification required in UI
        │
        ▼
Verified session may continue
        │
        ▼
Workflow Agent can issue temporary password
        │
        ▼
User must change password immediately after sign-in
```

### Important guardrails reflected in the notebook
- Password-reset actions are gated by identity verification in the UI flow.
- Temporary passwords are single-use and should be changed immediately.
- Unsupported or ambiguous requests do not stay in automation; they are escalated.
- Workflow actions leave an MCP audit trail for transparency.

## Why This Structure Works Well

- **Modular:** each agent has one main responsibility.
- **Explainable:** routing and execution are visible through `intent`, `trace`, and `mcp_calls`.
- **Safe:** failures fall back to escalation instead of producing uncertain automation.
- **Extensible:** more intents, tools, and specialist agents can be added without redesigning the whole graph.
- **Demo-friendly:** the same backend supports both terminal and Gradio interfaces.

## Suggested Future Extensions

```text
Current: Intake → Knowledge / Workflow / Escalation
Future : Intake → Security / Device Support / Access Mgmt / Reporting / Escalation
```

Possible additions:
- Dedicated **Security Agent** for suspicious-login and MFA anomaly cases.
- Dedicated **Device Support Agent** for laptop / printer / endpoint issues.
- Persistent ticket storage instead of mock ticket IDs.
- Real identity provider and directory integrations.
- Real notification or queueing for IT support handoff.
