# Design Document: YojanaAI - Government Scheme Eligibility Checker (MVP)

## Overview

YojanaAI is a hackathon MVP demonstrating an AI-powered platform that intelligently matches Indian citizens with government welfare schemes. The system uses a rule-based eligibility engine combined with AI-powered natural language explanations to provide personalized guidance in Hindi and English.

### Architecture Philosophy

The design follows a simplified full-stack architecture optimized for rapid development by a solo developer:
- **Frontend**: Next.js with server-side rendering and API routes (eliminates separate backend)
- **Core Logic**: Eligibility engine and AI explainer as server-side services
- **Data Source**: PostgreSQL via Supabase (or JSON file for ultra-fast MVP)
- **AI Layer**: LLM API (OpenRouter/OpenAI) for explanations

The system prioritizes:
- **Simplicity**: Minimal architecture, maximum functionality
- **Speed**: Fast development and fast user experience
- **Demonstrability**: Clear value proposition for hackathon judges
- **Scalability Potential**: Clean design that can grow post-MVP

## System Architecture

### Simplified Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Client (Browser)                          │
│  - React Components                                          │
│  - Tailwind CSS                                             │
│  - Language Switcher (Hindi/English)                        │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ HTTPS
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Next.js Application (Vercel)                    │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Frontend Pages                                        │  │
│  │  - Home (Profile Form)                                │  │
│  │  - Results (Eligible/Ineligible Schemes)              │  │
│  │  - Scheme Details                                     │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  API Routes (/api/*)                                  │  │
│  │  - POST /api/eligibility/check                        │  │
│  │  - GET /api/schemes                                   │  │
│  │  - POST /api/explain                                  │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Server-Side Services                                 │  │
│  │  - EligibilityService (matching logic)                │  │
│  │  - AIExplainerService (LLM integration)               │  │
│  │  - SchemeService (data access)                        │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                │                       │
                ▼                       ▼
        ┌──────────────────┐    ┌──────────────────┐
        │   Supabase       │    │   LLM API        │
        │   PostgreSQL     │    │  (OpenRouter/    │
        │                  │    │   OpenAI)        │
        │  - schemes table │    │                  │
        │  (or JSON file)  │    │  - Explanations  │
        └──────────────────┘    └──────────────────┘
```

### Technology Stack Rationale

**Frontend + Backend: Next.js**
- Single codebase for frontend and API
- Built-in API routes eliminate separate backend server
- Server-side rendering for SEO
- Vercel deployment with zero configuration
- Fast development for solo developer

**Database: Supabase PostgreSQL (or JSON)**
- Option 1: Supabase for production-like demo
- Option 2: JSON file for ultra-fast MVP (no database setup)
- Easy migration path from JSON to database

**AI Layer: OpenRouter/OpenAI**
- Simple REST API integration
- Reliable and fast responses
- Cost-effective for hackathon demo

**Styling: Tailwind CSS**
- Rapid UI development
- Responsive by default
- No custom CSS needed

## Core Components

### 1. EligibilityService

The heart of the system - matches user profiles against scheme criteria.

```typescript
interface UserProfile {
  age: number;
  annualIncome: number;
  state: string;
  occupation: string;
  category: 'SC' | 'ST' | 'OBC' | 'General' | 'EWS';
  gender: 'Male' | 'Female' | 'Other';
  locationType: 'Rural' | 'Urban';
}

interface Scheme {
  id: string;
  name: string;
  nameHi: string;
  description: string;
  descriptionHi: string;
  benefits: string;
  benefitsHi: string;
  eligibilityCriteria: {
    age?: { operator: 'between' | 'gte' | 'lte'; value: number | [number, number] };
    annualIncome?: { operator: 'lte' | 'gte'; value: number };
    state?: { operator: 'in'; value: string[] };
    category?: { operator: 'in'; value: string[] };
    gender?: { operator: 'eq'; value: string };
    locationType?: { operator: 'eq'; value: string };
  };
  documents: string[];
  applicationLink?: string;
}

interface EligibilityResult {
  scheme: Scheme;
  isEligible: boolean;
  matchedCriteria: string[];
  unmatchedCriteria: string[];
  relevanceScore: number;
}

class EligibilityService {
  evaluateEligibility(profile: UserProfile, schemes: Scheme[]): EligibilityResult[] {
    return schemes.map(scheme => this.evaluateSingleScheme(profile, scheme))
      .sort((a, b) => b.relevanceScore - a.relevanceScore);
  }

  private evaluateSingleScheme(profile: UserProfile, scheme: Scheme): EligibilityResult {
    const matched: string[] = [];
    const unmatched: string[] = [];
    
    // Check each criterion
    Object.entries(scheme.eligibilityCriteria).forEach(([field, criterion]) => {
      if (this.checkCriterion(profile[field], criterion)) {
        matched.push(field);
      } else {
        unmatched.push(field);
      }
    });
    
    return {
      scheme,
      isEligible: unmatched.length === 0,
      matchedCriteria: matched,
      unmatchedCriteria: unmatched,
      relevanceScore: matched.length
    };
  }

  private checkCriterion(userValue: any, criterion: any): boolean {
    switch (criterion.operator) {
      case 'eq': return userValue === criterion.value;
      case 'gte': return userValue >= criterion.value;
      case 'lte': return userValue <= criterion.value;
      case 'in': return criterion.value.includes(userValue);
      case 'between': 
        return userValue >= criterion.value[0] && userValue <= criterion.value[1];
      default: return false;
    }
  }
}
```

### 2. AIExplainerService

Generates natural language explanations using LLM API.

```typescript
class AIExplainerService {
  async generateExplanation(
    profile: UserProfile,
    result: EligibilityResult,
    language: 'en' | 'hi'
  ): Promise<string> {
    const prompt = this.buildPrompt(profile, result, language);
    
    try {
      const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${process.env.LLM_API_KEY}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          model: 'meta-llama/llama-3.1-8b-instruct:free',
          messages: [{ role: 'user', content: prompt }],
          temperature: 0.3
        })
      });
      
      const data = await response.json();
      return data.choices[0].message.content;
    } catch (error) {
      // Fallback to template-based explanation
      return this.generateFallbackExplanation(profile, result, language);
    }
  }

  private buildPrompt(profile: UserProfile, result: EligibilityResult, language: string): string {
    const lang = language === 'hi' ? 'Hindi' : 'English';
    const status = result.isEligible ? 'eligible' : 'not eligible';
    
    return `You are explaining government scheme eligibility to an Indian citizen.
    
User Profile:
- Age: ${profile.age}
- Annual Income: ₹${profile.annualIncome}
- State: ${profile.state}
- Category: ${profile.category}
- Gender: ${profile.gender}
- Location: ${profile.locationType}

Scheme: ${result.scheme.name}
Status: User is ${status}
Matched Criteria: ${result.matchedCriteria.join(', ') || 'None'}
Unmatched Criteria: ${result.unmatchedCriteria.join(', ') || 'None'}

Explain in ${lang} why the user ${status} for this scheme. Use simple language suitable for varying literacy levels. Keep it under 100 words.`;
  }

  private generateFallbackExplanation(
    profile: UserProfile,
    result: EligibilityResult,
    language: string
  ): string {
    // Simple template-based fallback
    if (result.isEligible) {
      return language === 'hi' 
        ? `आप इस योजना के लिए पात्र हैं क्योंकि आप सभी आवश्यक शर्तों को पूरा करते हैं।`
        : `You are eligible for this scheme because you meet all the required criteria.`;
    } else {
      return language === 'hi'
        ? `आप इस योजना के लिए पात्र नहीं हैं क्योंकि कुछ शर्तें पूरी नहीं होती हैं।`
        : `You are not eligible for this scheme because some criteria are not met.`;
    }
  }
}
```

### 3. SchemeService

Handles scheme data access (from database or JSON file).

```typescript
class SchemeService {
  private schemes: Scheme[] = [];

  async loadSchemes(): Promise<void> {
    // Option 1: Load from Supabase
    if (process.env.USE_DATABASE === 'true') {
      const { data } = await supabase.from('schemes').select('*');
      this.schemes = data || [];
    } else {
      // Option 2: Load from JSON file (faster for MVP)
      const fs = require('fs');
      const data = fs.readFileSync('./data/schemes.json', 'utf8');
      this.schemes = JSON.parse(data);
    }
  }

  getAllSchemes(): Scheme[] {
    return this.schemes;
  }

  getSchemeById(id: string): Scheme | undefined {
    return this.schemes.find(s => s.id === id);
  }

  searchSchemes(query: string, language: 'en' | 'hi'): Scheme[] {
    const lowerQuery = query.toLowerCase();
    return this.schemes.filter(scheme => {
      const name = language === 'hi' ? scheme.nameHi : scheme.name;
      const desc = language === 'hi' ? scheme.descriptionHi : scheme.description;
      return name.toLowerCase().includes(lowerQuery) || 
             desc.toLowerCase().includes(lowerQuery);
    });
  }

  filterSchemes(filters: {
    state?: string;
    category?: string;
    eligibilityStatus?: 'eligible' | 'ineligible';
  }): Scheme[] {
    return this.schemes.filter(scheme => {
      if (filters.state && scheme.eligibilityCriteria.state) {
        if (!scheme.eligibilityCriteria.state.value.includes(filters.state)) {
          return false;
        }
      }
      if (filters.category && scheme.eligibilityCriteria.category) {
        if (!scheme.eligibilityCriteria.category.value.includes(filters.category)) {
          return false;
        }
      }
      return true;
    });
  }
}
```

## API Endpoints

### POST /api/eligibility/check

Check eligibility for all schemes based on user profile.

**Request:**
```json
{
  "profile": {
    "age": 25,
    "annualIncome": 200000,
    "state": "Maharashtra",
    "occupation": "Student",
    "category": "SC",
    "gender": "Female",
    "locationType": "Rural"
  },
  "language": "en"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "eligible": [
      {
        "scheme": { /* scheme object */ },
        "isEligible": true,
        "matchedCriteria": ["age", "category", "gender"],
        "unmatchedCriteria": [],
        "relevanceScore": 3
      }
    ],
    "ineligible": [
      {
        "scheme": { /* scheme object */ },
        "isEligible": false,
        "matchedCriteria": ["age"],
        "unmatchedCriteria": ["annualIncome", "category"],
        "relevanceScore": 1
      }
    ],
    "totalSchemes": 50
  }
}
```

### GET /api/schemes

Get all schemes or search/filter schemes.

**Query Parameters:**
- `search`: Search query string
- `state`: Filter by state
- `category`: Filter by category
- `language`: Response language (en/hi)

**Response:**
```json
{
  "success": true,
  "data": {
    "schemes": [ /* array of scheme objects */ ],
    "total": 50
  }
}
```

### POST /api/explain

Generate AI explanation for a specific scheme and user profile.

**Request:**
```json
{
  "profile": { /* user profile */ },
  "schemeId": "scheme-123",
  "language": "hi"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "explanation": "आप इस योजना के लिए पात्र हैं...",
    "isEligible": true
  }
}
```

## Data Model

### Scheme Data Structure (JSON/Database)

```json
{
  "id": "pm-kisan",
  "name": "PM-KISAN Scheme",
  "nameHi": "पीएम-किसान योजना",
  "description": "Income support to farmer families",
  "descriptionHi": "किसान परिवारों को आय सहायता",
  "benefits": "₹6000 per year in three installments",
  "benefitsHi": "तीन किस्तों में प्रति वर्ष ₹6000",
  "eligibilityCriteria": {
    "occupation": { "operator": "eq", "value": "Farmer" },
    "locationType": { "operator": "eq", "value": "Rural" },
    "annualIncome": { "operator": "lte", "value": 200000 }
  },
  "documents": [
    "Aadhaar Card",
    "Bank Account Details",
    "Land Ownership Documents"
  ],
  "applicationLink": "https://pmkisan.gov.in/"
}
```

### Database Schema (if using Supabase)

```sql
CREATE TABLE schemes (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  name_hi TEXT,
  description TEXT NOT NULL,
  description_hi TEXT,
  benefits TEXT NOT NULL,
  benefits_hi TEXT,
  eligibility_criteria JSONB NOT NULL,
  documents JSONB NOT NULL,
  application_link TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Index for faster queries
CREATE INDEX idx_schemes_criteria ON schemes USING GIN(eligibility_criteria);
```

## Frontend Components

### 1. ProfileForm Component

```typescript
export default function ProfileForm({ onSubmit }: { onSubmit: (profile: UserProfile) => void }) {
  const [profile, setProfile] = useState<UserProfile>({
    age: 0,
    annualIncome: 0,
    state: '',
    occupation: '',
    category: 'General',
    gender: 'Male',
    locationType: 'Urban'
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(profile);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input 
        type="number" 
        placeholder="Age"
        value={profile.age}
        onChange={(e) => setProfile({...profile, age: parseInt(e.target.value)})}
        className="w-full p-2 border rounded"
      />
      {/* More form fields... */}
      <button type="submit" className="w-full bg-blue-600 text-white p-2 rounded">
        Check Eligibility
      </button>
    </form>
  );
}
```

### 2. SchemeCard Component

```typescript
export default function SchemeCard({ 
  result, 
  language,
  onExplain 
}: { 
  result: EligibilityResult;
  language: 'en' | 'hi';
  onExplain: () => void;
}) {
  const scheme = result.scheme;
  const name = language === 'hi' ? scheme.nameHi : scheme.name;
  const desc = language === 'hi' ? scheme.descriptionHi : scheme.description;

  return (
    <div className={`p-4 border rounded ${result.isEligible ? 'border-green-500' : 'border-red-500'}`}>
      <h3 className="text-xl font-bold">{name}</h3>
      <p className="text-gray-600">{desc}</p>
      <div className="mt-2">
        <span className={`px-2 py-1 rounded text-sm ${result.isEligible ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800'}`}>
          {result.isEligible ? (language === 'hi' ? 'पात्र' : 'Eligible') : (language === 'hi' ? 'अपात्र' : 'Not Eligible')}
        </span>
      </div>
      <button onClick={onExplain} className="mt-2 text-blue-600 underline">
        {language === 'hi' ? 'विवरण देखें' : 'View Explanation'}
      </button>
    </div>
  );
}
```

### 3. LanguageSelector Component

```typescript
export default function LanguageSelector({ 
  language, 
  onChange 
}: { 
  language: 'en' | 'hi';
  onChange: (lang: 'en' | 'hi') => void;
}) {
  return (
    <div className="flex gap-2">
      <button
        onClick={() => onChange('en')}
        className={`px-4 py-2 rounded ${language === 'en' ? 'bg-blue-600 text-white' : 'bg-gray-200'}`}
      >
        English
      </button>
      <button
        onClick={() => onChange('hi')}
        className={`px-4 py-2 rounded ${language === 'hi' ? 'bg-blue-600 text-white' : 'bg-gray-200'}`}
      >
        हिंदी
      </button>
    </div>
  );
}
```

## Project Structure

```
yojana-ai/
├── src/
│   ├── app/
│   │   ├── page.tsx                 # Home page with profile form
│   │   ├── results/
│   │   │   └── page.tsx             # Results page
│   │   ├── scheme/
│   │   │   └── [id]/
│   │   │       └── page.tsx         # Scheme details page
│   │   ├── api/
│   │   │   ├── eligibility/
│   │   │   │   └── check/
│   │   │   │       └── route.ts     # Eligibility check API
│   │   │   ├── schemes/
│   │   │   │   └── route.ts         # Schemes API
│   │   │   └── explain/
│   │   │       └── route.ts         # Explanation API
│   │   └── layout.tsx               # Root layout
│   ├── components/
│   │   ├── ProfileForm.tsx
│   │   ├── SchemeCard.tsx
│   │   ├── SchemeList.tsx
│   │   ├── LanguageSelector.tsx
│   │   └── SearchBar.tsx
│   ├── services/
│   │   ├── eligibilityService.ts
│   │   ├── aiExplainerService.ts
│   │   └── schemeService.ts
│   ├── types/
│   │   └── index.ts
│   └── lib/
│       └── supabase.ts              # Supabase client (optional)
├── data/
│   └── schemes.json                 # Scheme data (if not using database)
├── public/
│   └── images/
├── .env.local
├── package.json
├── next.config.js
├── tailwind.config.js
└── tsconfig.json
```

## Error Handling

### API Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "age": "Age must be between 0 and 120"
    }
  }
}
```

### Error Categories

1. **Validation Errors (400)**
   - Missing required fields
   - Invalid data types
   - Out-of-range values

2. **Not Found (404)**
   - Scheme ID doesn't exist

3. **External Service Errors (503)**
   - LLM API timeout
   - Database connection failure
   - Fallback to template-based responses

4. **Server Errors (500)**
   - Unhandled exceptions
   - Log error and return generic message

## Performance Optimization

### 1. Data Loading
- Load schemes once at startup
- Cache in memory for fast access
- No database queries per request (for JSON approach)

### 2. AI Explanations
- Generate on-demand (not pre-generated)
- Implement timeout (3 seconds)
- Fallback to template if LLM fails

### 3. Frontend Optimization
- Use Next.js Image component
- Lazy load scheme details
- Debounce search input

### 4. Response Times
- Eligibility check: < 500ms (in-memory calculation)
- AI explanation: < 3s (LLM API call)
- Page load: < 2s (SSR + optimized assets)

## Deployment

### Vercel Deployment (Recommended)

1. **Connect GitHub Repository**
   - Push code to GitHub
   - Connect to Vercel
   - Auto-deploy on push

2. **Environment Variables**
   ```
   LLM_API_KEY=your_openrouter_key
   USE_DATABASE=false
   NEXT_PUBLIC_APP_URL=https://your-app.vercel.app
   ```

3. **Build Settings**
   - Framework: Next.js
   - Build Command: `npm run build`
   - Output Directory: `.next`

### Alternative: Local Development

```bash
npm install
npm run dev
# Open http://localhost:3000
```

## Testing Strategy (Simplified for MVP)

### 1. Manual Testing
- Test all user flows manually
- Test on mobile devices
- Test both languages
- Test with various profile combinations

### 2. Basic Unit Tests (Optional)
- Test eligibility matching logic
- Test criterion evaluation
- Use Jest for testing

```typescript
describe('EligibilityService', () => {
  it('should mark user eligible when all criteria match', () => {
    const service = new EligibilityService();
    const profile = { age: 25, annualIncome: 150000, /* ... */ };
    const scheme = { /* ... */ };
    const result = service.evaluateSingleScheme(profile, scheme);
    expect(result.isEligible).toBe(true);
  });
});
```

### 3. Integration Testing
- Test API endpoints with Postman
- Verify response formats
- Test error scenarios

## Security Considerations

### 1. Input Validation
- Validate all user inputs
- Sanitize before processing
- Use Zod for schema validation

```typescript
import { z } from 'zod';

const profileSchema = z.object({
  age: z.number().min(0).max(120),
  annualIncome: z.number().min(0),
  state: z.string().min(1),
  // ... more fields
});
```

### 2. API Security
- Rate limiting (100 requests/minute per IP)
- CORS configuration
- HTTPS only (enforced by Vercel)

### 3. Data Privacy
- No permanent storage of user data
- Session-based only
- No tracking or analytics (for MVP)

## Future Scope (Post-Hackathon Features)

The following features are intentionally excluded from the MVP to maintain focus on core functionality and enable rapid development within hackathon constraints. These will be prioritized for post-demo implementation:

### Authentication & User Management
- User registration and login system
- Profile saving and retrieval across sessions
- Social login (Google OAuth)
- Password reset functionality
- Session management and security

### Admin Panel
- Full CRUD interface for scheme management
- Bulk import/export of scheme data (CSV/JSON)
- Scheme approval workflow
- User analytics dashboard
- Content moderation tools

### Notifications
- Email notifications for new schemes
- SMS alerts for application deadlines
- Push notifications (mobile app)
- Personalized scheme recommendations

### Advanced Analytics
- User behavior tracking and insights
- Scheme effectiveness metrics
- Geographic distribution analysis
- A/B testing framework
- Conversion funnel analysis
- Government insights dashboard

### Infrastructure Enhancements
- Redis caching layer for performance
- Read replicas for database scaling
- CI/CD pipeline automation
- Advanced monitoring and alerting (DataDog/Sentry)
- Auto-scaling infrastructure
- CDN for static assets

### Additional Features
- More regional languages (Tamil, Telugu, Bengali, Marathi)
- Mobile native applications (iOS/Android)
- Offline mode for rural areas
- Document upload and verification
- Application status tracking
- Chatbot interface with voice support
- WhatsApp integration
- Community features (reviews, Q&A)
- Government API integration

## Success Metrics

### Demo Metrics
- 50+ users complete eligibility check
- < 2s average response time
- 30%+ users try Hindi language
- Positive feedback from judges

### Technical Metrics
- 99% uptime during demo
- < 2% error rate
- Works on 3G networks
- Mobile-responsive

## Assumptions and Constraints

### Assumptions
- 50-100 curated schemes sufficient for demo
- Free tier services adequate for hackathon
- Manual data entry acceptable for MVP
- No real-time government API needed

### Constraints
- 5-7 day development timeline
- Solo developer capacity
- $0-50 budget for MVP
- No authentication system
- No admin panel
- Hindi + English only

## Conclusion

YojanaAI MVP design focuses on core functionality: intelligent eligibility matching and AI-powered explanations in bilingual format. The simplified architecture using Next.js for both frontend and backend enables rapid development by a solo developer while maintaining technical credibility and demonstrating clear social impact for hackathon judges.
