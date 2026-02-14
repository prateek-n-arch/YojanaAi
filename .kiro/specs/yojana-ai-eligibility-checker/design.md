# Design Document: YojanaAI - Government Scheme Eligibility Checker

## Overview

YojanaAI is a full-stack web application that intelligently matches Indian citizens with government welfare schemes they qualify for. The system uses a rule-based eligibility engine combined with AI-powered natural language explanations to provide personalized guidance.

### Architecture Philosophy

The design follows a three-tier architecture with clear separation of concerns:
- **Presentation Layer**: Next.js frontend with server-side rendering for SEO and performance
- **Application Layer**: Node.js/Express API server handling business logic
- **Data Layer**: PostgreSQL database via Supabase for structured data storage

The system prioritizes:
- **Scalability**: Stateless API design enabling horizontal scaling
- **Performance**: Aggressive caching and database indexing for sub-2-second responses
- **Maintainability**: Modular service-oriented architecture
- **Security**: Defense-in-depth with encryption, authentication, and input validation

## Architecture

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Next.js Frontend (Vercel)                               │  │
│  │  - React Components                                       │  │
│  │  - Tailwind CSS                                          │  │
│  │  - i18n (Hindi/English)                                  │  │
│  │  - Client-side validation                                │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS/REST
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Application Layer                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Node.js/Express API Server (Render)                     │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │  Controllers                                        │  │  │
│  │  │  - AuthController                                   │  │  │
│  │  │  - ProfileController                                │  │  │
│  │  │  - SchemeController                                 │  │  │
│  │  │  - EligibilityController                            │  │  │
│  │  │  - AdminController                                  │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │  Services                                           │  │  │
│  │  │  - EligibilityService (Core matching logic)        │  │  │
│  │  │  - AIExplainerService (LLM integration)            │  │  │
│  │  │  - SchemeService (CRUD operations)                 │  │  │
│  │  │  - NotificationService (Email/alerts)              │  │  │
│  │  │  - CacheService (Redis integration)                │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │  Middleware                                         │  │  │
│  │  │  - authMiddleware (JWT validation)                 │  │  │
│  │  │  - rateLimitMiddleware                             │  │  │
│  │  │  - validationMiddleware                            │  │  │
│  │  │  - errorHandler                                    │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────────┐    ┌──────────────┐
│   Supabase   │    │   LLM API        │    │    Redis     │
│  PostgreSQL  │    │  (OpenRouter/    │    │   Cache      │
│              │    │   OpenAI)        │    │              │
│  - Users     │    │                  │    │  - Schemes   │
│  - Profiles  │    │  - Explanations  │    │  - Results   │
│  - Schemes   │    │  - Translations  │    │              │
│  - Audit     │    │                  │    │              │
└──────────────┘    └──────────────────┘    └──────────────┘
```

### Technology Stack Rationale

**Frontend: Next.js + Tailwind CSS**
- Server-side rendering for SEO and initial load performance
- Built-in API routes for BFF (Backend for Frontend) pattern
- Tailwind for rapid UI development with consistent design system
- Native i18n support for multilingual requirements

**Backend: Node.js + Express**
- JavaScript ecosystem consistency with frontend
- Large ecosystem of libraries for rapid development
- Non-blocking I/O suitable for I/O-bound eligibility matching
- Easy integration with LLM APIs

**Database: PostgreSQL (Supabase)**
- ACID compliance for data integrity
- Rich querying capabilities for complex eligibility matching
- JSON support for flexible scheme criteria storage
- Built-in authentication and row-level security
- Managed hosting reduces operational overhead

**AI Layer: OpenRouter/OpenAI**
- Access to multiple LLM models for cost optimization
- Reliable API with high uptime
- Support for structured outputs for consistent explanations

**Caching: Redis**
- In-memory performance for frequently accessed schemes
- Reduces database load for read-heavy workload
- TTL support for cache invalidation

## Components and Interfaces

### Frontend Components

#### 1. ProfileForm Component
```typescript
interface ProfileFormProps {
  initialData?: UserProfile;
  onSubmit: (profile: UserProfile) => Promise<void>;
  isLoading: boolean;
}

interface UserProfile {
  age: number;
  annualIncome: number;
  state: string;
  occupation: string;
  category: 'SC' | 'ST' | 'OBC' | 'General' | 'EWS';
  gender: 'Male' | 'Female' | 'Other';
  locationType: 'Rural' | 'Urban';
}
```

**Responsibilities:**
- Render form fields with validation
- Handle user input with real-time validation
- Submit profile data to API
- Display validation errors

#### 2. SchemeCard Component
```typescript
interface SchemeCardProps {
  scheme: Scheme;
  eligibilityStatus: 'eligible' | 'ineligible' | 'partial';
  explanation: string;
  onViewDetails: (schemeId: string) => void;
}

interface Scheme {
  id: string;
  name: string;
  description: string;
  benefits: string;
  eligibilityCriteria: EligibilityCriteria;
  documents: string[];
  applicationLink?: string;
  state?: string;
  category?: string[];
  deadline?: Date;
}
```

**Responsibilities:**
- Display scheme summary information
- Show eligibility status with visual indicators
- Render AI-generated explanation
- Provide action buttons for details and application

#### 3. SearchAndFilter Component
```typescript
interface SearchAndFilterProps {
  onSearch: (query: string) => void;
  onFilterChange: (filters: FilterOptions) => void;
  availableFilters: FilterMetadata;
}

interface FilterOptions {
  categories?: string[];
  states?: string[];
  schemeTypes?: string[];
  eligibilityStatus?: 'eligible' | 'ineligible' | 'all';
}
```

**Responsibilities:**
- Provide search input with debouncing
- Render filter controls
- Emit search and filter events to parent

#### 4. AdminSchemeForm Component
```typescript
interface AdminSchemeFormProps {
  scheme?: Scheme;
  onSave: (scheme: Scheme) => Promise<void>;
  onCancel: () => void;
}
```

**Responsibilities:**
- Render comprehensive scheme editing form
- Handle eligibility criteria builder
- Support multilingual content entry
- Validate scheme data before submission

### Backend Services

#### 1. EligibilityService

```typescript
class EligibilityService {
  /**
   * Evaluates user profile against all schemes
   */
  async evaluateEligibility(
    userProfile: UserProfile
  ): Promise<EligibilityResult[]>;

  /**
   * Evaluates user profile against a single scheme
   */
  async evaluateSingleScheme(
    userProfile: UserProfile,
    scheme: Scheme
  ): Promise<EligibilityResult>;

  /**
   * Calculates relevance score for ranking
   */
  private calculateRelevanceScore(
    userProfile: UserProfile,
    scheme: Scheme,
    matchedCriteria: string[]
  ): number;

  /**
   * Checks if user meets specific criterion
   */
  private checkCriterion(
    userValue: any,
    criterion: Criterion
  ): boolean;
}

interface EligibilityResult {
  scheme: Scheme;
  isEligible: boolean;
  matchedCriteria: string[];
  unmatchedCriteria: string[];
  relevanceScore: number;
}

interface Criterion {
  field: string;
  operator: 'eq' | 'gt' | 'lt' | 'gte' | 'lte' | 'in' | 'between';
  value: any;
}
```

**Algorithm:**
```
For each scheme:
  1. Initialize matchedCriteria = []
  2. Initialize unmatchedCriteria = []
  3. For each criterion in scheme.eligibilityCriteria:
       a. If criterion.field not in userProfile:
            Skip (optional criterion)
       b. Else:
            result = checkCriterion(userProfile[field], criterion)
            If result == true:
              Add to matchedCriteria
            Else:
              Add to unmatchedCriteria
  4. isEligible = (unmatchedCriteria.length == 0)
  5. relevanceScore = calculateRelevanceScore(...)
  6. Return EligibilityResult
