# TerraVision AI — API Design

## 1. Overview

The TerraVision AI API provides programmatic access to land valuation forecasts, property information, market trends, investment scores, and model-generated explanations.

The API acts as the communication layer between:

* The machine learning prediction service
* The Snowflake analytics warehouse
* The Tableau or web dashboard
* Future mobile and third-party applications

The API will be developed using **FastAPI** because it provides strong performance, automatic API documentation, request validation, and easy integration with Python-based machine learning models.

---

## 2. API Objectives

The API should allow users and applications to:

* Search for land parcels or properties
* Retrieve property and location details
* Generate future land value predictions
* Retrieve existing forecasts
* Compare land markets
* View investment and risk scores
* Access model explanations
* Check service health

---

## 3. Base URL

### Local Development

```text
http://localhost:8000/api/v1
```

### Production

```text
https://api.terravision.ai/api/v1
```

---

## 4. API Versioning

The API will use URL-based versioning.

```text
/api/v1
```

Future versions may use:

```text
/api/v2
```

This approach allows the platform to introduce major changes without breaking existing applications.

---

## 5. Authentication

Authentication is not required for the first local MVP.

Future production versions will support API key or token-based authentication.

Example request header:

```http
Authorization: Bearer <access_token>
```

Possible future authentication methods:

* API keys
* OAuth 2.0
* JSON Web Tokens
* Role-based access control

---

## 6. Standard Response Format

Successful API responses should follow a consistent structure.

```json
{
  "success": true,
  "data": {},
  "message": "Request completed successfully",
  "request_id": "req_7c3042bf",
  "timestamp": "2026-07-16T18:30:00Z"
}
```

Error responses should follow this structure:

```json
{
  "success": false,
  "error": {
    "code": "PROPERTY_NOT_FOUND",
    "message": "The requested property could not be found.",
    "details": {}
  },
  "request_id": "req_7c3042bf",
  "timestamp": "2026-07-16T18:30:00Z"
}
```

---

# 7. Core Endpoints

## 7.1 Health Check

### Endpoint

```http
GET /health
```

### Purpose

Checks whether the API is available and whether its major dependencies are operating correctly.

### Example Response

```json
{
  "success": true,
  "data": {
    "service": "TerraVision AI API",
    "status": "healthy",
    "version": "1.0.0",
    "database": "connected",
    "model_service": "available"
  },
  "message": "Service is healthy",
  "request_id": "req_a123",
  "timestamp": "2026-07-16T18:30:00Z"
}
```

---

## 7.2 Search Properties

### Endpoint

```http
GET /properties
```

### Purpose

Searches for properties or parcels using location, parcel ID, county, city, ZIP code, or geographic coordinates.

### Query Parameters

| Parameter   | Type    | Required | Description                |
| ----------- | ------- | -------: | -------------------------- |
| `parcel_id` | string  |       No | Unique parcel identifier   |
| `address`   | string  |       No | Property address           |
| `city`      | string  |       No | City name                  |
| `county`    | string  |       No | County name                |
| `state`     | string  |       No | Two-letter state code      |
| `zip_code`  | string  |       No | ZIP code                   |
| `latitude`  | number  |       No | Property latitude          |
| `longitude` | number  |       No | Property longitude         |
| `min_acres` | number  |       No | Minimum land area          |
| `max_acres` | number  |       No | Maximum land area          |
| `page`      | integer |       No | Result page number         |
| `page_size` | integer |       No | Number of results per page |

### Example Request

```http
GET /api/v1/properties?city=Austin&state=TX&page=1&page_size=20
```

