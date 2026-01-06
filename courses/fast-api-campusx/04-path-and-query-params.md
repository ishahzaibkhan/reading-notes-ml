# FastAPI: Path Parameters and Query Parameters

## Path Parameters

**Definition**: Dynamic segments of a URL path used to identify specific resources. They are integral parts of the URL structure.

**Purpose**: 
- Allow requesting specific data instead of all data
- Example: `localhost:8000/view/3` fetches the third patient instead of all patients
- Dynamic part changes based on client's request

**Use Cases**:
- Retrieval: Fetching specific items (e.g., user profile)
- Update: Modifying single resources
- Delete: Removing particular resources
- **Nature**: Required for endpoint to function

**Implementation Example**:
```python
@app.get("/patient/{patient_id}")
def view_patient(patient_id: str):
    # Load data and check if patient_id exists
    # Return patient data if found, else error
```

### Path Function for Enhanced Documentation

**Purpose**: Provides metadata, validation rules, and documentation hints for path parameters.

**Capabilities**:
- Metadata: Add title, description, examples
- Validation: Apply constraints (gt, ge, lt, le, min_length, max_length, regex)

```python
from fastapi import Path

def view_patient(
    patient_id: str = Path(..., description="ID of the patient in the DB", example="P001")
):
    # ... indicates required parameter
```

## HTTP Status Codes

**Definition**: Three-digit numbers indicating the result of a client's request.

**Major Categories**:
- **2xx (Success)**: Request successfully received and accepted
- **3xx (Redirection)**: Further action needed
- **4xx (Client Error)**: Request contains error or client not authorized
- **5xx (Server Error)**: Server failed to fulfill valid request

**Common Status Codes**:
- `200 OK`: Request succeeded
- `201 Created`: New resource created (POST requests)
- `204 No Content`: Successfully processed, no content returned (DELETE)
- `400 Bad Request`: Client error (missing fields, wrong data type)
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: No access rights
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Unexpected server condition
- `502 Bad Gateway`: Invalid response from upstream server
- `503 Service Unavailable`: Server down or overloaded

### HTTPException for Proper Error Handling

**Problem**: Returning error messages with incorrect status codes (e.g., 200 OK for not found)

**Solution**: Use FastAPI's HTTPException

```python
from fastapi import HTTPException

if patient_id in data:
    return data[patient_id]
else:
    raise HTTPException(status_code=404, detail="Patient Not Found")
```

## Query Parameters

**Definition**: Optional key-value pairs appended to URL end, used to pass additional data.

**URL Structure**: `?key1=value1&key2=value2`
- Question mark (?) after main path
- Ampersands (&) separate multiple parameters

**Characteristics**:
- **Optional**: Endpoint functions without them (can use defaults)
- **Use Cases**: Filtering, sorting, searching, pagination

**Implementation Example**:
```python
from fastapi import Query, HTTPException

@app.get("/sort")
def sort_patients(
    sort_by: str = Query(..., description="Sort on basis of Height, Weight or BMI"),
    order: str = Query("ascending", description="Sort in ascending or descending order")
):
    # Validate sort_by field
    if sort_by not in ["Height", "Weight", "BMI"]:
        raise HTTPException(status_code=400, detail="Valid fields: Height, Weight, BMI")
    
    # Validate order
    if order not in ["ascending", "descending"]:
        raise HTTPException(status_code=400, detail="Valid options: ascending, descending")
    
    # Load data and sort
    sort_order = True if order == "descending" else False
    sorted_data = sorted(data.values(), key=lambda x: x[sort_by], reverse=sort_order)
    return sorted_data
```

**Example URLs**:
- `http://127.0.0.1:8000/sort?sort_by=Height&order=descending`
- `http://127.0.0.1:8000/sort?sort_by=BMI` (uses default order)

## Path vs Query Parameters Summary

| Aspect | Path Parameters | Query Parameters |
|--------|----------------|------------------|
| Location | Dynamic part of URL | After `?` in URL |
| Purpose | Fetch specific resource | Add features (filter, sort, paginate) |
| Required | Yes | No (optional) |
| Use Case | CRUD on single item | Search, filter, sort operations |

**Note**: Both can be combined in a single endpoint.