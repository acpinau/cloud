# Azure Budget Assessment Tool

This script analyzes Azure cost budgets across Management Groups, Subscriptions, and Resource Groups. It calculates recent usage, compares it against existing budgets, highlights missing budgets, and suggests more appropriate budget values based on historical and forecasted spend.

The script generates a single Excel workbook with dedicated tabs for each scope:

- MG_Budgets
- Sub_Budgets
- Sub_NoBudget
- RG_Budgets

---

## Features

- Enumerates all Management Groups and Subscriptions under a root Management Group.
- Reads budgets at all Azure scopes:
  - Management Group
  - Subscription
  - Resource Group
- For each scope:
  - Queries last N months of actual cost
  - Queries current month forecast
  - Computes cost trends and budget accuracy
  - Suggests improved budgets with 10 percent headroom
- Identifies subscriptions missing budgets and recommends values
- Supports filtering by:
  - Subscription IDs (`--subscription-ids`)
  - Resource Group names (`--rg-names`)
- Robust retry logic for all ARM and Cost APIs
- Verbose logging for visibility
- Excel workbook output with multiple sheets

---

## Prerequisites

### Python and packages

You need Python 3.x installed.

Install required packages:

```bash
pip install azure-identity requests python-dateutil openpyxl
```

### Azure Authentication

This script uses Azure's DefaultAzureCredential and supports:

#### 1. Browser Login (default)
A browser window will open automatically when you run the script.

#### 2. Azure CLI Login
Run:

```bash
az login
```

The script will authenticate using AzureCliCredential.

### Required Permissions

| Component | Required RBAC Role |
|----------|--------------------|
| Management Groups | Reader + Cost Management Reader |
| Subscriptions | Reader + Cost Management Reader |
| Resource Groups | Reader |

If you use subscription filtering, you only need access to those subscriptions.

---

## Usage Syntax

Base syntax:

```bash
python budget_assessment.py <management_group_id> [options]
```

Example:

```bash
python budget_assessment.py mg-demo
```

---

## Scope Options

### All scopes (default)

```bash
python budget_assessment.py mg-demo
```

### MG only

```bash
python budget_assessment.py mg-demo --scopes mg
```

### Subscriptions only

```bash
python budget_assessment.py mg-demo --scopes sub
```

### Resource Groups only

```bash
python budget_assessment.py mg-demo --scopes rg
```

### Subscriptions + RGs

```bash
python budget_assessment.py mg-demo --scopes sub,rg
```

---

## Filtering Options

### Filter by Subscription IDs

One subscription:

```bash
python budget_assessment.py mg-demo --subscription-ids <sub-id>
```

Multiple subscriptions:

```bash
python budget_assessment.py mg-demo --subscription-ids <sub1> <sub2> <sub3>
```

### Filter by Resource Groups

Applies to all included subscriptions:

```bash
python budget_assessment.py mg-demo --rg-names rg1 rg2 rg3
```

---

## Output Options

### Custom output file

```bash
python budget_assessment.py mg-demo --out demo_Budget_Assessment.xlsx
```

### Reduce cost API usage (faster)

```bash
python budget_assessment.py mg-demo --months 1
```

### Verbose logging

```bash
python budget_assessment.py mg-demo --verbose
```

---

## Full Argument Reference

```
positional arguments:
  management_group_id       Root Management Group ID (for example mg-demo)

optional arguments:
  --out OUT                 Output Excel (.xlsx)
  --months MONTHS           Past months to query (default 3)
  --scopes SCOPES           mg,sub,rg (default = mg,sub,rg)
  --verbose                 Enable detailed logging
  --subscription-ids ...    Only include specified subscriptions
  --rg-names ...            Only include specified resource groups
```

---

## Excel Workbook Structure

| Sheet Name | Meaning |
|------------|---------|
| MG_Budgets | MG-level budgets |
| Sub_Budgets | Subscriptions with budgets |
| Sub_NoBudget | Subscriptions missing budgets |
| RG_Budgets | RG-level budgets |

Only sheets with data are generated.

---

## Column Definitions

Each row includes:

- ScopeType  
- ScopeId  
- SubscriptionName  
- SubscriptionId  
- ResourceGroup  
- BudgetName  
- BudgetAmount  
- BudgetTimeGrain  
- BudgetStartDate  
- BudgetEndDate  
- ConditionKey  
- ThresholdType  
- Operator  
- ThresholdPercent  
- Enabled  
- ContactEmails  
- ContactGroups  
- ContactRoles  
- LastMonthCost  
- PrevMonthCost  
- Prev2MonthCost  
- PercentOfBudgetLastMonth  
- BudgetAccuracy  
- CurrentMonthForecastTotal  
- ForecastPercentOfBudget  
- ForecastConditionWillTrigger  
- SuggestedBudget_ActualBased  
- SuggestedBudget_ForecastBased  
- SuggestionNote  

---

## Budget Suggestion Logic

Recommended budget is calculated as:

```
ceil(max(last_month_cost, average_of_last_3_months) * 1.10 / 100) * 100
```

Includes:
- Trend adjustment
- 10 percent headroom  
- Rounding to nearest 100

---

## Example Commands

### 1. Full tenant run (all scopes)

```bash
python budget_assessment.py mg-demo --verbose
```

### 2. MG only

```bash
python budget_assessment.py mg-demo --scopes mg
```

### 3. Subscriptions only

```bash
python budget_assessment.py mg-demo --scopes sub
```

### 4. One subscription only

```bash
python budget_assessment.py mg-demo --subscription-ids <sub-id>
```

### 5. Subscription + filtered RGs

```bash
python budget_assessment.py mg-demo --subscription-ids <sub-id> --rg-names app-rg core-rg
```

### 6. Fast run (less data)

```bash
python budget_assessment.py mg-demo --months 1
```

---

## Troubleshooting

### Authentication fails
Run:

```bash
az login
```

Verify:

```bash
az account show
```

### Script slow / timeouts
Try:
- --months 1
- --subscription-ids
- Exclude RG scanning:

```bash
python budget_assessment.py mg-demo --scopes mg,sub
```

### Missing Excel sheets
A sheet is omitted if no data exists.

---

## How the Script Works

1. Authenticate using DefaultAzureCredential  
2. Enumerate MGs and subscriptions  
3. Apply filters  
4. Query budgets at each scope  
5. Query historical cost  
6. Query forecast  
7. Compute suggestions and metrics  
8. Write all results to Excel  
9. Save final workbook  

---
