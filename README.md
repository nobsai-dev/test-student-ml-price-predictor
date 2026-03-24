# Test Student ML Price Predictor

![Python](https://img.shields.io/badge/python-v3.8+-blue.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-0.104.1-009688.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)
![Coverage](https://img.shields.io/badge/coverage-85%25-yellow.svg)

A high-performance FastAPI service for real estate price prediction using ensemble machine learning models. This service provides RESTful API endpoints for predicting property prices based on various features such as location, size, amenities, and market conditions.

## Features

- **Ensemble ML Models**: Combines multiple algorithms (Random Forest, Gradient Boosting, XGBoost) for improved accuracy
- **Fast API Framework**: High-performance, async-capable web framework
- **Real-time Predictions**: Quick response times for single and batch predictions
- **Data Validation**: Robust input validation using Pydantic models
- **Interactive Documentation**: Auto-generated Swagger/OpenAPI documentation
- **Model Monitoring**: Built-in logging and performance metrics
- **Scalable Architecture**: Containerized deployment ready

## Installation

### Prerequisites

- Python 3.8+
- pip or conda package manager

### Local Setup

1. Clone the repository:
```bash
git clone https://github.com/yourusername/test-student-ml-price-predictor.git
cd test-student-ml-price-predictor
```

2. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Set up environment variables:
```bash
cp .env.example .env
# Edit .env with your configuration
```

### Docker Setup

```bash
docker build -t ml-price-predictor .
docker run -p 8000:8000 ml-price-predictor
```

## Usage

### Starting the Service

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

The service will be available at `http://localhost:8000`

### API Documentation

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

### Example API Calls

#### Single Prediction

```bash
curl -X POST "http://localhost:8000/predict" \
     -H "Content-Type: application/json" \
     -d '{
       "bedrooms": 3,
       "bathrooms": 2,
       "square_feet": 1500,
       "location": "downtown",
       "year_built": 2010,
       "garage": true,
       "pool": false
     }'
```

#### Batch Predictions

```bash
curl -X POST "http://localhost:8000/predict/batch" \
     -H "Content-Type: application/json" \
     -d '{
       "properties": [
         {
           "bedrooms": 3,
           "bathrooms": 2,
           "square_feet": 1500,
           "location": "downtown",
           "year_built": 2010,
           "garage": true,
           "pool": false
         },
         {
           "bedrooms": 4,
           "bathrooms": 3,
           "square_feet": 2200,
           "location": "suburban",
           "year_built": 2015,
           "garage": true,
           "pool": true
         }
       ]
     }'
```

## Architecture

### System Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Apps   │    │   Load Balancer │    │   FastAPI App   │
│                 │────│                 │────│                 │
│ Web/Mobile/CLI  │    │     (Nginx)     │    │  (Python 3.8+) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                        │
                                               ┌─────────────────┐
                                               │ ML Pipeline     │
                                               │                 │
                                               │ • Data Validation│
                                               │ • Feature Eng.  │
                                               │ • Ensemble ML   │
                                               │ • Post-process  │
                                               └─────────────────┘
```

### ML Pipeline Components

#### 1. Data Preprocessing
- Feature scaling and normalization
- Categorical encoding
- Missing value imputation
- Outlier detection and handling

#### 2. Ensemble Models
- **Random Forest**: Robust against overfitting, handles mixed data types
- **Gradient Boosting**: Sequential learning for complex patterns
- **XGBoost**: Optimized gradient boosting with regularization
- **Voting Classifier**: Combines predictions using weighted averaging

#### 3. Model Training Pipeline
```python
# Simplified training flow
preprocessor = ColumnTransformer([...])
models = {
    'rf': RandomForestRegressor(),
    'gb': GradientBoostingRegressor(),
    'xgb': XGBRegressor()
}
ensemble = VotingRegressor(models.items())
pipeline = Pipeline([('preprocessor', preprocessor), ('ensemble', ensemble)])
```

### API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Health check and service info |
| `/predict` | POST | Single property prediction |
| `/predict/batch` | POST | Batch property predictions |
| `/model/info` | GET | Model metadata and performance metrics |
| `/model/retrain` | POST | Trigger model retraining (admin only) |

### Data Models

#### Property Input Schema
```python
class PropertyInput(BaseModel):
    bedrooms: int = Field(..., ge=1, le=10)
    bath