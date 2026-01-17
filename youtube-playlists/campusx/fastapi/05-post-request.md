# FastAPI: POST Request - Creating New Resources

## POST Request Fundamentals

**Definition**: HTTP method used to send data to the server to create new resources.

**Purpose**: 
- Create new resources on the server
- Send structured data via request body
- Different from GET which only retrieves data

**Request Body**:
- Data sent in JSON format containing resource information
- **Definition**: "A request body is the portion of an HTTP request that contains data sent by the client to the server, typically used in POST or PUT methods to transmit structured data like JSON or XML for creating or updating resources."

**Use Cases**:
- Creating new records (patients, users, products)
- Submitting forms
- Uploading structured data
- **Nature**: Requires data validation before processing

## Pydantic Model for Validation

**Purpose**: Validates incoming request body data automatically with type safety and constraints.

**Installation**: `pip install pydantic`

**Implementation Example**:
```python
from pydantic import BaseModel, Field, computed_field
from typing import Annotated, Literal

class Patient(BaseModel):
    id: Annotated[str, Field(..., description="ID of the patient", examples=["P01"])]
    
    name: Annotated[str, Field(..., description="Name of the patient")]
    
    city: Annotated[str, Field(..., description="City where the patient is living")]
    
    age: Annotated[int, Field(..., gt=0, lt=120, description="Age of the patient")]
    
    gender: Annotated[
        Literal["Male", "Female", "Others"], 
        Field(..., description="Gender of the patient")
    ]
    
    height: Annotated[float, Field(..., gt=0, description="Height of the patient in meters")]
    
    weight: Annotated[float, Field(..., gt=0, description="Weight of the patient in kgs")]
```

**Validation Constraints**:
- `gt` (greater than), `lt` (less than): Numeric range validation
- `Literal`: Restricts to specific allowed values
- `...`: Indicates required parameter
- `description`: Appears in API documentation
- `examples`: Provides sample values in Swagger UI

### Computed Fields

**Definition**: Fields calculated dynamically by the server, not provided by the client.

**Purpose**: Automatically derive values based on other fields.

```python
@computed_field
@property
def bmi(self) -> float:
    return round(self.weight / (self.height ** 2), 2)

@computed_field
@property
def verdict(self) -> str:
    if self.bmi < 18.5:
        return "Under Weight"
    elif self.bmi < 25:
        return "Normal"
    elif self.bmi < 30:
        return "Over Weight"
    else:
        return "Obese"
```

**Behavior**: Pydantic automatically handles dependencies - when `verdict` is accessed, `bmi` is calculated first.

## Creating the POST Endpoint

**Endpoint Definition**:
```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import JSONResponse

@app.post("/create")
async def create_patient(patient: Patient):
    # 1. Load existing data
    data = load_data()
    
    # 2. Check if patient already exists
    if patient.id in data:
        raise HTTPException(status_code=400, detail="Patient already exists")
    
    # 3. Convert Pydantic object to dictionary (exclude id as it's the key)
    new_patient_data = patient.model_dump(exclude={"id"})
    
    # 4. Add new patient to data dictionary
    data[patient.id] = new_patient_data
    
    # 5. Save updated data to JSON file
    save_data(data)
    
    # 6. Return success response
    return JSONResponse(
        status_code=201, 
        content={"message": "Patient created successfully"}
    )
```

**Key Methods**:
- `patient.model_dump(exclude={"id"})`: Converts Pydantic object to Python dictionary, excluding specified fields
- Computed fields (BMI, verdict) are automatically included in the dictionary

**Utility Function**:
```python
import json

def save_data(data):
    with open("patients.json", "w") as f:
        json.dump(data, f)
```

## HTTP Status Codes

**Definition**: Three-digit numbers indicating the result of a client's request.

**Common Status Codes for POST**:
- `201 Created`: New resource successfully created (preferred for POST)
- `400 Bad Request`: Invalid data, validation failures, or duplicate resources
- `500 Internal Server Error`: Server failed to process valid request

### HTTPException for Error Handling

**Purpose**: Return appropriate HTTP status codes with error messages.

```python
from fastapi import HTTPException

if patient.id in data:
    raise HTTPException(status_code=400, detail="Patient already exists")
```

## Testing Examples

### Test Case 1: Duplicate Patient
**Input**: Existing patient ID (e.g., P01)

**Result**: 
```json
Status: 400 Bad Request
{"detail": "Patient already exists"}
```

### Test Case 2: Successful Creation
**Input**:
```json
{
  "id": "P006",
  "name": "Rahul Gupta",
  "city": "Hyderabad",
  "age": 19,
  "gender": "Male",
  "height": 1.74,
  "weight": 55
}
```

**Result**: 
```json
Status: 201 Created
{"message": "Patient created successfully"}
```

**Verification**: New patient appears with BMI (20.91) and Verdict (Normal) automatically calculated.

## POST vs GET Summary

| Aspect | POST Request | GET Request |
|--------|-------------|-------------|
| Purpose | Create new resources | Retrieve existing resources |
| Request Body | Required (contains data) | Not used |
| Status Code | 201 Created | 200 OK |
| Data Flow | Client to Server | Server to Client |

**Note**: FastAPI automatically validates request body using Pydantic models, raising errors for invalid data before endpoint logic executes.