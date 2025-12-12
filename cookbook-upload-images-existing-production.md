# Cookbook: Upload Images to an Existing Production

This guide walks you through uploading images to an existing Grand Shooting production.

## Prerequisites

- A valid authentication token
- The production ID (`bench_root_id`)
- The target bench ID (`bench_id`)

## Authentication

```bash
Authorization: Bearer {your_token}
```

---

## Step 1: Identify the Production and Target Bench

### List Productions

```bash
curl -X GET "https://api.grand-shooting.com/v3/production" \
  -H "Authorization: Bearer {your_token}"
```

#### Response

**Note**: The response is a **nested array** (array of arrays). Each inner array represents one production with all its benches grouped together.

```json
[
  [
    {
      "bench_id": 100,
      "root_id": 100,
      "smalltext": "Spring 2025 Collection",
      "benchsteptype": 10,
      "benchstatus": 10,
      "startdate": "2025-03-15T09:00:00",
      "enddate": "2025-03-15T18:00:00"
    },
    {
      "bench_id": 101,
      "root_id": 100,
      "parent_id": 100,
      "smalltext": "Spring 2025 Collection (Phase 1)",
      "benchsteptype": 20,
      "benchstatus": 20,
      "startdate": "2025-03-15T09:00:00",
      "enddate": "2025-03-15T18:00:00"
    }
  ],
  [
    {
      "bench_id": 200,
      "root_id": 200,
      "smalltext": "Summer 2025 Campaign",
      "benchsteptype": 10,
      "benchstatus": 20,
      "startdate": "2025-04-01T09:00:00"
    }
  ]
]
```

**Tip**: To find the Live bench (where you can upload), look for `benchsteptype: 10`. The `root_id` is your `bench_root_id` for API calls.

### List Benches for a Production

```bash
curl -X GET "https://api.grand-shooting.com/v3/production/{bench_root_id}/bench" \
  -H "Authorization: Bearer {your_token}"
```

#### Example

```bash
curl -X GET "https://api.grand-shooting.com/v3/production/100/bench" \
  -H "Authorization: Bearer {your_token}"
```

#### Response

```json
[
  {
    "bench_id": 100,
    "root_id": 100,
    "benchsteptype": 10,
    "step_label": "Live",
    "benchstatus": 10
  },
  {
    "bench_id": 101,
    "root_id": 100,
    "benchsteptype": 20,
    "step_label": "Phase 1",
    "benchstatus": 10
  },
  {
    "bench_id": 102,
    "root_id": 100,
    "benchsteptype": 40,
    "step_label": "Export",
    "benchstatus": 10
  }
]
```

### Which Bench to Choose?

| benchsteptype | Label | Can receive uploads? |
|---------------|-------|---------------------|
| 10 | Live (Capture) | Yes |
| 20 | Post-production Phase 1 | Yes |
| 30 | Post-production Phase 2 | Yes |
| 40 | Export/Validation | No |

**Note**: Typically upload to the "Live" bench (benchsteptype: 10) or "Phase 1" bench (benchsteptype: 20).

---

## Step 2: Upload Images

### Method 1: Upload by URL (Recommended)

Ideal for bulk uploads from an external server.

#### Endpoint

```
POST /v3/production/{bench_root_id}/bench/{bench_id}/upload/url
```

#### Example: Simple Upload

```bash
curl -X POST "https://api.grand-shooting.com/v3/production/100/bench/100/upload/url" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "wait": true,
    "urls": [
      {
        "url": "https://cdn.example.com/photos/IMG_001.jpg",
        "filename": "IMG_001.jpg"
      },
      {
        "url": "https://cdn.example.com/photos/IMG_002.jpg",
        "filename": "IMG_002.jpg"
      },
      {
        "url": "https://cdn.example.com/photos/IMG_003.jpg",
        "filename": "IMG_003.jpg"
      }
    ]
  }'
```

#### Example: Upload with Folder Organization

```bash
curl -X POST "https://api.grand-shooting.com/v3/production/100/bench/100/upload/url" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "wait": true,
    "path": "/Session_Day1",
    "urls": [
      {
        "url": "https://cdn.example.com/photos/packshot_001.jpg",
        "filename": "packshot_001.jpg",
        "subpath": "Packshots"
      },
      {
        "url": "https://cdn.example.com/photos/lifestyle_001.jpg",
        "filename": "lifestyle_001.jpg",
        "subpath": "Lifestyle"
      }
    ]
  }'
```

Result:
- `/Session_Day1/Packshots/packshot_001.jpg`
- `/Session_Day1/Lifestyle/lifestyle_001.jpg`

