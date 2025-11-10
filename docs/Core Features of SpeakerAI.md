# Core Features of SpeakerAI

### **1\. Real-Time (During the Call)**

These are low-latency, streaming features designed for immediate insights and agent assistance:

* **Speaker Diarization (Who is speaking when)**  
   Continuous separation of speakers in real time; tag them as *Agent* vs *Customer*.

* **Automatic Speech Recognition (ASR)**  
   Streaming transcription with confidence scores and timestamps.

* **Intent & Sentiment Detection**  
   Real-time classification of customer intent (complaint, query, order, escalation, etc.) and emotion (frustrated, calm, angry, happy).

* **Toxicity / Escalation Alerts**  
   Detection of rising negative sentiment or specific keywords (e.g., “cancel”, “manager”, “refund”) to trigger supervisor alerts.

* **Agent Guidance (Real-time CoPilot)**  
   Suggest next best actions, policy scripts, empathy phrases, or product information dynamically during the call.

* **Knowledge Retrieval (RAG layer)**  
   Search internal knowledge base or CRM to fetch relevant responses based on context and intent.

### **2\. Post-Call (After the Call)**

These handle analytics, quality control, and continuous improvement:

* **Speaker Attribution & Summarization**  
   Generate a structured call summary: issue → resolution → next steps, tagged by speaker role.

* **Quality Scoring & Compliance Checks**  
   Automatically evaluate if agents followed required scripts, disclosures, and policies.

* **Customer Experience Metrics**  
   Derive *CSAT proxy scores* using tone, sentiment trajectory, and language markers.

* **Topic & Entity Extraction**  
   Extract named entities (product names, ticket IDs, dates, etc.) and conversation topics for CRM enrichment.

* **Conversation Embeddings for Search & Analytics**  
   Embed call transcripts for semantic search, clustering, and similarity-based QA.

* **Training Data Generation for Agents**  
   Auto-generate snippets of high-performing calls for new agent training and benchmarking.

* **Supervisor Dashboards & Trends**  
   Aggregate data by agent, team, sentiment, topics, and compliance for BI visualization.

