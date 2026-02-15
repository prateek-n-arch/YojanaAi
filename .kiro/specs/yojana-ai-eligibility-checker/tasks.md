# Implementation Tasks: YojanaAI MVP

## Phase 1: Project Setup and Foundation

### 1. Project Initialization
- [ ] 1.1 Create Next.js project with TypeScript and Tailwind CSS
- [ ] 1.2 Set up project structure (components, services, types, api routes)
- [ ] 1.3 Configure environment variables (.env.local)
- [ ] 1.4 Install required dependencies (zod for validation, optional: supabase client)
- [ ] 1.5 Set up Git repository and initial commit

### 2. Data Preparation
- [ ] 2.1 Create schemes.json file with 50-100 representative schemes
- [ ] 2.2 Include bilingual content (English + Hindi) for each scheme
- [ ] 2.3 Define eligibility criteria in standardized format
- [ ] 2.4 Add documents and application links
- [ ] 2.5 Validate JSON structure

## Phase 2: Core Backend Logic

### 3. Eligibility Engine
- [ ] 3.1 Create EligibilityService class
- [ ] 3.2 Implement evaluateSingleScheme method with criterion checking
- [ ] 3.3 Implement evaluateEligibility method for all schemes
- [ ] 3.4 Implement relevance scoring logic
- [ ] 3.5 Add unit tests for eligibility matching

### 4. AI Explainer Service
- [ ] 4.1 Create AIExplainerService class
- [ ] 4.2 Implement LLM API integration (OpenRouter/OpenAI)
- [ ] 4.3 Build prompt generation logic
- [ ] 4.4 Implement fallback template-based explanations
- [ ] 4.5 Add error handling and timeout logic

### 5. Scheme Service
- [ ] 5.1 Create SchemeService class
- [ ] 5.2 Implement loadSchemes method (from JSON or database)
- [ ] 5.3 Implement getAllSchemes method
- [ ] 5.4 Implement searchSchemes method
- [ ] 5.5 Implement filterSchemes method

## Phase 3: API Routes

### 6. Eligibility Check API
- [ ] 6.1 Create /api/eligibility/check route
- [ ] 6.2 Implement request validation using Zod
- [ ] 6.3 Integrate EligibilityService
- [ ] 6.4 Format response with eligible/ineligible lists
- [ ] 6.5 Add error handling

### 7. Schemes API
- [ ] 7.1 Create /api/schemes route
- [ ] 7.2 Implement search functionality
- [ ] 7.3 Implement filter functionality
- [ ] 7.4 Support language parameter
- [ ] 7.5 Add error handling

### 8. Explanation API
- [ ] 8.1 Create /api/explain route
- [ ] 8.2 Integrate AIExplainerService
- [ ] 8.3 Support bilingual explanations
- [ ] 8.4 Add caching for repeated requests (optional)
- [ ] 8.5 Add error handling

## Phase 4: Frontend Components

### 9. Profile Form Component
- [ ] 9.1 Create ProfileForm component
- [ ] 9.2 Add form fields (age, income, state, occupation, category, gender, location)
- [ ] 9.3 Implement client-side validation
- [ ] 9.4 Add bilingual labels
- [ ] 9.5 Style with Tailwind CSS (mobile-responsive)

### 10. Results Display Components
- [ ] 10.1 Create SchemeCard component
- [ ] 10.2 Create SchemeList component
- [ ] 10.3 Implement eligible/ineligible sections
- [ ] 10.4 Add bilingual content display
- [ ] 10.5 Style with Tailwind CSS

### 11. Language Selector
- [ ] 11.1 Create LanguageSelector component
- [ ] 11.2 Implement language state management
- [ ] 11.3 Add language toggle functionality
- [ ] 11.4 Style with Tailwind CSS

### 12. Search and Filter Components
- [ ] 12.1 Create SearchBar component
- [ ] 12.2 Create FilterPanel component
- [ ] 12.3 Implement debounced search
- [ ] 12.4 Add filter options (state, category, eligibility status)
- [ ] 12.5 Style with Tailwind CSS

## Phase 5: Pages and User Flow