#### Example: Upload with Reference Assignment

```bash
curl -X POST "https://api.grand-shooting.com/v3/production/100/bench/100/upload/url" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "wait": true,
    "urls": [
      {
        "url": "https://cdn.example.com/FW25_JACKET_BLK_F.jpg",
        "filename": "FW25_JACKET_BLK_F.jpg",
        "ref": "FW25_JACKET_BLACK",
        "view_type_code": "FRONT"
      },
      {
        "url": "https://cdn.example.com/FW25_JACKET_BLK_B.jpg",
        "filename": "FW25_JACKET_BLK_B.jpg",
        "ref": "FW25_JACKET_BLACK",
        "view_type_code": "BACK"
      },
      {
        "url": "https://cdn.example.com/FW25_JACKET_BLK_D.jpg",
        "filename": "FW25_JACKET_BLK_D.jpg",
        "ref": "FW25_JACKET_BLACK",
        "view_type_code": "DETAIL"
      }
    ]
  }'
```

#### Example: Upload Videos and Images

```bash
curl -X POST "https://api.grand-shooting.com/v3/production/100/bench/100/upload/url" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "wait": false,
    "allowed_type": "images and videos",
    "urls": [
      {
        "url": "https://cdn.example.com/product_photo.jpg",
        "filename": "product_photo.jpg"
      },
      {
        "url": "https://cdn.example.com/product_video.mp4",
        "filename": "product_video.mp4"
      }
    ]
  }'
```

#### Typical Response

```json
[
  {
    "url": "https://cdn.example.com/FW25_JACKET_BLK_F.jpg",
    "status": "OK",
    "mime_type": "image/jpeg",
    "status_desc": "Success",
    "picture": {
      "picture_id": 5001,
      "bench_id": 100,
      "ref": "FW25_JACKET_BLACK",
      "file_path": "JPG/FW25_JACKET_BLACK-1.jpg",
      "picturestatus": 30,
      "view_type_code": "FRONT"
    }
  },
  {
    "url": "https://cdn.example.com/FW25_JACKET_BLK_B.jpg",
    "status": "OK",
    "mime_type": "image/jpeg",
    "status_desc": "Success",
    "picture": {
      "picture_id": 5002,
      "bench_id": 100,
      "ref": "FW25_JACKET_BLACK",
      "file_path": "JPG/FW25_JACKET_BLACK-2.jpg",
      "picturestatus": 30,
      "view_type_code": "BACK"
    }
  }
]
```

### Method 2: Direct Upload (binary file)

For uploading a file from your local machine.

#### Endpoint

```
POST /v3/production/{bench_root_id}/bench/{bench_id}/upload
```

#### Example

```bash
curl -X POST "https://api.grand-shooting.com/v3/production/100/bench/100/upload" \
  -H "Authorization: Bearer {your_token}" \
  -F "file=@/Users/photographer/Photos/IMG_4523.jpg" \
  -F "path=/Session_March15" \
  -F "wait=true"
```

#### Multiple Upload with Bash Script

```bash
#!/bin/bash

API_BASE="https://api.grand-shooting.com/v3"
TOKEN="your_token"
BENCH_ROOT_ID=100
BENCH_ID=100
FOLDER="/Users/photographer/Photos/Session_March15"

for file in "$FOLDER"/*.jpg; do
  echo "Uploading: $file"
  curl -X POST "$API_BASE/production/$BENCH_ROOT_ID/bench/$BENCH_ID/upload" \
    -H "Authorization: Bearer $TOKEN" \
    -F "file=@$file" \
    -F "wait=true"
  echo ""
done
```

---

## Advanced Parameters

### The `wait` Parameter

| Value | Behavior |
|-------|----------|
| `true` | Request waits for processing to complete (synchronous) |
| `false` | Request returns immediately (asynchronous) |

**Recommendation**: Use `wait: true` for small uploads, `wait: false` for large volumes.

### The `allowed_type` Parameter

| Value | Accepted files |
|-------|----------------|
| `"images"` | Images only (JPG, PNG, TIFF...) |
| `"images and videos"` | Images and videos (MP4, MOV...) |
| `"all"` | All file types |

### Link an Image to a Parent Image

Useful for retouched versions or alternatives:

```json
{
  "urls": [
    {
      "url": "https://cdn.example.com/retouch_v2.jpg",
      "filename": "retouch_v2.jpg",
      "parent_id": 5001
    }
  ]
}
```

---

## Verify the Result

### List Production Images

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?bench_id=100" \
  -H "Authorization: Bearer {your_token}"
