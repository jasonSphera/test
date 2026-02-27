# Software Design Document (SDD) v3.0
## AI-Assisted Smart Mapping for Sustainability Disclosure Data

**Version:** 3.0  
**Date:** February 10, 2026  
**Status:** Draft  
**Authors:** Development Team  
**Reviewers:** Robert Päßler, Hermann, Antonia

---

## Document Purpose

This SDD defines the technical approach for implementing smart mapping functionality that suggests positions and reports for disclosure questions, with support for both programmatic (rule-based) and AI-assisted implementations.

---

## 1. Overview

### 1.1 Background

Customers creating sustainability disclosures (ESRS, GRI, ISSB, CDP) must manually map disclosure questions to positions and reports in the CS system. This process is:
- Time-consuming (weeks to months)
- Error-prone (incorrect mappings)
- Requires deep technical knowledge
- Must be repeated for each disclosure

### 1.2 Goals

1. **Reduce customer effort by 70%+** through intelligent mapping suggestions
2. **Support both implementation approaches:**
   - **Option 1:** Programmatic/rule-based service (CS-native)
   - **Option 2:** AI-assisted service (external AI platform)
3. **Enable confidence-based suggestions** with transparent reasoning
4. **Collect feedback** for continuous improvement
5. **Maintain flexibility** to switch or blend approaches

### 1.3 Scope

**In Scope:**
- Template context enrichment (prerequisite for both options)
- Position semantic tagging system
- Mapping suggestion API endpoints
- Frontend suggestion UI
- Feedback collection mechanism
- Both implementation options (programmatic + AI)

**Out of Scope:**
- AI model training (future phase)
- Automatic answer generation (text responses only)
- Real-time data collection automation
- Multi-disclosure bulk operations (initial release)

---

## 2. Architecture Overview

### 2.1 System Components

