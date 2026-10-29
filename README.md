# StockSense

A full-stack stock forecasting and analysis dashboard that combines **LSTM neural networks**, **genetic-algorithm hyperparameter optimization**, and **Mamdani fuzzy logic** to turn historical market data into predictions and interpretable trading signals.

> **Disclaimer:** StockSense is an educational and experimental project. Its predictions and signals are not financial advice, and past or simulated performance does not guarantee future results. Always perform your own research before making financial decisions.

## What it does

- Downloads historical market data through [`yfinance`](https://github.com/ranaroussi/yfinance).
- Trains an LSTM model to forecast closing prices.
- Optionally searches for model hyperparameters with a genetic algorithm.
- Calculates technical indicators such as RSI and MACD.
- Converts model and indicator values into fuzzy Buy, Hold, and Sell-style signals.
- Serves results through a FastAPI backend and displays them in a browser dashboard.
- Can use checked-in prediction artifacts for fast results without retraining.

## Architecture

```text
Yahoo Finance / saved artifacts
              |
              v
        FastAPI backend
  data pipeline -> LSTM model
                 -> GA optimizer (optional)
                 -> fuzzy signal engine
              |
              v
     Express + EJS frontend
```

## Repository layout

```text
.
├── stocksense/
│   ├── backend/
│   │   ├── main.py                 # FastAPI application and API routes
│   │   ├── data_pipeline.py        # Data download and feature preparation
│   │   ├── lstm_model.py            # LSTM training and evaluation
│   │   ├── ga_optimizer.py          # Genetic-algorithm optimization
│   │   ├── fuzzy_engine.py          # Fuzzy signal generation
│   │   └── requirements.txt
│   ├── frontend/
│   │   ├── server.js               # Express static server
│   │   ├── views/                  # EJS page templates
│   │   ├── public/                 # Browser assets
│   │   └── package.json
│   ├── stocksense_artifacts/       # Pre-trained predictions and metrics
│   └── README.md
└── stocksense_colab_train_fixed.ipynb  # Multi-company training notebook
```

## Requirements

- Python 3.10+ recommended
- Node.js 18+ and npm
- Internet access for live Yahoo Finance data when using the training or live-analysis paths
- Enough memory for TensorFlow model training

## Quick start

### 1. Start the backend

From the repository root:

```bash
cd stocksense/backend
python -m venv .venv

# Linux/macOS
source .venv/bin/activate

# Windows PowerShell
# .venv\\Scripts\\Activate.ps1

pip install -r requirements.txt
uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

The backend will be available at:

- Dashboard API: <http://127.0.0.1:8000>
- Interactive API documentation: <http://127.0.0.1:8000/docs>
- OpenAPI schema: <http://127.0.0.1:8000/openapi.json>

### 2. Start the frontend

In a second terminal:

```bash
cd stocksense/frontend
npm install
npm start
```

Open <http://127.0.0.1:5500> in your browser.

The frontend expects the backend at `http://127.0.0.1:8000`. Start both services before using the dashboard.

## API reference

All endpoints are exposed under `/api`.

### List ticker suggestions

```http
GET /api/tickers
```

Returns the popular ticker symbols shown by the dashboard.

### Quick prediction

```http
GET /api/predict?ticker=AAPL
```

Runs a prediction using default hyperparameters. This is the fastest live-analysis route.

### Batch prediction

```http
GET /api/predict/batch?tickers=AAPL,MSFT,NVDA
```

Requests predictions for multiple comma-separated ticker symbols.

### Full analysis

```http
GET /api/analyze?ticker=AAPL&run_ga=false&use_artifacts=true
```

Parameters:

| Parameter | Default | Description |
| --- | --- | --- |
| `ticker` | `AAPL` | Yahoo Finance ticker symbol, for example `AAPL` or `RELIANCE.NS`. |
| `run_ga` | `false` | Runs genetic-algorithm hyperparameter optimization before training. This is slower. |
| `use_artifacts` | `true` | Uses the latest matching saved prediction artifact when available. |

Example with live retraining and GA optimization:

```bash
curl "http://127.0.0.1:8000/api/analyze?ticker=MSFT&run_ga=true&use_artifacts=false"
```

Typical responses include dates, actual and predicted prices, RSI, MACD, fuzzy signals, signal confidence, model hyperparameters, GA history, RMSE, MAE, and processing time.

## Saved artifacts and training

Prediction files and summary metrics are stored in `stocksense/stocksense_artifacts`. When `use_artifacts=true`, the API can serve an existing prediction file instead of retraining a model.

To regenerate multi-company artifacts, open `stocksense_colab_train_fixed.ipynb` in Google Colab or Jupyter and run the notebook cells in order. The notebook is intended for training and experimentation; the FastAPI service is the serving layer.

## Troubleshooting

- **Frontend cannot reach the API:** confirm that the backend is running on port `8000` and that the browser is opened through the Express server on port `5500`.
- **No live data is returned:** check the ticker symbol and your internet connection. Yahoo Finance availability can vary.
- **Analysis is slow:** use `/api/predict`, keep `run_ga=false`, or use saved artifacts.
- **TensorFlow installation issues:** use a supported Python version and install the pinned dependencies from `stocksense/backend/requirements.txt` inside a clean virtual environment.

## Technology stack

- Python, FastAPI, Uvicorn
- TensorFlow/Keras and scikit-learn
- pandas, NumPy, yfinance
- DEAP for genetic algorithms
- scikit-fuzzy for fuzzy inference
- Node.js, Express, and EJS

## License

No license has been specified for this repository yet. Treat the code as **all rights reserved** unless the repository owner adds a license.
