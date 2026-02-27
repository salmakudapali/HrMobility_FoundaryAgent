
 **Internal Mobility & Skills “Opportunity Broker” Agent**
**Problem:** Employees leave because they can’t see internal opportunities that fit their skills; managers hoard talent; HR spends weeks matching people to roles/projects.

**What the agent does (unique):**
- Turns a resume/LinkedIn-style profile + a short “what I like doing” intake into a **skills graph**.
- Matches employees to **internal gigs** (short-term projects), not just full roles.
- Produces a **growth plan** (courses + mentor + 30/60/90-day path) and a **manager negotiation brief** (“why releasing them is good + backfill suggestions”).

**Usability:** HR uploads role templates; employees fill a 5-minute intake; outputs ranked matches + scripts for HR/manager/employee.

**Impact:**
- Quant: reduce regrettable attrition; increase internal fill rate; reduce time-to-fill.
- Qual: better engagement, fairer access to opportunities.

**7-day MVP:** profile intake → matching → 1-page mobility plan + manager message drafts.

# Sample data (synthetic) you can use immediately
Below is a complete, ready-to-use synthetic dataset design. If you want, tell me whether you prefer **CSV** or **JSON**, and I’ll output the files in that format.

### Dataset components
1) **employees** (80–200 rows)
- employee_id, role_family, current_role, location, tenure_months  
- skills (list), tools (list), certifications, languages  
- career_interests (text), constraints (remote only / shift)  
- performance_band (A/B/C), manager_support (1–5) *(optional)*

2) **internal_gigs** (30–60 rows)
- gig_id, department, duration_weeks, time_percent (10–50%)  
- required_skills, nice_to_have, timezone needs  
- outcome (deliverable), hiring_manager persona

3) **open_roles** (20–40 rows)
- role_id, level, location, required_skills, salary_band (range)  
- interview_loop (text), urgency

4) **learning_catalog** (40–80 rows)
- course_id, skill_tags, difficulty, duration_hours, provider

5) **mobility_history** (optional, 50–150 rows)
- employee_id, applied_to, outcome, reason_declined (text)

### Where to get “realistic” synthetic text
- Use public job descriptions (copy *structure*, not proprietary text)
- Use O*NET skill taxonomies (public) as your skill list seed
- Generate interests/constraints with your model, but keep it plausible and non-sensitive

---

# MVP in 7 days (solo) — scoped plan for Azure AI Foundry

### Day 1–2: Data + schema + evaluation set
- Build synthetic employees/gigs/roles dataset
- Create 20 “ground truth” test cases (employee → expected top 3 matches)

### Day 3: Retrieval + ranking
- RAG over gigs/roles + skill normalization (skill synonyms)
- Output: top matches with explanations (“matched because X, lacks Y”)

### Day 4: Action outputs (what makes it feel like an *agent*)
- Generate: 30/60/90-day plan + learning plan + manager release pitch + HR email drafts

### Day 5: Guardrails + fairness checks (high business value)
- Add “avoid sensitive attributes” rule (gender, age, etc.)
- Provide “why this match” transparency + allow user to edit skills/interests

### Day 6: UI + demo script
- Simple web form (or Foundry chat) + downloadable PDF/Doc output

### Day 7: Metrics + ROI story
- Report: projected internal fill lift, time-to-fill reduction, attrition reduction assumptions


## Synthetic sample data for **Internal Mobility & Skills “Opportunity Broker”** (Tech company, India, 
### Files included (CSV)
- `employees.csv` (24 employees)
- `internal_gigs.csv` (12 internal gigs / projects)
- `open_roles.csv` (8 open roles)
- `learning_catalog.csv` (16 learning items)
- `skill_synonyms.csv` (normalization map)

You can use this for RAG + ranking + plan generation.

---

