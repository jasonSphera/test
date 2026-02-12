# Position Mapping Decision Tree - Visual Diagrams

**Document Version:** 1.0  
**Date:** February 12, 2026  
**Purpose:** Visual decision tree diagrams for position mapping (optimized for export to images)

---

## Overview Diagram

```mermaid
graph TD
    A[Disclosure Question] --> B[Extract XBRL Concept]
    B --> C[Parse Attributes]
    C --> D{Position Type Filter}
    D -->|emission| E1[120 positions]
    D -->|energy| E2[80 positions]
    D -->|water| E3[40 positions]
    
    E1 --> F{Scope Filter}
    F -->|scope1| G1[30 positions]
    F -->|scope2_location| G2[25 positions]
    F -->|scope3| G3[65 positions]
    
    G2 --> H{Unit Type Filter}
    H -->|mass| I[15 positions]
    
    I --> J{Site Filter}
    J --> K[8 positions]
    
    K --> L{Data Period Filter}
    L --> M[5 positions]
    
    M --> N[Score by Completeness]
    N --> O[Top 3 Suggestions]
    
    style A fill:#e1f5ff
    style O fill:#c3f0c3
    style D fill:#fff4e6
    style F fill:#fff4e6
    style H fill:#fff4e6
```

---

## Example 1: Scope 2 GHG Emissions (Location-Based)

### Question Details Box

```
╔═══════════════════════════════════════════════════════════════╗
║  QUESTION 1.2: Gross Scope 2 Location-Based GHG Emissions    ║
║  XBRL: esrs:GrossLocationBasedScope2GreenhouseGasEmissions   ║
║  Unit: tCO2eq                                                 ║
║  Type: Number                                                 ║
║  Mandatory: Yes                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

### Decision Flow with Mermaid

```mermaid
flowchart TD
    Start([Question 1.2<br/>Scope 2 Emissions]) --> Extract[Extract from XBRL Tag]
    
    Extract --> Attr1{{"🏷️ Attribute 1<br/>Position Type"}}
    Attr1 -->|emission| Filter1["✂️ Filter by Type<br/>500 → 120 positions"]
    
    Filter1 --> Attr2{{"🎯 Attribute 2<br/>Scope"}}
    Attr2 -->|scope2_location| Filter2["✂️ Filter by Scope<br/>120 → 25 positions"]
    
    Filter2 --> Attr3{{"⚖️ Attribute 3<br/>Unit Type"}}
    Attr3 -->|mass tCO2eq| Filter3["✂️ Filter by Unit<br/>25 → 15 positions"]
    
    Filter3 --> Attr4{{"📍 Attribute 4<br/>Site Assignment"}}
    Attr4 -->|disclosure sites| Filter4["✂️ Filter by Site<br/>15 → 8 positions"]
    
    Filter4 --> Attr5{{"📅 Attribute 5<br/>Reporting Period"}}
    Attr5 -->|2024 data exists| Filter5["✂️ Filter by Data<br/>8 → 5 positions"]
    
    Filter5 --> Attr6{{"📊 Attribute 6<br/>Data Completeness"}}
    Attr6 --> Score["Score & Rank<br/>By Quality"]
    
    Score --> Result([🎯 Top 3 Results])
    
    Result --> R1["1️⃣ Scope 2 Purchased Electricity<br/>Confidence: 92%"]
    Result --> R2["2️⃣ Scope 2 Heating/Cooling<br/>Confidence: 85%"]
    Result --> R3["3️⃣ Scope 2 District Energy<br/>Confidence: 78%"]
    
    style Start fill:#e3f2fd
    style Result fill:#c8e6c9
    style R1 fill:#a5d6a7
    style R2 fill:#a5d6a7
    style R3 fill:#a5d6a7
    style Attr1 fill:#fff9c4
    style Attr2 fill:#fff9c4
    style Attr3 fill:#fff9c4
    style Attr4 fill:#fff9c4
    style Attr5 fill:#fff9c4
    style Attr6 fill:#fff9c4
```

### Attribute Toolkit Table

```
┏━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┓
┃ ATTRIBUTE          ┃ VALUE               ┃ FILTER RESULT       ┃
┡━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━┩
│ 1. Position Type   │ emission            │ 500 → 120 positions │
│ 2. Scope           │ scope2_location     │ 120 → 25 positions  │
│ 3. Unit Type       │ mass                │ 25 → 15 positions   │
│ 4. Site            │ [Site A, Site B]    │ 15 → 8 positions    │
│ 5. Reporting Period│ 2024-01 to 2024-12  │ 8 → 5 positions     │
│ 6. Completeness    │ Score & Rank        │ 5 → 3 suggestions   │
└────────────────────┴─────────────────────┴─────────────────────┘
```

---

## Example 2: Renewable Energy Percentage

### Question Details Box

```
╔═══════════════════════════════════════════════════════════════╗
║  QUESTION E1-5: Percentage of Renewable Energy               ║
║  XBRL: esrs:PercentageOfEnergyConsumptionFromRenewableSources║
║  Unit: %                                                      ║
║  Type: Calculated (Indicator or Report)                      ║
║  Mandatory: Yes                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

