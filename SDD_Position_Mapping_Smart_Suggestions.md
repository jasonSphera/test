# Software Design Document (SDD)
# Disclosure Template Enrichment for Smart Position Mapping

**Initiative Name:** Disclosure Template Enrichment for Smart Position Mapping  
**Link to Initiative in ADO:** INITIATIVE [TBD]  
**Document Version:** 1.0  
**Date:** February 12, 2026  
**Author:** Engineering Team  

---

## Overview

**Ready Framework Phase Focus:** Definition of the agreed solution and the areas that impact budget, with well-considered budget costs as the key output.

This document outlines the technical leadership's guidance on the solution direction for implementing an intelligent position mapping system for corporate disclosure responses. It provides the considerations that formed the Ready Framework engineering staff month estimates and budget. This documented thinking can be retrospectively used to learn lessons to help continually improve.

---

## Must-Haves

### Proposed Solution Approach

**Problem Statement:**  
Currently, when completing disclosure questionnaires (ESRS, GRI, TCFD), users must manually select from 100+ positions, reports, and indicators to answer each question. This process is:
- Time-consuming (hours per disclosure)
- Error-prone (wrong positions selected)
- Requires deep system knowledge
- Not scalable as disclosure frameworks expand

**Proposed Solution:**  
Implement a programmatic **Position Filtering and Scoring Service** that enriches disclosure template XML/JSON with metadata attributes, enabling automatic suggestion of the top 1-3 most relevant positions per question with confidence scoring.

**Solution Components:**

#### 1. XML Template Enrichment (Week 1-2)
- Extend disclosure template XML schema to include `<mappingHints>` element
- Add metadata attributes:
  - `unitType` (mass, energy, volume, percentage)
  - `positionType` (emission, energy, water, waste)
  - `scope` (scope1, scope2, scope3, etc.)
  - `requiresCalculation` (true/false - position vs indicator)
  - `periodType` (instant, duration)
- Manual enrichment by domain expert (Antonia) for ESRS E1 questions (~50 questions)
- CSV override mechanism for edge cases

#### 2. Position Filtering Service (Week 2-4)
- Progressive filtering algorithm with 5 stages:
  1. **Position Type Filter:** `position.position_type` → `position_type.name`
  2. **Unit Type Filter:** `position.unit_class_id` → `unit_class.type`
  3. **Scope Filter:** `position.path` or `position.tags` LIKE pattern
  4. **Site Assignment Filter:** `transaction.site_id` IN disclosure sites
  5. **Data Completeness Scoring:** `COUNT(DISTINCT occurrence_date)`
- Reduces candidate pool: 200+ → 50 → 30 → 10 → 5 → 1-3

#### 3. Scoring Algorithm (Week 3-4)
Point-based scoring system:
- Unit match: +30 points
- Position type match: +25 points
- Scope match: +20 points
- Data completeness: +15 points (scaled by %)
- Site assignment: +10 points
- Data recency: +5 points
- User disclosure tag: +50 points (override boost)

**Confidence Levels:**
- HIGH: Score ≥ 90
- MEDIUM: Score 70-89
- LOW: Score < 70

#### 4. API Endpoint (Week 4)
```
POST /api/disclosure/suggest-positions
{
  "questionId": "379907aa-...",
  "disclosureId": 12345,
  "siteIds": [3419, 3420],
  "period": {"start": "2025-01-01", "end": "2025-12-31"}
}

Response:
{
  "suggestions": [
    {
      "positionId": 12345,
      "termStart": 202501,
      "name": "Electricity Location-Based Site A",
      "unit": "tCO2eq",
      "scope": "scope2_location",
      "completeness": 0.98,
      "confidence": "HIGH",
      "score": 105,
      "factors": ["unit_match", "type_match", "scope_match", 
                  "high_completeness", "site_match", "recent_data"]
    }
  ]
}
```

#### 5. UI Integration (Week 5)
- Display top 3 suggestions with confidence badges
- Show match factors as tooltips
- Allow user override with reason tracking
- "Tag for disclosure" feature for future auto-selection
- Bulk apply to similar questions

**Major Features Summary:**
1. ✅ XML schema extension with `<mappingHints>`
2. ✅ Progressive filtering service (5 stages)
3. ✅ Intelligent scoring algorithm
4. ✅ REST API for position suggestions
5. ✅ UI components for suggestion display
6. ✅ User override tracking and learning
7. ✅ Manual CSV override mechanism
8. ✅ Indicator vs Position detection
9. ✅ Formula validation for calculated values
10. ✅ Per-category handling for Scope 3

**Success Metrics:**
- Position reduction: 100+ → 1-3 per question
- Top suggestion accuracy: 80%+ (user selects top suggestion)
- User override rate: <20%
- API response time: <2s per question
- ESRS E1 coverage: 100% questions enriched

---

## Architecture Review

### C4 Model Context

**Impacted Components (Level 2):**

![C4 Architecture Diagram](sdd_c4_architecture.png)

**Component Overview:**
- **Green boxes**: New Position Filtering Service components
- **Yellow boxes**: Existing Disclosure Module (modified)
- **Blue boxes**: Database tables (existing + new column/table)

**Tech Stack Changes:**
- **Backend:** PHP 8.1+ (existing)
- **Database:** MySQL 8.0 (existing schema extension)
- **API:** REST/JSON (existing framework)
- **Template Format:** XML → JSON conversion (existing, enhanced)

**3rd Party Licensing:**
- ❌ No new 3rd party services required
- ❌ No AI/LLM services needed (programmatic approach)
- ✅ All filtering done within existing SCCS infrastructure

**Changes Required:**
1. **disclosure_template table:** Add `enrichment_metadata` JSON column
2. **New table:** `position_suggestion_overrides` (track user selections)
3. **Backend service:** New `PositionFilteringService` class
4. **API route:** `POST /api/disclosure/suggest-positions`
5. **Frontend:** React components for suggestion display

