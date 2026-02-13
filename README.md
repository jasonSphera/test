# Corporate Disclosure Smart Position Mapping - Solution Wiki

**Version:** 1.0  
**Last Updated:** February 13, 2026  
**Status:** 🟢 Active Development  
**Initiative:** AI-Assisted Corporate Disclosure

---

## 🎯 Overview

This wiki provides a comprehensive guide to the **Smart Position Mapping** solution for corporate disclosure questionnaires (ESRS, GRI, TCFD). The solution uses XML/JSON template enrichment and intelligent filtering to automatically suggest the most relevant positions, reports, and indicators for each disclosure question.

### The Problem We're Solving

When completing disclosure questionnaires, users currently face:
- ❌ **Manual Selection:** Reviewing 200+ positions per question
- ❌ **Time-Intensive:** 30+ minutes per question
- ❌ **Error-Prone:** Wrong positions frequently selected
- ❌ **Knowledge-Dependent:** Requires deep system expertise
- ❌ **Not Scalable:** Growing disclosure frameworks compound the issue

### The Solution

A programmatic **Position Filtering and Scoring Service** that:
- ✅ **Auto-Suggests:** Top 1-3 most relevant positions per question
- ✅ **High Accuracy:** 95% success rate with proper enrichment
- ✅ **Fast:** <2 second response time
- ✅ **Transparent:** Confidence scores with explanations
- ✅ **Scalable:** Works across all disclosure frameworks

---

## 📚 Key Documentation

### 1. Software Design Document (SDD)

**📄 [SDD_Position_Mapping_Smart_Suggestions.md](SDD_Position_Mapping_Smart_Suggestions.md)**

**Purpose:** Technical design specification for implementing the smart mapping solution

**What's Inside:**
- 🏗️ **Architecture Overview** - C4 diagrams and system components
- 🔧 **Technical Implementation** - 5-stage filtering algorithm
- 📊 **Scoring System** - Point-based confidence scoring
- 🗂️ **Database Schema** - Table structures and relationships
- 🚀 **Deployment Plan** - DEV → TEST → UAT → Production
- 💰 **Budget & Timeline** - 4-month implementation at $95K
- 🧪 **Testing Strategy** - Unit, integration, and UAT tests

**Key Sections:**
```
├── Problem Statement & Solution Approach
├── XML Template Enrichment Strategy
├── Position Filtering Service (5-stage algorithm)
├── Scoring Algorithm (90-point system)
├── API Endpoint Specifications
├── UI Integration Pattern
├── Database Schema Design
├── Deployment & Rollout Plan
└── Success Metrics & KPIs
```

**Visual Diagrams:**
- ![C4 Architecture](images/sdd_c4_architecture.png) - System architecture
- ![Timeline](images/sdd_milestone_timeline.png) - 4-month implementation plan
- ![Budget](images/sdd_budget_breakdown.png) - Cost breakdown
- ![Deployment](images/sdd_deployment_flow.png) - Release workflow

---

### 2. XML to JSON Enrichment Analysis (V2)

**📄 [XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md](XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md)**

**Purpose:** Comprehensive visual guide to template enrichment with real-world examples

**What's Inside:**
- 🎨 **6 High-Resolution Diagrams** (4K quality) showing data flows
- 🔄 **Before/After Comparisons** demonstrating enrichment impact
- 🗺️ **Attribute Mapping Matrices** with database field mappings
- 📋 **CSV Override Mechanism** for edge case handling
- 📊 **Question Type Coverage** for all ESRS scenarios
- 📐 **Multi-Column Handling** for complex table questions

**Key Sections:**
```
├── Executive Summary (Visual First Approach)
├── Complete Data Flow: XML → JSON → Database
├── Before/After Impact Analysis
├── Attribute Mapping Matrix
├── Enrichment Attribute Specifications
│   ├── unitType (mass, energy, volume, percentage)
│   ├── positionType (emission, energy, water, waste)
│   ├── scope (scope1, scope2, scope3, etc.)
│   ├── requiresCalculation (position vs indicator)
│   └── periodType (instant, duration)
├── CSV Override Priority System
├── Question Type Coverage & Complexity
├── Multi-Column Table Handling
└── Implementation Examples
```

**Visual Documentation:**

| Diagram | Description | Key Insights |
|---------|-------------|--------------|
| ![Data Flow](images/enrichment_xml_json_db_flow_v2.png) | End-to-end pipeline | XML → Parser → JSON → Database → API → UI |
| ![Before/After](images/enrichment_before_after_v2.png) | Impact comparison | 200+ positions → 1-3 suggestions |
| ![Attribute Mapping](images/enrichment_attribute_mapping_v2.png) | XML↔DB mappings | How enrichment drives queries |
| ![CSV Override](images/enrichment_csv_override_v2.png) | Priority system | Expert overrides for edge cases |
| ![Question Types](images/enrichment_question_types_v2.png) | Coverage matrix | All ESRS question patterns |
| ![Multi-Column](images/enrichment_multi_column_v2.png) | Complex tables | Column-level hint handling |