### 13. Home Page
- [ ] 13.1 Create home page (app/page.tsx)
- [ ] 13.2 Add ProfileForm component
- [ ] 13.3 Implement form submission logic
- [ ] 13.4 Add loading state
- [ ] 13.5 Add bilingual welcome message

### 14. Results Page
- [ ] 14.1 Create results page (app/results/page.tsx)
- [ ] 14.2 Display eligible schemes
- [ ] 14.3 Display ineligible schemes
- [ ] 14.4 Add search and filter functionality
- [ ] 14.5 Implement "View Explanation" functionality

### 15. Scheme Details Page
- [ ] 15.1 Create scheme details page (app/scheme/[id]/page.tsx)
- [ ] 15.2 Display full scheme information
- [ ] 15.3 Show required documents
- [ ] 15.4 Add application link button
- [ ] 15.5 Display AI-generated explanation

## Phase 6: Polish and Optimization

### 16. UI/UX Enhancements
- [ ] 16.1 Add loading spinners and skeletons
- [ ] 16.2 Implement error messages and empty states
- [ ] 16.3 Add success animations
- [ ] 16.4 Ensure mobile responsiveness
- [ ] 16.5 Test on multiple devices and browsers

### 17. Performance Optimization
- [ ] 17.1 Optimize images with Next.js Image component
- [ ] 17.2 Implement code splitting
- [ ] 17.3 Add caching headers
- [ ] 17.4 Minimize bundle size
- [ ] 17.5 Test performance on 3G network

### 18. Bilingual Content
- [ ] 18.1 Create translation files (locales/en.json, locales/hi.json)
- [ ] 18.2 Translate all UI labels and messages
- [ ] 18.3 Ensure proper Hindi font rendering
- [ ] 18.4 Test language switching
- [ ] 18.5 Verify all content displays correctly in both languages

## Phase 7: Testing and Deployment

### 19. Testing
- [ ] 19.1 Manual testing of all user flows
- [ ] 19.2 Test eligibility matching with various profiles
- [ ] 19.3 Test AI explanations in both languages
- [ ] 19.4 Test on mobile devices
- [ ] 19.5 Test error scenarios

### 20. Deployment
- [ ] 20.1 Create Vercel account and connect GitHub repo
- [ ] 20.2 Configure environment variables in Vercel
- [ ] 20.3 Deploy to production
- [ ] 20.4 Test production deployment
- [ ] 20.5 Set up custom domain (optional)

## Phase 8: Demo Preparation

### 21. Demo Content
- [ ] 21.1 Prepare demo script
- [ ] 21.2 Create sample user profiles for demo
- [ ] 21.3 Prepare talking points (social impact, AI innovation, bilingual support)
- [ ] 21.4 Create demo video (optional)
- [ ] 21.5 Prepare presentation slides (optional)

### 22. Documentation
- [ ] 22.1 Write README.md with project overview
- [ ] 22.2 Document API endpoints
- [ ] 22.3 Add setup instructions
- [ ] 22.4 Document environment variables
- [ ] 22.5 Add screenshots to README

## Optional Enhancements (If Time Permits)

### 23. Advanced Features
- [ ]* 23.1 Add scheme comparison feature
- [ ]* 23.2 Implement "Save Profile" to localStorage
- [ ]* 23.3 Add print/download results functionality
- [ ]* 23.4 Implement scheme bookmarking
- [ ]* 23.5 Add social sharing buttons

### 24. Analytics (Basic)
- [ ]* 24.1 Add basic usage tracking (page views)
- [ ]* 24.2 Track popular schemes
- [ ]* 24.3 Track language preference distribution
- [ ]* 24.4 Create simple analytics dashboard
- [ ]* 24.5 Export analytics data

## Notes

- Tasks marked with `*` are optional and should only be attempted if core features are complete
- Prioritize core user flow: Profile Form → Eligibility Check → Results Display → Scheme Details
- Focus on demo-ready features over production-grade infrastructure
- Keep code simple and maintainable
- Test frequently on mobile devices
- Ensure bilingual support works perfectly (key differentiator)
