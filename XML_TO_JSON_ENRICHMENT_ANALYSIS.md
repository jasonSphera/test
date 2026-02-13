# XML to JSON Enrichment Analysis for Disclosure Templates

**Document Version:** 1.0  
**Date:** February 12, 2026  
**Author:** Development Team  
**Purpose:** Analyze XML structure and define enrichment attributes needed for position/report mapping

---

## Executive Summary

This document analyzes the current `disclosure_esrs_2024_v1.5.xml` template structure and identifies enrichment attributes needed to enable programmatic position and report mapping without requiring AI database access. The solution focuses on minimal schema changes while maximizing backward compatibility.

### Key Findings

1. **Current State**: XBRL concepts exist but lack contextual metadata
2. **Gap**: Missing unit type, scope, position type, and calculation indicators
3. **Solution**: Add metadata container to questions and columns
4. **Impact**: Schema extension (non-breaking) + backward compatible

---

## Current XML Structure Analysis

### 1. Question Structure

**Current Structure:**
```xml
<question id="cd23f3d4-6969-44fa-9270-e6543ab1b499" mandatory="true">
    <question xml:lang="en" allowHtml="true">§ 5</question>
    <xbrlConcepts>
        <xbrlConcept>esrs:BasisForPreparationOfSustainabilityStatement</xbrlConcept>
        <xbrlConcept>esrs:ScopeOfConsolidationOfConsolidatedSustainabilityStatementIsSameAsForFinancialStatements</xbrlConcept>
    </xbrlConcepts>
    <contentRelation id="acf76f06-b844-42a4-93ca-927b243e9df0"/>
    <griGuidance xml:lang="en" allowHtml="true">...</griGuidance>
    <peReportGuidance xml:lang="en" allowHtml="true">...</peReportGuidance>
</question>
```

**Observations:**
- ✅ XBRL concepts already present
- ❌ No unit type metadata
- ❌ No scope indicators
- ❌ No position type hints
- ❌ No calculation flags

### 2. Table Column Structure

**Current Structure:**
```xml
<column id="4b80ea22-5ecb-4133-82e7-5adbe2a02439">
    <name xml:lang="en">Name of metric</name>
    <text/>
</column>
```

**Observations:**
- ✅ Column types defined (<text/>, <number/>, <singleChoice/>)
- ❌ No XBRL concepts at column level (only comments)
- ❌ No unit type associations
- ❌ No position type hints

### 3. XBRL Concept Examples

**Emission-Related:**
```xml
<xbrlConcept>esrs:GrossScope1GreenhouseGasEmissions</xbrlConcept>
<xbrlConcept>esrs:GrossScope2GreenhouseGasEmissions</xbrlConcept>
<xbrlConcept>esrs:GrossScope3GreenhouseGasEmissions</xbrlConcept>
```

**Energy-Related:**
```xml
<xbrlConcept>esrs:EnergyConsumption</xbrlConcept>
<xbrlConcept>esrs:TotalEnergyConsumptionFromRenewableSources</xbrlConcept>
```

**Observations:**
- ✅ Concepts are well-named and semantic
- ✅ Scope information sometimes in concept name
- ❌ Unit type not explicitly stated
- ❌ Calculation requirements not indicated

---

## Gap Analysis

### Missing Attributes for Position Mapping

| Attribute | Purpose | Current State | Needed? |
|-----------|---------|---------------|---------|
| **unitType** | Filter positions by unit (mass, energy, area, etc.) | ❌ Not present | ✅ Critical |
| **scope** | Filter by emission scope (1, 2, 3, upstream, downstream) | ⚠️ Sometimes in concept name | ✅ Critical |
| **positionType** | Filter by position type (energy, emission, water, etc.) | ❌ Not present | ✅ Critical |
| **requiresCalculation** | Indicates if indicator/report needed | ❌ Not present | ✅ High Priority |
| **periodType** | Instant (date) vs Duration (span) | ❌ Not present | ✅ Medium Priority |
| **aggregationType** | Sum, average, max, min | ❌ Not present | ⚠️ Nice to have |
| **geographicScope** | Site, region, global | ❌ Not present | ⚠️ Nice to have |

