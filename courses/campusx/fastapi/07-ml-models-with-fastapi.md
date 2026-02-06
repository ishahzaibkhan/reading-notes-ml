# Serving Machine Learning Models with FastAPI - Core Concepts

## Three-Part Architecture

### 1. Model Building
- Train ML model locally (e.g., Jupyter Notebook)
- Apply feature engineering to transform raw data
- Export trained model as `.pkl` file using pickle

### 2. API Endpoint (FastAPI)
- Load the trained model
- Create POST endpoint (`/predict`)
- Handle input validation using Pydantic
- Perform feature engineering internally (user sends raw data only)
- Return predictions in JSON format

### 3. Front-End (Streamlit)
- Build user interface with input forms
- Send HTTP POST requests to API
- Display predictions to users

## Application Flow
User Input → Front-End → API (validates & processes) → ML Model → Prediction → API → Front-End → Display Result

---

## Key Design Decisions

### Feature Engineering Strategy
**Problem:** Insurance premium prediction (High/Medium/Low)

**Raw Features:** Age, Weight, Height, Income, Smoker status, City, Occupation

**Engineered Features:**
- **BMI** - calculated from weight/height
- **Age Group** - categorized (Young/Adult/Middle Age/Senior)
- **Lifestyle Risk** - derived from BMI + smoker status (High/Medium/Low)
- **City Tier** - categorized cities (Tier 1/2/3)

### Why Feature Engineering in API?
- Simplifies user experience (no manual calculations)
- Centralizes logic in one place
- User only provides raw, intuitive inputs
- API handles all transformations internally

### Using Computed Fields (Pydantic)
- Automatically calculate derived features from raw inputs
- Validation happens before computation
- Clean separation between input and processed data

---

## Important Technical Concepts

### Why POST Method for ML APIs?
- POST sends data to server for processing
- Standard for all ML/DL prediction endpoints
- Used when creating resources or performing computations

### Input Validation Benefits
- Ensures data quality before reaching model
- Prevents errors (e.g., negative age, invalid occupation)
- Uses type constraints and literal types for strict validation

### Model Serving Pipeline
1. Receive raw user data
2. Validate inputs
3. Calculate engineered features
4. Format data as DataFrame matching training structure
5. Generate prediction
6. Return JSON response

---

## Practical Advantages

### For Users
- Simple, intuitive inputs
- No technical knowledge required
- Immediate predictions via web interface

### For Developers
- Centralized feature engineering logic
- Reusable API for multiple front-ends
- Easy testing via Swagger UI
- Same approach works for any ML/DL framework

### For Organizations
- Makes models accessible beyond data science team
- Standardized interface for model consumption
- Separates concerns (model, API, UI)

---

## Universal Applicability

This architecture works for:
- Traditional ML (scikit-learn, XGBoost)
- Deep Learning (PyTorch, TensorFlow)
- Any serializable trained model
- Classification, regression, or other prediction tasks