```
┌─────────────────────────────────────────────────────────────┐
│                    DISCLOSURE MAPPING SYSTEM                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │         PREREQUISITE: CONTEXT PREPARATION          │    │
│  │  (Required for both Option 1 and Option 2)         │    │
│  ├────────────────────────────────────────────────────┤    │
│  │ 1. Template Enrichment Service                     │    │
│  │    ├─ XBRL Parser                                  │    │
│  │    ├─ NLP Keyword Extractor                        │    │
│  │    ├─ Out-of-Box Pattern Analyzer                  │    │
│  │    └─ Metadata Generator                           │    │
│  │                                                     │    │
│  │ 2. Position Tagging Service                        │    │
│  │    ├─ Auto-Tagger (NLP, Path, Impact, Unit)       │    │
│  │    ├─ Manual Curation Interface                    │    │
│  │    └─ Tag Storage (position_semantic_tag)          │    │
│  │                                                     │    │
│  │ 3. Context Data Layer                              │    │
│  │    ├─ Enhanced Template JSON                       │    │
│  │    ├─ Position Semantic Tags                       │    │
│  │    ├─ XBRL Mappings                                │    │
│  │    └─ Out-of-Box Reference Data                    │    │
│  └────────────────────────────────────────────────────┘    │
│                           │                                  │
│                           ↓                                  │
│  ┌────────────────────────┴───────────────────────────┐    │
│  │                                                      │    │
│  │        IMPLEMENTATION OPTIONS (Choose One)          │    │
│  │                                                      │    │
│  ├──────────────────────┬───────────────────────────┤    │
│  │                      │                            │    │
│  │   OPTION 1:          │      OPTION 2:             │    │
│  │   Programmatic       │      AI-Assisted           │    │
│  │   (CS Service)       │      (External AI)         │    │
│  │                      │                            │    │
│  │   [Details below]    │      [Details below]       │    │
│  │                      │                            │    │
│  └──────────────────────┴────────────────────────────┘    │
│                           │                                  │
│                           ↓                                  │
│  ┌────────────────────────────────────────────────────┐    │
│  │              COMMON OUTPUT LAYER                    │    │
│  ├────────────────────────────────────────────────────┤    │
│  │ • Suggestion API                                    │    │
│  │ • Confidence Scoring                                │    │
│  │ • Frontend UI                                       │    │
│  │ • Feedback Collection                               │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. PREREQUISITE: Context Preparation

**Status:** Required for BOTH implementation options  
**Effort:** 4-6 weeks  
**Dependencies:** None

### 3.1 Objective

Prepare machine-readable context that enables intelligent mapping suggestions, regardless of implementation approach.

### 3.2 Components

#### 3.2.1 Template Enrichment

**Purpose:** Add semantic metadata to disclosure template questions

**Input Sources:**
1. **XBRL Taxonomy** (Auto-extractable)
   - Item types (energy, mass, GHG emissions, etc.)
   - Unit definitions and mappings
   - Concept metadata
   - Data type specifications (text, numeric, date)

2. **Out-of-Box Mappings** (Semi-manual)
   - Pre-mapped position assignments
   - Position name patterns
   - Hierarchical path patterns
   - Common customer setups

3. **Question Text Analysis** (NLP)
   - Keyword extraction (scope, emissions, energy, etc.)
   - Unit detection (tCO2e, kWh, m³)
   - Question classification (qualitative/quantitative)
   - Aggregation level detection (site/organization)

**Output Schema:**

```json
{
  "nodeId": "question-uuid",
  "nodeType": "question",
  "title": {"en": "Report gross direct (Scope 1) GHG emissions"},
  
  "enrichedMetadata": {
    "semanticKeywords": [
      "scope_1",
      "direct_emissions",
      "ghg_emissions",
      "co2e"
    ],
    
    "dataRequirements": {
      "impactCategories": ["climate_change"],
      "scopes": [1],
      "unitClasses": ["mass"],
      "preferredUnits": ["tCO2e", "kgCO2e"],
      "positionTypes": ["indicator"],
      "aggregationLevel": "organization"
    },
    
    "mappingHints": {
      "positionPathPatterns": ["/*/Emissions/Scope 1/*"],
      "positionNamePatterns": [".*scope.*1.*"],
      "excludePositionTypes": ["text", "question"]
    },
    
    "expectedFormat": {
      "answerType": "quantitative",
      "requiresPositions": true,
      "minPositions": 1
    },
    
    "xbrlConcepts": [
      "ifrs-full:DirectGreenhouseGasEmissionsScope1"
    ]
  }
}
```

**Storage:** `disclosure_template.content` JSON field

**Requirements:**
- ✅ REQ-PREP-1: Extract XBRL item types and map to CS unit classes
- ✅ REQ-PREP-2: Parse out-of-box mappings into reusable patterns
- ✅ REQ-PREP-3: Implement NLP keyword extraction service
- ✅ REQ-PREP-4: Validate enriched metadata against schema
- ✅ REQ-PREP-5: Admin UI for manual metadata curation
- ✅ REQ-PREP-6: Bulk enrichment tool for existing templates

**Deliverables:**
- Enrichment service implementation
- Admin UI for metadata management
- Enriched ESRS, GRI, ISSB templates
- Validation and testing suite

#### 3.2.2 Position Semantic Tagging

**Purpose:** Tag positions with semantic keywords for matching

**Auto-Tagging Sources:**
1. Position name analysis (NLP)
2. Hierarchical path structure
3. Impact mapping assignments
4. Unit class associations
5. Documentation field parsing

**Database Schema:**

```sql
CREATE TABLE position_semantic_tag (
    position_semantic_tag_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    position_id INT UNSIGNED NOT NULL,
    tag VARCHAR(100) NOT NULL,
    source ENUM('auto', 'manual', 'user') DEFAULT 'auto',
    confidence TINYINT UNSIGNED DEFAULT 100,
    creator INT UNSIGNED NOT NULL,
    created DATETIME NOT NULL,
    UNIQUE KEY (position_id, tag),
    INDEX idx_tag (tag),
    INDEX idx_position_confidence (position_id, confidence),
    FOREIGN KEY (position_id) REFERENCES position_index(position_id)
) ENGINE=InnoDB;
```

**Example:**
```
Position: "Natural Gas Combustion - Site A"
Path: "/Global/Emissions/Scope 1/Natural Gas"
Impact: Climate Change
Unit: tCO2e

