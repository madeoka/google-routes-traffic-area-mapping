# Google Routes API Traffic-Aware Area Mapping

A reusable Python/Jupyter workflow for evaluating road traffic conditions between a focal location and multiple surrounding entry points using the **Google Maps Platform Routes API**.

This repository is intended as a reusable research and teaching workflow. The public version is sanitized: project-specific locations, coordinates, credentials, and other sensitive information are replaced with explicit placeholders.

> **Important**
> This repository contains source code only. It does not include a Google Maps Platform API key, private study-area coordinates, confidential project identifiers, or redistributable Google Maps Platform traffic datasets.

---

## Overview

The workflow sends route requests between a configurable focal location and multiple surrounding entry points, extracts route and traffic information, and can export tabular and geospatial outputs for local analysis.

Typical uses include:

- traffic-aware accessibility analysis;
- route comparison around a study area;
- entry/exit corridor assessment;
- transport and infrastructure research;
- urban mobility monitoring;
- reproducible classroom demonstrations of API-based routing; and
- exploratory traffic-condition mapping.

The notebook supports both directions:

1. **focal location → entry point**
2. **entry point → focal location**

This makes it possible to compare inbound and outbound conditions around a study area.

---

## Repository Structure

```text
google-routes-traffic-area-mapping/
├── README.md
├── LICENSE
├── google-routes-traffic-area-mapping.ipynb
├── requirements.txt
├── .gitignore
└── output_google_routes_traffic/
```

Recommended treatment of the output directory:

```text
output_google_routes_traffic/
```

Keep this directory excluded from Git by default unless you have independently confirmed that the files you intend to publish may be redistributed.

---

## Main Features

- Configurable focal location.
- Configurable surrounding entry points.
- Bidirectional route requests.
- Live traffic-aware routing.
- Future scheduled departure-time requests.
- Optional alternative-route requests where available.
- Traffic-aware route summaries.
- Traffic segment extraction.
- CSV export.
- GeoJSON export for local geospatial analysis.
- PNG map generation.
- Local request-count safeguard.
- Explicit validation of public placeholders before API execution.
- API-key loading through an environment variable or hidden prompt.
- Error logging for failed requests.

---

## Requirements

Recommended environment:

- Python 3.10 or newer
- Jupyter Notebook, JupyterLab, VS Code, or another compatible notebook environment
- A Google Cloud project with the required Google Maps Platform routing service enabled
- A valid Google Maps Platform API key
- Billing configured as required by Google Maps Platform

### Python dependencies

```bash
pip install jupyter requests pandas matplotlib
```

A minimal `requirements.txt` may contain:

```text
requests
pandas
matplotlib
```

For reproducible research, consider pinning exact package versions after testing the notebook in your environment.

---

## Configuration

The public repository intentionally does **not** contain real study-area coordinates.

Replace all placeholder values before running the notebook.

### Required configuration

| Configuration | Public placeholder | Required replacement |
|---|---|---|
| Focal location name | `REPLACE_ME_FOCAL_LOCATION_NAME` | Study-area center/reference name |
| Focal latitude | `None` | Decimal-degree latitude |
| Focal longitude | `None` | Decimal-degree longitude |
| Entry-point names | `REPLACE_ME_ENTRY_XX` | Road or area entry-point names |
| Entry-point latitude | `None` | Decimal-degree latitude |
| Entry-point longitude | `None` | Decimal-degree longitude |
| Local timezone | Example: `Asia/Makassar` | Appropriate IANA timezone |
| Google Maps API key | Environment variable or hidden prompt | Your private API key |

### Example

```python
FOCAL_LOCATION = {
    "name": "REPLACE_ME_FOCAL_LOCATION_NAME",
    "latitude": None,   # REPLACE_ME
    "longitude": None,  # REPLACE_ME
}

AREA_ENTRY_POINTS = [
    {
        "name": "REPLACE_ME_ENTRY_01",
        "latitude": None,   # REPLACE_ME
        "longitude": None,  # REPLACE_ME
    },
    {
        "name": "REPLACE_ME_ENTRY_02",
        "latitude": None,   # REPLACE_ME
        "longitude": None,  # REPLACE_ME
    },
    # Add or remove entry points as required.
]
```