```

**Note**: Use the query parameter `bench_id` to filter images by bench.

#### Response

```json
[
  {
    "picture_id": 5001,
    "ref": "FW25_JACKET_BLACK",
    "file_path": "JPG/FW25_JACKET_BLACK-1.jpg",
    "picturestatus": 30,
    "view_type_code": "FRONT",
    "create_date": "2025-03-15T10:30:00"
  },
  {
    "picture_id": 5002,
    "ref": "FW25_JACKET_BLACK",
    "file_path": "JPG/FW25_JACKET_BLACK-2.jpg",
    "picturestatus": 30,
    "view_type_code": "BACK",
    "create_date": "2025-03-15T10:30:05"
  }
]
```

---

## Error Handling

### Common Errors

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| 400 | "Invalid bench" | Validation bench targeted | Use a Live or Phase 1 bench |
| 400 | "Production archived" | Production is archived | Reactivate the production or create a new one |
| 404 | "Production not found" | Invalid bench_root_id | Verify the production ID |
| 404 | "Bench not found" | Invalid bench_id | Verify the bench ID |
| 413 | "File too large" | File > 50 GB | Reduce file size |
| 429 | "Rate limit exceeded" | Too many requests | Wait and retry |

### Error Handling in URL Response

```json
[
  {
    "url": "https://cdn.example.com/image_ok.jpg",
    "status": "OK",
    "picture": { ... }
  },
  {
    "url": "https://cdn.example.com/image_error.jpg",
    "status": "ERROR",
    "status_desc": "Unable to download file: 404 Not Found"
  }
]
```

---

## Complete Python Example

```python
import requests
import time

API_BASE = "https://api.grand-shooting.com/v3"
TOKEN = "your_token"
HEADERS = {
    "Authorization": f"Bearer {TOKEN}",
    "Content-Type": "application/json"
}

def find_production_by_name(name):
    """Find a production by name."""
    response = requests.get(f"{API_BASE}/production", headers=HEADERS)
    productions = response.json()
    for prod in productions:
        if name.lower() in prod["smalltext"].lower():
            return prod
    return None

def get_live_bench(bench_root_id):
    """Get the Live bench of a production."""
    response = requests.get(
        f"{API_BASE}/production/{bench_root_id}/bench",
        headers=HEADERS
    )
    benches = response.json()
    for bench in benches:
        if bench["benchsteptype"] == 10:  # Live
            return bench
    return None

def upload_images(bench_root_id, bench_id, images):
    """Upload a list of images by URL."""
    payload = {
        "wait": True,
        "urls": images
    }

    response = requests.post(
        f"{API_BASE}/production/{bench_root_id}/bench/{bench_id}/upload/url",
        headers=HEADERS,
        json=payload
    )
    return response.json()

# --- Usage ---

# 1. Find the production
production = find_production_by_name("Spring 2025")
if not production:
    print("Production not found")
    exit(1)

print(f"Production found: {production['smalltext']} (ID: {production['root_id']})")

# 2. Get the Live bench
bench = get_live_bench(production["root_id"])
if not bench:
    print("Live bench not found")
    exit(1)

print(f"Live bench: {bench['bench_id']}")

# 3. Prepare images to upload
images_to_upload = [
    {
        "url": "https://cdn.example.com/new_photo_001.jpg",
        "filename": "new_photo_001.jpg",
        "ref": "SS25_DRESS_RED"
    },
    {
        "url": "https://cdn.example.com/new_photo_002.jpg",
        "filename": "new_photo_002.jpg",
        "ref": "SS25_DRESS_BLUE"
    }
]

# 4. Upload
print(f"Uploading {len(images_to_upload)} images...")
results = upload_images(production["root_id"], bench["bench_id"], images_to_upload)

# 5. Display results
success = 0
errors = 0
for result in results:
    if result["status"] == "OK":
        success += 1
        print(f"  ✓ {result['picture']['ref']} (ID: {result['picture']['picture_id']})")
    else:
        errors += 1
        print(f"  ✗ {result['url']}: {result['status_desc']}")

print(f"\nResult: {success} success, {errors} errors")
```

---

## Best Practices

1. **Check production status** before uploading (benchstatus < 35)
2. **Use `wait: false`** for large volumes and handle tracking client-side
3. **Batch uploads** in groups of 50-100 images maximum
4. **Respect rate limiting** (5 requests/second)
5. **Assign references** at upload time when possible to avoid manual processing
6. **Check results** and handle errors individually

---

## Additional Resources

- [Cookbook: Create a production and upload images](./cookbook-creation-production-upload-images.md)
- [Cookbook: Reference management](./cookbook-reference-management.md)