```csv name=employees.csv
employee_id,full_name,current_role,role_family,level,location,work_mode,tenure_months,manager_support_1to5,performance_band,skills,tools,certifications,languages,career_interests,constraints
E001,Aarav Mehta,Backend Engineer,Engineering,L2,Bengaluru,Hybrid,18,4,A,"Java|Spring Boot|Microservices|REST APIs|SQL|System Design","PostgreSQL|Redis|Kafka|Docker|Git","",English|Hindi,"Move towards platform engineering and reliability; likes performance work","Prefer Bengaluru; ok with on-call"
E002,Diya Sharma,Frontend Engineer,Engineering,L2,Bengaluru,Hybrid,14,3,B,"React|TypeScript|HTML|CSS|Accessibility|UI Performance","Next.js|Redux|Jest|Cypress","",English|Hindi,"Interested in design systems and UX engineering","No night calls"
E003,Rohan Iyer,Data Analyst,Data,L2,Hyderabad,Hybrid,22,4,A,"SQL|Data Modeling|Dashboards|A/B Testing|Statistics","BigQuery|Looker|Excel|Python","",English|Telugu,"Wants to become analytics engineer; likes building reliable datasets","Prefers backend/data work over presentations"
E004,Ananya Nair,QA Engineer,Engineering,L2,Pune,Hybrid,16,3,B,"Test Automation|API Testing|Selenium|Test Strategy|Bug Triage","Postman|JUnit|Selenium Grid|Git","",English|Malayalam,"Move into SDET/quality platform; build test frameworks","Open to travel quarterly"
E005,Vikram Singh,DevOps Engineer,Engineering,L3,Bengaluru,Hybrid,30,4,A,"CI/CD|Kubernetes|Terraform|Linux|Observability|Incident Response","GitHub Actions|ArgoCD|Prometheus|Grafana|AWS","CKA",English|Hindi,"Interested in SRE and reliability leadership","On-call ok; prefers infra/platform teams"
E006,Meera Patel,Product Analyst,Product,L2,Mumbai,Hybrid,20,4,A,"Product Analytics|SQL|Funnel Analysis|Experiment Design|Storytelling","Amplitude|Mixpanel|Looker|Excel","",English|Gujarati,"Wants PM track in B2B SaaS; likes pricing and onboarding","Mumbai preferred; some travel ok"
E007,Kunal Verma,Customer Support Specialist,Support,L1,Gurugram,Onsite,10,3,B,"Ticket Triage|Communication|Root Cause Notes|Customer Empathy","Zendesk|Jira","ITIL Foundation",English|Hindi,"Wants to move into Technical Account Management","Needs fixed shift"
E008,Pooja Kulkarni,UX Designer,Design,L2,Bengaluru,Hybrid,26,4,A,"UX Research|Wireframing|Prototyping|Design Systems|Information Architecture","Figma|Miro","",English|Marathi,"Interested in research ops and accessibility design","Remote 2 days/week minimum"
E009,Arjun Rao,ML Engineer,Engineering,L3,Bengaluru,Hybrid,28,3,A,"Python|Machine Learning|Feature Engineering|Model Monitoring|NLP","PyTorch|MLflow|Airflow|Docker","",English|Kannada,"Wants applied NLP in customer support automation","Ok with on-call for model monitoring"
E010,Sneha Bose,HR Generalist,HR,L2,Bengaluru,Hybrid,24,5,A,"HR Ops|Employee Relations|Policy Support|HRIS Basics","Workday|Excel","",English|Bengali,"Interested in People Analytics and HR tech","Bengaluru preferred"
E011,Nikhil Jain,Full Stack Engineer,Engineering,L2,Jaipur,Remote,12,4,B,"Node.js|React|REST APIs|SQL|Cloud Basics","AWS|PostgreSQL|Docker","",English|Hindi,"Wants to join internal tools / developer productivity","Remote only"
E012,Shreya Das,Tech Writer,Operations,L2,Kolkata,Remote,34,4,A,"Technical Writing|API Docs|Information Architecture|Docs-as-Code","Markdown|Docusaurus|Git","",English|Bengali,"Interested in developer education and enablement content","Remote only"
E013,Rahul Khanna,Security Analyst,Security,L2,Bengaluru,Hybrid,15,3,B,"AppSec Basics|Threat Modeling|Vulnerability Management|SOC Processes","Burp Suite|Splunk|OWASP ZAP","",English|Hindi,"Wants cloud security and DevSecOps exposure","Ok with on-call 1 week/month"
E014,Isha Menon,Engineering Manager,Engineering,L5,Bengaluru,Hybrid,60,4,A,"People Management|Delivery|System Design|Hiring|Roadmapping","Jira|Confluence","",English|Malayalam,"Wants to mentor internal mobility; open to cross-team rotations","No travel"
E015,Aditya Joshi,Data Engineer,Data,L3,Pune,Hybrid,32,4,A,"Python|ETL|Data Warehousing|SQL|Data Quality","Airflow|dbt|Snowflake","",English|Marathi,"Interested in building a unified metrics layer (analytics engineering)","Pune/Bengaluru ok"
E016,Neha Kapoor,Sales Ops Analyst,Operations,L2,Gurugram,Hybrid,19,3,B,"Sales Ops|Excel|Forecasting|CRM Hygiene|Process Docs","Salesforce|Excel|Looker","",English|Hindi,"Wants business systems / RevOps automation","No late meetings"
E017,Sameer Ali,Site Reliability Engineer,Engineering,L4,Bengaluru,Hybrid,40,4,A,"SRE|Distributed Systems|Incident Command|Capacity Planning|Observability","Kubernetes|Prometheus|Grafana|Go","",English|Hindi,"Wants to build reliability tooling and runbooks","On-call ok"
E018,Ritika Chawla,Data Scientist,Data,L3,Hyderabad,Hybrid,27,3,A,"Statistics|Causal Inference|Experimentation|Python|Forecasting","Python|Jupyter|BigQuery","",English|Hindi,"Interested in churn prediction + lifecycle messaging","Hybrid ok; prefers Hyderabad"
E019,Manish Gupta,Android Engineer,Engineering,L3,Bengaluru,Hybrid,36,4,A,"Kotlin|Android|Performance|Offline-first|REST APIs","Firebase|Gradle","",English|Hindi,"Wants to move into mobile platform and build shared components","Bengaluru preferred"
E020,Aditi Rao,Finance Analyst,Finance,L2,Mumbai,Hybrid,21,4,B,"Financial Modeling|Budgeting|Variance Analysis|Excel|Process Improvement","Excel|PowerPoint","",English|Marathi,"Interested in FP&A automation and analytics","Mumbai preferred"
E021,Harsha Reddy,Cloud Engineer,Engineering,L3,Hyderabad,Hybrid,29,4,A,"AWS|Networking|Terraform|Security Groups|Kubernetes Basics","AWS|Terraform|CloudWatch","AWS SAA",English|Telugu,"Wants cloud cost optimization and FinOps","Prefers Hyderabad; on-call ok"
E022,Tanvi Sinha,People Operations Coordinator,HR,L1,Bengaluru,Hybrid,9,5,B,"HR Coordination|Onboarding|Scheduling|Documentation","Google Workspace|Excel","",English|Hindi,"Interested in HR systems and automation","Bengaluru only"
E023,Pranav Kulkarni,Solutions Engineer,PreSales,L3,Mumbai,Hybrid,25,4,A,"Customer Demos|APIs|Solution Design|Stakeholder Management","Postman|Python|Salesforce","",English|Hindi,"Interested in product specialist role; likes discovery calls","Travel up to 20%"
E024,Kavya Iyer,Backend Engineer,Engineering,L2,Chennai,Remote,11,3,B,"Java|Spring Boot|SQL|Unit Testing|REST APIs","MySQL|Docker","",English|Tamil,"Wants to move into data engineering or event-driven systems","Remote only"
```