The notebook should validate these values before sending any API request.

---

## Coordinate Convention

Use decimal degrees in the following form:

```text
latitude, longitude
```

Example format:

```text
-8.123456, 115.123456
```

Do not include restricted, private, or confidential coordinates in a public repository.

---

## API-Key Security

Never hard-code a real API key in the notebook.

The recommended workflow is:

1. Check the `GOOGLE_MAPS_API_KEY` environment variable.
2. If it is unavailable, request the key through hidden input at runtime.
3. Never print the key to notebook output.
4. Never store the key in committed files.

### Linux/macOS

```bash
export GOOGLE_MAPS_API_KEY="YOUR_PRIVATE_GOOGLE_MAPS_API_KEY"
```

### Windows PowerShell

```powershell
$env:GOOGLE_MAPS_API_KEY="YOUR_PRIVATE_GOOGLE_MAPS_API_KEY"
```

### Python pattern

```python
import os
from getpass import getpass

GOOGLE_MAPS_API_KEY = os.getenv("GOOGLE_MAPS_API_KEY")

if not GOOGLE_MAPS_API_KEY:
    GOOGLE_MAPS_API_KEY = getpass("Enter your Google Maps API key: ")
```

### Security recommendations

- Restrict the API key in Google Cloud Console.
- Enable only the APIs required by the project.
- Apply appropriate API and application restrictions.
- Do not commit `.env` files.
- Do not save keys in notebook output cells.
- Do not include credentials in screenshots.
- Review Git history before making a repository public.
- If a key has ever been publicly exposed, rotate or revoke it. Deleting it only from the latest commit is not sufficient.

---

## Traffic Modes

The notebook can conceptually support two operating modes.

### `LIVE`

A request is evaluated using the traffic conditions available when the request is submitted.

When no explicit departure time is provided, the request time is effectively used as the departure time.

Use this mode for current traffic-aware analysis.

### `SCHEDULED`

A future local date and time is provided by the user and converted to an RFC 3339 timestamp before the request is submitted.

Use this mode for future traffic prediction or scenario comparison.

Example conceptual workflow:

```text
local date/time
    ↓
local timezone
    ↓
timezone-aware datetime
    ↓
RFC 3339 timestamp
    ↓
Routes API request
```

### Important limitation

For driving requests, a past `departureTime` should not be used as a historical-traffic reconstruction mechanism.

A future scheduled request represents a **traffic prediction**, not a historical traffic archive.

If historical traffic observations are required for research, use a data source that explicitly licenses and provides historical traffic data.

---

## Traffic-Aware Routing

Traffic-aware routing should be explicitly requested in the API request configuration.

Depending on the routing configuration, the Routes API can account for live traffic conditions when computing the route.

For predictive future travel-time analysis, traffic-model behavior may combine historical traffic information with live traffic information, depending on the selected model and how close the requested departure time is to the present.

Do not describe a predicted route as an observed historical traffic condition.

---

## Traffic-Aware Polyline Categories

Traffic-aware polyline intervals may use the following traffic categories:

- `NORMAL`
- `SLOW`
- `TRAFFIC_JAM`

If the notebook internally assigns an additional value such as:

```text
UNKNOWN
```

that value should be treated as a **local fallback or missing-value category**, not as an additional Google traffic-speed category.

This distinction is important when documenting or publishing results.

---

## Request Structure

For each entry point, the notebook may submit two origin-destination requests:

```text
focal location → entry point
entry point → focal location
```

Therefore:

```text
requests per cycle = number of entry points × number of directions
```

For example:

```text
8 entry points × 2 directions = 16 origin-destination requests
```

