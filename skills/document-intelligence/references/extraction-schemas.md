# Extraction schemas by document type

Every field is `{ value, unit, page, verbatim_source_text }` (see SKILL.md). Missing →
`null` and listed in `gaps[]`. Add fields freely when a document offers more; never drop
a required field silently.

## utility_bill

```json
{
  "utility_name": "", "account_number": "", "service_address": "",
  "meter_number": "", "fuel_type": "electric|gas|water|steam|other",
  "period_start": "", "period_end": "",
  "usage": { "value": 0, "unit": "kWh|therms|kGal|Mlb" },
  "demand": { "value": 0, "unit": "kW" },
  "cost_total": 0, "cost_breakdown": { "supply": 0, "delivery": 0, "taxes_fees": 0 },
  "rate_schedule": "", "gaps": []
}
```
Sanity: cost/usage within plausible regional $/unit band; period ≈ 25–35 days unless
clearly a multi-month bill.

## energy_audit

```json
{
  "audit_level": "ASHRAE 1|2|3|other", "audit_date": "", "auditor": "",
  "baseline_site_eui": { "value": 0, "unit": "kBtu/ft2yr" },
  "baseline_year": "", "gross_floor_area": { "value": 0, "unit": "ft2" },
  "recommendations": [{
    "measure": "", "system_category": "HVAC|envelope|lighting|controls|DHW|renewables|other",
    "est_cost": 0, "est_savings_kwh": 0, "est_savings_therms": 0,
    "est_savings_usd": 0, "simple_payback_yrs": 0
  }],
  "total_savings_potential_pct": 0, "gaps": []
}
```

## lease (clause extraction)

```json
{
  "clause_type": "utilities|capex_passthrough|green_lease|alterations|data_sharing|other",
  "clause_text_verbatim": "", "section_ref": "",
  "landlord_obligation": "", "tenant_obligation": "",
  "cost_recovery_allowed": null,
  "green_lease_indicators": ["energy data sharing", "capex cost recovery", "sustainability standards"],
  "gaps": []
}
```

## pca (property condition assessment)

```json
{
  "assessment_date": "", "assessor": "",
  "findings": [{
    "deficiency": "", "system_category": "structure|envelope|HVAC|plumbing|electrical|conveyance|site|life_safety",
    "severity": "immediate|short_term|long_term",
    "immediate_cost": 0, "reserve_cost_5yr": 0, "eul_remaining_yrs": null
  }],
  "replacement_reserve_total": 0, "gaps": []
}
```
PCA capital tables are prime chart-parser targets — run `parse_pdf_tables` first.

## rent_roll

```json
{
  "as_of_date": "",
  "units": [{ "unit": "", "type": "", "sqft": 0, "tenant": "", "lease_start": "",
              "lease_end": "", "monthly_rent": 0, "status": "occupied|vacant|notice" }],
  "occupancy_pct": 0, "gaps": []
}
```
Always via `parse_pdf_tables` when tabular.

## om / regulatory_filing / architectural_drawing / other

No fixed schema — extract the fields the requesting workflow needs (RSRA has its own
extraction list), cite pages, and store what was pulled. Drawings: floor areas, system
schedules, and equipment schedules are the usual targets; digitize schedule tables via
the chart parser.
