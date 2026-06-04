# Why AI Projects Fail: A Data-Driven Analysis of Organizational Adoption

**A full-stack analytics project — Python · SQL · Tableau**  
**300 simulated AI projects · 8 industries · 5 regions · 2019–2024**

***

## About

This project investigates the structural and organizational reasons behind AI project failures using a synthetic dataset of 300 enterprise AI initiatives spanning 8 industries, 5 global regions, and 6 years (2019–2024).

The analysis replicates the kind of post-mortem diagnostic work done by consulting firms and AI governance teams — identifying which organizational factors (data readiness, leadership support, talent, change management, governance) most strongly predict whether an AI project succeeds or fails.

The full pipeline covers data generation, exploratory analysis, statistical testing, SQL-based querying, and an interactive Tableau dashboard.

***

## Repository Contents

```
why-ai-fails/
│
├── data/
│   └── ai_projects.csv          # 300 synthetic AI projects (19 columns)
│
├── python/
│   ├── generate_data.py         # Synthetic dataset generator
│   └── analysis.py              # Full EDA, statistical tests, visualizations
│
├── sql/
│   └── queries.sql              # Schema + 10 analytical queries + dashboard view
│
├── outputs/
│   └── ai_failure_analysis.png  # 6-panel matplotlib figure
│
└── README.md
```

***

## Dataset Description

### `ai_projects.csv` — 300 AI Project Records

| Column | Description | Type |
|---|---|---|
| `project_id` | Unique project identifier (e.g. AIPROJ-1001) | String |
| `industry` | One of 8 industries | Categorical |
| `company_size` | Small / Mid-size / Large / Enterprise | Categorical |
| `region` | One of 5 global regions | Categorical |
| `project_type` | e.g. NLP/Chatbot, Computer Vision, GenAI | Categorical |
| `outcome` | Failed / Partial Failure / Ongoing (At Risk) / Succeeded | Categorical |
| `budget_usd_m` | Project budget in USD millions | Float |
| `duration_months` | Project duration in months | Integer |
| `primary_failure_reason` | Top reason for failure (10 categories) | Categorical |
| `secondary_failure_reason` | Secondary failure reason | Categorical |
| `data_readiness_score` | Score 1–10: quality and availability of training data | Integer |
| `leadership_support_score` | Score 1–10: executive sponsorship strength | Integer |
| `talent_score` | Score 1–10: availability of AI/ML talent | Integer |
| `change_mgmt_score` | Score 1–10: organizational change management | Integer |
| `year_launched` | Year project was initiated (2019–2024) | Integer |
| `roi_pct` | Return on investment percentage (null for At Risk) | Float |
| `has_dedicated_ai_team` | Whether org had a dedicated AI team | Boolean |
| `used_external_vendor` | Whether external AI vendor was used | Boolean |
| `has_data_governance` | Whether formal data governance existed | Boolean |

**Sample size:** 300 projects  
**Outcome distribution:** ~35% Failed · ~27% Partial Failure · ~23% Succeeded · ~15% Ongoing (At Risk)  
**Overall failure/partial-failure rate:** ~62%

***

**Data Generation Methodology**

Real enterprise AI project data is almost never publicly available — companies don't publish internal post-mortems, failure rates are underreported, and sensitive organizational details stay behind NDAs. To enable rigorous analysis, we generated a synthetic dataset of 300 AI projects calibrated against published research (McKinsey Global AI Survey, Gartner failure rate studies, Stanford HAI Index Report). The ~62% failure rate, budget ranges by company size, and score distributions were all anchored to real-world estimates before a single row was generated.

Critically, the data was built with structural realism — not random assignment. Succeeded projects were sampled from higher score distributions (data readiness ~7.5, leadership ~7.8) while failed projects drew from lower ones (~4.5, ~4.2), governance presence raised success probability, and budgets scaled with company size. This means correlations, t-tests, and chi-square tests return statistically meaningful results rather than noise. A fixed random seed (np.random.seed(42)) ensures full reproducibility across runs.

***

## Python Pipeline

### Step 1 — Generate Dataset

```bash
python python/generate_data.py
```

Generates `data/ai_projects.csv` with 300 synthetic but statistically realistic records. The generator uses calibrated normal distributions for scores, industry-specific outcome probabilities, and budget ranges by company size.

### Step 2 — Run Analysis

```bash
python python/analysis.py
```

Executes the following steps in order:

| Step | What it does |
|---|---|
| 1. Load & validate | Reads CSV, checks dtypes, prints shape and outcome distribution |
| 2. EDA | Outcome counts, average scores by outcome, top failure reasons |
| 3. Statistical tests | Two-sample t-test (data readiness: Succeeded vs Failed); Chi-square (governance vs success); Pearson correlations with success binary |
| 4. Visualizations | 6-panel matplotlib figure saved to `outputs/` |
| 5. Impact tables | Governance × AI team crosstabs showing success rates |

### Expected Outputs

```
outputs/
└── ai_failure_analysis.png    # 6-panel analysis figure
```

**Six panels:**
- Panel 1: Project outcome distribution (bar chart)
- Panel 2: Top 10 primary failure reasons (horizontal bar)
- Panel 3: Key factor boxplots — Failed vs Succeeded
- Panel 4: Failure rate by industry (with average reference line)
- Panel 5: Budget vs ROI scatter (coloured by outcome)
- Panel 6: Failure rate trend by year launched (line chart)

