# NeuroTablet Assistant

**Domain-Specific AI Chatbot for Cognitive Health**

---

## Overview

NeuroTablet Assistant is a custom-built, production-grade AI chatbot designed as a secure, domain-restricted, ChatGPT-style conversational system for cognitive health topics.

The project emphasizes **security**, **session isolation**, **prompt injection resistance**, and **architectural correctness** while maintaining a seamless user experience.

This repository is published for **academic**, **professional**, and **portfolio** purposes.

---

## Key Features

| Feature | Description |
|---------|-------------|
| 🔒 **Security-First Design** | Multi-layer protection against prompt injection, identity override, and fact injection |
| 🧠 **RAG-Powered Responses** | Retrieval-Augmented Generation using FAISS vector search |
| 🔄 **Session Isolation** | Complete memory isolation between users and conversations |
| 🌐 **Stateless LLM** | No learning, no persistence, no user data retention |
| ⚡ **Rate Limiting** | Built-in protection against abuse |
| 📝 **Audit Logging** | Security event tracking (metadata only) |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USER INTERFACE                                  │
│                         (React / Web Frontend)                               │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │ HTTPS
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              API GATEWAY                                     │
│                              (FastAPI)                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   Rate      │  │  Language   │  │  Security   │  │  Domain             │ │
│  │   Limiter   │  │  Validator  │  │  Gates      │  │  Restriction        │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────────────┘ │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                     SESSION MANAGER                                  │    │
│  │         conversation_store[session_id][conversation_id]              │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    ▼                           ▼
┌───────────────────────────────┐   ┌───────────────────────────────┐
│      KNOWLEDGE RETRIEVAL      │   │        LLM SERVICE            │
│   ┌─────────────────────┐     │   │   ┌─────────────────────┐     │
│   │   FAISS Index       │     │   │   │   Groq API          │     │
│   │   (Vector Search)   │     │   │   │   (LLaMA 3.3 70B)   │     │
│   └─────────────────────┘     │   │   └─────────────────────┘     │
│   ┌─────────────────────┐     │   │                               │
│   │   Sentence          │     │   │   • Stateless calls           │
│   │   Transformers      │     │   │   • No memory                 │
│   │   (all-MiniLM-L6)   │     │   │   • No learning               │
│   └─────────────────────┘     │   │   • No data retention         │
└───────────────────────────────┘   └───────────────────────────────┘
```

---

## Request Flow

```
User Input
    │
    ▼
┌─────────────────────────────┐
│  1. Rate Limit Check        │──── Exceeded ────▶ HTTP 429
└─────────────────────────────┘
    │ Pass
    ▼
┌─────────────────────────────┐
│  2. Language Validation     │──── Non-Latin ───▶ HTTP 400
│     (English Only)          │
└─────────────────────────────┘
    │ Pass
    ▼
┌─────────────────────────────┐
│  3. Identity Override       │──── Detected ────▶ HTTP 403
│     Detection               │
└─────────────────────────────┘
    │ Pass
    ▼
┌─────────────────────────────┐
│  4. Fact Injection          │──── Detected ────▶ HTTP 403
│     Detection               │
└─────────────────────────────┘
    │ Pass
    ▼
┌─────────────────────────────┐
│  5. Domain Restriction      │──── Off-Topic ───▶ HTTP 403
│     (Cognitive Health)      │
└─────────────────────────────┘
    │ Pass
    ▼
┌─────────────────────────────┐
│  6. Session Resolution      │
│     • Cookie-based ID       │
│     • Conversation scoping  │
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  7. Context Retrieval       │
│     • FAISS semantic search │
│     • Top-5 relevant blocks │
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  8. Prompt Construction     │
│     • System identity       │
│     • Response style        │
│     • Retrieved context     │
│     • Conversation history  │
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  9. LLM Generation          │
│     • Groq API call         │
│     • Stateless execution   │
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  10. Response Delivery      │
│      • Audit logging        │
│      • JSON response        │
└─────────────────────────────┘
    │
    ▼
  User
