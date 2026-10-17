# SecondLook — Same qualifications. Different decisions. Who notices?
**Team WISE — IBM Z Datathon 2026 Project** (theme: Real-Time AI for Critical Decisions)
IBM Z Datathon 2026. Advisory audit of historical HMDA decisions: finds denied applications with financially comparable approved ones, queues them for
human review, and reports demographic patterns only as observed disparities (never proof of discrimination).
## Run
    python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
    .venv/bin/python src/data/download.py && .venv/bin/python src/data/preprocess.py
    .venv/bin/python -m src.models.run_matching && .venv/bin/python -m src.models.run_audit && .venv/bin/python -m src.models.run_eval
    .venv/bin/python -m pytest tests
    .venv/bin/streamlit run dashboard/app.py          # local; set SL_API_URL=http://<linuxone>:8000 for remote
    .venv/bin/uvicorn src.api.main:app --port 8000
Docs: `docs/`. Status: MVP works locally, Granite notes, Model Sensitivity Analysis, SHED and Census context are in, and the API is deployed on the LinuxONE VM; Granite notes, ablation model are pending.

## Reviewer dashboard (`ui_demo/`) — step by step
A single-page app (plain HTML/JS, no build, works offline). It shows **LIVE API** when connected to the backend, **DEMO DATA** otherwise.

**A. Quick look, no backend (bundled demo data)**
1. `python3 ui_demo/serve.py` — opens http://localhost:8080.

**B. Live data from the LinuxONE VM (what the demo video uses)**
1. Put `wise_key.pem` in the repo root: `chmod 600 wise_key.pem` (it is git-ignored).
2. Get the audit token once: `ssh -i wise_key.pem linux1@148.100.109.157 'cat ~/secondlook/.api.env'` (needed only for the Fairness page).
3. Terminal 1 — tunnel to the VM's API, leave it running:
   `ssh -i wise_key.pem -N -L 18000:localhost:8000 linux1@148.100.109.157`
4. Terminal 2 — dashboard, proxying every API call through the tunnel:
   `python3 ui_demo/serve.py --api http://localhost:18000 --token <AUDIT_TOKEN>`
5. Open http://localhost:8080/?reset#overview and check the sidebar badge says **LIVE API**.
   (`?reset` clears locally saved demo decisions and selects case `b81daae6eb`.)

**C. Same machine as the API** (laptop with `data/processed/`, or on the VM): `uvicorn src.api.main:app --port 8000` then open http://localhost:8000/ui.

**Pages:** Overview → Queue (▾ on a row: *View in detail* / *Compare twins*) → Compare (denied vs. any ranked approved comparator, gap bars in tolerance units) → Case detail (stated reasons vs data, Granite note, grounded Q&A, advisory decision + audit trail) → Simulate (live `/analyse-application`) → Fairness (raw vs matched gap, matching-setting dropdown, reviewer consistency, model sensitivity) → System.
Decisions saved in live mode go to the VM's append-only review log. Demo script: `docs/demo_walkthrough.md`.

## Data sources and licences
| Data | Source | Terms |
|---|---|---|
| HMDA 2025 LAR (Maryland) | FFIEC / CFPB Data Browser API | US public data |
| SHED 2025 public-use file | Federal Reserve Board | public use; used with survey weights, as context only |
| ACS 2024 5-year (county) | US Census Bureau, via Census Reporter (official API needs a key) | public data |

## Libraries and open-source resources used (links per Datathon rule 5)
| Library / resource | Link | Licence |
|---|---|---|
| pandas | https://pandas.pydata.org | BSD-3 |
| NumPy | https://numpy.org | BSD-3 |
| SciPy | https://scipy.org | BSD-3 |
| scikit-learn | https://scikit-learn.org | BSD-3 |
| XGBoost | https://github.com/dmlc/xgboost | Apache-2.0 |
| FastAPI | https://fastapi.tiangolo.com | MIT |
| Uvicorn | https://www.uvicorn.org | BSD-3 |
| Pydantic | https://docs.pydantic.dev | MIT |
| Requests / HTTPX | https://requests.readthedocs.io , https://www.python-httpx.org | Apache-2.0 / BSD-3 |
| Streamlit | https://streamlit.io | Apache-2.0 |
| Plotly | https://plotly.com/python | MIT |
| PyArrow | https://arrow.apache.org | Apache-2.0 |
| pytest | https://pytest.org | MIT |
| llama.cpp / llama-cpp-python (IBM-provided LinuxONE container) | https://github.com/ggml-org/llama.cpp , https://github.com/abetlen/llama-cpp-python | MIT |
| IBM Granite 4.0-H 350M base (IBM-provided in the container) | https://huggingface.co/ibm-granite | Apache-2.0 |
| Mermaid (architecture diagram) | https://mermaid.js.org | MIT |
All SecondLook code in `src/`, `dashboard/`, `scripts/`, `tests/` was written during the datathon window (first commit 2026-10-17); the IBM loan demo notebook was not reused.

Docs: `docs/architecture.md`, `docs/runbook.md`, `docs/demo_script.md`, `docs/model_evaluation.md`, `docs/dataset_methodology.md`. LinuxONE evidence: `docs/proof/linuxone_evidence.txt`.
