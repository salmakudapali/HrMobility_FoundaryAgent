# Azure AI Foundry Mobility Agent — 5-Day Hackathon Plan

## TL;DR

Build an **Internal Mobility "Opportunity Broker" Agent** using Azure AI Foundry portal (no local code). The agent takes an employee profile, matches them to internal gigs/roles via RAG over your CSV data (indexed in Azure AI Search), and generates growth plans + manager briefs using GPT-4o orchestrated in Prompt Flow. The 5-day plan compresses the original 7-day MVP, prioritizing demo-ready outputs. Data gaps (v2/v3 schemas are mostly empty) must be filled on Day 1 using the generation recipe in the markdown.

## Key Findings from Data Research

### Usable data now
| File | Rows | Columns |
|---|---|---|
| `v1_employees.csv` | 24 | 16 |
| `v1_internal_gigs.csv` | 12 | 11 |
| `v1_open_roles.csv` | 8 | 9 |
| `v1_learning_catalog.csv` | 16 | 7 |
| `v1_skill_synonyms.csv` | 15 | 2 |

### Schema-only (no data — need generation)
| File | Columns | New Fields vs v1 |
|---|---|---|
| `v2_employees.csv` | 22 | `pay_band`, `last_promotion_months_ago`, `manager_release_friction_1to5`, `aspiration_level`, `aspiration_track`, `flight_risk_1to5`, `mobility_eligibility` |
| `v2_internal_gigs.csv` | 13 | `time_zone_overlap`, `sensitive_team` |
| `v2_open_roles.csv` | 12 | `pay_band_min`, `pay_band_max`, `interview_complexity` |

### Partial data
| File | Rows | Notes |
|---|---|---|
| `v3_employees.csv` | 12 | Enriched with v2 schema (22 cols), first 12 employees only |
| `v2_mobility_history.csv` | 8 | 4 Accepted, 4 Declined (reasons: Timing, Manager Block, Eligibility, Skill Gap) |

### Critical observations
- v2/v3 schemas add HR-critical fields (`pay_band`, `flight_risk`, `mobility_eligibility`, `manager_release_friction`) essential for a compelling demo
- 10 demo personas in `internal_mobility_agent_for_hr.md` are pre-mapped to expected matches — these become evaluation ground truth
- Skill synonyms (15 rows) enable normalization layer critical for matching (e.g., "SRE" ↔ "DevOps" → "Site Reliability Engineering")

---

## Day 1: Azure AI Foundry Setup + Data Preparation