### Missing Attributes for JSON Generation

| Attribute | Purpose | Current State | Needed? |
|-----------|---------|---------------|---------|
| **mappingHints** | Metadata container for enrichment | ❌ Not present | ✅ Critical |
| **positionFilters** | Structured filter criteria | ❌ Not present | ✅ High Priority |
| **reportFilters** | Report selection criteria | ❌ Not present | ✅ High Priority |
| **confidence** | Mapping confidence level | ❌ Not present | ⚠️ Nice to have |

---

## Proposed Enrichment Attributes

### Option 1: Inline Metadata (Recommended)

**Pros:**
- Self-contained within XML
- Version controlled with template
- Easy to maintain
- No external dependencies

**Cons:**
- Slightly larger XML files
- Requires schema extension

**Structure:**
```xml
<question id="379907aa-75e5-4206-a37d-2b1f963a4bc6" mandatory="true">
    <question xml:lang="en" allowHtml="true">Gross Scope 1 GHG Emissions</question>
    
    <!-- Existing XBRL Concepts -->
    <xbrlConcepts>
        <xbrlConcept>esrs:GrossScope1GreenhouseGasEmissions</xbrlConcept>
    </xbrlConcepts>
    
    <!-- NEW: Mapping Hints for Position/Report Selection -->
    <mappingHints>
        <positionMapping>
            <unitType>mass</unitType>
            <positionType>emission</positionType>
            <scope>scope1</scope>
            <requiresCalculation>false</requiresCalculation>
            <periodType>duration</periodType>
        </positionMapping>
        <reportMapping>
            <reportType>emission_summary</reportType>
            <scope>scope1</scope>
            <aggregation>sum</aggregation>
        </reportMapping>
        <confidence>high</confidence>
    </mappingHints>
    
    <contentRelation id="2fe46895-ec3a-47b9-897f-8954e8d2abea" />
    <griGuidance xml:lang="en" allowHtml="true">...</griGuidance>
</question>
```

### Option 2: External CSV Mapping

**Pros:**
- No XML schema changes
- Can be updated without redeploying templates

**Cons:**
- External dependency
- Risk of CSV/template sync issues
- Harder to version control

**Structure (CSV):**
```csv
question_sid,unit_type,position_type,scope,requires_calculation,period_type,confidence
379907aa-75e5-4206-a37d-2b1f963a4bc6,mass,emission,scope1,false,duration,high
cd23f3d4-6969-44fa-9270-e6543ab1b499,,,,,instant,medium
```

### Option 3: Hybrid Approach (Recommended Final)

**Pros:**
- Best of both worlds
- Schema-defined defaults + manual overrides
- Flexibility for edge cases

**Cons:**
- Slightly more complex implementation
- Two sources of truth (managed through priority)

**Priority Order:**
1. Manual CSV override (highest)
2. XML inline metadata
3. Inferred from XBRL concept name
4. Default values (lowest)

---

## XML Schema Extension Proposal

### New Elements

#### 1. `<mappingHints>` Container

**Purpose:** Group all enrichment metadata  
**Location:** Child of `<question>` and/or `<column>`  
**Cardinality:** 0..1 (optional for backward compatibility)

```xml
<xs:element name="mappingHints" minOccurs="0" maxOccurs="1">
    <xs:complexType>
        <xs:sequence>
            <xs:element name="positionMapping" minOccurs="0" maxOccurs="1"/>
            <xs:element name="reportMapping" minOccurs="0" maxOccurs="1"/>
            <xs:element name="confidence" type="xs:string" minOccurs="0"/>
        </xs:sequence>
    </xs:complexType>
</xs:element>
```

#### 2. `<positionMapping>` Element

**Purpose:** Define position selection criteria  
**Attributes:**