### Decision Flow with Mermaid

```mermaid
flowchart TD
    Start([Question E1-5<br/>Renewable Energy %]) --> Extract[Extract from XBRL Tag]
    
    Extract --> Attr1{{"🏷️ Attribute 1<br/>Position Type"}}
    Attr1 -->|energy| Filter1["✂️ Filter by Type<br/>500 → 80 positions"]
    
    Filter1 --> Attr2{{"📐 Attribute 2<br/>Unit Type"}}
    Attr2 -->|percentage| Decision{"🤔 Decision Point<br/>Calculation Required?"}
    
    Decision -->|Yes| Attr3{{"🧮 Attribute 3<br/>Requires Calculation"}}
    Attr3 -->|true| PathSplit{Split Path}
    
    PathSplit -->|Path A| PathA["🔍 Search Indicators<br/>With Formula"]
    PathSplit -->|Path B| PathB["📊 Search Reports<br/>With Energy Mix"]
    
    PathA --> Formula{{"🔢 Attribute 4<br/>Formula Type"}}
    Formula -->|ratio A/B| FilterA["✂️ Filter Indicators<br/>80 → 12 positions"]
    
    PathB --> ReportType{{"📈 Report Metadata"}}
    ReportType -->|has renewable breakdown| FilterB["✂️ Filter Reports<br/>20 → 8 reports"]
    
    FilterA --> Merge[Combine Results]
    FilterB --> Merge
    
    Merge --> Attr5{{"📍 Attribute 5<br/>Site Assignment"}}
    Attr5 -->|disclosure sites| Filter5["✂️ Filter by Site<br/>20 → 5 options"]
    
    Filter5 --> Attr6{{"📅 Attribute 6<br/>Data Period"}}
    Attr6 -->|numerator & denominator exist| Filter6["✂️ Filter by Data<br/>5 → 2-3 options"]
    
    Filter6 --> Result([🎯 Top 2 Results])
    
    Result --> R1["INDICATOR:<br/>1️⃣ Renewable Energy Mix<br/>Confidence: 75%"]
    Result --> R2["REPORT:<br/>2️⃣ Energy Summary Report<br/>Confidence: 70%"]
    
    style Start fill:#e3f2fd
    style Result fill:#c8e6c9
    style R1 fill:#ffecb3
    style R2 fill:#b3e5fc
    style Decision fill:#ffccbc
    style PathSplit fill:#ffccbc
    style Attr1 fill:#fff9c4
    style Attr2 fill:#fff9c4
    style Attr3 fill:#fff9c4
    style Formula fill:#fff9c4
    style ReportType fill:#fff9c4
    style Attr5 fill:#fff9c4
    style Attr6 fill:#fff9c4
```

### Dual Path Visualization

```
                    ┌─────────────────────────────────┐
                    │  Calculation Required = TRUE     │
                    └──────────────┬──────────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
         ┌──────────▼──────────┐      ┌─────────▼──────────┐
         │   PATH A:           │      │   PATH B:          │
         │   Indicators        │      │   Reports          │
         └──────────┬──────────┘      └─────────┬──────────┘
                    │                           │
         ┌──────────▼──────────┐      ┌─────────▼──────────┐
         │ Look for Formula    │      │ Look for Energy    │
         │ Type: ratio (A/B)   │      │ Mix Breakdown      │
         └──────────┬──────────┘      └─────────┬──────────┘
                    │                           │
         ┌──────────▼──────────┐      ┌─────────▼──────────┐
         │ 12 Indicators       │      │ 8 Reports          │
         │ Found               │      │ Found              │
         └──────────┬──────────┘      └─────────┬──────────┘
                    │                           │
                    └──────────┬────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Merge & Filter     │
                    │  by Site + Period   │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Final: 2-3 Options │
                    └─────────────────────┘
```

---

## Example 3: Total Energy Consumption (Table with Hierarchy)

### Question Details Box

