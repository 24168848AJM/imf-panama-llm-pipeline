# IMF Article IV Report → LLM Insight Dashboard

Assignment 1, *Data Analysis with AI* (7PAM2032). An end-to-end pipeline that turns an unstructured IMF Article IV
Staff Report (PDF) into audited, machine-readable outputs and a 4-panel executive dashboard, using Gemini for
extraction and an independent Gemini model as an LLM-as-a-Judge.

**Assigned report:** Panama: 2025 Article IV Consultation (IMF Country Report No. 25/245, Aug 2025) · **SRN:** 24168848 · **Author:** Mohammed Abdul Jilani

## Pipeline

```
PDF ─► pypdf ─► clean (running headers/footers, line joins, NFKC) ─► heading-based sections
    ─► sentence-aligned token chunks (400 tok, 50 overlap, embedding-model tokenizer)
        ├─► section summaries (<100 words, verified + self-correcting) ─┐
        ├─► executive summary (page-cited)                               ├─► Judge (gemini-3.5-flash):
        ├─► theme scan (regex lexicon + LLM chunk classifier)            │   faithfulness / completeness /
        ├─► strengths & vulnerabilities (5–7 each)                       │   consistency, RAG triad,
        └─► bge-small embeddings ─► FAISS IndexFlatL2 ─► RAG             │   indicator verdicts
              ├─► per-indicator retrieval ─► JSON extraction ─► range + quote/number grounding ─┘
              ├─► time-series extraction ─► number grounding
              └─► grounded Q&A vs no-RAG baseline
    ─► pd.melt tidy frames ─► 4-panel dashboard (300 dpi) + model benchmark (extension)
```

## How to run (Google Colab)

1. Open `IMF_Article_IV_LLM_Pipeline.ipynb` in Colab.
2. Add your Gemini key in **Secrets (🔑)** as `GEMINI_API_KEY` and enable notebook access.
   *Not needed if the `cache/` folder from this repository is present, because every response is then replayed from disk.*
3. Provide the report in one of these ways: put `imf_country_report.pdf` next to the notebook, set `PDF_URL`
   in the configuration cell, or upload it when prompted.
4. `Runtime → Run all`. A first run takes about 5–8 min on the free tier (rate-limited). A cached re-run takes about 1 min.

To reproduce the exact reported numbers in Colab, clone the repo first so that `cache/` sits next to the notebook:

```python
  !git clone https://github.com/24168848AJM/imf-panama-llm-pipeline.git
  %cd imf-panama-llm-pipeline
```

## Outputs (`outputs/`)

| File | Contents |
|---|---|
| `section_summaries.json`, `executive_summary.json` | Summaries + judge scores |
| `themes.json` | 7-theme counts, density per 10k words, lexicon vs LLM agreement |
| `qualitative_insights.json` | Strengths and vulnerabilities with page citations |
| `core_indicators.json` | Target schema (null = fallback) |
| `indicators_audit.json` | Evidence quotes, pages, grounding status, judge verdicts, fallback reasons |
| `time_series.json` | Accepted and rejected multi-year series with grounding rates |
| `rag_results.json` | Retrieved chunks, answers, RAG-triad scores, no-RAG baseline |
| `extension_model_benchmark.json` | Schema compliance, latency, judge accuracy per model |
| `figures/*.png` | Dashboard and panels, 300 dpi |
| `llm_call_log.csv` | Every LLM call: model, cache hit, attempts, latency |

## Models and libraries

Gemini `gemini-3.5-flash-lite` (extractor) and `gemini-3.5-flash` (judge) via `google-genai`;
`BAAI/bge-small-en-v1.5` (Xiao et al., 2023) via `sentence-transformers`; FAISS (Johnson et al., 2019); `pypdf`;
`pydantic`; `pandas`; `seaborn`/`matplotlib`. The report document is © International Monetary Fund.

## Licence

Code released under the MIT License (see `LICENSE`).
