# Skillsphere21 — AI-Powered IDP System

An educational mini-project that demonstrates an AI-driven Individual Development Plan (IDP) generator using the 9-Box Performance–Potential concept. This repository contains a minimal backend (FastAPI) and a lightweight frontend to showcase how employee profiles can be analyzed and IDP recommendations generated.

## Features
- Sample data ingestion
- Simple skill-gap analysis and hybrid recommender stub
- REST API (FastAPI) to create / view recommendations
- Static frontend to demo the API
- Docker + docker-compose for easy launching
- Unit test scaffold

## Quickstart (local)

1. Backend (recommended to use a Python virtualenv):

```bash
cd backend/app
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 8000