```

#### 2. AIExplainerService

```typescript
class AIExplainerService {
  /**
   * Generates natural language explanation for eligibility
   */
  async generateExplanation(
    userProfile: UserProfile,
    scheme: Scheme,
    eligibilityResult: EligibilityResult,
    language: 'en' | 'hi'
  ): Promise<string>;

  /**
   * Builds prompt for LLM
   */
  private buildPrompt(
    userProfile: UserProfile,
    scheme: Scheme,
    eligibilityResult: EligibilityResult,
    language: string
  ): string;

  /**
   * Calls LLM API with retry logic
   */
  private async callLLM(
    prompt: string,
    temperature: number
  ): Promise<string>;
}
```

#### 3. SchemeService

```typescript
class SchemeService {
  /**
   * Retrieves all schemes with optional filtering
   */
  async getSchemes(filters?: SchemeFilters): Promise<Scheme[]>;

  /**
   * Retrieves single scheme by ID
   */
  async getSchemeById(id: string): Promise<Scheme | null>;

  /**
   * Creates new scheme (admin only)
   */
  async createScheme(scheme: Scheme, adminId: string): Promise<Scheme>;

  /**
   * Updates existing scheme (admin only)
   */
  async updateScheme(
    id: string,
    updates: Partial<Scheme>,
    adminId: string
  ): Promise<Scheme>;

  /**
   * Deletes scheme (admin only)
   */
  async deleteScheme(id: string, adminId: string): Promise<void>;

  /**
   * Bulk imports schemes from CSV/JSON
   */
  async bulkImport(
    data: any[],
    format: 'csv' | 'json',
    adminId: string
  ): Promise<ImportResult>;

  /**
   * Searches schemes by text query
   */
  async searchSchemes(
    query: string,
    language: string
  ): Promise<Scheme[]>;
}
```

#### 4. CacheService

```typescript
class CacheService {
  /**
   * Gets value from cache
   */
  async get<T>(key: string): Promise<T | null>;

  /**
   * Sets value in cache with TTL
   */
  async set<T>(key: string, value: T, ttlSeconds: number): Promise<void>;

  /**
   * Deletes value from cache
   */
  async delete(key: string): Promise<void>;

  /**
   * Invalidates cache by pattern
   */
  async invalidatePattern(pattern: string): Promise<void>;
}
```

**Caching Strategy:**
- All schemes: TTL 1 hour (key: `schemes:all`)
- Single scheme: TTL 1 hour (key: `scheme:{id}`)
- Eligibility results: TTL 5 minutes (key: `eligibility:{profileHash}`)
- Search results: TTL 15 minutes (key: `search:{query}:{language}`)
- Invalidate on scheme updates/deletes

#### 5. NotificationService

```typescript
class NotificationService {
  /**
   * Sends email notification
   */
  async sendEmail(
    to: string,
    subject: string,
    body: string,
    language: string
  ): Promise<void>;

  /**
   * Sends new scheme notification to eligible users
   */
  async notifyNewScheme(scheme: Scheme): Promise<void>;

  /**
   * Sends deadline reminder
   */
  async sendDeadlineReminder(
    scheme: Scheme,
    daysBeforeDeadline: number
  ): Promise<void>;

  /**
   * Processes daily digest for opted-in users
   */
  async processDailyDigest(): Promise<void>;
}
```

### API Endpoints

#### Authentication Endpoints

```
POST /api/v1/auth/register
Body: { email, password, name }
Response: { user, token }

POST /api/v1/auth/login
Body: { email, password }
Response: { user, token }

POST /api/v1/auth/logout
Headers: Authorization: Bearer {token}
Response: { success: true }

POST /api/v1/auth/forgot-password
Body: { email }
Response: { success: true }

POST /api/v1/auth/reset-password
Body: { token, newPassword }
Response: { success: true }
```

#### Profile Endpoints

```
GET /api/v1/profile
Headers: Authorization: Bearer {token}
Response: { profile: UserProfile }

PUT /api/v1/profile
Headers: Authorization: Bearer {token}
Body: { profile: UserProfile }
Response: { profile: UserProfile }

DELETE /api/v1/profile
Headers: Authorization: Bearer {token}
Response: { success: true }
```

#### Eligibility Endpoints

```
POST /api/v1/eligibility/check
Headers: Authorization: Bearer {token}
Body: { profile: UserProfile, language?: string }
Response: {
  eligible: EligibilityResult[],
  ineligible: EligibilityResult[],
  totalSchemes: number
}

GET /api/v1/eligibility/explain/:schemeId
Headers: Authorization: Bearer {token}
Query: ?language=en
Response: {
  explanation: string,
  eligibilityStatus: string
}
```

#### Scheme Endpoints

```
GET /api/v1/schemes
Query: ?state=&category=&page=1&limit=20&language=en
Response: {
  schemes: Scheme[],
  total: number,
  page: number,
  totalPages: number
}

GET /api/v1/schemes/:id
Query: ?language=en
Response: { scheme: Scheme }

GET /api/v1/schemes/search
Query: ?q=education&language=en
Response: { schemes: Scheme[], total: number }
```

#### Admin Endpoints

```
POST /api/v1/admin/schemes
Headers: Authorization: Bearer {adminToken}
Body: { scheme: Scheme }
Response: { scheme: Scheme }

PUT /api/v1/admin/schemes/:id
Headers: Authorization: Bearer {adminToken}
Body: { updates: Partial<Scheme> }
Response: { scheme: Scheme }

DELETE /api/v1/admin/schemes/:id
Headers: Authorization: Bearer {adminToken}
Response: { success: true }

POST /api/v1/admin/schemes/bulk-import
Headers: Authorization: Bearer {adminToken}
Body: FormData with file
Response: {
  imported: number,
  failed: number,
  errors: ImportError[]
}

GET /api/v1/admin/audit-logs
Headers: Authorization: Bearer {adminToken}
Query: ?page=1&limit=50
Response: {
  logs: AuditLog[],
  total: number
}
```

## Data Models

### Database Schema

```sql
-- Users table (managed by Supabase Auth)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- User profiles table
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  age INTEGER NOT NULL CHECK (age >= 0 AND age <= 120),
  annual_income DECIMAL(12, 2) NOT NULL CHECK (annual_income >= 0),
  state VARCHAR(50) NOT NULL,
  occupation VARCHAR(100) NOT NULL,
  category VARCHAR(20) NOT NULL CHECK (category IN ('SC', 'ST', 'OBC', 'General', 'EWS')),
  gender VARCHAR(20) NOT NULL CHECK (gender IN ('Male', 'Female', 'Other')),
  location_type VARCHAR(20) NOT NULL CHECK (location_type IN ('Rural', 'Urban')),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id)
);

-- Schemes table
CREATE TABLE schemes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(255) NOT NULL,
  name_hi VARCHAR(255),
  description TEXT NOT NULL,
  description_hi TEXT,
  benefits TEXT NOT NULL,
  benefits_hi TEXT,
  eligibility_criteria JSONB NOT NULL,
  documents JSONB NOT NULL,
  application_link VARCHAR(500),
  state VARCHAR(50),
  scheme_type VARCHAR(50),
  priority INTEGER DEFAULT 0,
  deadline TIMESTAMP,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_schemes_state ON schemes(state);
CREATE INDEX idx_schemes_active ON schemes(is_active);
CREATE INDEX idx_schemes_eligibility ON schemes USING GIN(eligibility_criteria);
CREATE INDEX idx_user_profiles_user_id ON user_profiles(user_id);

