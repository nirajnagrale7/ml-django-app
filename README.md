# Django ML Iris Predictor

A minimal Django application that serves a trained scikit-learn model (RandomForest on the Iris dataset) via both a simple web UI and a JSON API.

- Web UI: form to submit sepal/petal measurements and view the prediction
- API: `POST /api/predict/` returns class and probabilities as JSON
- Model: trained with scikit-learn and saved with joblib

## Tech stack
- Python 3.10+
- Django 5
- scikit-learn, numpy, scipy
- joblib

## Project structure
```
django-ml-app/
├─ mlapp/                 # Django project settings and root URLs
├─ predictor/             # App with forms, views, services (inference)
│  ├─ model/              # Serialized model artifact
│  │  └─ iris_rf.joblib
│  ├─ forms.py
│  ├─ services.py         # Loads model and runs predictions
│  ├─ views.py            # Web + API views
│  ├─ urls.py             # App routes
│  └─ tests.py
├─ templates/
│  └─ predictor/
│     └─ predict_form.html
├─ train.py               # Script to train and (re)generate the model
├─ manage.py
└─ requirements.txt
```

## Setup
These commands assume your working directory is `django-ml-app/` (the same folder as `manage.py`).

1) Create and activate a virtual environment (Unix/macOS):
```
python3 -m venv .venv
source .venv/bin/activate
```

2) Install dependencies:
```
pip install -r requirements.txt
```

3) Run database migrations:
```
python manage.py migrate
```

4) (Optional) Retrain/regenerate the model artifact:
```
python train.py
```
This will produce `predictor/model/iris_rf.joblib`. The repo already contains a copy, but you can regenerate anytime.

5) Start the development server:
```
python manage.py runserver
```

- Web UI: http://127.0.0.1:8000/
- Admin (if you create a superuser): http://127.0.0.1:8000/admin/

## Usage
### Web UI
Open http://127.0.0.1:8000/ and submit the four Iris measurements to see the predicted class and per-class probabilities.

### API
- Endpoint: `POST /api/predict/`
- Content-Type: `application/json` (form-encoded also accepted)
- Body fields (all required): `sepal_length`, `sepal_width`, `petal_length`, `petal_width`

Example (JSON):
```
curl -s \
  -X POST http://127.0.0.1:8000/api/predict/ \
  -H 'Content-Type: application/json' \
  -d '{
        "sepal_length": 5.0,
        "sepal_width": 3.6,
        "petal_length": 1.4,
        "petal_width": 0.2
      }'
```

Example response:
```
{
  "class_index": 0,
  "class_name": "setosa",
  "probabilities": {
    "setosa": 0.98,
    "versicolor": 0.01,
    "virginica": 0.01
  }
}
```

## Tests
Run the unit tests:
```
python manage.py test
```

## Implementation notes
- The model is loaded and cached in `predictor/services.py` (function `get_model_bundle`).
- `train.py` trains a `RandomForestClassifier` on scikit-learn's Iris dataset and writes a bundle containing the estimator, target names, and feature names to `predictor/model/iris_rf.joblib`.
- The API view `predict_api` accepts JSON and returns a `JsonResponse` with the predicted class and probabilities. It is marked `csrf_exempt` for simplicity in local development; adjust for production environments.

## Troubleshooting
- Model not found errors: run `python train.py` to generate `predictor/model/iris_rf.joblib`.
- Dependency issues: ensure you are using Python 3.10+ and a clean virtual environment.
- Port already in use: either stop the other process or choose a different port, e.g. `python manage.py runserver 0.0.0.0:8001`.

## Production considerations
- Configure `ALLOWED_HOSTS` and `DEBUG` in `mlapp/settings.py` appropriately.
- Serve static files via a proper web server or Django’s `collectstatic` + CDN.
- Remove or replace `@csrf_exempt` on API views and add authentication/authorization as needed.

## License
Add your chosen license here.