```csv name=internal_gigs.csv
gig_id,title,department,location_preference,remote_allowed,duration_weeks,time_percent,required_skills,nice_to_have_skills,description,deliverable
G001,Build Developer Portal MVP,Platform,Bengaluru,Yes,6,30,"React|TypeScript|API Docs|Docs-as-Code","Node.js|Docusaurus","Internal portal for APIs, runbooks, and onboarding docs","Working portal + 10 key API pages"
G002,FinOps Cost Dashboard,Cloud,Hyderabad,Yes,5,20,"AWS|Cost Optimization|SQL|Dashboards","Python|Looker","Create a weekly cost visibility dashboard and anomaly alerts","Cost dashboard + alert thresholds"
G003,Experimentation Quality Review,Data,Hyderabad,Yes,4,20,"A/B Testing|Statistics|Experiment Design","Causal Inference","Audit experiment setup quality and build a checklist","Experiment QA checklist + examples"
G004,Accessibility Audit for Web App,Design,Bengaluru,Yes,3,15,"Accessibility|HTML|CSS|WCAG","React","Audit key flows and propose fixes with dev-ready guidance","Audit report + prioritized backlog"
G005,Support Ticket Auto-Triage Pilot,ML,Bengaluru,Yes,6,25,"Python|NLP|Labeling Strategy|Model Monitoring","Zendesk|Prompting","Prototype triage for support tickets and routing","Pilot model + evaluation report"
G006,Test Automation Framework Upgrade,Quality,Pune,Yes,5,25,"Test Automation|API Testing|Framework Design","CI/CD","Improve reliability and speed of test suite","Updated framework + CI pipeline changes"
G007,Security Threat Modeling Workshop Pack,Security,Bengaluru,Yes,2,10,"Threat Modeling|AppSec Basics|Communication","OWASP ASVS","Create a reusable workshop and templates for squads","Workshop deck + templates"
G008,Onboarding Journey Rewrite,Product,Mumbai,Yes,4,20,"Funnel Analysis|Product Analytics|Storytelling","UX Research","Identify onboarding friction and propose improvements","Insights doc + prioritized experiments"
G009,Mobile Platform Component Library,Mobile,Bengaluru,No,6,20,"Kotlin|Android|Design Systems","Performance","Build shared UI components and guidelines","Component library + usage docs"
G010,Reliability Runbook Standardization,SRE,Bengaluru,Yes,4,20,"Incident Response|Runbooks|Observability","Kubernetes","Standardize runbooks and alert playbooks across services","Runbook templates + 5 service runbooks"
G011,Analytics Engineering Metrics Layer,Data,Pune,Yes,6,30,"dbt|SQL|Data Modeling|Data Quality","Snowflake","Create a metrics layer for core KPIs","dbt models + data tests + documentation"
G012,CRM Automation for Sales Ops,RevOps,Gurugram,Yes,4,20,"Salesforce|Process Improvement|CRM Hygiene","Python|APIs","Automate lead routing and data quality checks","Flow/spec + automation prototype"
```

