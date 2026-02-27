# Initiative: Disclosure Template Enrichment for Smart Position/Report Mapping

**Link to Initiative in ADO: RELATED TO INITIATIVE 1397587 (AI-assisted Corporate Disclosure)**


Please be advised that the grey text is intended to provide guidance while completing this document.
---

## 

---

## Overview


Ready Framework phase focus should be on definition of the agreed solution and the areas
that impact budget, with well-considered budget costs the key output.


> **Guidance:**
> * Please review this entire document and ensure all applicable sections are filled in and 
> treat this as the technical leadership's guidance on solution direction for the squads 
> that plan, construct, and release this project. 
> * It outlines the considerations that formed the technical leadership's Ready Framework 
> engineering staff month estimates & the budget. 
> * This documented thinking can be retrospectively used to learn lessons to help continually improve.

---

## 

---

## Must-Haves


### Proposed Solution Approach and/or Options



> **Guidance:**
> * [MUST] Outline the proposed solution approach that guides the sizing. 
> * List major features to help estimation.



**SOLUTION OVERVIEW:**


This initiative addresses the challenge of manually mapping disclosure template questions
to appropriate Corporate Sustainability (CS) positions and reports. Currently, users must
select from hundreds of potential positions per question, which is time-consuming and error-prone.

The proposed solution enriches disclosure templates (XML/JSON) with metadata attributes
that enable programmatic filtering and scoring of position/report candidates. This reduces
manual selection from 100+ options to 1-3 high-confidence suggestions per question.


**KEY DIFFERENTIATOR FROM AI INITIATIVE 1397587:**

- AI Initiative: Uses LLM with database access for narrative text responses
- This Initiative: Programmatic filtering based on template metadata (no AI needed)
- Complementary: This provides position/report mapping; AI handles text generation


**TECHNICAL APPROACH:**


1. XML Schema Extension
   - Add <mappingHints> container element to <question> and <column> elements
   - Define structured metadata: unitType, positionType, scope, calculation flags
   - Maintain 100% backward compatibility (optional elements)

2. JSON Converter Enhancement
   - Parse enrichment attributes from XML
   - Include in generated disclosure template JSON
   - Implement fallback logic for unenriched templates

3. Manual Enrichment Process
   - Domain expert (Antonia) tags high-priority questions via CSV
   - CSV provides overrides for complex/ambiguous mappings
   - Automated validation of enrichment data quality

4. Position Filtering Service
   - New backend service: PositionMappingService.php
   - Filters positions by: unit type, position type, scope, site assignment
   - Scores by data completeness, geographic coverage, recency
   - Returns top 1-3 candidates with confidence scores

5. UI Integration
   - Display smart suggestions in disclosure response UI
   - Show confidence indicators (high/medium/low)
   - Allow user override with tracking for continuous improvement


**MAJOR FEATURES:**


Feature 1: XML Schema Extension for Enrichment Metadata
- Define <mappingHints>, <positionMapping>, <reportMapping> elements
- Create enumerations: unitType, positionType, scope, periodType, aggregation
- Validate backward compatibility with existing templates
- Update XSD schema documentation

Feature 2: JSON Converter Enhancement
- Parse mappingHints from XML during template import
- Generate enriched JSON with metadata included
- Implement inference rules from XBRL concept names
- Handle missing/partial metadata gracefully

Feature 3: Manual Enrichment Workflow
- CSV schema: question_sid, unit_type, position_type, scope, requires_calculation
- Import mechanism with validation
- Override priority: CSV > XML inline > inferred > default
- Version control for enrichment data

Feature 4: Position/Report Filtering Service
- Backend service class: PositionMappingService
- Filtering algorithm using metadata criteria
- Scoring algorithm based on data quality metrics
- Caching layer for performance optimization

Feature 5: Smart Suggestion UI Components
- Auto-populate position/report selections
- Display confidence scores and match reasoning
- User override tracking for learning
- Bulk assignment mode for entire disclosure

Feature 6: XBRL to CS Unit Type Mapping
- Mapping table: XBRL taxonomy types → CS unit types
- Configuration storage (database or config file)
- Support for custom unit mappings per customer
- Validation of unit compatibility


**SOLUTION OPTIONS CONSIDERED:**


Option A: XML Inline Metadata (RECOMMENDED)
Pros:
- Self-contained, version controlled with template
- No external dependencies
- Easy maintenance
Cons:
- Requires schema extension
- Slightly larger XML files

Option B: External CSV Only
Pros:
- No XML schema changes
- Easy for non-technical users
Cons:
- Sync issues between CSV and templates
- External dependency
- Version control complexity

Option C: Hybrid (SELECTED)
Pros:
- Best of both worlds
- Flexibility for edge cases
- Schema defaults + manual overrides
Cons:
- Slightly more complex implementation
Priority: CSV override > XML inline > inferred > default


**PHASED ROLLOUT:**


Phase 1: Schema & Converter (Weeks 1-2)
- XML schema extension
- JSON converter updates
- Unit tests and validation

Phase 2: Manual Enrichment (Weeks 2-3)
- CSV import mechanism
- Antonia tags ESRS E1 questions
- Data quality validation

Phase 3: Filtering Service (Weeks 3-4)
- PositionMappingService implementation
- Filtering and scoring algorithms
- API endpoints

Phase 4: UI Integration (Weeks 4-5)
- Smart suggestion components
- Confidence display
- User override tracking

Phase 5: Testing & Refinement (Weeks 5-6)
- End-to-end testing with real ESRS disclosure
- Accuracy measurement and tuning
- Performance optimization


**SUCCESS METRICS:**

- Auto-match accuracy: 80%+ (correct top suggestion)
- Position reduction: 100+ → 1-3 candidates
- User override rate: <20%
- Coverage: 100% ESRS E1 Climate Change
- Performance: <2s filtering per question
---

## 

---

## Architecture Review


### C4 Model Context



> **Guidance:**
> * [MUST] Which part of the C4 model (level 2 or 3) is impacted 
> (highlight where work happens OR if changes proposed changes) - tech stack changes/updates? 
> Paste in a C4 annotated.
> * Focus is on budget costs - Are new 3rd party licensing or services likely to be needed 
> for this solution?


Rule:
* Prefer multi-tenant services even when extending single tenant products.

IMPACTED COMPONENTS (C4 Level 2):

1. Corporate Sustainability (CS) - Backend
   Location: sofi/app/src/SoFi/DisclosureManagement/
   Changes:
   - NEW: Service/PositionMappingService.php (position filtering)
   - UPDATE: Service.php (integrate filtering service)
   - UPDATE: Controller.php (new API endpoint for suggestions)
   - NEW: Service/EnrichmentImporter.php (CSV import)

2. Corporate Sustainability (CS) - Frontend
   Location: sofi/app/workspace/packages/sofi/src/module/disclosuremanagement/
   Changes:
   - UPDATE: panel/DetailPage.js (suggestion display)
   - UPDATE: panel/question/QuestionContainer.js (auto-populate)
   - NEW: panel/question/SmartSuggestionMixin.js (suggestion logic)
   - UPDATE: panel/question/position/PositionContainer.js (confidence display)

3. Disclosure Template Import/Export
   Location: sofi/app/src/SoFi/DisclosureManagement/Import/
   Changes:
   - UPDATE: XmlImporter.php (parse mappingHints)
   - UPDATE: JsonConverter.php (include metadata in JSON)
   - NEW: EnrichmentValidator.php (validate enrichment data)

4. Database Schema
   Location: sofi/app/migrations/
   Changes:
   - NEW TABLE: disclosure_enrichment_override (CSV data storage)
     Columns: question_sid, unit_type, position_type, scope, requires_calculation,
              confidence, created_by, created_at, updated_at
   - NEW INDEX: idx_disclosure_position_mapping (position_id, unit_type, type, scope)

