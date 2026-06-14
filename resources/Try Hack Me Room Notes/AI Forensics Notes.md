# AI in Digital Forensics (DFIR)

**Context:** How AI/ML is being applied to Digital Forensics and Incident Response — capabilities, limitations, and legal/ethical considerations.

---

## Core AI Capabilities Relevant to DFIR

**Data Processing** — Transformer models process entire bodies of text in parallel (milliseconds). Invaluable for processing the massive data volumes inherent to forensic investigations.

**Anomaly Detection** — ML algorithms learn what "normal" looks like for users, systems, and networks. Supervised and unsupervised models flag deviations that could indicate malicious behavior — turning a haystack into a handful of hay.

**Scalability** — Modern infrastructure spans cloud, hybrid, and remote endpoints generating more forensic data than ever. AI scales effortlessly across distributed environments without proportional increases in analyst workload.

---

## AI/ML Applied Across DFIR Tasks

| Task | What AI Enables | How |
|---|---|---|
| Anomaly Detection / UEBA | Flags unusual user/system behavior vs learned baseline | Unsupervised learning (Isolation Forests, Autoencoders) learns normal — deviations flagged |
| Phishing Detection | Classifies emails as phishing or legitimate | Transformer models (BERT, RoBERTa) analyze tone, structure, known attack patterns — 99% accuracy in studies |
| Malware Classification | Classifies files as malicious or benign | Trained on malware corpora — analyzes metadata, code signatures, sandbox behavior |
| Alert Triage | Scores, ranks, and filters alerts | Analyzes past alert data and outcomes to surface highest-priority issues first |
| Timeline Reconstruction | Correlates logs across sources into attack timelines | Clusters similar events, identifies causal relationships, aligns activity across systems |

**Example tools:** Splunk UEBA, Elastic ML, Exabeam, Microsoft Defender, Timesketch, Velociraptor, Cortex XSOAR

---

## Key Concepts

### Probabilistic vs Deterministic
Traditional software: same input → same output (deterministic)  
AI/ML: same input → potentially different output (probabilistic)

Non-determinism allows AI to handle uncertainty and adapt to new patterns — valuable capability. But forensics demands consistent and repeatable results. This tension requires careful consideration when using AI in legal contexts.

### Accuracy, Precision, and Recall

Never evaluate model performance on one metric alone — especially with imbalanced data (few malicious files among thousands of benign ones).

**Accuracy** — overall rate of correct predictions. Misleading in isolation. A model that always predicts "benign" achieves 99% accuracy if 99 of 100 files are benign — but it's useless.

**Precision** — of all results flagged as positive, how many actually were? High precision = fewer false positives = less time chasing irrelevant leads. Risk: model becomes too selective and misses real threats.

**Precision = True Positives / (True Positives + False Positives)**

**Recall** — of all actual positives, how many did the model find? High recall = fewer missed threats. Risk: model casts wide net and flags everything, generating massive false positive noise.

**Recall = True Positives / (True Positives + False Negatives)**

Use all three together for a complete picture of model performance.

### GIGO (Garbage In, Garbage Out)
Quality of AI output is directly determined by quality of training data. AI trained on biased or incomplete data will confidently generate wrong conclusions. Critical in forensics where findings may pursue justice.

---

## AI Applications in Specific DFIR Areas

### Image and Video Forensics

**CNN (Convolutional Neural Networks)** — learn spatial patterns in data using small filters. Primary tool for image forensics.

- **ELA + CNN forgery detection** — Error Level Analysis highlights compression inconsistencies, CNN classifies regions as forged or authentic. ~94% accuracy.
- **Deepfake detection** — CNN models analyze subtle facial video inconsistencies. Achieves state-of-the-art deepfake identification accuracy.
- **GANs (Generative Adversarial Networks)** — two competing networks: one generates fakes, one detects them. Both improve through competition. Used offensively to create deepfakes, defensively to train detectors. Arms race between both sides.

### Communication Analysis

**NLP (Natural Language Processing)** using Transformer models (BERT, RoBERTa):
- Phishing email classification — 99% accuracy in studies vs rule-based approaches
- Chat/social media scanning — keyword and pattern detection across massive datasets
- Sentiment analysis — gauges emotional tone of communications
- Moves from rule-based (looking for known bad URLs/keywords) to context-aware deep learning

### Timeline Reconstruction

- ML ingests logs, filesystem timestamps, network records simultaneously
- Builds chronological event timelines automatically
- Merges server logs, firewall alerts, and application logs into unified view
- Particularly valuable when attacker has deliberately altered logs to obscure activity

