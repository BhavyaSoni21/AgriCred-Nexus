# Design Document - AgriCred Nexus

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT LAYER                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   Landing    │  │  Applicant   │  │   Lender     │       │
│  │     Page     │  │   Portal     │  │  Dashboard   │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│                   React 18 + Vite                           │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP/REST (Port 3000)
                         │
┌────────────────────────▼────────────────────────────────────┐
│                   API GATEWAY LAYER                         │
│                   FastAPI (Port 8000)                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  CORS Middleware │ Validation │ Error Handling       │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
┌───────▼──────┐  ┌──────▼──────┐   ┌─────▼──────┐
│   ROUTES     │  │  SERVICES   │   │  DATABASE  │
│              │  │             │   │            │
│ • borrower   │  │ • risk      │   │ SQLAlchemy │
│ • prediction │  │ • explain   │   │    ORM     │
│ • dashboard  │  │ • analytics │   │            │
└──────────────┘  └──────┬──────┘   └─────┬──────┘
                         │                │
                  ┌──────▼──────┐  ┌──────▼──────┐
                  │  ML MODEL   │  │   SQLite    │
                  │  (.pkl)     │  │  Database   │
                  └─────────────┘  └─────────────┘
```

### 1.2 Technology Stack

#### Frontend Stack
- **Framework**: React 18.2.0
- **Build Tool**: Vite 4.3.9
- **Routing**: React Router DOM 6.11.2
- **HTTP Client**: Axios 1.4.0
- **Charts**: Recharts 2.6.2
- **Styling**: CSS3 (Custom)
- **Font**: Inter (Google Fonts)

#### Backend Stack
- **Framework**: FastAPI 0.95.1
- **ORM**: SQLAlchemy 2.0.13
- **Validation**: Pydantic 1.10.7
- **Database**: SQLite 3
- **ML Support**: scikit-learn 1.2.2, joblib 1.2.0
- **Server**: Uvicorn 0.22.0

#### Development Tools
- **Python**: 3.8+
- **Node.js**: 16+
- **Package Managers**: pip, npm

---

## 2. Database Design

### 2.1 Entity Relationship Diagram

```
┌─────────────────────────┐
│      BORROWERS          │
├─────────────────────────┤
│ PK  id                  │
│ UQ  record_id           │
│     state_region        │
│     monthly_income      │
│     business_revenue    │
│     digital_trans_freq  │
│     existing_loans_cnt  │
│     past_repay_history  │
└───────────┬─────────────┘
            │
            │ 1:N
            │
┌───────────▼─────────────┐
│        LOANS            │
├─────────────────────────┤
│ PK  id                  │
│ FK  borrower_id         │
│     loan_amount_req     │
│     loan_tenure_months  │
│     risk_score          │
│     risk_category       │
│     approval_recommend  │
│     explanation_text    │
│     created_at          │
└─────────────────────────┘

┌─────────────────────────┐
│       LENDERS           │
├─────────────────────────┤
│ PK  id                  │
│ UQ  username            │
│     password_hash       │
│     institution_name    │
│     created_at          │
└─────────────────────────┘
```

### 2.2 Table Specifications

#### Borrowers Table
```sql
CREATE TABLE borrowers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    record_id INTEGER UNIQUE NOT NULL,
    state_region VARCHAR(100),
    monthly_income FLOAT,
    business_revenue FLOAT,
    digital_transaction_freq INTEGER,
    existing_loans_count INTEGER,
    past_repayment_history VARCHAR(50)
);
```

#### Loans Table
```sql
CREATE TABLE loans (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    borrower_id INTEGER NOT NULL,
    loan_amount_requested FLOAT NOT NULL,
    loan_tenure_months INTEGER NOT NULL,
    risk_score FLOAT,
    risk_category VARCHAR(20),
    approval_recommendation VARCHAR(20),
    explanation_text TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (borrower_id) REFERENCES borrowers(id)
);
```


#### Lenders Table (Future)
```sql
CREATE TABLE lenders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    institution_name VARCHAR(200),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2.3 Indexes
```sql
CREATE INDEX idx_borrower_record_id ON borrowers(record_id);
CREATE INDEX idx_loan_borrower_id ON loans(borrower_id);
CREATE INDEX idx_loan_risk_category ON loans(risk_category);
CREATE INDEX idx_loan_created_at ON loans(created_at);
```

