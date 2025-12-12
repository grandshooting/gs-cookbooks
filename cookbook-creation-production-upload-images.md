# Cookbook: Create a Production and Upload Images

This guide walks you through creating a new production and uploading images via the Grand Shooting API.

## Prerequisites

- A valid authentication token (Bearer, OAuth, or JWT)
- API base URL: `https://api.grand-shooting.com/v3`

## Authentication

All requests must include an authentication header:

```bash
# Option 1: Bearer Token
Authorization: Bearer {your_token}

# Option 2: OAuth Access Token
Authorization: access_token {your_oauth_token}
```

---

## Step 1: Create a Production

### Endpoint

```
POST /v3/production
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `smalltext` | string | Production name/label |
| `startdate` | string (ISO 8601) | Start date in ISO 8601 format |
| `shooting_method_id` | integer | Shooting method ID |

### Optional Fields

| Field | Type | Description | Default Value |
|-------|------|-------------|---------------|
| `enddate` | string (ISO 8601) | End date | startdate + 8 hours |
| `timezone` | string | Timezone | "Europe/Paris" |
| `tzoffset` | integer | Offset in minutes | 60 |
| `bench_template_id` | integer | Template ID to apply | - |
| `template_account_id` | integer | Account ID for template | - |
| `briefing` | string (URL) | URL to briefing document | - |
| `info` | object | Free-form information (key/value) | - |
| `info_model` | array | Schema for the info field | - |

### Example: Minimal Creation

```bash
curl -X POST "https://api.grand-shooting.com/v3/production" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "smalltext": "Spring 2025 Collection - Packshot",
    "startdate": "2025-03-15T09:00:00",
    "shooting_method_id": 1
  }'
```

### Example: Complete Creation

```bash
curl -X POST "https://api.grand-shooting.com/v3/production" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "smalltext": "Spring 2025 Collection - Packshot",
    "startdate": "2025-03-15T09:00:00",
    "enddate": "2025-03-15T18:00:00",
    "shooting_method_id": 1,
    "timezone": "Europe/Paris",
    "bench_template_id": 5,
    "briefing": "https://drive.google.com/file/d/xxx/briefing.pdf",
    "info": {
      "Photographer": "John Smith",
      "Stylist": "Jane Doe",
      "Client": "Brand XYZ"
    },
    "info_model": [
      {
        "key": "Photographer",
        "label": "Photographer",
        "type": "text"
      },
      {
        "key": "Stylist",
        "label": "Stylist",
        "type": "text"
      },
      {
        "key": "online_date",
        "label": "Online Date",
        "type": "date"
      }
    ]
  }'
```

### Response

Creation returns an array of "benches" (workflow steps) created:

```json
[
  {
    "bench_id": 100,
    "parent_id": 100,
    "root_id": 100,
    "account_id": 1,
    "ext_lib_id": "1",
    "smalltext": "Spring 2025 Collection - Packshot",
    "benchsteptype": 10,
    "benchstatus": 10,
    "shooting_method_id": 1,
    "step_label": "Live"
  },
  {
    "bench_id": 101,
    "parent_id": 100,
    "root_id": 100,
    "smalltext": "Spring 2025 Collection - Packshot",
    "benchsteptype": 20,
    "benchstatus": 10,
    "step_label": "Phase 1"
  }
]
```

**Important**: Note the `root_id` (here `100`) and the `bench_id` of the "Live" bench (here `101`) - you'll need these to upload images.

### Status Reference

**benchsteptype** (step type):
- `10`: Live (Capture)
- `20`: Post-production Phase 1
- `30`: Post-production Phase 2
- `40`: Export/Validation

**benchstatus** (production status):
- `10`: Active
- `20`: In progress
- `30`: Finished
- `35`: Archiving
- `40`: Archived
- `50`: Flushed

---

## Step 2: Upload Images

Two methods are available for uploading images.

### Method A: Direct Upload (binary file)

#### Endpoint

```
POST /v3/production/{bench_root_id}/bench/{bench_id}/upload
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | binary | Yes | The image file |
| `wait` | boolean | No | Wait for upload completion (default: true) |
| `path` | string | No | Destination path (default: "/") |
| `transfer_id` | integer | No | ID for plugin tracking |

#### Example

```bash
curl -X POST "https://api.grand-shooting.com/v3/production/100/bench/101/upload" \
  -H "Authorization: Bearer {your_token}" \
  -F "file=@/path/to/image.jpg" \
  -F "path=/" \
  -F "wait=true"
```

### Method B: Upload by URL (recommended for bulk uploads)

#### Endpoint

