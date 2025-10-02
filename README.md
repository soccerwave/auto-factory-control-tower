\# Auto Factory – Production \& Quality Control Tower (Power BI + Power Automate)



\*\*Goal.\*\* An end-to-end industrial dashboard for an automotive plant (Martorell-like) covering Production OEE, Quality FPY \& Pareto, and Spares logistics (Fill Rate \& OTIF).  

\*\*Deliverables.\*\* Power BI report (`pbix/AutoFactory\_ControlTower.pbix`), daily PDF export (`exports/car\_factory.pdf`), and data samples (`/data`).



---



\## 1) Data Inputs



\- `production\_shifts.csv`: per-shift production KPIs (date, shift\_id, line\_id, model\_id, produced\_qty, good\_qty, scrap\_qty, downtime\_min, planned\_time\_min, ideal\_cycle\_sec).  

\- `quality\_defects.csv`: defects \& rework (defect\_date, model\_id, line\_id, defect\_code, defect\_category, supplier\_id, defect\_qty, rework\_qty).  

\- `spares\_orders.csv`: spares demand \& fulfillment (order\_id, part\_id, warehouse\_id, demand\_qty, fulfilled\_qty, promised\_date, delivered\_date).  

\*\*Date format:\*\* `YYYY-MM-DD`.



\## 2) Data Model



Star-schema with shared dimensions:

\- \*\*Fact tables:\*\* `Production\_Shifts`, `Quality\_Defects`, `Spares\_Orders`.

\- \*\*Dimensions:\*\* `Dim\_Model (model\_id)`, `Dim\_Date (Date)`.  

\*\*Relationships:\*\*  

\- `Production\_Shifts\[model\_id]` → `Dim\_Model\[model\_id]` (M:1, single)  

\- `Quality\_Defects\[model\_id]` → `Dim\_Model\[model\_id]` (M:1, single)  

\- Dates from facts → `Dim\_Date\[Date]` (M:1, single)  

\_No direct relationship between `Production\_Shifts` and `Quality\_Defects` to avoid ambiguity.\_



\## 3) Core Measures (DAX)



\- `Produced Qty`, `Good Qty`, `Scrap Qty`, `Downtime Min`, `Planned Time Min`  

\- `Quality % = DIVIDE(\[Good Qty],\[Produced Qty])`  

\- `Availability % = 1 - DIVIDE(\[Downtime Min],\[Planned Time Min])`  

\- `Ideal Cycle Sec = AVERAGE(Production\_Shifts\[ideal\_cycle\_sec])`  

\- `Operating Time Sec = SUMX(Production\_Shifts, (Production\_Shifts\[planned\_time\_min] - Production\_Shifts\[downtime\_min]) \* 60)`  

\- `Performance % = DIVIDE(\[Ideal Cycle Sec]\*\[Produced Qty],\[Operating Time Sec])`  

\- `OEE % = \[Availability %]\*\[Performance %]\*\[Quality %]`  

\- `Defect Qty`, `Rework Qty`, `FPY % = DIVIDE(\[Good Qty],\[Good Qty]+\[Rework Qty])`  

\- `Demand Qty`, `Fulfilled Qty`, `Fill Rate % = DIVIDE(\[Fulfilled Qty],\[Demand Qty])`  

\- `OTIF % = DIVIDE( COUNTROWS( FILTER(Spares\_Orders, Spares\_Orders\[delivered\_date] <= Spares\_Orders\[promised\_date]) ), COUNTROWS(Spares\_Orders) )`



\## 4) Report Pages



\- \*\*Landing:\*\* KPI cards (OEE %, FPY %, Fill Rate %, OTIF %), slicers (Date, Model, Line), OEE trend.  

\- \*\*Production (OEE):\*\* OEE trend, Downtime by line, table by line\_id \& shift\_id (Produced/Good/Scrap).  

\- \*\*Quality (FPY \& Pareto):\*\* FPY %, Pareto by defect\_code (Defect Qty + cumulative %), matrix by defect\_category × supplier\_id.  

\- \*\*Spares (Fill Rate \& OTIF):\*\* KPIs (Fill Rate %, OTIF %), Demand vs Fulfilled by warehouse, orders table with `OnTime` flag.



\## 5) RLS (Row-Level Security)



\- `Production\_Role`: restrict to `Production\_Shifts\[line\_id] IN {"Line\_1","Line\_2"}`  

\- `Quality\_Role`: full `Quality\_Defects` or restrict by `model\_id` if needed  

\- `Spares\_Role`: restrict to a single `warehouse\_id` (e.g., "WH01")



\## 6) Power Automate Flows (documented design)



> Requires Power BI Service. In this repo we document the flow design:

\- \*\*Daily PDF email (08:00):\*\* Recurrence → Refresh dataset → Export Landing to PDF → Send email with attachment.  

\- \*\*Alerts:\*\* If `OEE % < 0.85` or `Fill Rate % < 0.95` (or critical part backorder), send Email/Teams message with report link.



\## 7) How to Run



1\. Open `pbix/AutoFactory\_ControlTower.pbix` in Power BI Desktop.  

2\. Update file paths to `/data/\*.csv` if prompted.  

3\. Verify relationships in Model view and refresh.  

4\. Export a summary PDF from the Landing page if needed (`exports/`).



\## 8) Screenshots



See `/screenshots` for page previews (Landing, Production, Quality, Spares).



\## 9) Why this project



\- Industrial KPIs (OEE, FPY, Fill Rate, OTIF) with reusable star schema and clean DAX.  

\- Practical RLS roles for secure sharing.  

\- Automation-ready design (Power Automate) for daily reporting and alerts.





