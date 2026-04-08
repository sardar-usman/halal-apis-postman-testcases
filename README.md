# Echo Halal API - Postman Test Collection

A comprehensive Postman test suite for the **Echo Halal Certification API** with **202 test cases** (23 positive, 179 negative), covering end-to-end testing of halal food certification workflows including authorization, document signing, file uploads, and certificate creation.

## API Overview

The Echo Halal API is built on the Smart Cert platform and provides endpoints for:

- **Authorization** - OAuth-based token generation
- **Document Signing** - Digital signing of certification documents (PDF)
- **File Upload** - Upload signed documents with signature verification
- **Certificate Creation** - Create and manage halal certification records

### Certificate Types Covered

| Certificate Type | Description |
| --- | --- |
| Halal Certificate | Primary halal food certification |
| Health Certificate | Health compliance documentation |
| Certificate of Origin | Product origin verification |
| Commercial Invoice | Trade invoice certification |
| Additional Documents | Supporting documentation |

## Test Structure

### Positive Test Scenarios (23 tests)

- Authorization with valid credentials
- Document signing for all 5 certificate types
- File upload with valid signatures
- Certificate creation (8 payload variants + India Authority scenarios)

### Negative Test Scenarios (179 tests)

- Invalid authentication (wrong ID, wrong secret)
- Invalid file types (PNG, DOC, XLS instead of PDF)
- Invalid file uploads (wrong signatures, missing files)
- Field validation for all certificate types:
  - Empty/missing required fields
  - Invalid email formats
  - Invalid country codes
  - Invalid date formats
  - Negative numeric values
  - Boundary value testing

## Setup

### Prerequisites

- [Postman](https://www.postman.com/downloads/) (v10+)
- API credentials (client ID and secret)

### Import & Configure

1. Import the collection file into Postman:

   ```text
   Echo Halal API's Testing.postman_collection.json
   ```

2. Copy `.env.example` to `.env` and fill in your actual credentials:

   ```bash
   cp .env.example .env
   ```

3. Create an environment in Postman with the following required variables:

   | Variable | Description | Example |
   | --- | --- | --- |
   | `baseUrl` | API base URL (no trailing slash) | `https://your-api-host.com` |
   | `id` | Client ID for authorization | `your-client-id` |
   | `Secret` | API secret for authorization | `your-api-secret` |
   | `x-userid` | System user ID (header value) | `your-user-id` |

   > See `.env.example` for the full list of variables, including auto-generated ones.

4. The `sample-files/` directory already contains all required test files (see below).

### Running Tests

1. **Run in order** - Execute the "Positive Test Scenarios" folder first (authorization generates a token and variables used by subsequent requests)
2. **Collection Runner** - Use Postman's Collection Runner to execute all tests in sequence
3. **Newman CLI** (optional):

   ```bash
   npm install -g newman
   newman run "Echo Halal API's Testing.postman_collection.json" \
     --environment your-environment.json
   ```

> **Note:** Negative tests depend on variables set by the positive test suite (e.g., date variables, signatures). Always run positive tests first.

## Collection Variables

The collection uses variables that are automatically populated during test execution:

| Variable | Set By |
| --- | --- |
| `accessToken` | Authorization request |
| `halalSignature` | Sign Document - Halal Cert |
| `healthSignature` | Sign Document - Health Cert |
| `originCertSignature` | Sign Document - Origin Cert |
| `commercialInvoiceSignature` | Sign Document - Commercial Invoice |
| `additionalFileSignature` | Sign Document - Additional Document |
| `halalCertFileId` | Upload - Halal Cert |
| `healthCertFileId` | Upload - Health Cert |
| `originCertFileId` | Upload - Origin Cert |
| `commercialInvoiceFileId` | Upload - Commercial Invoice |
| `addFileId` | Upload - Additional Document |

## Testing Approach

This collection demonstrates several key API testing techniques:

### Chained Request Testing

Tests are designed to run sequentially where each step feeds into the next, simulating a real-world workflow:

```text
Authorization → Document Signing → File Upload → Certificate Creation
```

- The auth token from step 1 is stored in `accessToken` and reused by all subsequent requests
- Signatures from document signing are passed to the upload step for verification
- File IDs from uploads are injected into certificate creation payloads

### Dynamic Test Data Generation

Pre-request scripts use `moment.js` to generate realistic date values at runtime, ensuring tests never fail due to stale/expired dates:

- `certificateDate` — today's date
- `slaughterDate` — 10 days ago
- `shippingDate` — 5 days ago
- `certificateExpiryDate` — 1 year from today
- Certificate numbers are dynamically generated with random 9-digit padding

### Negative Testing & Input Validation

179 negative tests systematically verify that the API properly rejects invalid inputs across all endpoints:

| Validation Type | What's Tested | Expected Behavior |
| --- | --- | --- |
| File type validation | PNG, DOC, XLS uploads instead of PDF | `file.invalid.type` error |
| Email format | Missing `@`, missing domain, invalid format | `VE-001` validation error |
| Country code | Missing `+` prefix, letters, empty values | `VE-001` validation error |
| Date format | `DD-MM-YYYY` instead of `YYYY-MM-DD`, future slaughter dates | `VE-001` validation error |
| Required fields | Empty strings for mandatory fields | `VE-001` validation error |
| Numeric boundaries | Negative quantities, negative weights | `VE-001` validation error |
| Auth validation | Wrong client ID, wrong secret | `401 Unauthorized` |

### Boundary Value Analysis

Tests target edge cases at input boundaries:

- Zero-length strings for required text fields
- Negative numbers for quantity and weight fields
- Dates in wrong formats and invalid ranges (e.g., future slaughter dates)
- File type boundaries (valid PDF vs. invalid PNG/DOC/XLS)

### Response Schema Validation

Every test script validates the response structure using `pm.expect` assertions:

- **Positive tests** — verify status code `200`, presence of expected properties (`signature`, `referenceId`, `fileName`, `fileId`)
- **Negative tests** — verify appropriate error codes (`codeMesg`), error messages, and HTTP status codes

## Sample Test Files

The `sample-files/` directory includes all files referenced by the collection:

| File | Type | Purpose |
| --- | --- | --- |
| `halal-cert-sample.pdf` | PDF | Halal certificate signing & upload |
| `health-cert-sample.pdf` | PDF | Health certificate signing & upload |
| `origin-cert-sample.pdf` | PDF | Certificate of origin signing & upload |
| `commercial-invoice-sample.pdf` | PDF | Commercial invoice signing & upload |
| `additional-doc-sample.pdf` | PDF | Additional document signing & upload |
| `test-valid-file.pdf` | PDF | Valid file for negative test scenarios |
| `test-invalid-file.png` | PNG | Invalid file type - should be rejected |
| `test-invalid-file.doc` | DOC | Invalid file type - should be rejected |
| `test-invalid-file.xls` | XLS | Invalid file type - should be rejected |

The negative tests verify that the API correctly rejects non-PDF file formats (PNG, DOC, XLS) and returns appropriate error responses.

## Tech Stack

- **Postman** - API testing platform
- **JavaScript** - Test scripts (pm.test assertions)
- **Newman** - CLI runner (optional)

## Author

**Osman** - QA Engineer

---

> **Note:** All credentials, URLs, and sensitive data in this collection have been replaced with placeholders. You must configure your own environment before running the tests.