5. Configuration
   Location: sofi/app/config/
   Changes:
   - NEW: disclosure_unit_mapping.yml (XBRL → CS unit type mappings)
   - UPDATE: services.yml (register PositionMappingService)


**TECHNOLOGY STACK:**

- Backend: PHP 7.4+, Doctrine ORM
- Frontend: ExtJS 6.5+
- Database: MySQL 5.7+ (existing)
- XML: XSD validation
- No new external services required


**DEPENDENCY MAP:**

```
DisclosureManagement/Controller
    ↓
DisclosureManagement/Service
    ↓
PositionMappingService ← NEW
    ↓
Position Repository (existing)
    ↓
MySQL Database (existing)
```


**3RD PARTY LICENSING/SERVICES:**


**- NONE REQUIRED**

- Uses existing infrastructure
- No AI/LLM costs (unlike Initiative 1397587)
- XBRL taxonomy is open standard (EFRAG)

### Performance Considerations



> **Guidance:**
> * [MUST] Will performance requirements drive additional costs? 
> If so, state performance requirements used in the budget estimate, e.g., 
> report returns inside 20 seconds, page refreshes in 2 seconds, etc.



**PERFORMANCE REQUIREMENTS:**


1. Position Filtering Response Time
   Requirement: <2 seconds per question
   Rationale: User expects immediate suggestions after site assignment

   Optimization Strategy:
   - Database indexing on position attributes
   - Query optimization with proper JOINs
   - Caching of filter results per disclosure
   - Lazy loading of position details

2. Bulk Suggestion Generation
   Requirement: <30 seconds for entire disclosure (100+ questions)
   Rationale: Users want to pre-populate all questions at once

   Optimization Strategy:
   - Batch processing of questions
   - Parallel position queries
   - Result caching with TTL
   - Progressive loading in UI

3. Template Import with Enrichment
   Requirement: <10 seconds for ESRS template (20MB XML)
   Rationale: Template import is infrequent but must complete smoothly

   Optimization Strategy:
   - Streaming XML parser (not DOM)
   - Chunked JSON generation
   - Background processing for large templates
   - Progress indicator in UI

4. CSV Enrichment Import
   Requirement: <5 seconds for 1000 question mappings
   Rationale: Enrichment updates should be quick for iterative refinement

   Optimization Strategy:
   - Bulk INSERT with transaction
   - Validation before database writes
   - Rollback on error
   - Audit log for traceability


**EXPECTED LOAD:**

- Concurrent users: 10-20 per customer
- Disclosures per customer: 5-10 active
- Questions per disclosure: 100-500
- Positions per customer: 1,000-50,000
- Query volume: ~100 position lookups per minute (peak)


**PERFORMANCE IMPACT ON COSTS:**

- Minimal additional load on existing infrastructure
- No need for dedicated servers
- Existing database handles query volume
- Caching reduces repeated queries by 70%+

COST ESTIMATE: $0 additional hosting costs
(Uses existing CS infrastructure allocation)

### Data Strategy



> **Guidance:**
> * [MUST] Consider management of any PII. Hosting region impacts on data e.g., GDPR. 
> Data migration guidance for project. Consider volumes. What could be the extremes? 
> Archiving strategy for subscription leavers (including PII). 
> If storing huge volumes is there an archiving requirement, e.g., move date that is 
> over 5 years out, etc.



**PII CONSIDERATIONS:**

- NO PII STORED in enrichment data
- Enrichment metadata is purely technical (unit types, scopes, etc.)
- No customer-specific information in enrichment overrides
- User who creates override is logged (non-PII audit trail)


**HOSTING REGION:**

- Data stored in customer's existing CS database region
- Multi-tenant service model: enrichment data shared across customers
- Customer-specific overrides stored in tenant-specific database
- GDPR compliant: no personal data in enrichment metadata


**DATA VOLUMES:**


1. Template Enrichment Data
   - Metadata per question: ~500 bytes
   - Questions per template: 500-1000
   - Total per template: ~250KB-500KB
   - Templates per customer: 5-10
   - Total per customer: ~2.5MB-5MB
   - EXTREME: 100 templates × 1000 questions = 50MB per customer

2. CSV Override Data
   - Row size: ~200 bytes
   - Rows per customer: 100-500 (manual overrides)
   - Total per customer: ~20KB-100KB
   - Growth rate: +50 rows/year (refinements)
   - EXTREME: 5000 overrides = 1MB per customer

3. Audit/Tracking Data
   - User override event: ~300 bytes
   - Events per disclosure: 50-200
   - Total per disclosure: ~15KB-60KB
   - Retention: 2 years
   - EXTREME: 1000 disclosures × 200 events = 60MB per customer

4. Cache Data
   - Position suggestion cache: ~5KB per question
   - TTL: 1 hour
   - Max concurrent disclosures: 10
   - Max cached data: 10 disclosures × 500 questions × 5KB = 25MB per customer


**TOTAL STORAGE PER CUSTOMER: ~10-100MB**

EXTREME: ~120MB per customer


**ARCHIVING STRATEGY:**


1. Template Enrichment
   - Retention: Indefinite (templates are reusable)
   - When customer leaves: Mark templates as archived
   - Hard delete: After 6 months if no reactivation

2. Override Data
   - Retention: Active + 3 years
   - When customer leaves: Archive to cold storage
   - Hard delete: After 3 years post-cancellation

3. User Override Tracking
   - Retention: 2 years (for learning algorithm)
   - Automatic purge: Records older than 2 years
   - When customer leaves: Immediate deletion (no business value)

4. Cache Data
   - Retention: 1 hour TTL
   - When customer leaves: Immediate deletion
   - No archiving needed


**GDPR COMPLIANCE:**

- Right to erasure: Override audit logs deleted on customer cancellation
- Data portability: Enrichment CSV export functionality
- Data minimization: Only store necessary mapping metadata
- Purpose limitation: Data used only for position/report suggestions


**DATA MIGRATION:**

- NO migration of existing data needed
- Enrichment applied to templates going forward
- Existing disclosures continue with manual selection
- Optional: Backfill enrichment for active templates (1-2 days effort)
---

## 

---

## Project Management


### Scope Approach



> **Guidance:**
> * [MUST] Share scope advice for the implementation team. 
> What should they not attempt to deliver? What might cause scope creep. 
> What unknowns should they elaborate early.
> * [COULD] Advice on milestones that ensure incremental value delivery - 
> ensure something is released every 1 month, or release x on month 2, 
> y on month 4 and z on month 5.


IN SCOPE:

1. ESRS 2024 Template Enrichment (Primary Focus)
   - All ESRS E1 Climate Change questions (50+ questions)
   - Manual enrichment by domain expert (Antonia)
   - XML schema extension for mappingHints
   - JSON converter updates

2. Position Filtering Service
   - Backend service implementation
   - Filtering by: unit type, position type, scope, site
   - Scoring by data completeness
   - Top 3 suggestions with confidence

3. Basic UI Integration
   - Display suggestions in position selector
   - Show confidence indicators
   - Allow user override
   - Track override events

4. CSV Override Mechanism
   - CSV import functionality
   - Validation and error reporting
   - Version control for enrichment data
   - Admin UI for upload

5. Unit Type Mapping
   - XBRL taxonomy → CS unit type mapping table
   - Configuration file for mappings
   - Inference rules from concept names


**OUT OF SCOPE (TO AVOID SCOPE CREEP):**


1. AI/LLM Integration
   - Covered by separate Initiative 1397587
   - Position mapping is algorithmic, not AI-based
   - No machine learning model training

2. Other Disclosure Frameworks
   - GRI, TCFD, CDP, etc. not included in initial release
   - ESRS is priority; others are follow-on work
   - Enrichment framework extensible for future use

3. Automatic Enrichment
   - No automated tagging of templates
   - Manual enrichment by domain expert required
   - Inference rules are supplementary only

