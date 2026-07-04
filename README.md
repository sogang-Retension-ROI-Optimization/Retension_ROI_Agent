<div align="center">

# Retention ROI Agent

**An operational Retention Intelligence Copilot that goes beyond churn prediction to decide where, when, and how retention budget should be spent**

Upload a single CSV/TSV file, and the system connects customer churn risk, expected churn timing, intervention effect, customer value, budget constraints, personalized actions, and real-time action queues into one decision-making flow.

[Demo Video](https://drive.google.com/file/d/1WRAoRtl88iwrsZRCmMKu1qhgs2dDfbE5/view?usp=sharing) · [Differentiation Strategy](docs/product_differentiation.md) · [Dashboard Decision Loop](docs/dashboard_decision_loop.md) · [Decision Logic](docs/decision_logic.md) · [Technical Guide](docs/technical_guide.md) · [Presentation](docs/presentation.pdf)

</div>

---

## Core Differentiation of This Project

Most churn analytics stop at “Who is likely to leave?” Retention ROI Agent goes one step further and calculates **whether it is economically worthwhile to retain that customer**, **when to intervene**, **whether a coupon, consultation, push message, or wait action is better**, and **whether spending an additional KRW 1 million will increase profit**.

> **Customers who are likely to churn** and **customers who actually generate profit when retained** are not the same.  
> This platform creates a **Retention ROI decision**, not just a churn probability.

| Existing Solution | Limitation | How Retention ROI Agent Is Different |
| --- | --- | --- |
| General churn prediction model | Shows only the top-N risk customers; budget, action, and timing decisions are handled separately by humans | Combines Churn × Uplift × CLV × Cost × Timing to select **which customers and actions should be executed within budget** |
| GA4 / Amplitude / Mixpanel-style behavior analytics | Strong for funnel and event analysis, but customer-level intervention economics require separate work | Converts event logs into customer-level decision tables and connects them to **retention targets, expected ROI, and intervention timing** |
| Salesforce Marketing Cloud / Braze / HubSpot-style campaign tools | Strong for campaign delivery and automation, but deciding “who should receive how much investment for profit” requires separate analysis | Ranks targets before campaign execution using **incremental effect and expected revenue relative to cost** |
| Tableau / Power BI dashboards | Focused on status reporting. Changing conditions does not automatically recalculate decision candidates | When thresholds, budget, or target caps change, **target customers, segment budgets, recommended actions, and ROI are recalculated together** |
| Kaggle-style churn notebook / AutoML PoC | Focused on model accuracy and feature importance | Implements **Live DB, action queue, counterfactual lab, and LLM Q&A** that can be operated directly from the dashboard |

A more detailed comparison is provided in [Differentiation Strategy](docs/product_differentiation.md).

---

## Core Features

### 1. Industry-Specific Data Onboarding: Finance / E-commerce Modes

Instead of being fixed to simulator data from the beginning, the platform interprets uploaded data by allowing the user to select either **Finance Mode** or **E-commerce Mode**.

- Finance: churn/inactivity risk analysis based on deposits, loans, cards, transactions, balances, delinquency, and consultation history
- E-commerce: revisit/purchase churn analysis based on visits, searches, carts, purchases, coupons, and category preferences

<img src="assets/dash1.png" width="720" />

### 2. Automatic CSV/TSV Mapping and Event Standardization

Even when uploaded column names vary, the system automatically detects customer ID, event timestamp, event type, amount, category, and churn-label candidates. Event values are also mapped to internal standard types so that downstream modeling and real-time event processing use the same schema.

<img src="assets/dash2.png" width="720" />

### 3. Churn Status: Not Just Risk Scores, but an Operational Starting Point

The dashboard shows the total number of customers, number of risky customers, risk-customer ratio, and average churn probability. Before moving into cohort retention, segmentation, and Uplift/CLV analysis, it helps the user quickly identify “which customer groups are the problem.”

<img src="assets/dash3.png" width="720" />

### 4. Churn-Timing Prediction: When Should We Intervene?

Based on Survival Analysis, the system calculates each customer’s expected churn timing, probability of churn within 30 days, and expected loss. Instead of simply saying “high risk,” it converts risk into operational timing such as **contact immediately within 14 days**, **contact within 15–30 days**, or **plan a contact within 31–60 days**.

<img src="assets/dash5.png" width="720" />

### 5. Budget Allocation and Target Customers: Calculating Marginal ROI of Retention Budget

The platform recalculates customer-level intervention candidates based on the entered total budget, churn threshold, and maximum number of target customers. This screen is the most distinctive part of the project for competitions and demos.

- Automatically selects final target customers within the budget
- Calculates segment-level budget allocation and expected net profit
- Calculates the expected net-profit increase from spending an additional KRW 1 million
- Displays saturated budget ranges and low-efficiency budget ranges
- Prevents excessive cost concentration through a cap on the share of high-intensity interventions

<img src="assets/dash6.png" width="720" />

<img src="assets/dash7.png" width="720" />

<img src="assets/dash8.png" width="720" />

### 6. Customer-Level Response Strategy Comparison: Counterfactual Retention Lab

For the same customer, the system compares expected net profit across scenarios such as **no intervention, KRW 5,000 benefit, consultation call, push/email, and 7-day wait**. The recommended action does not end at “the model chose it”; the system explains why that action is better than no intervention or alternative actions.

For detailed logic, see the [Counterfactual Lab document](docs/counterfactual_retention_lab.md).

<img src="assets/dash9.png" width="720" />

<img src="assets/dash10.png" width="720" />


### 7. Personalized Recommendations for Final Target Customers

The platform does not simply display previously saved recommendation candidates. It regenerates recommendations **only for the final retention target customers** selected by the current budget, churn threshold, and maximum target conditions on the screen.

Recommendation scores combine each customer’s past purchase/transaction history, recent interest signals, similar-segment preferences, and overall popularity signals. In Finance Mode, categories and reason text are converted into financial-product and financial-behavior language.

<img src="assets/dash11.png" width="720" />

### 8. Real-Time Operations Monitor: Updating Not Only Scores, but Also the Action Queue

When events enter the PostgreSQL Live DB, customer status, churn score, recommendation candidates, and the action queue are updated together. The dashboard allows the user to check the number of events, total number of customers, queued-action count, latest score-update time, and action-queue details.

- FastAPI event ingestion
- Customer feature-state updates
- Churn/CLV/uplift rescoring
- Action-queue insertion based on expected ROI
- Automatic generation of new and existing customer events through a demo stream

<img src="assets/dash12.png" width="720" />

### 9. Screen-Aware AI Chatbot

The LLM does not read the entire dataset blindly. Instead, it answers based on the summary payload of the current dashboard screen. Users can ask questions such as “Why is this segment risky?”, “What changes if the budget increases?”, and “Which customers should we contact first?” while preserving the exact context of the current screen.

<img src="assets/dash4.png" width="260" />

---

## Decision Flow

```text
CSV/TSV upload
  → Automatic column-role detection
  → Event-value standardization
  → Churn-criteria configuration
  → Churn / Survival / Uplift / CLV calculation
  → Budget-constrained target and action optimization
  → Personalized recommendation generation
  → PostgreSQL Live DB seed
  → Score and action-queue updates when new events are received
```

The core logic is designed to answer the following questions, not merely to produce “prediction scores.”

| Question | What the Platform Calculates |
| --- | --- |
| Who is at risk? | churn probability, risk segment |
| When should we intervene? | predicted time to churn, timing urgency, recommended intervention window |
| Is the customer worth retaining? | CLV, expected loss, expected incremental profit |
| Is the customer likely to respond to intervention? | uplift score, persuadable segment |
| How much should we spend? | coupon/action cost, budget allocation, marginal ROI |
| What should we do? | recommended action, intervention intensity, next best recommendation |
| Should this customer be placed in the operations queue now? | live score, expected ROI, action queue status |

The calculation formulas and constraints are summarized in [Decision Logic](docs/decision_logic.md).

---

## Supported Domains

| Mode | Target Industries | Example Data | Main Decisions |
| --- | --- | --- | --- |
| **Finance Mode** | Banks, card companies, fintech, insurance/wealth management | Deposits/withdrawals, loan repayments, card payments, balance changes, delinquency, consultation history | Identify customers at risk of cancellation/inactivity and prioritize consultations, benefits, and product guidance |
| **E-commerce Mode** | Online stores, subscription commerce, marketplaces | Visits, searches, carts, purchases, coupon usage, category preferences | Select targets for revisit/repurchase activation, recommend coupons/categories, and create CRM action queues |

The workflow can proceed even if the uploaded data is a customer snapshot. If sufficient event logs are available, behavior time-series analysis, churn-timing estimation, and real-time operations analysis become richer.

---

## Quick Start

```bash
# 1. Start services
docker compose up -d --build

# 2. Open the dashboard
open http://localhost:8501
```

For detailed installation, API examples, directory structure, and validation checklist, see the [Technical Guide](docs/technical_guide.md).

---

## Documents

| Document | Description |
| --- | --- |
| [Differentiation Strategy](docs/product_differentiation.md) | Differentiation from existing churn dashboards, CRM, CDP, and BI tools |
| [Dashboard Decision Loop](docs/dashboard_decision_loop.md) | Explains how the seven core screens lead to business decisions |
| [Decision Logic](docs/decision_logic.md) | Calculation methods for budget optimization, counterfactual analysis, personalized recommendations, and live action queues |
| [Technical Guide](docs/technical_guide.md) | Installation, API, directory structure, and validation checklist |
| [Analysis Process](docs/analysis_process.md) | Modeling, survival analysis, and real-time deployment flow |
| [Feature Dictionary](docs/feature_dictionary.md) | Main generated features and their meanings |
| [Retention Strategy](docs/retention_strategy.md) | Segment-level retention strategies and cost/effect assumptions |
| [Counterfactual Lab](docs/counterfactual_retention_lab.md) | Logic for comparing expected profit and loss by action against no intervention |
| [Presentation](docs/presentation.pdf) | Project presentation slides |

---
