# Procedure: How the Diabetes Risk Dashboard Was Built

This document walks through the full workflow, from raw data to finished dashboard, so anyone can reproduce or audit the work.

**Tools:** Microsoft Excel (desktop, 2016 or later)
**Input:** 15,000 patient records, 19 columns
**Output:** One workbook with three sheets: `Data`, `Pivot Table`, `Dashboard`

---

## Step 1: Load the data and create an Excel Table

1. Paste or import the dataset into a sheet and name the sheet **Data**.
2. Click any cell in the data, then press **Ctrl + T** to convert the range to a Table (`A1:S15001`, headers on).
3. Keep one header row and no merged cells, blank rows or blank columns.

**Why:** A Table auto-expands, keeps clean field names for PivotTables, and makes the source easy to refresh.

## Step 2: Check data quality

| Check | Method | Result |
|-------|--------|--------|
| Missing values | Filter each column for blanks, or `=COUNTBLANK(range)` | None found |
| Duplicate records | Data > Remove Duplicates on `Patient ID` (preview only) | None found |
| Valid ranges | Sort or filter numeric columns for min and max | Age 18 to 80, BMI up to 40.8, HbA1c 4.8 to 13.4 |
| Category spelling | Filter text columns and read the distinct values | 18 cities; consistent labels |
| Placeholder values | Review category lists | "Not Reported" appears in Smoking, Alcohol and Income and was kept as its own category |

**Decision:** "Not Reported" is real information about missing disclosure, so it was kept, not deleted or guessed.

## Step 3: Prepare the Age Group field

1. Build the first PivotTable (Step 4) and place `Age` in the Rows area.
2. Right-click any age > **Group**.
3. Set **Starting at** 18, **Ending at** 60 and **By** 14 (this gives 18-31, 32-45, 46-60), so that patients above 60 form a fourth **60+** band.
4. Rename the groups to `18-31`, `32-45`, `46-60`, `60+`.

**Why:** Individual ages are too granular to read. Four bands show the pattern at a glance and also work as a slicer.

## Step 4: Build the PivotTables

Create all PivotTables from `Table1` on the `Pivot Table` sheet, using the **same data source**, so they share one pivot cache and respond to the same slicers.

| # | PivotTable | Rows | Columns | Values | Filter |
|---|-----------|------|---------|--------|--------|
| 1 | Overall risk distribution | Diabetes Risk | none | Count of Patient ID, shown as **% of Grand Total** | none |
| 2 | Family history vs risk | Family Diabetes History | Diabetes Risk | Count of Patient ID | none |
| 3 | Clinical averages by tier | Diabetes Risk | none | Average of HbA1c, BMI, Fasting Blood Sugar | none |
| 4 | Risk by age group | Age Group | Diabetes Risk | Count of Patient ID | none |
| 5 | High-risk patients by city | City | none | Count of Patient ID | Diabetes Risk = **High**; City filtered to **Top 10** by count |
| 6 | Risk by physical activity | Diabetes Risk | Physical Activity Level | Count of Patient ID | none |
| 7 | KPI source | none | none | Count of Patient ID, Average of HbA1c, Average of BMI | none |

Formatting tips:

- Set number formats on the values (0.0 for averages, 0% for shares).
- Turn off subtotals or grand totals where they add clutter.
- Sort the city table in descending order so the chart reads top to bottom.

## Step 5: Build the KPI cards

The KPI numbers are pulled from the PivotTables with `GETPIVOTDATA`, so they update when slicers change.

Examples:

```excel
=GETPIVOTDATA("Count of Patient ID", $A$61)
=GETPIVOTDATA("Average of HBA1C Level", $A$61)
=GETPIVOTDATA("Average of BMI", $A$61)
=GETPIVOTDATA("Patient ID", $A$3, "Diabetes Risk", "High")
```

The cards show: **Total Patients, Avg BMI, Avg HbA1c, High Risk %, Moderate Risk %, Low Risk %**.