***

## SQL

### Schema

```sql
CREATE TABLE ai_projects (
    project_id               VARCHAR(20) PRIMARY KEY,
    industry                 VARCHAR(50),
    company_size             VARCHAR(30),
    region                   VARCHAR(50),
    project_type             VARCHAR(50),
    outcome                  VARCHAR(30),
    budget_usd_m             DECIMAL(8,2),
    duration_months          INTEGER,
    primary_failure_reason   VARCHAR(80),
    secondary_failure_reason VARCHAR(80),
    data_readiness_score     INTEGER,
    leadership_support_score INTEGER,
    talent_score             INTEGER,
    change_mgmt_score        INTEGER,
    year_launched            INTEGER,
    roi_pct                  DECIMAL(8,1),
    has_dedicated_ai_team    BOOLEAN,
    used_external_vendor     BOOLEAN,
    has_data_governance      BOOLEAN
);
```

Compatible with: **PostgreSQL · SQLite · SQL Server**

### Analytical Queries

| Query | What it answers |
|---|---|
| Q1: Outcome Distribution | Counts, % share, avg budget and duration per outcome |
| Q2: Failure Reasons Ranked | Top failure reasons by frequency and avg ROI |
| Q3: Industry Failure Rates | Failure %, total projects, avg ROI by industry |
| Q4: Success Factor Scores | Avg scores for all 4 factors + composite score by outcome |
| Q5: ROI by Project Type | Success count, avg ROI (success vs failure) by project type |
| Q6: Governance & AI Team Impact | Success/failure rate by governance × AI team combination |
| Q7: Year-over-Year Trend | Failure rate trend from 2019–2024 |
| Q8: Budget Tier Analysis | Failure rate and avg ROI by budget bracket |
| Q9: Regional Breakdown | Failure rate, avg readiness and change mgmt score by region |
| Q10: Dashboard Master View | `CREATE VIEW vw_ai_dashboard` — source for Tableau |

***

## Tableau Dashboard

The dashboard connects directly to `vw_ai_dashboard` (SQL View from Q10) or `ai_projects.csv`.

### Pages

| Page | Charts |
|---|---|
| Overview | KPI cards (Total, Failure %, Avg Budget, Budget Lost) · Donut: outcomes · Bar: failure reasons |
| Industry Deep Dive | Failure rate by industry · Score matrix by industry |
| Success Factors | Composite score by outcome · Scatter: budget vs ROI · Score gap gauge |
| Governance & Teams | Grouped bar: success rate with/without governance and AI team |
| Trends | Line: failure rate by year · Area: projects by year and outcome |
| Project Explorer | Full data table with slicers |

### Key DAX Measures (Power BI compatible)

```dax
Failure Rate % =
DIVIDE(
    COUNTROWS(FILTER(ai_projects,
        ai_projects[outcome] IN {"Failed", "Partial Failure"})),
    COUNTROWS(ai_projects)
) * 100

Composite Readiness Score =
AVERAGEX(ai_projects,
    (ai_projects[data_readiness_score]
   + ai_projects[leadership_support_score]
   + ai_projects[talent_score]
   + ai_projects[change_mgmt_score]) / 4
)
```

***

## Key Findings

| Shock / Factor | Effect | Evidence |
|---|---|---|
| Poor Data Quality | #1 failure driver (40/300 projects) | EDA + Q2 |
| Leadership Support | Avg 7.8/10 (Succeeded) vs 4.2/10 (Failed) | T-test p < 0.001 |
| Data Governance | ~25% higher success rate with governance | Chi-square p < 0.05 |
| Dedicated AI Team | Success rate ~35% (with) vs ~20% (without) | Q6 |
| Budget vs ROI | No significant positive correlation | Scatter / Q5 |
| Manufacturing | Highest failure rate: 69.4% | Q3 |
| Retail | Lowest failure rate: 48.6% | Q3 |
| Risk / VIX trend | Failure rate peaked 2022 at 73.1%, declining to 62.5% in 2024 | Q7 |

**Core finding:** Organizational and governance factors — not budget size — are the strongest predictors of AI project success. A project with strong leadership support, data readiness, and formal governance is approximately 3× more likely to succeed than one without, regardless of budget.

***

## Installation

```bash
git clone https://github.com/YOUR_USERNAME/why-ai-fails.git
cd why-ai-fails
pip install -r requirements.txt
```

**Required packages:**
```
numpy>=1.24
pandas>=2.0
matplotlib>=3.7
seaborn>=0.12
scipy>=1.10
scikit-learn>=1.3
```

***

## Usage

```bash
# Step 1: Generate dataset
python python/generate_data.py

# Step 2: Run full analysis
python python/analysis.py

# Step 3: Load ai_projects.csv into Tableau or run queries.sql in your SQL client
```

***

## Author

**Mayank**  
Department of Economic Sciences,
IIT Kanpur, India
May 2026

***

## License

This project is for educational and portfolio purposes.  
The dataset is fully synthetic — no real company or individual data is included.

***

## Acknowledgements

Inspired by real-world AI adoption research including:
- McKinsey Global Survey on AI Adoption (2023)
- Gartner AI Failure Rate Studies (2022–2024)
- Stanford HAI Index Report (2024)
