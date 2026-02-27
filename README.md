# Internal Mobility "Opportunity Broker" Agent for HR

An AI-powered agent built on **Azure AI Foundry** that matches employees to internal gigs, open roles, and growth opportunities — reducing regrettable attrition and accelerating internal talent mobility.

## Problem

- Employees leave because they can't discover internal opportunities that fit their skills.
- Managers hoard talent; HR spends weeks manually matching people to roles/projects.
- No single system bridges employee aspirations, skill gaps, and available opportunities.

## What the Agent Does

1. **Employee Intake** — Parses a resume/profile + career interests into a structured skills graph.
2. **Skill Normalization** — Maps skill variants to canonical forms (e.g., "SRE" → "Site Reliability Engineering", "K8s" → "Kubernetes") using a synonym map.
3. **RAG Matching** — Uses Azure AI Search (vector + hybrid) to match employees against internal gigs and open roles, scoring each on skill overlap, location fit, and constraints.
4. **Growth Plan Generation** — Produces a 30/60/90-day plan with courses, mentoring, and milestones to close skill gaps.
5. **Manager Release Brief** — Generates a tactful, data-backed brief explaining why releasing the employee benefits the org (includes backfill suggestions and flight-risk projections).
6. **HR & Employee Messages** — Drafts HR process emails, motivating employee messages, and flight-risk reduction projections.

## Architecture

```
┌─────────────────┐
│ Employee Intake  │  ← employee_id or free-text profile
└────────┬────────┘
         ▼
┌─────────────────┐
│ Skill Normalize  │  ← synonym map (15+ mappings)
└────────┬────────┘
         ▼
   ┌─────┴─────┐
   ▼           ▼
┌──────┐  ┌──────┐
│ Gig  │  │ Role │   ← Azure AI Search (vector + hybrid)
│Match │  │Match │
└──┬───┘  └──┬───┘
   └─────┬───┘
         ▼
┌─────────────────┐
│ Growth Plan Gen  │  ← learning catalog + skill gaps
└────────┬────────┘
         ▼
┌─────────────────┐
│ Manager Brief    │  ← release friction, backfill, flight risk
└────────┬────────┘
         ▼
┌─────────────────┐
│ HR/Employee Msgs │  ← emails, next steps, risk projection
└─────────────────┘
```

**Orchestration:** Azure AI Foundry Prompt Flow (DAG-based, 7 nodes)
**LLM:** GPT-4o (reasoning, matching, generation)
**Embeddings:** text-embedding-3-large (vector search)
**Search:** Azure AI Search (hybrid: keyword + vector)

## Project Structure

```
├── README.md
├── plan-internalMobilityAgentForHr.prompt.md   # 5-day hackathon plan
├── internal_mobility_agent_for_hr.md           # Project brief, data schemas, demo personas
│
├── v1_employees.csv          # 24 employees (base dataset)
├── v1_internal_gigs.csv      # 12 gigs
├── v1_open_roles.csv         # 8 roles
├── v1_learning_catalog.csv   # 16 courses
├── v1_skill_synonyms.csv     # 15 synonym pairs
├── v2_employees.csv          # Extended schema (22 cols, needs generation)
├── v2_internal_gigs.csv      # Extended schema (13 cols)
├── v2_open_roles.csv         # Extended schema (12 cols)
├── v2_mobility_history.csv   # 8 sample events
├── v3_employees.csv          # 12 rows with v2 schema populated
│
└── mobility-broker-agent/          # Prompt Flow project
        ├── flow.dag.yaml               # Top-level flow definition
        ├── requirements.txt            # Python dependencies
        ├── connections/
        │   └── connections.yaml        # Azure OpenAI + AI Search connections
        ├── data/                       # Runtime CSV data
        │   ├── employees.csv
        │   ├── internal_gigs.csv
        │   ├── open_roles.csv
        │   ├── learning_catalog.csv
        │   ├── mobility_history.csv
        │   └── skill_synonyms.csv
        ├── evaluation/
        │   ├── eval_config.yaml        # Evaluation metrics config
        │   └── ground_truth_test_cases.jsonl  # 10 ground-truth test cases
        └── flows/
            └── mobility-broker-agent/
                ├── flow.dag.yaml                   # Detailed Prompt Flow DAG
                ├── node_employee_intake.py          # Parse employee profile
                ├── node_skill_normalization.py       # Normalize skills via synonym map
                ├── node_lookup_gigs.py               # Azure AI Search query for gigs
                ├── node_lookup_roles.py               # Azure AI Search query for roles
                ├── node_rag_matching_gigs.py          # LLM scoring of gig matches
                ├── node_rag_matching_roles.py         # LLM scoring of role matches
                ├── node_growth_plan_generator.py      # 30/60/90-day plan
                ├── node_manager_release_brief.py      # Manager negotiation brief
                ├── node_hr_employee_messages.py       # HR/employee communications
                └── prompts/                           # Jinja2 prompt templates
                    ├── employee_intake.jinja2
                    ├── skill_normalization.jinja2
                    ├── rag_matching_gigs.jinja2
                    ├── rag_matching_roles.jinja2
                    ├── growth_plan_generator.jinja2
                    ├── manager_release_brief.jinja2
                    └── hr_employee_messages.jinja2
```

## Synthetic Dataset

| File | Rows | Description |
|---|---|---|
| `employees.csv` | 24 (v1) / 150 target | Employee profiles with skills, interests, constraints, flight risk, pay band |
| `internal_gigs.csv` | 12 (v1) / 60 target | Short-term internal projects (10–50% time commitment) |
| `open_roles.csv` | 8 (v1) / 30 target | Full-time open positions with level/pay band requirements |
| `learning_catalog.csv` | 16 (v1) / 80 target | Courses for skill gap remediation |
| `mobility_history.csv` | 8 (v2) / 300+ target | Application history with outcomes and decline reasons |
| `skill_synonyms.csv` | 15 | Canonical mapping for skill normalization |