```csv name=open_roles.csv
role_id,title,department,level,location,remote_allowed,required_skills,nice_to_have_skills,role_summary
R001,Platform Engineer,Platform,L3,Bengaluru,Yes,"Kubernetes|Terraform|CI/CD|Linux","Go|Observability","Build internal developer platform capabilities and automation"
R002,Analytics Engineer,Data,L3,Pune,Yes,"SQL|dbt|Data Modeling|Data Quality","Snowflake|Looker","Own curated datasets and metrics for product and business"
R003,Technical Account Manager,Customer Success,L3,Gurugram,No,"Customer Communication|Technical Troubleshooting|Stakeholder Management","APIs|SaaS","Own post-sales success for strategic accounts"
R004,UX Researcher,Design,L3,Bengaluru,Yes,"UX Research|Interviewing|Synthesis","Experiment Design","Run continuous discovery and inform product decisions"
R005,Security Engineer (DevSecOps),Security,L3,Bengaluru,Yes,"Threat Modeling|CI/CD|AppSec|Cloud Security","Terraform|Kubernetes","Shift-left security and build guardrails"
R006,Developer Advocate (India),DevRel,L3,Bengaluru,Yes,"Technical Writing|Public Speaking|API Knowledge","Video Creation","Create content and enable external & internal developers"
R007,SRE (Observability),SRE,L4,Bengaluru,Yes,"Observability|Distributed Systems|Incident Command","Grafana|Prometheus","Improve monitoring, alerting, and reliability practices"
R008,Product Manager (Onboarding),Product,L3,Mumbai,Yes,"Product Thinking|Funnel Analysis|Experimentation","UX Research","Own onboarding roadmap and activation metrics"
```

