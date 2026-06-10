# AI Models and Data

**THM Room:** AI Models and Data  
**Path:** AI Security  
**Status:** In Progress

---

## Core Concept
Security risks in AI systems don't begin at deployment. They begin in the data supply chain — often invisible, poorly documented, and almost entirely unaudited.

---

## Where Training Data Comes From

| Source | Trust Level | Notes |
|---|---|---|
| **Web scraping** | Low | No curator, no version control, content changes after collection |
| **Licensed datasets** | Medium | Terms often unclear, original users rarely consented to AI training use |
| **Synthetic data** | Variable | ~12% of fine-tuning datasets now contain LLM-generated content |
| **Internal corpora** | Higher | Organization has control but also direct liability if mishandled |

**Common Crawl** — the most widely used public training corpus, underpins essentially every major model family. GPT-3 was 60% Common Crawl. DeepSeek-V3 trained on 14.8T tokens using it. LLaMA 4 scaled to 40T tokens across 200 languages.

---

## Data Provenance

The ability to answer three questions about any piece of training data:
1. Where did it come from?
2. When was it collected?
3. Has it been modified since?

**The problem:** Most models are trained on datasets of datasets — original attribution is lost or never recorded.

**Data Provenance Initiative audit findings:**
- 70%+ of licenses on popular hosting platforms listed as "Unspecified"
- 66% of labeled licenses were miscategorized — usually more permissive than reality

**ML-BOM** — Machine Learning Bill of Materials. The AI equivalent of a software SBOM. Documents dataset sources, licenses, PII categories, and filtering decisions. Adoption is still rare.

---

## PII in the Pipeline

Large-scale web scraping sweeps up personally identifiable information that gets baked into model weights. Once embedded, it cannot be patched out.

**Concrete example:** Truffle Security scanned the December 2024 Common Crawl archive (400TB, 2.67 billion pages) and found nearly **12,000 live, verified API keys and passwords**. A model trained on this data can sometimes be prompted to surface training content near-verbatim — including credentials.

GDPR requires data minimization. This conflicts directly with the "more data is always better" pre-training logic.

---

## Key Model-Building Concepts

### Epochs and Overfitting
- **Epoch** — one complete pass of the training algorithm through the entire dataset
- **Overfitting** — model memorizes training data instead of learning general patterns
- Security implication: overfitting increases the likelihood a model will reproduce specific sensitive details from training data when prompted

### Model Validation
- A validation set (held-back data never used for training) tests whether the model is generalizing or just memorizing
- If training accuracy climbs but validation accuracy plateaus — that's overfitting in real time
- Skipping thorough validation = unknown real-world behavior = security risk

### Pruning and Quantization (Post-Training Optimization)

| Technique | What It Does | Security Risk |
|---|---|---|
| **Pruning** | Removes low-contribution parameters to shrink model size | Changes model behavior post-training, rarely documented |
| **Quantization** | Reduces numerical precision of weights (e.g. 32-bit → 8-bit) | Can silently degrade safety-aligned behavior; backdoor defenses that worked on full-precision may fail on quantized versions |

Both are often applied by a third-party team packaging the model for distribution — after the original training is complete.

### Federated Learning
- Model trained across decentralized devices/organizations
- Each participant trains locally and sends only **weight updates** (not raw data) to a central server
- **Privacy benefit:** Data never leaves the participant
- **Security trade-off:** Participants can submit poisoned local updates (manipulated gradients) that are very difficult to detect at aggregation
- Solves one trust problem by distributing control, creates a different one

---

## Pre-Trained Models and Fine-Tuning

**Pre-trained model** — already trained on a large general-purpose dataset. Produced by a small number of well-resourced organizations (Meta LLaMA, OpenAI GPT, etc.)

**Fine-tuning** — continuing to train a pre-trained model on a smaller, task-specific dataset.

| What fine-tuning changes | What fine-tuning does NOT change |
|---|---|
| Task-specific behavior, tone, domain knowledge | Base model weights shaped by pre-training data you never audited |

---

## The Inheritance Problem

When you fine-tune a pre-trained model, you inherit everything beneath it:

**1. Safety alignment erodes, doesn't break**
- Stanford/Princeton research: safety mechanisms can be compromised by fine-tuning on as few as **10 adversarially crafted examples** at a cost of under $0.20
- Even benign fine-tuning on legitimate data degrades safety as a side effect
- Think of it like worn paths in a forest — new paths don't erase unsafe ones, they just make them less obvious

**2. Specialization increases attack surface**
- Cisco found fine-tuned models are measurably more susceptible to prompt injection than their base models
- Fine-tuning narrows focus, reducing resilience to unexpected inputs

**3. Version tracking is almost never done**
- Fine-tuning targets a specific checkpoint of a base model
- If that checkpoint contained a backdoor or problematic data, every derivative inherits it
- Without knowing the exact version, there's no way to assess exposure

---

## Models Are a Black Box

Trained model weights = billions of floating-point numbers with no human-readable record of what shaped them. You cannot open a model and find the decision that causes a specific behavior.

**What you can do:** Test behavior by sampling inputs, benchmarking, red teaming  
**What you cannot do:** Audit — testing tells you how the model behaved on inputs you tried, not what it will do on inputs you haven't thought of yet

---

## Model Cards

Structured documentation that accompanies a model — the closest thing the industry has to a transparency standard. Introduced by Google researchers in 2019.

| Section | What It Should Tell You |
|---|---|
| Training data | Sources used, filtering applied, known gaps or biases |
| Intended use | What the model was designed for (and what it wasn't) |
| Evaluation results | Performance across conditions and demographics |
| Known limitations | Where the model underperforms or behaves unexpectedly |
| Bias assessment | Where training or evaluation introduced skew |
| License | What you're legally permitted to do with the model |

**The gaps:** Model cards are voluntary — no regulatory requirement. Incentive to be thorough is weak when disclosing limitations reduces adoption. A sparse or missing model card is a warning sign, not an inconvenience.

---

## Key Takeaways

- AI training data comes from poorly documented, unaudited sources — most organizations have no reliable answer to what their training data contained
- PII and live credentials routinely end up baked into model weights and cannot be patched out post-deployment
- Quantization and federated learning introduce security trade-offs that are rarely documented
- Fine-tuning inherits everything from the base model — safety alignment erodes with as few as 10 adversarial examples
- Model weights are fundamentally opaque — security testing samples behavior, it doesn't audit it
- Model cards are the primary transparency mechanism but remain voluntary, frequently incomplete, and sometimes absent

---

## Security Checklist — Evaluating a Third-Party Model

- [ ] Does a model card exist?
- [ ] Is training data documented with sources and filtering decisions?
- [ ] Is there an ML-BOM or equivalent dataset inventory?
- [ ] Is the specific base model checkpoint and version documented?
- [ ] Were quantization or pruning steps applied, and are they documented?
- [ ] Has safety alignment been tested post-fine-tuning?
- [ ] Is the license clearly stated and appropriate for your use case?
- [ ] Has the model been red teamed for prompt injection?
