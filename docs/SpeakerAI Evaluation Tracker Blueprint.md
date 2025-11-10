# SpeakerAI Evaluation Tracker Blueprint

| Stage | Goal | Metric Families | Example Metrics (with multilingual variants) | Weave/W\&B Tracking Entities | Leaderboard Aggregation Logic |
| ----- | ----- | ----- | ----- | ----- | ----- |
| **1\. Audio Preprocessing** | Detect valid speech, remove noise | Detection accuracy, latency | \- VAD F1 (speech vs silence) \- SNR improvement (dB) \- Latency (ms) | `weave.log(metric=VAD_F1) wandb.log({"SNR": ...})` | Compare across models on `F1 + SNR/latency ratio` |
| **2\. Speaker Diarization** | Separate speakers correctly | Clustering & segmentation | \- **DER** (Diarization Error Rate) \- **JER** (Jaccard Error Rate) \- **\# Speakers Accuracy** (±1 tolerance) | `weave.table("diar_eval", ["DER","JER"])` | Rank models by lowest DER per language |
| **3\. ASR (Speech-to-Text)** | Accurate transcription | Text similarity | \- **WER** (Word Error Rate) \- **CER** (Character Error Rate)\*\* for multilingual\*\* \- **WER\_lang** (per language tag) | `weave.run_group("ASR_eval")` with `lang` dimension | Multi-language leaderboard (`WER_avg`, `WER_hi`, `WER_lo`) |
| **4\. Alignment (Diar \+ ASR)** | Speaker-text coherence | Temporal consistency | \- Segment alignment precision/recall (IoU-based) \- Timestamp drift (ms) | `wandb.Table(columns=[“start_diff_ms”])` | Average IoU per language |
| **5\. Intent Classification** | Detect customer intent | Multi-class accuracy | \- Accuracy \- Macro-F1 \- Matthews CorrCoef (MCC) | `weave.table("intent_eval")` | Leaderboard grouped by intent family |
| **6\. Sentiment / Emotion** | Detect affective state | Multiclass \+ trajectory | \- F1 per emotion \- Sentiment slope (start→end) \- Concordance Corr (emotion vs human label) | `wandb.log({"sent_traj_corr": ...})` | Leaderboard by macro-F1 multilingual |
| **7\. Toxicity / Escalation Detection** | Flag negative events | Binary classification | \- ROC-AUC \- Precision@Recall=0.9 \- Latency (ms) | `weave.metric("toxicity_auc", val)` | Rank by ROC-AUC and latency combined |
| **8\. Summarization** | Generate concise call summary | Generative evaluation | \- ROUGE-L, BLEU \- **BERTScore** \- **COMET (for multilingual)** \- Factual Consistency (LLM eval) | `wandb.Table({"model":..., "rougeL":..., "comet":...})` | Weighted composite leaderboard by `0.4*ROUGE + 0.6*COMET` |
| **9\. Entity Extraction (NER)** | Identify key entities | Token-level F1 | \- F1 (entity span) \- Recall@Top5 \- Entity coverage per language | `weave.dataset_eval("NER")` | Leaderboard grouped by language code |
| **10\. Summarized Quality & Compliance** | End-to-end integrity | Composite scoring | \- Script adherence % \- Compliance F1 \- Agent empathy score (LLM eval) | `wandb.log({"Compliance_F1":...})` | Aggregate all upstream metrics via weighted schema |
| **11\. Latency / Real-Time Performance** | Pipeline throughput | System-level | \- End-to-end latency per call (ms) \- Tokens/sec (if LLM) \- Cost-per-call ($) | `weave.log("latency_ms", avg)` | Efficiency leaderboard (latency vs cost Pareto) |

---

## **Multilingual Support & Evaluation Strategy**

Each run in W\&B / Weave should include:

`wandb.init(project="SpeakerAI", config={`  
    `"language": "es", "dataset": "callcenter_spanish_v2",`  
    `"stage": "ASR", "model": "whisper-large-v3"`  
`})`

And Weave’s schema for results:

`{`  
  `"stage": "ASR",`  
  `"language": "es",`  
  `"metrics": {`  
      `"WER": 0.093,`  
      `"CER": 0.028,`  
      `"Latency_ms": 54.2`  
  `},`  
  `"timestamp": "2025-11-08T02:45:00Z"`  
`}`

Then aggregate multilingual leaderboard:

`weave.query("""`  
`SELECT model, stage, language, AVG(WER) as avg_WER`  
`FROM SpeakerAI_runs`  
`GROUP BY model, stage, language`  
`ORDER BY avg_WER ASC`  
`""")`

## **Composite “System Leaderboard” Example**

| Model Version | Langs | DER ↓ | WER ↓ | Sent-F1 ↑ | COMET ↑ | Latency (ms) ↓ | Compliance ↑ | Overall Score |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| v1.2-whisper-ecapa | EN, ES, FR | 6.8 | 9.3 | 0.84 | 0.78 | 87 | 0.91 | 0.865 |
| v1.3-multilingual-conformer | EN, ES, FR, HI | 5.9 | 8.2 | 0.87 | 0.81 | 95 | 0.88 | **0.879** |
| v1.1-base | EN only | 9.1 | 11.2 | 0.79 | 0.72 | 61 | 0.77 | 0.776 |

## **Weave \+ W\&B Integration Flow**

1. **Each sub-pipeline** (ASR, diarization, summarization) logs its metrics as `weave.Dataset` and `wandb.Run`.

**Link runs hierarchically** using Weave’s `@weave.trace` decorators for traceability:

 `@weave.op()`  
`def asr_model(audio, lang):`  
    `result = asr(audio)`  
    `weave.log_metric("WER", compute_wer(result))`  
    `return result`

2. **Weave panel dashboards** then visualize comparisons over:  
   * per-language leaderboards  
   * cross-stage correlation plots (e.g. WER vs COMET)  
   * system-level performance maps.

### **Additional Evaluation Metrics per Stages:**

### **1\. Audio Preprocessing**

**Goal:** Ensure clean, intelligible, and consistent audio for all downstream models.  
 **Additional metrics:**

* **Perceptual Evaluation of Speech Quality (PESQ)** — perceptual clarity vs ground truth.  
* **Short-Term Objective Intelligibility (STOI)** — measures how understandable the speech remains after enhancement.  
* **Spectral Distortion Index** — average distance in log-mel space before and after denoising.  
* **Speaker Preservation Ratio** — cosine similarity of speaker embeddings before/after denoising.  
* **Dynamic Range Retention** — checks if loudness normalization over-compresses speech.  
* **Latency Distribution Jitter** — stdev of preprocessing delay per 10-sec window (for live pipelines).

### **2\. Speaker Diarization**

**Goal:** Identify “who spoke when” accurately.  
 **Additional metrics:**

* **Overlap-Handling Accuracy** — precision/recall specifically for overlapping speech segments.  
* **Turn-Boundary Sharpness** — mean transition error (ms) at speaker change boundaries.  
* **Cluster Purity / Normalized Mutual Information (NMI)** — quality of speaker clusters.  
* **Speaker Embedding Drift** — cosine distance between embeddings for same speaker across conversation (stability measure).  
* **False Speaker Rate** — extra clusters not corresponding to real speakers.  
* **Language-Conditioned DER** — DER split by language (e.g., bilingual agent switching).

### **3\. ASR (Speech-to-Text)**

**Goal:** Faithfully transcribe speech, across accents and languages.  
 **Additional metrics:**

* **Accent Robustness Score** — WER difference between native vs non-native segments.  
* **Code-Switch WER** — WER computed only for code-switched (multilingual) phrases.  
* **Proper Noun Retention Rate** — accuracy on named entities or uncommon words.  
* **Emotion Leakage** — percentage of emotionally charged words missed or distorted (proxy for tone-sensitive ASR).  
* **Confidence Calibration (ECE)** — Expected Calibration Error for token probabilities.  
* **Semantic Consistency (BERTScore)** — embedding similarity vs reference transcript.

### **4\. Alignment (Speaker-Text Sync)**

**Goal:** Ensure temporal coherence between diarization and transcription.  
 **Additional metrics:**

* **Temporal Drift RMSE** — root-mean-square offset across utterances.  
* **Speaker Confusion Index** — percentage of mis-attributed words.  
* **Cross-Modal Sync Score** — correlation of speech energy peaks with transcript timestamps.  
* **Latency Accumulation Gradient** — linear drift per minute of audio (for long calls).  
* **Dual-ASR Agreement Rate** — if you run two ASR models, measure alignment consistency.

### **5\. Intent Classification**

**Goal:** Detect what the user wants.  
 **Additional metrics:**