---

## 🏗️ Solution Architecture

### High-Level Components

```
┌─────────────────────────────────────────────────────────────────┐
│                     DISCLOSURE UI (ExtJS)                       │
│  Question Renderer • Position Selector • Confidence Display     │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│              Position Suggestion API Service                    │
│  POST /api/disclosure/suggest-positions                        │
│  • Load enrichment metadata from JSON                          │
│  • Execute 5-stage filtering algorithm                         │
│  • Calculate confidence scores                                 │
│  • Return top 1-3 positions                                    │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Database Layer (PostgreSQL)                  │
│  Tables: position, position_type, unit_class, transaction      │
│  Enriched Templates: disclosure_template (JSON column)         │
└─────────────────────────────────────────────────────────────────┘
```

### Decision Tree Examples

The solution includes **real-world decision tree examples** showing filtering logic:

| Decision Tree | Use Case | Visual Reference |
|---------------|----------|------------------|
| **Overview** | System architecture and filter progression | ![Overview](images/decision_tree_overview_v2.png) |
| **Filter Progression** | 200 → 50 → 30 → 10 → 5 → 3 positions | ![Progression](images/filter_progression_v2.png) |
| **Decision Matrix** | Attribute-based routing logic | ![Matrix](images/decision_matrix_v2.png) |
| **Scope 2 Emissions** | Electricity location-based calculation | ![Scope 2](images/decision_tree_1_scope2_emissions_v2.png) |
| **Energy Calculation** | Percentage-based energy indicator | ![Energy](images/decision_tree_2_energy_calculation_v2.png) |
| **Scope 3 Upstream** | Category-based upstream emissions | ![Scope 3](images/decision_tree_3_scope3_upstream_v2.png) |

**📄 Complete Visual Reference:** [DECISION_TREE_VISUAL_INDEX.md](DECISION_TREE_VISUAL_INDEX.md)

---

## 🔧 Technical Implementation

### 1. XML Template Enrichment

Extend disclosure templates with `<mappingHints>` metadata:

```xml
<question id="cd23f3d4-1c92-4a34-af49-e831e6acd75e">
    <question xml:lang="en">Gross Scope 2 GHG emissions</question>
    <xbrlConcepts>
        <xbrlConcept>esrs:GrossScope2GreenhouseGasEmissions</xbrlConcept>
    </xbrlConcepts>
    <mappingHints>
        <unitType>mass</unitType>
        <positionType>emission</positionType>
        <scope>scope2_location</scope>
        <requiresCalculation>false</requiresCalculation>
        <periodType>duration</periodType>
    </mappingHints>
</question>
```

### 2. Five-Stage Filtering Algorithm

Progressive reduction of candidate positions:

```
Stage 1: Position Type Filter   (200 → 50 positions)
   ↓     Filter: position.position_type → position_type.name = 'emission'
   
Stage 2: Unit Type Filter        (50 → 30 positions)
   ↓     Filter: position.unit_class_id → unit_class.type = 'mass'
   
Stage 3: Scope Filter            (30 → 10 positions)
   ↓     Filter: position.path LIKE '%scope2%' OR tags LIKE '%scope2%'
   
Stage 4: Site Assignment Filter  (10 → 5 positions)
   ↓     Filter: transaction.site_id IN (disclosure_site_ids)
   
Stage 5: Data Completeness       (5 → 1-3 positions)
         Score: COUNT(DISTINCT occurrence_date) / expected_occurrences
```

### 3. Scoring System

**Point Allocation:**
- **Unit Match:** +30 points (critical for accuracy)
- **Position Type Match:** +25 points (emission, energy, etc.)
- **Scope Match:** +20 points (scope1, scope2, scope3)
- **Data Completeness:** +15 points (scaled by percentage)
- **Site Assignment:** +10 points (position used for sites)
- **Data Recency:** +5 points (recent data preferred)
- **User Tag Override:** +50 points (manual disclosure tag)

**Confidence Levels:**
- 🟢 **HIGH:** Score ≥ 90 (strong match, high confidence)
- 🟡 **MEDIUM:** Score 70-89 (good match, review recommended)
- 🔴 **LOW:** Score < 70 (weak match, manual selection advised)

### 4. API Contract

**Endpoint:** `POST /api/disclosure/suggest-positions`

**Request:**
```json
{
  "questionId": "cd23f3d4-1c92-4a34-af49-e831e6acd75e",
  "disclosureId": 12345,
  "siteIds": [3419, 3420],
  "period": {
    "start": "2025-01-01",
    "end": "2025-12-31"
  }
}
```

