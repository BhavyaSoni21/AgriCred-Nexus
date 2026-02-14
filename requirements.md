# Requirements Document - AgriCred Nexus

## 1. Project Overview

### 1.1 Purpose
AgriCred Nexus is an AI-powered rural credit assessment system designed to provide fair and transparent loan evaluation for rural borrowers. The system combines financial data, behavioral insights, and machine learning to deliver objective risk assessments while promoting inclusive credit access.

### 1.2 Scope
A full-stack web application consisting of:
- Backend API for data management and risk assessment
- Frontend web interface for borrowers and lenders
- Machine learning integration for credit risk prediction
- Database for storing borrower profiles and loan applications

### 1.3 Target Users
- **Rural Borrowers**: Farmers and rural entrepreneurs seeking loans
- **Lenders**: Financial institutions evaluating loan applications
- **System Administrators**: Personnel managing the platform

---

## 2. Functional Requirements

### 2.1 Borrower Profile Management

#### 2.1.1 Profile Storage
- System shall store borrower profiles with the following attributes:
  - Record ID (unique identifier)
  - State/Region
  - Monthly Income
  - Business Revenue
  - Digital Transaction Frequency
  - Existing Loans Count
  - Past Repayment History

#### 2.1.2 Profile Retrieval
- System shall provide API endpoint to fetch borrower profile by record ID
- System shall return 404 error for non-existent record IDs
- System shall validate record ID format before database query

### 2.2 Loan Application Processing

#### 2.2.1 Application Submission
- System shall accept loan applications with:
  - Record ID (borrower identifier)
  - Loan Amount Requested
  - Loan Tenure (in months)

#### 2.2.2 Input Validation
- Loan amount must be positive number
- Loan tenure must be between 1 and 360 months
- Record ID must exist in database
- All required fields must be provided

#### 2.2.3 Application Storage
- System shall store all loan applications in database
- System shall record timestamp of application
- System shall link application to borrower profile

### 2.3 Risk Assessment

#### 2.3.1 ML Model Integration
- System shall integrate with trained ML model for risk prediction
- System shall support pluggable model architecture
- System shall handle missing model gracefully with mock predictions
- System shall maintain feature order consistency

#### 2.3.2 Risk Score Calculation
- System shall calculate risk score from ML model probability (0-100 scale)
- System shall categorize risk into three levels:
  - Low: 0-30
  - Medium: 31-70
  - High: 71-100

#### 2.3.3 Approval Recommendation
- System shall generate recommendation based on risk score:
  - Approve: Low risk (0-30)
  - Review: Medium risk (31-70)
  - Reject: High risk (71-100)

#### 2.3.4 Explanation Generation
- System shall provide human-readable explanation for each assessment
- Explanation shall reference key factors (income, repayment history, etc.)
- Explanation shall be contextual to risk category

### 2.4 Lender Dashboard

#### 2.4.1 Application Listing
- System shall display all loan applications
- System shall show borrower details for each application
- System shall display risk assessment results
- System shall support pagination for large datasets

#### 2.4.2 Filtering and Sorting
- System shall allow filtering by:
  - Risk category (Low/Medium/High)
  - Approval status (Approve/Review/Reject)
- System shall allow sorting by:
  - Record ID
  - State/Region
  - Monthly Income
  - Loan Amount
  - Loan Tenure
  - Risk Score
  - Risk Category
  - Approval Status

#### 2.4.3 Analytics
- System shall calculate and display:
  - Total applications count
  - Approved applications count
  - Review applications count
  - Rejected applications count
  - Average risk score
- System shall provide visual charts:
  - Approval distribution (pie chart)
  - Risk distribution (bar chart)

### 2.5 User Interface

#### 2.5.1 Landing Page
- System shall display hero section with value proposition
- System shall provide navigation to applicant portal and lender dashboard
- System shall explain how the system works (3-step process)
- System shall highlight trust factors and features

#### 2.5.2 Applicant Portal
- System shall implement 4-step application workflow:
  1. Enter Record ID and fetch profile
  2. Review borrower profile (read-only)
  3. Enter loan request details
  4. View risk assessment results
- System shall display loading states during API calls
- System shall show error messages for invalid inputs
- System shall visualize risk score with progress bar

#### 2.5.3 Lender Dashboard
- System shall display summary statistics in card format
- System shall show applications in sortable table
- System shall provide filter controls
- System shall render data visualization charts
- System shall update in real-time when filters change

### 2.6 Authentication (Future Enhancement)

#### 2.6.1 Farmer Authentication
- System shall support farmer login with credentials
- System shall allow farmers to view their loan history
- System shall restrict access to own applications only

#### 2.6.2 Lender Authentication
- System shall support lender login with credentials
- System shall allow lenders to access dashboard
- System shall support multiple lender accounts

---

## 3. Non-Functional Requirements

### 3.1 Performance

#### 3.1.1 Response Time
- API endpoints shall respond within 2 seconds under normal load
- ML model prediction shall complete within 1 second
- Frontend page load shall complete within 3 seconds