4. Advanced Scoring Algorithms
   - Keep scoring simple: data completeness + recency
   - No complex ML-based confidence scoring
   - No user behavior learning (track only)

5. Report Filtering (Deferred to Phase 2)
   - Focus on position mapping first
   - Report suggestions added in follow-on release
   - Same framework applies

6. Multi-Site Position Aggregation
   - Suggestions per disclosure site only
   - No cross-site position recommendations
   - Keep scope simple for v1


**SCOPE CREEP RISKS:**


Risk 1: "Can we also enrich framework X?"
Mitigation: Focus on ESRS only for v1; extensible design for future

Risk 2: "Can we auto-tag all questions?"
Mitigation: Manual enrichment is intentional; quality over speed

Risk 3: "Can AI improve the suggestions?"
Mitigation: Separate initiative; keep algorithmic approach simple

Risk 4: "Can we suggest multiple position combinations?"
Mitigation: Single position per suggestion; combinations in phase 2

Risk 5: "Can we support custom position types?"
Mitigation: Use standard taxonomy; custom types in config file only


**UNKNOWNS TO ELABORATE EARLY:**


1. Enrichment Data Quality
   - How accurate will manual tagging be?
   - What validation checks are needed?
   - Mitigation: Early CSV review with Antonia

2. Position Attribute Completeness
   - Do all positions have type/unit metadata?
   - What % of positions lack required attributes?
   - Mitigation: Audit position data in week 1

3. XBRL Taxonomy Coverage
   - Does ESRS taxonomy cover all position types?
   - Are there gaps in concept definitions?
   - Mitigation: Taxonomy analysis in week 1

4. User Override Rate
   - How often will users reject suggestions?
   - What's acceptable override threshold?
   - Mitigation: Track and review after pilot

5. Performance at Scale
   - How does filtering perform with 50K positions?
   - Is caching strategy sufficient?
   - Mitigation: Load testing with max data volume


**INCREMENTAL VALUE DELIVERY:**


Milestone 1 (Week 2): Schema + Converter
- Deliverable: Enriched JSON from XML
- Value: Foundation for all other work
- Demo: Show enriched JSON with metadata

Milestone 2 (Week 3): Manual Enrichment Complete
- Deliverable: 100% ESRS E1 questions tagged
- Value: Enables filtering service development
- Demo: Show enrichment CSV with all attributes

Milestone 3 (Week 4): Filtering Service Working
- Deliverable: API returns position suggestions
- Value: Core functionality complete
- Demo: API call returns top 3 positions with confidence

Milestone 4 (Week 5): UI Integration
- Deliverable: Users see suggestions in disclosure UI
- Value: End-to-end workflow functional
- Demo: User creates disclosure, sees smart suggestions

Milestone 5 (Week 6): Pilot with Real Disclosure
- Deliverable: Complete ESRS E1 disclosure using suggestions
- Value: Validation of accuracy and usability
- Demo: Show 80%+ auto-match rate

### Implementation Risks



> **Guidance:**
> * [MUST] Provide leadership's view on top budget impacting risks and opportunities 
> with proposed management strategies.


#  Type        What might happen                             Likelihood  Severity    How to manage
                                                              (1-5 Low)   (T-shirt)

1  Risk        Enrichment data quality issues                    3          M        Early validation with Antonia
               Manual tagging errors reduce accuracy                                 Automated validation rules
               Wrong unit types or scopes assigned                                   Review process before production

2  Risk        Position attribute data incomplete                2          M        Audit position data in week 1
               Many positions lack type/unit metadata                               Identify and fill gaps
               Filtering returns empty results                                       Default to broader filter if needed

3  Risk        XBRL taxonomy gaps                                2          L        Taxonomy analysis in week 1
               Some CS position types not in ESRS taxonomy                          Use inference rules as fallback
               Concepts don't map 1:1 to CS data model                              Document mapping exceptions

4  Risk        User override rate too high (>30%)                3          M        Track overrides from day 1
               Suggestions not accurate enough                                       Tune scoring algorithm
               Users don't trust system                                              Show reasoning for suggestions

5  Risk        Performance issues at scale                       2          M        Load testing with max data
               Filtering slow with 50K+ positions                                    Optimize queries and indexes
               UI becomes unresponsive                                               Add caching layer

6  Risk        Scope creep: "Can we also add..."                 4          L        Strict scope management
               Requests to enrich other frameworks                                   "Yes, in phase 2" response
               Feature requests during development                                   Parking lot for future work

7  Risk        Schema evolution breaks backward compat            1          H        Thorough testing with old templates
               Old templates fail to import                                          Version compatibility checks
               JSON converter errors                                                 Graceful degradation

8  Risk        CSV override conflicts                            2          L        Priority rules clearly defined
               Multiple overrides for same question                                  Last-modified-wins strategy
               Users confused about precedence                                       UI shows active override source

9  Opportunity Early adoption by other modules                   3          M        Design extensible framework
               Question mapping applies beyond disclosure                            Generic filtering service
               Reusable for other entity selection                                   Abstract position filtering logic

10 Opportunity Continuous improvement from tracking              4          M        Analytics dashboard for overrides
               Override data reveals patterns                                        Monthly review of suggestion accuracy
               Can refine scoring algorithm                                          Iterative tuning of weights

11 Risk        Domain expert (Antonia) availability              3          M        Buffer time in schedule
               Manual enrichment blocked if unavailable                              Backlog of questions to tag
               Quality depends on her knowledge                                      Documentation of tagging guidelines

12 Risk        Integration with AI initiative conflicts          2          L        Clear separation of concerns
               Overlap in position mapping approaches                                Coordinate with AI team
               Duplicate functionality                                               This = metadata-based; AI = text-based

### Dependencies [MUST]


Needs Sphera ARB - Architecture Review Board (Yes/No & why)

NO - This is an internal CS module enhancement. No cross-solution family interaction.
     Does not introduce new external services or shared platform components.
     Uses existing CS infrastructure and database.

Guidance re: ARB – if the solution interacts with Sphera services external to one
solution family. E.g., A new SCCS capability needs to interact with SCS.
If in doubt escalate to Engineering VP.

Area                    Is Area Involvement Needed in Planning onwards? (Yes/No & Why)

InfoSec                 YES - Review enrichment data storage and validation
                            Review XML schema extension for injection risks
                            Validate no PII in enrichment metadata
                            Confirm GDPR compliance for tracking data

DevOps                  YES - Database migration for new enrichment table
                            Index creation for position filtering queries
                            Config file deployment for unit mappings
                            No infrastructure changes needed

Platform                NO  - No shared platform services used
                            Self-contained CS module enhancement
                            No API gateway or service mesh changes

UX                      YES - Review smart suggestion UI components
                            Design confidence indicator display
                            User research: How to show 1-3 suggestions?
                            Override workflow design

Other Solution Family   NO  - No interaction with LCA, SCS, or other families
                            CS-only functionality
                            No shared data models

QA (end to end testing) YES - E2E test scenarios for suggestion workflow
                            Performance testing with large position volumes
                            Regression testing of existing disclosure flow
                            Validation of enrichment data accuracy

Platform Team           NO  - Not using AI platform services
                            Separate from Initiative 1397587
                            No LLM or RAG components
---

## 

---

## Solution Budget


### Engineering Staff Months Effort



> **Guidance:**
> * [MUST] Outline the likely construction effort
> * Estimates need to cover +/- 50% for budgeting. 
> Use +40% for all communicated budget requirements. 
> i.e. Plan for 10 SM effort, must ask for 14 SM budget.


Milestone                    Engineer Staff Months        Max Eng    Min Dur    Comments
                             Best    Mid     Worst

Schema & Converter            1.0    1.5      2.0           2        1.5 mth    Best: Experienced dev
                                                                                Mid: Some XML challenges
                                                                                Worst: Schema issues