* **Intent Ambiguity Index** — entropy of model’s predicted distribution (lower \= more confident).  
* **Cross-Turn Intent Consistency** — same intent maintained across semantically similar turns.  
* **Intent Drift Detection Rate** — correct identification of when user changes intent mid-call.  
* **Rare-Intent Recall** — recall on infrequent or unseen intents.  
* **Multilingual Intent Coherence** — cosine similarity of multilingual intent embeddings (XLM-R).

### **6\. Sentiment / Emotion Analysis**

**Goal:** Capture emotional tone, trajectory, and empathy signals.  
 **Additional metrics:**

* **Emotion Trajectory Coherence** — correlation between predicted and ground-truth emotion timelines.  
* **Sentiment Slope** — regression coefficient over time; measures escalation vs resolution.  
* **Emotion Transition Accuracy** — confusion matrix of emotion changes (e.g., angry→calm).  
* **Prosody-Text Correlation** — correlation between acoustic prosody features and sentiment predictions.  
* **Cultural Emotion Bias** — deviation in emotion labeling across languages (to detect model bias).  
* **Empathy Resonance Score** — correlation between agent tone and customer emotion shift (did empathy reduce negative tone?).

### **7\. Toxicity / Escalation Detection**

**Goal:** Detect when conversations become risky.  
 **Additional metrics:**

* **Escalation Early-Warning AUC** — how soon the model detects toxic cues before escalation.  
* **False-Positive Cost Metric** — penalty-weighted metric balancing false alarms vs misses.  
* **Contextual Toxicity Rate** — difference between local (utterance) and global (context) toxicity judgments.  
* **Linguistic Subtlety Recall** — ability to detect passive-aggressive or sarcastic toxicity (LLM-graded).  
* **Cross-Language Transfer F1** — performance on translated toxic utterances.  
* **Tone Recovery Latency** — average time to detect when tone de-escalates after a trigger.

### **8\. Summarization**

**Goal:** Generate accurate, useful call summaries.  
 **Additional metrics:**

* **Factual Precision (LLM-Eval)** — fraction of generated facts verifiable in transcript.  
* **Compression Ratio** — token ratio (summary/original) — sweet spot often 0.1–0.2.  
* **Entity Coverage Score** — % of key entities retained.  
* **Action Item Recall** — recall on tasks/promises identified in the summary.  
* **Structure Quality** — score for maintaining issue-→-resolution-→-next-steps sequence.  
* **Readability / Fluency Index** — text complexity and coherence (LLM-graded).  
* **Summarization Drift** — semantic divergence between model summaries across similar calls.  
* **Language Balance** — for multilingual transcripts, proportion of segments summarized correctly in each language.

### **9\. Entity Extraction (NER)**

**Goal:** Identify products, accounts, names, etc.  
 **Additional metrics:**

* **Cross-Utterance Consistency** — same entity labeled consistently throughout call.

* **Entity Co-Reference Resolution Accuracy** — detects “he”, “it”, “this order” references.

* **Custom Dictionary Recall** — accuracy against business-specific term lists (SKU, ticket ID).

* **Entity-Role Alignment** — correct attribution of entity to agent vs customer.

* **Transliteration Accuracy** — correct handling of names across scripts/languages.

### **10\. Compliance & Quality**

**Goal:** Evaluate adherence, ethics, and professional quality.  
 **Additional metrics:**

* **Disclosure Compliance Rate** — proportion of mandatory phrases present.  
* **Tone Appropriateness Score** — how well tone matches context (LLM-scored).  
* **Empathy Gap Index** — emotional mismatch between customer frustration and agent tone.  
* **Script Deviation Justification Rate** — when deviation improves outcome (LLM-graded).  
* **Temporal Compliance Density** — compliance events per minute of call.  
* **Cultural Sensitivity Violations** — language-specific compliance errors.

### **11\. System-Level (Real-Time \+ Cost)**

**Goal:** Optimize performance at scale.  
 **Additional metrics:**

* **Throughput-Latency Pareto Index** — composite of latency and throughput normalized.  
* **Compute Efficiency (tokens/ms per $)** — cost-adjusted speed.  
* **Memory Footprint Stability** — variance in memory usage across batch sizes.  
* **Streaming Continuity Score** — proportion of calls without pipeline stalls.  
* **Multilingual Coverage Rate** — fraction of supported languages with ≥ 90% of baseline metrics.  
* **End-to-End Correlation Index** — correlation between early-stage metrics (e.g., WER) and business metrics (CSAT, resolution rate).

