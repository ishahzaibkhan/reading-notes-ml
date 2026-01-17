# Pydantic Crash Course: Data Validation in Python

## What is Pydantic?

Pydantic is a Python library for data validation that solves problems arising from Python's dynamic typing in production-grade software. Used extensively in FastAPI, YAML configuration files, and ML pipelines.

## Problems Pydantic Solves

### 1. Type Validation
- **Issue**: Python's dynamic typing allows variables to change types, leading to database insertion errors
- **Type hints alone don't enforce types** - they're just informational
- **Manual validation** (`if type(age) == int`) works but doesn't scale across multiple functions

### 2. Data Validation
- Beyond types, data needs constraints (e.g., age > 0, valid emails)
- Manual validation with `if` statements creates unmaintainable boilerplate code

## How Pydantic Works (3 Steps)

### Step 1: Build a Pydantic Model
```python
from pydantic import BaseModel

class Patient(BaseModel):
    name: str
    age: int
```
- Inherit from `BaseModel`
- Define fields with types and constraints

### Step 2: Instantiate the Model
```python
patient = Patient(**patient_data)
```
- Automatic validation on instantiation
- **Auto type coercion**: converts `"30"` to `30` if possible
- Raises `ValidationError` if validation fails

### Step 3: Use the Object
```python
patient.name  # Access via dot notation
```

## Building Complex Models

### Basic Types
```python
name: str
age: int
weight: float
married: bool
```

### Collections
```python
from typing import List, Dict

allergies: List[str]  # Validates list and each item
contact_details: Dict[str, str]  # Validates keys and values
```

### Optional Fields
```python
from typing import Optional

field_name: Optional[str] = None  # Must provide default value
married: bool = False  # Any field can have defaults
```

## Data Validation

### Custom Pydantic Types
```python
from pydantic import EmailStr, AnyUrl

email: EmailStr  # Auto validates email format
linkedin_url: AnyUrl  # Auto validates URL format
```

### Field Constraints
```python
from pydantic import Field

age: int = Field(gt=0, lt=120)  # Range validation
name: str = Field(max_length=50)  # String length
allergies: List[str] = Field(max_length=5)  # List size
weight: float = Field(gt=0, description="Patient weight in kg")  # With metadata
```

**Field operators**: `gt`, `lt`, `ge`, `le`, `max_length`

## Field Validator

For custom validation logic beyond built-in constraints:

```python
from pydantic import field_validator

@field_validator('name')
@classmethod
def name_alphabetic(cls, value):
    if any(char.isdigit() for char in value):
        raise ValueError('Name should not contain numbers')
    return value
```

**Can also transform data** (pre-processing):
```python
@field_validator('allergies')
@classmethod
def allergies_lowercase(cls, allergies_list):
    return [allergy.lower() for allergy in allergies_list]
```

**Validating nested data** (e.g., dict fields):
```python
@field_validator('contact_details')
@classmethod
def validate_phone(cls, contact_dict):
    phone = contact_dict.get('phone')
    if phone and len(str(phone)) != 10:
        raise ValueError('Phone must be 10 digits')
    return contact_dict
```

## Model Validator

For **cross-field validation** (one field depends on another):

```python
from pydantic import model_validator

@model_validator(mode='after')
def check_age_weight(self):
    if self.age < 18 and self.weight > 60:
        raise ValueError('For patients under 18, weight cannot exceed 60kg')
    return self
```

- `mode='after'` runs after all field validations pass
- Receives full model instance, accesses fields via `self`

## Computed Fields

Derive fields from existing data (read-only):

```python
from pydantic.functional_validators import computed_field

@computed_field
@property
def bmi(self) -> float:
    if self.height == 0:
        return 0.0
    return self.weight / (self.height ** 2)
```

Access: `patient.bmi`

## Nested Models

Create hierarchical structures for modularity:

```python
class Address(BaseModel):
    street: str
    city: str
    zip_code: str

class Patient(BaseModel):
    name: str
    address: Address  # Nested model
```

Access: `patient.address.city`

## Serialization

Convert Pydantic objects to dict/JSON:

```python
# To dictionary
patient.model_dump()

# To JSON string
patient.model_dump_json()

# Selective serialization
patient.model_dump(include={'name', 'age'})
patient.model_dump(exclude={'emergency_contact'})

# Exclude fields with default values (useful for PATCH requests)
patient.model_dump(exclude_unset=True)
```

## Key Benefits

- Eliminates boilerplate validation code
- Automatic type coercion
- Clear error messages via `ValidationError`
- Foundation for FastAPI and modern Python applications
- Enables data transformation during validation