```csv name=learning_catalog.csv
course_id,title,provider,skill_tags,difficulty,duration_hours,format
C001,dbt Fundamentals,Internal L&D,"dbt|SQL|Data Modeling",Beginner,6,Video
C002,Practical Threat Modeling,Internal L&D,"Threat Modeling|AppSec Basics",Intermediate,4,Workshop
C003,FinOps for Engineers,External,"Cost Optimization|AWS",Beginner,5,Video
C004,Observability with Prometheus/Grafana,Internal L&D,"Observability|Prometheus|Grafana",Intermediate,6,Hands-on
C005,Accessibility for Product Teams,External,"Accessibility|WCAG|UX",Beginner,3,Video
C006,Experimentation Design Clinic,Internal L&D,"A/B Testing|Statistics|Experiment Design",Intermediate,4,Workshop
C007,Developer Portal Patterns,External,"Developer Experience|Docs-as-Code",Intermediate,4,Article
C008,CI/CD with GitHub Actions,Internal L&D,"CI/CD|GitHub Actions",Beginner,3,Hands-on
C009,Kubernetes Administration (CKA prep),External,"Kubernetes|Linux",Advanced,20,Course
C010,Practical NLP for Ticket Triage,External,"NLP|Python|Prompting",Intermediate,8,Course
C011,Analytics Storytelling,Internal L&D,"Storytelling|Dashboards",Beginner,3,Workshop
C012,Salesforce Flows Essentials,External,"Salesforce|Automation",Beginner,4,Video
C013,Go for Reliability Tooling,External,"Go|Distributed Systems",Intermediate,10,Course
C014,Writing Great API Docs,Internal L&D,"Technical Writing|API Docs",Beginner,3,Workshop
C015,System Design Interview Lite,Internal L&D,"System Design|Microservices",Intermediate,6,Workshop
C016,Leadership: Influencing Without Authority,Internal L&D,"Stakeholder Management|Negotiation",Beginner,3,Workshop
```

```csv name=skill_synonyms.csv
variant,canonical
SRE,Site Reliability Engineering
DevOps,Site Reliability Engineering
Observability,Monitoring & Observability
Grafana,Monitoring & Observability
Prometheus,Monitoring & Observability
API Docs,Technical Writing
Docs-as-Code,Technical Writing
A/B Testing,Experimentation
Experiment Design,Experimentation
Cost Optimization,FinOps
WCAG,Accessibility
Threat Modeling,Application Security
AppSec,Application Security
CI/CD,CI/CD
K8s,Kubernetes
```

---

## 10 demo personas (use these in your presentation)
1) **E007 Kunal (Support)** → best match: **R003 TAM**; gig: **G005 ticket triage pilot** (bridges to technical).
2) **E012 Shreya (Tech Writer)** → best match: **R006 Dev Advocate**; gig: **G001 developer portal**.
3) **E015 Aditya (Data Eng)** → best match: **R002 Analytics Engineer**; gig: **G011 metrics layer**.
4) **E005 Vikram (DevOps)** → best match: **R001 Platform Engineer** or **R007 SRE**; gig: **G010 runbooks**.
5) **E002 Diya (Frontend)** → gig: **G004 accessibility audit** + **G001 portal**.
6) **E009 Arjun (ML)** → gig: **G005 auto-triage** (perfect fit).
7) **E016 Neha (Sales Ops)** → gig: **G012 CRM automation**; role path: RevOps systems.
8) **E018 Ritika (Data Scientist)** → gig: **G003 experimentation quality** + role: **R008 PM onboarding** (if she’s PM-inclined).
9) **E019 Manish (Android)** → gig: **G009 mobile component library**.
10) **E013 Rahul (Security Analyst)** → gig: **G007 threat modeling pack**; role: **R005 DevSecOps**.