Generated Tags:
- scope_1 (confidence: 95%)
- emissions (confidence: 100%)
- natural_gas (confidence: 100%)
- combustion (confidence: 90%)
- climate_change (confidence: 100%)
```

**Requirements:**
- ✅ REQ-PREP-7: Implement auto-tagging service
- ✅ REQ-PREP-8: Create position_semantic_tag table
- ✅ REQ-PREP-9: Build manual curation UI
- ✅ REQ-PREP-10: Tag all out-of-box positions
- ✅ REQ-PREP-11: Bulk tagging for production positions
- ✅ REQ-PREP-12: Tag versioning and update workflow

**Deliverables:**
- Auto-tagging service
- Database migration script
- Curation UI
- Tagged position dataset

#### 3.2.3 Context Data Preparation

**Purpose:** Prepare supplementary data for matching algorithms

**Data Components:**

1. **XBRL-to-CS Mappings**
   - Unit mappings (already exists)
   - Quantity class mappings (new)
   - Item type to position type (new)

2. **Historical Usage Data**
   - Customer disclosure history
   - Position usage patterns
   - Common mappings by framework

3. **Data Completeness Index**
   - Positions with recent data
   - Data coverage by site/term
   - Quality indicators

**Requirements:**
- ✅ REQ-PREP-13: Extract and store XBRL-CS mappings
- ✅ REQ-PREP-14: Build historical usage tracking
- ✅ REQ-PREP-15: Implement data completeness scoring

---

## 4. OPTION 1: Programmatic Implementation (CS Service)

**Approach:** Rule-based matching service within CS  
**Effort:** 6-8 weeks (after prerequisite)  
**Dependencies:** Context preparation complete

### 4.1 Architecture

```
┌───────────────────────────────────────────────────┐
│         PROGRAMMATIC MAPPING SERVICE              │
├───────────────────────────────────────────────────┤
│                                                    │
│  Input:                                            │
│  ├─ Enhanced question metadata                    │
│  ├─ Disclosure context (site, tag, term)          │
│  ├─ Position semantic tags                        │
│  └─ Customer data availability                    │
│                                                    │
│  Processing:                                       │
│  ├─ Context Filtering                             │
│  │   ├─ Site/tag scope                            │
│  │   ├─ Temporal availability                     │
│  │   └─ Position type exclusions                  │
│  │                                                 │
│  ├─ Multi-Criteria Matching                       │
│  │   ├─ Semantic match (30% weight)               │
│  │   ├─ Impact match (25% weight)                 │
│  │   ├─ Unit class match (20% weight)             │
│  │   ├─ Path pattern match (15% weight)           │
│  │   └─ Name pattern match (10% weight)           │
│  │                                                 │
│  ├─ Confidence Scoring                            │
│  │   ├─ Weighted score calculation                │
│  │   ├─ Data completeness boost                   │
│  │   ├─ Historical usage boost                    │
│  │   └─ User feedback adjustments                 │
│  │                                                 │
│  └─ Ranking & Filtering                           │
│      ├─ Sort by confidence (descending)           │
│      ├─ Apply minimum threshold (40%)             │
│      └─ Limit to top N (default: 10)              │
│                                                    │
│  Output:                                           │
│  └─ Ranked suggestions with reasoning             │
│                                                    │
└───────────────────────────────────────────────────┘
```

### 4.2 Requirements

#### 4.2.1 Service Layer

**REQ-OPT1-1: MappingService Core**
- Implement multi-criteria matching engine
- Support configurable weights
- Thread-safe for concurrent requests
- Performance: <2s per question

**REQ-OPT1-2: Matcher Components**
```php
// Core matchers
- SemanticMatcher: Keyword overlap scoring
- ImpactMatcher: Impact category matching
- UnitClassMatcher: Unit type validation
- PathMatcher: Hierarchical pattern matching
- NameMatcher: Regex pattern matching
```

**REQ-OPT1-3: Context Filter**
- Filter positions by site/tag assignment
- Check temporal data availability
- Apply position type exclusions
- Hierarchical scope handling

**REQ-OPT1-4: Confidence Scorer**
- Calculate weighted scores
- Apply data completeness factor (0-100%)
- Apply historical usage factor (0-100%)
- Apply user feedback adjustments (±20%)

#### 4.2.2 Algorithm Specifications

**Semantic Matching:**
```
score = (matched_keywords / total_question_keywords) × 100

Example:
Question keywords: ["scope_1", "emissions", "direct", "combustion"]
Position tags: ["scope_1", "emissions", "natural_gas"]
Matched: 2/4 = 50%
```

**Impact Matching:**
```
IF question requires impact AND position has impact:
  IF impacts overlap: score = 100
  ELSE: score = 0
ELSE IF neither requires impact: score = 50 (neutral)
ELSE: score = 0
```

**Unit Class Matching:**
```
IF position.unit_class IN question.required_unit_classes:
  score = 100
ELSE:
  score = 0
```

**Path Pattern Matching:**
```
FOR EACH pattern IN question.path_patterns:
  IF fnmatch(position.path, pattern):
    score = 100
    BREAK
ELSE:
  score = 0
```

**Final Score:**
```
final = (semantic × 0.30) + 
        (impact × 0.25) + 
        (unit_class × 0.20) + 
        (path × 0.15) + 
        (name × 0.10) +
        data_completeness_boost +
        historical_usage_boost +
        user_feedback_adjustment
