# Market Evidence Gate B1 Pack

## 1. Ideal Customer Profile (ICP)

| Attribute | Detail |
|-----------|--------|
| **Organization Type** | Enterprise (≥1000 employees) or regulated mid-market (200‑1000 employees) in finance, healthcare, government, critical infrastructure, or technology. |
| **AI Maturity** | Actively experimenting with or deploying agentic AI that can interact with infrastructure, enterprise applications, or OS (e.g., LLM‑driven automation, AI‑ops bots, autonomous agents). |
| **Pain Owner** | CISO, CIO/CTO, Head of AI Governance, Head of Platform Engineering, Head of Infrastructure, Head of IAM/PAM, AI Security Architect. |
| **Budget Authority** | Controls discretionary budget for security, governance, or AI risk mitigation initiatives (≥$50k annual). |
| **Current Toolchain** | Uses IAM/PAM, ITSM, EDR, SIEM, change‑management tools (ServiceNow, Jira Service Management), but lacks explicit controls for AI‑initiated privileged actions. |
| **Geography** | Global; preference for English‑speaking markets initially (US, EU, Canada, APAC). |
| **Trigger Events** | Recent AI pilot, regulatory guidance on AI governance, internal audit finding on uncontrolled AI actions, or a security incident involving agentic behavior. |

## 2. Core Hypotheses to Validate

| # | Hypothesis | What to Prove | Success Signal |
|---|------------|---------------|----------------|
| H1 | **Problem** | AI agents are already performing privileged or security‑sensitive actions that existing controls (IAM/PAM, ITSM, EDR) do not adequately capture, authorize, or audit. | ≥5 respondents confirm they have observed AI‑initiated privileged actions; ≥3 rate the problem as high severity (≥4/5). |
| H2 | **Buyer** | There exists a person with clear responsibility and budget for governing AI‑initiated privileged changes. | ≥3 respondents identify a specific role (CISO, AI Governance Lead, etc.) that owns the problem and has budget influence. |
| H3 | **Urgency** | The problem requires a solution now or within the next 6‑12 months. | ≥4 respondents say they are actively looking for a solution or have a planned initiative in the next year. |
| H4 | **Differentiation** | Existing solutions (IAM/PAM, ITSM, EDR, MCP permissions) do not fully address the AI‑initiated intent → policy → human authority → bounded execution → evidence flow. | ≥3 respondents articulate a gap in current tooling for AI‑specific intent verification, policy enforcement tied to AI agents, or tamper‑evident evidence for AI actions. |
| H5 | **Willingness‑to‑Pay (WTP)** | Organizations would pay for a control layer that provides enforceable policy, approval workflow, and cryptographically verifiable evidence for AI‑initiated privileged changes. | ≥2 respondents express interest in a PoC, design partnership, or budget discussion; ≥1 indicates a specific budget range or willingness to allocate funds. |

## 3. Discovery Questions (10)

Ask in order; adapt tone for interview or survey.

1. **Context** – “Can you describe how your organization currently uses AI agents or LLMs to interact with Windows systems, cloud infrastructure, or enterprise applications?”  
2. **Observation** – “Have you observed an AI agent performing a privileged or security‑sensitive action (e.g., installing software, changing registry, modifying a service) without a clear human‑initiated ticket?”  
3. **Current Controls** – “How are such actions authorized, executed, and audited today? Which tools (IAM/PAM, ITSM, EDR, change‑management) are involved?”  
4. **Pain Points** – “What challenges do you face with the current process (e.g., lack of visibility, delayed approvals, insufficient audit trails, false positives/negatives)?”  
5. **Responsibility** – “Who is accountable for ensuring AI‑initiated changes are proper and compliant? What is their title and what budget do they control?”  
6. **Urgency** – “How urgent is it to improve control over AI‑initiated privileged actions? Is there a timeline, regulatory pressure, or planned initiative?”  
7. **Gap Analysis** – “Where do your existing controls fall short when dealing specifically with AI agents (e.g., inability to verify AI intent, enforce policies tied to agent identity, generate tamper‑evident evidence)?”  
8. **Solution Expectations** – “What capabilities would you expect from a dedicated control layer for AI‑initiated privileged changes? (Think policy engine, approval workflow, execution broker, evidence bundle.)”  
9. **Interest Level** – “Would you be interested in reviewing a proof‑of‑concept that demonstrates request → policy decision → plan → evidence without actual system changes?”  
10. **Next Steps** – “If the PoC addresses your needs, what would be the next step (design partnership, pilot, budget discussion) and what timeline would you expect?”

## 4. Scoring & Evidence Criteria

Each respondent yields a score per hypothesis.

| Hypothesis | Scoring (0‑2 per respondent) | Pass Threshold (≥) |
|------------|------------------------------|--------------------|
| H1 – Problem | 0 = no observation; 1 = observed low‑severity; 2 = observed high‑severity | Problem Confirmed ≥5 **AND** Problem High Severity ≥3 |
| H2 – Buyer | 0 = no clear owner; 1 = owner identified but no budget authority; 2 = owner with budget authority | Identifiable Buyer ≥3 |
| H3 – Urgency | 0 = not urgent; 1 = useful but not urgent; 2 = actively seeking solution now/≤12 mo | Follow‑up Requested ≥2 (derived from Q6 & Q10) |
| H4 – Differentiation | 0 = no gap perceived; 1 = minor gap; 2 = clear, significant gap | Existing Solution Gap ≥3 |
| H5 – WTP | 0 = no interest; 1 = interested in learning; 2 = interested in PoC/design partnership/budget talk | PoC Interest ≥1 **OR** Design Partner Interest ≥1 **OR** Budget Discussion ≥1 |

