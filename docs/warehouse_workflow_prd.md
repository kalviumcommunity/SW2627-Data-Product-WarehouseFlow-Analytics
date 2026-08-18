# Warehouse Workflow Failure Analysis — Product Requirements Document

## 1. Business Problem

**Problem**

A grocery delivery startup tracks order preparation time, packing accuracy, and delivery complaints at the warehouse level, but operations leads currently have no consolidated way to identify which specific warehouse workflows are most associated with downstream failures (customer complaints, packing errors, delivery issues). Each signal — prep time, packing accuracy, complaints — is currently viewed separately, making it difficult to trace a downstream failure back to the operational step that caused it.

**Goal**

Identify:
- Which warehouses and/or workflows have the highest downstream failure rates.
- Whether longer preparation time, lower packing accuracy, or specific workflow patterns are most associated with complaints.
- Where in the warehouse process failures most commonly originate.

**Primary Users**

Operations Leads responsible for warehouse performance.

**Success Criteria**

The product will be considered successful when:
- Warehouses can be ranked by downstream failure rate.
- The dashboard identifies which operational signal (prep time, packing accuracy, or another factor) is most associated with complaints for each warehouse.
- All defined business questions (Section 6) can be answered directly from the dashboard.
- Dashboard results match validated Python/SQL analysis.

**Important:** This PRD does not claim a specific financial-loss figure or a pre-measured baseline complaint rate, as no verified business-impact baseline currently exists for this project.

## 2. Stakeholders

| Stakeholder Type | Who | Responsibility |
|---|---|---|
| Primary Users | Operations Leads | Identify underperforming warehouses/workflows and prioritize fixes |
| Secondary Users | Warehouse Managers / Business Analysts | Investigate specific workflow steps and validate root-cause findings |
| Data Owner | Team member responsible for the dataset / data pipeline | Confirm field definitions, data quality, and generation/collection logic |
| Approver | Project reviewer / team lead | Review and approve the PRD, artifact, and dashboard requirements before build |

## 3. Business Impact

**Operational Impact**

Operations Leads currently cannot connect a downstream complaint back to the specific warehouse step (preparation, packing, or another workflow stage) that caused it. This product combines prep-time, packing-accuracy, and complaint signals into a single view so failures can be traced to a workflow rather than investigated one report at a time.

**Revenue / Cost Impact**

No verified monetary-loss baseline is currently available for this project, so this PRD does not claim a specific cost figure. Instead, the product will surface failure counts and rates (complaints, packing errors) that can support a future cost estimate once finance/ops data is available.

**User Experience Impact**

Indirectly, downstream failures (late or incorrect deliveries, packing errors) affect the end customer's experience. This PRD focuses on the operational signals the warehouse can control — prep time and packing accuracy — as the levers most likely to reduce customer-facing complaints.

## 4. Dataset & Data Source Documentation

The fields below are based on the problem statement and the example order-level data shared for this project. They should be treated as the working data model for v1 planning.

**Dataset Overview**

| Property | Value |
|---|---|
| Grain | One row per order |
| Format | CSV/Excel/JSON or SQL table |
| Size | TBD |
| Time period | TBD |
| Warehouses | Multiple (referenced as W1, W2, etc.) |
| Primary key | `order_id` |

**Field Documentation** (based on the example row: `order_id, warehouse, prep_time, packing_accuracy, complaint`)

| Field | Type | Nullable? | Business Meaning |
|---|---|---|---|
| `order_id` | String/Int | No | Unique identifier for the order |
| `warehouse` | String/categorical | No | Identifies which warehouse fulfilled the order |
| `prep_time` (minutes) | Float/Int | Unlikely | Time taken to prepare the order |
| `packing_accuracy` | Float/percentage | Unlikely | Accuracy of the packing process for that order |
| `complaint` | Boolean (Yes/No) | No | Whether the order resulted in a downstream customer complaint |
| `workflow` / `workflow_step` | String/categorical | Possible | Identifies the warehouse workflow or process stage the order went through |
| `complaint_reason` | String/categorical | Yes | Reason category for the complaint, where applicable |
| `order_timestamp` | Datetime | No | Timestamp of order placement, used for time-based analysis |

