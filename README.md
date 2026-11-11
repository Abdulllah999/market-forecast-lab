# Does sentiment improve a next-close forecast?

A Python forecasting experiment comparing Ridge regression with a simple “tomorrow equals today” baseline. The interesting result is that the simpler forecast wins on this synthetic series.

**Python · Pandas · Scikit-learn · Flask**

[Evaluation](artifacts/evaluation.json) · [Every held-out prediction](artifacts/predictions.csv) · [Leakage tests](test_app.py) · [Running interface](artifacts/demo.png)

## Main finding

| Model | Held-out MAE, synthetic price units |
| --- | ---: |
| Persistence: next close = current close | **0.7356** |
| Ridge, price features only | 0.8869 |
| Ridge, price + synthetic sentiment | 0.9177 |

The comparison covers **71 one-step-ahead origins**, using an expanding training window over the final 20% of eligible records. The first fold has 283 training rows. Sentiment does not improve this run. These are artificial prices, not investment returns or evidence of market predictability.

## Evaluation design

I chose a chronological evaluation because a random split could let future prices influence training. At each close `t`, the model uses only labels available by `t`, predicts the next close, and then advances one origin. StandardScaler is refitted inside each training window. Ridge alpha is fixed at 10; the held-out segment is not used to choose it.

The price-only ablation uses the same model and protocol, removing only sentiment. This separates “does a fitted model help?” from “does sentiment help?” A known weak lagged sentiment relationship is injected by the generator; that still does not guarantee a useful estimated forecast.

```text
Chronological CSV → past-only features → expanding-window fits → predictions → baseline comparison
```

Tests change future inputs and confirm that earlier feature values remain unchanged. They also check next-day label alignment, chronological input validation and finite forecasts.

## Run and reproduce

Python 3.12, from this repository:

```bash
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
. .venv/bin/activate
pip install -r requirements-dev.txt
python -m pytest -q
python evaluate.py
python app.py
```

Open http://127.0.0.1:5004. The Flask interface displays the latest synthetic origin and a next-close point estimate. No API keys or external downloads are required. GitHub Actions runs the Python tests and evaluation.

![Forecast interface after a local run](artifacts/demo.png)

## Code worth reviewing

- `core.py`: CSV validation, lagged features, fold construction, price-only ablation and forecasting.
- `test_app.py`: future-input perturbation and target-alignment checks.
- `data/synthetic_market.csv`: fixed input series; `core.make_data()` reproduces seed 19.
- `artifacts/predictions.csv`: actuals and all three predictions for each test origin.

## Limits and next step

This does not ingest news or real stock prices. The sentiment feature is generated, not extracted by an NLP service. Exchange calendars, adjusted prices, trading costs and prediction intervals are absent. The local development server is not a deployed service. Docker is provided as a recipe but has not been build-validated.

A useful next step is a licensed adjusted-price dataset and timestamped news, with publish-time availability recorded before forecasting. I would freeze a validation protocol and compare across multiple periods before adding model complexity.

Source code is MIT; the market fixture is authored synthetic data. See [data card](data/DATA_CARD.md), [provenance](PROVENANCE.md) and [licensing notes](LICENSE-REVIEW.md).