Manual Enrichment             0.5    0.75     1.0           1        1 mth      Antonia + dev support
(Antonia + validation)                                                          Mid: Learning curve
                                                                                Worst: Data quality issues

Filtering Service             1.5    2.0      3.0           2        1.5 mth    Best: Straightforward
                                                                                Mid: Algorithm tuning
                                                                                Worst: Performance issues

UI Integration                1.0    1.5      2.0           2        1 mth      Best: Existing components
                                                                                Mid: New suggestion UI
                                                                                Worst: ExtJS complexity

Testing & Refinement          1.0    1.5      2.5           2        1 mth      Best: High test coverage
                                                                                Mid: Accuracy tuning
                                                                                Worst: Major rework needed
### 

TOTALS                        5.0    7.25     10.5          2        3 mth      MID-CASE ESTIMATE

Preferred Timeline: 3 months with 2 engineers in parallel

Budget Request Calculation (using Mid-case):
- Mid-case: 7.25 SM
- Add 40% buffer: 7.25 × 1.4 = 10.15 SM
- BUDGET REQUEST: 10 Staff Months (rounded)

### Non Engineering Staff Months Effort



> **Guidance:**
> * [MUST] Outline other function Staff Month needs 
> (talk with function OLT owner to confirm estimate)
> * These non-engineering estimates must be obtained from the individual functional teams 
> after the architecture is approved


Milestone          DevOps   InfoSec   UX    PrjMgr   TW    Platform   Domain Expert (Antonia)

Schema & Converter   0.25     0.25    0.1    0.25    0.1      -              -
Manual Enrichment     -        -      -      0.1     -        -              0.75
Filtering Service    0.25     0.1     -      0.25    0.1      -              0.1
UI Integration        -       0.1     0.5    0.25    0.1      -              0.1
### Testing & Refine     0.25     0.25    0.1    0.5     0.1      -              0.25

TOTALS               0.75     0.7     0.7    1.35    0.4      0              1.2

TOTAL NON-ENGINEERING: 5.1 Staff Months


**GRAND TOTAL EFFORT:**

- Engineering: 10 SM (with buffer)
- Non-Engineering: 5.1 SM
- TOTAL: 15.1 Staff Months
---

## 

---

## Should-Haves


### Service Scaling



> **Guidance:**
> * [SHOULD] state scaling approach alongside predicted load over the next 3 years. 
> Consider the path for scaling extremes of use, e.g., 10,000 users or 1000s API 
> calls per second. 
> State if new service, if it can be multi-tenant, if project will carve out from monolith



**CURRENT ARCHITECTURE:**

- Monolithic CS application (not carved out)
- Multi-tenant by database (shared application, separate DB per customer)
- Position filtering service is internal (not exposed API)


**PREDICTED LOAD OVER 3 YEARS:**


Year 1 (2026):
- Customers using ESRS: 50
- Active disclosures: 250
- Questions per disclosure: 500
- Position filtering requests: ~125,000 per month
- Concurrent users: 10-20 per customer
- Peak load: ~50 requests/minute

Year 2 (2027):
- Customers using ESRS: 150
- Additional frameworks (GRI, TCFD): +100 customers
- Active disclosures: 1,000
- Position filtering requests: ~500,000 per month
- Peak load: ~200 requests/minute

Year 3 (2028):
- Customers using disclosure: 300
- Active disclosures: 2,500
- Position filtering requests: ~1.25M per month
- Peak load: ~500 requests/minute


**SCALING STRATEGY:**


Horizontal Scaling (Current Architecture):
- Application server: Add more instances (load balanced)
- Database: Read replicas for position queries
- Caching: Redis/Memcached for filter results
- Current infrastructure supports 3-year projection

Vertical Scaling (If Needed):
- Database: Increase CPU/memory for query performance
- Application: Larger instance types for parallel processing
- Not expected to be necessary within 3 years

Extreme Use Case (1000s requests/second):
- Carve out PositionMappingService as microservice
- Dedicated service with own database
- API gateway for rate limiting
- Horizontal scaling of service instances
- Estimate: Required if >10,000 customers


**MULTI-TENANCY:**

- Enrichment metadata: Shared across tenants (ESRS standard)
- Override data: Tenant-specific in customer database
- Filtering logic: Shared service, tenant-isolated data
- No customer data cross-contamination

CARVE-OUT STRATEGY (Future):
- Phase 1: Internal service within CS monolith (current)
- Phase 2: Separate service class with REST API (Year 2)
- Phase 3: Independent microservice (if >5000 customers)
- Phase 4: Platform service used by multiple solution families

### Resilience



> **Guidance:**
> * [SHOULD] State expected service level needed, consider; 
> is this a critical service, can the rest of the app function with this down. 
> How will this going down affect the rest of the application offering? 
> Will this have application health end point(s) etc.



**SERVICE CRITICALITY: MEDIUM**


- Not a critical service for application operation
- Disclosure functionality works without smart suggestions
- Fallback: Manual position selection (current behavior)
- No data loss if service unavailable


**FAILURE SCENARIOS:**


Scenario 1: Position Filtering Service Down
Impact: Users see all positions instead of suggestions
Workaround: Manual selection from full list (existing behavior)
Recovery: Auto-recover when service restarts
RTO: 1 hour
RPO: N/A (no data loss)

Scenario 2: Enrichment Data Unavailable
Impact: Suggestions use inference rules only (lower accuracy)
Workaround: Partial suggestions or full list
Recovery: Reload enrichment data from CSV backup
RTO: 30 minutes
RPO: Last CSV import (daily backup)

Scenario 3: Database Query Performance Degradation
Impact: Slow suggestion loading (>5 seconds)
Workaround: Timeout after 5s, show all positions
Recovery: Database optimization, query cache clear
RTO: 15 minutes
RPO: N/A

Scenario 4: Cache Service Down (Redis)
Impact: Higher database load, slower responses
Workaround: Direct database queries (no cache)
Recovery: Restart cache service
RTO: 5 minutes
RPO: N/A (cache rebuilt automatically)


**REST OF APPLICATION:**

- Disclosure creation: UNAFFECTED
- Question answering: UNAFFECTED
- Data entry: UNAFFECTED
- Report generation: UNAFFECTED
- XBRL export: UNAFFECTED
- Only position suggestion feature impacted


**HEALTH ENDPOINTS:**


1. /api/disclosure/health
   - Overall disclosure module health
   - Includes position filtering service status

2. /api/disclosure/position-mapping/health
   - Specific health check for filtering service
   - Returns: {status: "up", enrichmentLoaded: true, cacheAvailable: true}

3. Monitoring Metrics:
   - Position filtering request count
   - Average response time
   - Cache hit rate
   - Error rate
   - Override rate (quality indicator)


**DEGRADATION STRATEGY:**


Level 1: Full Functionality
- Smart suggestions with high confidence
- 1-3 position candidates
- <2s response time

Level 2: Degraded (Slow Queries)
- Suggestions without scoring
- Top 10 positions by data completeness
- <5s response time

Level 3: Fallback (Service Down)
- No suggestions, show all positions
- Manual selection only
- User notification: "Smart suggestions temporarily unavailable"

Level 4: Minimal (Database Issues)
- Disable position filtering entirely
- Manual entry of position IDs
- Admin notification for urgent fix

### Hosting Service Costs



> **Guidance:**
> * [COULD] This is a stretch target for this document - 
> Outline the hosting cost impact expected as a result of this project 
> (add both Dev and Prod costs). 
> Limit to top 5 cost areas, unless others are significantly impacted. 
> Assume preferred option but state how similar or different it is from other 
> options in opening paragraph here. 
> Discuss how costs will be managed, and the impact on ROI based on product's 
> pricing approach.


Component             Hosting Platform    Service Name      Meter Name       Current Monthly   New Monthly   Comment
                                                                              Spend ($k)        Delta ($k)

