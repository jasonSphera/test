# Decision Tree Visual Diagram Index

**Version:** 1.0  
**Date:** February 12, 2026  
**Purpose:** Quick reference to all decision tree visualization diagrams

---

## Overview Diagrams

### 1. System Overview
**File:** `images/decision_tree_overview.png`  
**Purpose:** High-level view of the complete position mapping system  
**Shows:**
- Attribute extraction toolkit
- 4-stage filtering process
- Scoring factors (30-5 points)
- Output structure with confidence scores

**Key Insight:** Shows how 200+ positions are reduced through progressive filtering stages.

---

### 2. Filter Progression
**File:** `images/filter_progression.png`  
**Purpose:** Linear flow showing reduction at each filter stage  
**Shows:**
- Filter 1: Position Type (200 → 50)
- Filter 2: Unit Type (50 → 30)
- Filter 3: Scope (30 → 10)
- Filter 4: Site Assignment (10 → 5)
- Filter 5: Data Completeness (5 → 1-3)

**Key Insight:** Each filter eliminates non-matching positions, with final scoring on remaining candidates.

---

### 3. Decision Matrix
**File:** `images/decision_matrix.png`  
**Purpose:** Attribute-based routing and branching logic  
**Shows:**
- Calculation requirement branching (Position vs. Indicator)
- Position type routing (Emission, Energy, Water, Waste)
- Scope sub-filtering (Scope 1/2/3, Location/Market)
- Formula validation for indicators
- Emission factor checking

**Key Insight:** Different question attributes lead to different search strategies.

---

## Detailed Decision Trees

### 4. Decision Tree #1: Scope 2 Emissions
**File:** `images/decision_tree_1_scope2_emissions.png`  
**Question Type:** Gross Scope 2 GHG emissions (location-based)  
**Complexity:** Medium  
**Shows:**
- 5-step filtering process
- Direct position lookup (no calculation)
- Scope-specific filtering
- Site and data completeness checks

**Use Case:** Standard emission questions with direct measurement data.

---

### 5. Decision Tree #2: Energy Calculation
**File:** `images/decision_tree_2_energy_calculation.png`  
**Question Type:** Percentage of renewable energy  
**Complexity:** High  
**Shows:**
- Calculation requirement detection
- Indicator search instead of position
- Formula keyword matching
- Input position validation
- Formula correctness verification

**Use Case:** Questions requiring calculated values (percentages, ratios, intensities).

---

### 6. Decision Tree #3: Scope 3 Upstream
**File:** `images/decision_tree_3_scope3_upstream.png`  
**Question Type:** Scope 3 upstream emissions by category  
**Complexity:** Very High  
**Shows:**
- Category-based grouping
- Indicator vs. Position branching
- GWP emission factor verification
- Per-category scoring
- Multi-position output (1-2 per category)

**Use Case:** Complex multi-category questions with potential calculated emissions.

---

## Mermaid Source Files

All diagrams are generated from Mermaid `.mmd` files:

| Diagram | Source File |
|---------|-------------|
| Overview | `decision_tree_overview.mmd` |
| Filter Progression | `filter_progression.mmd` |
| Decision Matrix | `decision_matrix.mmd` |
| Scope 2 Emissions | `decision_tree_1_scope2.mmd` |
| Energy Calculation | `decision_tree_2_energy.mmd` |
| Scope 3 Upstream | `decision_tree_3_scope3.mmd` |

---

## Regenerating Diagrams

To regenerate diagrams with updated styling or content:

```bash
cd /Users/joakes/Projects/sofi/learned/disclosure/design

# Generate individual diagrams
mmdc -i decision_tree_overview.mmd -o images/decision_tree_overview.png -w 1920 -H 1080 -b white
mmdc -i filter_progression.mmd -o images/filter_progression.png -w 1920 -H 1080 -b white
mmdc -i decision_matrix.mmd -o images/decision_matrix.png -w 1920 -H 1080 -b white

# Generate decision trees
mmdc -i decision_tree_1_scope2.mmd -o images/decision_tree_1_scope2_emissions.png -w 1920 -H 1080 -b white
mmdc -i decision_tree_2_energy.mmd -o images/decision_tree_2_energy_calculation.png -w 1920 -H 1080 -b white
mmdc -i decision_tree_3_scope3.mmd -o images/decision_tree_3_scope3_upstream.png -w 1920 -H 1080 -b white

# Batch regenerate all
for f in *.mmd; do mmdc -i "$f" -o "${f%.mmd}.png" -w 1920 -H 1080 -b white; done
```

---

## Color Coding Legend

| Color | Purpose | Hex Code |
|-------|---------|----------|
| Light Blue | Question Input | `#e1f5ff` |
| Light Yellow | Filter Stage | `#fff4e1` |
| Light Red | Calculation Branch | `#ffe1e1` |
| Light Purple | Scoring Stage | `#f0e1ff` |
| Light Green | Pass/Match | `#d4edda` |
| Dark Green | Final Output | `#5cb85c` |
| Light Pink | Excluded/Fail | `#f8d7da` |
| Dark Red | Excluded Pool | `#d9534f` |

---

## Usage in Documentation

### Markdown Reference
```markdown
![Decision Tree Overview](images/decision_tree_overview.png)
![Filter Progression](images/filter_progression.png)
![Decision Matrix](images/decision_matrix.png)
![Scope 2 Emissions](images/decision_tree_1_scope2_emissions.png)
![Energy Calculation](images/decision_tree_2_energy_calculation.png)
![Scope 3 Upstream](images/decision_tree_3_scope3_upstream.png)
```

### HTML Reference
```html
<img src="images/decision_tree_overview.png" alt="Decision Tree Overview" width="100%" />
```

---

## Diagram Summary

| Diagram | Type | Complexity | Nodes | Use Case |
|---------|------|------------|-------|----------|
| Overview | Architecture | Low | 25 | System understanding |
| Filter Progression | Flow | Low | 20 | Process visualization |
| Decision Matrix | Logic | High | 30+ | Attribute routing |
| Scope 2 Emissions | Tree | Medium | 24 | Standard questions |
| Energy Calculation | Tree | High | 20 | Calculated questions |
| Scope 3 Upstream | Tree | Very High | 32 | Complex multi-category |

---

## Related Documents

- **Main Analysis:** `DECISION_TREE_POSITION_MAPPING_ANALYSIS.md`
- **XML Enrichment:** `XML_TO_JSON_ENRICHMENT_ANALYSIS.md`
- **Meeting Summary:** `../meetings/AI_DISCLOSURE_MEETING_SUMMARY_FEB_11_2026.md`
- **SDD Document:** `SDD_Disclosure_Template_Enrichment_for_Smart_Mapping.md`

---

**Document Status:** Complete  
**Last Updated:** February 12, 2026  
**Total Diagrams:** 6 PNG files + 6 Mermaid source files