Format each as a card with a rounded rectangle, a label, an icon and a large number.

## Step 6: Create the PivotCharts

Select a PivotTable, then **PivotTable Analyze > PivotChart**.

| Chart | Type | Source |
|-------|------|--------|
| Overall Risk Distribution | Doughnut | PivotTable 1 |
| Risk by Age Group | Clustered column | PivotTable 4 |
| Top 10 Cities by High-Risk Count | Horizontal bar | PivotTable 5 |
| Risk by Physical Activity Level | Clustered column | PivotTable 6 |
| Avg HbA1c, BMI and Fasting Blood Sugar by Risk Tier | Clustered column | PivotTable 3 |

Styling rules used:

- One consistent colour per risk tier across all charts (for example red = High, amber = Moderate, green = Low)
- Clear chart titles, data labels only where they help, no gridline clutter
- Hide the field buttons on the charts for a cleaner look (right-click > Hide All Field Buttons)

Move each chart from the `Pivot Table` sheet to the `Dashboard` sheet (cut and paste).

## Step 7: Add slicers and connect them

1. Select any PivotTable > **PivotTable Analyze > Insert Slicer**.
2. Choose **Age** (which appears as *Age Group* after grouping) and **Gender**.
3. For each slicer, right-click > **Report Connections** and tick **all seven PivotTables**.
4. Set the slicer columns (4 for Age Group, 3 for Gender) and apply a dark slicer style to match the theme.

**Test:** Select "60+" in Age Group. All KPIs and charts should change together. Clear the filter with the slicer's clear icon.

## Step 8: Design the dashboard layout

1. Insert a new sheet called **Dashboard** and turn off gridlines (View > Gridlines).
2. Place a title banner: **DIABETES RISK DASHBOARD**.
3. Top row: KPI cards.
4. Middle: Overall Risk Distribution, Risk by Age Group.
5. Bottom: Top 10 Cities, Risk by Physical Activity Level, Clinical Averages by Risk Tier.
6. Right side: the two slicers and the navigation buttons.
7. Add navigation buttons (shapes with hyperlinks) for **Dashboard**, **Data** and **Summary** (the `Pivot Table` sheet).
8. Align all objects to a grid (Shape Format > Align) and keep consistent spacing.

## Step 9: Final checks before publishing

- [ ] Every chart title and axis label is readable
- [ ] Slicers change every KPI and chart
- [ ] Numbers on the dashboard match the PivotTables (for example 15,000 total; 15% / 25% / 60% split)
- [ ] Refresh all (Data > Refresh All) causes no errors
- [ ] No leftover external links (Data > Edit Links) and no broken named ranges (Formulas > Name Manager)
- [ ] File properties do not expose a personal folder path (File > Info > Inspect Document)
- [ ] Zoom level is set so the dashboard opens readable (about 60 to 80%)
- [ ] Take a screenshot of the Dashboard sheet for the README

## Step 10: Publish

**GitHub**

1. Create a repository, for example `diabetes-risk-dashboard`.
2. Upload `README.md`, `PROCEDURE.md`, the `.xlsx` file, and `images/dashboard.png`.
3. Confirm the image shows on the README page.

**LinkedIn**

- Post the dashboard screenshot with a 3 to 5 line summary of the top findings and the GitHub link.
- Add the project under **Featured** or **Projects** on your profile.

---

## Formula and method reference

| Item | Definition |
|------|-----------|
| Risk share | Count of patients in a tier ÷ total patients in the current filter |
| High-risk rate for a group | High-risk patients in the group ÷ all patients in the group |
| Age bands | 18-31, 32-45, 46-60, 60+ (over 60) |
| Top 10 cities | The 10 cities with the most High-risk patients |

## Reproducing the results

Open the workbook, go to `Pivot Table`, and compare each table with the figures in the README. All numbers come straight from the `Data` sheet, with no hidden inputs.