-- Full-text search indexes
CREATE INDEX idx_schemes_name_search ON schemes USING GIN(to_tsvector('english', name));
CREATE INDEX idx_schemes_description_search ON schemes USING GIN(to_tsvector('english', description));

-- Notification preferences table
CREATE TABLE notification_preferences (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  email_enabled BOOLEAN DEFAULT true,
  frequency VARCHAR(20) DEFAULT 'digest' CHECK (frequency IN ('immediate', 'digest', 'none')),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id)
);

-- Audit logs table
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  admin_id UUID REFERENCES users(id),
  action VARCHAR(50) NOT NULL,
  resource_type VARCHAR(50) NOT NULL,
  resource_id UUID,
  changes JSONB,
  ip_address INET,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_audit_logs_admin ON audit_logs(admin_id);
CREATE INDEX idx_audit_logs_created ON audit_logs(created_at DESC);

-- Admin roles table
CREATE TABLE admin_roles (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role VARCHAR(50) NOT NULL CHECK (role IN ('super_admin', 'admin', 'moderator')),
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id)
);
```

### Eligibility Criteria JSON Structure

```json
{
  "age": {
    "operator": "between",
    "value": [18, 35]
  },
  "annualIncome": {
    "operator": "lte",
    "value": 300000
  },
  "state": {
    "operator": "in",
    "value": ["Maharashtra", "Gujarat", "Karnataka"]
  },
  "category": {
    "operator": "in",
    "value": ["SC", "ST"]
  },
  "gender": {
    "operator": "eq",
    "value": "Female"
  },
  "locationType": {
    "operator": "eq",
    "value": "Rural"
  }
}
```

**Operator Semantics:**
- `eq`: Equal to
- `gt`: Greater than
- `lt`: Less than
- `gte`: Greater than or equal to
- `lte`: Less than or equal to
- `in`: Value is in array
- `between`: Value is between two numbers (inclusive)

### Documents JSON Structure

```json
[
  "Aadhaar Card",
  "Income Certificate",
  "Caste Certificate",
  "Bank Account Details",
  "Passport Size Photograph"
]
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*


### Property Reflection

After analyzing all acceptance criteria, I've identified the following redundancies and consolidations:

**Redundancies Eliminated:**
- Criteria 1.7-1.10 (dropdown options) can be combined into one property about form field options
- Criteria 4.2 and 4.4 (document display) are redundant - 4.2 covers 4.4
- Criteria 3.5 and 5.7 (AI language) are duplicates
- Criteria 7.3 and 14.1 (scheme validation) test the same behavior
- Multiple validation criteria (1.5, 1.6, 8.5, 14.3, 14.4) can be consolidated into edge case tests

**Properties Combined:**
- Profile validation (1.2, 1.3) combined into comprehensive validation property
- Scheme display properties (4.1, 4.2, 4.3) combined into complete rendering property
- Authentication flow properties (8.1, 8.3, 8.4) streamlined to avoid overlap
- Admin validation properties (7.3, 7.10, 14.1, 14.2) consolidated

This reduces ~100 testable criteria to ~45 unique, non-redundant properties.

### Correctness Properties


Property 1: Profile validation completeness
*For any* profile submission, if any required field (age, income, state, occupation, category, gender, locationType) is missing, the validation system should reject the submission and return specific error messages identifying all missing fields.
**Validates: Requirements 1.2, 1.3**

Property 2: Profile update triggers recalculation
*For any* user profile update, the eligibility engine should immediately recalculate eligible schemes and return updated results.
**Validates: Requirements 1.4**

Property 3: Eligibility evaluation completeness
*For any* user profile, the eligibility engine should evaluate all active schemes in the database and return results for every scheme.
**Validates: Requirements 2.1**

Property 4: Eligibility matching uses AND logic
*For any* user profile and scheme, the user should be marked eligible only if they satisfy ALL specified criteria in the scheme's eligibility requirements (logical AND).
**Validates: Requirements 2.2**

Property 5: Optional criteria are automatically satisfied
*For any* scheme with missing criteria fields, those unspecified criteria should be treated as automatically satisfied for all users.
**Validates: Requirements 2.3**

Property 6: Results sorted by relevance
*For any* eligibility evaluation result set, schemes should be ordered by relevance score in descending order.
**Validates: Requirements 2.4**


Property 7: Relevance score calculation
*For any* user profile and scheme, the relevance score should increase with the number of matched criteria and the scheme's priority value.
**Validates: Requirements 2.5**

Property 8: Results separated by eligibility
*For any* eligibility evaluation, eligible schemes and ineligible schemes should be returned in separate lists with no overlap.
**Validates: Requirements 2.6**

Property 9: AI explanation generation
*For any* scheme and user profile combination, the AI explainer should generate a natural language explanation in the requested language.
**Validates: Requirements 3.1, 3.5**

Property 10: Eligible explanations mention matched criteria
*For any* eligible scheme, the AI-generated explanation should reference at least one criterion that the user satisfies.
**Validates: Requirements 3.2**

Property 11: Ineligible explanations mention unmatched criteria
*For any* ineligible scheme, the AI-generated explanation should reference at least one criterion that the user does not satisfy.
**Validates: Requirements 3.3**

Property 12: Explanations include numerical comparisons
*For any* scheme with numerical criteria (age, income), the AI explanation should include specific numbers from the user's profile when explaining eligibility.
**Validates: Requirements 3.7**


Property 13: Scheme display completeness
*For any* scheme, the display should include name, description, benefits, eligibility criteria, and all required documents.
**Validates: Requirements 4.1, 4.2**

Property 14: Application link conditional display
*For any* scheme with an application link, the link should be displayed; for schemes without a link, no broken or empty link should appear.
**Validates: Requirements 4.3**

Property 15: Scheme display respects language
*For any* scheme and selected language, if a translation exists for that language, the translated content should be displayed; otherwise, the original content should be shown with a language indicator.
**Validates: Requirements 4.5, 5.5, 5.6**

Property 16: State-specific scheme filtering
*For any* user profile with a specific state, only schemes that either have no state restriction or match the user's state should be shown as relevant.
**Validates: Requirements 4.6**

Property 17: Deadline highlighting
*For any* scheme with a deadline field, the deadline should be visually highlighted in the display.
**Validates: Requirements 4.7**

Property 18: Language preference persistence
*For any* user who selects a language, that preference should be saved and restored in subsequent sessions.
**Validates: Requirements 5.2**


Property 19: UI translation on language change
*For any* language change, all UI elements, labels, and static content should update to the selected language immediately.
**Validates: Requirements 5.4**

Property 20: Search matches across fields
*For any* search query, the results should include all schemes where the query matches any part of the name, description, or benefits fields.
**Validates: Requirements 6.1**

Property 21: Cross-language search
*For any* search query in Hindi or English, schemes should be found regardless of the language in which the scheme data is stored.
**Validates: Requirements 6.2**

Property 22: Filter conjunction
*For any* combination of filters applied, only schemes matching ALL selected filter criteria should be returned.
**Validates: Requirements 6.3**

Property 23: Filter result count updates
*For any* filter change, the displayed result count should update to reflect the number of schemes matching the current filter combination.
**Validates: Requirements 6.5**

Property 24: Filter reset restores full list
*For any* filtered result set, clearing all filters should restore the complete list of eligible schemes.
**Validates: Requirements 6.6**


Property 25: Admin authentication required
*For any* admin panel access attempt, authentication credentials must be verified before granting access.
**Validates: Requirements 7.1**