---

## Business impact framing (for judges)
Use a simple ROI model they can understand:
- Company 500 employees; assume **10% annual attrition** (50 exits/year)
- Assume **30% are regrettable** and could be prevented with mobility (15 exits/year)
- If agent prevents **20% of those** via internal moves → **3 exits avoided/year**
- Replacement cost conservatively **₹8–15L per engineer** (recruiting + onboarding + productivity loss)
- ROI: **₹24L–₹45L/year** from attrition alone, plus faster hiring:
  - If internal fill saves **20 days time-to-fill** for even 10 roles/year, that’s major productivity gain


Below is an **expanded synthetic dataset design** that includes exactly what you asked for (150 employees + pay bands, aspiration levels, manager release friction, and mobility history) and is optimized for a **Foundry Chat MVP**.

Because 150 fully-populated rows would be very long to paste into chat, I’m giving you:
1) a **complete schema** (copy/paste into your project),
2) **starter CSV samples** (so you can test immediately),
3) a **deterministic generation recipe** (so you can generate the full 150 locally in seconds), and
4) a **Foundry chat prompt pack** to demo business impact.

If you want, tell me your preferred format for delivery and I can output the full 150 rows here in multiple messages: **CSV split by file (recommended)** or **JSONL**.

---

## A) Final dataset files (what you will have)

### 1) `employees.csv` (150 rows)
Adds the HR-real fields you requested:
- `pay_band` (e.g., PB3–PB8)
- `aspiration_level` (0–3: stability → role change → level-up → track change)
- `aspiration_track` (IC / Manager / Product / Data / Security / Design / GTM)
- `manager_release_friction_1to5` (5 = manager likely to block)
- `flight_risk_1to5` (synthetic proxy used only for ROI storyline)
- `mobility_eligibility` (Yes/No based on tenure/performance in your policy)
- `last_promotion_months_ago`

### 2) `internal_gigs.csv` (60 rows)
- includes `sensitive_team` flag (some gigs require approval)
- includes `time_zone_overlap` (IST-only vs. flexible)

### 3) `open_roles.csv` (30 rows)
- includes `pay_band_min/pay_band_max`
- includes `interview_complexity` (simple/standard/loop-heavy)

### 4) `learning_catalog.csv` (80 rows)

### 5) `mobility_history.csv` (300–600 rows)
- employee applications to gigs/roles and outcomes
- includes `reason_declined` (comp, manager block, location, skill gap, timing)

### 6) `skill_synonyms.csv` (expandable)

---

## B) CSV schemas (copy/paste as headers)

```csv name=employees.csv
employee_id,full_name,current_role,role_family,level,pay_band,location,work_mode,tenure_months,last_promotion_months_ago,performance_band,manager_support_1to5,manager_release_friction_1to5,aspiration_level,aspiration_track,career_interests,constraints,skills,tools,certifications,languages,flight_risk_1to5,mobility_eligibility
```

```csv name=mobility_history.csv
event_id,employee_id,event_type,target_type,target_id,event_date,outcome,reason_declined,notes
```

```csv name=internal_gigs.csv
gig_id,title,department,location_preference,remote_allowed,duration_weeks,time_percent,time_zone_overlap,sensitive_team,required_skills,nice_to_have_skills,description,deliverable
```

```csv name=open_roles.csv
role_id,title,department,level,location,remote_allowed,pay_band_min,pay_band_max,interview_complexity,required_skills,nice_to_have_skills,role_summary
```

---

## C) Starter samples (small but valid)