---

## 3. API Design

### 3.1 RESTful Endpoints

#### Borrower Endpoints
```
GET    /borrower/{record_id}     - Retrieve borrower profile
POST   /borrower                 - Create borrower (future)
PUT    /borrower/{record_id}     - Update borrower (future)
DELETE /borrower/{record_id}     - Delete borrower (future)
```

#### Prediction Endpoints
```
POST   /predict-risk             - Assess loan application
GET    /predictions/{id}         - Get prediction by ID (future)
```

#### Dashboard Endpoints
```
GET    /dashboard                - List all applications
GET    /dashboard/analytics      - Get portfolio analytics
GET    /dashboard/filters        - Get filter options (future)
```

#### Authentication Endpoints (Future)
```
POST   /auth/login               - User login
POST   /auth/logout              - User logout
POST   /auth/register            - User registration
GET    /auth/me                  - Get current user
```

### 3.2 Request/Response Schemas

#### BorrowerProfile Schema
```python
{
    "record_id": int,
    "state_region": str,
    "monthly_income": float,
    "business_revenue": float,
    "digital_transaction_freq": int,
    "existing_loans_count": int,
    "past_repayment_history": str
}
```

#### LoanRequest Schema
```python
{
    "record_id": int,
    "loan_amount_requested": float,  # > 0
    "loan_tenure_months": int        # 1-360
}
```

#### RiskAssessment Schema
```python
{
    "risk_score": float,              # 0-100
    "risk_category": str,             # Low/Medium/High
    "approval_recommendation": str,   # Approve/Review/Reject
    "explanation_text": str
}
```

#### DashboardApplication Schema
```python
{
    "record_id": int,
    "state_region": str,
    "monthly_income": float,
    "business_revenue": float,
    "loan_amount_requested": float,
    "loan_tenure_months": int,
    "risk_score": float,
    "risk_category": str,
    "approval_recommendation": str
}
```

#### Analytics Schema
```python
{
    "total_applications": int,
    "approved_count": int,
    "review_count": int,
    "rejected_count": int,
    "average_risk_score": float,
    "approval_distribution": {
        "Approve": int,
        "Review": int,
        "Reject": int
    },
    "risk_distribution": {
        "Low": int,
        "Medium": int,
        "High": int
    }
}
```

### 3.3 Error Response Format
```python
{
    "detail": str,           # Error message
    "status_code": int,      # HTTP status code
    "timestamp": str         # ISO 8601 timestamp
}
```

---

## 4. Backend Architecture

### 4.1 Directory Structure
```
backend/
├── main.py                    # FastAPI app entry point
├── config.py                  # Configuration settings
├── requirements.txt           # Python dependencies
│
├── routes/                    # API route handlers
│   ├── __init__.py
│   ├── borrower.py           # Borrower endpoints
│   ├── prediction.py         # Risk prediction endpoints
│   └── dashboard.py          # Dashboard endpoints
│
├── services/                  # Business logic layer
│   ├── __init__.py
│   ├── risk_engine.py        # ML model integration
│   ├── explanation_engine.py # Explanation generation
│   └── analytics_service.py  # Analytics calculations
│
├── database/                  # Data access layer
│   ├── __init__.py
│   ├── db.py                 # Database connection
│   └── models.py             # SQLAlchemy models
│
├── ml/                        # ML assets
│   ├── AgriCred.pkl          # Trained model
│   └── feature_order.json    # Feature specification
│
├── data/                      # Data files
│   └── mfi_complete_dataset_5000.csv
│
└── scripts/                   # Utility scripts
    ├── load_csv_data.py
    ├── load_sample_data.py
    ├── add_lenders.py
    └── retrain_model.py
```

### 4.2 Service Layer Design