```

#### 4.2.3 Data Requirements

**REQ-OPT1-5: Required CS Data**
- Position metadata (name, path, type, unit class)
- Position semantic tags
- Impact mapping assignments
- Questionnaire assignments (site/tag)
- Data collection status (by position, site, term)
- Historical disclosure mappings (if available)

**REQ-OPT1-6: Caching Strategy**
- Cache suggestions per question (TTL: 1 hour)
- Cache position metadata (TTL: 24 hours)
- Invalidate on position/template updates

#### 4.2.4 Testing Requirements

**REQ-OPT1-7: Validation**
- Test with out-of-box database (100% known mappings)
- Calculate precision/recall metrics
- Target: 80%+ accuracy on out-of-box data
- Edge case testing (missing data, unusual setups)

**REQ-OPT1-8: Performance**
- Benchmark with 1000+ positions
- Optimize query patterns
- Target: <2s response time per question

### 4.3 Advantages of Option 1

✅ **Full control** - No external dependencies  
✅ **Security** - All data stays within CS  
✅ **Performance** - Direct database access  
✅ **Explainable** - Clear rule-based reasoning  
✅ **Deterministic** - Reproducible results  
✅ **No cost** - No AI platform fees  
✅ **Immediate deployment** - No AI model training

### 4.4 Limitations of Option 1

❌ **Static rules** - Cannot learn from patterns  
❌ **Manual tuning** - Weights must be adjusted manually  
❌ **Complex edge cases** - May miss nuanced mappings  
❌ **Maintenance** - Rules need periodic updates

### 4.5 Deliverables

- MappingService implementation
- Matcher components (5 classes)
- ConfidenceScorer implementation
- API endpoints (3 endpoints)
- Unit tests (>80% coverage)
- Integration tests with out-of-box
- Performance benchmarks
- Documentation

---

## 5. OPTION 2: AI-Assisted Implementation

**Approach:** External AI platform for matching  
**Effort:** 4-6 weeks (after prerequisite + AI platform setup)  
**Dependencies:** Context preparation + AI platform integration

### 5.1 Architecture

```
┌────────────────────────────────────────────────────┐
│         AI-ASSISTED MAPPING SERVICE                │
├────────────────────────────────────────────────────┤
│                                                     │
│  CS System (Data Preparation)                      │
│  ├─ Collect question context                       │
│  ├─ Collect candidate positions                    │
│  ├─ Package as AI-friendly format                  │
│  └─ Send to AI platform                            │
│                   │                                 │
│                   ↓                                 │
│  AI Platform (External)                            │
│  ├─ Parse question semantics                       │
│  ├─ Understand position descriptions               │
│  ├─ Perform semantic similarity matching           │
│  ├─ Generate confidence scores                     │
│  └─ Return ranked suggestions                      │
│                   │                                 │
│                   ↓                                 │
│  CS System (Post-Processing)                       │
│  ├─ Validate AI responses                          │
│  ├─ Apply CS-specific filters                      │
│  ├─ Add data completeness info                     │
│  └─ Format for frontend                            │
│                                                     │
└────────────────────────────────────────────────────┘
```

### 5.2 Requirements

#### 5.2.1 Data Preparation for AI

**REQ-OPT2-1: Question Context Payload**

Must send to AI:
```json
{
  "question": {
    "id": "question-uuid",
    "text": "Report gross direct (Scope 1) GHG emissions",
    "description": "Include emissions from sources owned or controlled",
    "guidance": "Follow GHG Protocol guidelines",
    "framework": "ESRS",
    "section": "E1-6",
    "isMandatory": true,
    
    "semanticKeywords": ["scope_1", "direct_emissions", "ghg"],
    "xbrlConcepts": ["ifrs-full:DirectGreenhouseGasEmissionsScope1"],
    
    "expectedAnswer": {
      "type": "quantitative",
      "requiresPositions": true,
      "unitClass": "mass",
      "preferredUnits": ["tCO2e"]
    }
  },
  
  "disclosure": {
    "id": 12345,
    "site": {"id": 678, "name": "Global Operations"},
    "term": {"start": "202401", "end": "202412"}
  }
}
```

**REQ-OPT2-2: Position Candidate Payload**

Must send to AI:
```json
{
  "positions": [
    {
      "id": 5678,
      "name": "Natural Gas Combustion - Scope 1",
      "description": "Direct emissions from natural gas usage",
      "path": "/Global/Emissions/Scope 1/Natural Gas",
      "type": "indicator",
      "unitClass": "mass",
      "unit": "tCO2e",
      "semanticTags": ["scope_1", "emissions", "natural_gas"],
      "impactCategories": ["climate_change"],
      "hasData": true,
      "dataCompleteness": 92,
      "lastUpdated": "2024-01-15"
    }
    // ... more positions
  ]
}
```

**REQ-OPT2-3: Data Volume Limits**
- Maximum 100 candidate positions per request
- Maximum 10KB payload size
- Pre-filter positions in CS before sending to AI

#### 5.2.2 AI Platform Requirements

**REQ-OPT2-4: AI Capabilities Needed**
- ✅ Semantic similarity matching (embeddings)
- ✅ Multi-field comparison (name, description, tags)
- ✅ Confidence scoring (0-100%)
- ✅ Reasoning generation (explain matches)
- ✅ Batch processing (multiple questions)

**REQ-OPT2-5: AI Platform Integration**
- REST API endpoint for mapping requests
- Authentication/authorization mechanism
- Rate limiting compliance
- Error handling and retries
- Timeout handling (max 10s)

**REQ-OPT2-6: AI Response Format**
```json
{
  "suggestions": [
    {
      "positionId": 5678,
      "confidence": 87.5,
      "reasoning": {
        "semanticSimilarity": 0.92,
        "keywordMatches": ["scope_1", "emissions"],
        "unitClassMatch": true,
        "pathRelevance": 0.85
      },
      "explanation": "Strong match based on scope 1 keywords and emission unit type"
    }
  ],
  "processingTime": 1.2,
  "model": "gpt-4o-2024-11-20"
}
```

#### 5.2.3 CS System Requirements for Option 2

**REQ-OPT2-7: Position Pre-Filtering**
```php
// Filter positions before sending to AI
- Filter by site/tag assignment
- Filter by data availability (last 12 months)
- Filter by position type (exclude text/question types)
- Apply customer-specific exclusions
- Limit to top 100 by relevance (basic scoring)
```

**REQ-OPT2-8: AI Response Post-Processing**
```php
// After receiving AI suggestions
- Validate position IDs exist
- Add CS-specific metadata (data completeness, last update)
- Apply minimum confidence threshold (40%)
- Merge with historical usage data
- Format for frontend display
```

**REQ-OPT2-9: Fallback Mechanism**
```php
// If AI platform unavailable or fails
- Log error details
- Return cached suggestions (if available)
- OR fall back to basic rule-based matching
- Show user-friendly error message
```

#### 5.2.4 Data Requirements for AI Option

**REQ-OPT2-10: What CS Data to Send**

**Must send:**
- Question text (title, description, guidance)
- Semantic keywords (from enrichment)
- XBRL concepts (if available)
- Expected answer type (quantitative/qualitative)
- Unit class requirements
- Disclosure context (site, term)

**Must send for positions:**
- Position ID (for reference)
- Name and description
- Hierarchical path
- Semantic tags
- Unit class and unit
- Impact categories
- Data availability flag
- Data completeness percentage

**Must NOT send (privacy/security):**
- Actual data values
- Customer-specific configurations
- User information
- Site-specific sensitive data
- Historical financial data

**REQ-OPT2-11: Data Transformation**
```php
// Prepare positions for AI (anonymize/simplify)
foreach ($positions as $position) {
    $aiPayload[] = [
        'id' => $position->getPositionId(),
        'name' => $position->getName(),
        'path' => $this->anonymizePath($position->getPath()),
        'type' => $position->getType(),
        'tags' => $position->getTags(),
        'unitClass' => $position->getUnitClass()->getName(),
        'hasData' => $this->hasRecentData($position),
        // Exclude: actual values, site names, customer identifiers
    ];
}
```

#### 5.2.5 Testing Requirements

**REQ-OPT2-12: AI Integration Testing**
- Mock AI platform for unit tests
- Integration tests with AI sandbox environment
- Validate AI response parsing
- Test error handling (timeouts, invalid responses)
- Performance testing (response times)

**REQ-OPT2-13: Comparison Testing**
- Compare AI suggestions vs. programmatic (Option 1)
- Measure accuracy differences
- Analyze failure modes
- Document edge cases

### 5.3 Advantages of Option 2

✅ **Semantic understanding** - Better at nuanced language matching  
✅ **Adaptive** - Can learn patterns (with future training)  
✅ **Less manual tuning** - AI finds patterns automatically  
✅ **Better with ambiguity** - Handles edge cases more gracefully  
✅ **Scalable** - Leverages external AI compute resources

### 5.4 Limitations of Option 2

❌ **External dependency** - Relies on AI platform availability  
❌ **Latency** - Network calls add 1-3s per request  
❌ **Cost** - AI API calls have per-request fees  
❌ **Less explainable** - Black box reasoning  
❌ **Security concerns** - Data leaves CS system  
❌ **Rate limits** - May hit AI platform quotas  
❌ **No learning initially** - Still requires training data (future)

### 5.5 Deliverables

- AI integration service
- Data preparation pipeline
- Response post-processing
- Fallback mechanism
- Integration tests with AI platform
- Cost analysis and monitoring
- Documentation

---

## 6. Common Components (Both Options)

### 6.1 API Endpoints

**Endpoint 1: Suggest Mappings**
```
POST /api/DisclosureManagement.Controller/suggestPositionMappings

