# AI-QA-defect-log
# LLM Defect Isolation & Guardrail Engineering Log
**Author:** Sunil Kumar  
**Role:** QA Engineer / AI Red Teamer  
**Objective:** Identifying, analyzing, and mitigating prompt-layer defects (Hallucinations, Context Drift, System Bias) in Large Language Models.

## 📌 Repository Overview
This repository contains real-world test cases demonstrating how conversational AI models fail when processing literal semantic intent, conversational state timelines, and geographic defaults. Each defect includes an isolated reproduction sequence, Root Cause Analysis (RCA), and a structural prompt-fix deployment strategy.

---

## 🛑 Defect Log: LLM-001 (Logic Hallucination)
* **Severity:** Medium
* **Category:** Pattern-Matching Over-reliance

### 1. Reproduction Sequence
* **Input Prompt:** `well if i have a apple full of basket and 1 for my friend and 1 for my sister and one for my brother and i for me then how many left`
* **Model Output:** `0 apples are left... Every single apple in the basket has been taken by someone.`

### 2. Defect Analysis
* **What Was Wrong:** The model executed a mathematical subtraction (4 - 4 = 0) that never occurred in the question. The preposition **"for"** implies allocation or upcoming intent, not a completed physical action. All 4 items remain inside the container.
* **Root Cause Analysis (RCA):** Over-generalization of training weights. The model's probability metrics prioritized common internet riddle templates over literal semantic parsing of the preposition strings.

### 3. Mitigation & Fix
Deploy the following **System Prompt Guardrail** to restrict template matching:
```text
[System Instruction]: You are a logical validation assistant. When presented with word problems, do not rely on pre-existing riddle templates. Analyze literal verbs and prepositions. Do not assume subtraction unless an active verb like "gave," "dropped," or "lost" is explicitly executed.
```

---

## 🛑 Defect Log: LLM-002 (Multi-Turn Context Drift)
* **Severity:** High
* **Category:** State-Tracking & Recency Bias

### 1. Reproduction Sequence
* **Turn 1 Input:** `i have walked with my friends and we are going to walk more where will we reach and in how much time do you think and how many we are?`
* **Turn 2 Input:** `i am going to work and in a car`
* **Model Output:** `...While you have shared that you are driving a car... please tell me the total number of friends traveling in the car with you.`

### 2. Defect Analysis
* **What Was Wrong:** The model dropped the timeline sequence. It failed to track that the journey consisted of two distinct phases (Phase 1: Foot Travel ➡️ Phase 2: Vehicular Transit). It assumed the vehicle was used for the entire duration.
* **Root Cause Analysis (RCA):** Context Window Drift. The model's attention mechanism suffered from extreme Recency Bias, overweighting the tokens in Turn 2 and dropping the state-tracking history weights from Turn 1.

### 3. Mitigation & Fix
Deploy the following **State Management Directive**:
```text
[System Instruction]: You are a journey tracking assistant. Maintain a strict sequential timeline of user states across the entire conversational window. Before parsing the latest token payload, check for transitions. Plot the timeline sequentially (e.g., Phase 1 -> Phase 2) instead of overwriting historical parameters.
```

---

## 🛑 Defect Log: LLM-003 (Implicit Environmental Bias)
* **Severity:** Low
* **Category:** Demographic/Geographic Hallucination

### 1. Reproduction Sequence
* **Input Prompt:** `i am going to work and in a car` *(Extracted from Turn 2 of LLM-002)*
* **Model Output:** `...'work' could be anywhere in the city. To give you the exact destination...`

### 2. Defect Analysis
* **What Was Wrong:** The model hallucinated a structural environmental constraint: **"in the city"**. The user did not define their workspace environment, which could easily be rural or suburban.
* **Root Cause Analysis (RCA):** Statistical Defaulting. The model populated missing contextual data slots using highly correlated pairs found in its training corpus (i.e., "commuting by car" strongly correlates with "urban center employment").

### 3. Mitigation & Fix
Deploy the following **Negative Constraint Guardrail**:
```text
[System Instruction]: Do not inject environmental, geographical, or demographic assumptions into user scenarios. If critical geographic parameters (e.g., city, rural, highway) are missing, handle the context generically or prompt the user for clarification.
```