---

### Performance Considerations

**Performance Requirements:**
- API response time: <2 seconds per question
- Bulk suggestion generation: <30 seconds for 50 questions
- Database query optimization: Indexed fields used in all filters
- Caching: Template enrichment metadata cached in memory

**Scalability:**
- Single-tenant architecture (existing SCCS model)
- Each customer has isolated database
- Service scales horizontally with application servers
- No shared state between requests

**Database Indexes Required:**
```sql
-- Existing indexes (already in place):
CREATE INDEX IDX_POSITION_POSITION_TYPE ON position(position_type);
CREATE INDEX IDX_POSITION_UNIT_CLASS_ID ON position(unit_class_id);
CREATE INDEX IDX_TRANSACTION_SITE_ID ON transaction(site_id);
CREATE INDEX IDX_TRANSACTION_OCCURRENCE_DATE ON transaction(occurrence_date);

-- New composite indexes needed:
CREATE INDEX IDX_POSITION_TYPE_UNIT_CLASS ON position(position_type, unit_class_id);
CREATE INDEX IDX_TRANSACTION_POSITION_SITE_DATE 
  ON transaction(position_id, term_start, site_id, occurrence_date);
```

**Performance Testing Plan:**
- Load test with 200+ positions
- Test completeness calculation with 365 days of transactions
- Measure filter stage timing individually
- Optimize slowest filter stage first

**Expected Performance:**
- Filter Stage 1 (Type): <100ms
- Filter Stage 2 (Unit): <150ms
- Filter Stage 3 (Scope): <200ms
- Filter Stage 4 (Site): <300ms
- Filter Stage 5 (Completeness): <500ms
- Scoring & Ranking: <100ms
- **Total: <1.5 seconds** (within 2s target)

---

### Data Strategy

**Data Management:**

**PII Considerations:**
- No PII stored in enrichment metadata
- Position/transaction data already customer-isolated
- User override logs contain only position IDs and timestamps
- No customer data sent to external services

**Data Volumes:**
- Enrichment metadata: ~50KB per disclosure template
- Override logs: ~100 bytes per override event
- Expected: 10-50 overrides per customer per disclosure
- Annual growth: Minimal (~1MB per customer per year)

**Data Retention:**
- Enrichment metadata: Permanent (part of template version)
- Override logs: 2 years (for learning algorithm)
- Suggestion cache: 24 hours (memory only)

**Hosting Region:**
- Follows existing SCCS data residency rules
- EU customers: EU region
- US customers: US region
- No cross-region data transfer

**Data Migration:**
- Existing templates: Backward compatible (optional enrichment)
- New templates: Enriched by default (ESRS, GRI, TCFD)
- Migration script: Backfill existing templates with default enrichment

**Archiving Strategy:**
- Follows existing SCCS retention policies
- Override logs archived after 2 years
- No special archiving requirements for this feature

---

## Project Management

### Scope Approach

**In Scope:**
1. ✅ XML schema extension for enrichment metadata
2. ✅ Manual enrichment for ESRS E1 Climate Change (~50 questions)
3. ✅ Position filtering service (5-stage algorithm)
4. ✅ Scoring algorithm with confidence levels
5. ✅ REST API endpoint for suggestions
6. ✅ UI components for displaying suggestions
7. ✅ User override tracking
8. ✅ Indicator vs Position detection (formula-based)
9. ✅ Data completeness calculation
10. ✅ Documentation and training materials

**Out of Scope (Future Phases):**
1. ❌ AI/ML-based prediction (using LLM to enrich templates)
2. ❌ Automatic enrichment of all frameworks (only ESRS E1 in Phase 1)
3. ❌ Report-level suggestions (only positions/indicators)
4. ❌ Multi-framework cross-referencing
5. ❌ Automated learning from override patterns (manual analysis only)
6. ❌ Natural language query interface
7. ❌ Integration with external data sources
8. ❌ Mobile-optimized UI

**Scope Creep Risks:**
- ⚠️ **Risk:** Requests to enrich all frameworks immediately
  - **Mitigation:** Limit Phase 1 to ESRS E1, prove value, then expand
- ⚠️ **Risk:** "Can we also suggest reports/indicators/formulas?"
  - **Mitigation:** Phase 1 focuses on positions only; indicators in scope only if `requiresCalculation=true`
- ⚠️ **Risk:** "Can we make it smarter with AI?"
  - **Mitigation:** Programmatic approach first; AI considered in Phase 2 after measuring accuracy

**Unknowns to Elaborate Early:**
1. **Unit type mapping:** Do all XBRL unit types map cleanly to SCCS unit classes?
   - **Action:** Week 1 - Analyze XBRL taxonomy and create mapping table
2. **Scope metadata:** Where is scope stored? `position.path`, `position.tags`, or separate table?
   - **Action:** Week 1 - Investigate existing position data structure
3. **Indicator detection:** How reliably can we detect indicators via `position.formula IS NOT NULL`?
   - **Action:** Week 2 - Query production data and validate assumption
4. **Manual enrichment effort:** How long will it take Antonia to enrich 50 ESRS questions?
   - **Action:** Week 2 - Pilot with 10 questions, estimate remaining effort

**Milestones (Incremental Value Delivery):**

![4-Month Implementation Timeline](sdd_milestone_timeline.png)

**Milestone Details:**

**Month 1 (Weeks 1-4): Foundation**
- ✅ XML schema defined and validated
- ✅ Filtering service prototype (hardcoded enrichment)
- ✅ Test with 5 ESRS questions
- **Value:** Proof of concept with measurable accuracy

**Month 2 (Weeks 5-8): Core Service**
- ✅ Manual enrichment of ESRS E1 complete
- ✅ API endpoint deployed to DEV
- ✅ Unit tests for all filter stages
- **Value:** Functional service ready for internal testing