#### RiskEngine Service
```python
class RiskEngine:
    def __init__(self):
        self.model = load_model()
        self.feature_order = load_feature_order()
    
    def predict_risk(self, borrower_data, loan_data):
        # Prepare features
        features = self._prepare_features(borrower_data, loan_data)
        
        # Get prediction
        probability = self.model.predict_proba(features)[0][1]
        
        # Calculate risk score
        risk_score = probability * 100
        
        # Determine category
        risk_category = self._categorize_risk(risk_score)
        
        # Generate recommendation
        recommendation = self._recommend_action(risk_category)
        
        return {
            "risk_score": risk_score,
            "risk_category": risk_category,
            "approval_recommendation": recommendation
        }
    
    def _categorize_risk(self, score):
        if score <= 30:
            return "Low"
        elif score <= 70:
            return "Medium"
        else:
            return "High"
    
    def _recommend_action(self, category):
        mapping = {
            "Low": "Approve",
            "Medium": "Review",
            "High": "Reject"
        }
        return mapping[category]
```

#### ExplanationEngine Service
```python
class ExplanationEngine:
    def generate_explanation(self, borrower, loan, risk_assessment):
        category = risk_assessment["risk_category"]
        
        if category == "Low":
            return self._low_risk_explanation(borrower, loan)
        elif category == "Medium":
            return self._medium_risk_explanation(borrower, loan)
        else:
            return self._high_risk_explanation(borrower, loan)
    
    def _low_risk_explanation(self, borrower, loan):
        factors = []
        
        if borrower.monthly_income > 30000:
            factors.append("strong income")
        
        if borrower.past_repayment_history == "Good":
            factors.append("excellent repayment history")
        
        if borrower.existing_loans_count <= 1:
            factors.append("low existing debt")
        
        return f"Low risk due to {', '.join(factors)}."
```

#### AnalyticsService
```python
class AnalyticsService:
    def calculate_portfolio_metrics(self, loans):
        total = len(loans)
        
        approved = sum(1 for l in loans if l.approval_recommendation == "Approve")
        review = sum(1 for l in loans if l.approval_recommendation == "Review")
        rejected = sum(1 for l in loans if l.approval_recommendation == "Reject")
        
        avg_risk = sum(l.risk_score for l in loans) / total if total > 0 else 0
        
        return {
            "total_applications": total,
            "approved_count": approved,
            "review_count": review,
            "rejected_count": rejected,
            "average_risk_score": round(avg_risk, 2)
        }
```

### 4.3 Configuration Management

#### config.py
```python
class Settings:
    # Database
    DATABASE_URL = "sqlite:///./database/agricred.db"
    
    # ML Model
    MODEL_PATH = "./ml/AgriCred.pkl"
    FEATURE_ORDER_PATH = "./ml/feature_order.json"
    
    # Risk Thresholds
    LOW_RISK_THRESHOLD = 30
    HIGH_RISK_THRESHOLD = 70
    
    # API
    API_TITLE = "AgriCred Nexus API"
    API_VERSION = "1.0.0"
    CORS_ORIGINS = ["http://localhost:3000"]
    
    # Server
    HOST = "0.0.0.0"
    PORT = 8000
    RELOAD = True
```

---

## 5. Frontend Architecture

### 5.1 Directory Structure
```
frontend/src/
├── main.jsx                   # Entry point
├── App.jsx                    # Main app with routing
├── index.css                  # Global styles
├── theme.js                   # Design system tokens
│
├── components/                # Reusable UI components
│   ├── Button/
│   │   ├── Button.jsx
│   │   └── Button.css
│   ├── Card/
│   ├── Input/
│   ├── Select/
│   ├── Badge/
│   ├── Loading/
│   └── Navbar/
│
├── pages/                     # Page components
│   ├── LandingPage/
│   ├── ApplicantPortal/
│   ├── LenderDashboard/
│   ├── FarmerLogin/
│   ├── FarmerDashboard/
│   ├── LenderLogin/
│   └── LenderDashboard/
│
└── services/
    └── api.js                 # API client
```

### 5.2 Component Architecture