### `employees.csv` (first 12 sample rows)
```csv name=employees.csv
employee_id,full_name,current_role,role_family,level,pay_band,location,work_mode,tenure_months,last_promotion_months_ago,performance_band,manager_support_1to5,manager_release_friction_1to5,aspiration_level,aspiration_track,career_interests,constraints,skills,tools,certifications,languages,flight_risk_1to5,mobility_eligibility
E001,Aarav Mehta,Backend Engineer,Engineering,L2,PB4,Bengaluru,Hybrid,18,18,A,4,3,2,IC,"Move towards platform engineering and reliability; likes performance work","Prefer Bengaluru; ok with on-call","Java|Spring Boot|Microservices|REST APIs|SQL|System Design","PostgreSQL|Redis|Kafka|Docker|Git","",English|Hindi,2,Yes
E002,Diya Sharma,Frontend Engineer,Engineering,L2,PB4,Bengaluru,Hybrid,14,14,B,3,2,1,IC,"Interested in design systems and UX engineering","No night calls","React|TypeScript|HTML|CSS|Accessibility|UI Performance","Next.js|Redux|Jest|Cypress","",English|Hindi,2,Yes
E003,Rohan Iyer,Data Analyst,Data,L2,PB4,Hyderabad,Hybrid,22,22,A,4,2,3,Data,"Wants to become analytics engineer; likes building reliable datasets","Prefers backend/data work over presentations","SQL|Data Modeling|Dashboards|A/B Testing|Statistics","BigQuery|Looker|Excel|Python","",English|Telugu,2,Yes
E004,Ananya Nair,QA Engineer,Engineering,L2,PB3,Pune,Hybrid,16,16,B,3,3,2,IC,"Move into SDET/quality platform; build test frameworks","Open to travel quarterly","Test Automation|API Testing|Selenium|Test Strategy|Bug Triage","Postman|JUnit|Selenium Grid|Git","",English|Malayalam,3,Yes
E005,Vikram Singh,DevOps Engineer,Engineering,L3,PB5,Bengaluru,Hybrid,30,10,A,4,4,2,IC,"Interested in SRE and reliability leadership","On-call ok; prefers infra/platform teams","CI/CD|Kubernetes|Terraform|Linux|Observability|Incident Response","GitHub Actions|ArgoCD|Prometheus|Grafana|AWS","CKA",English|Hindi,2,Yes
E006,Meera Patel,Product Analyst,Product,L2,PB4,Mumbai,Hybrid,20,20,A,4,3,3,Product,"Wants PM track in B2B SaaS; likes pricing and onboarding","Mumbai preferred; some travel ok","Product Analytics|SQL|Funnel Analysis|Experiment Design|Storytelling","Amplitude|Mixpanel|Looker|Excel","",English|Gujarati,2,Yes
E007,Kunal Verma,Customer Support Specialist,Support,L1,PB3,Gurugram,Onsite,10,10,B,3,2,3,GTM,"Wants to move into Technical Account Management","Needs fixed shift","Ticket Triage|Communication|Root Cause Notes|Customer Empathy","Zendesk|Jira","ITIL Foundation",English|Hindi,4,No
E008,Pooja Kulkarni,UX Designer,Design,L2,PB4,Bengaluru,Hybrid,26,26,A,4,3,2,Design,"Interested in research ops and accessibility design","Remote 2 days/week minimum","UX Research|Wireframing|Prototyping|Design Systems|Information Architecture","Figma|Miro","",English|Marathi,2,Yes
E009,Arjun Rao,ML Engineer,Engineering,L3,PB5,Bengaluru,Hybrid,28,16,A,3,3,2,Data,"Wants applied NLP in customer support automation","Ok with on-call for model monitoring","Python|Machine Learning|Feature Engineering|Model Monitoring|NLP","PyTorch|MLflow|Airflow|Docker","",English|Kannada,2,Yes
E010,Sneha Bose,HR Generalist,HR,L2,PB4,Bengaluru,Hybrid,24,24,A,5,2,2,HR,"Interested in People Analytics and HR tech","Bengaluru preferred","HR Ops|Employee Relations|Policy Support|HRIS Basics","Workday|Excel","",English|Bengali,2,Yes
E011,Nikhil Jain,Full Stack Engineer,Engineering,L2,PB4,Jaipur,Remote,12,12,B,4,2,2,IC,"Wants to join internal tools / developer productivity","Remote only","Node.js|React|REST APIs|SQL|Cloud Basics","AWS|PostgreSQL|Docker","",English|Hindi,3,Yes
E012,Shreya Das,Tech Writer,Operations,L2,PB4,Kolkata,Remote,34,18,A,4,2,3,GTM,"Interested in developer education and enablement content","Remote only","Technical Writing|API Docs|Information Architecture|Docs-as-Code","Markdown|Docusaurus|Git","",English|Bengali,2,Yes
```