```xml
<xs:element name="positionMapping">
    <xs:complexType>
        <xs:sequence>
            <xs:element name="unitType" type="unitTypeEnum" minOccurs="0"/>
            <xs:element name="positionType" type="positionTypeEnum" minOccurs="0"/>
            <xs:element name="scope" type="scopeEnum" minOccurs="0"/>
            <xs:element name="requiresCalculation" type="xs:boolean" minOccurs="0"/>
            <xs:element name="periodType" type="periodTypeEnum" minOccurs="0"/>
            <xs:element name="aggregation" type="aggregationEnum" minOccurs="0"/>
        </xs:sequence>
    </xs:complexType>
</xs:element>
```

#### 3. `<reportMapping>` Element

**Purpose:** Define report selection criteria  
**Attributes:**

```xml
<xs:element name="reportMapping">
    <xs:complexType>
        <xs:sequence>
            <xs:element name="reportType" type="xs:string" minOccurs="0"/>
            <xs:element name="scope" type="scopeEnum" minOccurs="0"/>
            <xs:element name="aggregation" type="aggregationEnum" minOccurs="0"/>
            <xs:element name="formula" type="xs:string" minOccurs="0"/>
        </xs:sequence>
    </xs:complexType>
</xs:element>
```

### Enumerations

#### unitTypeEnum
```xml
<xs:simpleType name="unitTypeEnum">
    <xs:restriction base="xs:string">
        <xs:enumeration value="mass"/>
        <xs:enumeration value="energy"/>
        <xs:enumeration value="volume"/>
        <xs:enumeration value="area"/>
        <xs:enumeration value="distance"/>
        <xs:enumeration value="count"/>
        <xs:enumeration value="percentage"/>
        <xs:enumeration value="currency"/>
    </xs:restriction>
</xs:simpleType>
```

#### positionTypeEnum
```xml
<xs:simpleType name="positionTypeEnum">
    <xs:restriction base="xs:string">
        <xs:enumeration value="emission"/>
        <xs:enumeration value="energy"/>
        <xs:enumeration value="water"/>
        <xs:enumeration value="waste"/>
        <xs:enumeration value="resource"/>
        <xs:enumeration value="social"/>
        <xs:enumeration value="governance"/>
    </xs:restriction>
</xs:simpleType>
```

#### scopeEnum
```xml
<xs:simpleType name="scopeEnum">
    <xs:restriction base="xs:string">
        <xs:enumeration value="scope1"/>
        <xs:enumeration value="scope2"/>
        <xs:enumeration value="scope2_location"/>
        <xs:enumeration value="scope2_market"/>
        <xs:enumeration value="scope3"/>
        <xs:enumeration value="scope3_upstream"/>
        <xs:enumeration value="scope3_downstream"/>
        <xs:enumeration value="biogenic"/>
    </xs:restriction>
</xs:simpleType>
```

#### periodTypeEnum
```xml
<xs:simpleType name="periodTypeEnum">
    <xs:restriction base="xs:string">
        <xs:enumeration value="instant"/>
        <xs:enumeration value="duration"/>
    </xs:restriction>
</xs:simpleType>
```

#### aggregationEnum
```xml
<xs:simpleType name="aggregationEnum">
    <xs:restriction base="xs:string">
        <xs:enumeration value="sum"/>
        <xs:enumeration value="average"/>
        <xs:enumeration value="max"/>
        <xs:enumeration value="min"/>
        <xs:enumeration value="count"/>
    </xs:restriction>
</xs:simpleType>
```

---

## Backward Compatibility Strategy

### 1. Optional Elements

All new elements have `minOccurs="0"` - existing templates remain valid.

### 2. Graceful Degradation

**XML without enrichment:**
```xml
<question id="...">
    <xbrlConcepts>
        <xbrlConcept>esrs:GrossScope1GreenhouseGasEmissions</xbrlConcept>
    </xbrlConcepts>
</question>
```
**Behavior:** Fallback to concept name parsing + user selection