**Response:**
```json
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
      "factors": [
        "unit_match",
        "type_match",
        "scope_match",
        "high_completeness",
        "site_match",
        "recent_data"
      ]
    },
    {
      "positionId": 12346,
      "termStart": 202501,
      "name": "Electricity Market-Based Site A",
      "unit": "tCO2eq",
      "scope": "scope2_market",
      "completeness": 0.95,
      "confidence": "HIGH",
      "score": 100,
      "factors": [
        "unit_match",
        "type_match",
        "scope_match",
        "high_completeness"
      ]
    }
  ],
  "metadata": {
    "totalCandidates": 212,
    "filteredToCount": 2,
    "executionTimeMs": 1847
  }
}
```

---

## 📊 Success Metrics

### Performance KPIs

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Response Time** | <2 seconds | API response time (p95) |
| **Accuracy** | 95% | Suggested position accepted by user |
| **Time Savings** | 80% reduction | Minutes per question (30 → 6) |
| **User Satisfaction** | 4.5/5 stars | Post-implementation survey |
| **Coverage** | 100% | Questions with suggestions |

### Business Impact

| Area | Current State | Target State | Impact |
|------|--------------|--------------|--------|
| **Time per Disclosure** | 40 hours | 8 hours | 80% reduction |
| **Error Rate** | 15% | 3% | 80% improvement |
| **User Training Time** | 2 weeks | 2 days | 90% reduction |
| **Scalability** | Manual limit | Unlimited | Framework expansion enabled |

---

## 🚀 Implementation Timeline

**Total Duration:** 4 months (16 weeks)  
**Budget:** $95,000  
**Team:** 2-3 engineers + 1 domain expert

### Phase Breakdown

| Phase | Duration | Deliverables |
|-------|----------|--------------|
| **Phase 1: Template Enrichment** | Weeks 1-2 | Enriched ESRS E1 templates (50 questions) |
| **Phase 2: Filter Service** | Weeks 2-4 | 5-stage filtering algorithm |
| **Phase 3: Scoring Engine** | Weeks 3-4 | Point-based scoring system |
| **Phase 4: API Development** | Week 4 | REST endpoint + documentation |
| **Phase 5: UI Integration** | Week 5 | Position selector with confidence display |
| **Phase 6: Testing** | Weeks 6-8 | Unit, integration, UAT |
| **Phase 7: Deployment** | Weeks 9-12 | DEV → TEST → UAT → PROD |
| **Phase 8: Monitoring** | Weeks 13-16 | Performance tracking + optimization |