This count refers to the notebook's logical routing requests. Actual billable usage depends on the API configuration, requested features, SKU classification, retries, alternative routes, and Google's current billing rules.

Do not treat this calculation as a billing estimate.

---

## Alternative Routes

The notebook may request alternative routes where supported.

Alternative routes are **not guaranteed** to be returned for every origin-destination pair.

Code should therefore handle:

```python
0
```

additional alternatives without treating the response as an error.

---

## Local Request Safeguard

Example:

```python
LOCAL_MONTHLY_SAFETY_LIMIT = 100
```

This value is only a **local notebook safeguard** designed to reduce accidental repeated execution.

It is **not**:

- a Google Cloud quota;
- a Google billing limit;
- a monthly API entitlement;
- a guaranteed cost-control mechanism; or
- an authoritative record of Google Maps Platform usage.

Always verify actual requests, quotas, pricing, and billing in Google Cloud.

A local counter may be stored in:

```text
output_google_routes_traffic/_local_request_counter.json
```

Because it is a local execution artifact, it should normally remain excluded from version control.

---

## Running the Notebook

### 1. Clone the repository

```bash
git clone YOUR_REPOSITORY_URL
cd google-routes-traffic-area-mapping
```

### 2. Create a virtual environment

Linux/macOS:

```bash
python -m venv .venv
source .venv/bin/activate
```

Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure the API key

Set:

```text
GOOGLE_MAPS_API_KEY
```

in your environment.

### 5. Replace public placeholders

Configure:

- focal location;
- focal coordinates;
- entry-point names;
- entry-point coordinates;
- local timezone;
- request mode; and
- any optional route settings.

### 6. Run the notebook

```bash
jupyter notebook google-routes-traffic-area-mapping.ipynb
```

Run the notebook cells in order.

---

## Outputs

Depending on the enabled notebook sections, the workflow can generate:

- route-level CSV summaries;
- traffic-segment CSV files;
- GeoJSON route/segment geometry;
- error logs;
- PNG traffic-condition maps; and
- a local request-counter file.

Example local output structure:

```text
output_google_routes_traffic/
├── area_route_summary_YYYYMMDDTHHMMSS_LOCAL_LIVE.csv
├── area_traffic_segments_YYYYMMDDTHHMMSS_LOCAL_LIVE.csv
├── area_traffic_segments_YYYYMMDDTHHMMSS_LOCAL_LIVE.geojson
├── area_traffic_map_YYYYMMDDTHHMMSS_LOCAL_LIVE.png
└── _local_request_counter.json
```

For scheduled runs, filenames may use a corresponding scheduled-mode label.

---

## Output Interpretation

Route and segment outputs should be interpreted as API-derived routing information associated with the request configuration and request time.

They should not automatically be interpreted as:

- field-observed traffic measurements;
- permanent road characteristics;
- historical traffic archives;
- official transportation statistics; or
- independently validated congestion measurements.

For research use, document at minimum:

- request date and time;
- timezone;
- origin and destination definitions;
- routing preference;
- travel mode;
- requested departure time, if applicable;
- traffic model, if used;
- field mask / returned fields;
- whether alternative routes were requested;
- software version or repository commit; and
- any post-processing applied to the response.

---

## Data, Caching, Display, and Redistribution

Google Maps Platform services and returned content are governed by Google's applicable terms, policies, attribution requirements, and service-specific conditions.

Before publishing, redistributing, caching, storing, displaying, or creating derivative datasets from API results, review the current Google Maps Platform requirements that apply to your implementation.

In particular:

- do not assume that API-derived CSV or GeoJSON files may be publicly redistributed;
- do not use a source-code license as permission to redistribute third-party content;
- preserve required attribution where applicable;
- do not remove required notices or attribution;
- review storage/caching restrictions before retaining API content; and
- review the current rules before displaying Routes API results on maps or in public applications.

For this reason, generated output files are excluded from Git by default.

---

## Recommended `.gitignore`

