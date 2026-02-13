# Decision Tree V2 Enhanced Diagrams for SDD

**Version:** 2.0  
**Date:** February 12, 2026  
**Purpose:** Enhanced decision tree diagrams with real data examples, SQL queries, and field mappings

---

## What's New in V2

### V1 vs V2 Comparison

| Feature | V1 (Original) | V2 (Enhanced) |
|---------|---------------|---------------|
| **Data Examples** | Generic placeholders | Real position names, IDs, units |
| **SQL Queries** | Conceptual filters | Actual SQL with WHERE clauses |
| **XML→JSON Mappings** | High-level only | Explicit field mappings shown |
| **XBRL Concepts** | Generic mentions | Actual ESRS concepts (e.g., `esrs:GrossLocationBasedScope2...`) |
| **Scoring Details** | Basic algorithm | Complete scoring breakdown with points |
| **Output Examples** | Simple JSON | Full response JSON with all fields |
| **Position Names** | Abstract | Concrete examples (e.g., "Electricity Location-Based Site A") |
| **Intermediate Results** | Not shown | Every filter step shows count and examples |
| **Emission Factors** | Mentioned | Specific factors (GWP AR5, AR6) with validation |
| **Category Details** | Not shown | GHG Protocol categories with grouping logic |

---

## Enhanced Diagram Details

### 1. Decision Tree V2 #1: Scope 2 Emissions (Location-Based)

**File:** `images/decision_tree_1_scope2_emissions_v2.png` (426KB)

**Key Enhancements:**

#### XML→JSON→Database Mappings Shown
```
XML:  <xbrlConcept>esrs:GrossLocationBasedScope2GreenhouseGasEmissions</xbrlConcept>
JSON: xbrlConcepts[0]
PARSE: scope2_location

XML:  <unitType>mass</unitType>
JSON: mappingHints.positionMapping.unitType
DB:   Position.unit.type = 'mass'
```

#### Real SQL Queries at Each Filter
```sql
-- Filter 1
SELECT * FROM positions WHERE type = 'emission'
→ Result: 48 positions

-- Filter 2
AND unit_id IN (SELECT id FROM units WHERE type = 'mass')
→ Result: 28 positions (tCO2eq, kg CO2eq, MT CO2eq)

-- Filter 3
AND scope = 'scope2_location' OR tags LIKE '%scope2_location%'
→ Result: 8 positions

-- Filter 4
AND site_id IN (3419, 3420)
→ Result: 4 positions with data percentages shown
```

#### Concrete Position Examples
- Position 12345: "Electricity Location-Based Site A" - 100% data
- Position 12346: "Electricity Location-Based Site B" - 98% data
- Position 12347: "Steam Location-Based Site A" - 75% data
- Position 12348: "Heating Location-Based Site B" - 45% data

#### Complete Scoring Breakdown
```
Position 12345:
  Unit match mass:         +30
  Type match emission:     +25
  Scope match:             +20
  Data 100%:               +15
  Site match:              +10
  Recent 15 days:          +5
  ─────────────────────────────
  TOTAL: 105 → HIGH confidence
```

#### Full Output JSON
```json
{
  "questionId": "1.2",
  "suggestions": [
    {
      "positionId": 12345,
      "name": "Electricity Location-Based Site A",
      "unit": "tCO2eq",
      "scope": "scope2_location",
      "completeness": 1.00,
      "confidence": "HIGH",
      "score": 105,
      "factors": ["unit_match", "type_match", "scope_match", 
                  "high_completeness", "site_match", "recent_data"]
    }
  ]
}
```

---

### 2. Decision Tree V2 #2: Energy Calculation with Indicators

**File:** `images/decision_tree_2_energy_calculation_v2.png` (520KB)

**Key Enhancements:**

#### Calculation Branching Logic
```
XML:  <requiresCalculation>true</requiresCalculation>
JSON: mappingHints.positionMapping.requiresCalculation
LOGIC: Search INDICATORS not POSITIONS
```

#### Real Indicator Examples
- Indicator 4567: "Renewable Energy Share %"
  - Formula: `SUM(renewable) / SUM(total) * 100`
  - Unit: percentage
  - Input positions: 5 positions with data completeness