### Step 1.1 — Create Azure AI Foundry Hub & Project
1. Go to [https://ai.azure.com](https://ai.azure.com) → sign in with your Azure subscription
2. Click **"+ New project"** → name it `mobility-broker-hackathon`
3. Under **Hub**, either select an existing hub or create a new one:
   - Hub name: `mobility-hub`
   - Region: **East US 2** or **Sweden Central** (best GPT-4o availability)
   - Pricing: Standard
4. This auto-provisions: Azure OpenAI resource, Azure AI Search, Azure Storage Account, Key Vault
5. Once created, note the **Project overview** page — you'll work from here

### Step 1.2 — Deploy Models (Azure OpenAI)
1. In your AI Foundry project → **Models + endpoints** → **+ Deploy model** → **Deploy base model**
2. Deploy **GPT-4o** (version 2024-08-06 or latest):
   - Deployment name: `gpt-4o`
   - TPM: 30K (hackathon scale is fine)
   - This handles: matching reasoning, plan generation, manager briefs
3. Deploy **text-embedding-3-large**:
   - Deployment name: `text-embedding-3-large`
   - TPM: 30K
   - This handles: vectorizing employee skills, gig/role descriptions for RAG

### Step 1.3 — Generate Full Synthetic Dataset
1. In the AI Foundry **Playground** (Chat), use GPT-4o to generate the remaining data:
   - Prompt it with the v2 schema headers from `v2_employees.csv` + the 12 populated rows from `v3_employees.csv` as examples + the distribution recipe from `internal_mobility_agent_for_hr.md`:
     - Locations: Bengaluru 45%, Hyderabad 20%, Pune 15%, Gurugram 10%, Mumbai 7%, Remote(India) 3%
     - Role families: Engineering 55%, Data 15%, Product 10%, Design 7%, Security 5%, Support 5%, HR/Ops/Finance 3%
     - Levels: L1→PB3, L2→PB4, L3→PB5, L4→PB6, L5→PB7–PB8
     - Mobility eligibility: tenure ≥ 12 months AND performance_band ∈ {A, B}
     - Flight risk: correlated with (high aspiration + high friction + long time since promotion)
   - Generate employees E013–E150 in batches of 20–30
   - Generate 60 internal gigs using `v2_internal_gigs.csv` schema (extend from the 12 in v1)
   - Generate 30 open roles using `v2_open_roles.csv` schema (extend from the 8 in v1)
   - Generate 300+ mobility history events (outcome distribution: 30% Accepted, 50% Declined, 20% Pending)
   - Expand `v1_learning_catalog.csv` from 16 → 80 courses
2. Save all generated CSVs locally and validate consistency:
   - Employee IDs referenced in history must exist
   - Skill names should use the synonym map
   - Pay bands must align with levels

### Step 1.4 — Upload Data to Azure AI Foundry
1. In your project → **Data** → **+ New data** → **Upload files**
2. Upload all final CSVs: `employees.csv`, `internal_gigs.csv`, `open_roles.csv`, `learning_catalog.csv`, `mobility_history.csv`, `skill_synonyms.csv`
3. These land in the project's Azure Blob Storage container automatically

---

## Day 2: Azure AI Search Index + RAG Pipeline

### Step 2.1 — Create Azure AI Search Index (Vectorized)
1. In AI Foundry project → **Indexes** → **+ New index**
2. Source: select the uploaded CSV data files
3. Index configuration:
   - Index name: `mobility-index`
   - Select **Azure OpenAI** → linked embedding model: `text-embedding-3-large`
   - Enable **vector search** + **hybrid search** (keyword + vector)
   - Chunk size: Since CSVs are row-based, each row becomes a document. Configure the indexer to treat each CSV row as one searchable document
4. For **employees**: index fields — `employee_id` (key), `full_name`, `skills`, `tools`, `career_interests`, `constraints`, `role_family`, `level`, `location`, `aspiration_track`, `mobility_eligibility`, `flight_risk_1to5`
5. For **internal_gigs**: index fields — `gig_id` (key), `title`, `required_skills`, `nice_to_have_skills`, `description`, `department`, `remote_allowed`, `time_percent`
6. For **open_roles**: index fields — `role_id` (key), `title`, `required_skills`, `nice_to_have_skills`, `role_summary`, `level`, `location`, `pay_band_min`, `pay_band_max`
7. For **learning_catalog**: index fields — `course_id` (key), `title`, `skill_tags`, `difficulty`, `duration_hours`, `format`
8. Run the indexer — wait for completion (should take < 5 minutes for this data volume)

### Step 2.2 — Build Skill Normalization Logic
1. Upload `v1_skill_synonyms.csv` as a **lookup enrichment** or handle it in the prompt:
   - **Option A (recommended for hackathon speed):** Include the full synonym map in the system prompt so GPT-4o normalizes skills at query time
   - Option B: Create a custom skill set in AI Search that maps variants to canonical forms during indexing
2. The 15 synonym rows are small enough to embed directly in the system prompt

### Step 2.3 — Test RAG Retrieval in Playground
1. Go to **Playground** → **Chat** → select `gpt-4o` deployment
2. Click **"Add your data"** → select the `mobility-index` you created
3. Test with: *"Find the best gig matches for employee E005 Vikram Singh who has CI/CD, Kubernetes, Terraform, Observability skills and wants SRE leadership"*
4. Verify it returns G010 (Runbook Standardization) as a top match — this validates your index
5. Test all 10 demo personas against expected matches:

| Employee | Expected Role | Expected Gig |
|---|---|---|
| E007 Kunal (Support) | R003 TAM | G005 ticket triage pilot |
| E012 Shreya (Tech Writer) | R006 Dev Advocate | G001 developer portal |
| E015 Aditya (Data Eng) | R002 Analytics Engineer | G011 metrics layer |
| E005 Vikram (DevOps) | R001 Platform Eng / R007 SRE | G010 runbooks |
| E002 Diya (Frontend) | — | G004 accessibility audit + G001 portal |
| E009 Arjun (ML) | — | G005 auto-triage |
| E016 Neha (Sales Ops) | — | G012 CRM automation |
| E018 Ritika (Data Scientist) | R008 PM onboarding | G003 experimentation quality |
| E019 Manish (Android) | — | G009 mobile component library |
| E013 Rahul (Security) | R005 DevSecOps | G007 threat modeling pack |

---

## Day 3: Prompt Flow Agent Orchestration

### Step 3.1 — Create the Prompt Flow
1. In AI Foundry project → **Prompt flow** → **+ Create** → **Chat flow**
2. Flow name: `mobility-broker-agent`
3. Design the flow with these nodes:

#### Node 1: Employee Intake (LLM node)
- **Input:** employee_id OR free-text profile + goal (role/gig/skill growth) + constraints + timeline + manager situation
- **System prompt:** Parse intake into structured JSON:
  ```json
  {
    "employee_id": "E007",
    "skills": ["Ticket Triage", "Communication", "Root Cause Notes"],
    "interests": "Technical Account Management",
    "constraints": "Needs fixed shift",
    "goal_type": "role_change",
    "timeline": "next_quarter",
    "manager_friction": 2
  }
  ```
- If employee_id provided, retrieve their full profile from the index

#### Node 2: Skill Normalization (LLM node)
- **Input:** parsed skills from Node 1
- **System prompt:** includes the full synonym map from `v1_skill_synonyms.csv`:
  ```
  Normalize these skills using the following synonym map:
  SRE → Site Reliability Engineering
  DevOps → Site Reliability Engineering
  Observability → Monitoring & Observability
  Grafana → Monitoring & Observability
  Prometheus → Monitoring & Observability
  API Docs → Technical Writing
  Docs-as-Code → Technical Writing
  A/B Testing → Experimentation
  Experiment Design → Experimentation
  Cost Optimization → FinOps
  WCAG → Accessibility
  Threat Modeling → Application Security
  AppSec → Application Security
  K8s → Kubernetes
  ```
- **Output:** normalized skill list using canonical names

#### Node 3: RAG Matching — Gigs (LLM + retrieval node)
- Query the `mobility-index` for internal gigs matching normalized skills
- System prompt instructs GPT-4o to:
  - Score each gig (0–100) based on skill overlap, location fit, time commitment fit
  - Flag skill gaps
  - Rank top 3 with explanations ("matched because X; gap: Y")
  - Check constraints (remote preference, location, on-call)

#### Node 4: RAG Matching — Roles (LLM + retrieval node)
- Same pattern as Node 3 but queries open roles
- Additional checks: pay band compatibility, level progression logic (L2→L3 ok, L2→L5 flag), interview complexity

#### Node 5: Growth Plan Generator (LLM node)
- **Input:** skill gaps identified in Nodes 3–4 + learning catalog results from index
- **System prompt:** Generate a 30/60/90-day plan:
  - **Day 1–30:** Quick wins — courses from learning catalog for immediate gaps, shadow/mentor pairing
  - **Day 31–60:** Apply — start gig, build portfolio evidence
  - **Day 61–90:** Transition — interview prep, role application, manager alignment

#### Node 6: Manager Release Brief (LLM node)
- **Input:** employee profile, matched opportunity, `manager_release_friction_1to5`
- **System prompt:** Generate a tactful, data-backed 1-page brief:
  - Why releasing this employee benefits the org
  - Backfill suggestions (other employees with overlapping skills)
  - Timeline that respects current commitments
  - Risk of NOT releasing (flight risk data)

#### Node 7: HR & Employee Messages (LLM node)
- Generate 3 outputs:
  - **HR email draft** (process + timeline + approval steps)
  - **Employee message** (motivating, clear next steps)
  - **Flight risk reduction projection** (e.g., "This move likely reduces flight risk from 4 → 2")

### Step 3.2 — Add Connections
1. Wire nodes sequentially: Intake → Normalize → [Gig Match + Role Match in parallel] → Growth Plan → Manager Brief → Messages
2. Connect the Azure OpenAI resource (`gpt-4o` deployment) to all LLM nodes
3. Connect the Azure AI Search index (`mobility-index`) to retrieval nodes

### Step 3.3 — Test the Flow
1. Use the built-in **Test** pane in Prompt Flow
2. Run all 10 demo personas and compare outputs against expected matches
3. Create a ground truth evaluation set (20 test cases) and note match accuracy

---

## Day 4: Guardrails + Fairness + Evaluation

### Step 4.1 — Content Safety Guardrails
1. In AI Foundry → **Safety + security** → enable **Azure AI Content Safety**
2. In your Prompt Flow, add a **Content Safety** node before the final output:
   - Block outputs containing: gender bias, age discrimination, salary specifics of real employees
   - Add a system prompt rule: *"Never reference protected attributes (gender, age, religion, disability) in match reasoning. Match only on skills, experience, interests, and constraints."*

### Step 4.2 — Transparency & Explainability
1. Ensure every match output includes:
   - **"Matched because:"** (skill overlap list)
   - **"Gaps to address:"** (missing skills + suggested courses)
   - **"Risk flags:"** (eligibility issues, manager friction, pay band mismatch)
2. Add an "Edit my skills/interests" prompt at the end of each response so employees can refine

### Step 4.3 — Evaluation in AI Foundry
1. Go to **Evaluation** → **+ New evaluation**
2. Upload your 20 ground-truth test cases as a dataset
3. Configure metrics:
   - **Groundedness** — are match reasons grounded in actual CSV data?
   - **Relevance** — do top 3 matches align with expected personas?
   - **Coherence** — are growth plans logical and actionable?
4. Run evaluation, review scores, iterate on prompts for any < 3/5 scores

---

## Day 5: Demo UI + ROI Story + Presentation

### Step 5.1 — Deploy as Chat App
1. In **ai.azure.com** → **Prompt flow** → select `mobility-broker-agent` → **Deploy**.
2. Deploy as a **real-time endpoint** (managed online deployment):
   - Endpoint name: `mobility-broker-endpoint`
   - Deployment name: `mobility-broker-v1`
   - Keep default output/connection mappings unless you need custom response shaping.
3. For a chat UI demo, use one of these current options:
   - **Fastest (zero deployment):** use **Prompt Flow Test/Playground** in Foundry.
   - **Branded web chat:** use **Azure OpenAI web app** and connect your prompt flow endpoint as the data source.
4. Keep this endpoint URL/key ready for Day 5 demo and fallback testing.

### Step 5.2 — Prepare Demo Script
Run through these 5 personas live in the demo (selected for variety):

| # | Persona | Mobility Story | Why it's compelling |
|---|---|---|---|
| 1 | E007 Kunal (Support → TAM) | Cross-function mobility | Shows the agent bridges non-obvious career paths |
| 2 | E012 Shreya (Tech Writer → Dev Advocate) | Career track change | Shows aspiration_track matching |
| 3 | E005 Vikram (DevOps → SRE) | Within-function level-up | Shows level + pay band logic |
| 4 | E009 Arjun (ML → NLP gig) | Gig matching precision | Shows short-term project fit |
| 5 | E016 Neha (Sales Ops → RevOps) | Non-engineering mobility | Shows the agent works beyond engineering |

### Step 5.3 — ROI Metrics Slide
Use the model from the project brief:
- 500 employees, 10% attrition, 30% regrettable, 20% preventable via mobility → **3 exits avoided/year**
- Replacement cost: **₹8–15L/engineer** → **₹24L–₹45L/year savings**
- Internal fill: **20 days faster** per role × 10 roles = **200 person-days productivity gain**
- Engagement: measurable via gig participation rate and mobility history acceptance rate trends

---

## Checkpoints & Verification

| Day | Checkpoint | Pass Criteria |
|---|---|---|
| Day 1 | Data uploaded, models deployed | All 6 CSVs in blob storage; GPT-4o + embedding model responding |
| Day 2 | Search index working | All 10 demo persona queries return expected top matches |
| Day 3 | Prompt Flow end-to-end | E007 Kunal query returns gig + role match + growth plan + manager brief in one response |
| Day 4 | Evaluation passing | Scores ≥ 4/5 for groundedness, relevance, coherence; no protected attribute leakage |
| Day 5 | Demo ready | Full 5-persona run-through in < 15 minutes with no errors |

---

## Key Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Data baseline | v1 files (24 employees, 12 gigs, 8 roles) | Complete and consistent; generate remaining to fill v2 schemas |
| Skill normalization | System prompt (not search enrichment) | Faster to implement; 15 synonym rows fit easily in prompt |
| Orchestration | Prompt Flow chat flow (not Agents) | More control over output structure; easier to demo step-by-step reasoning |
| Demo UI | Deploy-to-web-app from Prompt Flow | Clean chat interface with zero frontend code |
| Timeline compression | 7-day → 5-day | Merged guardrails + UI into Days 4–5; integrated ROI into demo |

---

## Azure Services Used

| Service | Purpose | SKU/Tier |
|---|---|---|
| Azure AI Foundry | Project hub, Prompt Flow, Evaluation | Standard |
| Azure OpenAI | GPT-4o (reasoning), text-embedding-3-large (vectors) | Standard, 30K TPM each |
| Azure AI Search | Vector + hybrid search over CSV data | Basic (sufficient for < 1K docs) |
| Azure Blob Storage | CSV data hosting | Standard (auto-provisioned) |
| Azure App Service | Chat web app deployment | Basic B1 (demo only) |
| Azure AI Content Safety | Guardrails for bias/PII filtering | Standard |
| Azure Key Vault | Secrets management | Standard (auto-provisioned) |
