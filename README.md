# Project 9: Hospital Appointment Operations Analysis

**Industry:** Healthcare Operations  |  **Tools:** Python only (pandas, NumPy, matplotlib, seaborn)

## Objective
Analyse hospital appointment records to find operational bottlenecks in **attendance, waiting times,
department workload and consultation fees**. This is an *operational* dataset: the project makes **no medical
or clinical conclusions**.

## Dataset
`data/09_hospital_appointments.csv` - 500 appointments, 9 columns, 1 Jan 2025 to 29 Jun 2025.

| Column | Description |
|---|---|
| Appointment_ID | Unique appointment code (e.g. AP00000) |
| Age | Patient age in years (used only as a scheduling segment) |
| Department | Pediatrics, Dermatology, General, Cardiology, Orthopedics |
| Appointment_Date | Date of the appointment (converted with `pd.to_datetime`) |
| Wait_Time_Min | Waiting time in minutes (10 values missing) |
| Appointment_Type | New / Follow-up |
| SMS_Reminder | Yes / No |
| Attendance_Status | Attended / No-show / Cancelled |
| Consultation_Fee | Fee listed for the appointment (also present on non-attended rows) |

## How to run
```bash
# 1. (optional) create a virtual environment
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# 2. install the libraries
pip install -r requirements.txt

# 3a. run the script from the PROJECT folder (creates charts and tables in outputs/)
python src/hospital_analysis.py

# 3b. or open the notebook (it reads ../data, so keep it inside the notebooks/ folder)
jupyter notebook notebooks/hospital_appointments_analysis.ipynb
```

## Folder structure
```
hospital-appointments-analysis/
├── data/
│   └── 09_hospital_appointments.csv
├── notebooks/
│   └── hospital_appointments_analysis.ipynb   # same analysis, with outputs and charts
├── src/
│   └── hospital_analysis.py                   # beginner-friendly commented script (headings = task numbers)
├── outputs/
│   ├── chart1 ... chart7 (PNG charts)
│   ├── department_summary.csv
│   ├── recommendations.csv
│   └── cleaned_hospital_appointments.csv
├── docs/
│   └── SRS_Hospital_Appointment_Operations_Analysis.docx   # export the SRS Doc as Word/PDF and save it here
├── README.md
└── requirements.txt
```

## Data-cleaning decisions
| Step | What was found | What was done | Why |
|---|---|---|---|
| Date type | `Appointment_Date` stored as text, 0 invalid dates | Converted with `pd.to_datetime` and validated | Needed for monthly and weekday analysis |
| Missing wait times | 10 of 500 rows (2.00%) | Filled with the **median wait of the same department**; added a `Wait_Time_Imputed` flag | Median ignores extreme waits, department level is fairer, and all 500 rows are kept. Mean moved from 28.16 to 28.15 |
| Duplicate IDs | 0 duplicates | No rows removed | Each row is a unique appointment |
| Fees on non-attended rows | 69,600 listed on No-show / Cancelled rows | Excluded from the earned-fee estimate | A fee is only earned when the patient attends |

## Key findings
- **Attendance:** 79.40% attended, 12.00% no-show, 8.60% cancelled (500 appointments).
- **Waiting time:** average 28.15 min (median 27.50, IQR 18-38). Longest: Pediatrics 30.42 min and Dermatology 29.94 min; shortest: Cardiology 25.80 min.
- **Long waits:** 41.60% of appointments wait more than 30 minutes; only 2 outliers (>68 min), both in Orthopedics.
- **No-shows by department:** highest in Pediatrics (14.95%), Dermatology (14.29%) and Orthopedics (13.04%); lowest in Cardiology (8.16%).
- **SMS reminders:** no-show rate 10.53% with a reminder vs 13.44% without (2.91 points lower). With about 250 rows in each group, treat this as promising, not proven.
- **New vs Follow-up:** almost identical waits (28.05 vs 28.26 min).
- **Fees:** estimated 296,200 from attended visits (of 365,800 listed on all rows). Dermatology 64,900 is the highest, Orthopedics 48,300 the lowest.
- **Extra:** Saturday has the highest no-show rate (20.29%); the 60+ age group has the highest (16.88%) and 18-35 the lowest (7.08%).

## Recommendations (summary)
1. Send SMS reminders for every appointment (13.44% vs 10.53% no-show).
2. Allow controlled overbooking in Pediatrics and Dermatology (14.29%-14.95% no-show).
3. Add queue support in Pediatrics and Dermatology (30.42 and 29.94 min average wait; 44.86% of Pediatrics patients wait over 30 min).
4. Balance the week: Thursday is busiest (85), Monday quietest (58); Saturday no-show is 20.29%.
5. Set a target to cut waits over 30 minutes (now 41.60%) and review the 2 outlier cases.

## Limitations
Small sample (500 rows, about 100 per department; the SMS result rests on about 250 rows per group); six months of data only (June ends on 29 Jun); fees have no stated
currency; ages appear in every department from 1 to 89, so Age was not used to judge departments; results show
association, not cause.