- Indicator 4568: "Renewable vs Non-Renewable" (excluded - wrong unit)
- Indicator 4569: "Green Energy Percentage" (excluded - missing Site B data)

#### Formula Validation Details
```
Parse indicator 4567 formula:
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Formula: SUM(renewable) / SUM(total) * 100
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Expected pattern:
  ✓ SUM or AGGREGATE numerator
  ✓ DIVIDE by denominator
  ✓ MULTIPLY by 100
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Renewable positions: [101, 102, 103]
Total positions: [104, 105]
✓ Formula structure matches expectation
```

#### Input Position Validation
```sql
SELECT i.*, COUNT(p.id) as position_count
FROM indicators i
JOIN indicator_positions ip ON i.id = ip.indicator_id
JOIN positions p ON ip.position_id = p.id
WHERE p.site_id IN (3419, 3420)
  AND EXISTS (
    SELECT 1 FROM transactions 
    WHERE position_id = p.id 
      AND date BETWEEN '2025-01-01' AND '2025-12-31'
  )
GROUP BY i.id
HAVING COUNT(*) >= i.required_positions
```

#### Input Positions with Completeness
```
✓ Indicator 4567 Input Positions:
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Position 101: Solar PV Site A        100% data
Position 102: Wind Energy Site A      95% data
Position 103: Biomass Site B          88% data
Position 104: Total Energy Site A    100% data
Position 105: Total Energy Site B     98% data
━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ All required positions have data
```

#### Full Output with Calculated Value
```json
{
  "questionId": "E1.5",
  "suggestions": [
    {
      "type": "indicator",
      "indicatorId": 4567,
      "name": "Renewable Energy Share %",
      "formula": "SUM(renewable) / SUM(total) * 100",
      "unit": "%",
      "confidence": "HIGH",
      "score": 105,
      "inputPositions": [
        {"id": 101, "name": "Solar PV Site A", "completeness": 1.00},
        {"id": 102, "name": "Wind Energy Site A", "completeness": 0.95},
        {"id": 103, "name": "Biomass Site B", "completeness": 0.88},
        {"id": 104, "name": "Total Energy Site A", "completeness": 1.00},
        {"id": 105, "name": "Total Energy Site B", "completeness": 0.98}
      ],
      "calculatedValue": 42.5
    }
  ]
}
```

---

### 3. Decision Tree V2 #3: Scope 3 Upstream Multi-Category

**File:** `images/decision_tree_3_scope3_upstream_v2.png` (484KB)

**Key Enhancements:**

#### Category Extraction Logic
```sql
SELECT p.*, p.scope3_category
FROM positions p
WHERE ...
GROUP BY p.scope3_category
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Category extraction from:
  - Position.scope3_category field
  - Position.tags LIKE '%cat1%'
  - Position.name pattern matching
```

#### GHG Protocol Categories with Examples
```
Category 1: Purchased Goods
━━━━━━━━━━━━━━━━━━━━━━━━━━━
3 positions found:
  P5001: Purchased Goods Emissions Site A
  P5002: Raw Materials Scope 3 Cat1
  I5003: Indicator - Purchased Goods GWP

Category 3: Fuel & Energy Related
━━━━━━━━━━━━━━━━━━━━━━━━━━━
4 positions found:
  P5010: Upstream Fuel Extraction
  P5011: T&D Losses Electricity
  I5012: Indicator - Fuel Chain GWP AR5
  I5013: Indicator - Energy T&D GWP AR6

Category 4: Upstream Transport
━━━━━━━━━━━━━━━━━━━━━━━━━━━
2 positions found:
  P5020: Freight Transport Upstream
  I5021: Indicator - Distance × EF
```

#### Indicator vs Position Detection
```
Category 1:
  → Has indicator I5003
  → Formula: SUM(spend) × emission_factor
  → Emission Factor: GWP AR5
  → ✓ Prefer indicator over direct position

Category 3:
  → Multiple indicators found
  → I5012: GWP AR5, 92% data, score 108
  → I5013: GWP AR6, 95% data, score 114 ← PREFERRED
```