```
╔═══════════════════════════════════════════════════════════════╗
║  QUESTION E1-4: Total Energy Consumption by Source           ║
║  XBRL: esrs:TotalEnergyConsumption                           ║
║  Format: Table (Source | MWh | %)                            ║
║  Type: Hierarchical with breakdown                           ║
║  Mandatory: Yes                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

### Decision Flow with Mermaid

```mermaid
flowchart TD
    Start([Question E1-4<br/>Energy Consumption Table]) --> Extract[Extract from XBRL Tag]
    
    Extract --> Attr1{{"🏷️ Attribute 1<br/>Position Type"}}
    Attr1 -->|energy| Filter1["✂️ Filter by Type<br/>500 → 80 positions"]
    
    Filter1 --> Attr2{{"⚖️ Attribute 2<br/>Unit Type"}}
    Attr2 -->|energy MWh, GJ| Filter2["✂️ Filter by Unit<br/>80 → 60 positions"]
    
    Filter2 --> Attr3{{"📊 Attribute 3<br/>Dimension"}}
    Attr3 -->|by_source| Decision{"🤔 Decision Point<br/>Grouped or Individual?"}
    
    Decision -->|Grouped| PathA["🌳 PATH A:<br/>Parent Position"]
    Decision -->|Individual| PathB["📋 PATH B:<br/>Multiple Positions"]
    
    PathA --> HierarchyA{{"🏗️ Attribute 4a<br/>Has Children?"}}
    HierarchyA -->|yes| FilterA["✂️ Find Parent with<br/>Child Positions"]
    
    PathB --> HierarchyB{{"🔗 Attribute 4b<br/>Same Parent?"}}
    HierarchyB -->|siblings| FilterB["✂️ Find Siblings<br/>Under Same Parent"]
    
    FilterA --> Merge[Combine Results]
    FilterB --> Merge
    
    Merge --> Attr5{{"📍 Attribute 5<br/>Site Assignment"}}
    Attr5 -->|disclosure sites| Filter5["✂️ Filter by Site<br/>15 → 8 groups"]
    
    Filter5 --> Attr6{{"📅 Attribute 6<br/>Data Period"}}
    Attr6 -->|all children have data| Filter6["✂️ Filter by Data<br/>8 → 3 groups"]
    
    Filter6 --> Score["Score by<br/>Data Completeness"]
    
    Score --> Result([🎯 Best Match])
    
    Result --> R1["🌳 Total Energy Consumption 95%<br/>├─ Renewable Energy ✓<br/>├─ Fossil Fuel Energy ✓<br/>└─ Nuclear Energy ✓"]
    
    style Start fill:#e3f2fd
    style Result fill:#c8e6c9
    style R1 fill:#a5d6a7
    style Decision fill:#ffccbc
    style PathA fill:#b3e5fc
    style PathB fill:#b3e5fc
    style Attr1 fill:#fff9c4
    style Attr2 fill:#fff9c4
    style Attr3 fill:#fff9c4
    style HierarchyA fill:#fff9c4
    style HierarchyB fill:#fff9c4
    style Attr5 fill:#fff9c4
    style Attr6 fill:#fff9c4
```

### Hierarchical Structure Visualization

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃  PARENT POSITION: Total Energy Consumption                   ┃
┃  Unit: MWh | Type: energy | Has Children: Yes                ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
┏━━━━━━━━━▼━━━━━━━━━┓ ┏━━━━━━▼━━━━━━━━┓ ┏━━━━━━▼━━━━━━━━┓
┃ CHILD 1:          ┃ ┃ CHILD 2:       ┃ ┃ CHILD 3:       ┃
┃ Renewable Energy  ┃ ┃ Fossil Fuel    ┃ ┃ Nuclear Energy ┃
┃ 15,000 MWh        ┃ ┃ 45,000 MWh     ┃ ┃ 10,000 MWh     ┃
┃ Data: ✓ Complete  ┃ ┃ Data: ✓ Complete┃ ┃ Data: ✓ Complete┃
┗━━━━━━━━━━━━━━━━━━━┛ ┗━━━━━━━━━━━━━━━━┛ ┗━━━━━━━━━━━━━━━━┛
          │                   │                   │
    ┌─────┴─────┐       ┌─────┴─────┐            │
    │           │       │           │            │
┌───▼───┐   ┌───▼───┐ ┌─▼─┐     ┌───▼───┐      │
│ Solar │   │ Wind  │ │Coal│     │Natural│      │
│ PV    │   │ Farm  │ │    │     │ Gas   │      │
└───────┘   └───────┘ └────┘     └───────┘      │
```

---

## Complete Attribute Toolkit (Visual Reference)

