# Skillsphere21 - AI-Powered IDP System 

##Name : Rohith V
##reg no : 212223040174

---

## Repository structure

```
Skillsphere21-IDP/
├── README.md
├── LICENSE
├── .gitignore
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   │   └── routes.py
│   │   ├── models/
│   │   │   └── schemas.py
│   │   ├── services/
│   │   │   ├── recommender.py
│   │   │   └── preprocess.py
│   │   ├── data/
│   │   │   └── sample_employees.json
│   │   └── requirements.txt
│   ├── Dockerfile
│   └── tests/
│       └── test_recommender.py
├── frontend/
│   ├── index.html
│   └── js/app.js
├── docker-compose.yml
├── .github/
│   └── workflows/ci.yml
└── CONTRIBUTING.md
```

---

> **Note:** This is a minimal, working starting point. The recommender is a stub demonstrating the data flow and how to integrate real ML models later (Scikit-learn/LightGBM/Transformer-based models).

---

## Files (copy each file content to the path indicated)

---

### README.md

````markdown
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
````

2. Open the frontend:

```bash
open ../frontend/index.html
# or serve it via a static file server
```

Or run everything with Docker Compose:

```bash
docker-compose up --build
```

## Project Structure

See the repository root for the full tree. The `backend` folder contains the API and recommendation logic. The `frontend` folder contains a tiny UI that calls the API.

## Extending the project

* Replace `services/recommender.py` with a production-ready model and add model versioning.
* Integrate with HRIS/LMS by implementing ingestion connectors.
* Add authentication/authorization (OAuth2 / JWT) and RBAC for role-based views.
* Implement XAI explanations for each recommendation.

## License

This project is released under the MIT License.

```
```

---

### LICENSE (MIT)

```text
MIT License

Copyright (c) 2025 Rohith V

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

[Full MIT text omitted for brevity — include full MIT license when creating repo]
```

---

### .gitignore

```
__pycache__/
venv/
.env
*.pyc
.DS_Store
.idea/
node_modules/
```

---

### backend/app/requirements.txt

```
fastapi
uvicorn[standard]
pydantic
numpy
pandas
pytest
```

---

### backend/app/main.py

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from api.routes import router

app = FastAPI(title="Skillsphere21 IDP API")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(router, prefix="/api")

@app.get("/")
def root():
    return {"message": "Skillsphere21 IDP API is running"}
```

---

### backend/app/api/routes.py

```python
from fastapi import APIRouter, HTTPException
from models.schemas import Employee, RecommendationResponse
from services.recommender import generate_recommendation

router = APIRouter()

@router.post('/recommend', response_model=RecommendationResponse)
def recommend(employee: Employee):
    try:
        rec = generate_recommendation(employee.dict())
        return rec
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

---

### backend/app/models/schemas.py

```python
from pydantic import BaseModel
from typing import List, Dict, Any

class Employee(BaseModel):
    id: str
    name: str
    role: str
    performance_rating: float  # 1-5
    potential_rating: float    # 1-5
    skills: Dict[str, float]   # e.g., {"python": 3.5, "sql": 4.0}
    target_roles: List[str] = []

class Recommendation(BaseModel):
    activity: str
    reason: str
    estimated_time_weeks: int
    kpi: Dict[str, Any]

class RecommendationResponse(BaseModel):
    employee_id: str
    employee_name: str
    nine_box: Dict[str, float]
    gaps: Dict[str, float]
    recommendations: List[Recommendation]
```

````

---

### backend/app/services/preprocess.py

```python
import numpy as np

def normalize_skills(skills: dict) -> dict:
    # simple min-max normalizer for demo
    vals = np.array(list(skills.values()), dtype=float)
    if len(vals) == 0:
        return {}
    mn, mx = vals.min(), vals.max()
    if mx == mn:
        return {k: 1.0 for k in skills}
    return {k: float((v - mn) / (mx - mn)) for k, v in skills.items()}
````

````

---

### backend/app/services/recommender.py

```python
from typing import Dict, Any
from models.schemas import Recommendation
from services.preprocess import normalize_skills

# a small competency framework for demo (in production fetch from DB)
ROLE_SKILL_REQUIREMENTS = {
    "data_scientist": {"python": 0.9, "ml": 0.85, "sql": 0.7},
    "data_engineer": {"python": 0.85, "sql": 0.9, "etl": 0.8},
}

RECOMMENDATION_POOL = [
    {"activity": "Online Course: Advanced ML", "tags": ["ml", "python"], "weeks": 6},
    {"activity": "Project Assignment: ETL pipeline", "tags": ["etl", "sql"], "weeks": 8},
    {"activity": "Mentorship: Senior Data Scientist", "tags": ["ml", "leadership"], "weeks": 12},
    {"activity": "Workshop: SQL optimization", "tags": ["sql"], "weeks": 2},
]


def compute_nine_box(perf: float, pot: float) -> Dict[str, float]:
    # return raw values and box id could be derived client-side
    return {"performance": perf, "potential": pot}


