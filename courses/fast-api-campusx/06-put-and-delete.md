# FastAPI PUT and DELETE Operations - Patient Management System

## Overview
This guide covers implementing UPDATE (PUT) and DELETE operations to complete a CRUD API for a patient management system in FastAPI.

## Update Endpoint Implementation

### HTTP Method & Design
- **Method**: PUT
- **Endpoint**: `/edit/{patient_id}`
- **Inputs**:
  - Patient ID (path parameter) - identifies which patient to update
  - Request body (JSON) - contains fields to be updated

### Challenge: Partial Updates
The main challenge is allowing clients to update only specific fields (e.g., just city and weight) rather than requiring all fields. The existing `Patient` model requires all fields, making it unsuitable for partial updates.

### Solution: PatientUpdate Model
Create a new Pydantic model where all fields are optional:
```python
from typing import Optional
from pydantic import BaseModel, Field

class PatientUpdate(BaseModel):
    name: Optional[str] = Field(None)
    city: Optional[str] = Field(None)
    age: Optional[int] = Field(None, ge=0)
    gender: Optional[str] = Field(None)
    height: Optional[float] = Field(None, gt=0)
    weight: Optional[float] = Field(None, gt=0)
```

**Key Points**:
- `id` is excluded (it's a path parameter)
- `bmi` and `verdict` are excluded (computed fields)
- All fields have default value of `None`

### Update Function
```python
@app.put("/edit/{patient_id}")
def update_patient(patient_id: str, patient_update: PatientUpdate):
    data = load_data()
    
    # Check if patient exists
    if patient_id not in data:
        raise HTTPException(status_code=404, detail="Patient Not Found")
    
    # Extract existing patient info
    existing_patient_info = data[patient_id]
    
    # Convert PatientUpdate to dictionary, excluding unset fields
    updated_patient_info_partial = patient_update.model_dump(exclude_unset=True)
    
    # Apply updates to existing info
    for key, value in updated_patient_info_partial.items():
        existing_patient_info[key] = value
    
    # Re-create Patient object to recalculate BMI and Verdict
    existing_patient_info['id'] = patient_id
    patient_pydantic_object = Patient(**existing_patient_info)
    
    # Convert back to dictionary, excluding 'id'
    updated_final_patient_data = patient_pydantic_object.model_dump(exclude={'id'})
    
    # Update and save
    data[patient_id] = updated_final_patient_data
    save_data(data)
    
    return JSONResponse(status_code=200, content={"message": "Patient Updated"})
```

### Critical Method: `model_dump(exclude_unset=True)`
This method converts the `PatientUpdate` object into a dictionary containing **only the fields that were actually provided** by the client, not all optional fields with `None` values.

### Recalculating Computed Fields
**Why the "roundabout" approach?**
- When weight or height are updated, BMI and Verdict must be recalculated
- Solution: Convert updated dictionary back into a `Patient` Pydantic object
- This triggers automatic recalculation of computed fields
- Then convert the `Patient` object back to dictionary for storage

### Update Endpoint Testing Examples

**Example 1: Update Multiple Fields**
- Patient P004: Arjun Verma, Bengaluru, Weight 95, BMI 29.32, Overweight
- Update: `city = "Mumbai"`, `weight = 90`
- Result: City → Mumbai, Weight → 90, BMI → 27, Verdict → Normal

**Example 2: Partial Update**
- Patient P001: Ananya Sharma
- Update: `name = "Ananya Verma"`
- Result: Only name changes, other fields unchanged

**Example 3: Non-existent Patient**
- Request: `patient_id = "P010"`
- Result: 404 Not Found error with "Patient Not Found"

## Delete Endpoint Implementation

### HTTP Method & Design
- **Method**: DELETE
- **Endpoint**: `/delete/{patient_id}`
- **Input**: Patient ID (path parameter only)

### Delete Function
```python
@app.delete("/delete/{patient_id}")
def delete_patient(patient_id: str):
    data = load_data()
    
    # Check if patient exists
    if patient_id not in data:
        raise HTTPException(status_code=404, detail="Patient Not Found")
    
    # Delete patient entry
    del data[patient_id]
    
    # Save updated data
    save_data(data)
    
    return JSONResponse(status_code=200, content={"message": "Patient Deleted"})
```

### Delete Endpoint Testing Examples

**Example 1: Successful Deletion**
- Request: `patient_id = "P006"`
- Result: 200 OK with "Patient Deleted" message

**Example 2: Non-existent Patient**
- Request: `patient_id = "P007"`
- Result: 404 Not Found error with "Patient Not Found"

## Complete CRUD Operations Summary

The patient management system now includes:
1. **Create**: Add new patients
2. **Retrieve**: View all patients, specific patient, sorted views
3. **Update**: Edit existing patient details with automatic recalculation of computed fields
4. **Delete**: Remove patient records

## Key Takeaways

- Use `Optional` fields with `Field(None)` for partial update models
- `model_dump(exclude_unset=True)` extracts only provided fields
- Recreate Pydantic objects to trigger computed field recalculation
- Always check for resource existence before update/delete operations
- Use appropriate HTTP status codes (200 for success, 404 for not found)