# Grand Shooting API Cookbooks

Welcome to the Grand Shooting API cookbooks. These cookbooks provide practical, step-by-step guides to help you integrate with the Grand Shooting platform quickly and efficiently.
For more details, the reference API documentation can be found here : https://api.grand-shooting.com/

## What are Cookbooks?

Cookbooks are task-oriented guides that focus on **how to accomplish specific goals** with the API. Rather than exhaustive reference documentation, they provide:

- **Real-world use cases** with complete, working examples
- **Copy-paste curl commands** you can run immediately
- **Python code samples** for common integration patterns
- **Best practices** learned from production implementations
- **Error handling** guidance for robust integrations

## Getting Started

### Authentication

All API requests require authentication. Include one of these headers with every request:

```bash
# Bearer Token
Authorization: Bearer {your_token}

# OAuth Access Token
Authorization: access_token {your_oauth_token}

```

### Base URL

```
https://api.grand-shooting.com/v3
```

### Rate Limiting

The API enforces a rate limit of **5 requests per second** per account. Exceeding this limit will result in HTTP 429 responses.

## Available Cookbooks

### 1. [Create a Production and Upload Images](./cookbook-creation-production-upload-images.md)

Learn how to create a new production (photo shoot session) and upload images to it.

**Topics covered:**
- Creating a production with required and optional fields
- Understanding benches and workflow steps
- Uploading images via direct file upload
- Bulk uploading images via URL
- Assigning references and view types during upload

**Best for:** Starting a new photo shoot workflow from scratch.

---

### 2. [Upload Images to an Existing Production](./cookbook-upload-images-existing-production.md)

Learn how to add images to a production that already exists.

**Topics covered:**
- Finding and identifying existing productions
- Selecting the correct bench for uploads
- Organizing uploads with folders and subpaths
- Handling upload responses and errors
- Linking images to parent images (for retouches)

**Best for:** Adding new images to ongoing productions or batch imports.

---

### 3. [Reference Management](./cookbook-reference-management.md)

Learn how to manage your product catalog (references) through the API.

**Topics covered:**
- Creating and updating references (upsert behavior)
- Bulk importing references from CSV or external systems
- Querying and filtering the reference catalog
- Using shotlists to organize products
- Managing barcodes (EAN/UPC) and extended metadata
- Associating references with productions

**Best for:** Synchronizing your product catalog with Grand Shooting.

---

## Quick Reference

### Common Endpoints

| Action | Method | Endpoint |
|--------|--------|----------|
| List productions | GET | `/v3/production` |
| Create production | POST | `/v3/production` |
| List benches | GET | `/v3/production/{id}/bench` |
| Upload image (file) | POST | `/v3/production/{id}/bench/{bench_id}/upload` |
| Upload images (URL) | POST | `/v3/production/{id}/bench/{bench_id}/upload/url` |
| List images | GET | `/v3/picture?bench_id={bench_id}` |
| List references | GET | `/v3/reference` |
| Create/update reference | POST | `/v3/reference` |
| Bulk create references | POST | `/v3/reference/bulk` |

### Key Concepts

| Term | Description |
|------|-------------|
| **Production** | A photo shoot session with a name, date, and workflow |
| **Bench** | A workflow step within a production (Live, Phase 1, Phase 2, Export) |
| **Reference** | A product in your catalog, identified by a unique `ref` |
| **Shotlist** | A group of references to be photographed together |

### Production Status Values

| benchstatus | Meaning |
|-------------|---------|
| 10 | Active |
| 20 | In progress |
| 30 | Finished |
| 35 | Archiving |
| 40 | Archived |
| 50 | Flushed |

### Workflow Step Types

| benchsteptype | Label | Accepts uploads? |
|---------------|-------|------------------|
| 10 | Live (Capture) | Yes |
| 20 | Post-production Phase 1 | Yes |
| 30 | Post-production Phase 2 | Yes |
| 40 | Export/Validation | No |

## Important Notes

### Understanding the Production Response Structure

When you create a production, the API returns an **array of benches** (workflow steps). The key IDs to remember:

- `root_id`: The production identifier (same as the `bench_id` of the first bench)
- `bench_id`: The identifier for each workflow step

**Example**: If the response shows `bench_id: 100` with `benchsteptype: 10` (Live), you use:
- `bench_root_id = 100` (in the URL path)
- `bench_id = 100` (for uploading to the Live bench)

### List Productions Response Format

The `GET /v3/production` endpoint returns a **nested array** (array of arrays), where each inner array represents one production with all its benches:

```json
[
  [
    { "bench_id": 100, "root_id": 100, "benchsteptype": 10, ... },
    { "bench_id": 101, "root_id": 100, "benchsteptype": 20, ... }
  ],
  [
    { "bench_id": 200, "root_id": 200, "benchsteptype": 10, ... }
  ]
]
```

### Finding the Right shooting_method_id

The `shooting_method_id` is required when creating a production. Common values:
- `1`: Standard packshot
- Contact your Grand Shooting administrator for the list of available shooting methods in your account.

---

## Tests

Automated tests are available in the `tests/` directory to validate all cookbook use cases. See [tests/README.md](./tests/README.md) for details.

```bash
cd tests
npm run test:all
```

---

## Need Help?

If you encounter issues or have questions about the API:

1. Check the error codes section in each cookbook
2. Verify your authentication token is valid
3. Ensure you're not exceeding rate limits
4. Contact Grand Shooting support for additional assistance