#### Emission Factor Validation
```
I5003: Purchased Goods Indicator
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Formula: SUM(spend) × emission_factor
Emission Factor: GWP AR5
✓ Check: Formula contains 'GWP'
Input positions: [P5001, P5002]

I5013: Energy T&D GWP AR6
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Formula: energy_kwh × GWP_AR6
Emission Factor: CO2e per kWh
Data completeness: 95%
```

#### Per-Category Scoring
```
Category 1 - I5003:
  Type match:         +25
  Unit match:         +30
  Scope match:        +20
  GWP factor:         +10
  Data 95%:           +14
  Site match:         +10
  ─────────────────────────
  TOTAL: 109 → HIGH

Category 3 - I5013:
  All matches:        +85
  GWP AR6:            +10
  Data 95%:           +14
  Recent:             +5
  ─────────────────────────
  TOTAL: 114 → HIGH (BEST)

Category 3 - I5012:
  Similar but GWP AR5
  Data 92%:           +13
  ─────────────────────────
  TOTAL: 108 → HIGH

Category 4 - I5021:
  All matches:        +85
  Data 70%:           +10
  ─────────────────────────
  TOTAL: 95 → MEDIUM
```

#### Multi-Category Output
```json
{
  "questionId": "E1.8",
  "suggestions": [
    {
      "category": "Category 1: Purchased Goods",
      "positions": [
        {
          "type": "indicator",
          "id": 5003,
          "name": "Purchased Goods GWP",
          "emissionFactor": "GWP_AR5",
          "completeness": 0.95,
          "confidence": "HIGH",
          "score": 109
        }
      ]
    },
    {
      "category": "Category 3: Fuel & Energy",
      "positions": [
        {
          "type": "indicator",
          "id": 5013,
          "name": "Energy T&D GWP AR6",
          "emissionFactor": "GWP_AR6",
          "completeness": 0.95,
          "confidence": "HIGH",
          "score": 114
        },
        {
          "type": "indicator",
          "id": 5012,
          "name": "Fuel Chain GWP AR5",
          "emissionFactor": "GWP_AR5",
          "completeness": 0.92,
          "confidence": "HIGH",
          "score": 108
        }
      ]
    },
    {
      "category": "Category 4: Upstream Transport",
      "positions": [
        {
          "type": "indicator",
          "id": 5021,
          "name": "Transport Distance × EF",
          "completeness": 0.70,
          "confidence": "MEDIUM",
          "score": 95
        }
      ]
    }
  ]
}
```

---

## Key Improvements for SDD Documentation

### 1. Traceability
Every filter step now shows:
- ✅ Input XML attributes
- ✅ JSON field mappings
- ✅ Database queries (actual SQL)
- ✅ Intermediate results with counts
- ✅ Example position names and IDs

### 2. Concrete Examples
- Real XBRL concepts from ESRS taxonomy
- Actual position names users would see
- Specific sites (Site A = 3419, Site B = 3420)
- Real data completeness percentages
- Concrete scoring calculations

### 3. Validation Logic
- Formula parsing and validation
- Emission factor checking (GWP AR5 vs AR6)
- Input position verification
- Site data availability checks
- Category grouping logic

### 4. Complete Output
- Full JSON response structure
- All fields populated with examples
- Match factors explained
- Confidence scoring rationale
- Multiple suggestions per category

---

## Using V2 Diagrams in SDD

### Section 1: System Architecture
**Use:** `images/decision_tree_overview.png` (V1)  
**Purpose:** High-level architecture understanding

### Section 2: Filter Progression
**Use:** `images/filter_progression.png` (V1)  
**Purpose:** Conceptual filter stages

### Section 3: Example 1 - Direct Position Lookup
**Use:** `images/decision_tree_1_scope2_emissions_v2.png` (V2)  
**Purpose:** Complete walkthrough with real data for Scope 2 emissions

**Narrative:**
> "Consider a disclosure question asking for Scope 2 location-based emissions. The system extracts the XBRL concept `esrs:GrossLocationBasedScope2GreenhouseGasEmissions` and begins filtering 200+ positions. After applying type, unit, scope, and site filters with the SQL queries shown, only 4 positions remain. The scoring algorithm then evaluates each position's data completeness, with Position 12345 'Electricity Location-Based Site A' scoring 105 points due to 100% data coverage, earning a HIGH confidence rating."

