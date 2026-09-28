# Google Routes API Area Traffic Mapping

A reusable Python/Jupyter workflow for mapping road traffic conditions between a focal location and multiple surrounding entry points using the **Google Maps Platform Routes API**.

The public version intentionally removes project-specific location details and does **not** contain any API credentials.

## Repository files

- `google-routes-traffic-area-mapping.ipynb` — sanitized Jupyter notebook.
- `google-routes-traffic-area-mapping.md` — project documentation.
- `output_google_routes_traffic/` — generated CSV, GeoJSON, and PNG outputs; normally excluded from Git unless sample outputs are intentionally published.

## Important: replace configuration values before use

The following values are intentionally removed from the public version and **must be replaced**:

| Configuration | Public placeholder | What to provide |
|---|---|---|
| Focal location name | `REPLACE_ME_FOCAL_LOCATION_NAME` | Name of your study-area center/reference point |
| Focal latitude | `None` | Decimal-degree latitude |
| Focal longitude | `None` | Decimal-degree longitude |
| Entry-point names | `REPLACE_ME_ENTRY_XX` | Your own road/area entry-point names |
| Entry-point coordinates | `None` | Decimal-degree latitude/longitude |
| Local timezone | `Asia/Makassar` | Replace if your study area uses another IANA timezone |
| Google Maps API key | Environment variable / hidden prompt | Your private API key; **never commit it to GitHub** |

Example configuration:

```python
FOCAL_LOCATION = {
    "name": "REPLACE_ME_FOCAL_LOCATION_NAME",
    "latitude": None,   # REPLACE_ME
    "longitude": None,  # REPLACE_ME
}

AREA_ENTRY_POINTS = [
    {"name": "REPLACE_ME_ENTRY_01", "latitude": None, "longitude": None},
    {"name": "REPLACE_ME_ENTRY_02", "latitude": None, "longitude": None},
    # Add or remove points as required.
]
```

The notebook includes a validation step and will stop until these placeholders are replaced.

## API-key security

Do **not** write a real API key directly in the notebook. The notebook first checks the `GOOGLE_MAPS_API_KEY` environment variable and otherwise requests the key using hidden input.

### Linux/macOS

```bash
export GOOGLE_MAPS_API_KEY="YOUR_PRIVATE_GOOGLE_MAPS_API_KEY"
```

### Windows PowerShell

```powershell
$env:GOOGLE_MAPS_API_KEY="YOUR_PRIVATE_GOOGLE_MAPS_API_KEY"
```

Recommended precautions:

- Restrict the API key in Google Cloud Console.
- Enable only the APIs required by the project.
- Apply appropriate application/API restrictions.
- Do not commit `.env` files containing credentials.
- If a real key has ever been committed publicly, rotate/revoke it rather than merely deleting it from the latest commit.

## Main dependencies

```bash
pip install requests pandas matplotlib
```

Python's standard library is also used for JSON handling, date/time processing, paths, and timezone conversion.

## Traffic modes

### `LIVE`

Uses traffic conditions available when the request is submitted. No explicit `departureTime` is sent.

### `SCHEDULED`

Uses a user-defined **future** local date and time. The notebook converts the local time to an RFC3339 UTC timestamp before submitting it to the Routes API.

> Google Routes API `DRIVE` requests do not support using a past `departureTime` to reconstruct historical traffic. A scheduled request is therefore a traffic prediction, not a historical traffic archive.

## Request structure

For every configured entry point, the notebook can request:

1. focal location → entry point; and
2. entry point → focal location.

With eight entry points and bidirectional routing enabled, one complete cycle requires **16 origin–destination API requests**.

The notebook can also request alternative routes where available.

## Local request safeguard

```python
LOCAL_MONTHLY_SAFETY_LIMIT = 100
```

This counter is only a **local notebook safeguard**. It is not a Google Cloud quota, billing cap, or authoritative usage monitor. Always verify actual API usage and billing in Google Cloud Console.

## Outputs

The workflow can generate:

- route-level CSV summaries;
- traffic-segment CSV data;
- GeoJSON traffic-segment geometry;
- error logs when requests fail; and
- a PNG traffic-condition map.

Example output directory:

```text
output_google_routes_traffic/
├── area_route_summary_YYYYMMDDTHHMMSS_LOCAL_LIVE.csv
├── area_traffic_segments_YYYYMMDDTHHMMSS_LOCAL_LIVE.csv
├── area_traffic_segments_YYYYMMDDTHHMMSS_LOCAL_LIVE.geojson
├── area_traffic_map_YYYYMMDDTHHMMSS_LOCAL_LIVE.png
└── _local_request_counter.json
```

## Traffic categories

The visualization uses the traffic status returned by the API, including categories such as:

- `NORMAL`
- `SLOW`
- `TRAFFIC_JAM`
- `UNKNOWN`

## Suggested `.gitignore`

```gitignore
# Python / Jupyter
__pycache__/
.ipynb_checkpoints/

# Credentials
.env
.env.*
*.key

# Generated outputs
output_google_routes_traffic/

# OS files
.DS_Store
Thumbs.db
```

## Before publishing to GitHub

Check the repository for sensitive information before every public push. In particular, confirm that it does not contain:

- Google Maps API keys or other credentials;
- private coordinates or restricted study locations;
- personal email addresses or account identifiers;
- local absolute filesystem paths;
- confidential project names or internal IDs; or
- generated data that you do not have permission to redistribute.

## Data and platform terms

Use Google Maps Platform data, caching, storage, visualization, and attribution in accordance with the applicable Google Maps Platform terms and documentation. Review the current requirements before distributing derived datasets or persistent cached results.

## License

Add the license appropriate to your project before public release, for example MIT, Apache-2.0, GPL-3.0, or a custom research-data/software license.