Application Server    Azure               VM B4ms           Compute Hours         50              +2         Minimal CPU increase
                                                                                                             Filtering is lightweight

Database              Azure               MySQL 5.7         Storage/IOPS          30              +1         New enrichment table
                                                                                                             ~100MB + indexes

Cache Layer           Azure               Redis Basic       Memory                10              +0.5       Suggestion cache
                                                                                                             Small footprint

Bandwidth             Azure               Network Egress    GB Transferred        5               +0.1       JSON responses
                                                                                                             Minimal increase

Monitoring            Azure               App Insights      Events/Storage        2               +0.2       Health endpoints
### Performance metrics

TOTALS                                                                            97              +3.8


**DEVELOPMENT ENVIRONMENT COSTS:**

- Dev/UAT environments: +$1.5k/month (same proportion)
- Total Dev/UAT: $1.5k/month


**TOTAL MONTHLY COST INCREASE:**

- Production: +$3.8k/month
- Dev/UAT: +$1.5k/month
- TOTAL: +$5.3k/month = $63.6k/year


**COST PER PAYING TENANT:**

- Current subscribed tenants: ~500
- Monthly increase per tenant: $5.3k / 500 = $10.60/tenant
- Annual increase per tenant: $127/tenant


**COST MANAGEMENT:**

1. Caching Strategy: 70% reduction in database queries
2. Query Optimization: Efficient indexes on position attributes
3. Lazy Loading: Load suggestions on-demand, not upfront
4. TTL on Cache: 1-hour expiry prevents stale data accumulation


**ROI IMPACT:**

- Product pricing: ESRS module is premium feature (+$5k-10k/year per customer)
- Smart mapping adds value: Reduces customer effort by 60%
- Customer retention: Improved UX reduces churn
- Cost per customer: $127/year (1.3% of premium pricing)
- Net profit margin: Remains >90% after cost increase

CONCLUSION: Minimal hosting cost impact with strong ROI
- Cost increase <2% of overall hosting budget
- Cost per customer negligible compared to feature value
- No additional infrastructure required
- Scales efficiently with existing architecture

### 3rd Party Costs



> **Guidance:**
> * [SHOULD] Share leadership's view on potential incremental impact on 
> Software License costs, Data Service costs, tooling costs, etc., 
> including new services. Lay out on an annual or one-off basis.


SUMMARY: $0 incremental 3rd party costs


**3RD PARTY SERVICES/LICENSES REQUIRED:**


1. XBRL Taxonomy (EFRAG ESRS)
   Cost: FREE (Open standard)
   License: Creative Commons
   Source: https://www.efrag.org/
   Update frequency: Annual (free downloads)
   Comment: No licensing fees

2. XML Schema Validation
   Cost: FREE (Open source)
   Tool: libxml2 (PHP built-in)
   License: MIT License
   Comment: Already included in PHP

3. Database (MySQL)
   Cost: EXISTING (no increase)
   License: GPLv2 or Commercial (already licensed)
   Comment: Using existing CS database

4. Development Tools
   Cost: EXISTING (no increase)
   Tools: PHPStorm, Git, Jenkins
   Comment: Already licensed for team

5. Testing Tools
   Cost: EXISTING (no increase)
   Tools: PHPUnit, Selenium, JMeter
   Comment: Already in use


**SERVICES EXPLICITLY NOT USED:**

- NO AI/LLM services (separate initiative)
- NO 3rd party APIs
- NO data enrichment services
- NO cloud AI platforms (Azure OpenAI, AWS Bedrock, etc.)


**ONE-TIME COSTS: $0**


**ANNUAL COSTS: $0**


**TOTAL 3RD PARTY COST IMPACT: $0**

---

## 

---

## Could-Haves


### Deployment Strategy


Consider:
* [MUST] Which regions will this be deployed in as part of this project?
* [SHOULD] how will the product be checked in lower environments
  (especially where platform and other products involved in project).
* [COULD] How the application will be made available to subscribers, e.g.,
  toggles, configurations, licensing restrictions.
* [COULD] Can this project improve deployment downtime towards zero,
  with requirement to focus on making it no worse than today?
* [COULD] Is this needed on-prem?

DEPLOYMENT REGIONS [MUST]:
- Azure West Europe (primary)
- Azure East US (secondary)
- Azure Southeast Asia (future)
- Same regions as existing CS deployment

LOWER ENVIRONMENT STRATEGY [SHOULD]:

1. Development (DEV)
   - Deploy enriched ESRS template
   - Test manual CSV import
   - Validate filtering service with sample positions
   - Schema migration testing

2. User Acceptance Testing (UAT)
   - Full ESRS E1 enrichment deployed
   - Real-like position data (anonymized)
   - End-to-end disclosure workflow testing
   - Performance testing with 10K positions

3. Staging (STAGE)
   - Production-like data volume
   - Full regression testing
   - Load testing with max expected volume
   - Blue-green deployment rehearsal

4. Production (PROD)
   - Phased rollout (see below)
   - Monitoring dashboard active
   - Rollback plan ready

FEATURE TOGGLES [COULD]:

Toggle 1: smart_position_suggestions
- Default: OFF
- Enables: Position filtering service
- Rollout: Gradual by customer

Toggle 2: enrichment_csv_import
- Default: ON for admins only
- Enables: CSV enrichment upload
- Rollout: Internal first, then customer admins

Toggle 3: bulk_suggestion_mode
- Default: OFF
- Enables: Pre-populate all questions
- Rollout: After individual suggestion stable

Toggle 4: suggestion_confidence_display
- Default: ON
- Enables: Show confidence scores to users
- Rollout: All users (UX improvement)

AVAILABILITY STRATEGY [SHOULD]:

1. Subscriber Access
   - Licensing: Included in ESRS module subscription
   - No additional license required
   - Feature flag per customer

2. Gradual Rollout
   - Week 1: Internal testing (5 customers)
   - Week 2: Pilot customers (10 customers)
   - Week 3: Early adopters (50 customers)
   - Week 4: General availability (all ESRS subscribers)

3. Fallback Plan
   - Feature toggle OFF reverts to manual selection
   - No data loss or corruption
   - Instant rollback if issues detected

ZERO DOWNTIME DEPLOYMENT [COULD]:

Blue-Green Strategy:
1. Deploy to green environment
2. Run smoke tests
3. Switch traffic to green
4. Monitor for 1 hour
5. Decommission blue if stable

Database Migration:
1. Create new enrichment table (non-blocking)
2. Add indexes during maintenance window
3. Populate enrichment data asynchronously
4. Enable feature toggle after data ready

Backward Compatibility:
- Old templates without enrichment: Still work
- New code reads enrichment if present
- Graceful degradation if enrichment missing


**DOWNTIME ESTIMATE:**

- Database migration: 10 minutes (maintenance window)
- Application deployment: 0 minutes (blue-green)
- Total planned downtime: 10 minutes

ON-PREMISE DEPLOYMENT [COULD]:

Status: NOT PLANNED for v1
Reason: ESRS is EU regulation, typically cloud deployments

Future Consideration:
- If on-prem customers need ESRS compliance
- Deployment package: Same as cloud + enrichment data files
- Update mechanism: Manual CSV import for enrichment updates
- Effort estimate: +2 SM for on-prem packaging

### Test Maintainability



> **Guidance:**
> * [SHOULD] Outline how test automation coverage should be approached.
> * [COULD] Will end to end tests be developed, if so when by who.
> * [COULD] Is there a significant debt in automated testing in this 
> service/functionality area and potentially dependent parts of the codebase?



**TEST COVERAGE STRATEGY:**


1. UNIT TESTS (Target: 90% coverage)
   - PositionMappingService: All filtering logic
   - EnrichmentValidator: All validation rules
   - JsonConverter: Enrichment parsing
   - Inference rules: Concept name parsing
   - Scoring algorithm: Data completeness calculation

   Framework: PHPUnit
   Location: sofi/app/tests/unit/DisclosureManagement/
   Owner: Development team
   When: Written during development (TDD approach)