### `mobility_history.csv` (sample)
```csv name=mobility_history.csv
event_id,employee_id,event_type,target_type,target_id,event_date,outcome,reason_declined,notes
MH0001,E006,Applied,Role,R008,2025-11-14,Declined,Timing,"Role opened during peak quarterly planning"
MH0002,E006,Applied,Gig,G008,2025-12-02,Accepted,,"Joined onboarding insights gig (20%)"
MH0003,E005,Applied,Role,R007,2025-10-21,Declined,Manager Block,"Manager requested delay until after migration milestone"
MH0004,E005,Applied,Gig,G010,2025-11-05,Accepted,,"Contributed to runbook standardization"
MH0005,E007,Applied,Role,R003,2025-12-10,Declined,Eligibility,"Tenure < 12 months policy"
MH0006,E011,Applied,Gig,G001,2025-12-01,Accepted,,"Built portal UI components remotely"
MH0007,E003,Applied,Role,R002,2025-09-18,Declined,Skill Gap,"Needed dbt experience; suggested learning plan"
MH0008,E003,Applied,Gig,G011,2025-10-05,Accepted,,"Worked on metrics layer with Aditya"
```

---

## D) How to generate the full 150 employees + history (deterministic recipe)
If you can run *any* local script (Python/Node), this is the fastest way to get realistic volume without manual work.

### Distribution (tuned for a 500-employee tech company in India)
- Locations: Bengaluru 45%, Hyderabad 20%, Pune 15%, Gurugram 10%, Mumbai 7%, Remote(India) 3%
- Role families (for 150 sample employees): Engineering 55%, Data 15%, Product 10%, Design 7%, Security 5%, Support 5%, HR/Operations/Finance 3%
- Levels & pay bands:
  - L1 → PB3
  - L2 → PB4
  - L3 → PB5
  - L4 → PB6
  - L5 → PB7–PB8
- Mobility eligibility rule (example): tenure ≥ 12 months AND performance_band ∈ {A,B}
- Manager release friction: correlated with (team criticality + low headcount + performance A)
- Flight risk: correlated with (high aspiration_level + high friction + long time since promotion)

### Mobility history generation
- For each eligible employee: 2–6 events
- Outcomes:
  - Accepted 25–35%
  - Declined 45–55% (reasons: Skill Gap, Timing, Comp, Location, Manager Block)
  - Pending 10–20%

If you tell me whether you can run a script locally, I’ll provide the generator as a file. (If you can’t run code at all, I’ll paste the full CSVs here in chunks.)

---

## E) Foundry Chat: recommended agent behavior (to win on business impact)
In chat, make the agent feel like it’s doing *HR-grade brokering*, not job search:

### Inputs (chat asks 5 questions max)
1) Employee ID or paste profile
2) Goal: “new role / project / skill growth”
3) Constraints (location, remote, time %, on-call)
4) Timeline (now / next quarter)
5) Manager situation: supportive vs. friction

### Outputs (what you demo)
- **Top 3 gigs + Top 3 roles** with:
  - Fit score
  - Strengths + skill gaps
  - Risk flags: eligibility, manager block likelihood, comp band mismatch
- **Manager release pitch** (short, tactful)
- **Employee message** (motivating + clear next steps)
- **30/60/90-day learning plan** linked to catalog
- **HR metrics line**: “This move likely reduces flight risk from 4 → 2” (synthetic but compelling)

---