#### 3.1.2 Throughput
- System shall handle at least 100 concurrent users
- System shall process at least 50 loan applications per minute

### 3.2 Scalability

#### 3.2.1 Data Volume
- System shall support at least 10,000 borrower profiles
- System shall support at least 100,000 loan applications
- Database queries shall remain performant with growing data

#### 3.2.2 Architecture
- Backend shall be stateless to support horizontal scaling
- Database connections shall use connection pooling
- API shall support load balancing

### 3.3 Reliability

#### 3.3.1 Availability
- System shall maintain 99% uptime during business hours
- System shall handle database connection failures gracefully
- System shall provide meaningful error messages

#### 3.3.2 Data Integrity
- All database operations shall be transactional
- System shall validate all inputs before storage
- System shall prevent duplicate loan applications

### 3.4 Security

#### 3.4.1 Input Validation
- All API inputs shall be validated using Pydantic schemas
- System shall sanitize inputs to prevent SQL injection
- System shall reject malformed requests with 400 status

#### 3.4.2 Data Protection
- Sensitive data shall be stored securely
- API shall implement CORS restrictions
- System shall log security-relevant events

#### 3.4.3 Authentication & Authorization
- Protected endpoints shall require authentication
- Users shall only access authorized resources
- Session tokens shall expire after inactivity

### 3.5 Usability

#### 3.5.1 User Experience
- Interface shall be intuitive and require minimal training
- Error messages shall be clear and actionable
- Loading states shall provide visual feedback
- Forms shall validate inputs in real-time

#### 3.5.2 Accessibility
- Interface shall meet WCAG 2.1 Level AA standards
- Color contrast ratios shall exceed 4.5:1
- All interactive elements shall be keyboard accessible
- Screen readers shall be supported

#### 3.5.3 Responsiveness
- Interface shall work on desktop (1024px+)
- Interface shall work on tablet (768px-1023px)
- Interface shall work on mobile (320px-767px)
- Layout shall adapt to different screen sizes

### 3.6 Maintainability

#### 3.6.1 Code Quality
- Code shall follow PEP 8 style guide (Python)
- Code shall follow ESLint standards (JavaScript)
- Functions shall be documented with docstrings
- Complex logic shall include inline comments

#### 3.6.2 Testing
- Critical paths shall have unit tests
- API endpoints shall have integration tests
- ML model shall have validation tests
- Frontend components shall have component tests

#### 3.6.3 Documentation
- API shall provide OpenAPI/Swagger documentation
- README files shall explain setup and usage
- Code shall include architecture diagrams
- Design system shall be documented

### 3.7 Compatibility

#### 3.7.1 Browser Support
- System shall support Chrome 90+
- System shall support Firefox 88+
- System shall support Safari 14+
- System shall support Edge 90+

#### 3.7.2 Platform Support
- Backend shall run on Windows, macOS, Linux
- System shall support Python 3.8+
- System shall support Node.js 16+

---

## 4. Data Requirements

### 4.1 Database Schema

#### 4.1.1 Borrowers Table
```
- id (INTEGER, PRIMARY KEY, AUTO_INCREMENT)
- record_id (INTEGER, UNIQUE, NOT NULL)
- state_region (VARCHAR(100))
- monthly_income (FLOAT)
- business_revenue (FLOAT)
- digital_transaction_freq (INTEGER)
- existing_loans_count (INTEGER)
- past_repayment_history (VARCHAR(50))
```

#### 4.1.2 Loans Table
```
- id (INTEGER, PRIMARY KEY, AUTO_INCREMENT)
- borrower_id (INTEGER, FOREIGN KEY -> borrowers.id)
- loan_amount_requested (FLOAT)
- loan_tenure_months (INTEGER)
- risk_score (FLOAT)
- risk_category (VARCHAR(20))
- approval_recommendation (VARCHAR(20))
- explanation_text (TEXT)
- created_at (TIMESTAMP)
```

#### 4.1.3 Lenders Table (Future)
```
- id (INTEGER, PRIMARY KEY, AUTO_INCREMENT)
- username (VARCHAR(100), UNIQUE)
- password_hash (VARCHAR(255))
- institution_name (VARCHAR(200))
- created_at (TIMESTAMP)
```

### 4.2 ML Model Requirements

#### 4.2.1 Input Features
Model shall accept following features in order:
1. monthly_income
2. business_revenue
3. digital_transaction_freq
4. existing_loans_count
5. loan_amount_requested
6. loan_tenure_months
7. past_repayment_history (encoded)

#### 4.2.2 Output Format
- Model shall output probability value between 0 and 1
- Higher probability indicates higher default risk
- Model shall be serialized as .pkl file

#### 4.2.3 Feature Engineering
- Categorical features shall be encoded consistently
- Numerical features shall be scaled if required
- Feature order shall be documented in feature_order.json

---

## 5. API Requirements

### 5.1 Endpoint Specifications

#### 5.1.1 GET /borrower/{record_id}
**Purpose**: Retrieve borrower profile