#### Component Hierarchy
```
App
├── Navbar
├── LandingPage
│   ├── Hero Section
│   ├── How It Works
│   └── Trust Statement
│
├── ApplicantPortal
│   ├── Step 1: ID Input
│   │   └── Input
│   ├── Step 2: Profile Display
│   │   └── Card
│   ├── Step 3: Loan Request
│   │   ├── Input
│   │   └── Select
│   └── Step 4: Risk Result
│       ├── Card
│       ├── Badge
│       └── Progress Bar
│
└── LenderDashboard
    ├── Summary Cards (5x)
    │   └── Card
    ├── Filters
    │   └── Select
    ├── Applications Table
    │   └── Badge
    └── Charts
        ├── PieChart (Recharts)
        └── BarChart (Recharts)
```


#### Reusable Components

**Button Component**
```jsx
<Button 
  variant="primary|secondary|outline"
  size="small|medium|large"
  onClick={handleClick}
  disabled={false}
>
  Button Text
</Button>
```

**Card Component**
```jsx
<Card 
  title="Card Title"
  accent={true}
  className="custom-class"
>
  Card content
</Card>
```

**Input Component**
```jsx
<Input
  label="Input Label"
  type="text|number|email"
  value={value}
  onChange={handleChange}
  placeholder="Placeholder"
  error="Error message"
  requ
ired={true}
/>
```

**Select Component**
```jsx
<Select
  label="Select Label"
  value={value}
  onChange={handleChange}
  options={[
    { value: 'option1', label: 'Option 1' },
    { value: 'option2', label: 'Option 2' }
  ]}
  error="Error message"
/>
```

**Badge Component**
```jsx
<Badge 
  variant="success|warning|danger"
  text="Badge Text"
/>
```

**Loading Component**
```jsx
<Loading 
  message="Loading message..."
/>
```

### 5.3 State Management

#### Application State Flow
```
User Action → Component State Update → API Call → Response → State Update → UI Re-render
```

#### State Management Strategy
- Local component state using React useState
- No global state management (Redux/Context) in v1.0
- API calls managed through services/api.js
- Form state managed within page components

#### Example State Structure (ApplicantPortal)
```javascript
const [step, setStep] = useState(1);
const [recordId, setRecordId] = useState('');
const [borrowerProfile, setBorrowerProfile] = useState(null);
const [loanRequest, setLoanRequest] = useState({
  loan_amount_requested: '',
  loan_tenure_months: ''
});
const [riskResult, setRiskResult] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState('');
```

### 5.4 API Integration

#### API Client Configuration
```javascript
// services/api.js
import axios from 'axios';

const API_BASE_URL = 'http://localhost:8000';

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
  timeout: 10000,
});

// Request interceptor
api.interceptors.request.use(
  (config) => {
    // Add auth token if available
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Handle unauthorized
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

#### API Methods
```javascript
// Borrower API
export const getBorrowerProfile = (recordId) => 
  api.get(`/borrower/${recordId}`);

// Prediction API
export const predictRisk = (data) => 
  api.post('/predict-risk', data);

// Dashboard API
export const getDashboardApplications = () => 
  api.get('/dashboard');

export const getDashboardAnalytics = () => 
  api.get('/dashboard/analytics');

// Auth API (Future)
export const login = (credentials) => 
  api.post('/auth/login', credentials);

export const logout = () => 
  api.post('/auth/logout');