### Key Data Fields (v2 Schema)

- **`pay_band`** — PB3–PB8, correlated with level (L1→PB3 … L5→PB7–PB8)
- **`flight_risk_1to5`** — Synthetic proxy correlated with aspiration + friction + time since promotion
- **`manager_release_friction_1to5`** — How likely a manager is to block the move
- **`mobility_eligibility`** — Policy: tenure ≥ 12 months AND performance_band ∈ {A, B}
- **`aspiration_track`** — IC / Manager / Product / Data / Security / Design / GTM

## Prerequisites

- **Azure Subscription** with access to:
  - Azure AI Foundry (Standard tier)
  - Azure OpenAI (GPT-4o + text-embedding-3-large deployments)
  - Azure AI Search (Basic tier)
- **Python 3.10+** (for local development/testing)

## Setup

### 1. Azure AI Foundry

1. Create a project at [ai.azure.com](https://ai.azure.com) → **+ New project** → `mobility-broker-hackathon`
2. Deploy models under **Models + endpoints**:
   - `gpt-4o` (30K TPM)
   - `text-embedding-3-large` (30K TPM)
3. Upload CSV data files under **Data** → **+ New data**

### 2. Azure AI Search Index

1. Create index `mobility-index` with vector + hybrid search enabled
2. Use `text-embedding-3-large` as the embedding model
3. Index all CSV datasets (employees, gigs, roles, learning catalog)

### 3. Local Development

```bash
# Clone the repository
git clone <repo-url>
cd internal_mobility_agent_for_hr

# Install dependencies
pip install -r tst.py/mobility-broker-agent/requirements.txt

# Update connections (connections/connections.yaml) with your Azure endpoints and keys
```

### 4. Run the Prompt Flow

Use Azure AI Foundry Prompt Flow UI or the `promptflow` CLI:

```bash
# Test locally
pf flow test --flow tst.py/mobility-broker-agent/flows/mobility-broker-agent \
  --inputs employee_id="E007" goal="role_change" constraints="Needs fixed shift" timeline="next_quarter"
```

## Demo Personas

| # | Employee | Current Role | Best Role Match | Best Gig Match | Story |
|---|---|---|---|---|---|
| 1 | E007 Kunal | Support Specialist | R003 TAM | G005 Ticket Triage Pilot | Cross-function mobility |
| 2 | E012 Shreya | Tech Writer | R006 Dev Advocate | G001 Developer Portal | Career track change |
| 3 | E015 Aditya | Data Engineer | R002 Analytics Engineer | G011 Metrics Layer | Within-function level-up |
| 4 | E005 Vikram | DevOps Engineer | R001 Platform Eng / R007 SRE | G010 Runbook Standardization | SRE leadership path |
| 5 | E002 Diya | Frontend Engineer | — | G004 Accessibility Audit + G001 Portal | Skill expansion via gigs |
| 6 | E009 Arjun | ML Engineer | — | G005 Auto-Triage | Perfect gig fit |
| 7 | E016 Neha | Sales Ops Analyst | — | G012 CRM Automation | Non-engineering mobility |
| 8 | E018 Ritika | Data Scientist | R008 PM (Onboarding) | G003 Experimentation Quality | Cross-track aspiration |
| 9 | E019 Manish | Android Engineer | — | G009 Mobile Component Library | Platform contribution |
| 10 | E013 Rahul | Security Analyst | R005 DevSecOps | G007 Threat Modeling Pack | Security specialization |

## Evaluation

Evaluation is configured in `evaluation/eval_config.yaml` with three metrics (threshold ≥ 4/5):

| Metric | What It Measures |
|---|---|
| **Groundedness** | Are match reasons grounded in actual employee/gig/role data? |
| **Relevance** | Do top 3 matches align with expected ground-truth personas? |
| **Coherence** | Are growth plans logical, actionable, and well-structured? |

Ground truth test cases are in `evaluation/ground_truth_test_cases.jsonl` (10 cases).

## Guardrails

- **Content Safety** — Azure AI Content Safety filters bias, PII, and protected-attribute leakage
- **Fairness Rule** — System prompts explicitly prohibit matching based on gender, age, religion, or disability
- **Transparency** — Every match includes "Matched because…", "Gaps to address…", and "Risk flags…"
- **Employee Control** — Users can edit their skills/interests and re-run matching

## Business Impact (ROI Model)

| Metric | Value |
|---|---|
| Company size | 500 employees |
| Annual attrition | 10% (50 exits/year) |
| Regrettable & preventable via mobility | 3 exits avoided/year |
| Replacement cost per engineer | ₹8–15L |
| **Annual savings from attrition reduction** | **₹24L–₹45L** |
| Time-to-fill reduction per internal fill | 20 days |
| Internal fills per year | 10 roles |
| **Productivity gain** | **200 person-days/year** |

## Azure Services Used

| Service | Purpose | SKU |
|---|---|---|
| Azure AI Foundry | Project hub, Prompt Flow, Evaluation | Standard |
| Azure OpenAI | GPT-4o (reasoning) + text-embedding-3-large (vectors) | Standard, 30K TPM |
| Azure AI Search | Vector + hybrid search over CSV data | Basic |
| Azure Blob Storage | CSV data hosting | Standard (auto-provisioned) |
| Azure AI Content Safety | Bias/PII guardrails | Standard |
| Azure Key Vault | Secrets management | Standard (auto-provisioned) |

## License

This project is licensed under the MIT License.