**XML with enrichment:**
```xml
<question id="...">
    <xbrlConcepts>
        <xbrlConcept>esrs:GrossScope1GreenhouseGasEmissions</xbrlConcept>
    </xbrlConcepts>
    <mappingHints>
        <positionMapping>
            <unitType>mass</unitType>
            <positionType>emission</positionType>
            <scope>scope1</scope>
        </positionMapping>
    </mappingHints>
</question>
```
**Behavior:** Enriched filtering + high-confidence suggestions

### 3. JSON Converter Updates

**Before:**
```json
{
    "sid": "379907aa-75e5-4206-a37d-2b1f963a4bc6",
    "question": "Gross Scope 1 GHG Emissions",
    "xbrlConcepts": ["esrs:GrossScope1GreenhouseGasEmissions"]
}
```

**After:**
```json
{
    "sid": "379907aa-75e5-4206-a37d-2b1f963a4bc6",
    "question": "Gross Scope 1 GHG Emissions",
    "xbrlConcepts": ["esrs:GrossScope1GreenhouseGasEmissions"],
    "mappingHints": {
        "positionMapping": {
            "unitType": "mass",
            "positionType": "emission",
            "scope": "scope1",
            "requiresCalculation": false,
            "periodType": "duration"
        },
        "reportMapping": {
            "reportType": "emission_summary",
            "scope": "scope1",
            "aggregation": "sum"
        },
        "confidence": "high"
    }
}
```

---

## Enrichment Examples

### Example 1: Scope 1 Emissions Question

**XML:**
```xml
<question id="379907aa-75e5-4206-a37d-2b1f963a4bc6" mandatory="true">
    <question xml:lang="en" allowHtml="true">Gross Scope 1 GHG emissions (tCO2eq)</question>
    
    <xbrlConcepts>
        <xbrlConcept>esrs:GrossScope1GreenhouseGasEmissions</xbrlConcept>
    </xbrlConcepts>
    
    <mappingHints>
        <positionMapping>
            <unitType>mass</unitType>
            <positionType>emission</positionType>
            <scope>scope1</scope>
            <requiresCalculation>false</requiresCalculation>
            <periodType>duration</periodType>
        </positionMapping>
        <reportMapping>
            <reportType>emission_summary</reportType>
            <scope>scope1</scope>
            <aggregation>sum</aggregation>
        </reportMapping>
        <confidence>high</confidence>
    </mappingHints>
    
    <griGuidance xml:lang="en" allowHtml="true">
        The disclosure on gross Scope 1 GHG emissions required by paragraph 44 (a) shall include:
        (a) the gross Scope 1 GHG emissions in metric tonnes of CO2eq;
    </griGuidance>
</question>
```

**Generated JSON:**
```json
{
    "sid": "379907aa-75e5-4206-a37d-2b1f963a4bc6",
    "question": "Gross Scope 1 GHG emissions (tCO2eq)",
    "mandatory": true,
    "xbrlConcepts": ["esrs:GrossScope1GreenhouseGasEmissions"],
    "mappingHints": {
        "positionMapping": {
            "unitType": "mass",
            "positionType": "emission",
            "scope": "scope1",
            "requiresCalculation": false,
            "periodType": "duration"
        },
        "reportMapping": {
            "reportType": "emission_summary",
            "scope": "scope1",
            "aggregation": "sum"
        },
        "confidence": "high"
    }
}
```

**Position Filtering Logic:**
```python
# Backend filtering service
positions = Position.query.filter(
    Position.site_id.in_(disclosure.site_ids),
    Position.unit.type == "mass",
    Position.type == "emission",
    Position.scope == "scope1",
    Position.has_transactions_in_period(disclosure.period)
).all()

# Score by data completeness
scored_positions = []
for pos in positions:
    completeness = pos.calculate_completeness(disclosure.period)
    scored_positions.append({
        'position': pos,
        'score': completeness,
        'confidence': 'high' if completeness > 0.8 else 'medium'
    })

# Return top 3
return sorted(scored_positions, key=lambda x: x['score'], reverse=True)[:3]
```

