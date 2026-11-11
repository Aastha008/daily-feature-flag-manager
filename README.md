# Daily Feature Flag Manager

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100.0%2B-009688.svg)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-18.0-61DAFB.svg)](https://react.dev/)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)]()
[![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A high-performance, self-hosted web application and microservice for creating, managing, and evaluating feature toggles across multiple application environments. Built with FastAPI, React, and SQLite, **Daily Feature Flag Manager** provides real-time rule evaluation, deterministic percentage rollouts via Murmur3 hashing, context attribute rule matching, and visual flag evaluation telemetry.

---

## Table of Contents

- [Problem Statement & Target Audience](#problem-statement--target-audience)
- [Key Features](#key-features)
- [Architecture & System Design](#architecture--system-design)
- [Technology Stack](#technology-stack)
- [Installation & Virtual Environment Setup](#installation--virtual-environment-setup)
- [Environment Variables Configuration](#environment-variables-configuration)
- [Usage & API Reference](#usage--api-reference)
  - [CLI & Server Execution](#cli--server-execution)
  - [API Endpoints](#api-endpoints)
  - [Sample cURL Commands](#sample-curl-commands)
- [Automated Testing Instructions](#automated-testing-instructions)
- [Project Structure](#project-structure)
- [Roadmap & Future Enhancements](#roadmap--future-enhancements)
- [License](#license)

---

## Problem Statement & Target Audience

### The Problem
Decoupling software deployment from feature releases is a core requirement for modern continuous delivery pipelines. However, enterprise feature management SaaS platforms often present major drawbacks:
* High recurring costs based on monthly active users (MAUs) or evaluation volumes.
* Third-party cloud dependencies that introduce latency and potential data privacy violations.
* Network vendor lock-in and vendor availability risks.

### Target Audience
* **DevOps & Platform Engineering Teams**: Looking for a zero-cost, self-hostable service to control release risk across staging, development, and production environments.
* **Backend & Frontend Developers**: Who need microsecond-level flag evaluation response times with zero external network hops.
* **Product Managers & Quality Assurance**: Who require fine-grained target percentage rollouts and user-attribute targeting without touching source code.

---

## Key Features

* **Interactive Management Dashboard**: Visual interface built with React, TypeScript, and TailwindCSS for managing environments, editing flag rules, and configuring percentage rollouts.
* **High-Throughput REST API**: Ultra-low-latency evaluation engine built on FastAPI for evaluating toggle states against request context and custom user attributes.
* **Environment Isolation**: Supports isolated flag states and rule configurations across distinct deployment tiers (e.g., `development`, `staging`, `production`).
* **Deterministic Percentage Rollouts & Targeting**: Uses Murmur3 hashing to consistently bucket evaluation contexts for sticky percentage rollouts and context-attribute rule matching.
* **Built-in Telemetry Tracking**: Logs toggle evaluation counts, hit rates, cache performance, and latency metrics directly to SQLite.

---

## Architecture & System Design

The system follows a modular architecture separating data persistence, execution logic, web REST endpoints, and the frontend dashboard.

```
┌─────────────────────────────────────────────────────────────────┐
│                      React / Tailwind UI                        │
│                (Vite Single Page Application)                   │
└────────────────────────────────┘────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                    FastAPI Layer (src/api.py)                   │
│         - Admin CRUD Operations  - Context Evaluation           │
│         - Middleware & CORS      - Telemetry Telemetry Aggs     │
└────────────────────────────────┘────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Core Engine (src/core.py)                      │
│   - Murmur3 Hashing Engine      - Attribute Rule Evaluator       │
│   - In-Memory Flag Cache        - Telemetry Aggregator          │
└────────────────────────────────┘────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│               Data Access Layer (src/models.py)                 │
│         - SQLAlchemy ORM Models  - Pydantic Schemas             │
│         - SQLite Persistent Storage Engine                      │
└─────────────────────────────────────────────────────────────────┘
```

### Module Responsibilities

| Module | Location | Core Functionality |
| :--- | :--- | :--- |
| **Models & Schemas** | `src/models.py` | Defines SQLAlchemy ORM entities (`Environment`, `FeatureFlag`, `FlagRule`, `TelemetryRecord`) and Pydantic validation schemas. |
| **Core Engine** | `src/core.py` | Contains the flag evaluation engine (Murmur3 percentage hashing, rule condition matching, high-speed in-memory caching, and evaluation telemetry logging). |
| **API Layer** | `src/api.py` | Exposes REST endpoints for flag administration, real-time context evaluation, and visual telemetry querying. |
| **Main Launcher** | `src/main.py` | Manages application lifespan, database schema initialization, global CORS/middleware, and Uvicorn web server execution. |

---

## Technology Stack

### Backend
* **Python 3.10+**: Core programming language.
* **FastAPI**: Asynchronous web framework for low-latency REST evaluation endpoints.
* **Uvicorn**: High-performance ASGI server engine.
* **SQLAlchemy**: Relational database ORM.
* **Pydantic v2**: Data validation and strict settings management.
* **SQLite**: Lightweight, zero-configuration embedded relational database.

### Frontend
* **React 18**: User interface components.
* **TypeScript**: Type-safe client-side application logic.
* **TailwindCSS**: Utility-first CSS framework for modern styling.
* **Vite**: Next-generation frontend build tooling.

### Testing & Quality Assurance
* **pytest**: Automated unit and integration testing suite.
* **pytest-asyncio**: Async test execution support for FastAPI endpoints.

---

## Installation & Virtual Environment Setup

### Prerequisites
* Python `3.10` or higher
* `pip` and `venv` (Python packaging utilities)
* Node.js `v18+` and `npm` (for building the frontend dashboard)

### Step 1: Clone Repository & Create Virtual Environment

```bash
# Clone the repository
git clone https://github.com/your-org/daily-feature-flag-manager.git
cd daily-feature-flag-manager

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Linux/macOS:
source venv/bin/activate
# On Windows (Command Prompt):
# venv\Scripts\activate.bat
# On Windows (PowerShell):
# venv\Scripts\Activate.ps1
```

### Step 2: Install Backend Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## Environment Variables Configuration

Copy the provided `.env.example` file to create your local `.env` configuration file:

```bash
cp .env.example .env
```

### Configuration Options

| Variable | Default Value | Description |
| :--- | :--- | :--- |
| `APP_ENV` | `development` | Application runtime environment (`development`, `staging`, `production`). |
| `DATABASE_URL` | `sqlite:///./feature_flags.db` | SQLAlchemy connection string. |
| `SERVER_HOST` | `127.0.0.1` | Host interface IP for Uvicorn binding. |
| `SERVER_PORT` | `8000` | Port for Uvicorn web server execution. |
| `CACHE_TTL_SECONDS` | `60` | In-memory evaluation cache retention time. |
| `LOG_LEVEL` | `INFO` | Logging verbosity (`DEBUG`, `INFO`, `WARNING`, `ERROR`). |

---

## Usage & API Reference

### CLI & Server Execution

To run the backend server locally using Uvicorn:

```bash
# Launch server via Python launcher module
python -m src.main
```

Alternatively, launch directly using Uvicorn with auto-reload:

```bash
uvicorn src.main:app --host 127.0.0.1 --port 8000 --reload
```

The interactive API documentation (Swagger UI) will be accessible at:
* **Interactive Docs**: `http://127.0.0.1:8000/docs`
* **ReDoc Interface**: `http://127.0.0.1:8000/redoc`

---

### API Endpoints

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/api/v1/environments` | Register a new deployment environment (`dev`, `prod`, etc.). |
| `GET` | `/api/v1/flags` | List all registered feature flags. |
| `POST` | `/api/v1/flags` | Create a new feature flag with rules and rollout percentages. |
| `GET` | `/api/v1/flags/{key}` | Retrieve feature flag details and environment states. |
| `POST` | `/api/v1/evaluate` | **Evaluate flag state** for a target user context and attributes. |
| `GET` | `/api/v1/telemetry` | Query aggregated flag hit rates, call counts, and evaluation latency. |

---

### Sample cURL Commands

#### 1. Create a Feature Flag

```bash
curl -X 'POST' \
  'http://127.0.0.1:8000/api/v1/flags' \
  -H 'Content-Type: application/json' \
  -d '{
    "key": "new_checkout_flow",
    "description": "Enables redesigned checkout UI and payment integration",
    "enabled": true,
    "environment": "production",
    "rollout_percentage": 50,
    "rules": [
      {
        "attribute": "beta_tester",
        "operator": "EQUALS",
        "value": "true"
      }
    ]
  }'
```

#### 2. Evaluate Flag for Context

```bash
curl -X 'POST' \
  'http://127.0.0.1:8000/api/v1/evaluate' \
  -H 'Content-Type: application/json' \
  -d '{
    "flag_key": "new_checkout_flow",
    "environment": "production",
    "context_id": "user_102938",
    "attributes": {
      "beta_tester": "true",
      "country": "US"
    }
  }'
```

##### Example Response:
```json
{
  "flag_key": "new_checkout_flow",
  "enabled": true,
  "reason": "MATCHED_RULE",
  "evaluation_time_ms": 0.42,
  "context_id": "user_102938"
}
```

#### 3. Fetch Flag Telemetry Metrics

```bash
curl -X 'GET' 'http://127.0.0.1:8000/api/v1/telemetry?flag_key=new_checkout_flow'
```

---

## Automated Testing Instructions

The project includes unit and integration tests using `pytest` to cover models, evaluation engine hashing logic, and API endpoints.

### Execute All Tests

```bash
pytest
```

### Run Tests with Verbose Output

```bash
pytest -v
```

### Run Specific Test Suites

```bash
# Core evaluation engine tests
pytest tests/test_core.py -v

# API integration tests
pytest tests/test_api.py -v
```

### Code Coverage Reporting

```bash
pytest --cov=src tests/
```

---

## Project Structure

```
daily-feature-flag-manager/
├── .env.example          # Sample environment variable template
├── .gitignore             # Git ignore list for Python & Node artifacts
├── requirements.txt       # Python backend dependencies
├── src/
│   ├── __init__.py       # Package marker
│   ├── api.py            # FastAPI endpoints and route handlers
│   ├── core.py           # Evaluation logic, Murmur3 hashing, and cache engine
│   ├── main.py           # Application entrypoint & server runner
│   └── models.py         # SQLAlchemy database models and Pydantic schemas
└── tests/
    ├── __init__.py       # Test package marker
    ├── test_api.py       # API route integration tests
    └── test_core.py      # Core engine unit tests
```

---

## Roadmap & Future Enhancements

- [ ] **Multi-Tenant Authorization**: Add Role-Based Access Control (RBAC) and API keys for dashboard users and clients.
- [ ] **Streaming Updates (SSE / WebSockets)**: Push instant flag configuration updates to connected client SDKs without polling.
- [ ] **Audit Logs**: Maintain a history of flag modifications and user changes for compliance auditing.
- [ ] **Redis Cache Adapter**: Provide an optional distributed Redis caching layer alongside the in-memory cache for multi-instance horizontal scaling.

---

## License

This project is open-source software licensed under the [MIT License](LICENSE). Feel free to use, modify, and distribute it in both personal and commercial settings.