def generate_recommendation(employee: Dict[str, Any]) -> Dict[str, Any]:
    emp_id = employee.get('id')
    name = employee.get('name')
    skills = employee.get('skills', {})
    target_roles = employee.get('target_roles', []) or [employee.get('role')]

    norm_skills = normalize_skills(skills)

    # compute aggregated gaps by comparing to first target role
    target = target_roles[0]
    reqs = ROLE_SKILL_REQUIREMENTS.get(target.lower(), {})
    gaps = {}
    for skill, req_val in reqs.items():
        have = norm_skills.get(skill, 0.0)
        gap = round(max(0.0, req_val - have), 3)
        gaps[skill] = gap

    # pick recommendations by matching tags to largest gaps
    sorted_gaps = sorted(gaps.items(), key=lambda x: x[1], reverse=True)
    top_skill = sorted_gaps[0][0] if sorted_gaps else None

    recs = []
    for item in RECOMMENDATION_POOL:
        if top_skill and top_skill in item['tags']:
            recs.append({
                "activity": item['activity'],
                "reason": f"Addresses gap in {top_skill}",
                "estimated_time_weeks": item['weeks'],
                "kpi": {"skill_improvement": 0.2}
            })
    # fallback
    if not recs and RECOMMENDATION_POOL:
        recs.append({
            "activity": RECOMMENDATION_POOL[0]['activity'],
            "reason": "General development",
            "estimated_time_weeks": RECOMMENDATION_POOL[0]['weeks'],
            "kpi": {"skill_improvement": 0.1}
        })

    response = {
        "employee_id": emp_id,
        "employee_name": name,
        "nine_box": compute_nine_box(employee.get('performance_rating', 0), employee.get('potential_rating', 0)),
        "gaps": gaps,
        "recommendations": recs,
    }
    return response
````

---

### backend/app/data/sample_employees.json

```json
[
  {
    "id": "E001",
    "name": "Rohith V",
    "role": "Data_Scientist",
    "performance_rating": 3.8,
    "potential_rating": 4.2,
    "skills": {"python": 3.5, "ml": 2.5, "sql": 4.0},
    "target_roles": ["data_scientist"]
  }
]
```

---

### backend/Dockerfile

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY app/requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY app/ ./
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### backend/tests/test_recommender.py

```python
from services.recommender import generate_recommendation


def test_generate_recommendation_basic():
    employee = {
        "id": "T1",
        "name": "Test",
        "role": "data_scientist",
        "performance_rating": 3.0,
        "potential_rating": 3.0,
        "skills": {"python": 1.0, "ml": 1.0},
        "target_roles": ["data_scientist"]
    }
    res = generate_recommendation(employee)
    assert res['employee_id'] == "T1"
    assert 'recommendations' in res
```

---

### frontend/index.html

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Skillsphere21 Demo</title>
</head>
<body>
  <h1>Skillsphere21 — Demo</h1>
  <div>
    <label>Employee JSON (paste):</label><br>
    <textarea id="emp" rows="10" cols="80">{
  "id": "E001",
  "name": "Rohith V",
  "role": "data_scientist",
  "performance_rating": 3.8,
  "potential_rating": 4.2,
  "skills": {"python": 3.5, "ml": 2.5, "sql": 4.0},
  "target_roles": ["data_scientist"]
}</textarea><br>
    <button id="send">Generate IDP</button>
  </div>
  <pre id="out"></pre>
  <script src="js/app.js"></script>
</body>
</html>
```

---

### frontend/js/app.js

```javascript
const btn = document.getElementById('send');
btn.onclick = async () => {
  const txt = document.getElementById('emp').value;
  let obj = null;
  try { obj = JSON.parse(txt); } catch(e) { alert('Invalid JSON'); return; }
  const res = await fetch('http://localhost:8000/api/recommend', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(obj)
  });
  const data = await res.json();
  document.getElementById('out').textContent = JSON.stringify(data, null, 2);
}
```

---

### docker-compose.yml

```yaml
version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - '8000:8000'
    volumes:
      - ./backend/app:/app
```

---

### .github/workflows/ci.yml

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - name: Install dependencies
        run: |
          pip install -r backend/app/requirements.txt
      - name: Run tests
        run: |
          pytest -q backend/app/tests
```

---

### CONTRIBUTING.md

```markdown
# Contributing

Thank you for contributing! Please follow these guidelines:

- Create an issue before major changes.
- Use `feature/` branches and open a PR.
- Add tests for new functionality.
```

---

## How to convert this into a real GitHub repository quickly

1. Create a new repository on GitHub named `Skillsphere21-IDP`.
2. Clone it locally and create files/folders as above.
3. Copy each code block into the corresponding file and `git add . && git commit -m "Initial commit" && git push`.
4. Optionally enable GitHub Actions and add secrets for deployment.

---

## Next steps and recommended improvements

* Add authentication (OAuth2 + JWT) and RBAC for employee/manager/HR roles.
* Replace the stub recommender with a trained model (persist model with MLflow/s3).
* Add XAI explanations (SHAP or LIME) for each recommended activity.
* Add database (Postgres) and migrations (Alembic) for production use.
* Build a modern React/Next.js frontend with role-based dashboards.

---

If you'd like, I can now:

* generate these files and place them into a `.zip` for download (I can create them in the canvas), or
* initialize a Git repository on your local machine by providing step-by-step commands tailored to your environment.

(If you want me to produce a zip of the files inside the canvas, tell me and I'll add it.)