### Example 2: Energy Consumption Table

**XML:**
```xml
<table id="e1-energy-consumption-table">
    <columns>
        <column id="col-energy-type">
            <name xml:lang="en">Energy Type</name>
            <singleChoice>
                <enumeration>
                    <value key="RENEWABLE">Renewable</value>
                    <value key="NONRENEWABLE">Non-renewable</value>
                </enumeration>
            </singleChoice>
        </column>
        
        <column id="col-energy-amount">
            <name xml:lang="en">Total Consumption (MWh)</name>
            <number/>
            <xbrlConcept>esrs:EnergyConsumption</xbrlConcept>
            <mappingHints>
                <positionMapping>
                    <unitType>energy</unitType>
                    <positionType>energy</positionType>
                    <requiresCalculation>false</requiresCalculation>
                    <periodType>duration</periodType>
                </positionMapping>
                <confidence>high</confidence>
            </mappingHints>
        </column>
        
        <column id="col-renewable-pct">
            <name xml:lang="en">Renewable (%)</name>
            <number/>
            <xbrlConcept>esrs:PercentageOfEnergyConsumptionFromRenewableSources</xbrlConcept>
            <mappingHints>
                <positionMapping>
                    <unitType>percentage</unitType>
                    <positionType>energy</positionType>
                    <requiresCalculation>true</requiresCalculation>
                </positionMapping>
                <reportMapping>
                    <reportType>energy_mix</reportType>
                    <aggregation>average</aggregation>
                </reportMapping>
                <confidence>medium</confidence>
            </mappingHints>
        </column>
    </columns>
</table>
```

**Generated JSON:**
```json
{
    "tableId": "e1-energy-consumption-table",
    "columns": [
        {
            "id": "col-energy-type",
            "name": "Energy Type",
            "type": "singleChoice",
            "enumeration": {
                "RENEWABLE": "Renewable",
                "NONRENEWABLE": "Non-renewable"
            }
        },
        {
            "id": "col-energy-amount",
            "name": "Total Consumption (MWh)",
            "type": "number",
            "xbrlConcept": "esrs:EnergyConsumption",
            "mappingHints": {
                "positionMapping": {
                    "unitType": "energy",
                    "positionType": "energy",
                    "requiresCalculation": false,
                    "periodType": "duration"
                },
                "confidence": "high"
            }
        },
        {
            "id": "col-renewable-pct",
            "name": "Renewable (%)",
            "type": "number",
            "xbrlConcept": "esrs:PercentageOfEnergyConsumptionFromRenewableSources",
            "mappingHints": {
                "positionMapping": {
                    "unitType": "percentage",
                    "positionType": "energy",
                    "requiresCalculation": true
                },
                "reportMapping": {
                    "reportType": "energy_mix",
                    "aggregation": "average"
                },
                "confidence": "medium"
            }
        }
    ]
}
```

### Example 3: Complex Multi-Scope Question

