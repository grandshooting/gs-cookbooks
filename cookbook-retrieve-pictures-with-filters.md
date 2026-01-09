# Cookbook: Retrieve Pictures with Filters

This guide explains how to retrieve pictures from the Grand Shooting API using various filters, including date ranges, status, and exports. It also covers the pagination pattern for handling large volumes of data.

## Prerequisites

- A valid authentication token
- API base URL: `https://api.grand-shooting.com/v3`

## Authentication

```bash
Authorization: Bearer {your_token}
```

---

## Understanding Picture Filters

The `GET /v3/picture` endpoint supports powerful filtering capabilities:

- **Exact match**: `field=value`
- **Multiple values (OR)**: `field=value1&field=value2`
- **Comparisons**: `field=gt:value`, `field=gte:value`, `field=lt:value`, `field=lte:value`
- **Combined ranges**: `field=gte:min&field=lte:max`

### Available Filter Fields

| Field | Type | Description |
|-------|------|-------------|
| `picture_id` | integer | Unique picture identifier |
| `bench_id` | integer | Bench (workflow step) ID |
| `bench_root_id` | integer | Production ID |
| `benchsteptype` | integer | Workflow step type (10=Live, 20=Phase1, 30=Phase2, 40=Export) |
| `picturestatus` | integer | Picture workflow status |
| `date_cre` | date | Creation date in Grand Shooting |
| `date_mod` | date | Last modification date |
| `transfer_date` | date | Transfer date |
| `validation_date` | date | Validation date |
| `ref` | string | Reference extracted from picture |
| `reference_ref` | string | Linked catalog reference |
| `reference_id` | integer | Linked reference ID |
| `export` | string | Export name |
| `view_type_code` | string | View type code |
| `shootingmethod` | string | Shooting method name |

### Picture Status Values (`picturestatus`)

| Value | Meaning |
|-------|---------|
| 1 | Ignored |
| 5 | To reshoot |
| 10 | Not selected |
| 30 | Selected |
| 31 | Refused |
| 35 | Refused |
| 40 | Submitted for approval |
| 50 | Validated |
| 51 | Ready to broadcast |
| 52 | Broadcast error |
| 55 | Broadcast |
| 80 | Archived |

---

## Use Case 1: Filter Pictures by Date Range

### Pictures Created After a Specific Date

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?date_cre=gt:2025-01-01" \
  -H "Authorization: Bearer {your_token}"
```

### Pictures Created Before a Specific Date

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?date_cre=lt:2025-06-01" \
  -H "Authorization: Bearer {your_token}"
```

### Pictures Created Within a Date Range

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?date_cre=gte:2025-01-01&date_cre=lte:2025-03-31" \
  -H "Authorization: Bearer {your_token}"
```

### Pictures Modified in the Last 7 Days

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?date_mod=gte:2025-01-02" \
  -H "Authorization: Bearer {your_token}"
```

### Pictures Validated in a Specific Period

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?validation_date=gte:2025-01-01&validation_date=lte:2025-01-31" \
  -H "Authorization: Bearer {your_token}"
```

---

## Use Case 2: Filter Pictures by Status

### Get All Validated Pictures

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?picturestatus=50" \
  -H "Authorization: Bearer {your_token}"
```

### Get All Selected Pictures (Not Yet Validated)

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?picturestatus=30" \
  -H "Authorization: Bearer {your_token}"
```

### Get Pictures Pending Approval

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?picturestatus=40" \
  -H "Authorization: Bearer {your_token}"
```

### Get Pictures with Multiple Statuses (OR)

```bash
# Validated OR Broadcast
curl -X GET "https://api.grand-shooting.com/v3/picture?picturestatus=50&picturestatus=55" \
  -H "Authorization: Bearer {your_token}"
```

### Get Pictures with Status >= 50 (All Validated States)

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?picturestatus=gte:50" \
  -H "Authorization: Bearer {your_token}"
```

---

## Use Case 3: Filter Pictures by Export

### Get Pictures from a Specific Export

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?export=E-commerce%20HD" \
  -H "Authorization: Bearer {your_token}"
```

### Get Pictures from Multiple Exports

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?export=E-commerce%20HD&export=Social%20Media" \
  -H "Authorization: Bearer {your_token}"