```

### 5.5 Design System

#### Color Palette
```javascript
// Earth-Tone Solid Design System
colors: {
  primary: {
    deepSoilBrown: '#5C4033',
    mutedRiverBlue: '#3E7C87',
    sandBeige: '#F5E6D3',
    charcoal: '#2E2E2E',
  },
  indicators: {
    success: '#7B9971',
    warning: '#D4A574',
    danger: '#8B4249',
  },
  neutral: {
    white: '#FFFFFF',
    lightBeige: '#F9F3E8',
    mediumBrown: '#8B7355',
    darkBrown: '#3D2E21',
  }
}
```

#### Typography
```javascript
typography: {
  fontFamily: "'Inter', sans-serif",
  fontSize: {
    xs: '12px',
    sm: '14px',
    base: '16px',
    lg: '18px',
    xl: '20px',
    xxl: '24px',
    xxxl: '32px',
    display: '48px',
  },
  fontWeight: {
    light: 300,
    normal: 400,
    medium: 500,
    semibold: 600,
    bold: 700,
  }
}
```

#### Spacing System
```javascript
spacing: {
  xs: '4px',
  sm: '8px',
  md: '16px',
  lg: '24px',
  xl: '32px',
  xxl: '48px',
  xxxl: '64px',
}
```

#### Shadows
```javascript
shadows: {
  subtle: '0 2px 4px rgba(92, 64, 51, 0.08)',
  card: '0 2px 8px rgba(92, 64, 51, 0.12)',
  elevated: '0 4px 12px rgba(92, 64, 51, 0.15)',
}
```

---

## 6. Machine Learning Integration

### 6.1 Model Architecture

#### Model Type
- **Algorithm**: Logistic Regression / Random Forest / Gradient Boosting
- **Framework**: scikit-learn
- **Serialization**: joblib/pickle (.pkl format)
- **Version**: Tracked in model metadata

#### Feature Engineering Pipeline
```python
# Feature Order (13 features total)
features = [
    'monthly_income',              # Numerical
    'business_revenue',            # Numerical
    'digital_transaction_freq',    # Numerical
    'existing_loans_count',        # Numerical
    'loan_amount_requested',       # Numerical
    'past_repayment_history',      # Numerical (encoded)
    'loan_tenure_months',          # Numerical
    'state_region_KA',             # One-hot encoded
    'state_region_KL',             # One-hot encoded
    'state_region_MH',             # One-hot encoded
    'state_region_Others',         # One-hot encoded
    'state_region_TN',             # One-hot encoded
    'state_region_UP',             # One-hot encoded
]
```

#### State Region Encoding
```python
state_mapping = {
    '1': 'MH',  '2': 'PB',  '3': 'KA',  '4': 'UP',
    '5': 'TN',  '6': 'GJ',  '7': 'RJ',  '8': 'WB',
    '9': 'MP',  '10': 'AP', '11': 'Others', '12': 'KL'
}

# Model recognizes: KA, KL, MH, Others, TN, UP
# All other states mapped to 'Others'
```

### 6.2 Prediction Pipeline

#### Step-by-Step Process
```
1. Receive borrower data + loan request
2. Extract and validate features
3. Encode categorical variables (state_region)
4. Convert to numerical format
5. Arrange features in correct order
6. Pass to ML model
7. Get probability output (0-1)
8. Convert to risk score (0-100)
9. Categorize risk (Low/Medium/High)
10. Generate recommendation (Approve/Review/Reject)
11. Create explanation text
12. Return complete assessment
```

#### Risk Scoring Logic
```python
# Probability to Risk Score
risk_score = probability * 100

# Risk Categorization
if risk_score <= 30:
    category = "Low"
    recommendation = "Approve"
elif risk_score <= 70:
    category = "Medium"
    recommendation = "Review"
else:
    category = "High"
    recommendation = "Reject"
```

### 6.3 Model Deployment

#### Model Loading
```python
class RiskEngine:
    def __init__(self):
        self.model = None
        self.feature_order = None
        self._load_model()
        self._load_feature_order()
    
    def _load_model(self):
        if MODEL_PATH.exists():
            with open(MODEL_PATH, 'rb') as f:
                self.model = pickle.load(f)
        else:
            # Fallback to mock predictions
            self.model = None
```

#### Model Hot-Swapping
- Replace AgriCred.pkl file
- Restart backend service
- Model automatically reloaded on startup
- No code changes required

#### Model Versioning (Future)
- Track model version in metadata
- Store multiple model versions
- A/B testing capability
- Rollback mechanism

---

## 7. Security Design

### 7.1 Authentication & Authorization (Future)

#### JWT Token-Based Authentication
```python
# Token Structure
{
  "sub": "user_id",
  "username": "john_doe",
  "role": "lender|farmer",
  "exp": 1234567890,
  "iat": 1234567890
}
```

#### Authentication Flow
```
1. User submits credentials
2. Backend validates against database
3. Generate JWT token with user info
4. Return token to frontend
5. Frontend stores token in localStorage
6. Include token in Authorization header
7. Backend validates token on protected routes
8. Grant/deny access based on role
```

#### Role-Based Access Control
```python
roles = {
    "farmer": [
        "view_own_profile",
        "submit_application",
        "view_own_applications"
    ],
    "lender": [
        "view_all_applications",
        "view_analytics",
        "update_application_status"
    ],
    "admin": [
        "all_permissions"
    ]
}
```

### 7.2 Input Validation

#### Backend Validation (Pydantic)
```python
from pydantic import BaseModel, Field, validator

