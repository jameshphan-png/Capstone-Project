# TESTING.md

## 📌 Overview

This document outlines the testing strategy for the **IT Password Reset Manager** system. The application is a multi-agent, RAG-powered help desk assistant designed to handle password reset requests, enforce IT policies, and escalate issues when necessary.

The system integrates:
- **RAG (BM25 / Chroma)** for policy-aware responses
- **LangGraph multi-agent workflows**
- **MCP-based tool integrations**

---

## 🧪 Testing Approach

Testing is divided into the following categories:

| Test Type | Purpose |
|----------|--------|
| Unit Testing | Validate individual components (retrieval, parsing, tools) |
| Integration Testing | Ensure agents communicate correctly |
| End-to-End Testing | Validate full password reset workflow |
| Negative Testing | Handle invalid or malicious inputs |

---

## 🔁 Core Workflow to Test

1. User submits password reset request
2. Intake agent captures request details
3. Knowledge agent retrieves policy
4. Workflow agent determines action
5. System performs reset OR escalates

---

## ✅ Test Scenarios

### 1. Successful Password Reset
**Input:**
- "I forgot my password"

**Expected Behavior:**
- System verifies identity (mocked)
- Retrieves password policy
- Generates reset instructions
- Returns success response

---

### 2. Account Lockout Case
**Input:**
- "My account is locked"

**Expected Behavior:**
- Retrieve lockout policy
- Suggest unlock steps or wait time
- Avoid direct password reset if policy restricts it

---

### 3. MFA Issue
**Input:**
- "I can’t access my MFA"

**Expected Behavior:**
- Retrieve MFA help guide
- Provide recovery steps
- Escalate if needed

---

### 4. Escalation Scenario
**Input:**
- "Reset password for another employee"

**Expected Behavior:**
- Detect policy violation
- Trigger escalation agent
- Provide denial + escalation message

---

### 5. Invalid Input
**Input:**
- "asdf123!!"

**Expected Behavior:**
- Ask for clarification
- Do not proceed with workflow

---

## 🔍 Retrieval Testing (RAG)

| Test | Expected Result |
|------|----------------|
| Query matches policy | Correct document retrieved |
| No match | Fallback or clarification |
| Multiple matches | Best-ranked response returned |

---

## 🤖 Agent Behavior Testing

| Agent | What to Validate |
|------|------------------|
| Intake Agent | Correct intent classification |
| Knowledge Agent | Accurate document retrieval |
| Workflow Agent | Proper decision making |
| Escalation Agent | Triggered when required |

---

## ⚠️ Edge Cases

- Missing policy documents
- LLM unavailable (fallback to BM25)
- Ambiguous user intent
- Repeated requests (idempotency)

---

## 🔌 Tool / MCP Testing

- Validate tool invocation format
- Ensure correct inputs/outputs
- Simulate failures (timeouts, bad responses)

---

## 📊 Expected Outputs

- Structured response
- Policy-grounded reasoning
- Clear instructions or escalation

---

## 🚀 How to Run Tests

1. Open the notebook (`Phase_4_5_6_Work.ipynb`)
2. Run all cells
3. Execute test prompts manually OR via scripted inputs
4. Observe:
   - Agent transitions
   - Retrieval outputs
   - Final responses

---

## 📊 Metrics & Evaluation

The system should track and report the following performance metrics:

| Metric | Description |
|-------|------------|
| Accuracy | Correctness of responses based on policy alignment |
| Latency | Time taken to process a request end-to-end |
| Retrieval Precision | Relevance of retrieved documents |
| User Satisfaction | Feedback score (simulated or real) |

### Measurement Approach

- **Accuracy:** Compare system responses against expected outputs in test scenarios
- **Latency:** Measure response time per request (ms)
- **Retrieval Quality:** Evaluate top-k document relevance
- **Satisfaction:** Simulated rating (e.g., 👍 / 👎 or 1–5 scale)

---

## 🎭 Realistic Scenario Testing

To ensure production readiness, include real-world simulations:

- Employee locked out during peak hours
- MFA reset while traveling (device unavailable)
- Suspicious request (possible social engineering)
- Repeated failed login attempts
- VIP/exec escalation handling

Each scenario should validate:
- Policy adherence
- Correct agent routing
- Clear communication
- Proper escalation when required

---

## 🧩 Future Testing Improvements

- Automated test suite (pytest)
- Mock MCP server
- Continuous metric tracking dashboard
- Prompt regression testing

---

## 📝 Notes

- This is a **first draft testing plan** and should evolve
- Focus is on **functional correctness over UI/UX**
- Report issues and edge cases during usage

---

## ✅ Summary

The testing strategy ensures that the Password Reset Manager:
- Follows IT policies
- Handles real-world scenarios
- Fails safely
- Escalates when necessary

---