```

---

## Use Case 4: Combine Multiple Filters

### Validated Pictures from a Specific Export Created This Month

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?picturestatus=50&export=E-commerce%20HD&date_cre=gte:2025-01-01" \
  -H "Authorization: Bearer {your_token}"
```

### Selected Pictures for a Specific Reference

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?picturestatus=30&ref=FW25_JACKET_BLACK" \
  -H "Authorization: Bearer {your_token}"
```

### Pictures from a Production Validated Last Week

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?bench_root_id=100&validation_date=gte:2025-01-01&validation_date=lte:2025-01-07" \
  -H "Authorization: Bearer {your_token}"
```

---

## Use Case 5: Iterate Over Large Volumes

The API limits pagination offset to **10,000 records**. For larger datasets, use the **cursor-based pagination pattern** with `picture_id` and `sort_by`.

### The Pattern

1. Sort by `picture_id` ascending
2. Use `picture_id=gt:last_id` to get the next page
3. Repeat until no more results

### Example: First Request

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?picture_id=gte:0&sort_by=picture_id" \
  -H "Authorization: Bearer {your_token}"
```

**Response** (simplified):
```json
[
  {"picture_id": 1, "ref": "REF001", ...},
  {"picture_id": 2, "ref": "REF002", ...},
  ...
  {"picture_id": 100, "ref": "REF100", ...}
]
```

### Example: Second Request (Continue from Last ID)

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?picture_id=gt:100&sort_by=picture_id" \
  -H "Authorization: Bearer {your_token}"
```

### Example: Third Request

```bash
curl -X GET "https://api.grand-shooting.com/v3/picture?picture_id=gt:200&sort_by=picture_id" \
  -H "Authorization: Bearer {your_token}"
```

---

## Complete Python Example: Retrieve All Validated Pictures

```python
import requests

API_BASE = "https://api.grand-shooting.com/v3"
TOKEN = "your_token"
HEADERS = {
    "Authorization": f"Bearer {TOKEN}"
}

def get_pictures_page(filters, last_picture_id=0):
    """Retrieve a page of pictures using cursor-based pagination."""
    params = {
        **filters,
        "picture_id": f"gt:{last_picture_id}",
        "sort_by": "picture_id"
    }
    response = requests.get(
        f"{API_BASE}/picture",
        headers=HEADERS,
        params=params
    )
    return response.json(), response.headers

def get_all_pictures(filters):
    """Retrieve all pictures matching filters, handling large volumes."""
    all_pictures = []
    last_picture_id = 0

    while True:
        pictures, headers = get_pictures_page(filters, last_picture_id)

        if not pictures:
            break

        all_pictures.extend(pictures)
        last_picture_id = pictures[-1]["picture_id"]

        print(f"Retrieved {len(all_pictures)} pictures so far...")

        # Optional: check if we got less than 100, meaning last page
        if len(pictures) < 100:
            break

    return all_pictures

# --- USAGE EXAMPLES ---

# Example 1: Get all validated pictures
print("=== Retrieving all validated pictures ===")
validated_pictures = get_all_pictures({"picturestatus": 50})
print(f"Total validated pictures: {len(validated_pictures)}")

# Example 2: Get all pictures from January 2025
print("\n=== Retrieving pictures from January 2025 ===")
january_pictures = get_all_pictures({
    "date_cre": ["gte:2025-01-01", "lte:2025-01-31"]
})
print(f"Total pictures from January: {len(january_pictures)}")

# Example 3: Get validated pictures from a specific export
print("\n=== Retrieving validated pictures from E-commerce export ===")
export_pictures = get_all_pictures({
    "picturestatus": 50,
    "export": "E-commerce HD"
})
print(f"Total export pictures: {len(export_pictures)}")

# Example 4: Get pictures modified in the last 30 days
from datetime import datetime, timedelta
thirty_days_ago = (datetime.now() - timedelta(days=30)).strftime("%Y-%m-%d")
print(f"\n=== Retrieving pictures modified since {thirty_days_ago} ===")
recent_pictures = get_all_pictures({
    "date_mod": f"gte:{thirty_days_ago}"
})
print(f"Total recently modified pictures: {len(recent_pictures)}")
```

---

## Complete Python Example: Export Pictures with Filters