Property 26: Scheme list pagination
*For any* scheme list request with pagination parameters, the response should contain the correct page of results and total count.
**Validates: Requirements 7.2**

Property 27: Scheme creation validation
*For any* scheme creation attempt, all required fields must be present and valid, or the creation should be rejected with specific error messages.
**Validates: Requirements 7.3, 14.1**

Property 28: Scheme update persistence
*For any* scheme update, the changes should be immediately saved to the database and reflected in subsequent queries.
**Validates: Requirements 7.4**

Property 29: Deletion requires confirmation
*For any* scheme deletion attempt, a confirmation step must be completed before the scheme is permanently removed.
**Validates: Requirements 7.5**

Property 30: Bulk import format support
*For any* bulk import operation, both CSV and JSON formats should be accepted and processed correctly.
**Validates: Requirements 7.6**


Property 31: Import validation with detailed errors
*For any* bulk import with invalid entries, the system should validate each entry and return detailed error messages identifying which entries failed and why.
**Validates: Requirements 7.7**

Property 32: Audit log creation
*For any* scheme modification (create, update, delete), an audit log entry should be created with timestamp, administrator identity, and change details.
**Validates: Requirements 7.8**

Property 33: Eligibility criteria validation
*For any* scheme with eligibility criteria, the criteria should use valid operators (eq, gt, lt, gte, lte, in, between) and appropriate data types for each field.
**Validates: Requirements 7.10, 14.2**

Property 34: User registration creates account
*For any* valid registration request with email and password, a new user account should be created and a verification email sent.
**Validates: Requirements 8.1, 8.2**

Property 35: Login creates session
*For any* valid login with correct credentials, a session should be created and returned to the user.
**Validates: Requirements 8.3**

Property 36: Logout invalidates session
*For any* logout request with a valid session, that session should be immediately invalidated and no longer grant access.
**Validates: Requirements 8.4**


Property 37: Password reset email sent
*For any* password reset request with a registered email, a reset link should be sent to that email address.
**Validates: Requirements 8.6**

Property 38: Authenticated profile loading
*For any* authenticated user, their saved profile data should be automatically loaded and available.
**Validates: Requirements 8.8**

Property 39: Login rate limiting
*For any* IP address, after 5 failed login attempts within 15 minutes, subsequent login attempts should be blocked until the time window expires.
**Validates: Requirements 8.9**

Property 40: Password hashing with bcrypt
*For any* password stored in the system, it should be hashed using bcrypt with at least 12 rounds, not stored in plain text.
**Validates: Requirements 9.3**

Property 41: User data deletion
*For any* user data deletion request, all associated user data should be permanently removed from the system.
**Validates: Requirements 9.5**

Property 42: Role-based access control
*For any* admin function access attempt, the user's role should be verified, and access should be denied if the user lacks the required role.
**Validates: Requirements 9.6**


Property 43: Input sanitization
*For any* user input received by the API, the input should be validated and sanitized to prevent injection attacks before processing.
**Validates: Requirements 9.7**

Property 44: Security event logging
*For any* security-relevant event (failed login, unauthorized access attempt, data deletion), a log entry should be created.
**Validates: Requirements 9.8**

Property 45: MFA for admin access
*For any* admin function access, multi-factor authentication should be required in addition to password authentication.
**Validates: Requirements 9.9**

Property 46: Cache TTL respected
*For any* cached scheme data, the cache should expire after 1 hour and fresh data should be fetched from the database.
**Validates: Requirements 10.4**

Property 47: Keyboard navigation support
*For any* interactive element in the UI, it should be accessible and operable via keyboard navigation.
**Validates: Requirements 11.2**

Property 48: ARIA labels present
*For any* form input or button, appropriate ARIA labels should be present for screen reader accessibility.
**Validates: Requirements 11.3**


Property 49: Color contrast compliance
*For any* text displayed in the UI, the color contrast ratio between text and background should be at least 4.5:1.
**Validates: Requirements 11.4**

Property 50: Image alt text
*For any* image displayed in the UI, descriptive alt text should be provided.
**Validates: Requirements 11.5**

Property 51: Heading hierarchy
*For any* page in the UI, headings should follow proper hierarchical structure (h1, h2, h3) for screen reader navigation.
**Validates: Requirements 11.6**

Property 52: Focus indicators visible
*For any* interactive element, a visible focus indicator should appear when the element receives keyboard focus.
**Validates: Requirements 11.8**

Property 53: Monitoring alerts triggered
*For any* 5-minute window where the error rate exceeds 5%, an alert should be sent to administrators.
**Validates: Requirements 12.2**

Property 54: Analytics event tracking
*For any* user action (search, scheme view, profile completion), an analytics event should be tracked.
**Validates: Requirements 12.4**


Property 55: Popular content aggregation
*For any* analytics query for most viewed schemes or popular searches, the system should return aggregated data sorted by frequency.
**Validates: Requirements 12.5**

Property 56: PII exclusion from analytics
*For any* analytics data collected, personally identifiable information (email, name, exact income) should not be included.
**Validates: Requirements 12.7**

Property 57: JSON response format
*For any* API endpoint, responses should be in valid JSON format with consistent structure (success/error, data, message).
**Validates: Requirements 13.2**

Property 58: HTTP status codes for errors
*For any* API error, the response should include an appropriate HTTP status code (400 for validation, 401 for auth, 404 for not found, 500 for server error).
**Validates: Requirements 13.3**

Property 59: API rate limiting
*For any* API key, after 100 requests within 1 minute, subsequent requests should be rejected with 429 status until the window resets.
**Validates: Requirements 13.6**

Property 60: JWT authentication for protected endpoints
*For any* protected API endpoint, a valid JWT token must be provided in the Authorization header, or the request should be rejected with 401 status.
**Validates: Requirements 13.7**


Property 61: Field-level validation errors
*For any* API request with invalid data, the response should include field-level error details identifying which fields are invalid and why.
**Validates: Requirements 13.8**

Property 62: Pagination support
*For any* list API endpoint, pagination parameters (page, limit) should be supported and the response should include total count and page metadata.
**Validates: Requirements 13.9**

Property 63: State code validation
*For any* scheme with state criteria, the state codes should be validated against the list of valid Indian state identifiers.
**Validates: Requirements 14.5**

Property 64: Category validation
*For any* scheme with category criteria, the category values should be validated to ensure they are from the defined set (SC, ST, OBC, General, EWS).
**Validates: Requirements 14.6**

Property 65: URL validation
*For any* scheme with an application link, the URL should be validated for proper format.
**Validates: Requirements 14.7**

Property 66: Duplicate scheme prevention
*For any* scheme creation attempt, if a scheme with the same name and state already exists, the creation should be rejected.
**Validates: Requirements 14.8**


Property 67: New scheme notifications
*For any* new scheme added that matches a user's profile, if the user has opted in to notifications, an email notification should be sent.
**Validates: Requirements 15.1**

Property 68: Notification frequency preferences
*For any* user with notification preferences set, notifications should be sent according to their chosen frequency (immediate or daily digest).
**Validates: Requirements 15.2**

Property 69: Notification content completeness
*For any* notification sent, it should include the scheme name, brief description, and a direct link to view the scheme.
**Validates: Requirements 15.4**

Property 70: Unsubscribe functionality
*For any* notification email, an unsubscribe link should be included, and clicking it should disable future notifications for that user.
**Validates: Requirements 15.5**

Property 71: Digest mode rate limiting
*For any* user in daily digest mode, no more than one notification email should be sent per day.
**Validates: Requirements 15.6**

Property 72: Deadline reminder notifications
*For any* scheme with an approaching deadline, eligible users who have opted in should receive reminder notifications.
**Validates: Requirements 15.7**