class LoanRequest(BaseModel):
    record_id: int = Field(..., gt=0)
    loan_amount_requested: float = Field(..., gt=0)
    loan_tenure_months: int = Field(..., ge=1, le=360)
    
    @validator('loan_amount_requested')
    def validate_amount(cls, v):
        if v > 10000000:  # 1 crore max
            raise ValueError('Loan amount too high')
        return v
```

#### Frontend Validation
```javascript
const validateLoanRequest = (data) => {
  const errors = {};
  
  if (!data.loan_amount_requested || data.loan_amount_requested <= 0) {
    errors.amount = 'Amount must be positive';
  }
  
  if (!data.loan_tenure_months || 
      data.loan_tenure_months < 1 || 
      data.loan_tenure_months > 360) {
    errors.tenure = 'Tenure must be 1-360 months';
  }
  
  return errors;
};
```

### 7.3 Data Protection

#### Sensitive Data Handling
- Passwords hashed using bcrypt (cost factor 12)
- No plain text password storage
- PII encrypted at rest (future)
- HTTPS enforced in production

#### SQL Injection Prevention
- SQLAlchemy ORM parameterized queries
- No raw SQL string concatenation
- Input sanitization at API layer

#### CORS Configuration
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # Specific origins
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
)
```

---

## 8. Error Handling

### 8.1 Backend Error Handling

#### Exception Hierarchy
```python
class AgriCredException(Exception):
    """Base exception"""
    pass

class BorrowerNotFoundException(AgriCredException):
    """Borrower not found"""
    pass

class ModelNotLoadedException(AgriCredException):
    """ML model not loaded"""
    pass

class ValidationException(AgriCredException):
    """Input validation failed"""
    pass
```

#### Error Response Format
```python
{
    "detail": "Borrower with record_id 9999 not found",
    "status_code": 404,
    "timestamp": "2026-02-14T10:30:00Z",
    "path": "/borrower/9999"
}
```

#### Global Exception Handler
```python
@app.exception_handler(BorrowerNotFoundException)
async def borrower_not_found_handler(request, exc):
    return JSONResponse(
        status_code=404,
        content={
            "detail": str(exc),
            "status_code": 404,
            "timestamp": datetime.utcnow().isoformat()
        }
    )
```

### 8.2 Frontend Error Handling

#### Error Display Strategy
```javascript
const [error, setError] = useState({
  message: '',
  type: 'error|warning|info',
  field: 'fieldName' // For field-specific errors
});

// Display error
{error.message && (
  <div className={`alert alert-${error.type}`}>
    {error.message}
  </div>
)}
```

#### API Error Handling
```javascript
try {
  const response = await api.post('/predict-risk', data);
  setRiskResult(response.data);
} catch (error) {
  if (error.response) {
    // Server responded with error
    setError({
      message: error.response.data.detail,
      type: 'error'
    });
  } else if (error.request) {
    // No response received
    setError({
      message: 'Server not responding. Please try again.',
      type: 'error'
    });
  } else {
    // Request setup error
    setError({
      message: 'An unexpected error occurred.',
      type: 'error'
    });
  }
}
```

### 8.3 Logging Strategy