Request:
{
  "disclosureId": 12345,
  "questionUuid": "question-uuid",
  "implementation": "programmatic" | "ai",  // Option selector
  "limit": 10,
  "minConfidence": 40
}

Response:
{
  "success": true,
  "implementation": "programmatic",
  "suggestions": [
    {
      "positionId": 5678,
      "name": "Natural Gas - Scope 1",
      "confidence": 87.5,
      "reasoning": {
        "semantic": "Matched 3/4 keywords",
        "impact": "Climate Change ✓",
        "unit": "Mass (tCO2e) ✓",
        "dataStatus": "11/12 months with data"
      }
    }
  ],
  "totalCandidates": 45,
  "processingTime": 1.2
}
```

**Endpoint 2: Apply Suggestions**
```
POST /api/DisclosureManagement.Controller/applyMappingSuggestions

Request:
{
  "disclosureQuestionId": 789,
  "suggestionIds": [1, 2, 5]
}

Response:
{
  "success": true,
  "applied": 3,
  "disclosureQuestionPositionIds": [101, 102, 103]
}
```

**Endpoint 3: Provide Feedback**
```
POST /api/DisclosureManagement.Controller/provideMappingFeedback

Request:
{
  "disclosureQuestionId": 789,
  "positionId": 5678,
  "action": "accepted" | "rejected" | "modified",
  "suggestedConfidence": 87.5,
  "actualConfidence": 95,
  "feedbackText": "Perfect match"
}

