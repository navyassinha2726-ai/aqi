# AetherShield: AI-Powered AQI Monitoring & Prediction System
AetherShield is a next-generation Air Quality Index (AQI) forecasting and atmospheric dispersion tracking application. By analyzing public weather conditions and gas concentrations, the system computes the **Atmospheric Stability Index (ASI)** to determine if pollutants are locked near the surface (**High Gravity** state) or dispersing dynamically (**Anti-Gravity** state) and uses a machine learning model to forecast AQI levels for the next 24 hours.
---
## System Architecture
The application is built on a distributed agentic microservices architecture consisting of:
1. **Data Ingestion (Open-Meteo)**: Free, keyless API fetching real-time and historical air quality indices ($PM_{2.5}, PM_{10}, O_3, NO_2, SO_2, CO$) and weather factors (Wind Speed, planetary boundary layer height).
2. **FastAPI Backend (`backend/`)**: Calculations for AQI, Atmospheric Stability states, and endpoints for metrics, forecasts, and model retraining.
3. **ML Time-Series Model (`scikit-learn`)**: A recursive autoregressive `RandomForestRegressor` that leverages time-lagged variables and forecasted weather elements to predict future AQI.
4. **Streamlit Frontend (`frontend/`)**: High-fidelity dashboard visualizing current criteria pollutants, dispersion trends, forecasting charts, and proactive warnings.
5. **Docker Containerization**: Configuration files for building Docker images and running the whole system via `docker-compose`.
```mermaid
graph TD
    OM[Open-Meteo Public API] -->|Historical + Forecast Meteorological & AQI Data| BE[FastAPI Backend]
    BE -->|Calculates Stability & Ingests Features| ML[Random Forest Time-Series Model]
    ML -->|Autoregressive 24h Predictions| BE
    BE -->|REST Endpoints: /aqi/current, /aqi/forecast| FE[Streamlit Dashboard UI]
    FE -->|Data Analysis & Alerts| User((User))
```
---
## Meteorological Formulation: Atmospheric Stability Index (ASI)
Atmospheric dispersion determines whether pollution remains suspended at ground level or escapes. We define the **Atmospheric Stability Index (ASI)** as:
$$ASI = \frac{\text{Wind Speed (m/s)} \times \text{Planetary Boundary Layer Height (m)}}{1000}$$
Where:
* **Wind Speed (m/s)**: Ingested wind speed converted from km/h ($v_{m/s} = v_{km/h} / 3.6$).
* **Planetary Boundary Layer (PBL) Height (m)**: The thickness of the lowest layer of the atmosphere where pollutants mix.
### Dispersion Classification
* **High Gravity (Stable / Pollutants Trapped)** ($ASI < 1.0$): Under low boundary layers and stagnant winds, air currents are locked. Pollutants cannot escape, triggering accumulation and high-risk breathing warnings.
* **Anti-Gravity (Unstable / Pollutants Dispersing)** ($ASI \ge 1.0$): Strong winds and elevated boundary layers act as a dispersion chimney, sweeping pollutants away rapidly.
---
## Getting Started (Local Setup)
### Option A: Using Docker Compose (Recommended)
This method spins up both the FastAPI backend and Streamlit dashboard automatically, isolated within a Docker virtual network.
1. Ensure you have **Docker** and **Docker Compose** installed.
2. From the project root directory, run:
   ```bash
   docker-compose up --build
   ```
3. Once running, access:
   - **Streamlit Frontend**: [http://localhost:8501](http://localhost:8501)
   - **FastAPI Documentation (Swagger)**: [http://localhost:8000/docs](http://localhost:8000/docs)
---
### Option B: Local Python Execution
#### 1. Start the Backend
1. Navigate to the backend directory:
   ```bash
   cd backend
   ```
2. Create a virtual environment and install dependencies:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```
3. Set your `PYTHONPATH` to the parent folder and run the FastAPI server:
   ```bash
   # On Windows (PowerShell)
   $env:PYTHONPATH=".."
   uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
   
   # On Linux/macOS
   PYTHONPATH=.. uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
   ```
#### 2. Start the Frontend Dashboard
1. Open a new terminal tab/window and navigate to the frontend directory:
   ```bash
   cd frontend
   ```
2. Create a virtual environment and install dependencies:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```
3. Start the Streamlit application:
   ```bash
   streamlit run app.py
   ```
4. Access the UI at [http://localhost:8501](http://localhost:8501).
---
## API Documentation
FastAPI automatically exposes an interactive OpenAPI specification. The core endpoints are:
* **`GET /health`**: Returns the health status of the services.
* **`GET /aqi/current?lat={latitude}&lon={longitude}`**: Ingests weather and air pollutants for the closest hour. Calculates the Atmospheric Stability Index and reports the current dispersion state ("High Gravity" vs "Anti-Gravity").
* **`GET /aqi/forecast?lat={latitude}&lon={longitude}`**: Triggers the machine learning service to run a 24-hour hourly forecast.
* **`POST /ml/train?lat={latitude}&lon={longitude}&past_days={days}`**: Dynamically downloads historical meteorology and pollutant values for the coordinate, transforms data into lagged features, trains a Random Forest model, and saves the weights.
---
## Deployment to Google Cloud Run
Google Cloud Run allows serverless execution of containerized applications. Below are the steps to deploy the Backend and Frontend services.
### Prerequisites
1. Install [Google Cloud CLI](https://cloud.google.com/sdk/docs/install).
2. Authenticate and configure your Google Cloud project:
   ```bash
   gcloud auth login
   gcloud config set project [YOUR_PROJECT_ID]
   ```
### Step 1: Deploy Backend to Cloud Run
1. Navigate to the `backend/` directory:
   ```bash
   cd backend
   ```
2. Build and deploy the container:
   ```bash
   gcloud run deploy aqi-backend \
     --source . \
     --region us-central1 \
     --allow-unauthenticated
   ```
3. Copy the returned URL (e.g., `https://aqi-backend-xxxxxx.a.run.app`).
### Step 2: Deploy Frontend to Cloud Run
1. Navigate to the `frontend/` directory:
   ```bash
   cd frontend
   ```
2. Deploy the container, passing the backend URL as an environment variable:
   ```bash
   gcloud run deploy aqi-frontend \
     --source . \
     --region us-central1 \
     --allow-unauthenticated \
     --set-env-vars BACKEND_URL=https://aqi-backend-xxxxxx.a.run.app
   ```
3. Access your live, cloud-deployed Streamlit dashboard at the provided URL!
---
## Agentic Roles and State Management
- **Researcher Agent**: Selected Open-Meteo as the unified endpoint for meteorological and gas levels. It requires zero authentication, operates at high frequency, and returns past historical data up to 90 days.
- **Backend Agent**: Established the FastAPI endpoints and calculates the Atmospheric Stability Index.
- **ML Agent**: Formulated the lag features and constructed a recursive prediction loop to feed predicted outputs as historical lags to later timestamps.
- **Frontend Agent**: Customized CSS styles for an Outfit font stack, custom metrics dashboard, and color-graded progress arcs.
- **Deployment Agent**: Separated docker configurations to allow deployment as decoupled, microservice-friendly Cloud Run containers.