**Data Verification Note**

The `workflow` field is central to this project's core question and should be confirmed against the source data early in development. If a dedicated workflow field is not available, workflow stages can be approximated using available proxies (e.g., binned preparation-time stages) or the analysis can be scoped to warehouse-level comparisons instead.

## 5. KPIs & Success Metrics

Each KPI below follows the required format: **Metric + Measurement Method + Numeric Target + Timeline**.

| KPI | Measurement Method | Target | Timeline |
|---|---|---|---|
| Complaint Rate | `orders with complaint = true / total orders`, per warehouse | Flag warehouses above 10% | Full dataset window |
| Average Prep Time | Mean `prep_time`, per warehouse | Flag warehouses above baseline threshold | Full dataset window |
| Packing Accuracy Rate | Mean `packing_accuracy`, per warehouse | Flag warehouses below 95% | Full dataset window |
| Prep-Time / Complaint Correlation | Correlation between `prep_time` and `complaint`, per warehouse | Flag correlations above defined threshold | Full dataset window |
| Packing-Accuracy / Complaint Correlation | Correlation between `packing_accuracy` and `complaint`, per warehouse | Flag correlations above defined threshold | Full dataset window |
| Workflow Failure Score | Composite of complaint rate, prep-time deviation, and packing-accuracy deviation, per workflow | Rank workflows; flag bottom performers | Full dataset window |

**KPI Threshold Note**

The numerical thresholds above are working project thresholds and will be validated against the actual dataset during analysis.

**Workflow Failure Score — placeholder formula**

```
Workflow Failure Score = 1 − (
    0.4 × (1 − complaint_rate)
  + 0.3 × (1 − normalized_prep_time_deviation)
  + 0.3 × packing_accuracy
)
```

The exact weighting will be finalized during the analysis phase once real score distributions are visible. The formula assumes a `workflow` field to group by — if unavailable, this KPI is redefined at the warehouse level instead.

## 6. Business Questions

| ID | Business Question |
|---|---|
| BQ1 | Which warehouses have the highest downstream failure (complaint) rates? |
| BQ2 | Does longer preparation time correlate with more complaints? |
| BQ3 | Does lower packing accuracy correlate with more complaints? |
| BQ4 | Which warehouse workflows are associated with the most downstream failures? |
| BQ5 | Is there a combination of factors (e.g., long prep time + low packing accuracy) that predicts higher failure rates? |
| BQ6 | Do failure patterns differ meaningfully across warehouses, or is the pattern consistent company-wide? |

## 7. Downstream Failure Model

Downstream failure will be evaluated using multiple signals rather than a single field:
- `complaint` — direct customer-facing failure signal.
- `prep_time` — process delay signal.
- `packing_accuracy` — process quality signal.

No single raw field will be treated as the sole measure of failure. The dashboard will show individual component metrics alongside any composite score, so the composite does not hide which underlying signal is driving a problem.

## 8. User Stories

Each story follows the required **Role + Action + Business Benefit** format.

**US-01** — As an Operations Lead, I want to compare downstream failure rates across warehouses, so that I can identify which warehouses need operational attention.

**US-02** — As an Operations Lead, I want to see the relationship between preparation time and complaint rate, so that I can determine whether slow prep is driving customer complaints.

**US-03** — As an Operations Lead, I want to see the relationship between packing accuracy and complaint rate, so that I can determine whether packing quality is driving customer complaints.

**US-04** — As an Operations Lead, I want to see failure rates broken down by workflow, so that I can pinpoint which specific process step is underperforming.

**US-05** — As a Warehouse Manager, I want to filter results by my own warehouse and time period, so that I can investigate patterns specific to my operation.

## 9. Product Scope

**In Scope — V1**
- Warehouse-level comparison of complaint rate, prep time, and packing accuracy
- Correlation analysis between prep time / packing accuracy and complaints
- Workflow-level failure analysis
- Interactive dashboard with filters
- Warehouse-level drill-down

**Out of Scope — V1**
- Real-time order tracking or live dispatch
- Automatically changing warehouse staffing or workflows
- Predictive/ML-based failure forecasting
- Individual employee-level performance tracking
- Mobile application
- Customer-facing features