2. INTEGRATION TESTS (Target: 80% coverage)
   - XML import with enrichment
   - JSON generation with metadata
   - CSV import and validation
   - Position filtering with database
   - Cache layer integration

   Framework: PHPUnit + Doctrine test helpers
   Location: sofi/app/tests/integration/DisclosureManagement/
   Owner: Development team
   When: After unit tests pass

3. API TESTS (Target: 100% endpoint coverage)
   - GET /api/disclosure/position-suggestions
   - POST /api/disclosure/enrichment/import
   - GET /api/disclosure/position-mapping/health
   - Error scenarios (404, 400, 500)
   - Authentication/authorization

   Framework: PHPUnit + HTTP client
   Location: sofi/app/tests/api/DisclosureManagement/
   Owner: Development team
   When: After integration tests pass

4. END-TO-END TESTS (Target: 10 critical paths)
   - Create disclosure → Assign site → See suggestions
   - Accept suggestion → Save disclosure → Verify position linked
   - Override suggestion → Track override event
   - Bulk assignment mode → Verify all questions populated
   - Import enrichment CSV → Verify suggestions updated

   Framework: Selenium + Behat
   Location: sofi/app/tests/e2e/DisclosureManagement/
   Owner: QA team
   When: Week 5 (after UI integration complete)


**5. PERFORMANCE TESTS**

   - Position filtering with 1K, 10K, 50K positions
   - Concurrent suggestion requests (50 users)
   - Cache hit rate under load
   - Database query performance

   Framework: JMeter + custom scripts
   Location: sofi/app/tests/performance/
   Owner: QA team + DevOps
   When: Week 5 (before production rollout)


**6. REGRESSION TESTS**

   - Existing disclosure functionality unaffected
   - Old templates still import correctly
   - Manual position selection still works
   - XBRL export not impacted

   Framework: Existing Behat test suite
   Location: sofi/app/tests/behat/
   Owner: QA team
   When: Week 6 (final validation)


**TEST DATA STRATEGY:**


1. Enrichment Test Data
   - Sample ESRS template with 50 questions
   - Fully enriched with all metadata attributes
   - CSV with 100 manual overrides
   - Location: sofi/app/tests/fixtures/disclosure/

2. Position Test Data
   - 1,000 positions with various types/units/scopes
   - 100 positions per type (emission, energy, water, etc.)
   - Mix of complete and incomplete data
   - Location: sofi/app/tests/fixtures/positions/

3. Mock XBRL Taxonomy
   - 50 ESRS concepts
   - Unit type mappings
   - Inference rule test cases
   - Location: sofi/app/tests/fixtures/xbrl/


**EXISTING TEST DEBT:**


Area: Disclosure Management
Current Coverage: ~60% unit, ~40% integration
Debt: Missing tests for complex position linking
Impact: Will add tests for this area during development
Effort: +0.5 SM to improve existing coverage

Area: Template Import/Export
Current Coverage: ~70% unit, ~50% integration
Debt: XML validation tests incomplete
Impact: Enrichment tests will improve this
Effort: Included in project scope


**CONTINUOUS INTEGRATION:**


1. Pre-Commit Hooks
   - Run unit tests (<30s)
   - Code style validation (PHP_CodeSniffer)

2. Pull Request Checks
   - Full unit + integration tests (<5 min)
   - Code coverage report
   - Static analysis (PHPStan)

3. Nightly Builds
   - All tests including E2E (<30 min)
   - Performance tests (<1 hour)
   - Coverage report generation


**TEST MAINTENANCE PLAN:**


1. Ownership
   - Unit/Integration: Development team
   - E2E: QA team with dev support
   - Performance: QA + DevOps

2. Update Cadence
   - Unit tests: Updated with every code change
   - Integration tests: Updated weekly
   - E2E tests: Reviewed monthly
   - Performance tests: Run before each release

3. Flaky Test Management
   - Retry strategy: Max 3 retries
   - Quarantine: Move flaky tests to separate suite
   - Fix within 1 sprint or delete

### Disaster Recovery Strategy


Consider:
* [MUST] How will data be backed up and restored across all deployment regions?
* [SHOULD] What is the failover process if the primary region becomes unavailable?
* [SHOULD] How will the system handle partial outages
  (database, network, specific services)?
* [SHOULD] What monitoring and alerting will detect system failures requiring DR activation?
* [COULD] How will disaster recovery procedures be tested and validated regularly?
* [COULD] How will users be notified and redirected during a disaster recovery scenario?

DATA BACKUP STRATEGY [MUST]:

1. Enrichment Metadata (Templates)
   - Backup frequency: Daily
   - Retention: 30 days rolling + 12 monthly snapshots
   - Location: Azure Blob Storage (geo-redundant)
   - Recovery method: Template re-import from XML + enrichment CSV
   - RTO: 2 hours (re-import all templates)
   - RPO: 24 hours (last daily backup)

2. Override Data (CSV imports)
   - Backup frequency: Real-time (after each import)
   - Retention: All versions (immutable log)
   - Location: Azure Blob Storage + Git repository
   - Recovery method: Re-import last CSV
   - RTO: 15 minutes
   - RPO: 0 (no data loss)

3. User Override Tracking (Analytics)
   - Backup frequency: Hourly
   - Retention: 90 days
   - Location: Database backup (part of CS backup)
   - Recovery method: Restore from database backup
   - RTO: 4 hours (included in database restore)
   - RPO: 1 hour

4. Cache Data
   - Backup frequency: NONE (ephemeral, TTL 1 hour)
   - Recovery method: Rebuilt automatically on first request
   - RTO: Immediate (cache-aside pattern)
   - RPO: N/A


**CROSS-REGION REPLICATION:**


Primary Region: Azure West Europe
Secondary Region: Azure East US

Replication Strategy:
- Database: Active-passive replication (async)
- Blob Storage: Geo-redundant (automatic)
- Application: Blue-green in both regions
- DNS: Azure Traffic Manager for failover

FAILOVER PROCESS [SHOULD]:

Trigger Conditions:
1. Primary region unavailable >5 minutes
2. Database connection failures >threshold
3. Manual failover initiated by ops team

Failover Steps:
1. Detect: Monitoring alerts ops team (auto or manual)
2. Assess: Verify secondary region health
3. Switch: Traffic Manager redirects to secondary
4. Validate: Health checks pass in secondary
5. Notify: Users see banner "Operating in backup region"
6. Monitor: Ops team watches for issues
7. Failback: Return to primary when stable

Failover Time: 10-15 minutes (automated)

PARTIAL OUTAGE HANDLING [SHOULD]:

Scenario 1: Database Slow/Unavailable
- Impact: Position suggestions unavailable
- Handling: Return cached results (if available) OR fallback to full position list
- User Experience: Show message "Loading positions..." with timeout
- Recovery: Auto-retry after 30 seconds

Scenario 2: Cache Service Down
- Impact: Higher database load, slower responses
- Handling: Direct database queries (no cache)
- User Experience: Slightly slower suggestion loading (2s → 5s)
- Recovery: Automatic when cache service restarts

Scenario 3: Enrichment Data Unavailable
- Impact: Suggestions use inference rules only (lower accuracy)
- Handling: Fallback to concept name parsing + broader filters
- User Experience: More position candidates shown (1-3 → 5-10)
- Recovery: Reload enrichment data from backup

Scenario 4: Network Partition
- Impact: Some users can't reach application
- Handling: Traffic Manager routes to healthy endpoints
- User Experience: Brief connection retry, then success
- Recovery: Automatic when network restored

MONITORING & ALERTING [SHOULD]:

Health Checks (every 30 seconds):
- Application health endpoint: /api/health
- Position mapping service: /api/disclosure/position-mapping/health
- Database connection: Active query test
- Cache service: Ping + set/get test

Alert Conditions:
- Health check fails 3 consecutive times → P3 alert (5 min)
- Health check fails 10 consecutive times → P2 alert (2 min)
- Primary region unavailable → P1 alert (1 min) + auto-failover

Alert Channels:
- PagerDuty: Ops on-call engineer
- Email: DevOps team + engineering lead
- Slack: #incidents channel
- SMS: P1 alerts only

Metrics Monitored:
- Request success rate (target: >99.5%)
- Average response time (target: <2s)
- Error rate (target: <0.5%)
- Cache hit rate (target: >70%)
- Database query time (target: <500ms)

DR TESTING PLAN [COULD]:

Quarterly DR Drill:
1. Schedule maintenance window (Sunday 2am)
2. Simulate primary region failure
3. Verify automatic failover
4. Test application functionality in secondary
5. Verify data integrity
6. Failback to primary
7. Document lessons learned

Annual Full DR Test:
1. Restore from backup to clean environment
2. Import all templates
3. Import all enrichment data
4. Verify position filtering works
5. Test with pilot users
6. Measure RTO/RPO achieved

DR Test Checklist:
□ Verify backup files accessible
□ Test database restore procedure
□ Validate enrichment CSV import
□ Confirm position filtering works
□ Check cache rebuild
□ Verify user authentication
□ Test XBRL export (dependency)
□ Measure time to recovery
□ Document any issues

USER COMMUNICATION [COULD]:

During Planned Maintenance:
- Notification: 7 days advance via email + in-app banner
- Message: "System maintenance scheduled [date/time]. Enrichment features unavailable for 1 hour."
- Alternative: Manual position selection available

During Unplanned Outage:
- Detection: Automatic monitoring alert (within 2 min)
- Status Page: status.spherasolutions.com updated
- In-App Banner: "Position suggestions temporarily unavailable. Manual selection available."
- Email: Sent to active users after 15 minutes
- Resolution: Status page updated when restored

During Failover:
- Banner: "Operating in backup region. Functionality normal."
- No email: If transparent failover (<15 min)
- Email: If extended failover (>1 hour) explaining situation
- Resolution: "Returned to primary region. Thank you for patience."

Recovery Notification:
- In-App Banner: "Smart suggestions restored. Thank you for patience."
- Email: Sent to users who attempted to use feature during outage
- Post-Mortem: Published within 5 business days (if P1 incident)


**DATA INTEGRITY VALIDATION:**


After Recovery:
1. Run integrity checks: Verify template count, enrichment data completeness
2. Test suggestion quality: Run automated tests against known good results
3. Compare metrics: Pre-outage vs post-outage (cache hit rate, response times)
4. User validation: Pilot users test critical workflows
5. Sign-off: Ops lead confirms system healthy before removing alerts

### Onboarding New Customers/Users Strategy



> **Guidance:**
> * [MUST] Define the step-by-step process for new customer/user provisioning 
> and account setup.
> * [MUST] Identify who is responsible for each stage of the onboarding process 
> (technical teams, support, sales, etc.).
> * [SHOULD] Establish clear timelines and SLAs for onboarding completion.
> * [COULD] Define rollback procedures if onboarding issues are discovered post-deployment.
> * [COULD] Establish monitoring and metrics to track onboarding success rates and bottlenecks.


NEW CUSTOMER ONBOARDING [MUST]:


**PREREQUISITES:**

- Customer has active CS subscription
- ESRS module licensed
- Database provisioned
- Sites configured


**STEP-BY-STEP PROCESS:**


Step 1: Template Provisioning (Day 1)
- Action: Deploy enriched ESRS template to customer database
- Method: Automated script during customer setup
- Responsible: DevOps team
- SLA: Within 24 hours of subscription activation
- Validation: Verify template import successful, enrichment data present

Step 2: Position Data Audit (Day 1-2)
- Action: Check customer position data completeness
- Method: Automated report: % positions with type/unit metadata
- Responsible: Implementation team
- SLA: Report generated within 24 hours
- Validation: If <80% complete, flag for data cleanup

Step 3: Feature Toggle Activation (Day 2)
- Action: Enable smart_position_suggestions toggle for customer
- Method: Admin portal toggle + database flag
- Responsible: Customer success team
- SLA: After position audit complete
- Validation: Test with sample disclosure

Step 4: User Training (Day 3-5)
- Action: Provide training materials and demo
- Materials: Video tutorial, PDF guide, sample disclosure
- Responsible: Customer success team
- SLA: Training scheduled within 5 business days
- Validation: User confirms understanding

Step 5: First Disclosure Support (Day 5-10)
- Action: Support customer through first ESRS disclosure
- Method: Live screen share or asynchronous support
- Responsible: Support team + domain expert
- SLA: Response within 4 hours during business hours
- Validation: First disclosure completed successfully

Step 6: Feedback Collection (Day 10-14)
- Action: Survey customer on suggestion accuracy and usability
- Method: In-app survey + follow-up call
- Responsible: Product manager
- SLA: Survey sent after first disclosure complete
- Validation: NPS score >7, accuracy >70%


**RESPONSIBILITY MATRIX:**


Task                          Responsible           Accountable        Consulted         Informed
Template Provisioning         DevOps                Tech Lead          -                 Customer Success
Position Data Audit           Implementation        Tech Lead          Customer          Customer Success
Feature Toggle Activation     Customer Success      Product Manager    DevOps            Customer
User Training                 Customer Success      Training Lead      Product Manager   Customer
First Disclosure Support      Support Team          Support Manager    Domain Expert     Customer Success
Feedback Collection           Product Manager       VP Product         Customer Success  Engineering

ONBOARDING TIMELINE [SHOULD]:

Standard Timeline:
- Day 0: Customer subscription activated
- Day 1: Template provisioned, position audit complete
- Day 2: Feature toggle activated
- Day 3-5: User training delivered
- Day 5-10: First disclosure with support
- Day 10-14: Feedback collected
- TOTAL: 14 days to full adoption

Fast Track (Enterprise):
- Day 0: Subscription + immediate provisioning
- Day 1: Position audit + feature activation
- Day 2: Live training session
- Day 3-5: Dedicated support for first disclosure
- TOTAL: 5 days to full adoption

SLAs:
- Template provisioning: 24 hours
- Position audit report: 24 hours
- Training materials available: Immediately
- Live training scheduled: 5 business days
- Support response time: 4 hours (business hours)
- First disclosure completion support: 10 business days

NEW USER ONBOARDING (Existing Customer):

Step 1: User Account Creation (Immediate)
- Action: Admin adds user to ESRS module
- Method: CS admin portal
- Responsible: Customer admin
- Validation: User receives welcome email

Step 2: Permission Assignment (Immediate)
- Action: Grant disclosure creation/edit permissions
- Method: Role-based access control
- Responsible: Customer admin
- Validation: User can access disclosure module

Step 3: Self-Service Training (On-demand)
- Action: User completes training module
- Method: In-app tutorial + help center
- Responsible: User (self-paced)
- Validation: Tutorial completion tracked

Step 4: First Use (Day 1-3)
- Action: User creates first disclosure with suggestions
- Method: Guided workflow with tooltips
- Responsible: User
- Validation: Disclosure saved successfully

ROLLBACK PROCEDURES [COULD]:

Rollback Trigger Conditions:
1. Suggestion accuracy <50% (customer complaint)
2. Performance issues (>10s response time)
3. Data corruption detected
4. Customer requests feature disable

Rollback Steps:
1. Disable feature toggle for customer (immediate)
2. Investigate root cause
3. Fix data or code issue
4. Test in UAT environment
5. Re-enable for customer after validation
6. Monitor closely for 48 hours