### Section 4: Example 2 - Calculated Indicator
**Use:** `images/decision_tree_2_energy_calculation_v2.png` (V2)  
**Purpose:** Show indicator search with formula validation

**Narrative:**
> "When a question requires calculation (e.g., renewable energy percentage), the system searches indicators instead of direct positions. It validates the indicator's formula structure, checks that all input positions have data for the disclosure sites, and verifies the calculation logic matches expectations. Indicator 4567 'Renewable Energy Share %' passes all validations with 5 input positions averaging 96% data completeness."

### Section 5: Example 3 - Multi-Category Complex
**Use:** `images/decision_tree_3_scope3_upstream_v2.png` (V2)  
**Purpose:** Demonstrate category grouping and emission factor validation

**Narrative:**
> "For multi-category questions like Scope 3 upstream emissions, the system groups positions by GHG Protocol category, then determines whether to use indicators (with GWP factors) or direct positions. In Category 3 (Fuel & Energy), two indicators are found: I5012 using GWP AR5 (92% data) and I5013 using GWP AR6 (95% data). The system suggests both, ranking I5013 higher due to better data completeness and newer emission factor version."

---

## File Comparison

| Diagram | V1 Size | V2 Size | Size Increase | Reason |
|---------|---------|---------|---------------|--------|
| Scope 2 Emissions | 181KB | 426KB | +135% | Added SQL, data examples, scoring details |
| Energy Calculation | 181KB | 520KB | +187% | Added formula validation, input positions, JSON output |
| Scope 3 Upstream | 244KB | 484KB | +98% | Added category grouping, emission factors, multi-position output |

**Note:** Larger file sizes due to rich annotation text in diagrams, making them more suitable for SDD documentation and stakeholder communication.

---

## Files Location

**Directory:** `/Users/joakes/Projects/sofi/learned/disclosure/design/`

**V2 Files:**
```
images/decision_tree_1_scope2_emissions_v2.png       (426KB)
decision_tree_1_scope2_v2.mmd                 (source)

images/decision_tree_2_energy_calculation_v2.png     (520KB)
decision_tree_2_energy_v2.mmd                 (source)

images/decision_tree_3_scope3_upstream_v2.png        (484KB)
decision_tree_3_scope3_v2.mmd                 (source)
```

**V1 Files (preserved):**
```
images/decision_tree_1_scope2_emissions.png          (181KB)
images/decision_tree_2_energy_calculation.png        (181KB)
images/decision_tree_3_scope3_upstream.png           (244KB)
images/decision_tree_overview.png                    (104KB)
images/filter_progression.png                        (70KB)
images/decision_matrix.png                           (132KB)
```

---

## Regenerating V2 Diagrams

```bash
cd /Users/joakes/Projects/sofi/learned/disclosure/design

# Regenerate individual V2 diagrams
mmdc -i decision_tree_1_scope2_v2.mmd -o images/decision_tree_1_scope2_emissions_v2.png -w 1920 -H 1080 -b white
mmdc -i decision_tree_2_energy_v2.mmd -o images/decision_tree_2_energy_calculation_v2.png -w 1920 -H 1080 -b white
mmdc -i decision_tree_3_scope3_v2.mmd -o images/decision_tree_3_scope3_upstream_v2.png -w 1920 -H 1080 -b white

# Batch regenerate all V2
for f in *_v2.mmd; do mmdc -i "$f" -o "${f%.mmd}.png" -w 1920 -H 1080 -b white; done
```

---

## Summary

**V2 Enhancements Complete:**
- ✅ Real XBRL concepts and position examples
- ✅ Actual SQL queries at every filter step
- ✅ XML→JSON→Database field mappings shown explicitly
- ✅ Complete scoring breakdowns with point calculations
- ✅ Full JSON output examples with all fields
- ✅ Emission factor validation logic (GWP AR5, AR6)
- ✅ Category grouping for Scope 3
- ✅ Input position validation for indicators
- ✅ Data completeness percentages and examples

**Ready for:** SDD documentation with concrete, traceable examples

---

**Document Status:** Complete  
**Last Updated:** February 12, 2026  
**Total V2 Files:** 3 PNG + 3 Mermaid source files