### Example Response

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "property_id": "prop_100245",
        "parcel_id": "TX-TRV-889102",
        "address": "1200 Example Road",
        "city": "Austin",
        "county": "Travis",
        "state": "TX",
        "zip_code": "78701",
        "latitude": 30.2672,
        "longitude": -97.7431,
        "land_area_acres": 2.35,
        "current_assessed_value": 185000,
        "last_sale_price": 172500,
        "last_sale_date": "2024-09-14"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total_items": 1,
      "total_pages": 1
    }
  },
  "message": "Properties retrieved successfully",
  "request_id": "req_b456",
  "timestamp": "2026-07-16T18:31:00Z"
}
```

---

## 7.3 Get Property Details

### Endpoint

```http
GET /properties/{property_id}
```

### Purpose

Returns detailed information for a specific property or parcel.

### Path Parameter

| Parameter     | Type   | Required | Description                      |
| ------------- | ------ | -------: | -------------------------------- |
| `property_id` | string |      Yes | Internal TerraVision property ID |

### Example Request

```http
GET /api/v1/properties/prop_100245
```

### Example Response

```json
{
  "success": true,
  "data": {
    "property_id": "prop_100245",
    "parcel_id": "TX-TRV-889102",
    "location": {
      "address": "1200 Example Road",
      "city": "Austin",
      "county": "Travis",
      "state": "TX",
      "zip_code": "78701",
      "latitude": 30.2672,
      "longitude": -97.7431
    },
    "property": {
      "land_area_acres": 2.35,
      "zoning_code": "R1",
      "land_use": "Residential",
      "road_access": true,
      "utilities_available": true
    },
    "valuation": {
      "current_assessed_value": 185000,
      "last_sale_price": 172500,
      "last_sale_date": "2024-09-14"
    },
    "risk": {
      "flood_zone": "Low",
      "wildfire_risk": "Low",
      "environmental_risk_score": 18
    }
  },
  "message": "Property details retrieved successfully",
  "request_id": "req_c789",
  "timestamp": "2026-07-16T18:32:00Z"
}
```

---

## 7.4 Generate Land Value Prediction

### Endpoint

```http
POST /predict
```

### Purpose

Generates current and future land value estimates using the latest available machine learning model.

### Request Body

```json
{
  "property_id": "prop_100245",
  "forecast_horizons": [1, 3, 5],
  "include_explanation": true
}
```

### Request Fields

| Field                 | Type    | Required | Description                    |
| --------------------- | ------- | -------: | ------------------------------ |
| `property_id`         | string  |      Yes | Property to evaluate           |
| `forecast_horizons`   | array   |      Yes | Forecast periods in years      |
| `include_explanation` | boolean |       No | Include explainability results |
| `model_version`       | string  |       No | Specific model version         |

### Example Response

```json
{
  "success": true,
  "data": {
    "prediction_id": "pred_928104",
    "property_id": "prop_100245",
    "model_version": "land-value-xgb-1.0.0",
    "prediction_date": "2026-07-16",
    "current_estimated_value": 188500,
    "forecasts": [
      {
        "year": 2027,
        "forecast_horizon_years": 1,
        "predicted_value": 201000,
        "growth_percentage": 6.63,
        "lower_confidence_value": 193500,
        "upper_confidence_value": 209200
      },
      {
        "year": 2029,
        "forecast_horizon_years": 3,
        "predicted_value": 224000,
        "growth_percentage": 18.83,
        "lower_confidence_value": 207000,
        "upper_confidence_value": 241500
      },
      {
        "year": 2031,
        "forecast_horizon_years": 5,
        "predicted_value": 252000,
        "growth_percentage": 33.69,
        "lower_confidence_value": 224000,
        "upper_confidence_value": 281000
      }
    ],
    "investment_score": 8.7,
    "risk_score": 2.4,
    "confidence_score": 0.89,
    "recommendation": "Strong long-term growth potential",
    "top_drivers": [
      {
        "feature": "population_growth",
        "impact": "positive",
        "importance": 0.24
      },
      {
        "feature": "median_income_growth",
        "impact": "positive",
        "importance": 0.19
      },
      {
        "feature": "distance_to_highway",
        "impact": "positive",
        "importance": 0.14
      },
      {
        "feature": "interest_rate",
        "impact": "negative",
        "importance": 0.11
      }
    ]
  },
  "message": "Prediction generated successfully",
  "request_id": "req_d012",
  "timestamp": "2026-07-16T18:33:00Z"
}
```

---

## 7.5 Retrieve Existing Forecast

### Endpoint

```http
GET /forecast/{property_id}
```

### Purpose

Returns the most recent stored forecast for a property without creating a new prediction.

### Query Parameters

| Parameter          | Type    | Required | Description                   |
| ------------------ | ------- | -------: | ----------------------------- |
| `forecast_horizon` | integer |       No | Forecast horizon in years     |
| `model_version`    | string  |       No | Filter by model version       |
| `latest_only`      | boolean |       No | Return only latest prediction |

### Example Request

```http
GET /api/v1/forecast/prop_100245?latest_only=true
```

### Example Response

```json
{
  "success": true,
  "data": {
    "property_id": "prop_100245",
    "prediction_id": "pred_928104",
    "current_estimated_value": 188500,
    "forecast_1_year": 201000,
    "forecast_3_year": 224000,
    "forecast_5_year": 252000,
    "confidence_score": 0.89,
    "generated_at": "2026-07-16T18:33:00Z"
  },
  "message": "Forecast retrieved successfully",
  "request_id": "req_e345",
  "timestamp": "2026-07-16T18:34:00Z"
}
```

---

## 7.6 Market Search

### Endpoint

```http
GET /markets
```

### Purpose

Returns land market analytics by city, county, state, ZIP code, or region.

### Query Parameters

| Parameter | Type    | Required | Description                              |
| --------- | ------- | -------: | ---------------------------------------- |
| `state`   | string  |       No | State filter                             |
| `county`  | string  |       No | County filter                            |
| `city`    | string  |       No | City filter                              |
| `sort_by` | string  |       No | Growth, value, risk, or investment score |
| `limit`   | integer |       No | Number of markets returned               |

### Example Request

```http
GET /api/v1/markets?state=TX&sort_by=projected_growth&limit=10
```

### Example Response

```json
{
  "success": true,
  "data": {
    "markets": [
      {
        "market_id": "market_travis_tx",
        "county": "Travis",
        "state": "TX",
        "average_land_value": 215000,
        "projected_3_year_growth": 17.8,
        "investment_score": 8.9,
        "risk_score": 3.1,
        "property_count": 18240
      },
      {
        "market_id": "market_hays_tx",
        "county": "Hays",
        "state": "TX",
        "average_land_value": 168000,
        "projected_3_year_growth": 16.2,
        "investment_score": 8.5,
        "risk_score": 2.8,
        "property_count": 9032
      }
    ]
  },
  "message": "Markets retrieved successfully",
  "request_id": "req_f678",
  "timestamp": "2026-07-16T18:35:00Z"
}
```

---

## 7.7 Market Summary

### Endpoint

```http
GET /markets/{market_id}
```

### Purpose

Returns detailed analytics for a selected market.

### Example Response

```json
{
  "success": true,
  "data": {
    "market_id": "market_travis_tx",
    "county": "Travis",
    "state": "TX",
    "average_land_value": 215000,
    "median_land_value": 198500,
    "average_price_per_acre": 86400,
    "year_over_year_growth": 5.9,
    "projected_3_year_growth": 17.8,
    "population_growth": 3.2,
    "median_income_growth": 4.1,
    "unemployment_rate": 3.4,
    "investment_score": 8.9,
    "risk_score": 3.1
  },
  "message": "Market summary retrieved successfully",
  "request_id": "req_g901",
  "timestamp": "2026-07-16T18:36:00Z"
}
```

---

## 7.8 Comparable Properties

### Endpoint

```http
GET /properties/{property_id}/comparables
```

### Purpose

Returns similar properties based on location, land size, zoning, market value, and recent sales.

### Query Parameters

| Parameter             | Type    | Required | Description                   |
| --------------------- | ------- | -------: | ----------------------------- |
| `radius_miles`        | number  |       No | Geographic search radius      |
| `max_results`         | integer |       No | Maximum comparable properties |
| `sale_recency_months` | integer |       No | Maximum sale age              |

### Example Response

```json
{
  "success": true,
  "data": {
    "property_id": "prop_100245",
    "comparables": [
      {
        "property_id": "prop_100390",
        "distance_miles": 1.8,
        "land_area_acres": 2.1,
        "sale_price": 181000,
        "sale_date": "2026-03-12",
        "price_per_acre": 86190,
        "similarity_score": 0.92
      }
    ]
  },
  "message": "Comparable properties retrieved successfully",
  "request_id": "req_h234",
  "timestamp": "2026-07-16T18:37:00Z"
}
```

---

## 7.9 Investment Analysis

### Endpoint

```http
GET /properties/{property_id}/investment-analysis
```

### Purpose

Returns investment potential, expected growth, risk indicators, and forecasted return.

### Example Response

```json
{
  "success": true,
  "data": {
    "property_id": "prop_100245",
    "investment_score": 8.7,
    "risk_score": 2.4,
    "growth_potential": "High",
    "estimated_3_year_roi_percentage": 18.83,
    "estimated_5_year_roi_percentage": 33.69,
    "liquidity_rating": "Moderate",
    "recommendation": "Potential long-term investment opportunity",
    "strengths": [
      "Strong population growth",
      "Improving median household income",
      "Close access to major highways",
      "Low environmental risk"
    ],
    "risks": [
      "Interest-rate sensitivity",
      "Limited short-term liquidity"
    ]
  },
  "message": "Investment analysis retrieved successfully",
  "request_id": "req_i567",
  "timestamp": "2026-07-16T18:38:00Z"
}
```

---

## 7.10 Model Explanation

### Endpoint

```http
GET /predictions/{prediction_id}/explanation
```

### Purpose

Returns the factors that contributed to a specific prediction.

### Example Response

```json
{
  "success": true,
  "data": {
    "prediction_id": "pred_928104",
    "explanation_method": "SHAP",
    "baseline_value": 175000,
    "predicted_value": 224000,
    "feature_contributions": [
      {
        "feature": "population_growth",
        "display_name": "Population Growth",
        "feature_value": 3.2,
        "contribution": 14500,
        "direction": "positive"
      },
      {
        "feature": "median_income_growth",
        "display_name": "Median Income Growth",
        "feature_value": 4.1,
        "contribution": 11200,
        "direction": "positive"
      },
      {
        "feature": "flood_risk_score",
        "display_name": "Flood Risk",
        "feature_value": 18,
        "contribution": -2200,
        "direction": "negative"
      }
    ],
    "summary": "The forecast is primarily driven by strong population and income growth, partially offset by moderate environmental risk."
  },
  "message": "Prediction explanation retrieved successfully",
  "request_id": "req_j890",
  "timestamp": "2026-07-16T18:39:00Z"
}
```

---

# 8. Administrative Endpoints

## 8.1 Model Information

### Endpoint

```http
GET /models
```

### Purpose

Returns information about currently deployed machine learning models.

### Example Response

```json
{
  "success": true,
  "data": {
    "models": [
      {
        "model_name": "land-value-xgb",
        "model_version": "1.0.0",
        "status": "active",
        "trained_at": "2026-07-01T10:00:00Z",
        "training_dataset_version": "dataset_2026_06",
        "mae": 12450,
        "rmse": 18290,
        "r2": 0.86
      }
    ]
  },
  "message": "Model information retrieved successfully",
  "request_id": "req_k123",
  "timestamp": "2026-07-16T18:40:00Z"
}
```

---

## 8.2 Data Freshness

### Endpoint

```http
GET /data-status
```

### Purpose

Reports the latest successful update time for each major data source.

### Example Response

```json
{
  "success": true,
  "data": {
    "sources": [
      {
        "source": "US Census",
        "last_updated": "2026-06-30T04:00:00Z",
        "status": "current"
      },
      {
        "source": "FRED",
        "last_updated": "2026-07-15T05:00:00Z",
        "status": "current"
      },
      {
        "source": "County Parcel Data",
        "last_updated": "2026-07-10T03:00:00Z",
        "status": "current"
      }
    ]
  },
  "message": "Data freshness retrieved successfully",
  "request_id": "req_l456",
  "timestamp": "2026-07-16T18:41:00Z"
}
```

---

# 9. HTTP Status Codes

| Status Code | Meaning                         |
| ----------: | ------------------------------- |
|       `200` | Request completed successfully  |
|       `201` | Resource created successfully   |
|       `400` | Invalid request                 |
|       `401` | Authentication required         |
|       `403` | Permission denied               |
|       `404` | Resource not found              |
|       `409` | Resource conflict               |
|       `422` | Request validation failed       |
|       `429` | Rate limit exceeded             |
|       `500` | Internal server error           |
|       `503` | Service temporarily unavailable |

---

# 10. Error Codes

| Error Code              | Description                              |
| ----------------------- | ---------------------------------------- |
| `INVALID_REQUEST`       | Request format or parameters are invalid |
| `PROPERTY_NOT_FOUND`    | Property ID does not exist               |
| `MARKET_NOT_FOUND`      | Market ID does not exist                 |
| `PREDICTION_NOT_FOUND`  | Prediction ID does not exist             |
| `MODEL_UNAVAILABLE`     | Prediction model is not available        |
| `DATA_NOT_AVAILABLE`    | Required input data is missing           |
| `DATABASE_ERROR`        | Data warehouse query failed              |
| `VALIDATION_ERROR`      | Request body validation failed           |
| `RATE_LIMIT_EXCEEDED`   | Too many requests                        |
| `INTERNAL_SERVER_ERROR` | Unexpected server error                  |

---

# 11. Pagination

Endpoints that return lists should support pagination.

### Example Request

```http
GET /properties?page=2&page_size=25
```

### Example Pagination Object

```json
{
  "page": 2,
  "page_size": 25,
  "total_items": 240,
  "total_pages": 10,
  "has_next": true,
  "has_previous": true
}
```

Default values:

```text
page = 1
page_size = 20
maximum_page_size = 100
```

---

# 12. Filtering and Sorting

List endpoints should support filtering and sorting where appropriate.

Example:

```http
GET /markets?state=TX&min_investment_score=8&sort_by=projected_growth&sort_order=desc
```

Supported sorting fields may include:

* Current value
* Predicted growth
* Investment score
* Risk score
* Price per acre
* Population growth
* Last sale date

---

# 13. Rate Limiting

Rate limiting will be introduced in the production version.

Example limits:

| User Type        |                  Requests |
| ---------------- | ------------------------: |
| Anonymous        |    30 requests per minute |
| Registered user  |   120 requests per minute |
| Premium user     | 1,000 requests per minute |
| Internal service |              Custom limit |

Example response headers:

```http
X-RateLimit-Limit: 120
X-RateLimit-Remaining: 94
X-RateLimit-Reset: 1721156400
```

---

# 14. Security Requirements

The production API should include:

* HTTPS-only communication
* Token-based authentication
* Input validation
* Request size limits
* Rate limiting
* SQL injection protection
* Secure secret management
* Structured audit logging
* Role-based authorization
* Personally identifiable information controls
* Dependency vulnerability scanning

Secrets must not be stored directly in source code.

Recommended secret storage options:

* AWS Secrets Manager
* Environment variables
* GitHub Actions secrets
* HashiCorp Vault

---

# 15. Data Validation

FastAPI and Pydantic should validate:

* Required fields
* Data types
* Valid state codes
* Valid latitude and longitude ranges
* Positive land area
* Supported forecast horizons
* Allowed sorting fields
* Maximum pagination size

Example validation rules:

```text
latitude: -90 to 90
longitude: -180 to 180
land_area_acres: greater than 0
forecast_horizon: 1, 3, or 5
page_size: 1 to 100
```

---

# 16. API Documentation

FastAPI automatically generates interactive documentation.

### Swagger UI

```text
http://localhost:8000/docs
```

### ReDoc

```text
http://localhost:8000/redoc
```

### OpenAPI JSON

```text
http://localhost:8000/openapi.json
```

These interfaces will allow developers to inspect endpoints and test requests directly.

---

# 17. Logging and Monitoring

Each API request should produce structured logs containing:

* Request ID
* Endpoint
* HTTP method
* Response status
* Processing time
* User or API key identifier
* Model version
* Database query duration
* Error details

Example log:

```json
{
  "request_id": "req_d012",
  "method": "POST",
  "endpoint": "/api/v1/predict",
  "status_code": 200,
  "duration_ms": 438,
  "model_version": "land-value-xgb-1.0.0",
  "timestamp": "2026-07-16T18:33:00Z"
}
```

Future monitoring tools may include:

* AWS CloudWatch
* Prometheus
* Grafana
* Datadog
* Sentry

---

# 18. Performance Targets

| Metric                    |      MVP Target |
| ------------------------- | --------------: |
| Health check response     |    Under 200 ms |
| Property search           | Under 2 seconds |
| Stored forecast retrieval |  Under 1 second |
| New prediction request    | Under 5 seconds |
| API availability          |     99% for MVP |
| Maximum request payload   |            1 MB |

Performance targets may change as the project scales.

---

# 19. Caching Strategy

Frequently requested data may be cached to improve performance.

Good candidates for caching:

* Market summaries
* Existing property forecasts
* Public economic indicators
* Model metadata
* Geographic lookup values

Potential caching technologies:

* Redis
* AWS ElastiCache
* In-memory caching for the MVP

Predictions should include a generation timestamp so users know when the result was created.

---

# 20. Idempotency

Prediction requests may support an idempotency key to prevent duplicate processing.

Example header:

```http
Idempotency-Key: 4e8ff1fa-128c-4bfd-a120-f76509c20de1
```

When the same request is submitted with the same idempotency key, the API should return the original response rather than generating a duplicate prediction.

---

# 21. Future API Endpoints

The following endpoints may be added after the MVP:

```http
POST /auth/register
POST /auth/login
GET /users/me
GET /portfolios
POST /portfolios
POST /portfolios/{portfolio_id}/properties
GET /alerts
POST /alerts
GET /recommendations
POST /reports
GET /reports/{report_id}
GET /zoning
GET /environmental-risks
GET /infrastructure-projects
POST /compare-properties
POST /opportunity-search
```

---

# 22. MVP Endpoint Summary

| Method | Endpoint                                        | Purpose                         |
| ------ | ----------------------------------------------- | ------------------------------- |
| `GET`  | `/health`                                       | Check API health                |
| `GET`  | `/properties`                                   | Search properties               |
| `GET`  | `/properties/{property_id}`                     | Retrieve property details       |
| `POST` | `/predict`                                      | Generate land value prediction  |
| `GET`  | `/forecast/{property_id}`                       | Retrieve stored forecast        |
| `GET`  | `/markets`                                      | Search land markets             |
| `GET`  | `/markets/{market_id}`                          | Retrieve market analytics       |
| `GET`  | `/properties/{property_id}/comparables`         | Retrieve comparable properties  |
| `GET`  | `/properties/{property_id}/investment-analysis` | Retrieve investment analysis    |
| `GET`  | `/predictions/{prediction_id}/explanation`      | Retrieve prediction explanation |
| `GET`  | `/models`                                       | Retrieve model metadata         |
| `GET`  | `/data-status`                                  | Retrieve source freshness       |

---

# 23. Initial FastAPI Project Structure

```text
api/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── dependencies.py
│   ├── routers/
│   │   ├── health.py
│   │   ├── properties.py
│   │   ├── predictions.py
│   │   ├── markets.py
│   │   └── models.py
│   ├── schemas/
│   │   ├── common.py
│   │   ├── property.py
│   │   ├── prediction.py
│   │   └── market.py
│   ├── services/
│   │   ├── property_service.py
│   │   ├── prediction_service.py
│   │   ├── market_service.py
│   │   └── explanation_service.py
│   ├── repositories/
│   │   ├── property_repository.py
│   │   ├── prediction_repository.py
│   │   └── market_repository.py
│   ├── models/
│   ├── middleware/
│   │   ├── request_id.py
│   │   ├── logging.py
│   │   └── error_handler.py
│   └── utils/
├── tests/
│   ├── test_health.py
│   ├── test_properties.py
│   ├── test_predictions.py
│   └── test_markets.py
├── requirements.txt
├── Dockerfile
└── README.md
```

---

# 24. Design Principles

The TerraVision AI API should follow these principles:

1. **Consistency**
   Use predictable endpoint names, response formats, and error messages.

2. **Separation of concerns**
   API routes, business logic, database access, and machine learning inference should remain separate.

3. **Versioning**
   Breaking changes must be released under a new API version.

4. **Explainability**
   Prediction endpoints should expose confidence scores and major contributing factors.

5. **Security**
   Validate all inputs and protect credentials and sensitive data.

6. **Observability**
   Every request should be traceable through logs and request IDs.

7. **Scalability**
   The architecture should allow independent scaling of the API, database, and prediction service.

8. **Reproducibility**
   Every prediction should record the model version and data version used.

---

# 25. Conclusion

The TerraVision AI API will provide a secure and scalable interface for property search, land value prediction, market analytics, investment intelligence, and explainable machine learning.

The MVP will begin with a small group of essential endpoints. Additional authentication, portfolio management, alerts, recommendation, and reporting capabilities will be added as the platform evolves.