Response:
{
  "success": true,
  "message": "Feedback recorded"
}
```

### 6.2 Frontend UI

**REQ-UI-1: Suggestion Display**
```
┌─────────────────────────────────────────────────┐
│  Question: Report Scope 1 emissions             │
├─────────────────────────────────────────────────┤
│  [5 Suggested Positions] [Confidence ▼]         │
│                                                  │
│  ┌─────────────────────────────────────────┐   │
│  │ ✓ Natural Gas - Scope 1        [87%] ✓  │   │
│  │   Path: /Global/Emissions/Scope 1       │   │
│  │   Data: 11/12 months   [ⓘ Details]      │   │
│  │   [Accept] [Reject] [Modify]            │   │
│  └─────────────────────────────────────────┘   │
│                                                  │
│  ┌─────────────────────────────────────────┐   │
│  │   Diesel Fleet - Scope 1       [82%]    │   │
│  │   Path: /Global/Emissions/Scope 1       │   │
│  │   Data: 12/12 months   [ⓘ Details]      │   │
│  │   [Accept] [Reject] [Modify]            │   │
│  └─────────────────────────────────────────┘   │
│                                                  │
│  [Show More (3)] [Suggest Again]                │
└─────────────────────────────────────────────────┘
```

**REQ-UI-2: Confidence Visualization**
- 90-100%: Dark green bar + "Excellent" label
- 70-89%: Green bar + "Good" label  
- 50-69%: Orange bar + "Fair" label
- 30-49%: Red bar + "Weak" label (show on request)
- 0-29%: Hidden by default

**REQ-UI-3: Reasoning Tooltip**
```
Hover over [ⓘ Details]:
┌──────────────────────────────────┐
│ Why this suggestion?             │
├──────────────────────────────────┤
│ ✓ Keywords matched: 3/4          │
│ ✓ Impact: Climate Change         │
│ ✓ Unit: Mass (tCO2e)             │
│ ✓ Path matches pattern           │
│ ✓ Data available: 92%            │
│ ~ Historical usage: Low          │
└──────────────────────────────────┘
```

### 6.3 Feedback Collection

**REQ-FEED-1: Feedback Storage**
```sql
CREATE TABLE disclosure_mapping_feedback (
    feedback_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    disclosure_question_id INT UNSIGNED NOT NULL,
    position_id INT UNSIGNED NOT NULL,
    implementation VARCHAR(20) NOT NULL, -- 'programmatic' or 'ai'
    suggested_confidence TINYINT UNSIGNED NOT NULL,
    user_action ENUM('accepted', 'rejected', 'modified') NOT NULL,
    actual_confidence TINYINT UNSIGNED NULL,
    feedback_text TEXT NULL,
    user_id INT UNSIGNED NOT NULL,
    created DATETIME NOT NULL,
    INDEX idx_implementation (implementation),
    INDEX idx_action (user_action),
    FOREIGN KEY (disclosure_question_id) 
        REFERENCES disclosure_question(disclosure_question_id),
    FOREIGN KEY (position_id) 
        REFERENCES position_index(position_id),
    FOREIGN KEY (user_id) 
        REFERENCES user(user_id)
) ENGINE=InnoDB;
```

**REQ-FEED-2: Feedback Analytics**
- Track acceptance rate by confidence level
- Identify common rejection reasons
- Compare Option 1 vs Option 2 performance
- Use for future algorithm tuning

### 6.4 Performance Requirements

**REQ-PERF-1: Response Times**
- Suggestion generation: <2s (Option 1), <5s (Option 2)
- Apply suggestions: <500ms
- Feedback submission: <200ms

**REQ-PERF-2: Scalability**
- Support 100+ concurrent users
- Handle 1000+ positions per customer
- Cache frequently requested suggestions

**REQ-PERF-3: Availability**
- 99.5% uptime (Option 1)
- 95% uptime (Option 2, depends on AI platform)
- Graceful degradation on failures

---

## 7. Implementation Phases

### Phase 0: Context Preparation (REQUIRED)
**Duration:** 4-6 weeks  
**Team:** 2 Backend + 1 PM

**Tasks:**
1. Template enrichment service (2 weeks)
2. Position tagging service (2 weeks)
3. Admin UI for curation (1 week)
4. Bulk enrichment of templates (1 week)

**Deliverables:**
- Enriched ESRS, GRI, ISSB templates
- Tagged positions (out-of-box + sample production)
- Curation tools
- Documentation

### Phase 1A: Option 1 Implementation (IF SELECTED)
**Duration:** 6-8 weeks  
**Team:** 2 Backend + 1 Frontend + 1 QA

**Tasks:**
1. MappingService core (2 weeks)
2. Matcher components (2 weeks)
3. API endpoints (1 week)
4. Frontend UI (2 weeks)
5. Testing & calibration (1-2 weeks)

### Phase 1B: Option 2 Implementation (IF SELECTED)
**Duration:** 4-6 weeks  
**Team:** 1 Backend + 1 Integration + 1 Frontend + 1 QA

**Tasks:**
1. AI integration service (2 weeks)
2. Data preparation pipeline (1 week)
3. API endpoints (1 week)
4. Frontend UI (2 weeks)
5. Testing with AI platform (1 week)

### Phase 2: Pilot & Feedback (BOTH OPTIONS)
**Duration:** 4 weeks  
**Team:** Full team + pilot customers

**Tasks:**
1. Deploy to pilot customers (5-10)
2. Collect usage data and feedback
3. Analyze acceptance rates
4. Tune weights/parameters (Option 1) or prompts (Option 2)
5. Bug fixes and refinements

### Phase 3: Production Rollout
**Duration:** 4 weeks  
**Team:** Full team + DevOps

**Tasks:**
1. Performance optimization
2. Enrichment of remaining templates
3. Bulk position tagging for all customers
4. User documentation and training
5. Gradual rollout (10% → 50% → 100%)

---

## 8. Decision Matrix: Option 1 vs Option 2

| Criterion | Option 1 (Programmatic) | Option 2 (AI-Assisted) | Weight |
|-----------|-------------------------|------------------------|--------|
| **Development Time** | 6-8 weeks | 4-6 weeks | High |
| **Accuracy (Expected)** | 75-85% | 80-90% | Critical |
| **Explainability** | ★★★★★ Full transparency | ★★☆☆☆ Limited | High |
| **Maintenance Effort** | Medium (tune weights) | Low (AI adapts) | Medium |
| **Cost** | None (CS resources) | $$ per API call | High |
| **Security** | ★★★★★ All in CS | ★★★☆☆ Data leaves system | Critical |
| **Performance** | ★★★★★ <2s | ★★★☆☆ <5s | High |
| **Scalability** | ★★★★☆ CS servers | ★★★★★ AI platform | Medium |
| **Edge Cases** | ★★★☆☆ Rule-based limits | ★★★★☆ Better handling | Medium |
| **Learning Capability** | ❌ Static (initial) | ✅ Future potential | Low |
| **Dependencies** | None | AI platform availability | High |

### Recommendation Criteria

**Choose Option 1 (Programmatic) if:**
- Security is paramount (data cannot leave CS)
- Budget is limited (no AI costs)
- Need full explainability (regulatory requirements)
- Have capacity for manual tuning
- AI platform not yet available

**Choose Option 2 (AI-Assisted) if:**
- Higher accuracy is critical
- AI platform integration ready
- Budget available for AI costs
- Need better edge case handling
- Plan for future learning capability

**Hybrid Approach (Recommended):**
1. Implement Option 1 first (faster, no dependencies)
2. Add Option 2 as enhancement (parallel capability)
3. Let customers choose preferred implementation
4. Compare performance and gather data
5. Converge to best-performing option over time

---

## 9. Success Metrics

### Technical Metrics
| Metric | Target (Option 1) | Target (Option 2) |
|--------|-------------------|-------------------|
| Matching Accuracy | 80%+ | 85%+ |
| Response Time | <2s | <5s |
| Coverage (questions with ≥1 suggestion) | 90%+ | 95%+ |
| False Positive Rate | <15% | <10% |
| System Availability | 99.5% | 95% |

### User Metrics
| Metric | Target |
|--------|--------|
| Time Savings | 70% reduction in mapping time |
| Error Reduction | 80% fewer incorrect mappings |
| User Satisfaction | 8/10 rating |
| Adoption Rate | 90% of users try suggestions |
| Acceptance Rate | 75%+ of suggestions accepted |

### Business Metrics
| Metric | Target |
|--------|--------|
| Disclosure Completion Time | 40% faster |
| Support Tickets | 60% reduction |
| Training Time | 50% reduction for new users |
| Customer Satisfaction | +2 NPS points |

---

## 10. Risks and Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| **Manual enrichment takes too long** | High | Medium | Maximize auto-extraction from XBRL; prioritize high-usage frameworks |
| **Position tagging incomplete** | High | Medium | Phased rollout; focus on out-of-box first |
| **AI platform unavailable** | High | Low | Fallback to Option 1; cache suggestions |
| **Accuracy below expectations** | High | Medium | Extensive testing with out-of-box; weight calibration |
| **Performance too slow** | Medium | Low | Caching; pre-filtering; async processing |
| **User rejection of suggestions** | High | Medium | Transparency features; confidence thresholds; allow overrides |
| **Cost overruns (Option 2)** | Medium | Medium | Monitor API usage; set quotas; optimize calls |
| **Customer data variety** | High | High | Extensive edge case testing; graceful failures |

---

## 11. Appendices

### Appendix A: Database Schema Changes

**New Tables:**
1. `position_semantic_tag` - Position semantic tagging
2. `disclosure_mapping_feedback` - User feedback collection
3. `disclosure_mapping_suggestion` (optional) - Cached suggestions

**Modified Tables:**
- `disclosure_template` - Enhanced content JSON (no schema change)

### Appendix B: API Specifications

See Section 6.1 for complete API endpoint specifications.

### Appendix C: UI Mockups

See Section 6.2 for UI component specifications.

### Appendix D: Test Plan

**Unit Tests:**
- Matcher components (>90% coverage)
- Context filters
- Confidence scoring
- Data preparation

**Integration Tests:**
- End-to-end suggestion flow
- API endpoint validation
- Database operations
- Cache behavior

**System Tests:**
- Performance benchmarks
- Load testing
- Failure scenarios
- Edge case validation

**User Acceptance Tests:**
- Pilot customer feedback
- Usability testing
- Accuracy validation
- Documentation review

### Appendix E: Related Documents

- Meeting Transcript Summary (Feb 9, 2026)
- Smart Mapping Ideas Meeting Summary (Feb 9, 2026)
- TEMPLATE_MAPPING_AUTOMATION_ANALYSIS.md
- DISCLOSURE_ARCHITECTURE_SCCS_INTEGRATION.md
- DISCLOSURE_API_INTEGRATION_REFERENCE.md

---

## 12. Open Questions

1. **Final decision on Option 1 vs Option 2?**
   - Need stakeholder input on security/cost/accuracy tradeoffs

2. **AI platform selection (if Option 2)?**
   - Which AI platform (OpenAI, Azure, internal)?
   - Pricing model and budget approval

3. **Rollout strategy?**
   - Which framework first (ESRS, GRI, ISSB)?
   - Which customers for pilot?

4. **Manual enrichment workflow?**
   - Who performs curation?
   - How to maintain over time?

5. **Weight calibration approach (Option 1)?**
   - Initial weights vs. data-driven optimization
   - Frequency of recalibration

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Dev Manager | Robert Päßler | | |
| Lead PM | Hermann | | |
| Sr. PM | Antonia | | |
| Principal Engineer | Jason Oakes | | |

---

**Document Version:** 3.0  
**Last Updated:** February 10, 2026  
**Next Review:** After Jason's investigation (Feb 14, 2026)  
**Status:** Draft - Awaiting Stakeholder Review
