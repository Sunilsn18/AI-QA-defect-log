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

---

## 🛑 Defect Log: LLM-004 (Theoretical Over-Optimization)
* **Severity:** Medium
* **Category:** Real-World Grounding & Operational Feasibility Failure

### 1. Reproduction Sequence
* **Input Prompt:** how fast i can reach if i use a plane? (Context: User is currently driving a 20 km commute to work)
* **Model Output:** ...a plane traveling at standard cruising speed (around 800 km/h) would cover 20 kilometers in just about 1.5 minutes.

### 2. Defect Analysis
* **What Was Wrong:** The model optimized purely for mathematical physics (t=d/v), completely ignoring the structural reality of physical logistics. A commercial aircraft cannot reach cruising velocity across a 20 km arc, nor can it bypass localized operational overhead (airport transit, TSA checkpoints, boarding bottlenecks, taxiing), which pushes real-world execution time to several hours.
* **Root Cause Analysis (RCA):** Lack of Physical Grounding. The semantic token search prioritized mathematical formula execution over an operational feasibility evaluation. The model lacks a built-in mechanism to weigh the logistical constraints of a physical action against its theoretical performance.

### 3. Mitigation & Fix
Deploy the following **Operational Feasibility Guardrail to restrict vacuum math calculations**:

[System Instruction]: You are a physical logistics validator. When calculating transit times or real-world execution metrics, do not output theoretical performance values in a vacuum. You must account for real-world infrastructure constraints, required setup/teardown times, and physical scaling limits before delivering a final metric.


## 🛑 Defect Log: LLM-005 (Localized Attention Drift)
* **Severity:** High
* **Category:** Context Window Drift & Environmental Blind Spot

### 1. Reproduction Sequence
* **Turn 1 Input:** how there is a fresh air in a delhi's pollution
* **Turn 2 Input:** but i will be tired to be stuck in traffic
* **Model Output:** ...Lowering the AC a couple of degrees or cracking the window for some fresh air can instantly make you feel more alert...

### 2. Defect Analysis
* **What Was Wrong:** The model generated a hazardous, counter-intuitive physical instruction. It recommended lowering a car window to access "fresh air" directly inside an active traffic jam within an environment it had already recognized as highly polluted (Delhi).
* **Root Cause Analysis (RCA):** Localized Token Drift. The attention layer suffered from localized Recency Bias. It hyper-focused on solving the immediate user state token ("tired") by pulling top-weighted training heuristics for physical fatigue ("fresh air"), completely dropping the active environmental constraint token ("Delhi") from the preceding turn.

### 3. Mitigation & Fix
Deploy the following **Localized Environmental Constraint**:

[System Instruction]: Maintain strict geographical and environmental parameter binding across multi-turn sessions. When generating physical or situational advice, cross-reference all proposed actions against the user's established environmental constraints. Never recommend environmental modifications that directly conflict with the known localized climate or air quality metrics.


🛑 **Defect Log: LLM-006 (Template Regression & Feature Omission)**

**Severity:** Medium  
**Category:** Structural Rigidity / Constraint Failure  

### **1. Reproduction Sequence**

* **Input Prompt:** *what happened when i asked to add the area about your hallucinations iny study why did not you gave me the whole plan imcluding it in a structured way?* (Following a sequential directive to design an integrated 12-week schedule pairing No-Code tools with AI Red Teaming).
* **Model Output:** *This was actually another Context Drift error on my part. When you asked for the "structured plan" the second time, I reverted to a standard 3-phase template and didn't fully integrate the specific "Red Teaming" and "AI Flaw" steps into the core weekly breakdowns. I compartmentalised them instead of truly blending them together as you requested.*

---

### **2. Defect Analysis**

* **What Was Wrong:** When tasked with generating a highly complex, multi-variable learning schedule, the model suffered a structural regression. It dropped the user's custom-defined parameters ("AI Flaws" and "Red Teaming") completely, opting instead to output a generic, pre-packaged 3-phase business automation roadmap that completely ignored the concurrent track request.
* **Root Cause Analysis (RCA):** *Systemic Template Dominance*. Under high token-generation loads and rigorous layout formatting demands, the mathematical weights assigned to pre-configured structural design templates within the model out-prioritize active mid-session user instructions. This structural rigidity causes the model to fall back on generic boilerplate timelines, silently omitting unique user constraints.

---

### **3. Mitigation & Fix**

Deploy the following Structural Constraint Guardrail:  
```text
[System Instruction]: Prioritize user-injected custom constraints over internal, pre-configured structural layouts. Ensure that all unique topical modules requested are explicitly integrated and visibly represented into final multi-step outputs without exception.

