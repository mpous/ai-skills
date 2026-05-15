# Skill: edge-impulse-api

## Description
Helper for interacting with the Edge Impulse Studio API and Ingestion API. Use this skill when you need to manage projects, upload data, start training jobs, or deploy models programmatically.

## Preparation
Check for credentials in the environment before proceeding:
- `EI_API_KEY` — Edge Impulse API key (starts with `ei_`)
- `EI_PROJECT_ID` — numeric project ID

If either is missing, ask the user. They can find the API key in the project Dashboard under the **Keys** tab, and the project ID under **Project Info**.

## API Reference

Base URL: `https://studio.edgeimpulse.com/v1`
Ingestion URL: `https://ingestion.edgeimpulse.com`
OpenAPI spec: `https://studio.edgeimpulse.com/openapi.yml`

**Authentication**: All Studio API requests require the header: `x-api-key: <EI_API_KEY>`

### Key Endpoints

#### **Project & Samples**
- `GET /api/{projectId}`: Project info.
- `GET /api/`: List all user projects.
- `GET /api/{projectId}/raw-data`: List samples (supports `category`, `limit`, `offset`, `labels`).
- `GET /api/{projectId}/raw-data/{sampleId}`: Get a specific sample.
- `DELETE /api/{projectId}/raw-data/{sampleId}`: Delete a sample.
- `POST /api/{projectId}/raw-data/{sampleId}/rename`: Relabel a sample with `{ "newLabel": "label" }`.

#### **Data Ingestion (Ingestion API)**
Used to upload data to the platform.
- `POST /api/training/data`: Upload training sample.
- `POST /api/testing/data`: Upload testing sample.
- `POST /api/anomaly/data`: Upload anomaly sample.

**Headers**: `x-api-key`, `x-label`, `x-file-name`.
**Accepted Formats**: Edge Impulse JSON, audio/WAV, image/JPEG, CSV.

**Edge Impulse JSON Format**:
```json
{
  "protected": { "ver": "v1", "alg": "none", "iat": 1735689600 },
  "signature": "empty",
  "payload": {
    "device_name": "device",
    "device_type": "TYPE",
    "interval_ms": 10,
    "sensors": [{ "name": "accX", "units": "m/s2" }],
    "values": [[0.1], [0.2]]
  }
}
```

#### **Device Management**
- `GET /api/{projectId}/devices`: List devices.
- `POST /api/{projectId}/device/{deviceId}/start-sampling`: Start sampling using a body defining label, category, interval, and duration.

#### **Impulse & Training**
- `GET /api/{projectId}/impulse`: Get pipeline definition.
- `POST /api/{projectId}/impulse`: Update pipeline.
- `POST /api/{projectId}/jobs/train/{versionId}`: Start training job $\to$ returns `jobId`.
- `GET /api/{projectId}/jobs`: List jobs.
- `GET /api/{projectId}/jobs/{jobId}/status`: Check job status.
- `GET /api/{projectId}/jobs/{jobId}/stdout/download`: Download job log (plain text).

**Note**: Jobs are asynchronous. Poll `/jobs/{jobId}/ $\text{status}$` until `.job.finished` is `true`. If `.job.finishedSuccessful` is `false`, check the stdout log for errors. Monitor progress at: `https://studio.edgeimpulse.com/studio/<projectId>/jobs`.

#### **Deployment & Versions**
- `POST /api/{projectId}/deploy`: Start build (e.g., `deployType: "tflite"`).
- `GET  /api/{projectId}/deploy`: Get artifact download URL.
- `GET  /api/{projectId}/versions`: List snapshots.
- `POST /api/{projectId}/versions`: Create snapshot.

#### **Testing & EON Tuner**
- `POST /api/{projectId}/jobs/classify`: Classify a sample.
- `GET  /api/{projectId}/test-results`: Get model test results.
- `POST /api/{projectId}/tuner/start`: Start EON Tuner AutoML run.
- `GET  /api/{projectId}/tuner/runs`: List tuner runs and results.

### Implementation Pattern (Python)

```python
import os, requests

API_KEY = os.environ.get("EI_API_KEY") or input("API key: ")
PROJECT_ID = int(os.environ.get("EI_PROJECT_ID") or input("Project ID: "))
STUDIO = "https://studio.edgeimpulse.com/v1/api"
INGEST = "https://ingestion.edgeimpulse.com/api"

s = requests.Session()
s.headers.update({"x-api-key": API_KEY, "Accept": "application/json"})

def get(path, **params):
    r = s.get(f"{STUDIO}/{PROJECT_ID}{path}", params=params)
    r.raise_for_status()
    d = r.json()
    if not d.get("success"):
        raise RuntimeError(d.get("error", "unknown error"))
    return d

def post(path, body=None, base=STUDIO):
    # Handle both Studio and Ingestion base URLs
    base_url = f"{STUDIO}/{PROJECT_ID}{path}" if base == STUDIO else f"{base}{path}"
    r = s.post(base_url, json=body)
    r.raise_for_status()
    d = r.json()
    if not d.get("success"):
        raise RuntimeError(d.get("error", "unknown error"))
    return d
```