**Month 3 (Weeks 9-12): UI & Integration**
- ✅ React components for suggestion display
- ✅ User override tracking implemented
- ✅ Deployed to UAT for user testing
- **Value:** End-to-end workflow functional

**Month 4 (Weeks 13-16): Production & Refinement**
- ✅ Production deployment
- ✅ Accuracy measurement and tuning
- ✅ Documentation and training
- **Value:** Production-ready feature with measured ROI

---

### Implementation Risks

| # | Type | What Might Happen | Likelihood (1-5) | Severity | How to Manage |
|---|------|-------------------|------------------|----------|---------------|
| 1 | **Risk** | Enrichment metadata is insufficient - still too many position candidates after filtering | 3 | Medium | Start with most common questions. Add more attributes incrementally based on testing. CSV override mechanism for edge cases. |
| 2 | **Risk** | XBRL unit types don't map cleanly to SCCS unit classes | 4 | Medium | Create comprehensive mapping table in Week 1. Document gaps. Use fuzzy matching with confidence penalty for ambiguous cases. |
| 3 | **Risk** | Scope metadata missing or inconsistent in existing position data | 3 | Medium | Infer scope from `position.path` naming conventions. Add scope tagging to position creation workflow for new positions. |
| 4 | **Risk** | Data completeness calculation is too slow for real-time API | 3 | High | Pre-calculate completeness scores nightly for common periods. Cache results. Optimize with indexed date queries. |
| 5 | **Risk** | Users reject top suggestions (low accuracy) | 2 | High | Set accuracy target at 80% (acceptable). Track overrides. Iterate on scoring weights based on patterns. User tags provide manual improvement path. |
| 6 | **Risk** | Manual enrichment takes longer than estimated (Antonia's time) | 3 | Medium | Pilot with 10 questions first. Adjust timeline if needed. Consider batch enrichment across multiple templates. Prioritize high-usage questions. |
| 7 | **Risk** | Indicator detection via formula field is unreliable | 2 | Medium | Query production data in Week 2 to validate. Have fallback logic based on position naming patterns. |
| 8 | **Opportunity** | High accuracy achieved early - expand to more frameworks | 2 | Low | Document success. Prepare business case for Phase 2 (GRI, TCFD enrichment). |
| 9 | **Risk** | Composite keys (position_id, term_start) not handled correctly in queries | 3 | High | Validate all JOIN conditions include both keys. Create database helper functions. Extensive testing with historical data. |
| 10 | **Risk** | UI/UX doesn't clearly communicate confidence levels to users | 3 | Medium | Include UX review in Week 5. User testing in UAT phase. Visual confidence badges and tooltips explaining match factors. |

---

### Dependencies

**Needs Sphera ARB - Architecture Review Board:** **YES**

**Reason:**  
While this solution is contained within SCCS (single solution family), it introduces:
1. New database schema patterns (JSON metadata columns)
2. New API patterns (suggestion/scoring services)
3. Potential future integration with shared enrichment services
4. Performance implications requiring infrastructure review

**ARB Review Topics:**
- Database schema extension approach
- API design patterns for suggestion services
- Caching strategy for enrichment metadata
- Potential for cross-solution reuse (other modules needing position suggestions)

---

**Area Dependencies:**

| Area | Involvement Needed? | Why |
|------|---------------------|-----|
| **InfoSec** | **YES** | User override tracking stores selection patterns. Privacy review needed for GDPR compliance. No PII expected but must validate. |
| **DevOps** | **YES** | New database indexes may impact deployment. Need migration scripts for schema changes. Performance testing infrastructure required. |
| **Platform** | **NO** | Fully self-contained within SCCS. No shared platform services used. (Future: Could leverage platform AI if Phase 2 includes AI enrichment) |
| **UX** | **YES** | New UI components for displaying suggestions. Need design review for confidence badges, match factors, and override workflow. Accessibility considerations. |
| **Other Solution Family** | **NO** | Disclosure is SCCS-specific. No cross-solution dependencies. (Future: GaBi/LCA integration may benefit from similar suggestion logic) |
| **QA (end-to-end testing)** | **YES** | Need E2E tests for full disclosure workflow with suggestions. Performance testing for filter stages. Accuracy validation with real disclosure data. |
| **Product Management** | **YES** | Define acceptance criteria for accuracy targets. Prioritize which frameworks to enrich first. User training and communication plan. |
| **Documentation** | **YES** | Update user guides for suggestion workflow. Developer docs for enrichment process. API documentation for suggestion endpoint. |

---

## Solution Budget

### Engineering Staff Months Effort

![Budget Breakdown by Component](sdd_budget_breakdown.png)

**Effort Breakdown by Component:**

| Component | Best Case | Mid Case | Worst Case | Notes |
|-----------|-----------|----------|------------|-------|
| **XML Schema Extension** | 0.25 SM | 0.5 SM | 1.0 SM | XSD updates, validation, backward compatibility testing |
| **Filtering Service** | 1.0 SM | 1.5 SM | 2.5 SM | 5-stage filter implementation, database query optimization |
| **Scoring Algorithm** | 0.5 SM | 1.0 SM | 1.5 SM | Point calculation, confidence levels, edge cases |
| **API Endpoint** | 0.5 SM | 0.75 SM | 1.0 SM | REST endpoint, request validation, response formatting |
| **UI Components** | 1.0 SM | 1.5 SM | 2.5 SM | React components, confidence badges, override workflow |
| **Manual Enrichment Support** | 0.25 SM | 0.5 SM | 1.0 SM | CSV tools, validation scripts, enrichment UI for Antonia |
| **Testing (Unit + Integration)** | 1.0 SM | 1.5 SM | 2.5 SM | Filter stage tests, API tests, UI tests, E2E scenarios |
| **Performance Optimization** | 0.5 SM | 1.0 SM | 2.0 SM | Database indexing, query optimization, caching |
| **Documentation** | 0.25 SM | 0.5 SM | 0.75 SM | API docs, user guides, developer docs, training materials |
| **Buffer (unknowns, rework)** | 0.5 SM | 1.0 SM | 2.0 SM | Schema discovery, data quality issues, accuracy tuning |

**Total Engineering Effort:**

| Milestone | Best | Mid | Worst | Max Engineers | Min Duration | Comments |
|-----------|------|-----|-------|---------------|--------------|----------|
| **Phase 1 (ESRS E1)** | 5.75 SM | 9.25 SM | 16.25 SM | 3 engineers | 3 months | Mid case assumes 1 senior + 2 mid-level engineers. Worst case accounts for learning curve on database schema complexity. |

**Budget Request (Mid Case + 40% buffer):**
- Mid case: 9.25 SM
- With 40% buffer: **13 SM** (round up for budgeting)

**Team Composition (Recommended):**
- 1 Senior Backend Engineer (filtering service, API) - 4 months
- 1 Mid-level Full-Stack Engineer (UI, integration) - 3 months
- 1 Mid-level Backend Engineer (testing, optimization) - 3 months
- **Average: ~3.3 months with 3 engineers**

---

### Non-Engineering Staff Months Effort

| Milestone | DevOps | InfoSec | UX | Project Manager | Technical Writer | Platform | QA Lead | Product Manager |
|-----------|--------|---------|----|-----------------|--------------------|----------|---------|-----------------|
| **Phase 1** | 0.5 SM | 0.25 SM | 0.75 SM | 0.5 SM | 0.25 SM | 0 SM | 0.5 SM | 0.25 SM |

**Notes:**
- **DevOps (0.5 SM):** Database migration scripts, index creation, deployment support
- **InfoSec (0.25 SM):** Privacy review of override tracking, data isolation validation
- **UX (0.75 SM):** Design suggestion display components, confidence badges, override workflow
- **Project Manager (0.5 SM):** Coordination, timeline tracking, stakeholder communication
- **Technical Writer (0.25 SM):** User documentation, API docs, training materials
- **QA Lead (0.5 SM):** E2E test planning, accuracy validation, performance testing
- **Product Manager (0.25 SM):** Acceptance criteria, prioritization, user communication

**Total Non-Engineering:** **3.0 SM**

**Combined Total Budget:** **13 SM (engineering) + 3 SM (non-engineering) = 16 SM**

---

## Should-Haves

### Service Scaling

**Scaling Approach:**
- **Architecture:** Single-tenant (existing SCCS model)
- **Scaling Factor:** Per-customer database isolation
- **Horizontal Scaling:** Add application servers as needed
- **No shared state:** Each request is stateless

**Predicted Load (Next 3 Years):**

| Year | Active Customers | Disclosures/Year | Questions/Disclosure | Total Suggestions/Year | Concurrent Users (Peak) |
|------|------------------|------------------|----------------------|------------------------|-------------------------|
| 2026 | 50 | 2 per customer | 50 | 5,000 | 10 |
| 2027 | 100 | 2 per customer | 50 | 10,000 | 20 |
| 2028 | 200 | 3 per customer | 75 | 45,000 | 40 |

**Peak Load Calculation:**
- Worst case: 40 concurrent users × 50 questions = 2,000 suggestions in <1 hour
- Average: ~33 suggestions/minute
- With 2s per suggestion: Well within single server capacity

**Scaling Path:**
1. **Current (2026):** Single application server per region
2. **2027:** Add load balancer, 2 application servers
3. **2028:** 3-4 application servers, read replicas for database

**Extreme Scaling (10,000 users / 1000s API calls/second):**
- Not anticipated for this feature
- Disclosure completion is infrequent (2-3x per year per customer)
- Not a high-throughput API requirement
- If needed: Implement Redis caching layer for suggestions

**Multi-Tenant Consideration:**
- Current SCCS is single-tenant by design (customer data isolation)
- This feature maintains that architecture
- No changes to multi-tenancy model

---

### Resiliency

**Service Level Expectations:**

**Criticality:** **Medium**
- Disclosure module can function without suggestions (manual position selection)
- Loss of service does not block disclosure completion
- Users can continue working with degraded experience (no auto-suggestions)

**Impact of Service Down:**
- ❌ No automatic position suggestions
- ✅ Manual position selection still available
- ✅ Existing disclosure responses unchanged
- ✅ Other SCCS modules unaffected

**Resilience Strategy:**

1. **Graceful Degradation:**
   ```javascript
   // Frontend fallback logic
   try {
     const suggestions = await fetchPositionSuggestions(questionId);
     displaySuggestions(suggestions);
   } catch (error) {
     console.error('Suggestion service unavailable:', error);
     displayManualSelectionOnly(); // Fallback to dropdown
   }
   ```

2. **Health Endpoint:**
   ```
   GET /api/health/position-suggestions
   Response:
   {
     "status": "healthy",
     "checks": {
       "database": "ok",
       "filterService": "ok",
       "averageResponseTime": "1.2s"
     }
   }
   ```

3. **Timeout Protection:**
   - API timeout: 5 seconds (fail fast)
   - Retry logic: 1 retry with exponential backoff
   - Circuit breaker: After 5 failures in 60s, open circuit for 2 minutes

4. **Monitoring:**
   - Track suggestion API success rate (target: >99%)
   - Alert if average response time >3s
   - Alert if error rate >5%
   - Dashboard: Suggestions/hour, accuracy rate, override rate

5. **Database Resilience:**
   - Use read replicas for filtering queries (no writes)
   - Connection pooling with retry logic
   - Fallback to cached suggestions if database unavailable

**Service Level Objective (SLO):**
- **Availability:** 99% (acceptable downtime: 7 hours/month)
- **Response Time:** 95th percentile <2s
- **Error Rate:** <1% of requests

---

### Hosting Service Costs

**Cost Impact Analysis:**

**Current Baseline:**
- SCCS infrastructure: Existing application servers and databases
- No incremental hosting costs for DEV/UAT (shared resources)
- Production: Cost scales with customer count (single-tenant)

**New Costs (Per Environment):**

| Component | Platform | Service Name | Meter Name | Current Monthly ($k) | New Delta ($k) | Comment |
|-----------|----------|--------------|------------|----------------------|----------------|---------|
| **Application Server** | AWS EC2 / Azure VM | SCCS App Server | Compute Hours | [Existing] | $0 | No additional servers needed initially. Filtering service runs on existing app servers. |
| **Database Storage** | MySQL | Customer DB | Storage GB | [Existing] | +$0.05 per customer | Enrichment metadata adds ~50KB per template. Negligible storage increase. |
| **Database Indexes** | MySQL | Customer DB | IOPS | [Existing] | +$0.10 per customer | New composite indexes may slightly increase IOPS. Minimal impact. |
| **API Gateway** | Existing | SCCS API | Requests | [Existing] | $0 | New endpoint uses existing API infrastructure. |
| **Monitoring** | Existing | AppDynamics/New Relic | - | [Existing] | $0 | Add new metrics to existing monitoring. |

**Total Incremental Cost:**
- **Per Customer (Production):** ~$0.15/month ($1.80/year)
- **Dev/UAT:** Negligible (shared infrastructure)
- **50 Customers:** ~$7.50/month = **~$90/year**
- **100 Customers:** ~$15/month = **~$180/year**
- **200 Customers:** ~$30/month = **~$360/year**

**Cost Management:**
- No new infrastructure required initially
- Scales linearly with customer count (single-tenant model)
- Monitoring for inefficient queries that increase IOPS

**ROI Impact:**
- Estimated time savings: 2 hours per disclosure × $50/hour = $100 saved per disclosure
- With 2 disclosures/year: $200 saved per customer/year
- Cost: $1.80/year per customer
- **Net benefit:** $198.20 per customer per year
- **ROI:** >10,000%

**Tenant Cost Impact:**
- Current total SCCS cost per tenant: ~$500-1000/month (varies by usage)
- This feature adds: ~$0.15/month per tenant
- **Impact:** 0.015% - 0.03% (negligible)

---

### 3rd Party Costs

**No New 3rd Party Costs Required**

| Service Type | Current Provider | New Provider | Cost | Comment |
|--------------|------------------|--------------|------|---------|
| **AI/LLM Services** | None | None | $0 | Programmatic approach - no AI required |
| **Data Services** | None | None | $0 | Uses existing SCCS customer data |
| **XBRL Taxonomy** | ESRS Foundation (public) | Same | $0 | Public standard, no licensing fees |
| **Monitoring Tools** | AppDynamics/New Relic | Same | $0 | Existing monitoring extended |
| **Development Tools** | Existing IDE/Git/CI-CD | Same | $0 | No new tooling required |

**Future Considerations (Phase 2 - AI Enrichment):**
- If AI-based enrichment is added in Phase 2:
  - LLM API costs (OpenAI, Azure OpenAI, etc.)
  - Estimated: $0.10 - $0.50 per template enrichment
  - One-time cost per template (not per-disclosure)
  - 50 templates × $0.30 = ~$15 one-time cost
- Phase 1 has **zero 3rd party costs**

---

## Could-Haves

### Deployment Strategy

**Deployment Regions:**

| Region | Customers | Priority | Phase 1 Deployment |
|--------|-----------|----------|--------------------|
| **EU (Ireland)** | 30 | High | ✅ Yes |
| **US (Virginia)** | 15 | High | ✅ Yes |
| **APAC (Singapore)** | 5 | Medium | ⚠️ If time permits |

**Deployment Flow:**

![Deployment Flow - DEV to Production](sdd_deployment_flow.png)

**Deployment Approach:**

**Phase 1 (ESRS E1 - EU Focus):**
1. ✅ Deploy to EU region first (majority ESRS users)
2. ✅ Deploy to US region (some EU subsidiaries reporting)
3. ⚠️ APAC region: Deferred unless strong customer demand

**Environment Progression:**
1. **DEV:** Week 4 (filtering service ready)
2. **UAT:** Week 8 (manual enrichment complete)
3. **STAGE:** Week 11 (after internal user testing)
4. **PROD:** Week 14 (after UAT validation)

**Rollout Strategy:**

**Approach:** Feature Toggle (Gradual Rollout)
```php
// Feature toggle in SCCS configuration
$config['feature_flags'] = [
    'disclosure_position_suggestions' => [
        'enabled' => true,
        'frameworks' => ['ESRS'], // Only ESRS in Phase 1
        'customers' => [123, 456, 789], // Pilot customers
        'questions' => ['E1.*'], // Only E1 Climate questions
    ]
];
```

**Rollout Phases:**
1. **Week 14-15:** Enable for 3 pilot customers (friendly users)
2. **Week 16:** Gather feedback, adjust scoring if needed
3. **Week 17:** Enable for all ESRS E1 questions
4. **Week 18:** Monitor accuracy, iterate on overrides
5. **Week 19:** Full production release (all ESRS customers)

**Availability to Subscribers:**
- **Licensing:** Included in SCCS Disclosure module (no extra charge)
- **Configuration:** Enabled by default for ESRS templates
- **Opt-out:** Can disable suggestions via user preferences
- **Discovery:** In-app notification when first opening ESRS disclosure

**Deployment Downtime:**
- **Database migration:** <5 minutes (add columns, indexes)
- **Application deployment:** Zero-downtime (rolling deployment)
- **Impact:** No user-facing downtime expected

**On-Premise Consideration:**
- ⚠️ **Not applicable in Phase 1**
- SCCS is cloud-only (AWS/Azure)
- If on-prem required in future: Same deployment package, self-managed

---

### Test Maintainability

**Test Automation Coverage:**

**Target Coverage:**
- **Unit Tests:** 80% code coverage
- **Integration Tests:** 100% filter stages
- **API Tests:** 100% endpoints
- **E2E Tests:** Top 5 user journeys

**Test Pyramid Structure:**

| Layer | Percentage | Count | Focus |
|-------|------------|-------|-------|
| **E2E Tests** | 10% | ~10 tests | User journeys, full workflow |
| **Integration Tests** | 30% | ~30 tests | API + Database integration |
| **Unit Tests** | 60% | ~100 tests | Business logic, filter stages |

**Unit Tests (Week 2-4, ongoing):**
- ✅ Filter stage isolation (mock database)
- ✅ Scoring algorithm with edge cases
- ✅ Attribute extraction from XML/JSON
- ✅ Confidence level calculation
- ✅ Edge cases: Empty data, null values, invalid input

**Integration Tests (Week 3-5):**
- ✅ End-to-end filtering with real database
- ✅ API endpoint with real customer data
- ✅ Data completeness calculation accuracy
- ✅ Performance benchmarks (2s target)

**E2E Tests (Week 8-10):**
1. **Happy Path:** User opens disclosure, sees suggestions, selects top suggestion
2. **Override Path:** User rejects top 3, selects manually, adds disclosure tag
3. **No Suggestions:** Question has no enrichment, falls back to manual
4. **Bulk Apply:** User selects suggestion, applies to similar questions
5. **Indicator Detection:** Calculated question returns indicator, not position

**Test Data Strategy:**
- **Synthetic Data:** Generate positions with known attributes for filtering
- **Anonymized Production Data:** Subset of real customer data (10 customers)
- **Edge Cases:** Missing data, multiple scopes, ambiguous unit types

**Automated Testing Debt:**
- ⚠️ **Current State:** Disclosure module has ~60% test coverage
- ⚠️ **Debt:** Some legacy code paths untested
- ✅ **Approach:** Focus new tests on suggestion feature only
- ✅ **Future:** Gradually increase coverage in surrounding code

**Test Execution:**
- **CI/CD:** Unit + Integration tests run on every commit
- **Nightly:** E2E tests + Performance benchmarks
- **Pre-Release:** Full regression suite (all tests)

**Test Maintenance:**
- ✅ Tests co-located with code (easy to find)
- ✅ Test naming convention: `test_filterStage_unitType_returnsExpectedPositions()`
- ✅ Test data factories for easy fixture creation
- ⚠️ E2E tests may need updates if UI changes frequently

---

### Disaster Recovery Strategy

**Backup Strategy:**

**Database Backups (Existing SCCS Process):**
- Daily automated backups (retained 30 days)
- Weekly backups (retained 90 days)
- Monthly backups (retained 1 year)
- Backup includes enrichment metadata (part of disclosure_template table)

**New Backup Items (This Feature):**
- `disclosure_template.enrichment_metadata` column (JSON)
- `position_suggestion_overrides` table (user selections)
- No special backup requirements - follows existing SCCS backup policies

**Restore Process:**
1. Standard SCCS database restore from backup
2. Enrichment metadata restored automatically (part of template)
3. Override logs restored (for learning algorithm)
4. No application code changes needed for restore

**Multi-Region Backup:**
- EU backups: Stored in EU region (GDPR compliant)
- US backups: Stored in US region
- Cross-region replication: Optional (customer-specific)

---

**Failover Strategy:**

**Primary Region Failure:**
- **Detection:** Health checks every 60s, alert after 3 failures
- **Failover:** Manual (DR region activation by DevOps)
- **RTO (Recovery Time Objective):** 4 hours
- **RPO (Recovery Point Objective):** 24 hours (last backup)

**Failover Process:**
1. Detect primary region down (monitoring alerts)
2. Promote DR region database (latest backup)
3. Update DNS to point to DR region
4. Validate suggestion service in DR region
5. Communicate to users (in-app banner)

**Partial Outage Handling:**

| Component Down | Impact | Mitigation |
|----------------|--------|------------|
| **Database** | No suggestions | Graceful degradation to manual selection |
| **Application Server** | API unavailable | Load balancer redirects to healthy server |
| **Filtering Service** | Errors returned | Frontend fallback to manual selection |
| **Network** | Regional outage | Failover to DR region |

---

**Monitoring & Alerting:**

**System Failure Detection:**
- ✅ API health endpoint: `/api/health/position-suggestions`
- ✅ Database connection check every 60s
- ✅ Average response time >3s for 5 minutes → Alert
- ✅ Error rate >5% for 10 minutes → Alert
- ✅ Zero successful suggestions in 1 hour → Critical alert

**Alert Channels:**
- PagerDuty (critical alerts)
- Slack #sccs-alerts (warnings)
- Email (summaries)

**On-Call Rotation:**
- Follow existing SCCS on-call schedule
- No dedicated on-call for this feature

---

**DR Testing:**

**Testing Frequency:**
- DR failover test: Quarterly
- Database restore test: Monthly
- Health check validation: Weekly (automated)

**DR Test Plan:**
1. Schedule maintenance window (off-peak hours)
2. Simulate primary region failure
3. Execute failover to DR region
4. Validate suggestion API in DR region
5. Test with sample disclosure (5 questions)
6. Fail back to primary region
7. Document lessons learned

**Test Success Criteria:**
- ✅ Failover completes within 4 hours
- ✅ Suggestion API returns valid results
- ✅ Data completeness matches primary region
- ✅ No data loss (override logs intact)

---

**User Communication During DR:**

**In-App Notification:**
```
⚠️ System Maintenance
Disclosure suggestions may be temporarily unavailable. 
You can still complete your disclosure using manual position selection.
Estimated restoration: [Time]
```

**Status Page:**
- Update SCCS status page (status.sccs.com or similar)
- Show service status: Operational / Degraded / Outage
- ETA for restoration

**Post-Incident Communication:**
- Email to affected customers
- Summary of incident, root cause, resolution
- Assurance of data integrity

---

### Onboarding New Customers/Users Strategy

**Customer Provisioning Process:**

**Step-by-Step Onboarding:**

1. **Customer Signup (Existing Process)**
   - Customer subscribes to SCCS with Disclosure module
   - Database provisioned (existing SCCS process)
   - No special provisioning for suggestion feature

2. **Template Selection**
   - Customer selects frameworks (ESRS, GRI, TCFD)
   - Enriched templates automatically available for ESRS
   - Other frameworks: Manual selection until enriched

3. **Feature Discovery**
   - In-app notification on first disclosure creation:
     ```
     🎉 New Feature: Smart Position Suggestions
     SCCS now automatically suggests the most relevant positions 
     for your disclosure questions based on your data.
     [Learn More] [Got It]
     ```

4. **First Use**
   - User opens first ESRS question
   - Suggestions appear automatically (if enriched)
   - Tooltip explains confidence badges and match factors
   - User can accept, override, or manually select

5. **Training & Documentation**
   - Link to knowledge base article
   - Video tutorial (optional, 2 minutes)
   - In-app help tooltips

**Responsible Parties:**

| Stage | Responsible | Timeline |
|-------|-------------|----------|
| Database Provisioning | DevOps / Automation | <1 hour (existing) |
| Template Deployment | Backend (Automated) | Immediate |
| Feature Toggle Enablement | Product/Support | Week 1 of subscription |
| User Training | Customer Success | Week 1-2 of onboarding |
| Ongoing Support | Support Team | Ongoing |

**SLAs:**
- Database provisioning: <4 hours (existing SLA)
- Feature availability: Immediate (no manual steps)
- First support response: 24 hours (existing SLA)

**Rollback Procedures:**

**If Issues Discovered Post-Deployment:**
1. Disable feature toggle for affected customer
2. User automatically reverts to manual selection
3. Investigate issue (bad suggestions, performance, etc.)
4. Fix and re-enable for customer

**Rollback Process:**
```bash
# Disable suggestions for specific customer
php bin/console feature:disable disclosure_position_suggestions --customer-id=123

# Disable for all customers (emergency)
php bin/console feature:disable disclosure_position_suggestions --global
```

**Monitoring Onboarding Success:**
- **Metric 1:** % of new customers using suggestions (target: >80%)
- **Metric 2:** Average time to complete first disclosure (target: -30%)
- **Metric 3:** User satisfaction score (target: >8/10)
- **Metric 4:** Support tickets related to suggestions (target: <5% of users)

---

### Configuration Management Approach

**Configuration Complexity:**

**Spectrum:** **Tenant-specific configuration with user preferences**

**Configuration Levels:**

1. **System-Wide (Default)**
   - Scoring algorithm weights (unit=30, type=25, scope=20, ...)
   - Confidence thresholds (HIGH≥90, MEDIUM≥70)
   - API timeout settings (5 seconds)
   - Feature toggle defaults

2. **Template-Level (Per Framework)**
   - Enrichment metadata per question (XBRL, mappingHints)
   - Framework-specific rules (ESRS vs GRI)
   - Question grouping for bulk apply

3. **Tenant-Level (Per Customer)**
   - Feature toggle override (enable/disable for customer)
   - Custom scoring weights (if requested - rare)
   - Disclosure tag preferences

4. **User-Level (Individual Preferences)**
   - Show/hide suggestions
   - Confidence threshold filter (only show HIGH confidence)
   - Auto-apply suggestions (trust top suggestion always)

**Configuration Storage:**

```sql
-- System-wide config
CREATE TABLE system_config (
  config_key VARCHAR(255) PRIMARY KEY,
  config_value JSON,
  updated_at TIMESTAMP
);

-- Tenant-level config
CREATE TABLE tenant_config (
  tenant_id INT,
  config_key VARCHAR(255),
  config_value JSON,
  PRIMARY KEY (tenant_id, config_key)
);

-- User preferences
CREATE TABLE user_preferences (
  user_id INT,
  preference_key VARCHAR(255),
  preference_value JSON,
  PRIMARY KEY (user_id, preference_key)
);
```

**Configuration Management:**

**Intellectual Property Considerations:**
- ✅ **Enrichment metadata (XBRL mappingHints):** Version controlled in Git
  - Can be considered IP (business logic for mapping)
  - Shared across all customers (standard ESRS interpretation)
  - Changes require code review and testing

- ⚠️ **Customer-specific overrides:** Stored in database (not Git)
  - Customer-specific rules (if any)
  - Not considered shared IP

**Environment Propagation:**

**DEV → UAT → STAGE → PROD**

1. **Enrichment Metadata (Template Changes):**
   - Developer updates XML template in Git
   - CI/CD pipeline validates XML schema
   - Deployment script imports templates to environment
   - Automated on every deployment

2. **System Config Changes:**
   - Config changes in Git (YAML file)
   - Applied via migration scripts
   - Can be rolled back via Git revert

3. **User Preferences:**
   - User-specific, not propagated across environments
   - UAT has isolated user preferences

**Maintenance Strategy:**

**Who maintains what:**

| Config Type | Owner | Change Frequency | Process |
|-------------|-------|------------------|---------|
| **Enrichment Metadata** | Product + Engineering | Per template version | Git PR → Review → Deploy |
| **Scoring Weights** | Engineering (based on data) | Quarterly tuning | Config file change → Deploy |
| **Feature Toggles** | Product/Support | Per customer request | Admin UI or CLI |
| **User Preferences** | End users | On-demand | Self-service via UI |

**Tooling:**

1. **Template Enrichment Tool (for Antonia/Product):**
   ```bash
   # CSV to XML enrichment script
   php bin/console disclosure:enrich-template \
     --framework=ESRS \
     --csv-file=enrichment_E1.csv \
     --output=disclosure_esrs_2024_v1.6.xml
   ```

2. **Feature Toggle CLI:**
   ```bash
   # Enable/disable per customer
   php bin/console feature:toggle \
     disclosure_position_suggestions \
     --customer-id=123 \
     --enable
   ```

3. **Config Validation:**
   ```bash
   # Validate enrichment metadata before deployment
   php bin/console disclosure:validate-enrichment \
     --file=disclosure_esrs_2024_v1.6.xml
   ```

**Configuration Documentation:**
- ✅ All config keys documented in wiki
- ✅ Example values provided
- ✅ Change history tracked in Git
- ✅ Impact analysis documented per config change

---

## Appendix: Visual Decision Tree Documentation

**Reference Diagrams (Generated with Mermaid CLI):**

All decision tree diagrams are available in:
`/Users/joakes/Projects/sofi/learned/disclosure/design/`

### Overview Diagrams (4K Resolution for Clarity)

#### 1. Decision Tree Overview
![System Overview - Position Filtering Architecture](decision_tree_overview_v2.png)

**Shows:**
- System architecture with attribute toolkit
- 4-stage filtering process with database sources
- Scoring factors with point allocations (30, 25, 20, 15, 10, 5 points)
- Output structure with confidence levels

#### 2. Filter Progression
![Progressive Filter Reduction - 200 to 3 Positions](filter_progression_v2.png)

**Shows:**
- Progressive filtering visualization with SQL queries
- Position count reduction at each stage: 200 → 50 → 30 → 10 → 5 → 3
- Real examples of positions with data completeness
- Excluded pool tracking

#### 3. Decision Matrix
![Attribute-Based Routing and Branching Logic](decision_matrix_v2.png)

**Shows:**
- Attribute-based routing logic (Position vs Indicator)
- Scope/type filtering decision tree
- Formula validation for calculated values
- Complete scoring and validation flow

### Detailed Decision Trees (High Resolution Examples)

#### 4. Example 1: Scope 2 Location-Based Emissions
![Decision Tree: Scope 2 Emissions](decision_tree_1_scope2_emissions_v2.png)

**Question Type:** Gross Scope 2 GHG emissions (location-based)  
**Complexity:** Medium  
**Shows:**
- Complete SQL queries per filter stage with real table/field names
- Real position examples with IDs (12345, 12346, 12347)
- Scoring breakdown: 105 points = HIGH confidence
- Full JSON output structure with all fields

**Use Case:** Standard emission questions with direct measurement data

#### 5. Example 2: Renewable Energy Calculation
![Decision Tree: Energy Percentage Calculation](decision_tree_2_energy_calculation_v2.png)

**Question Type:** Percentage of renewable energy  
**Complexity:** High  
**Shows:**
- Indicator detection via `position.formula IS NOT NULL`
- Formula validation logic (SUM renewable / SUM total * 100)
- Input position verification with 5 positions
- Data completeness per input (100%, 95%, 88%, etc.)
- Calculated value in response (42.5%)

**Use Case:** Questions requiring calculated values (percentages, ratios, intensities)

#### 6. Example 3: Scope 3 Upstream Multi-Category
![Decision Tree: Scope 3 Upstream by Category](decision_tree_3_scope3_upstream_v2.png)

**Question Type:** Scope 3 upstream emissions by category  
**Complexity:** Very High  
**Shows:**
- Category grouping per GHG Protocol (Cat 1, 3, 4)
- Emission factor validation (GWP AR5 vs AR6)
- Per-category scoring with multiple suggestions
- Multi-position output structure (1-2 per category)

**Use Case:** Complex multi-category questions with potential calculated emissions

**Database Schema Validation:**
- All SQL queries validated against actual Entity classes
- Field names match: `position.position_type`, `transaction.occurrence_date`, etc.
- Foreign key relationships verified
- Composite keys (position_id, term_start) handled correctly

**Documentation:**
- `DECISION_TREE_POSITION_MAPPING_ANALYSIS.md` - Complete analysis
- `DECISION_TREE_V2_ENHANCEMENTS.md` - V2 improvements
- `DATABASE_SCHEMA_VALIDATION_AND_V2_COMPLETE.md` - Schema validation
- `DECISION_TREE_VISUAL_INDEX.md` - Visual diagram index

---

## Approval & Sign-Off

**Technical Leadership Approval:**

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Engineering VP | [Name] | _____________ | ______ |
| Principal Engineer | [Name] | _____________ | ______ |
| Solution Architect | [Name] | _____________ | ______ |

**Cross-Functional Approval:**

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Director | [Name] | _____________ | ______ |
| DevOps Manager | [Name] | _____________ | ______ |
| InfoSec Lead | [Name] | _____________ | ______ |
| UX Lead | [Name] | _____________ | ______ |

**ARB Approval:**

| ARB Member | Name | Approval Status | Date |
|------------|------|-----------------|------|
| ARB Chair | [Name] | ☐ Approved ☐ Conditional ☐ Rejected | ______ |
| Architecture | [Name] | ☐ Approved ☐ Conditional ☐ Rejected | ______ |
| Security | [Name] | ☐ Approved ☐ Conditional ☐ Rejected | ______ |

**Conditions (if conditional approval):**
1. _________________________________________
2. _________________________________________
3. _________________________________________

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | 2026-02-12 | Engineering Team | Initial draft based on BridgeTheGap.md requirements |
| 0.2 | 2026-02-12 | Engineering Team | Added database schema validation, decision tree diagrams |
| 1.0 | 2026-02-12 | Engineering Team | Final SDD ready for review |

---

**Document Status:** ✅ **Ready for Review**  
**Next Steps:** ARB submission, cross-functional review, approval sign-off  
**Target Implementation Start:** Week 1 of approved budget

---

**END OF DOCUMENT**