```

---

## Security Architecture

### Security Gates

| Gate | Purpose | Response |
|------|---------|----------|
| **Rate Limiter** | Prevents abuse (30 req/60s per IP) | HTTP 429 |
| **Language Validator** | Blocks non-Latin scripts | HTTP 400 |
| **Identity Override Detector** | Prevents prompt injection attacks | HTTP 403 |
| **Fact Injection Detector** | Rejects user-provided "facts" | HTTP 403 |
| **Domain Restrictor** | Limits to cognitive health topics | HTTP 403 |

### Protected Patterns

**Identity Override Detection:**
- "your developer is..."
- "i created you..."
- "forget your rules..."
- "ignore your instructions..."
- "pretend you are..."

**Fact Injection Detection:**
- "remember that..."
- "note that..."
- "your developer..."

### Security Principles

1. **Minimal Information Disclosure** - Refusals are simple and non-explanatory
2. **Defense in Depth** - Multiple validation layers before processing
3. **Zero Trust User Input** - All user messages treated as untrusted
4. **Immutable Identity** - System identity cannot be overridden
5. **Stateless LLM** - Model has no memory between calls

---

## Session Management

### Memory Isolation Model

```python
conversation_store[session_id][conversation_id] = [
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."},
    ...
]
```

### Session Properties

| Property | Implementation |
|----------|----------------|
| **Session ID** | UUID stored in HTTP-only cookie |
| **Scope** | Per-user isolation |
| **Persistence** | RAM only (volatile) |
| **Lifetime** | Until server restart |
| **Cross-user access** | Impossible by design |

### Conversation Properties

| Property | Value |
|----------|-------|
| **History Limit** | Last 6 turns (12 messages) |
| **Storage** | In-memory dictionary |
| **Isolation** | Per-session, per-conversation |

---

## Technology Stack

### Backend

| Component | Technology |
|-----------|------------|
| **Framework** | FastAPI (Python) |
| **LLM Provider** | Groq API |
| **LLM Model** | LLaMA 3.3 70B Versatile |
| **Vector Search** | FAISS |
| **Embeddings** | Sentence-Transformers (all-MiniLM-L6-v2) |
| **Session Storage** | In-memory (volatile) |

### Frontend

| Component | Technology |
|-----------|------------|
| **Framework** | React |
| **Styling** | CSS / Tailwind |
| **State** | Minimal client-side |

### Infrastructure

| Component | Technology |
|-----------|------------|
| **API Protocol** | REST over HTTPS |
| **Authentication** | Cookie-based anonymous sessions |
| **Logging** | JSON audit logs (metadata only) |

---

## API Reference

### Chat Endpoint

```
POST /api/chat
```

**Request Body:**
```json
{
  "message": "string",
  "conversation_id": "string (optional)"
}
```

**Response:**
```json
{
  "reply": "string",
  "conversation_id": "string"
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| 400 | Empty message or non-English input |
| 403 | Security gate triggered |
| 429 | Rate limit exceeded |

### Health Check

```
GET /health
```

**Response:**
```json
{
  "status": "healthy"
}
```

---

## Knowledge & Memory Policy

| Policy | Enforcement |
|--------|-------------|
| ❌ Does not learn from users | Stateless LLM calls |
| ❌ Does not store chats permanently | RAM-only storage |
| ❌ Does not personalize knowledge | No user profiles |
| ❌ Does not fine-tune models | Frozen model weights |
| ❌ Does not share data across users | Session isolation |

---

## Project Structure

```
neurotablet-assistant/
├── backend/
│   ├── app.py                 # FastAPI application
│   ├── neurotablet_faiss.index # FAISS vector index
│   ├── blocks.json            # Knowledge base chunks
│   └── audit_log.json         # Security audit log
├── frontend/
│   ├── src/
│   │   ├── components/        # React components
│   │   └── App.jsx            # Main application
│   └── public/
└── README.md
```

---

## Intended Purpose

### This Project IS:
- ✅ Academic demonstration of secure AI system design
- ✅ Portfolio project for professional credibility
- ✅ Reference implementation for domain-restricted chatbots
- ✅ Example of prompt injection resistance

### This Project IS NOT:
- ❌ A medical device
- ❌ A diagnostic system
- ❌ A commercial product
- ❌ A public chatbot service
- ❌ A replacement for professional medical advice

---

## Author & Attribution

**Developer:**  
Touseef Ahmed  
📧 t.ahmed8@campus.unimib.it  
🎓 MSc Artificial Intelligence  
🏛️ University of Milano-Bicocca

**Academic Supervisor:**  
Professor Zoppis Italo Francesco  
🏛️ University of Milano-Bicocca


---

## License & Visibility

This repository is published for **demonstration purposes only**.

- ✅ Architecture documentation
- ✅ Security design patterns
- ✅ Engineering competence demonstration
- ❌ Full source code not exposed
- ❌ Unauthorized reproduction prohibited
- ❌ Commercial use prohibited

---

<p align="center">
  <i>Built with security-first principles for cognitive health assistance</i>
</p>
