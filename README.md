# Enterprise Email Triage & Response Assistant

Track: **Enterprise Agents**  
Framework: **Google Agent Development Kit (ADK) for Python**  
Model: **Gemini (e.g., `gemini-2.0-flash`)**

## 1. Overview

This project implements an **AI-powered email triage and response assistant** for enterprise workflows.

Given an incoming email (subject, body, and optional thread context), the agent:

1. Infers **intent** (support, sales, scheduling, escalation, other)
2. Estimates **priority** (low, medium, high)
3. Decides whether **human review** is recommended
4. Drafts a suggested reply in a configurable enterprise tone
5. Uses **tools** to store and retrieve thread-level context and generated drafts

The agent is built using the **Google Agent Development Kit (ADK)** and powered by **Gemini**.

---

## 2. Problem & Motivation

Enterprise teams (support, sales, operations) spend a lot of time on repetitive email handling:

- Reading and understanding customer messages
- Classifying and prioritizing them
- Writing and rewriting similar responses many times
- Maintaining context over longer threads

This causes slower response times and inconsistent quality.

This agent focuses on **reducing manual effort** while keeping humans in control.

---

## 3. Features & Key Concepts (for Capstone Rubric)

This project demonstrates at least **three key concepts** from the course:

1. **Tools**
   - Custom tools are registered with the ADK agent:
     - `save_email_thread` – store thread metadata and decisions
     - `load_thread_history` – retrieve prior context for a thread
     - `save_draft` – persist generated drafts for downstream use

2. **Sessions & Memory**
   - The agent leverages ADK’s session concept (via `adk run` / `adk web`) for conversational continuity.
   - A simple in-memory store provides thread-level memory (e.g., last triage, last draft).
   - The design is compatible with replacing this with a persistent store or Vertex AI Memory Bank.

3. **Observability**
   - Structured logging is used to trace:
     - Email metadata on input
     - Triage decisions (intent, priority, escalation flag)
     - Tool invocations
     - Final response metadata (length, presence of apology, etc.)

Additional aspects:

- Use of **Gemini** as the LLM model.
- Optional **deployment to Agent Engine** using `adk deploy`.

---

## 4. Architecture

### 4.1 High-Level Flow

1. **Input**: Email subject + body (+ optional thread ID/context)
2. **Agent (Gemini via ADK)**:
   - Reads email
   - Performs triage in a structured way
   - Drafts a reply and self-checks for clarity and tone
3. **Tools**:
   - Save or load thread-level context
   - Save drafted response
4. **Output**:
   - JSON-like structured metadata (intent, priority, escalation flag)
   - Human-readable reply text

### 4.2 Mermaid Diagram

```mermaid
flowchart TD
    U[User / System] -->|Email subject + body| A[Email Triage Agent (Gemini via ADK)]

    subgraph Agent Logic
        A --> Triage[Infer intent + priority + escalation flag]
        Triage --> Draft[Draft reply in enterprise tone]
        Draft --> SelfCheck[Self-check for clarity / completeness]
    end

    SelfCheck --> ToolsDecision{Need tools?}

    ToolsDecision -->|Yes| SaveThread[Tool: save_email_thread]
    ToolsDecision -->|Yes| SaveDraft[Tool: save_draft]
    ToolsDecision -->|Yes| LoadHistory[Tool: load_thread_history]
    ToolsDecision -->|No| Finalize[Assemble final reply]

    SaveThread --> Finalize
    SaveDraft --> Finalize
    LoadHistory --> Draft

    Finalize --> Logs[Logging / Observability]
    Finalize --> UOut[Structured triage + suggested reply]