Property 73: Notification delivery tracking
*For any* notification sent, the delivery status and any bounce information should be tracked and stored.
**Validates: Requirements 15.8**



## Error Handling

### Error Categories and Handling Strategy

#### 1. Validation Errors (400 Bad Request)

**Scenarios:**
- Missing required fields in profile or scheme data
- Invalid data types (non-numeric age, negative income)
- Out-of-range values (age > 120, invalid state codes)
- Malformed eligibility criteria JSON

**Handling:**
```typescript
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "age",
        "message": "Age must be between 0 and 120"
      },
      {
        "field": "state",
        "message": "Invalid state code"
      }
    ]
  }
}
```

**Implementation:**
- Use express-validator middleware for request validation
- Return field-level errors with specific messages
- Log validation errors for monitoring

#### 2. Authentication Errors (401 Unauthorized)

**Scenarios:**
- Missing JWT token
- Expired JWT token
- Invalid JWT signature
- Revoked session

**Handling:**
```typescript
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication required"
  }
}
```

**Implementation:**
- Verify JWT in authMiddleware
- Return 401 with clear message
- Do not expose token details for security



#### 3. Authorization Errors (403 Forbidden)

**Scenarios:**
- Non-admin accessing admin endpoints
- Insufficient role permissions
- MFA not completed for admin actions

**Handling:**
```typescript
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "Insufficient permissions"
  }
}
```

**Implementation:**
- Check user role in authorization middleware
- Return 403 for insufficient permissions
- Log unauthorized access attempts

#### 4. Not Found Errors (404 Not Found)

**Scenarios:**
- Scheme ID does not exist
- User profile not found
- Invalid API endpoint

**Handling:**
```typescript
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "Scheme not found"
  }
}
```

**Implementation:**
- Check resource existence before operations
- Return 404 with resource type in message
- Distinguish between invalid endpoint and missing resource

#### 5. Rate Limiting Errors (429 Too Many Requests)

**Scenarios:**
- Exceeded API rate limit (100 req/min)
- Exceeded login attempts (5 per 15 min)

**Handling:**
```typescript
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retryAfter": 45
  }
}
```

**Implementation:**
- Use express-rate-limit middleware
- Include Retry-After header
- Log rate limit violations



#### 6. External Service Errors (502/503)

**Scenarios:**
- LLM API timeout or failure
- Database connection failure
- Email service unavailable

**Handling:**
```typescript
{
  "success": false,
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "AI explanation service temporarily unavailable"
  }
}
```

**Implementation:**
- Implement retry logic with exponential backoff
- Graceful degradation (return results without AI explanations)
- Circuit breaker pattern for repeated failures
- Log external service errors with correlation IDs

#### 7. Server Errors (500 Internal Server Error)

**Scenarios:**
- Unhandled exceptions
- Database query errors
- Unexpected null/undefined values

**Handling:**
```typescript
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred",
    "requestId": "req_abc123"
  }
}
```

**Implementation:**
- Global error handler middleware
- Log full error details server-side
- Return generic message to client (no stack traces)
- Include request ID for support tracking

### Error Logging Strategy

**Log Levels:**
- ERROR: Authentication failures, database errors, unhandled exceptions
- WARN: Rate limit violations, validation errors, external service timeouts
- INFO: Successful operations, audit events
- DEBUG: Detailed request/response data (development only)

**Log Format:**
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "requestId": "req_abc123",
  "userId": "user_xyz789",
  "error": {
    "code": "DATABASE_ERROR",
    "message": "Connection timeout",
    "stack": "..."
  },
  "context": {
    "endpoint": "/api/v1/eligibility/check",
    "method": "POST"
  }
}
```



## Testing Strategy

### Overview

YojanaAI employs a comprehensive testing strategy combining unit tests, property-based tests, integration tests, and end-to-end tests. The dual approach of unit testing and property-based testing ensures both specific examples and universal properties are validated.

### Property-Based Testing

**Framework:** fast-check (for TypeScript/JavaScript)

**Configuration:**
- Minimum 100 iterations per property test
- Each test tagged with feature name and property number
- Seed-based reproducibility for failed tests

**Tag Format:**
```typescript
// Feature: yojana-ai-eligibility-checker, Property 4: Eligibility matching uses AND logic
```

**Key Property Tests:**

1. **Eligibility Matching (Properties 3-8)**
   - Generate random user profiles and schemes
   - Verify AND logic for criteria matching
   - Test optional criteria handling
   - Validate relevance scoring

2. **Validation (Properties 1, 27, 33, 43)**
   - Generate invalid inputs with missing/malformed fields
   - Verify all validation rules catch errors
   - Test boundary conditions

3. **Search and Filter (Properties 20-24)**
   - Generate random search queries and filter combinations
   - Verify result correctness
   - Test cross-language search

4. **Authentication and Authorization (Properties 34-36, 39, 42)**
   - Generate random credentials and session tokens
   - Verify authentication flows
   - Test rate limiting with rapid requests

5. **Caching (Property 46)**
   - Test cache hit/miss behavior
   - Verify TTL expiration
   - Test cache invalidation

**Example Property Test:**
```typescript
import fc from 'fast-check';

