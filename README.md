# Daily Digest Agent  
*A role-aware, agentic system for surfacing the most relevant information from Slack.*

---

## ⭐ Overview
This project presents an end-to-end AI agent architecture that generates **personalized daily or weekly digests** for each team member based on their role, project context, and evolving priorities.

Modern engineering teams generate huge amounts of information across Slack channels, threads, and integrations. Important updates get buried, roles have different priorities, and manual alignment becomes expensive.

This system solves that by using **autonomous AI agents** that:
- analyze Slack conversations  
- track user-specific priorities  
- understand project phases  
- detect blockers and dependencies  
- generate structured digests  
- adapt based on feedback  

The result is a high-signal, role-specific summary that keeps everyone aligned without constant manual coordination.

---

## 🧠 Core Architecture

### **1. LLM Backbone**
Uses **Claude 4.5 Sonnet** for:
- large-scale text analysis  
- long-context Slack thread understanding  
- role-aware summarization  

### **2. Reasoning Strategy**
Implements a **plan-and-execute** approach:
- *planner* → identifies what to analyze  
- *executor* → produces the digest  

This reduces hallucinations and improves reliability on large message sets.

### **3. Tools & Actions**
Agents retrieve structured data through:
- Slack API  
- Calendar API  
- GitHub/GitLab notifications  
- Notion links & project management updates  

This gives the agent unified visibility into communication + timing.

### **4. Memory & State Management**
A multi-layered memory system ensures long-term personalization:
- **Qdrant Vector DB** → semantic search on Slack messages  
- **JSON storage** → user preferences, priority models  
- **Buffer memory** → avoids duplicates and tracks context across digests  

### **5. Control Loop**
Runs weekly with urgent overrides.

1. **Observe**
   - Slack activity, meetings, user feedback  

2. **Decide**
   - Rank messages by relevance, priority, urgency  

3. **Act**
   - Generate structured digest + deliver via Slack  

Urgent triggers allow mid-week digest generation based on blockers or @mentions.

---

## 🛠️ Technology Stack

- **Language Model:** Claude 4.5 Sonnet  
- **Vector DB:** Qdrant  
- **Orchestration:** CrewAI (multi-agent collaboration)  
- **APIs:** Slack, Calendar, GitHub/GitLab, Notion  
- **State Storage:** JSON + vector embeddings  

---

## 🤖 Multi-Agent System with CrewAI
Each user has their own **Role Agent** that:
- learns their evolving priorities  
- tracks role-specific concerns  
- identifies user-relevant blockers  
- aligns updates across team dependencies  

Agents coordinate through CrewAI to share context and ensure  
**team-level alignment without forcing more meetings.**

---

## 📦 Pseudocode

```python
def generate_weekly_digest(user):
    role = fetch_user_role(user)
    msgs = slack_api.get_messages(user, days=7, include_threads=True)
    events_next = calendar_api.get_events(user, days=7)

    prev = storage.load_priorities(user) or []
    prev_digest = storage.load_last_digest(user)

    thread_id = storage.get_or_create_thread(user)

    embs = qdrant.embed(msgs)
    q_text = " ".join(prev + role_priority_hints(role) + agenda_hints(events_next))
    q_emb = embed(q_text)

    hits = qdrant.search(q_emb, corpus=embs, top_k=50, min_sim=0.62, recency_decay=0.015)
    hits = dedupe_by_ts(hits)

    digest = llm.summarize_to_schema(
        messages=hits,
        role=role,
        previous_digest=prev_digest,
        goal=project_goal(),
        deadline=project_deadline()
    )

    changes = diff_priorities(prev, digest.priorities)
    storage.save_priorities(user, digest.priorities)
    storage.save_last_digest(user, digest)

    slack_api.post(thread_id, render_digest(digest, changes))
