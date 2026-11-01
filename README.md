# FraudLens AI 🔍

Graduation project: an end-to-end **fraud detection** system on the **BankSim1** synthetic payment dataset, covering EDA, unsupervised and supervised models, explainability (SHAP), and a **chatbot** (LLM tool calling) that lets an analyst query transactions and take risky actions behind a per-session **OTP**.

> Everything is a demo: `fraud` / `isFraud` is a historical label and customer IDs are synthetic.

Dataset: [BankSim1 on Kaggle](https://www.kaggle.com/datasets/ealaxi/banksim1) (`bs140513_032310.csv`)

---

## 📊 Model Results (BankSim1, fraud class)

| Model | Precision (Fraud) | Recall (Fraud) | F1 (Fraud) |
|---|---|---|---|
| XGBoost (tuned) | 0.743 | 0.843 | 0.790 |
| CatBoost (tuned) | 0.729 | 0.844 | 0.783 |
| LightGBM (tuned) | 0.674 | 0.878 | 0.763 |
| Random Forest | 0.669 | 0.867 | 0.755 |
| XGBoost (base) | 0.666 | 0.867 | 0.753 |
| Logistic Regression | 0.645 | 0.881 | 0.745 |
| Decision Tree | 0.705 | 0.785 | 0.743 |
| XGBoost (Kaggle settings) | 0.673 | 0.809 | 0.735 |

Full table: [`reports/model_comparison.csv`](reports/model_comparison.csv)

Explainability outputs (SHAP for the supervised model, feature importance for LOF and One-Class SVM) are in [`reports/xai/`](reports/xai).

---

## 📂 Project Structure

```text
FraudLens-AI/
├── README.md
├── requirements.txt
├── .env.example            # copy to .env and fill in
├── .gitignore
├── main.py                 # FastAPI app
├── src/                    # chatbot backend
│   ├── configure.py        # settings (from .env)
│   ├── session.py          # Session (history + pending OTP) and ChatReply
│   ├── db.py               # queries + confirmed-only write actions
│   ├── confirmation.py     # OTP flow
│   ├── llm.py              # system prompt, tools, OpenRouter call
│   ├── chat.py             # chat(session, message) -> ChatReply
│   ├── build_db.py         # builds the SQLite DB from the BankSim CSV
│   ├── preprocessing.py    # sklearn transformers
│   └── cli.py              # terminal chat
├── frontend/               # browser UI
├── tests/                  # pytest (no LLM / key needed)
├── notebooks/
│   ├── 01_eda_and_unsupervised_models.ipynb
│   ├── 02_supervised_models_and_xai.ipynb
│   ├── 03_chatbot_prototype.ipynb
│   ├── 04_ieee_cis_eda.ipynb
│   └── 05_ieee_cis_preprocessing.ipynb
├── models/                 # final trained model (lr_final.joblib)
├── artifacts/              # preprocessing pipeline + preprocessed splits
├── reports/                # model comparison + XAI plots
├── docs/                   # project presentation (.pptx)
├── scripts/                # helper scripts (MLflow recovery)
└── data/                   # local SQLite DB goes here (git-ignored)
```

---

## ⚙️ Setup

```bash
git clone https://github.com/Hamdy812/FraudLens-AI.git
cd FraudLens-AI
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
# Set OPENROUTER_API_KEY in .env to enable LLM chat.
```

### Database (BankSim1)

Download the dataset from Kaggle, then:

```bash
python -m src.build_db --csv path/to/bs140513_032310.csv
# or choose the output path
python -m src.build_db --csv path/to/bs140513_032310.csv --out data/fraudlens_demo.db
```

Re-running resets the demo data (blocked customers, hidden transactions, audit log).

| BankSim column | App field | Notes |
|---|---|---|
| customer | card_id | full customer group ID |
| (hash) | card_short_id | 8-char short ID for the UI |
| amount | TransactionAmt | |
| fraud | isFraud | 0/1 label |
| category | category / ProductCD | e.g. es_food, es_travel |
| merchant | merchant | |
| step | step | simulation time step |

---

## ▶️ Run

```bash
pytest -q                                   # tests (no API key needed)
python -m src.cli                           # terminal chat
uvicorn main:app --reload --port 8000       # API
# in another terminal, serve the frontend
python -m http.server 5500 --directory frontend
# open http://localhost:5500 (API at http://localhost:8000)
```

### OTP / same-chat rule (important)
- The OTP is tied to the **same `session_id`**.
- Do **not** click "New chat" while a confirmation is pending.
- If the server restarts (`uvicorn --reload`), in-memory sessions are lost: request the action again in the **same** chat window.

### API notes
- One `Session` per user/conversation (dict keyed by `session_id`). Never share one Session between users.
- `chat(session, message)` returns a `ChatReply(type, text, data)`; types: `text`, `otp_required`, `result`.
- With `SHOW_DEMO_OTP=false` the OTP is never in the reply. Implement `deliver_otp` in `confirmation.py` first.

---

## 🛠️ Tech Stack

Python · pandas · scikit-learn · XGBoost / LightGBM / CatBoost · SHAP · MLflow · FastAPI · SQLite · OpenRouter (LLM tool calling) · HTML/JS

---

## 👨‍💻 Author

**Hamdy Osama**: [GitHub](https://github.com/Hamdy812) · [LinkedIn](https://www.linkedin.com/in/hamdy-osama)