// Feature: yojana-ai-eligibility-checker, Property 4: Eligibility matching uses AND logic
describe('Eligibility Matching', () => {
  it('should mark user eligible only if ALL criteria are satisfied', () => {
    fc.assert(
      fc.property(
        fc.record({
          age: fc.integer({ min: 0, max: 120 }),
          annualIncome: fc.integer({ min: 0, max: 10000000 }),
          state: fc.constantFrom('Maharashtra', 'Gujarat', 'Karnataka'),
          category: fc.constantFrom('SC', 'ST', 'OBC', 'General', 'EWS'),
          gender: fc.constantFrom('Male', 'Female', 'Other'),
          locationType: fc.constantFrom('Rural', 'Urban')
        }),
        fc.record({
          eligibilityCriteria: fc.record({
            age: fc.option(fc.record({
              operator: fc.constantFrom('gte', 'lte', 'between'),
              value: fc.oneof(
                fc.integer({ min: 0, max: 120 }),
                fc.tuple(fc.integer({ min: 0, max: 60 }), fc.integer({ min: 61, max: 120 }))
              )
            })),
            annualIncome: fc.option(fc.record({
              operator: fc.constantFrom('lte', 'gte'),
              value: fc.integer({ min: 0, max: 10000000 })
            }))
          })
        }),
        (userProfile, scheme) => {
          const result = eligibilityService.evaluateSingleScheme(userProfile, scheme);
          
          // If eligible, all specified criteria must be satisfied
          if (result.isEligible) {
            expect(result.unmatchedCriteria).toHaveLength(0);
          }
          
          // If ineligible, at least one criterion must be unsatisfied
          if (!result.isEligible) {
            expect(result.unmatchedCriteria.length).toBeGreaterThan(0);
          }
        }
      ),
      { numRuns: 100 }
    );
  });
});
```



### Unit Testing

**Framework:** Jest

**Focus Areas:**
- Specific examples demonstrating correct behavior
- Edge cases (empty inputs, boundary values, special characters)
- Error conditions and exception handling
- Integration points between components

**Key Unit Tests:**

1. **Profile Form Validation**
   - Test age boundaries (0, 120, -1, 121)
   - Test negative income rejection
   - Test empty required fields
   - Test dropdown options presence

2. **Eligibility Engine**
   - Test specific scheme matching scenarios
   - Test empty scheme list handling
   - Test zero eligible schemes message
   - Test relevance score calculation with known inputs

3. **AI Explainer**
   - Test explanation generation with mock LLM responses
   - Test timeout handling
   - Test retry logic
   - Test language parameter passing

4. **Admin Operations**
   - Test scheme CRUD operations
   - Test bulk import with sample CSV/JSON
   - Test audit log creation
   - Test duplicate prevention

5. **Authentication**
   - Test registration with valid/invalid emails
   - Test password hashing verification
   - Test JWT token generation and validation
   - Test session invalidation

**Example Unit Test:**
```typescript
describe('EligibilityService', () => {
  describe('evaluateSingleScheme', () => {
    it('should return eligible for user meeting all criteria', () => {
      const userProfile = {
        age: 25,
        annualIncome: 200000,
        state: 'Maharashtra',
        category: 'SC',
        gender: 'Female',
        locationType: 'Rural'
      };
      
      const scheme = {
        name: 'Women Empowerment Scheme',
        eligibilityCriteria: {
          age: { operator: 'between', value: [18, 35] },
          annualIncome: { operator: 'lte', value: 300000 },
          gender: { operator: 'eq', value: 'Female' }
        }
      };
      
      const result = eligibilityService.evaluateSingleScheme(userProfile, scheme);
      
      expect(result.isEligible).toBe(true);
      expect(result.matchedCriteria).toContain('age');
      expect(result.matchedCriteria).toContain('annualIncome');
      expect(result.matchedCriteria).toContain('gender');
      expect(result.unmatchedCriteria).toHaveLength(0);
    });
    
    it('should return ineligible when age is out of range', () => {
      const userProfile = {
        age: 40,
        annualIncome: 200000,
        state: 'Maharashtra',
        category: 'SC',
        gender: 'Female',
        locationType: 'Rural'
      };
      
      const scheme = {
        name: 'Youth Scheme',
        eligibilityCriteria: {
          age: { operator: 'between', value: [18, 35] }
        }
      };
      
      const result = eligibilityService.evaluateSingleScheme(userProfile, scheme);
      
      expect(result.isEligible).toBe(false);
      expect(result.unmatchedCriteria).toContain('age');
    });
  });
});
```



### Integration Testing

**Framework:** Jest + Supertest

**Focus:** Testing API endpoints with real database (test instance)

**Key Integration Tests:**

1. **End-to-End Eligibility Flow**
   - Create user profile via API
   - Fetch eligible schemes
   - Verify response structure and data

2. **Admin Workflow**
   - Admin login
   - Create scheme
   - Verify scheme appears in public API
   - Update scheme
   - Delete scheme

3. **Search and Filter**
   - Create test schemes
   - Test search queries
   - Test filter combinations
   - Verify result accuracy

4. **Authentication Flow**
   - Register user
   - Verify email sent (mock)
   - Login
   - Access protected endpoint
   - Logout
   - Verify access denied

**Example Integration Test:**
```typescript
describe('Eligibility API Integration', () => {
  let authToken: string;
  
  beforeAll(async () => {
    // Setup test database
    await setupTestDatabase();
    // Create test user and get token
    authToken = await createTestUser();
  });
  
  afterAll(async () => {
    await cleanupTestDatabase();
  });
  
  it('should return eligible schemes for user profile', async () => {
    const profile = {
      age: 25,
      annualIncome: 200000,
      state: 'Maharashtra',
      category: 'SC',
      gender: 'Female',
      locationType: 'Rural'
    };
    
    const response = await request(app)
      .post('/api/v1/eligibility/check')
      .set('Authorization', `Bearer ${authToken}`)
      .send({ profile, language: 'en' })
      .expect(200);
    
    expect(response.body.success).toBe(true);
    expect(response.body.data.eligible).toBeInstanceOf(Array);
    expect(response.body.data.ineligible).toBeInstanceOf(Array);
    expect(response.body.data.totalSchemes).toBeGreaterThan(0);
    
    // Verify each eligible scheme has required fields
    response.body.data.eligible.forEach(result => {
      expect(result.scheme).toHaveProperty('name');
      expect(result.scheme).toHaveProperty('description');
      expect(result).toHaveProperty('isEligible', true);
      expect(result).toHaveProperty('relevanceScore');
    });
  });
});
```



### End-to-End Testing

**Framework:** Playwright

**Focus:** Testing complete user journeys through the UI

**Key E2E Tests:**

1. **Citizen Journey**
   - Visit homepage
   - Fill profile form
   - Submit and view results
   - Click on scheme card
   - View scheme details
   - Change language
   - Verify translated content

2. **Admin Journey**
   - Admin login
   - Navigate to scheme management
   - Create new scheme
   - Verify scheme in list
   - Edit scheme
   - Delete scheme

3. **Search and Filter Journey**
   - Enter search query
   - Verify results update
   - Apply filters
   - Verify filtered results
   - Clear filters
   - Verify full list restored

### Test Coverage Goals

- **Unit Tests:** 80% code coverage
- **Property Tests:** All 73 correctness properties
- **Integration Tests:** All API endpoints
- **E2E Tests:** Critical user journeys

### Continuous Integration

**CI Pipeline (GitHub Actions):**
1. Run linting (ESLint)
2. Run unit tests
3. Run property tests
4. Run integration tests
5. Generate coverage report
6. Run E2E tests (on staging)
7. Build and deploy (if all pass)

**Test Execution Time Targets:**
- Unit tests: < 30 seconds
- Property tests: < 2 minutes
- Integration tests: < 1 minute
- E2E tests: < 5 minutes

### Test Data Management

**Strategy:**
- Use factories for generating test data
- Seed test database with representative schemes
- Mock external services (LLM API, email service)
- Clean up test data after each test suite

**Example Test Data Factory:**
```typescript
export const createTestScheme = (overrides?: Partial<Scheme>): Scheme => ({
  id: faker.string.uuid(),
  name: faker.company.catchPhrase(),
  description: faker.lorem.paragraph(),
  benefits: faker.lorem.sentences(2),
  eligibilityCriteria: {
    age: { operator: 'between', value: [18, 60] },
    annualIncome: { operator: 'lte', value: 500000 }
  },
  documents: ['Aadhaar Card', 'Income Certificate'],
  applicationLink: faker.internet.url(),
  isActive: true,
  ...overrides
});
```



## Deployment Architecture

### Infrastructure Components

**Frontend (Vercel):**
- Next.js application with SSR
- Automatic deployments from main branch
- Edge caching for static assets
- Environment variables for API endpoints

**Backend (Render):**
- Node.js/Express API server
- Auto-scaling based on CPU/memory
- Health check endpoint: `/health`
- Environment variables for secrets

**Database (Supabase):**
- PostgreSQL with automatic backups
- Connection pooling via PgBouncer
- Row-level security policies
- Read replicas for scaling

**Cache (Redis Cloud):**
- Managed Redis instance
- Automatic failover
- Persistence enabled

**Monitoring (Sentry + DataDog):**
- Error tracking and alerting
- Performance monitoring
- Log aggregation

### Environment Configuration

**Development:**
```
NODE_ENV=development
DATABASE_URL=postgresql://localhost:5432/yojana_dev
REDIS_URL=redis://localhost:6379
LLM_API_KEY=sk-dev-...
JWT_SECRET=dev-secret-change-in-prod
FRONTEND_URL=http://localhost:3000
```

**Production:**
```
NODE_ENV=production
DATABASE_URL=postgresql://prod-db.supabase.co:5432/yojana
REDIS_URL=redis://prod-cache.redis.cloud:6379
LLM_API_KEY=sk-prod-...
JWT_SECRET=<strong-random-secret>
FRONTEND_URL=https://yojana-ai.vercel.app
SENTRY_DSN=https://...
```

### Deployment Process

1. **Code Push:** Developer pushes to GitHub
2. **CI Pipeline:** GitHub Actions runs tests
3. **Build:** If tests pass, build artifacts
4. **Deploy Backend:** Render deploys API server
5. **Deploy Frontend:** Vercel deploys Next.js app
6. **Smoke Tests:** Run basic health checks
7. **Notify:** Send deployment notification to team

### Rollback Strategy

- Keep last 3 deployments available
- One-click rollback in Vercel/Render dashboards
- Database migrations use up/down scripts
- Feature flags for gradual rollout



## Project Structure

### Backend (Node.js/Express)

```
backend/
├── src/
│   ├── controllers/
│   │   ├── authController.ts
│   │   ├── profileController.ts
│   │   ├── schemeController.ts
│   │   ├── eligibilityController.ts
│   │   └── adminController.ts
│   ├── services/
│   │   ├── eligibilityService.ts
│   │   ├── aiExplainerService.ts
│   │   ├── schemeService.ts
│   │   ├── notificationService.ts
│   │   ├── cacheService.ts
│   │   └── authService.ts
│   ├── middleware/
│   │   ├── authMiddleware.ts
│   │   ├── rateLimitMiddleware.ts
│   │   ├── validationMiddleware.ts
│   │   ├── errorHandler.ts
│   │   └── logger.ts
│   ├── models/
│   │   ├── User.ts
│   │   ├── UserProfile.ts
│   │   ├── Scheme.ts
│   │   └── AuditLog.ts
│   ├── routes/
│   │   ├── authRoutes.ts
│   │   ├── profileRoutes.ts
│   │   ├── schemeRoutes.ts
│   │   ├── eligibilityRoutes.ts
│   │   └── adminRoutes.ts
│   ├── utils/
│   │   ├── validators.ts
│   │   ├── criteriaEvaluator.ts
│   │   ├── relevanceScorer.ts
│   │   └── helpers.ts
│   ├── config/
│   │   ├── database.ts
│   │   ├── redis.ts
│   │   └── constants.ts
│   ├── types/
│   │   ├── index.ts
│   │   ├── scheme.types.ts
│   │   └── eligibility.types.ts
│   └── app.ts
├── tests/
│   ├── unit/
│   │   ├── services/
│   │   ├── controllers/
│   │   └── utils/
│   ├── integration/
│   │   └── api/
│   ├── property/
│   │   └── eligibility.property.test.ts
│   └── fixtures/
│       └── testData.ts
├── migrations/
│   ├── 001_create_users.sql
│   ├── 002_create_schemes.sql
│   └── 003_create_audit_logs.sql
├── package.json
├── tsconfig.json
└── .env.example
```

### Frontend (Next.js)

```
frontend/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── profile/
│   │   │   └── page.tsx
│   │   ├── schemes/
│   │   │   ├── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   └── admin/
│   │       ├── layout.tsx
│   │       ├── page.tsx
│   │       └── schemes/
│   │           └── page.tsx
│   ├── components/
│   │   ├── ProfileForm.tsx
│   │   ├── SchemeCard.tsx
│   │   ├── SchemeList.tsx
│   │   ├── SearchAndFilter.tsx
│   │   ├── LanguageSelector.tsx
│   │   ├── AdminSchemeForm.tsx
│   │   └── ui/
│   │       ├── Button.tsx
│   │       ├── Input.tsx
│   │       └── Select.tsx
│   ├── lib/
│   │   ├── api.ts
│   │   ├── auth.ts
│   │   └── i18n.ts
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useProfile.ts
│   │   └── useSchemes.ts
│   ├── types/
│   │   └── index.ts
│   └── locales/
│       ├── en.json
│       └── hi.json
├── public/
│   ├── images/
│   └── icons/
├── tests/
│   └── e2e/
│       ├── citizen-journey.spec.ts
│       └── admin-journey.spec.ts
├── package.json
├── next.config.js
├── tailwind.config.js
└── tsconfig.json
```



## Security Considerations

### Authentication Security

1. **Password Security**
   - Bcrypt hashing with 12 rounds
   - Minimum 8 characters with complexity requirements
   - Password reset tokens expire in 1 hour
   - Rate limiting on login attempts

2. **Session Management**
   - JWT tokens with 24-hour expiration
   - Refresh tokens with 7-day expiration
   - Token revocation on logout
   - Secure, HttpOnly cookies for token storage

3. **Multi-Factor Authentication**
   - Required for admin accounts
   - TOTP-based (Google Authenticator compatible)
   - Backup codes provided

### API Security

1. **Input Validation**
   - Validate all inputs against schemas
   - Sanitize HTML and SQL inputs
   - Reject oversized payloads (max 1MB)

2. **Rate Limiting**
   - 100 requests/minute per API key
   - 5 login attempts per 15 minutes per IP
   - Exponential backoff for repeated violations

3. **CORS Configuration**
   - Whitelist frontend domain only
   - No wildcard origins in production
   - Credentials allowed for authenticated requests

### Data Security

1. **Encryption**
   - TLS 1.3 for all connections
   - AES-256 for data at rest
   - Encrypted database backups

2. **Access Control**
   - Role-based access control (RBAC)
   - Principle of least privilege
   - Row-level security in database

3. **Data Privacy**
   - No PII in logs or analytics
   - User data deletion on request
   - GDPR compliance considerations

### Infrastructure Security

1. **Network Security**
   - Private subnets for database
   - Security groups restricting access
   - DDoS protection via Cloudflare

2. **Secrets Management**
   - Environment variables for secrets
   - No secrets in code or version control
   - Rotate secrets quarterly

3. **Monitoring and Alerting**
   - Real-time security event monitoring
   - Alerts for suspicious activity
   - Regular security audits



## Performance Optimization

### Database Optimization

1. **Indexing Strategy**
   - B-tree indexes on foreign keys
   - GIN indexes for JSONB eligibility criteria
   - Full-text search indexes on scheme content
   - Composite indexes for common queries

2. **Query Optimization**
   - Use prepared statements
   - Batch operations where possible
   - Limit result sets with pagination
   - Avoid N+1 queries with eager loading

3. **Connection Pooling**
   - Min 10, max 100 connections
   - Connection timeout: 30 seconds
   - Idle connection timeout: 10 minutes

### Caching Strategy

1. **Cache Layers**
   - Redis for application cache
   - CDN for static assets
   - Browser cache for UI resources

2. **Cache Keys and TTL**
   - Schemes list: 1 hour
   - Single scheme: 1 hour
   - Eligibility results: 5 minutes
   - Search results: 15 minutes

3. **Cache Invalidation**
   - Invalidate on scheme updates
   - Pattern-based invalidation
   - Lazy loading for cache misses

### API Optimization

1. **Response Compression**
   - Gzip compression for responses > 1KB
   - Brotli for static assets

2. **Payload Optimization**
   - Return only requested fields
   - Paginate large result sets
   - Use ETags for conditional requests

3. **Async Processing**
   - Queue email notifications
   - Background jobs for bulk imports
   - Non-blocking AI explanation generation

### Frontend Optimization

1. **Code Splitting**
   - Route-based code splitting
   - Lazy load admin components
   - Dynamic imports for heavy libraries

2. **Asset Optimization**
   - Image optimization with Next.js Image
   - SVG for icons
   - Font subsetting

3. **Rendering Strategy**
   - SSR for SEO-critical pages
   - ISR for scheme listings
   - CSR for interactive components



## Monitoring and Observability

### Metrics to Track

1. **Application Metrics**
   - API response times (p50, p95, p99)
   - Error rates by endpoint
   - Request throughput
   - Active users

2. **Business Metrics**
   - Eligibility checks per day
   - Most viewed schemes
   - Search queries
   - User registrations
   - Scheme applications (if tracked)

3. **Infrastructure Metrics**
   - CPU and memory utilization
   - Database connection pool usage
   - Cache hit/miss rates
   - Disk I/O

### Logging Strategy

**Log Aggregation:** DataDog or CloudWatch

**Log Levels:**
- ERROR: System failures, unhandled exceptions
- WARN: Degraded performance, rate limits
- INFO: Business events, audit logs
- DEBUG: Detailed debugging (dev only)

**Structured Logging:**
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "service": "eligibility-api",
  "requestId": "req_abc123",
  "userId": "user_xyz789",
  "event": "eligibility_check",
  "duration": 245,
  "eligibleCount": 12,
  "ineligibleCount": 88
}
```

