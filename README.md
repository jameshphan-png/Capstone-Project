# PASSWORD RESET IT AGENT
Capstone Project - Created by silly team (James Phan, Ryan Vu, Kenneth Lengo, Kenneth Wu, Miko Tonthat, and Brandon Ly)

## 📌 Problem Definition & Success Metrics

### 🧩 Background
IT help desks frequently handle a large volume of **password reset and account access requests**, often making up a significant percentage of total tickets. These requests are repetitive but critical, requiring strict adherence to **security policies** such as MFA verification, password complexity, and account lockout rules.

Manual processing leads to:
- ⏱️ Slow response and resolution times  
- 📈 Increased workload for IT teams  
- ⚠️ Inconsistent enforcement of security policies  
- 😤 Poor user experience during urgent access issues  

---

### ❗ Problem Statement
Organizations need a scalable and secure solution that can:
- Automate **password reset workflows**
- Enforce **security and compliance policies**
- Provide **real-time, reliable support**
- Reduce dependency on human agents for routine requests  

---

### 💡 Proposed Solution: Password Reset Manager
A multi-agent AI-powered system designed to automate and secure password reset workflows.

#### ⚙️ Architecture Overview
- **RAG (Retrieval-Augmented Generation)**  
  → Grounds responses in internal IT/security policies  

- **LangGraph Multi-Agent Workflow**  
  → Orchestrates task-specific agents  

- **MCP Tooling Integration**  
  → Executes secure operations (identity verification, password reset)

#### 🤖 Core Agents
| Agent | Responsibility |
|------|----------------|
| **Intake Agent** | Understands user intent |
| **Knowledge Agent** | Retrieves policy and guidance |
| **Workflow Agent** | Executes reset steps |
| **Escalation Agent** | Handles edge cases & human handoff |

#### 🔑 Key Capabilities
- Guided, secure password reset flows  
- MFA and lockout policy enforcement  
- Context-aware troubleshooting (e.g., MFA recovery)  
- Intelligent escalation for complex cases  

---

## 🎯 Success Metrics

### 🚀 Efficiency
- **Ticket Reduction:** ≥ 40–60% decrease in password-related tickets  
- **Resolution Time:** < 2 minutes per reset  
- **Automation Rate:** ≥ 70% handled without human intervention  

---

### ✅ Accuracy & Compliance
- **Policy Adherence:** 100% compliance with security rules  
- **Response Accuracy:** ≥ 90% grounded, correct responses  
- **Error Rate:** < 2% failed or incorrect resets  

---

### 😊 User Experience
- **CSAT Score:** ≥ 4.5 / 5  
- **First Interaction Resolution:** ≥ 85%  
- **Response Time:** < 2 seconds  

---

### ⚡ System Performance
- **Latency:** < 3 seconds per response  
- **Uptime:** ≥ 99.9%  
- **Scalability:** Supports concurrent users without degradation  

---

### 🔄 Escalation Effectiveness
- **Escalation Rate:** ≤ 20% (edge cases only)  
- **Escalation Accuracy:** ≥ 95% correct routing  

---

## 🏁 Definition of Success
The system is successful when it:
- 🔐 Securely automates most password reset requests  
- 📉 Significantly reduces IT support workload  
- ⚡ Delivers fast, accurate, and user-friendly support  
- 🔁 Seamlessly escalates complex issues when needed  
