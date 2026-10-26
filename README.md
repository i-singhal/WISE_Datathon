# SecondLook

**Same qualifications. Different decisions. Who notices?**

Team WISE · IBM Z Datathon 2026
Theme: Real-Time AI for Critical Decisions

SecondLook audits historical HMDA lending decisions. It finds denied applications with financially comparable approved applications, queues them for human review, and reports demographic patterns as observed disparities. These patterns are not proof of discrimination.

## Explore the project

- [Architecture](docs/architecture.md)
- [Runbook](docs/runbook.md)
- [Demo script](docs/demo_script.md)
- [Model evaluation](docs/model_evaluation.md)
- [Dataset methodology](docs/dataset_methodology.md)
- [LinuxONE deployment evidence](docs/proof/linuxone_evidence.txt)
- [Presentation](SecondLook%20-%20Team%20WISE.pdf)

## Run locally

From the repository root:

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt

# Download and prepare the data.
.venv/bin/python src/data/download.py
.venv/bin/python src/data/preprocess.py

# Run matching, auditing, and evaluation.
.venv/bin/python -m src.models.run_matching
.venv/bin/python -m src.models.run_audit
.venv/bin/python -m src.models.run_eval

# Run the test suite.
.venv/bin/python -m pytest tests
```

Start the dashboard and API in separate terminals:

```bash
.venv/bin/streamlit run dashboard/app.py
```

```bash
.venv/bin/uvicorn src.api.main:app --port 8000
```

For a remote API, set `SL_API_URL=http://<linuxone>:8000` before starting the dashboard. See the [runbook](docs/runbook.md) for deployment details.

## Project status

Project notes describe a working local MVP, an API deployed on LinuxONE, and model sensitivity analysis with SHED and Census context. An ablation model is listed as pending. See the linked runbook and evaluation documents for implementation and deployment details.

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