```python
import requests
import csv
from datetime import datetime

API_BASE = "https://api.grand-shooting.com/v3"
TOKEN = "your_token"
HEADERS = {"Authorization": f"Bearer {TOKEN}"}

def export_pictures_to_csv(filters, output_file):
    """Export filtered pictures to a CSV file."""
    all_pictures = []
    last_picture_id = 0

    print(f"Fetching pictures with filters: {filters}")

    while True:
        params = {
            **filters,
            "picture_id": f"gt:{last_picture_id}",
            "sort_by": "picture_id"
        }
        response = requests.get(f"{API_BASE}/picture", headers=HEADERS, params=params)
        pictures = response.json()

        if not pictures:
            break

        all_pictures.extend(pictures)
        last_picture_id = pictures[-1]["picture_id"]
        print(f"  Fetched {len(all_pictures)} pictures...")

        if len(pictures) < 100:
            break

    if not all_pictures:
        print("No pictures found matching the filters.")
        return

    # Write to CSV
    fieldnames = [
        "picture_id", "ref", "reference_ref", "file_path", "smalltext",
        "picturestatus", "date_cre", "date_mod", "validation_date",
        "bench_id", "bench_root_id", "view_type_code", "export"
    ]

    with open(output_file, 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames, extrasaction='ignore')
        writer.writeheader()
        for picture in all_pictures:
            writer.writerow(picture)

    print(f"\nExported {len(all_pictures)} pictures to {output_file}")

# --- USAGE ---

# Export all validated pictures from Q1 2025
export_pictures_to_csv(
    filters={
        "picturestatus": 50,
        "validation_date": ["gte:2025-01-01", "lte:2025-03-31"]
    },
    output_file="validated_pictures_q1_2025.csv"
)

# Export all pictures from a specific production
export_pictures_to_csv(
    filters={"bench_root_id": 100},
    output_file="production_100_pictures.csv"
)
```

---

## Response Headers

The API returns pagination information in response headers:

| Header | Description |
|--------|-------------|
| `X-Total-Count` | Total number of matching pictures |
| `X-Offset` | Current offset position |
| `X-Count` | Number of pictures in this response |

### Example: Reading Headers in Python

```python
response = requests.get(f"{API_BASE}/picture", headers=HEADERS, params=filters)
pictures = response.json()

total = response.headers.get("X-Total-Count")
offset = response.headers.get("X-Offset")
count = response.headers.get("X-Count")

print(f"Retrieved {count} of {total} pictures (offset: {offset})")
```

---

## Sorting Results

Use the `sort_by` parameter with `+` (ascending) or `-` (descending) prefix:

```bash
# Sort by picture_id ascending (required for large volume iteration)
curl -X GET "https://api.grand-shooting.com/v3/picture?sort_by=picture_id" \
  -H "Authorization: Bearer {your_token}"

# Sort by creation date descending (newest first)
curl -X GET "https://api.grand-shooting.com/v3/picture?sort_by=-date_cre" \
  -H "Authorization: Bearer {your_token}"

# Sort by validation date ascending
curl -X GET "https://api.grand-shooting.com/v3/picture?sort_by=+validation_date" \
  -H "Authorization: Bearer {your_token}"
```

---

## Error Codes

| Code | Description |
|------|-------------|
| 400 | Invalid filter or query parameter |
| 401 | Not authenticated |
| 429 | Rate limit exceeded (5 req/s) |
| 500 | Server error |

---

## Best Practices

1. **Use cursor-based pagination** for large datasets: Always use `picture_id=gt:last_id&sort_by=picture_id` instead of offset-based pagination when retrieving more than 10,000 pictures.

2. **Apply filters early**: The more specific your filters, the faster the response. Filter by `bench_root_id` or `picturestatus` to reduce the result set.

3. **Use date ranges**: When possible, limit queries by date range to improve performance.

4. **Handle rate limits**: Implement exponential backoff if you receive HTTP 429 responses.

5. **Process in batches**: For very large exports, process pictures in batches of 100 (the default page size) rather than loading everything into memory.

---

## Additional Resources

- [Cookbook: Create a production and upload images](./cookbook-creation-production-upload-images.md)
- [Cookbook: Upload images to an existing production](./cookbook-upload-images-existing-production.md)
- [Cookbook: Reference management](./cookbook-reference-management.md)