### Alerting Rules

1. **Critical Alerts (PagerDuty)**
   - API error rate > 5% for 5 minutes
   - Database connection failures
   - Service downtime
   - Security incidents

2. **Warning Alerts (Slack)**
   - API response time p95 > 3 seconds
   - Cache hit rate < 70%
   - Disk usage > 80%
   - Failed background jobs

3. **Info Alerts (Email)**
   - Daily summary report
   - Weekly usage statistics
   - Monthly cost report

### Health Checks

**Endpoint:** `GET /health`

**Response:**
```json
{
  "status": "healthy",
  "timestamp": "2024-01-15T10:30:00Z",
  "services": {
    "database": "healthy",
    "redis": "healthy",
    "llm_api": "healthy"
  },
  "version": "1.2.3"
}
```

**Checks:**
- Database connectivity
- Redis connectivity
- LLM API availability
- Disk space availability



## Scalability Considerations

### Horizontal Scaling

1. **Stateless API Design**
   - No server-side session storage
   - JWT tokens for authentication
   - All state in database or cache

2. **Load Balancing**
   - Round-robin distribution
   - Health check-based routing
   - Session affinity not required

3. **Auto-Scaling Rules**
   - Scale up: CPU > 70% for 5 minutes
   - Scale down: CPU < 30% for 10 minutes
   - Min 2 instances, max 10 instances