```
POST /v3/production/{bench_root_id}/bench/{bench_id}/upload/url
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `urls` | array | Yes | List of URLs to upload |
| `wait` | boolean | No | Wait for completion (default: false) |
| `path` | string | No | Base path (default: "/") |
| `allowed_type` | string | No | Allowed types: "all", "images", "images and videos" |
| `transfer_id` | integer | No | ID for tracking |

#### URL Object Structure

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | Image URL |
| `filename` | string | Yes | Filename |
| `subpath` | string | No | Subfolder (combined with path) |
| `ref` | string | No | Force a reference |
| `view_type_code` | string | No | Force a view type (e.g., "FRONT", "BACK") |
| `parent_id` | integer | No | Parent image ID |

#### Example: Simple Upload

```bash
curl -X POST "https://api.grand-shooting.com/v3/production/100/bench/101/upload/url" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "wait": true,
    "path": "/",
    "urls": [
      {
        "url": "https://storage.example.com/images/product_001_front.jpg",
        "filename": "product_001_front.jpg"
      },
      {
        "url": "https://storage.example.com/images/product_001_back.jpg",
        "filename": "product_001_back.jpg"
      }
    ]
  }'
```

#### Example: Upload with Reference Assignment

```bash
curl -X POST "https://api.grand-shooting.com/v3/production/100/bench/101/upload/url" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "wait": true,
    "allowed_type": "images and videos",
    "urls": [
      {
        "url": "https://storage.example.com/FW25_ALDA_PINK_front.jpg",
        "filename": "FW25_ALDA_PINK_front.jpg",
        "ref": "FW25_ALDA_PINK",
        "view_type_code": "FRONT"
      },
      {
        "url": "https://storage.example.com/FW25_ALDA_PINK_back.jpg",
        "filename": "FW25_ALDA_PINK_back.jpg",
        "ref": "FW25_ALDA_PINK",
        "view_type_code": "BACK"
      },
      {
        "url": "https://storage.example.com/FW25_ALDA_BLUE_front.jpg",
        "filename": "FW25_ALDA_BLUE_front.jpg",
        "ref": "FW25_ALDA_BLUE",
        "view_type_code": "FRONT"
      }
    ]
  }'
```

#### Response

```json
[
  {
    "url": "https://storage.example.com/FW25_ALDA_PINK_front.jpg",
    "status": "OK",
    "mime_type": "image/jpeg",
    "status_desc": "Success",
    "picture": {
      "picture_id": 1023,
      "bench_id": 101,
      "ref": "FW25_ALDA_PINK",
      "file_path": "JPG/FW25_ALDA_PINK-1.jpg",
      "picturestatus": 30
    }
  },
  {
    "url": "https://storage.example.com/FW25_ALDA_PINK_back.jpg",
    "status": "OK",
    "mime_type": "image/jpeg",
    "status_desc": "Success",
    "picture": {
      "picture_id": 1024,
      "bench_id": 101,
      "ref": "FW25_ALDA_PINK",
      "file_path": "JPG/FW25_ALDA_PINK-2.jpg",
      "picturestatus": 30
    }
  }
]
```

---

## Restrictions and Limits

- **Maximum size**: 50 GB per file
- **Target bench**: Cannot upload to a validation bench (benchsteptype: 40)
- **Archived production**: Cannot upload if benchstatus >= 35
- **Rate limiting**: 5 requests per second per account

---

## Complete Workflow: Python Example

```python
import requests

API_BASE = "https://api.grand-shooting.com/v3"
HEADERS = {
    "Authorization": "Bearer YOUR_TOKEN",
    "Content-Type": "application/json"
}

# Step 1: Create the production
production_data = {
    "smalltext": "Spring 2025 Collection",
    "startdate": "2025-03-15T09:00:00",
    "shooting_method_id": 1,
    "info": {
        "Photographer": "John Smith"
    }
}

response = requests.post(
    f"{API_BASE}/production",
    headers=HEADERS,
    json=production_data
)
benches = response.json()

# Get the IDs
bench_root_id = benches[0]["root_id"]
bench_live_id = next(b["bench_id"] for b in benches if b["benchsteptype"] == 10)

print(f"Production created: root_id={bench_root_id}, live_bench_id={bench_live_id}")

# Step 2: Upload images
upload_data = {
    "wait": True,
    "urls": [
        {
            "url": "https://storage.example.com/image1.jpg",
            "filename": "image1.jpg",
            "ref": "REF001"
        },
        {
            "url": "https://storage.example.com/image2.jpg",
            "filename": "image2.jpg",
            "ref": "REF002"
        }
    ]
}

response = requests.post(
    f"{API_BASE}/production/{bench_root_id}/bench/{bench_live_id}/upload/url",
    headers=HEADERS,
    json=upload_data
)

results = response.json()
for result in results:
    if result["status"] == "OK":
        print(f"✓ {result['picture']['ref']} uploaded (ID: {result['picture']['picture_id']})")
    else:
        print(f"✗ Error for {result['url']}: {result['status_desc']}")
```

---

## Common Error Codes

| Code | Description |
|------|-------------|
| 400 | Invalid request (check required fields) |
| 401 | Not authenticated |
| 404 | Production or bench not found |
| 423 | Resource locked (export in progress) |
| 429 | Rate limit exceeded |
| 500 | Server error |

---

## Additional Resources

- [Cookbook: Upload images to an existing production](./cookbook-upload-images-existing-production.md)
- [Cookbook: Reference management](./cookbook-reference-management.md)
