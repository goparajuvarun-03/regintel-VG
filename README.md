# RegIntel-AI

> Closing the gap between regulatory change and compliance action.

A working production-quality prototype that ingests CMS regulations, identifies which internal VG Health Plan policies are impacted, generates cited remediation language, and gives compliance teams hours back per regulation.

## What it does

Five capabilities, all in the live app:

1. **Impact Analysis** — Score how a regulation affects internal policies, SOPs, and systems. Every claim cited to the source chunk.
2. **What-If Simulation** — Project financial, regulatory, member, and operational exposure if the gap remains unaddressed.
3. **Version Comparison** — Side-by-side diff of Proposed vs Final rules. Added / Removed / Modified changes flagged with compliance-risk delta.
4. **Proposed Text Generator** — Pulls verbatim sentences from your uploaded artifacts. Returns the exact passage to find and the compliant replacement language.
5. **Audit Timeline** — Every ingest, analysis, comparison, and remediation logged with user and timestamp.

## Quick start

### Run locally

```bash
git clone https://github.com/yourusername/regintel-ai.git
cd regintel-ai
pip install -r requirements.txt

# Copy secrets template and fill in your keys
cp .streamlit/secrets.toml.example .streamlit/secrets.toml
# Edit .streamlit/secrets.toml — at minimum set LLM_PROVIDER and the matching key

streamlit run streamlit_app.py
```

App opens at http://localhost:8501.

On first launch the seed documents in `seed/` are automatically loaded into the local SQLite database, so you'll have 3 regulations and 4 internal artifacts ready to analyze.

### Deploy to Streamlit Cloud

See **DEPLOYMENT.md** for the full step-by-step.

Short version: push this repo to GitHub → connect at share.streamlit.io → paste your secrets → done in 5 minutes.

## Folder layout

```
regintel-ai/
├── streamlit_app.py              # The whole app — single file, ~3,500 lines
├── requirements.txt              # Python dependencies
├── .gitignore                    # Keeps secrets and runtime data out of git
├── .streamlit/
│   ├── config.toml               # Theme + error display
│   └── secrets.toml.example      # Template — copy to secrets.toml locally
├── seed/                         # Auto-loaded on first startup
│   ├── 01_CMS-4201-F_Continuity_of_Care.txt
│   ├── 02_CMS-2439-F_Network_Adequacy.txt
│   ├── 03_CMS-MLN-7521_Claims_Timely_Filing.txt
│   ├── 04_VG_PA_Continuity_Policy.txt
│   ├── 05_VG_Provider_Network_SOP.txt
│   ├── 06_VG_Claims_System_Spec.txt
│   └── 07_VG_Member_Onboarding_Workflow.txt
├── README.md                     # This file
└── DEPLOYMENT.md                 # Step-by-step deploy guide
```

## Architecture (short)

- **UI**: Streamlit with Direction-C visual system (navy + teal + mint, Georgia/Calibri).
- **Retrieval**: TF-IDF + BM25 lexical retrieval over section-aware ~600-token chunks. Pluggable to pgvector at the HIPAA tier.
- **LLM**: Provider-agnostic. Switch between Groq (free, fast Llama 3.3 70B on LPU), Gemini Flash-Lite, or mock by changing one secret. Multi-layer fallback ensures the demo never breaks.
- **Storage**: SQLite for demo + non-HIPAA prod. Optional GitHub-backed snapshot sync prevents data loss on Streamlit Cloud container restarts.
- **Output validation**: Hallucination detector verifies LLM "verbatim quotes" actually exist in your uploaded documents. Parrot detector rejects responses where proposed text matches current text. Deterministic per-artifact rewrite as final fallback.

## Three deployment tiers

| Tier | Cost/mo | Hosting | LLM | Auth | For |
|------|---------|---------|-----|------|-----|
| **Demo / Pilot** | $0–60 | Streamlit Cloud Free | Groq or Gemini | None (public URL) | Demos, POCs, dev |
| **Non-HIPAA Prod** | $50–65 | DigitalOcean droplet + Cloudflare | Groq or Gemini | Cloudflare Access SSO | Pilot with CMS regs + synthetic data |
| **HIPAA Enterprise** | $1,200+ | AWS ECS Fargate Multi-AZ | Bedrock Claude under BAA | Cognito + Okta + MFA | Real PHI workloads |

## Honest scope notes

- The demo tier uses **synthetic policy data** modelled on real CMS regulations. The `VG Health Plan` artifacts in `seed/` are illustrative — not real production policies.
- **No HIPAA scope today.** Don't upload real PHI to the Streamlit Cloud deployment. The HIPAA tier above is the path when that becomes the goal.
- Citation accuracy is bounded by retrieval quality. Every claim shows its source chunk; the analyst makes the final call.