**XML:**
```xml
<question id="scope3-upstream-emissions" mandatory="true">
    <question xml:lang="en" allowHtml="true">Scope 3 Upstream Emissions by Category</question>
    
    <xbrlConcepts>
        <xbrlConcept>esrs:GrossScope3GreenhouseGasEmissions</xbrlConcept>
        <xbrlConcept>esrs:GrossIndirectGHGEmissionsScope3Upstream</xbrlConcept>
    </xbrlConcepts>
    
    <mappingHints>
        <positionMapping>
            <unitType>mass</unitType>
            <positionType>emission</positionType>
            <scope>scope3_upstream</scope>
            <requiresCalculation>true</requiresCalculation>
            <periodType>duration</periodType>
        </positionMapping>
        <reportMapping>
            <reportType>scope3_summary</reportType>
            <scope>scope3_upstream</scope>
            <aggregation>sum</aggregation>
            <formula>SUM(scope3_categories WHERE upstream=true)</formula>
        </reportMapping>
        <confidence>medium</confidence>
    </mappingHints>
    
    <table id="scope3-breakdown">
        <columns>
            <column id="col-category">
                <name xml:lang="en">Scope 3 Category</name>
                <singleChoice>
                    <enumeration>
                        <value key="CAT1">Category 1: Purchased goods</value>
                        <value key="CAT2">Category 2: Capital goods</value>
                        <value key="CAT3">Category 3: Fuel and energy</value>
                        <value key="CAT4">Category 4: Upstream transport</value>
                    </enumeration>
                </singleChoice>
            </column>
            
            <column id="col-emissions">
                <name xml:lang="en">Emissions (tCO2eq)</name>
                <number/>
                <xbrlConcept>esrs:GrossIndirectGHGEmissionsScope3Upstream</xbrlConcept>
                <mappingHints>
                    <positionMapping>
                        <unitType>mass</unitType>
                        <positionType>emission</positionType>
                        <scope>scope3_upstream</scope>
                        <requiresCalculation>false</requiresCalculation>
                    </positionMapping>
                    <confidence>high</confidence>
                </mappingHints>
            </column>
        </columns>
    </table>
</question>
```

---

## XBRL to CS Unit Type Mapping

### Unit Type Mapping Table

| XBRL Type | CS Unit Type | Example Units | Position Types |
|-----------|--------------|---------------|----------------|
| `xbrli:massItemType` | `mass` | kg, t, MT, kg CO2eq | Emission, Waste |
| `xbrli:energyItemType` | `energy` | MWh, GJ, kWh | Energy |
| `xbrli:volumeItemType` | `volume` | m³, L, gal | Water, Fuel |
| `xbrli:areaItemType` | `area` | m², ha, km² | Land, Biodiversity |
| `xbrli:lengthItemType` | `distance` | km, mi, m | Transport |
| `xbrli:pureItemType` | `count` | units, items | Products, Assets |
| `esrs:percentItemType` | `percentage` | % | Ratios, Shares |
| `iso4217:*` | `currency` | EUR, USD | Financial |

### Inference Rules

**Rule 1: Concept Name Contains Scope**
```
"GrossScope1GreenhouseGasEmissions" → scope = "scope1"
"GrossScope2GreenhouseGasEmissions" → scope = "scope2"
"GrossScope3GreenhouseGasEmissions" → scope = "scope3"
```

**Rule 2: Concept Name Contains Unit Hint**
```
"*Emissions*" → unitType = "mass", positionType = "emission"
"*Energy*" → unitType = "energy", positionType = "energy"
"*Water*" → unitType = "volume", positionType = "water"
"*Waste*" → unitType = "mass", positionType = "waste"
```

**Rule 3: Calculation Keywords**
```
"Percentage*" → requiresCalculation = true
"*Intensity*" → requiresCalculation = true
"*Ratio*" → requiresCalculation = true
"Gross*" → requiresCalculation = false (direct measurement)
```

**Rule 4: Period Type from Display**
```
"as of [date]" → periodType = "instant"
"during [period]" → periodType = "duration"
"total *" → periodType = "duration"
```

---

## Implementation Roadmap

### Phase 1: Schema Extension (Week 1)
- [ ] Define XSD extensions for `<mappingHints>`
- [ ] Add enumerations (unitType, positionType, scope, etc.)
- [ ] Validate backward compatibility
- [ ] Update XML import parser

### Phase 2: JSON Converter Updates (Week 1-2)
- [ ] Extend JSON converter to include `mappingHints`
- [ ] Add fallback logic for missing attributes
- [ ] Implement inference rules from XBRL concepts
- [ ] Create unit/test cases

### Phase 3: Manual Enrichment (Week 2-3)
- [ ] Antonia tags high-priority ESRS questions
- [ ] Create CSV override mechanism
- [ ] Validate enrichment data quality
- [ ] Document enrichment process

### Phase 4: Position Filtering Service (Week 3-4)
- [ ] Build filtering algorithm
- [ ] Implement scoring logic
- [ ] Create API endpoint
- [ ] Add confidence calculation