**Request**:
- Path parameter: record_id (integer)

**Response** (200):
```json
{
  "record_id": 1001,
  "state_region": "Maharashtra",
  "monthly_income": 35000,
  "business_revenue": 60000,
  "digital_transaction_freq": 15,
  "existing_loans_count": 1,
  "past_repayment_history": "Good"
}
```

**Error Responses**:
- 404: Borrower not found
- 500: Server error

#### 5.1.2 POST /predict-risk
**Purpose**: Assess loan application risk

**Request Body**:
```json
{
  "record_id": 1001,
  "loan_amount_requested": 50000,
  "loan_tenure_months": 12
}
```

**Response** (200):
```json
{
  "risk_score": 35.5,
  "risk_category": "Medium",
  "approval_recommendation": "Review",
  "explanation_text": "Medium risk due to moderate income..."
}
```

**Error Responses**:
- 400: Invalid input
- 404: Borrower not found
- 500: Server error

#### 5.1.3 GET /dashboard
**Purpose**: Retrieve all loan applications

**Response** (200):
```json
[
  {
    "record_id": 1001,
    "state_region": "Maharashtra",
    "monthly_income": 35000,
    "loan_amount_requested": 50000,
    "loan_tenure_months": 12,
    "risk_score": 35.5,
    "risk_category": "Medium",
    "approval_recommendation": "Review"
  }
]
```

#### 5.1.4 GET /dashboard/analytics
**Purpose**: Retrieve portfolio analytics

**Response** (200):
```json
{
  "total_applications": 125,
  "approved_count": 45,
  "review_count": 60,
  "rejected_count": 20,
  "average_risk_score": 42.3,
  "approval_distribution": {
    "Approve": 45,
    "Review": 60,
    "Reject": 20
  },
  "risk_distribution": {
    "Low": 45,
    "Medium": 60,
    "High": 20
  }
}
```

### 5.2 API Standards

#### 5.2.1 HTTP Methods
- GET: Retrieve resources
- POST: Create resources
- PUT: Update resources (future)
- DELETE: Remove resources (future)

#### 5.2.2 Status Codes
- 200: Success
- 201: Created
- 400: Bad request
- 404: Not found
- 500: Server error

#### 5.2.3 Response Format
- All responses shall be JSON
- Error responses shall include error message
- Timestamps shall use ISO 8601 format

---

## 6. Integration Requirements

### 6.1 ML Model Integration
- System shall load model from backend/ml/AgriCred.pkl
- System shall read feature order from backend/ml/feature_order.json
- System shall handle model loading errors gracefully
- System shall support model hot-swapping without restart

### 6.2 Database Integration
- System shall use SQLAlchemy ORM
- System shall support SQLite for development
- System shall support PostgreSQL for production (future)
- System shall use connection pooling

### 6.3 Frontend-Backend Integration
- Frontend shall communicate via REST API
- API base URL shall be configurable
- System shall handle CORS properly
- System shall implement retry logic for failed requests

---

## 7. Deployment Requirements

### 7.1 Development Environment
- Backend shall run on localhost:8000
- Frontend shall run on localhost:3000
- Database shall be SQLite file
- Setup scripts shall automate installation

### 7.2 Production Environment (Future)
- Backend shall be containerized with Docker
- Frontend shall be served via CDN
- Database shall be managed PostgreSQL
- System shall use environment variables for configuration

### 7.3 Monitoring (Future)
- System shall log all API requests
- System shall track error rates
- System shall monitor response times
- System shall alert on anomalies

---

## 8. Constraints and Assumptions

### 8.1 Constraints
- Initial deployment limited to SQLite database
- No real-time collaboration features
- Single-tenant architecture
- Limited to English language

### 8.2 Assumptions
- Users have stable internet connection
- Borrower data is pre-populated in database
- ML model is pre-trained and provided
- Users have modern web browsers

---

## 9. Future Enhancements

### 9.1 Phase 2 Features
- User authentication and authorization
- Farmer dashboard for loan history
- Lender approval workflow
- Email notifications
- Document upload capability

### 9.2 Phase 3 Features
- Multi-language support
- Mobile native applications
- Advanced analytics and reporting
- Model retraining pipeline
- Integration with external credit bureaus

### 9.3 Phase 4 Features
- Multi-tenant architecture
- Real-time collaboration
- Blockchain integration for transparency
- AI-powered chatbot support
- Predictive analytics for portfolio management

---

## 10. Success Criteria

### 10.1 Technical Success
- All API endpoints functional and tested
- Frontend renders correctly on all supported browsers
- ML model integrates successfully
- System handles 100 concurrent users
- 99% uptime achieved

### 10.2 Business Success
- Loan processing time reduced by 80%
- Fair assessment for 95%+ of applications
- User satisfaction score above 4/5
- Lender adoption rate above 70%
- Borrower application completion rate above 85%

### 10.3 User Experience Success
- Application process completed in under 5 minutes
- Dashboard loads in under 3 seconds
- Error rate below 1%
- Accessibility compliance verified
- Mobile usability score above 90%
