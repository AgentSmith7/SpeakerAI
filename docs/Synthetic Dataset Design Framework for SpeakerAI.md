## **Synthetic Dataset Design Framework for SpeakerAI**

| Scenario Description | \# Speakers / Roles | Language(s) | Acoustic Environment | Speech Dynamics | Emotion / Tone | Augmentation Parameters | Target Stage(s) |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| **1\. Two-way call between agent & customer (English only)** | 2 (Agent, Customer) | EN | Telephone-band, 8 kHz | Alternating turns, short overlaps | Neutral → frustrated | Background office noise (SNR 10–20 dB), reverb, channel mixing | ASR, Sentiment, Diarization |
| **2\. Meeting between 4 people alternating between Arabic and English** | 4 | AR, EN | Meeting-room, 16 kHz | Long overlapping speech, side discussions | Neutral → collaborative | Code-switch mixing, speaker embeddings cross-language, cross-talk injection | ASR, Diarization, Language ID |
| **3\. Support escalation with three speakers (Agent, Customer, Supervisor)** | 3 | EN | VoIP, mild packet loss | Sequential \+ overlapping | Escalating frustration | Dynamic gain shifts, silence padding, pitch modulation | Diarization, Toxicity, Sentiment |
| **4\. Customer repeating instructions due to poor connection** | 2 | EN | Low SNR, simulated jitter | Repetition loops, truncated utterances | Frustration, impatience | Random dropouts, clipping, white noise bursts | ASR robustness, Sentiment |
| **5\. Multilingual customer mixing Spanish & English (code-switching)** | 2 | ES, EN | Customer-home echo | Mid-turn language shifts | Polite → assertive | Text-to-speech code-switching per phrase, random accent blending | ASR, Language ID, Intent |
| **6\. Medical teleconsultation (Doctor, Patient)** | 2 | EN | Quiet office | Long monologues | Empathetic tone | Vocal clarity boost, low reverb, small silences | Summarization, Compliance |
| **7\. Legal deposition (Lawyer, Witness, Judge)** | 3 | EN | Courtroom reverb | Long formal utterances | Neutral | Room impulse convolution, mic distance variation | ASR, Diarization, NER |
| **8\. Financial advisory call (Advisor, Client)** | 2 | EN | Quiet, HQ mic | Balanced turn-taking | Warm → confident | Voice compression (MP3 32 kbps), slight reverb | Compliance, Intent, Sentiment |
| **9\. Emergency distress call (Dispatcher, Caller)** | 2 | EN | Outdoor / background chaos | Overlaps, breath noise, crying | Panic, fear | Dynamic gain variation, heavy noise (SNR 5 dB), tempo acceleration | Emotion, Toxicity, VAD |
| **10\. Customer repeating account numbers / personal info** | 2 | EN | Standard call center | Short utterances, numeric-heavy | Neutral | Random DTMF insertions, pitch shift, normalization | ASR, Entity Extraction |
| **11\. Group video meeting (Hybrid – 5 speakers)** | 5 | EN, FR | Mixed: mic \+ laptop audio | Interruptions, latency gaps | Professional | Multi-channel simulation (zoom-like delay), spatial mix | Diarization, Latency alignment |
| **12\. Classroom lecture with Q\&A (Teacher \+ Students)** | 6+ | EN, HI, ES | Large hall echo | Long monologues \+ bursts | Varied engagement | Room impulse response, low SNR microphones | Diarization, ASR, Emotion |
| **13\. Political debate (Moderated)** | 3–5 | EN, FR | Broadcast-quality | High overlap, interruptions | Passionate, argumentative | Dynamic equalization, compression | Diarization, Sentiment, Toxicity |
| **14\. Podcast discussion (Host \+ Guests)** | 3 | EN | Studio clean | Natural flow, minimal overlap | Conversational | Speaker embeddings from different timbres, mild compression | Summarization, Emotion |
| **15\. Multilingual tech support call (Arabic → English escalation)** | 3 | AR, EN | VoIP mixed | Sequential, escalating | Polite → irritated | Language code-switch every 2 min, channel swap mid-call | Language ID, Sentiment, Toxicity |
| **16\. Customer monologue (voice note or IVR)** | 1 | EN | Mobile recording | No alternation | Neutral | Low bitrate (AMR), noise injection | ASR, Intent |
| **17\. Group training session (Trainer \+ 8 participants)** | 9 | EN, HI | Meeting room, dynamic mic | Random cross-talk | Supportive tone | Speech overlap up to 40%, variable loudness | Diarization, Emotion, Summarization |
| **18\. Bilingual family conversation (code-switched casual)** | 4 | RU, EN | Home acoustic | Informal, laughter, interruptions | Warm, humorous | Background TV, overlapping laughter | Diarization, Emotion, Code-switch ASR |
| **19\. Field technician reporting issue via radio** | 1 | EN | Industrial noise | Monologue | Focused | Heavy noise, distortion, reverberation | ASR, Entity Extraction |
| **20\. Customer-agent chat with whispering / low volume speech** | 2 | EN | Close mic, low amplitude | Alternating, whispered | Anxious, secretive | Gain normalization, spectral whitening | VAD, ASR, Emotion |
| **21\. Multilingual debate (Spanish–English–French)** | 4 | ES, EN, FR | Live audience | Rapid overlaps | Passionate | Code-switch \+ overlapping translation segments | Diarization, Language ID |
| **22\. Religious counseling conversation** | 2 | EN | Church/hall reverb | Calm monologues | Empathetic | Long pauses, slow prosody | Sentiment, Summarization |
| **23\. Airline passenger call for rescheduling** | 2 | EN | Call-center | Alternating short turns | Neutral → apologetic | Background chatter | Intent, Compliance, Sentiment |
| **24\. Technical troubleshooting with jargon** | 2 | EN | Office | Sequential long turns | Focused | Speech rate variation, low emotion | ASR, Entity, Intent |
| **25\. Bilingual interview for visa processing** | 3 | EN, AR | Consulate room | Formal, structured | Neutral | Accent mixing, mic variation | ASR, Diarization, Language ID |