## 10. Data Workflow Architecture

The product follows five stages:

```
1. INGESTION
      ↓
2. CLEANING & VALIDATION
      ↓
3. SQL / PYTHON ANALYSIS
      ↓
4. VISUALISATION
      ↓
5. DELIVERY
```

**Stage 1 — Ingestion**
Load the order-level dataset into the analysis environment.

**Stage 2 — Cleaning & Validation**
- `order_id` uniqueness
- Missing-value checks on `prep_time`, `packing_accuracy`, `complaint`
- Valid range checks (e.g., `packing_accuracy` between 0–100%, `prep_time` non-negative)
- Confirm the `workflow` field's category list

**Stage 3 — SQL / Python Analysis**
Generate: warehouse-level summaries, prep-time vs. complaint correlation, packing-accuracy vs. complaint correlation, workflow-level failure analysis.

**Stage 4 — Visualisation**
Streamlit (or equivalent) dashboard displaying KPI cards, warehouse comparisons, correlation views, and workflow breakdowns.

**Stage 5 — Delivery**
Dashboard delivered to Operations Leads and Warehouse Managers.

## 11. Dashboard Requirements

**Dashboard Layout (draft wireframe)**

```
┌──────────────────────────────────────────────────────────┐
│           WAREHOUSE WORKFLOW FAILURE DASHBOARD             │
├──────────────────────────────────────────────────────────┤
│ Total Orders │ Complaint Rate │ Avg Prep Time │ Avg Accuracy│
├──────────────────────────────────────────────────────────┤
│                 WAREHOUSE COMPARISON                       │
│ Warehouse | Complaint Rate | Prep Time | Packing Accuracy  │
├────────────────────────────┬──────────────────────────────┤
│ PREP TIME vs COMPLAINTS     │ PACKING ACCURACY vs COMPLAINTS│
│ (scatter / correlation)     │ (scatter / correlation)       │
├────────────────────────────┴──────────────────────────────┤
│                    WORKFLOW BREAKDOWN                      │
└──────────────────────────────────────────────────────────┘
```

**Sidebar Filters:** Warehouse, date range, workflow.

## 12. Risk & Assumption Analysis

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| The `workflow` field's exact category list is not yet finalized | Medium | High — BQ4, US-04, and the Workflow Failure Score all depend on it | Confirm field categories with the Data Owner early; if a dedicated field is unavailable, define workflow via a proxy (e.g., binned prep-time stages) |
| Correlation between prep time/packing accuracy and complaints may be mistaken for causation | Medium | Medium | Present findings as associations, not proven causes, in all dashboard copy |
| Composite Workflow Failure Score may hide which individual signal drives it | Medium | High | Always display component metrics (complaint rate, prep time, packing accuracy) alongside any composite score |
| KPI thresholds are working values not yet validated against real data | Medium | Medium | Validate all targets against the actual dataset during analysis before finalizing |

**Key Assumptions**
- The dataset is at order-level grain (one row per order).
- Core fields include `order_id`, `warehouse`, `prep_time`, `packing_accuracy`, and `complaint`.
- A `workflow` field exists or can be derived from available data.
- Complaint is recorded as a binary Yes/No outcome per order.
- No monetary-loss baseline currently exists for this project; cost impact will be estimated once finance/ops data is available.

## 13. Data Quality & Validation Requirements

Required checks before KPI calculations:
- `order_id` uniqueness
- Missing-value checks on all core fields
- Valid range validation (`packing_accuracy` 0–100%, `prep_time` ≥ 0)
- Boolean/categorical validation on `complaint`
- Validation of the `workflow` field's category list

## 14. Final Success Criteria

The product is ready for final review when:
- All business questions (Section 6) are answerable from the dashboard.
- Warehouses can be ranked by complaint rate.
- The relationship between prep time, packing accuracy, and complaints is clearly visualized.
- Workflow-level analysis is delivered and traceable to the underlying data.
- Dashboard results can be traced back to the underlying dataset.

## 15. Final Product Question

Which warehouse workflows are most associated with downstream failures — customer complaints, packing errors, or delivery issues — and what operational signals (preparation time, packing accuracy, or others) best explain that association?

*End of PRD*