### Database Scaling

1. **Read Replicas**
   - Route read queries to replicas
   - Write queries to primary
   - Eventual consistency acceptable

2. **Partitioning Strategy**
   - Partition schemes by state (future)
   - Partition audit logs by date
   - Archive old data quarterly

3. **Connection Pooling**
   - PgBouncer for connection management
   - Separate pools for read/write
   - Monitor pool saturation

### Caching for Scale

1. **Multi-Level Caching**
   - L1: In-memory application cache
   - L2: Redis distributed cache
   - L3: CDN for static content

2. **Cache Warming**
   - Pre-populate popular schemes
   - Warm cache after deployments
   - Background refresh for expiring keys

### Cost Optimization

1. **Resource Right-Sizing**
   - Monitor actual usage
   - Adjust instance sizes quarterly
   - Use spot instances for non-critical jobs

2. **Data Transfer Optimization**
   - Compress responses
   - Use CDN for static assets
   - Optimize database queries

3. **LLM API Cost Management**
   - Cache AI explanations
   - Use cheaper models for simple cases
   - Batch requests where possible
   - Set monthly budget alerts



## Future Enhancements

### Phase 2 Features

1. **Additional Languages**
   - Tamil, Telugu, Bengali, Marathi
   - Regional language support
   - Automatic translation pipeline

2. **Mobile Applications**
   - Native iOS and Android apps
   - Offline mode for rural areas
   - Push notifications

3. **Advanced Matching**
   - ML-based scheme recommendations
   - Personalized scheme ranking
   - Similar scheme suggestions

4. **Document Upload**
   - Upload and verify documents
   - OCR for automatic data extraction
   - Document status tracking

5. **Application Tracking**
   - Track application status
   - Integration with government portals
   - Status notifications

### Phase 3 Features

1. **Chatbot Interface**
   - Conversational eligibility checking
   - Voice input support
   - WhatsApp integration

2. **Community Features**
   - User reviews and ratings
   - Success stories
   - Q&A forum

3. **Analytics Dashboard**
   - Government insights
   - Scheme effectiveness metrics
   - Geographic distribution

4. **API Marketplace**
   - Public API for third-party integrations
   - Developer portal
   - API usage analytics

## Assumptions and Constraints

### Assumptions

1. **Data Availability**
   - Scheme data will be provided by government sources
   - Data format will be standardized
   - Updates will be periodic (weekly/monthly)

2. **User Behavior**
   - Users will provide accurate profile information
   - Most users will access via mobile devices
   - Peak usage during business hours

3. **Infrastructure**
   - Cloud services will maintain 99.9% uptime
   - LLM API will remain available and affordable
   - Database can scale to 10M+ users

### Constraints

1. **Technical Constraints**
   - Must support browsers from last 2 years
   - Must work on 3G networks
   - Must handle 10K concurrent users

2. **Budget Constraints**
   - Infrastructure cost < $500/month initially
   - LLM API cost < $200/month
   - Total operational cost < $1000/month

3. **Timeline Constraints**
   - MVP delivery in 8-10 weeks
   - Phase 2 features in 6 months
   - Full production readiness in 3 months

4. **Regulatory Constraints**
   - Must comply with Indian data protection laws
   - Must not store sensitive documents long-term
   - Must provide data deletion on request

## Success Metrics

### User Metrics

- **User Registrations:** 10,000 in first 3 months
- **Active Users:** 5,000 monthly active users
- **Engagement:** Average 3 scheme views per session
- **Retention:** 40% monthly retention rate

### Performance Metrics

- **API Response Time:** < 2 seconds (p95)
- **Uptime:** 99.5% availability
- **Error Rate:** < 1% of requests
- **Cache Hit Rate:** > 80%

### Business Metrics

- **Scheme Applications:** Track referrals to government portals
- **User Satisfaction:** > 4.0/5.0 rating
- **Coverage:** Support for 500+ schemes at launch
- **Languages:** Hindi and English at launch, 5+ by end of year

## Risk Analysis

### Technical Risks

1. **LLM API Reliability**
   - Risk: API downtime or rate limits
   - Mitigation: Fallback to cached explanations, multiple provider support

2. **Database Performance**
   - Risk: Slow queries at scale
   - Mitigation: Aggressive indexing, caching, read replicas

3. **Security Breaches**
   - Risk: Unauthorized access to user data
   - Mitigation: Defense-in-depth, regular audits, encryption

### Business Risks

1. **Data Quality**
   - Risk: Inaccurate or outdated scheme information
   - Mitigation: Regular data validation, user feedback mechanism

2. **User Adoption**
   - Risk: Low user engagement
   - Mitigation: Marketing campaigns, partnerships with NGOs

3. **Regulatory Changes**
   - Risk: New data protection regulations
   - Mitigation: Flexible architecture, legal consultation

### Operational Risks

1. **Cost Overruns**
   - Risk: Higher than expected infrastructure costs
   - Mitigation: Cost monitoring, optimization, budget alerts

2. **Team Capacity**
   - Risk: Insufficient development resources
   - Mitigation: Phased rollout, prioritization, outsourcing

## Conclusion

YojanaAI provides a comprehensive, scalable solution for helping Indian citizens discover government welfare schemes. The design prioritizes user experience, performance, security, and maintainability while remaining cost-effective and technically feasible. The modular architecture allows for incremental development and future enhancements while the dual testing approach ensures correctness and reliability.