### Malware Detection/Analysis

- **STAMINA (Microsoft/Intel)** — represents malware files as images processible by deep neural networks, classifies as malicious or benign
- **Dynamic analysis** — converts API call sequences into 2D images, classifies based on behavioral patterns
- Now standard in most AV and EDR products

---

## Legal and Ethical Considerations

### Explainability and Transparency

Many AI models are "black boxes" — they don't explain how they reached a conclusion. This conflicts with forensics' core requirement for defensible evidence interpretation.

**Real case:** LLM flagged emails as suspicious in civil litigation. Opposing counsel demanded explanation of the model's reasoning. Legal team couldn't explain it. AI-generated evidence was excluded by the court.

**Daubert test** — U.S. legal standard for admissibility of expert/scientific testimony. AI-generated evidence must meet this standard. Without explainability, it likely won't.

**Implication:** AI findings must be validated by human experts who can explain the reasoning in court.

### Bias and Fairness

ML models trained on historical data inherit biases from that data. In forensics, bias can influence what evidence is prioritized or what conclusions are drawn.

**Real example:** Facial recognition used by law enforcement misidentifies Black and minority individuals at significantly higher rates than white individuals. At least 7 known wrongful arrests in the U.S. due to faulty facial recognition — almost all victims were African American.

**Legal implication:** If defense can show AI technique is biased, judges may exclude its results.  
**Ethical duty:** Forensic experts must validate and correct biases — diverse training data, bias mitigation techniques.

### Accountability and Chain of Custody

Courts require digital evidence to be handled in a traceable manner with integrity preserved at each step — chain of custody and audit trail.

**Real case:** LLM used to summarize a suspect's mobile phone data inadvertently violated chain of custody because intermediate AI outputs weren't logged. Defense successfully challenged forensic findings on procedural grounds.

**Requirements:**
- AI processes must be carefully documented
- On-premises or controlled systems preferred over cloud-based services
- All intermediate AI outputs must be logged

### Privacy and Data Protection

AI models require large datasets. Using them in investigations can trigger privacy compliance issues:

- Cloud servers may expose sensitive evidence to third parties
- GDPR may restrict how personal data is processed even for law enforcement
- Evidence obtained without proper authority may be ruled inadmissible

**Mitigation:** Run AI tools in secure offline environments or use federated learning.

**Federated Learning** — ML performed without transferring sensitive data to a central server. Model trains locally on distributed data and shares only model updates (not raw data). Preserves privacy while enabling AI capability.

---

## Key Takeaway

AI is not a replacement for human expertise in DFIR — it is a force multiplier. The combination of AI's ability to process massive datasets and identify patterns at scale, combined with human judgment, domain expertise, and legal accountability, is what makes effective forensic investigation possible.

AI findings always require human validation. The RobbCo case study demonstrated this explicitly — the AI correctly flagged malicious files AND incorrectly flagged legitimate proprietary source code as suspicious. Without human insight, both findings would have been treated the same.

---

## Case Study — RobbCo Breach Summary

**Attack chain reconstructed using AI-assisted forensics:**

```
1. Phishing email sent to j.morgan (attacker: akeane@poseidonenergy.net)
2. Malicious invoice_Q1_2075.ods opened — embedded macro harvests:
   - bash_history, SSH keys, active sessions, /etc/passwd
   - Data dumped to /tmp/invoice_dump.txt
   - Exfiltrated to 192.168.0.100
3. Attacker logs in as j.morgan at 03:01:02 using harvested credentials
4. First reverse shell deployed: /tmp/.x → 10.0.0.66:4444
5. Second stage dropper: /tmp/.syncd → retrieves payload.sh from 10.0.0.66
6. Privilege escalation: sudo nano /home/r.house/.ssh/authorized_keys
   (SSH key planted giving access to r.house account)
7. Persistence established: /usr/local/bin/sysmon (fake monitoring tool)
   Supported by fabricated: /opt/robbco/sys/boot_monitor.log
8. Source code stolen:
   - RETROS BIOS: /opt/robbco/firmware/RETROS_BIOS/core.asm
   - MF Boot Agent: /opt/robbco/engineering/MFBootAgent/mfboot_main.c
   - Compressed and encoded: /dev/shm/.core_dump_2025.tgz.enc
   - Staged for exfiltration from shared memory
```

**AI tools used:** scikit-learn classify_logs.py and file_anomalies.py  
**Human validation required:** AI correctly flagged malicious files but also incorrectly flagged legitimate proprietary source code — human review confirmed which findings were real.