Rollback Responsibility:
- Initiate: Customer Success or Support
- Execute: DevOps team
- Validate: QA team
- Approve re-enable: Tech Lead

Rollback SLA:
- Feature disable: 30 minutes
- Issue resolution: 5 business days
- Re-enable: After successful UAT test


**CUSTOMER COMMUNICATION DURING ROLLBACK:**

- Notification: Email + in-app message
- Message: "Smart suggestions temporarily disabled while we improve accuracy. Manual selection available."
- Updates: Daily status email if issue extends >3 days
- Resolution: "Smart suggestions re-enabled with improved accuracy."

ONBOARDING METRICS [COULD]:

Success Metrics:
1. Onboarding completion rate: Target >95%
2. Time to first disclosure: Target <10 days
3. User adoption rate: Target >80% of ESRS users
4. Suggestion acceptance rate: Target >70%
5. Customer satisfaction (NPS): Target >8

Monitoring Dashboard:
- New customers onboarded per month
- Average onboarding duration
- Position audit pass rate (>80% complete)
- First disclosure success rate
- Feature toggle activation rate
- Training completion rate
- Support ticket volume (onboarding-related)

Bottleneck Detection:
- If avg onboarding >14 days → Investigate delays
- If position audit fails >30% → Improve data quality process
- If first disclosure fails >20% → Improve training materials
- If support tickets >5 per customer → Improve documentation

Continuous Improvement:
- Monthly review of onboarding metrics
- Quarterly update of training materials
- Annual review of onboarding process
- Customer feedback incorporated into process updates

### Configuration Management Approach



> **Guidance:**
> * [COULD] Outline configuration approach



**CONFIGURATION LEVELS:**


1. Global Configuration (All Customers)
   - Location: sofi/app/config/disclosure_unit_mapping.yml
   - Contains: XBRL → CS unit type mappings
   - Format: YAML
   - Version control: Git
   - Deployment: Part of application deployment
   - Update frequency: As needed (XBRL taxonomy updates)

   Example:
   ```yaml
   unit_mappings:
     xbrli:massItemType: mass
     xbrli:energyItemType: energy
     xbrli:volumeItemType: volume
     esrs:percentItemType: percentage

   position_types:
     emission:
       keywords: [emission, GHG, CO2, greenhouse]
       default_unit: mass
     energy:
       keywords: [energy, consumption, electricity]
       default_unit: energy
   ```

2. Customer-Specific Configuration
   - Location: Database table (disclosure_customer_config)
   - Contains: Feature toggles, enrichment preferences
   - Format: JSON in database
   - Version control: Database migration + audit log
   - Update frequency: As needed per customer

   Fields:
   - customer_id
   - feature_toggles (JSON)
   - enrichment_enabled (boolean)
   - bulk_assignment_enabled (boolean)
   - confidence_threshold (string: high/medium/low)
   - custom_unit_mappings (JSON)

3. Enrichment Override Data
   - Location: Database table (disclosure_enrichment_override)
   - Contains: Manual question → position type mappings
   - Format: Relational table
   - Version control: Audit trail with created_at/updated_at
   - Update frequency: As needed (Antonia updates)

   Fields:
   - question_sid (UUID)
   - template_id (foreign key)
   - unit_type (enum)
   - position_type (enum)
   - scope (enum)
   - requires_calculation (boolean)
   - confidence (enum: high/medium/low)
   - created_by (user_id)
   - created_at (timestamp)
   - updated_at (timestamp)

Complexity [COULD]:


**SPECTRUM OF CONFIGURATION:**


Level 1: Simple System-Wide Flag ✓
- Feature toggle: ON/OFF per customer
- No complex configuration needed
- Example: smart_position_suggestions = true/false

Level 2: Tenant-Specific Configuration ✓
- Customer preferences for confidence threshold
- Custom unit type mappings per customer (rare)
- Example: Customer A wants high-confidence only, Customer B wants all suggestions

Level 3: Business Process Specific (NOT APPLICABLE)
- N/A for this feature
- Position mapping is not business-process dependent

Level 4: User-Specific Config (NOT APPLICABLE)
- N/A for this feature
- All users in a customer see same suggestions

SELECTED APPROACH: Level 1 + Level 2
- Default: Global unit mappings + enrichment overrides
- Optional: Customer-specific confidence threshold
- Rationale: Simple enough to maintain, flexible for edge cases

Management [COULD]:


**INTELLECTUAL PROPERTY IN CONFIGURATION:**


Q: Is configuration IP-sensitive?
A: YES - Enrichment override data represents domain expertise

Protection Strategy:
1. Enrichment CSV: Stored in private Git repository (access controlled)
2. XBRL mappings: Based on public taxonomy (not IP)
3. Inference rules: Standard keyword matching (not IP)
4. Customer overrides: Isolated per customer database (secure)

Q: How to propagate to higher environments?
A:
1. Global Config: Git → Jenkins → Deploy to UAT → Deploy to PROD
2. Enrichment CSV: Import via admin UI in each environment
3. Customer config: Database migration + manual review in UAT before PROD

Q: How to maintain over time?
A:
1. Global Config: Update via code deployment (versioned)
2. Enrichment CSV: Upload via admin UI, version tracked in table
3. Customer config: Change via admin portal, audit log for compliance

TOOLING:

For Customer Admins:
- Feature: None needed (configured by Sphera admins only)

For Sphera Internal (SI):
- Feature: Admin portal page for enrichment CSV upload
- Location: /admin/disclosure/enrichment
- Permissions: Admin role only
- Functions:
  - Upload CSV
  - Validate CSV format
  - Preview before commit
  - View current enrichment data
  - Export enrichment to CSV

For Services (Automation):
- Feature: CLI command for bulk enrichment update
- Command: `bin/console disclosure:enrichment:import path/to/file.csv`
- Use case: Automated deployment of enrichment updates
- Validation: Pre-flight checks before database insert


**CONFIGURATION VERSIONING:**


Approach: Immutable Audit Trail
- Each CSV import creates new version
- Old versions retained in database (soft delete)
- Version number auto-incremented
- Rollback: Reactivate previous version

Version Table:
- id (PK)
- version_number (integer)
- imported_at (timestamp)
- imported_by (user_id)
- status (enum: active/archived)
- row_count (integer)
- checksum (MD5 hash)

Query Pattern:
```sql
SELECT * FROM disclosure_enrichment_override
WHERE version_id = (SELECT id FROM enrichment_version WHERE status = 'active')
```


**MAINTENANCE WORKFLOW:**


1. Update Global Mappings (Quarterly)
   - Review XBRL taxonomy updates
   - Update unit_mapping.yml
   - Test in DEV
   - Deploy to UAT for validation
   - Deploy to PROD

2. Update Enrichment Data (As Needed)
   - Antonia edits CSV (Excel or Google Sheets)
   - Save as CSV
   - Upload via admin portal
   - System validates format
   - Preview changes before commit
   - Commit to database
   - New version created automatically

3. Update Customer Config (Rare)
   - Customer requests change via support ticket
   - Admin reviews request
   - Update via admin portal
   - Customer notified of change
   - Audit log entry created


**CONFIGURATION DOCUMENTATION:**


Location: Confluence wiki + inline code comments
Contents:
- Configuration file format reference
- Enum value definitions (unitType, positionType, scope)
- CSV schema documentation
- Example configurations
- Troubleshooting guide
---

## 

---

## Document Status


Created: February 12, 2026
Version: 1.0
Status: DRAFT - Ready for Technical Review

Reviewers:
- Tech Lead: [ ] Approved
- Product Manager: [ ] Approved
- InfoSec: [ ] Approved
- DevOps: [ ] Approved
- Domain Expert (Antonia): [ ] Approved

Next Steps:
1. Technical review by engineering leadership
2. Budget approval by finance
3. Resource allocation by engineering VP
4. Kickoff meeting scheduling

@2026 Sphera. All rights reserved.