### Gate B1 Decision Rules

- **PASS** (Proceed to Gate 2 – Canonical Operation Model) if **all** five hypothesis thresholds are met.
- **PIVOT** (Refine ICP/hypotheses, repeat discovery) if **at least three** thresholds are met but one or two are just below (e.g., 4/5 for Problem Confirmed) – adjust targeting or messaging.
- **STOP** (Cease investment in this direction) if **fewer than three** thresholds are met or if critical hypotheses (H1 Problem, H2 Buyer, H5 WTP) fail.

### Evidence Log Template (per respondent)

Create a Markdown file under `evidence/market_interviews/` named `YYYY-MM-DD_<organization>_<role>.md`.

```markdown
# Market Interview Evidence

- **Date**: YYYY-MM-DD
- **Organization**: <Name, size, industry>
- **Interviewee**: <Name, Title>
- **Interaction Type**: Interview / Survey
- **ICP Match**: Yes/No (explain deviations)
- **Responses**:
  1. <Answer to Q1>
  2. <Answer to Q2>
  3. <Answer to Q3>
  4. <Answer to Q4>
  5. <Answer to Q5>
  6. <Answer to Q6>
  7. <Answer to Q7>
  8. <Answer to Q8>
  9. <Answer to Q9>
  10. <Answer to Q10>
- **Hypothesis Scoring**:
  - H1 Problem: <0‑2> (Observed: <yes/no>, Severity: <low/medium/high>)
  - H2 Buyer: <0‑2> (Owner: <role>, Budget authority: <yes/no>)
  - H3 Urgency: <0‑2> (Timeline: <now/6‑12mo/>later/not urgent>)
  - H4 Differentiation: <0‑2> (Gap description)
  - H5 WTP: <0‑2> (Interest level, next step suggested)
- **Overall Gate B1 Verdict**: PASS / PIVOT / STOP (with brief justification)
- **Attachments**: (recording link, notes, consent form if applicable)
```

## 5. Outreach Templates

### English (Initial LinkedIn/Email)

Subject: Quick research on governing AI‑initiated privileged changes – 5‑minute input?

Hi [First Name],

I’m Krzysztof Skomra, working on AI governance & control infrastructure at Hermes Agent Commons. We’re researching how enterprises manage privileged actions initiated by AI agents (e.g., software installs, registry changes) and whether existing controls (IAM/PAM, ITSM, EDR) are sufficient.

Would you be open to a brief 5‑minute call or asynchronous answers to 10 short questions? Your insight would help shape a solution that gives organizations verifiable control over AI‑driven system changes.

No pitch, no commitment—just learning from peers like you.

If interested, please reply with a convenient time or simply answer the questions attached.

Thanks,
Krzysztof Skomra
[LinkedIn] | [Email]

### Polish (for local outreach)

Temat: Krótkie badania dotyczące kontroli uprzywilejowanych działań AI – 5 minut Twojego czasu?

Cześć [Imię],

Nazywam się Krzysztof Skomra i pracuję nad infrastrukturą kontroli i zarządzania AI w Hermes Agent Commons. Badajemy, jak przedsiębiorstwa zarządzają uprzywilejowanymi działaniami inicjowanymi przez agenty AI (np. instalacja oprogramowania, zmiany rejestru) oraz czy obecne narzędzia (IAM/PAM, ITSM, EDR) są wystarczające.

Czy byłbyś/abyś otwarty/a na krótką 5‑minutową rozmowę lub odpowiedzi na 10 prostych pytań? Twoja perspektywa pomoże nam stworzyć rozwiązanie zapewniające weryfikowalną kontrolę nad zmianami systemowymi wykonywanymi przez AI.

 bez prezentacji, bez zobowiązań – tylko wymiana doświadczeń.

Jeśli jesteś zainteresowany/a, daj znać odpowiedni termin lub po prostu odpowiedz na załączone pytania.

Pozdrawiam,
Krzysztof Skomra
[LinkedIn] | [E‑mail]

## 6. Next Steps After Gate B1

If **PASS**:
1. Synthesize interview findings into a Market Evidence Report (evidence/market_evidence_report.md).
2. Proceed to **Gate 2 – Canonical Operation Model** to formalize the WCC‑MCP API based on validated canonical operations (SYSTEM_INVENTORY, APPLICATION_INVENTORY, PLAN_APPLICATION_INSTALL, etc.).
3. Begin development of a **read‑only MCP** that enforces policy and generates evidence bundles (Product Evidence v0.2+).
4. Prepare a lightweight PoC demo for interested respondents (design partners).

If **PIVOT**:
- Adjust ICP/hypotheses based on feedback.
- Refine outreach messaging and question set.
- Repeat discovery with a new cohort (target 5‑7 additional respondents).

If **STOP**:
- Document learnings in a Decision Log (decisions/market_evidence_stop.md).
- Redirect efforts to alternative use‑cases or technology areas.

--- 

*End of Gate B1 Pack. All artifacts should be stored under the `evidence/` directory of the workspace.*