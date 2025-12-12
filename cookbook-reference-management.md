# Cookbook: Load, Create and Update References

This guide walks you through managing your product catalog (references) via the Grand Shooting API: creation, updates, and bulk loading.

## Prerequisites

- A valid authentication token
- API base URL: `https://api.grand-shooting.com/v3`

## Authentication

```bash
Authorization: Bearer {your_token}
```

---

## Understanding References

A **reference** represents a product in your catalog. It is uniquely identified by the `ref` field.

### Available Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ref` | string | **Yes** | Unique reference identifier |
| `ean` | string | No | Main barcode (EAN/UPC) |
| `eans` | array[string] | No | Alternative barcodes |
| `eans_extended` | array[object] | No | Barcodes with metadata |
| `univers` | string | No | Category level 1 |
| `gamme` | string | No | Category level 2 |
| `family` | string | No | Category level 3 |
| `sku` | string | No | Internal SKU |
| `brand` | string | No | Brand |
| `smalltext` | string | No | Label/short description |
| `category_id` | integer | No | Category ID |
| `product_ref` | string | No | Product reference (groups variants) |
| `product_smalltext` | string | No | Product label |
| `gender` | string | No | Gender (Woman, Man, Unisex...) |
| `color` | string | No | Color |
| `hexa_color` | string | No | Hexadecimal color code (#RRGGBB) |
| `size` | string | No | Size |
| `collection` | string | No | Collection (FW25, SS25...) |
| `comment` | string | No | Free comment |
| `tags` | array[string] | No | Tags/labels |
| `online` | string | No | Online date |
| `extra` | object | No | Custom fields |
| `shotlist_id` | integer | No | Existing shotlist ID |
| `shotlist` | string | No | Shotlist name (created if non-existent) |

---

## Use Case 1: Create a New Reference

### Endpoint

```
POST /v3/reference
```

### Minimal Example

```bash
curl -X POST "https://api.grand-shooting.com/v3/reference" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "ref": "FW25_JACKET_BLACK_M"
  }'
```

### Complete Example

```bash
curl -X POST "https://api.grand-shooting.com/v3/reference" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "ref": "FW25_JACKET_BLACK_M",
    "ean": "3701234567890",
    "eans": ["3701234567891", "3701234567892"],
    "univers": "Ready-to-Wear",
    "gamme": "Outerwear",
    "family": "Jackets",
    "sku": "JKT-BLK-M-2025",
    "brand": "MyBrand",
    "smalltext": "Black leather jacket size M",
    "product_ref": "FW25_JACKET_BLACK",
    "product_smalltext": "Black leather jacket",
    "gender": "Man",
    "color": "Black",
    "hexa_color": "#000000",
    "size": "M",
    "collection": "FW25",
    "comment": "Expected bestseller",
    "tags": ["leather", "premium", "new"],
    "online": "15/09/2025",
    "extra": {
      "composition": "100% genuine leather",
      "origin": "Italy",
      "retail_price": 599.00
    }
  }'
```

### Response

```json
{
  "reference_id": 12345,
  "ref": "FW25_JACKET_BLACK_M",
  "ean": "3701234567890",
  "eans": ["3701234567891", "3701234567892"],
  "univers": "Ready-to-Wear",
  "gamme": "Outerwear",
  "family": "Jackets",
  "sku": "JKT-BLK-M-2025",
  "brand": "MyBrand",
  "smalltext": "Black leather jacket size M",
  "product_ref": "FW25_JACKET_BLACK",
  "gender": "Man",
  "color": "Black",
  "hexa_color": "#000000",
  "size": "M",
  "collection": "FW25",
  "tags": ["leather", "premium", "new"],
  "extra": {
    "composition": "100% genuine leather",
    "origin": "Italy",
    "retail_price": 599.00
  },
  "create_date": "2025-03-15T10:00:00Z",
  "update_date": "2025-03-15T10:00:00Z"
}
```

---

## Use Case 2: Update an Existing Reference

### Behavior

The `POST /v3/reference` endpoint performs an **upsert**:
- If a reference with the same `ref` exists → **update**
- Otherwise → **create**

### Example: Update

```bash
curl -X POST "https://api.grand-shooting.com/v3/reference" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "ref": "FW25_JACKET_BLACK_M",
    "smalltext": "Black leather jacket size M - LIMITED EDITION",
    "tags": ["leather", "premium", "new", "limited-edition"],
    "extra": {
      "composition": "100% genuine leather",
      "origin": "Italy",
      "retail_price": 699.00,
      "edition": "limited"
    }
  }'
```

**Note**: Only the provided fields are updated. Other fields retain their values.

---

## Use Case 3: Bulk Load References

### Endpoint

```
POST /v3/reference/bulk
```

### Example

```bash
curl -X POST "https://api.grand-shooting.com/v3/reference/bulk" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '[
    {
      "ref": "FW25_JACKET_BLACK_S",
      "product_ref": "FW25_JACKET_BLACK",
      "smalltext": "Black leather jacket S",
      "color": "Black",
      "size": "S",
      "collection": "FW25",
      "brand": "MyBrand"
    },
    {
      "ref": "FW25_JACKET_BLACK_M",
      "product_ref": "FW25_JACKET_BLACK",
      "smalltext": "Black leather jacket M",
      "color": "Black",
      "size": "M",
      "collection": "FW25",
      "brand": "MyBrand"
    },
    {
      "ref": "FW25_JACKET_BLACK_L",
      "product_ref": "FW25_JACKET_BLACK",
      "smalltext": "Black leather jacket L",
      "color": "Black",
      "size": "L",
      "collection": "FW25",
      "brand": "MyBrand"
    },
    {
      "ref": "FW25_JACKET_BROWN_M",
      "product_ref": "FW25_JACKET_BROWN",
      "smalltext": "Brown leather jacket M",
      "color": "Brown",
      "hexa_color": "#8B4513",
      "size": "M",
      "collection": "FW25",
      "brand": "MyBrand"
    }
  ]'
```

### Response

```json
[
  {
    "reference_id": 12346,
    "ref": "FW25_JACKET_BLACK_S",
    "product_ref": "FW25_JACKET_BLACK",
    "smalltext": "Black leather jacket S",
    "size": "S"
  },
  {
    "reference_id": 12345,
    "ref": "FW25_JACKET_BLACK_M",
    "product_ref": "FW25_JACKET_BLACK",
    "smalltext": "Black leather jacket M",
    "size": "M"
  },
  {
    "reference_id": 12347,
    "ref": "FW25_JACKET_BLACK_L",
    "product_ref": "FW25_JACKET_BLACK",
    "smalltext": "Black leather jacket L",
    "size": "L"
  },
  {
    "reference_id": 12348,
    "ref": "FW25_JACKET_BROWN_M",
    "product_ref": "FW25_JACKET_BROWN",
    "smalltext": "Brown leather jacket M",
    "size": "M"
  }
]
```

---

## Use Case 4: Query References

### List All References

```bash
curl -X GET "https://api.grand-shooting.com/v3/reference" \
  -H "Authorization: Bearer {your_token}"
```

### Pagination

- **100 results per page** by default
- Use the `offset` header to paginate

```bash
# Page 1 (0-99)
curl -X GET "https://api.grand-shooting.com/v3/reference" \
  -H "Authorization: Bearer {your_token}" \
  -H "offset: 0"

# Page 2 (100-199)
curl -X GET "https://api.grand-shooting.com/v3/reference" \
  -H "Authorization: Bearer {your_token}" \
  -H "offset: 100"
```

**Response headers**:
- `X-Total-Count`: Total number of references
- `X-Offset`: Current offset
- `X-Count`: Number of results returned

### Filter References

#### By Simple Field

```bash
# By exact reference
curl -X GET "https://api.grand-shooting.com/v3/reference?ref=FW25_JACKET_BLACK_M" \
  -H "Authorization: Bearer {your_token}"

# By collection
curl -X GET "https://api.grand-shooting.com/v3/reference?collection=FW25" \
  -H "Authorization: Bearer {your_token}"

# By brand
curl -X GET "https://api.grand-shooting.com/v3/reference?brand=MyBrand" \
  -H "Authorization: Bearer {your_token}"

# By color
curl -X GET "https://api.grand-shooting.com/v3/reference?color=Black" \
  -H "Authorization: Bearer {your_token}"
```

#### Multiple Values (OR)

```bash
# Multiple colors
curl -X GET "https://api.grand-shooting.com/v3/reference?color=Black&color=Brown" \
  -H "Authorization: Bearer {your_token}"

# Multiple references
curl -X GET "https://api.grand-shooting.com/v3/reference?ref=FW25_JACKET_BLACK_M&ref=FW25_JACKET_BLACK_L" \
  -H "Authorization: Bearer {your_token}"
```

#### Comparisons

```bash
# ID greater than 1000
curl -X GET "https://api.grand-shooting.com/v3/reference?reference_id=gt:1000" \
  -H "Authorization: Bearer {your_token}"

# ID between 1000 and 2000
curl -X GET "https://api.grand-shooting.com/v3/reference?reference_id=gte:1000&reference_id=lte:2000" \
  -H "Authorization: Bearer {your_token}"
```

Available operators:
- `gt:`: greater than (>)
- `gte:`: greater than or equal (>=)
- `lt:`: less than (<)
- `lte:`: less than or equal (<=)

#### Sorting

```bash
# Ascending sort by reference_id
curl -X GET "https://api.grand-shooting.com/v3/reference?sort_by=+reference_id" \
  -H "Authorization: Bearer {your_token}"

# Descending sort by reference_id
curl -X GET "https://api.grand-shooting.com/v3/reference?sort_by=-reference_id" \
  -H "Authorization: Bearer {your_token}"
```

### Get a Reference by ID

```bash
curl -X GET "https://api.grand-shooting.com/v3/reference/12345" \
  -H "Authorization: Bearer {your_token}"
```

---

## Use Case 5: Associate References with a Production

### Add References to a Production

```bash
curl -X POST "https://api.grand-shooting.com/v3/production/100/reference" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '[
    {"ref": "FW25_JACKET_BLACK_M"},
    {"ref": "FW25_JACKET_BLACK_L"},
    {"ref": "FW25_JACKET_BROWN_M"}
  ]'
```

### List References for a Production

```bash
curl -X GET "https://api.grand-shooting.com/v3/production/100/reference" \
  -H "Authorization: Bearer {your_token}"
```

---

## Use Case 6: Using Shotlists

Shotlists allow you to group references to organize shootings.

### Create a Reference with a New Shotlist

```bash
curl -X POST "https://api.grand-shooting.com/v3/reference" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "ref": "FW25_DRESS_RED_M",
    "smalltext": "Red dress M",
    "shotlist": "March 2025 Shooting"
  }'
```

If the shotlist "March 2025 Shooting" doesn't exist, it will be created.

### Add to an Existing Shotlist by ID

```bash
curl -X POST "https://api.grand-shooting.com/v3/reference" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "ref": "FW25_DRESS_BLUE_M",
    "smalltext": "Blue dress M",
    "shotlist_id": 42
  }'
```

---

## Use Case 7: Extended Barcodes (EAN Extended)

For rich barcode information:

```bash
curl -X POST "https://api.grand-shooting.com/v3/reference" \
  -H "Authorization: Bearer {your_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "ref": "FW25_JACKET_BLACK",
    "smalltext": "Black leather jacket",
    "ean": "3701234567890",
    "eans_extended": [
      {
        "ean": "3701234567890",
        "smalltext": "Black leather jacket S",
        "star": true,
        "extra": {
          "sku": "JKT-BLK-S",
          "size": "S"
        }
      },
      {
        "ean": "3701234567891",
        "smalltext": "Black leather jacket M",
        "star": false,
        "extra": {
          "sku": "JKT-BLK-M",
          "size": "M"
        }
      },
      {
        "ean": "3701234567892",
        "smalltext": "Black leather jacket L",
        "star": false,
        "extra": {
          "sku": "JKT-BLK-L",
          "size": "L"
        }
      }
    ]
  }'
```

The `star: true` field indicates the primary EAN.

---

## Complete Python Example

```python
import requests
import json

API_BASE = "https://api.grand-shooting.com/v3"
TOKEN = "your_token"
HEADERS = {
    "Authorization": f"Bearer {TOKEN}",
    "Content-Type": "application/json"
}

def create_reference(ref_data):
    """Create or update a reference."""
    response = requests.post(
        f"{API_BASE}/reference",
        headers=HEADERS,
        json=ref_data
    )
    return response.json()

def create_references_bulk(references):
    """Create or update multiple references."""
    response = requests.post(
        f"{API_BASE}/reference/bulk",
        headers=HEADERS,
        json=references
    )
    return response.json()

def get_references(filters=None):
    """Retrieve references with optional filters."""
    response = requests.get(
        f"{API_BASE}/reference",
        headers=HEADERS,
        params=filters
    )
    return {
        "data": response.json(),
        "total": response.headers.get("X-Total-Count"),
        "offset": response.headers.get("X-Offset"),
        "count": response.headers.get("X-Count")
    }

def get_reference_by_id(reference_id):
    """Retrieve a reference by its ID."""
    response = requests.get(
        f"{API_BASE}/reference/{reference_id}",
        headers=HEADERS
    )
    return response.json()

def add_references_to_production(bench_root_id, refs):
    """Add references to a production."""
    payload = [{"ref": ref} for ref in refs]
    response = requests.post(
        f"{API_BASE}/production/{bench_root_id}/reference",
        headers=HEADERS,
        json=payload
    )
    return response.json()

# --- USAGE EXAMPLE ---

# 1. Create a simple reference
print("=== Creating a reference ===")
new_ref = create_reference({
    "ref": "SS25_TSHIRT_WHITE_M",
    "smalltext": "White T-shirt M",
    "brand": "MyBrand",
    "collection": "SS25",
    "color": "White",
    "size": "M",
    "gender": "Unisex",
    "tags": ["basic", "cotton"]
})
print(f"Reference created: {new_ref['ref']} (ID: {new_ref['reference_id']})")

# 2. Bulk import from simulated CSV file
print("\n=== Bulk import ===")
catalog_data = [
    {
        "ref": "SS25_TSHIRT_WHITE_S",
        "smalltext": "White T-shirt S",
        "brand": "MyBrand",
        "collection": "SS25",
        "color": "White",
        "size": "S",
        "product_ref": "SS25_TSHIRT_WHITE"
    },
    {
        "ref": "SS25_TSHIRT_WHITE_L",
        "smalltext": "White T-shirt L",
        "brand": "MyBrand",
        "collection": "SS25",
        "color": "White",
        "size": "L",
        "product_ref": "SS25_TSHIRT_WHITE"
    },
    {
        "ref": "SS25_TSHIRT_BLACK_M",
        "smalltext": "Black T-shirt M",
        "brand": "MyBrand",
        "collection": "SS25",
        "color": "Black",
        "size": "M",
        "product_ref": "SS25_TSHIRT_BLACK"
    }
]

bulk_result = create_references_bulk(catalog_data)
print(f"References imported: {len(bulk_result)}")
for ref in bulk_result:
    print(f"  - {ref['ref']} (ID: {ref['reference_id']})")

# 3. Search for references
print("\n=== Searching references ===")
results = get_references({"collection": "SS25", "color": "White"})
print(f"Total found: {results['total']}")
for ref in results["data"]:
    print(f"  - {ref['ref']}: {ref['smalltext']}")

# 4. Update a reference
print("\n=== Updating a reference ===")
updated_ref = create_reference({
    "ref": "SS25_TSHIRT_WHITE_M",
    "tags": ["basic", "cotton", "bestseller"],
    "extra": {
        "composition": "100% organic cotton",
        "care": "Machine wash 30°"
    }
})
print(f"Reference updated: {updated_ref['ref']}")
print(f"  Tags: {updated_ref['tags']}")

# 5. Associate references with a production
print("\n=== Associating with a production ===")
bench_root_id = 100  # Your production ID
refs_to_add = ["SS25_TSHIRT_WHITE_S", "SS25_TSHIRT_WHITE_M", "SS25_TSHIRT_WHITE_L"]
add_references_to_production(bench_root_id, refs_to_add)
print(f"References added to production {bench_root_id}")
```

---

## Import from CSV

### Python Script for CSV Import

```python
import csv
import requests

API_BASE = "https://api.grand-shooting.com/v3"
TOKEN = "your_token"
HEADERS = {
    "Authorization": f"Bearer {TOKEN}",
    "Content-Type": "application/json"
}

def import_from_csv(csv_file_path, batch_size=100):
    """Import references from a CSV file."""
    references = []

    with open(csv_file_path, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        for row in reader:
            # Clean empty fields
            ref_data = {k: v for k, v in row.items() if v}

            # Convert tags (format: "tag1,tag2,tag3")
            if 'tags' in ref_data:
                ref_data['tags'] = [t.strip() for t in ref_data['tags'].split(',')]

            references.append(ref_data)

    # Import in batches
    total_imported = 0
    for i in range(0, len(references), batch_size):
        batch = references[i:i + batch_size]
        response = requests.post(
            f"{API_BASE}/reference/bulk",
            headers=HEADERS,
            json=batch
        )
        if response.status_code == 200:
            total_imported += len(batch)
            print(f"Batch {i // batch_size + 1}: {len(batch)} references imported")
        else:
            print(f"Error batch {i // batch_size + 1}: {response.text}")

    return total_imported

# Expected CSV file example:
# ref,smalltext,brand,collection,color,size,ean,tags
# FW25_JACKET_BLACK_S,Black jacket S,MyBrand,FW25,Black,S,3701234567890,"leather,premium"
# FW25_JACKET_BLACK_M,Black jacket M,MyBrand,FW25,Black,M,3701234567891,"leather,premium"

total = import_from_csv("catalog.csv")
print(f"\nTotal imported: {total} references")
```

---

## Error Codes

| Code | Description |
|------|-------------|
| 400 | Invalid request (`ref` field missing or incorrect format) |
| 401 | Not authenticated |
| 404 | Reference not found (GET by ID) |
| 409 | Conflict (EAN already used by another reference) |
| 429 | Rate limit exceeded (5 req/s) |
| 500 | Server error |

---

## Best Practices

1. **Use explicit `ref` values**: e.g., `{COLLECTION}_{TYPE}_{COLOR}_{SIZE}`
2. **Group with `product_ref`**: to link variants (sizes/colors) of the same product
3. **Use bulk import** for large volumes (more efficient)
4. **Validate EANs**: they must be unique across the entire catalog
5. **Leverage the `extra` field**: to store custom business data
6. **Organize with shotlists**: to plan your shootings

---

## Additional Resources

- [Cookbook: Create a production and upload images](./cookbook-creation-production-upload-images.md)
- [Cookbook: Upload images to an existing production](./cookbook-upload-images-existing-production.md)