**📄 Detailed Timeline:** See [SDD Section 5.1](SDD_Position_Mapping_Smart_Suggestions.md#51-implementation-timeline)

---

## 🗂️ Database Schema

### Core Tables

```sql
-- Enriched template storage
disclosure_template (
    id, template_id, question_id, 
    enrichment_metadata JSONB,  -- Stores mappingHints
    version, created_at
)

-- Position filtering targets
position (
    id, name, position_type_id, unit_class_id, 
    path, tags, created_at
)

position_type (
    id, name, description
)

unit_class (
    id, name, type, symbol
)

-- Transaction data for completeness scoring
transaction (
    id, position_id, site_id, occurrence_date,
    value, created_at
)
```

**📄 Complete Schema:** See [SDD Section 4](SDD_Position_Mapping_Smart_Suggestions.md#4-database-design)

---

## 🧪 Testing Strategy

### Test Coverage

| Test Type | Coverage | Tools |
|-----------|----------|-------|
| **Unit Tests** | 80% code coverage | PHPUnit, Jest |
| **Integration Tests** | API endpoints + DB queries | PHPUnit + Test DB |
| **UAT Tests** | Real-world scenarios | Manual testing by users |
| **Performance Tests** | Response time benchmarks | Apache Bench, JMeter |

### Test Scenarios

**Filtering Accuracy:**
- ✅ Single-unit questions (e.g., Scope 2 emissions)
- ✅ Multi-unit questions (e.g., energy by source)
- ✅ Percentage-based indicators
- ✅ Period-specific questions (instant vs duration)
- ✅ Edge cases (no matches, multiple strong matches)

**CSV Override:**
- ✅ Expert override takes precedence
- ✅ Invalid overrides logged
- ✅ Fallback to enrichment metadata

**Performance:**
- ✅ <2s response time for 200+ position pool
- ✅ Concurrent request handling (10+ simultaneous users)
- ✅ Database query optimization

**📄 Detailed Test Plan:** See [SDD Section 6](SDD_Position_Mapping_Smart_Suggestions.md#6-testing-strategy)

---

## 🔐 Security & Compliance

### Data Protection

- ✅ **Access Control:** Position suggestions respect user site permissions
- ✅ **Data Privacy:** No PII in enrichment metadata
- ✅ **Audit Trail:** All suggestions logged with confidence scores
- ✅ **Validation:** Input sanitization on API endpoints

### Compliance Considerations

- ✅ **ESRS Compliance:** Enrichment aligns with XBRL concepts
- ✅ **GRI Compatibility:** Extensible to GRI frameworks
- ✅ **TCFD Support:** Planned for Phase 2 expansion

---

## 📖 Additional Resources

### Index Documents

| Document | Purpose |
|----------|---------|
| [DECISION_TREE_VISUAL_INDEX.md](DECISION_TREE_VISUAL_INDEX.md) | Complete visual reference for decision trees |
| [DECISION_TREE_V2_ENHANCEMENTS.md](DECISION_TREE_V2_ENHANCEMENTS.md) | V2 improvements and real-world examples |
| [IMAGE_ORGANIZATION_SUMMARY.md](IMAGE_ORGANIZATION_SUMMARY.md) | Visual asset inventory and regeneration guide |
| [EXECUTION_SUMMARY.md](EXECUTION_SUMMARY.md) | Project execution log and lessons learned |

### Supporting Documentation

| Document | Purpose |
|----------|---------|
| [BridgeTheGap.md](BridgeTheGap.md) | Original specifications and requirements |
| [ENRICHMENT_PROJECT_SUMMARY.md](ENRICHMENT_PROJECT_SUMMARY.md) | Project overview and context |
| [CONDITIONAL_CLEANUP_USAGE_ANALYSIS.md](CONDITIONAL_CLEANUP_USAGE_ANALYSIS.md) | Template cleanup strategy |
| [DATABASE_SCHEMA_VALIDATION_AND_V2_COMPLETE.md](DATABASE_SCHEMA_VALIDATION_AND_V2_COMPLETE.md) | Schema validation results |

---

## 🤝 Getting Started

### For Developers

1. **Read the SDD** → [SDD_Position_Mapping_Smart_Suggestions.md](SDD_Position_Mapping_Smart_Suggestions.md)
2. **Study Visual Documentation** → [XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md](XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md)
3. **Review Decision Trees** → [DECISION_TREE_VISUAL_INDEX.md](DECISION_TREE_VISUAL_INDEX.md)
4. **Check Database Schema** → SDD Section 4
5. **Test API Contract** → SDD Section 3.4

### For Product Owners

1. **Understand the Problem** → This Wiki Overview
2. **Review Success Metrics** → This Wiki, Success Metrics section
3. **Check Budget & Timeline** → [SDD Section 5](SDD_Position_Mapping_Smart_Suggestions.md#5-deployment-strategy)
4. **See Visual Impact** → [Before/After Diagram](images/enrichment_before_after_v2.png)

### For Domain Experts

1. **Template Enrichment Guide** → [XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md](XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md)
2. **CSV Override Mechanism** → [Enrichment Section 4](XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md#4-csv-override-priority-system)
3. **Question Type Coverage** → [Enrichment Section 5](XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md#5-question-type-coverage-and-complexity)
4. **Multi-Column Handling** → [Enrichment Section 6](XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md#6-multi-column-table-handling)

### For QA/Testers

1. **Testing Strategy** → [SDD Section 6](SDD_Position_Mapping_Smart_Suggestions.md#6-testing-strategy)
2. **Decision Tree Examples** → [DECISION_TREE_VISUAL_INDEX.md](DECISION_TREE_VISUAL_INDEX.md)
3. **Edge Cases** → [Enrichment Multi-Column Section](XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md#6-multi-column-table-handling)
4. **Performance Benchmarks** → [SDD Section 7](SDD_Position_Mapping_Smart_Suggestions.md#7-monitoring-and-success-metrics)

---

## 📞 Support & Feedback

### Questions?

- **Technical Questions:** See [SDD_Position_Mapping_Smart_Suggestions.md](SDD_Position_Mapping_Smart_Suggestions.md)
- **Enrichment Questions:** See [XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md](XML_TO_JSON_ENRICHMENT_ANALYSIS_V2.md)
- **Visual Examples:** See [DECISION_TREE_VISUAL_INDEX.md](DECISION_TREE_VISUAL_INDEX.md)

### Contributing

To improve this documentation:
1. All images are in `images/` subdirectory
2. Source files (images/mmd/*.mmd) available for regeneration
3. Follow existing diagram conventions
4. Update index documents when adding new content

---

## 📋 Document Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2026-02-13 | Initial wiki creation with references to SDD and enrichment docs | Development Team |

---

**Last Updated:** February 13, 2026  
**Status:** 🟢 Active - Implementation in Progress  
**Next Review:** March 1, 2026
