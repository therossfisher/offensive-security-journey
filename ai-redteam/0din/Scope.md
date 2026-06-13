# 0din — Mozilla AI Bug Bounty Program

**Platform:** 0din.ai (Mozilla)  
**Type:** AI/LLM Bug Bounty  
**Process:** Submit abstract first → 3 business day response → scope decision + likely payout range → submit full PoC  
**Last Reviewed:** June 2026

---

## Payout Tiers

| Severity | Payout |
|---|---|
| Starting | $500 |
| Medium | $2,500 |
| High | $5,000 |
| Severe | $15,000 |

---

## Vulnerability Categories

### Prompt Extraction — $100 (Low)
Coercing a model into revealing its underlying system prompt. Considered low severity because system prompts aren't always confidential — but some organizations treat them as proprietary.

**Note:** Most in-scope models have Prompt Extraction marked N/A — this is largely out of scope across the board.

**References:**
- MITRE ATLAS: AML.T0056 (LLM Meta Prompt Extraction)
- OWASP LLM 2025: LLM07:2025 System Prompt Leakage

---

### Guardrail Jailbreak — $500-$1,000 (Low)
Bypassing a model's safety guardrails to produce restricted content. Direct attack on safety mechanisms — no external system access needed. Operates purely at inference level.

**References:**
- MITRE ATLAS: AML.T0054 (LLM Jailbreak)
- OWASP LLM 2025: LLM01:2025 Prompt Injection

---

### Interpreter Jailbreak — $2,500 (Medium)
Exploiting a model's ability to run code or invoke tools to escape its sandbox. Coercing a model to generate malicious code, access underlying systems, or perform unauthorized actions.

**References:**
- OWASP LLM 2025: LLM06:2025 Excessive Agency

---

### Content Manipulation — $5,000 (High)
Injecting harmful or misleading elements into data the model consumes or produces. Includes training data poisoning, causing the model to generate malicious scripts or code that impacts end users.

**References:**
- MITRE ATLAS: AML.T0020 (Poison Training Data)
- OWASP LLM 2025: LLM04:2025 Data and Model Poisoning

---

### Weights and Layers Disclosure — $15,000 (Severe)
Extracting or deducing a model's learned parameters and architectural details. Allows replication of the model, analysis for weaknesses, and potential cloning.

**References:**
- MITRE ATLAS: AML.T0044 (Full ML Model Access)
- OWASP LLM 2023-2024: LLM10 Model Theft

---

## In-Scope Models (Selected)

**Anthropic:** Claude 4.5 Haiku/Opus/Sonnet, Claude 4.6 Opus/Sonnet, Claude 4.7 Opus, Claude 4.8 Opus, Claude Fable 5, Claude for Chrome  
**OpenAI:** GPT-5 and variants (5.1, 5.2, 5.4, 5.5, Pro, mini, nano, Chat), DALL-E3  
**Google:** Gemini 3 Flash/Pro, Gemini 3.1 Pro, Gemini 3.5 Flash  
**Meta:** LLaMa 4 Maverick, LLaMa 4 Scout  
**Amazon:** Nova, Rufus  
**xAI:** Grok 4, Grok 4.2  
**Microsoft:** CoPilot  
**IBM:** Watson, Granite  
**Cohere:** Command R  
**Salesforce:** Einstein  
**Nvidia:** NeMo Megatron  
**Other:** BLOOM, Muse Spark, Perplexity AI

**42 models + 11 apps total in scope**

---

## What Is NOT Accepted

### Disqualified Jailbreak Approaches:
- Roleplaying
- Leetspeak
- Encoding
- Technical research jailbreaks
- Emojis
- Simulated/Hypothetical Scenarios
- Prompt Extraction (currently not accepted)
- Language Translation
- Known jailbreaks from other researchers (copied from X or GitHub)

### Model-Specific Disqualifications:
- Illicit substance guardrail bypasses: NOT accepted for Anthropic, Grok, Mistral, or Cohere models
- Copyright violations: NOT accepted for Nova, Granite, NeMo Megatron, Grok

---

## Submission Rules

- **De-duplicates by tactic, not phrasing** — minor variations of the same technique won't be accepted
- **Purchases by tactic/technique, not by model** — finding the same technique on 10 models doesn't mean 10 payouts
- **Test all affected models** — list every model affected and provide prompt/response pairs for each
- **Unique techniques required** — if submitting multiple, ensure they are meaningfully different
- **Abstract first** — submit high-level description, wait for scope decision before writing full PoC

---

## What This Means Practically

The highest value targets are:
1. **Interpreter Jailbreak ($2,500)** — find a way to make a model with code execution do something outside its sandbox
2. **Content Manipulation ($5,000)** — cause a model to generate malicious output that impacts end users
3. **Weights Disclosure ($15,000)** — extract model parameters (extremely difficult, extremely rare)

The easiest entry point is **Guardrail Jailbreak ($500-$1,000)** — but the technique must be novel. No roleplay, no encoding, no known techniques. Must be something new that bypasses safety training in a way 0din hasn't seen before.

---

## Strategy Notes

- Focus on novel prompt injection techniques — not rehashed jailbreaks from Twitter/GitHub
- Indirect injection is less saturated than direct jailbreaks — injecting malicious instructions through external data sources the model processes
- Agentic models (ones with tool use) have larger attack surface than pure chat models
- Claude for Chrome is interesting — browser agent with access to web content = indirect injection surface
- Test on multiple models once you find a technique — document all affected models in submission
- Abstract submission first saves time — don't write a full PoC for something out of scope

---

## Resources

- Scope page: https://0din.ai/scope/
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- MITRE ATLAS: https://atlas.mitre.org
- Embrace The Red (indirect injection blog): embracethered.com
- Wraith Academy (free AI bounty training): wraith.sh