```gitignore
# Python
__pycache__/
*.py[cod]
*.so
.venv/
venv/

# Jupyter
.ipynb_checkpoints/

# Credentials and secrets
.env
.env.*
*.key
*.pem
secrets.*
credentials.*

# Generated API-derived outputs
output_google_routes_traffic/

# Local logs
*.log

# IDE / editor files
.vscode/
.idea/

# OS files
.DS_Store
Thumbs.db
```

If you intentionally publish a small example output, verify its redistribution status first and document exactly how it was generated.

---

## Recommended `requirements.txt`

```text
requests
pandas
matplotlib
```

After validating the workflow, a reproducible release can use pinned versions.

---

## Reproducibility Recommendations

For academic or technical use, record the following for each analysis run:

```text
repository commit:
Python version:
requests version:
pandas version:
matplotlib version:
request timestamp:
local timezone:
routing preference:
travel mode:
traffic model:
departure time:
number of entry points:
bidirectional routing:
alternative routes enabled:
```

Do not publish sensitive coordinates if the study location is restricted.

---

## Public-Repository Security Checklist

Before every public push, search the repository and Git history for:

- API keys;
- OAuth tokens;
- passwords;
- `.env` files;
- private coordinates;
- confidential project names;
- internal project IDs;
- personal email addresses;
- account identifiers;
- absolute local filesystem paths;
- temporary debug output;
- notebook cells containing secrets;
- screenshots containing credentials;
- raw API responses that should not be redistributed; and
- generated files that you do not have permission to publish.

Useful local checks may include:

```bash
git status
git diff
git log --stat
```

Also review notebook cell outputs before committing.

---

## Responsible Use

This workflow should be used in accordance with:

- applicable laws and regulations;
- Google Maps Platform terms and policies;
- institutional research requirements;
- data-protection obligations;
- API usage restrictions; and
- any contractual restrictions affecting the study.

The repository does not grant rights to third-party services, datasets, map content, or traffic information.

---

## Research Use

When using this workflow in a publication, report the API-based nature of the traffic information and distinguish:

```text
observed data
```

from:

```text
API-derived traffic-aware routing information
```

and from:

```text
future traffic predictions
```

This distinction helps prevent overinterpretation of the results.

---

## Citation

If this repository is used in academic work, cite the specific archived release or repository version used for the analysis.

A generic citation template is:

```text
Author(s). (Year). Google Routes API Traffic-Aware Area Mapping
(Version X.Y). GitHub repository.
```

For a formal release, consider creating a versioned archive and persistent DOI.

---

## License

This project is licensed under the **MIT License**.

See:

```text
LICENSE
```

for the complete license text.

The MIT License applies only to the original source code and documentation distributed by this repository.

It does **not** grant rights to:

- Google Maps Platform services;
- Google Routes API content;
- Google Maps content;
- traffic information;
- third-party datasets;
- third-party software; or
- other externally licensed material.

Users are responsible for obtaining their own API credentials and for complying with all applicable third-party terms and licenses.

---

## Disclaimer

This repository is provided for research, educational, and technical-development purposes.

API responses, road networks, traffic conditions, available routes, pricing, quotas, product behavior, and platform requirements may change over time.

Always verify current Google Maps Platform documentation, pricing, policies, attribution requirements, and terms before operational or public deployment.

---

## Contributing

Contributions are welcome if they:

- do not expose API credentials or sensitive information;
- preserve the sanitized public configuration;
- avoid committing restricted API-derived content;
- include clear documentation for behavioral changes; and
- maintain compatibility with the repository's licensing and third-party obligations.

Before submitting a pull request, remove notebook outputs that contain private or non-redistributable information.

---

## Suggested Repository Name

```text
google-routes-traffic-area-mapping
```

Suggested notebook name:

```text
google-routes-traffic-area-mapping.ipynb
```

Suggested repository description:

```text
A reusable Python/Jupyter workflow for traffic-aware route analysis between a focal location and surrounding entry points using the Google Maps Platform Routes API.
```