```mermaid
mindmap
  root((Attribute<br/>Toolkit))
    PRIMARY
      positionType
        emission
        energy
        water
        waste
      unitType
        mass
        energy
        volume
        percentage
      scope
        scope1
        scope2_location
        scope2_market
        scope3
      requiresCalculation
        true: indicator/report
        false: direct position
      site
        from disclosure
      reportingPeriod
        from disclosure
    SECONDARY
      hierarchy
        parent_id
        has_children
        siblings
      formula
        ratio
        sum
        weighted_average
      emissionFactor
        GWP_AR5
        GWP_AR6
      dimension
        by_source
        by_region
        by_category
      periodType
        instant
        duration
      aggregation
        sum
        average
        max
        min
```

---

## Filtering Progression Chart

```mermaid
graph LR
    A[500 Total<br/>Positions] -->|Position Type| B[120<br/>Emission]
    A -->|Position Type| C[80<br/>Energy]
    A -->|Position Type| D[40<br/>Water]
    
    B -->|Scope| E[25<br/>Scope 2 Loc]
    E -->|Unit Type| F[15<br/>Mass CO2eq]
    F -->|Site| G[8<br/>Assigned Sites]
    G -->|Period| H[5<br/>Has Data]
    H -->|Score| I[3<br/>Top Suggestions]
    
    style A fill:#ffccbc
    style B fill:#fff9c4
    style E fill:#e1bee7
    style F fill:#c5e1a5
    style G fill:#b3e5fc
    style H fill:#a5d6a7
    style I fill:#69f0ae
```

---

## Success Metrics Dashboard

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                    SUCCESS METRICS DASHBOARD                  ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

┌─────────────────────────────────────────────────────────────┐
│ Position Reduction                                          │
├─────────────────────────────────────────────────────────────┤
│ Before: ████████████████████████████████████ 100+ positions│
│ After:  ███ 1-5 positions                                   │
│ Target: 80% reduction ✓                                     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Auto-Match Accuracy                                         │
├─────────────────────────────────────────────────────────────┤
│ High Confidence:   ████████████████ 70%                     │
│ Medium Confidence: ████████ 20%                             │
│ Low Confidence:    ███ 10%                                  │
│ Target: 80%+ accuracy ✓                                     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ User Override Rate                                          │
├─────────────────────────────────────────────────────────────┤
│ Accepted Top Suggestion: ████████████████████ 82%          │
│ Override to #2 or #3:    ████ 13%                          │
│ Manual Selection:        █ 5%                              │
│ Target: <20% override ✓                                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Question Type Coverage Matrix

```mermaid
quadrantChart
    title Position Mapping Confidence by Question Type
    x-axis Low Complexity --> High Complexity
    y-axis Low Confidence --> High Confidence
    quadrant-1 Manual Review Needed
    quadrant-2 Medium Accuracy
    quadrant-3 High Automation
    quadrant-4 Target Zone
    Scope 1/2 Emissions: [0.7, 0.9]
    Direct Energy: [0.6, 0.85]
    Water Consumption: [0.65, 0.88]
    Percentage Calculations: [0.55, 0.7]
    Multi-Position Aggregate: [0.8, 0.65]
    Hierarchical Tables: [0.75, 0.8]
    Custom Indicators: [0.85, 0.6]
```

---

## Implementation Roadmap Gantt

```mermaid
gantt
    title Position Mapping Implementation Timeline
    dateFormat YYYY-MM-DD
    section Phase 1
    Schema Extension         :a1, 2026-02-12, 7d
    JSON Converter Update    :a2, after a1, 7d
    section Phase 2
    Manual Enrichment (ESRS) :b1, after a2, 7d
    CSV Import Tool          :b2, after a2, 5d
    section Phase 3
    Filtering Service        :c1, after b1, 10d
    Scoring Algorithm        :c2, after b1, 10d
    section Phase 4
    UI Integration           :d1, after c1, 7d
    User Testing             :d2, after d1, 7d
    section Phase 5
    Production Rollout       :e1, after d2, 5d
    Monitoring & Tuning      :e2, after e1, 14d
```

---

## Export Instructions

### For PNG/SVG Export:

1. **Mermaid Diagrams:** 
   - Copy each mermaid block
   - Use: https://mermaid.live or GitHub markdown preview
   - Export as PNG or SVG

2. **ASCII Diagrams:**
   - Use monospace font (Courier New, Consolas)
   - Screenshot or convert to image

3. **Recommended Tools:**
   - Mermaid Live Editor: https://mermaid.live
   - Markdown Preview Enhanced (VSCode extension)
   - Carbon (for code screenshots): https://carbon.now.sh

### For Presentation:

- **PowerPoint/Keynote:** Import SVG exports
- **Confluence/Wiki:** Paste mermaid code directly (if supported)
- **Documentation:** Use markdown with mermaid rendering

---

**Document Status:** Visual Export Ready  
**Optimized For:** Presentations, Documentation, Technical Reviews