### Phase 5: UI Integration (Week 4-5)
- [ ] Display suggestions in disclosure UI
- [ ] Show confidence scores
- [ ] Allow user override
- [ ] Track override patterns

### Phase 6: Testing & Refinement (Week 5-6)
- [ ] Test with real ESRS disclosure
- [ ] Measure accuracy (target: 80%+ auto-match)
- [ ] Refine scoring weights
- [ ] Document lessons learned

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Auto-match Accuracy** | 80%+ | % questions with correct top suggestion |
| **Position Reduction** | 100+ → 1-3 | Avg candidates shown per question |
| **User Override Rate** | <20% | % times user selects non-top suggestion |
| **Coverage** | 100% ESRS E1 | % questions with enrichment data |
| **Performance** | <2s | Filtering time per question |
| **Backward Compatibility** | 100% | Old templates still import |

---

## Risks & Mitigation

### Risk 1: Enrichment Data Quality
**Risk:** Manual tagging errors by Antonia  
**Mitigation:** 
- Validation rules in CSV import
- Automated consistency checks
- Review process before production

### Risk 2: Unit Type Mismatch
**Risk:** XBRL taxonomy units ≠ CS units  
**Mitigation:**
- Comprehensive mapping table
- Fallback to concept name parsing
- Allow manual override

### Risk 3: Schema Evolution
**Risk:** Future XBRL taxonomy changes  
**Mitigation:**
- Version mappings
- Extensible enum types
- Monitoring for taxonomy updates

### Risk 4: Performance at Scale
**Risk:** Filtering slow with many positions  
**Mitigation:**
- Database indexing on position attributes
- Caching of filter results
- Lazy loading of candidates

---

## Appendix: Detailed Mappings

### A. ESRS E1 Climate Change Mappings

| Question | XBRL Concept | Unit Type | Position Type | Scope | Calculation |
|----------|--------------|-----------|---------------|-------|-------------|
| Gross Scope 1 Emissions | esrs:GrossScope1GreenhouseGasEmissions | mass | emission | scope1 | false |
| Gross Scope 2 (Location) | esrs:GrossLocationBasedScope2GreenhouseGasEmissions | mass | emission | scope2_location | false |
| Gross Scope 2 (Market) | esrs:GrossMarketBasedScope2GreenhouseGasEmissions | mass | emission | scope2_market | false |
| Gross Scope 3 | esrs:GrossScope3GreenhouseGasEmissions | mass | emission | scope3 | false |
| Total Energy Consumption | esrs:TotalEnergyConsumption | energy | energy | - | false |
| Renewable Energy % | esrs:PercentageOfEnergyConsumptionFromRenewableSources | percentage | energy | - | true |

### B. Position Type Taxonomy

```
emission
  ├── scope1
  ├── scope2
  │   ├── location_based
  │   └── market_based
  ├── scope3
  │   ├── upstream
  │   └── downstream
  └── biogenic

energy
  ├── renewable
  ├── non_renewable
  └── total

water
  ├── consumption
  ├── withdrawal
  └── discharge

waste
  ├── hazardous
  └── non_hazardous

resource
  ├── material
  └── packaging
```

---

## Conclusion

The proposed enrichment strategy adds minimal complexity to the XML schema while enabling powerful programmatic position/report mapping. By combining inline metadata, manual CSV overrides, and intelligent inference rules, we can achieve 80%+ accuracy without requiring AI database access.

**Key Benefits:**
- ✅ Minimal schema changes
- ✅ Backward compatible
- ✅ Reduces user workload significantly
- ✅ No AI database access needed
- ✅ Extensible for future frameworks

**Next Steps:**
1. Review and approve schema extension
2. Begin Antonia's manual tagging for ESRS E1
3. Build position filtering service prototype
4. Test with real disclosure data

---

**Document Status:** Complete  
**Ready for:** Technical Review → SDD Creation → Implementation