#### Backend Logging
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('agricred.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

# Usage
logger.info(f"Risk assessment for record_id {record_id}")
logger.error(f"Model prediction failed: {str(e)}")
logger.warning(f"High risk application: {record_id}")
```

#### Log Levels
- **DEBUG**: Detailed diagnostic information
- **INFO**: General informational messages
- **WARNING**: Warning messages (high risk, etc.)
- **ERROR**: Error messages (exceptions, failures)
- **CRITICAL**: Critical system failures

---

## 9. Performance Optimization

### 9.1 Backend Optimization

#### Database Query Optimization
```python
# Use eager loading to prevent N+1 queries
loans = session.query(Loan)\
    .options(joinedload(Loan.borrower))\
    .all()

# Index frequently queried columns
CREATE INDEX idx_loan_risk_category ON loans(risk_category);
CREATE INDEX idx_loan_created_at ON loans(created_at);
```

#### Caching Strategy (Future)
```python
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_borrower_profile(record_id: int):
    # Cache borrower profiles
    return db.query(Borrower).filter_by(record_id=record_id).first()
```

#### Connection Pooling
```python
engine = create_engine(
    DATABASE_URL,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True
)
```

### 9.2 Frontend Optimization

#### Code Splitting
```javascript
// Lazy load pages
const LenderDashboard = lazy(() => 
  import('./pages/LenderDashboard/LenderDashboard')
);

// Use Suspense
<Suspense fallback={<Loading />}>
  <LenderDashboard />
</Suspense>
```

#### Memoization
```javascript
import { useMemo, useCallback } from 'react';

// Memoize expensive calculations
const filteredApplications = useMemo(() => {
  return applications.filter(app => 
    app.risk_category === selectedFilter
  );
}, [applications, selectedFilter]);

// Memoize callbacks
const handleFilterChange = useCallback((filter) => {
  setSelectedFilter(filter);
}, []);
```

#### Asset Optimization
- Minify CSS and JavaScript
- Compress images
- Use CDN for static assets
- Enable gzip compression
- Implement lazy loading for images

---

## 10. Testing Strategy

### 10.1 Backend Testing

#### Unit Tests
```python
# tests/test_risk_engine.py
def test_risk_categorization():
    engine = RiskEngine()
    
    assert engine.get_risk_category(25) == "Low"
    assert engine.get_risk_category(50) == "Medium"
    assert engine.get_risk_category(85) == "High"

def test_approval_recommendation():
    engine = RiskEngine()
    
    assert engine.get_approval_recommendation("Low") == "Approve"
    assert engine.get_approval_recommendation("Medium") == "Review"
    assert engine.get_approval_recommendation("High") == "Reject"
```

#### Integration Tests
```python
# tests/test_api.py
def test_predict_risk_endpoint():
    response = client.post('/predict-risk', json={
        'record_id': 1001,
        'loan_amount_requested': 50000,
        'loan_tenure_months': 12
    })
    
    assert response.status_code == 200
    assert 'risk_score' in response.json()
    assert 'risk_category' in response.json()
```

#### Test Coverage Goals
- Unit tests: 80%+ coverage
- Integration tests: All API endpoints
- Edge cases: Invalid inputs, missing data

### 10.2 Frontend Testing

#### Component Tests
```javascript
// Button.test.jsx
import { render, fireEvent } from '@testing-library/react';
import Button from './Button';

test('button click triggers callback', () => {
  const handleClick = jest.fn();
  const { getByText } = render(
    <Button onClick={handleClick}>Click Me</Button>
  );
  
  fireEvent.click(getByText('Click Me'));
  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

#### E2E Tests (Future)
```javascript
// Cypress or Playwright
describe('Loan Application Flow', () => {
  it('completes full application', () => {
    cy.visit('/apply');
    cy.get('input[name="recordId"]').type('1001');
    cy.get('button').contains('Fetch Profile').click();
    cy.get('input[name="amount"]').type('50000');
    cy.get('button').contains('Submit').click();
    cy.contains('Risk Assessment').should('be.visible');
  });
});
```

### 10.3 Manual Testing Checklist

#### Functional Testing
- [ ] Borrower profile retrieval
- [ ] Loan application submission
- [ ] Risk assessment calculation
- [ ] Dashboard data display
- [ ] Filtering and sorting
- [ ] Error handling

#### UI/UX Testing
- [ ] Responsive design (mobile, tablet, desktop)
- [ ] Loading states
- [ ] Error messages
- [ ] Form validation
- [ ] Navigation flow

#### Browser Compatibility
- [ ] Chrome 90+
- [ ] Firefox 88+
- [ ] Safari 14+
- [ ] Edge 90+

---

## 11. Deployment Architecture

### 11.1 Development Environment

#### Local Setup
```
Backend:  http://localhost:8000
Frontend: http://localhost:3000
Database: ./backend/database/agricred.db
```

#### Setup Scripts
```bash
# Windows
scripts/setup-backend.bat
scripts/setup-frontend.bat
scripts/run-backend.bat
scripts/run-frontend.bat
```

### 11.2 Production Architecture (Future)

#### Infrastructure Diagram
```
┌─────────────────────────────────────────────────┐
│              Load Balancer (Nginx)              │
└────────────┬────────────────────────┬───────────┘
             │                        │
    ┌────────▼────────┐      ┌────────▼────────┐
    │  Frontend CDN   │      │  API Gateway    │
    │  (Static Files) │      │  (FastAPI)      │
    └─────────────────┘      └────────┬────────┘
                                      │
                             ┌────────▼────────┐
                             │   PostgreSQL    │
                             │   (Primary DB)  │
                             └─────────────────┘
```

#### Container Strategy
```dockerfile
# Backend Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

# Frontend Dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
RUN npm run build
CMD ["npm", "run", "preview"]
```

#### Docker Compose
```yaml
version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/agricred
    depends_on:
      - db
  
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
  
  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=agricred
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 11.3 CI/CD Pipeline (Future)

#### GitHub Actions Workflow
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run backend tests
        run: |
          cd backend
          pip install -r requirements.txt
          pytest
      - name: Run frontend tests
        run: |
          cd frontend
          npm install
          npm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: |
          # Deployment commands
```

---

## 12. Monitoring & Observability (Future)

### 12.1 Application Monitoring

#### Metrics to Track
- Request rate (requests/second)
- Response time (p50, p95, p99)
- Error rate (%)
- Model prediction latency
- Database query time

#### Health Checks
```python
@app.get("/health")
def health_check():
    return {
        "status": "healthy",
        "database": check_db_connection(),
        "model": check_model_loaded(),
        "timestamp": datetime.utcnow().isoformat()
    }
```

### 12.2 Logging & Alerting

#### Structured Logging
```python
logger.info("risk_assessment", extra={
    "record_id": record_id,
    "risk_score": risk_score,
    "risk_category": risk_category,
    "processing_time_ms": processing_time
})
```

#### Alert Conditions
- Error rate > 5%
- Response time > 3 seconds
- Database connection failures
- Model prediction failures
- High risk application spike

---

## 13. Future Enhancements

### 13.1 Phase 2 Features

#### User Authentication
- JWT-based authentication
- Role-based access control
- Password reset functionality
- Session management

#### Enhanced Dashboard
- Advanced filtering options
- Export to CSV/PDF
- Custom date range selection
- Comparison views

#### Notifications
- Email notifications for application status
- SMS alerts for lenders
- In-app notification center

### 13.2 Phase 3 Features

#### Mobile Applications
- React Native mobile app
- Offline capability
- Push notifications
- Biometric authentication

#### Advanced Analytics
- Predictive portfolio analytics
- Trend analysis
- Risk forecasting
- Custom report builder

#### Document Management
- Upload supporting documents
- OCR for document extraction
- Document verification
- Secure document storage

### 13.3 Phase 4 Features

#### AI Enhancements
- Chatbot for applicant support
- Automated document verification
- Fraud detection
- Credit score prediction

#### Blockchain Integration
- Immutable audit trail
- Smart contracts for loan agreements
- Decentralized identity verification
- Transparent credit history

#### Multi-Tenancy
- Support multiple financial institutions
- Tenant isolation
- Custom branding per tenant
- Tenant-specific configurations

---

## 14. Appendices

### 14.1 Glossary

- **Risk Score**: Numerical value (0-100) indicating default probability
- **Risk Category**: Classification (Low/Medium/High) based on risk score
- **Approval Recommendation**: Suggested action (Approve/Review/Reject)
- **Record ID**: Unique identifier for borrower profile
- **Loan Tenure**: Duration of loan in months
- **Digital Transaction Frequency**: Number of digital transactions per month

### 14.2 References

- FastAPI Documentation: https://fastapi.tiangolo.com/
- React Documentation: https://react.dev/
- SQLAlchemy Documentation: https://docs.sqlalchemy.org/
- scikit-learn Documentation: https://scikit-learn.org/

### 14.3 Change Log

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2026-02-14 | Initial design document | AgriCred Team |
