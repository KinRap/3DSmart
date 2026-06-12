# 3DSmart AI Orchestrator

**Automated tender analysis for renewable energy installations — from PDF to GO/NO-GO in under 6 minutes.**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat-square&logo=python)](https://colab.research.google.com)
[![Gemini API](https://img.shields.io/badge/Gemini-2.5_Pro-7c6af7?style=flat-square&logo=google)](https://ai.google.dev)
[![License: MIT](https://img.shields.io/badge/License-MIT-4fc3f7?style=flat-square)](LICENSE)

---

## What it does

3DSmart AI Orchestrator is a **5-agent AI pipeline** that processes public procurement documents for renewable energy projects (photovoltaics, heat pumps, energy storage) and delivers a structured executive report with a GO / CONDITIONAL / NO-GO recommendation.

**Input:** PDF/DOCX procurement documents (tender notice, SWZ specification, bill of quantities, contract draft, competitive opening report)

**Output:** A downloadable PDF executive report containing:
- Formal compliance audit (14 categories)
- Red flag analysis with scoring (0–100)
- Technical feasibility assessment
- Strategic GO/NO-GO recommendation
- Questions to submit to the contracting authority

**Time:** Under 6 minutes from document upload to PDF download.

---

## 5-Agent Pipeline

```
PDF/DOCX documents
        │
        ▼
┌───────────────────┐
│  Agent 1 — DNPA   │  Data Normalization & Preparation
│  Gemini 2.5 Pro   │  Classifies documents, extracts metadata,
│  temp: 0.0        │  validates completeness, outputs JSON
└────────┬──────────┘
         │ JSON (normalized dossier)
         ▼
┌───────────────────┐
│  Agent 2 — FEA    │  Fact Extraction Agent
│  Gemini 2.5 Pro   │  Extracts 14 mandatory categories:
│  temp: 0.05       │  deadlines, wadium, penalties, OC insurance,
└────────┬──────────┘  experience requirements, competition data
         │ JSON (14-category fact set)
         ▼
┌─────────────────────────────────────────────┐
│  Agent 3 — CRAA          Agent 4 — TSFA     │
│  Compliance & Risk        Technical Scope    │
│  Audit Agent              & Feasibility      │
│  Gemini 2.5 Pro           Agent              │
│  temp: 0.1                Gemini 2.5 Pro     │
│                           temp: 0.2          │
│  • Red flag detection     • Installation     │
│  • Scoring formula          time estimation  │
│  • Contract traps         • Hidden costs     │
│  • KIO case law           • Tech compliance  │
└──────────────┬──────────────────────────────┘
               │ CRAA JSON + TSFA JSON
               ▼
┌───────────────────┐
│  Agent 5 — SOA    │  Strategic Orchestration Agent
│  Gemini 2.5 Pro   │  Validates consistency across agents,
│  temp: 0.3        │  synthesizes GO / CONDITIONAL / NO-GO,
└────────┬──────────┘  generates Executive Insight Report
         │
         ▼
    PDF Report
  (auto-download)
```

### Agent Roles & Operational Parameters

| Agent Name | Role | Description | Temperature |
| :--- | :--- | :--- | :--- |
| **DNPA** | Data Normalization & Preparation | Classifies and normalizes procurement documents, validates completeness, outputs JSON. | `temp: 0.0` |
| **FEA** | Fact Extraction Agent | Extracts 14 mandatory legal/financial categories (deadlines, wadium, penalties, insurance). | `temp: 0.05` |
| **CRAA** | Compliance & Risk Audit Agent | Audits formal compliance, detects red flags, applies corporate scoring matrix (0–100). | `temp: 0.1` |
| **TSFA** | Technical Scope & Feasibility Agent | Assesses technical execution, estimates timeline in brigade-days, identifies hidden costs. | `temp: 0.2` |
| **SOA** | Strategic Orchestration Agent | Validates cross-agent consistency, synthesizes insights, generates Executive Report. | `temp: 0.3` |

### Scoring formula (CRAA)

```
Score = (COMPLIANT_categories / 14) × 100
      − (CRITICAL_red_flags × 20)
      − (WARNING_red_flags × 5)

≥ 70  →  GO
50–69 →  CONDITIONAL
< 50  →  NO-GO
```

---

## Built with

- **Python 3.10+** — orchestration logic, file handling, PDF export
- **Google Gemini API** (`google-generativeai`) — all 5 agents run on Gemini 2.5 Pro
- **ipywidgets** — interactive UI (file upload, progress bar, output areas)
- **FPDF2** — PDF report generation with full Unicode support (DejaVu font)
- **python-docx** — DOCX text extraction including tables
- **Google Colab** — runtime environment with Secrets for API key management

---

## How to run

### Prerequisites

1. A Google account with access to [Google Colab](https://colab.research.google.com)
2. A [Gemini API key](https://aistudio.google.com/app/apikey) (free tier available)

### Setup

**Step 1 — Open in Colab**

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1q8c1EPH_uRCiGtnpUp2pq_kkeiVtd-7h)

**Step 2 — Add your API key to Colab Secrets**

1. In Colab, click the 🔑 key icon in the left sidebar
2. Add a new secret: **Name** = `API_KEY`, **Value** = your Gemini API key
3. Enable notebook access for this secret

**Step 3 — Run Cell 1 (installation)**

```python
!pip uninstall -y fpdf
!pip install -q -U google-generativeai fpdf2 python-docx
!apt-get install -q -y fonts-dejavu-core
```

**Step 4 — Run Cell 2 (main code)**

The dashboard will appear. Upload your procurement documents (PDF/DOCX) and click **URUCHOM ANALIZĘ I GENERUJ PDF**.

### Input Documents Overview

The system is calibrated to ingest and process standard Polish public procurement documentation formats:

| Document Type | Technical Polish Name | Required Status |
| :--- | :--- | :--- |
| **Tender notice** | Ogłoszenie o zamówieniu (BZP / TED) | ✅ Mandatory |
| **Specification** | SWZ (Specyfikacja Warunków Zamówienia) | ✅ Mandatory |
| **Bill of quantities** | Przedmiar robót | ✅ Mandatory |
| **Contract draft** | Projekt umowy / Istotne postanowienia | 🟡 Recommended |
| **Competitive opening** | Informacja z otwarcia ofert | 🔵 Optional |

### Output

A PDF report is automatically downloaded after analysis. The report contains:

1. **DNPA** — document inventory and normalization log
2. **FEA** — 14-category fact extraction with source citations
3. **CRAA** — compliance audit, red flags, scoring table, board narrative
4. **TSFA** — technical feasibility, timeline estimate, hidden costs
5. **SOA** — Executive Insight: GO / CONDITIONAL / NO-GO with action plan

---

## Technical documentation

Full operational documentation (30 pages) including agent system prompts, JSON interface contracts, red flag tables with scoring formulas, automation triggers, and POC test checklists:

📄 **[3DSmart_System_Agentowy_POC_v1.pdf](./docs/3DSmart_System_Agentowy_POC_v1.pdf)**

> **Note on model assignment:** The PDF describes the original platform-agnostic POC design, where each agent was assigned a different model (GPT and Gemini variants) based on task complexity and cost. The Colab implementation in this repository consolidates all five agents on **Gemini 2.5 Pro** via a single API — simplifying deployment while preserving the per-agent temperature settings and interface contracts.

---

## Security

API key management uses **Google Colab Secrets** (`userdata.get('API_KEY')`). The key is never stored in the source code or committed to version control.

---

## Domain context

3DSmart Sp. z o.o. is a renewable energy contractor (photovoltaics up to 2 MW, heat pumps up to 100 kW, energy storage up to 500 kWh). The system is calibrated for Polish public procurement law (Prawo zamówień publicznych) and KIO (National Appeals Chamber) case law.

---

## Author

**Kinga Rapacka** — AI Solutions Specialist  
[AI Portfolio](https://kinrap.github.io/AI_Portfolio/) · [GitHub](https://github.com/KinRap) · [LinkedIn](https://www.linkedin.com/in/kinga-rapacka-29315a2a3)

---

*Built as a production proof-of-concept for 3DSmart Sp. z o.o. | April 2026*
