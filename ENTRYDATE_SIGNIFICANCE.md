# Significance of entryDate in Heatmap & Work Statistics

## Overview
`entryDate` represents the **actual date when work was performed** within a report. While `reportDate` marks when the report was submitted, `entryDate` timestamps individual daily work entries, making it the foundation for **all heatmap visualizations and work hour aggregations**.

---

## 1. **Data Hierarchy**

### Relationships
```
WorkReport (submitted on reportDate)
    ├── reportDate: "2025-10-10" (when report was submitted)
    ├── reportID: "WR-1001"
    └── dateEntries (daily work logs)
         ├── entryDate: "2025-10-05" (actual work date)
         ├── workHours: 8.5
         ├── travelMins: 30
         └── workNote: "Installation work"
```

### Time Entry Table Structure
```python
{
    "entryID": "TE-9001",
    "reportID": "WR-1001",
    "entryDate": "2025-10-05",      # ← KEY FIELD for heatmap
    "startWork": "08:00",
    "endWork": "14:00",
    "travelMins": 30,
    "workHours": 8.5,
    "totalHours": 9.0,
    "workNote": "Installation"
}
```

**Key Difference:**
- `reportDate`: When was the report submitted? (Meta-data)
- `entryDate`: When did the work actually happen? (Data-driven)

---

## 2. **How entryDate Powers Heatmap Generation**

### A. Work Type Heatmap (`/management/stats/work`)

#### Backend Flow
```python
# 1. Scan all time entries
raw_entries = scan_table_items(time_entry_table)

# 2. For each entry, extract the actual work date
for raw_entry in raw_entries:
    entry = dynamo_to_python(raw_entry)
    entry_date = parse_entry_date(entry.get('entryDate'))  # Parse YYYY-MM-DD
    
    # Skip invalid or future dates
    if entry_date is None or entry_date > now:
        continue
    
    # 3. Determine time bucket (daily/weekly/monthly/quarterly)
    bucket = bucket_for_date(entry_date, aggregation='monthly')
    # Result: "2025-10" for monthly aggregation
    
    # 4. Extract work hours
    work_hours = safe_float(entry.get('workHours'))
    travel_hours = safe_float(entry.get('travelMins')) / 60.0
    
    # 5. Travel adjustment: only count excess over 2 hours as work
    if travel_hours > 2:
        work_hours += (travel_hours - 2)
    
    # 6. Add to cluster → worktype → bucket aggregation
    aggregates[cluster_id][work_type][bucket] += work_hours
```

#### Output Structure
```python
{
    "aggregation": "monthly",
    "buckets": ["2025-01", "2025-02", ..., "2025-12"],
    "clusters": [
        {
            "clusterId": "C1",
            "clusterName": "Shanghai",
            "series": [
                {
                    "workType": "Installation",
                    "series": [
                        {"bucket": "2025-01", "totalHours": 45.5},
                        {"bucket": "2025-02", "totalHours": 52.3},
                        ...
                    ],
                    "totalHours": 547.2
                },
                ...
            ],
            "imageData": "data:image/png;base64,..."  # Rendered heatmap
        }
    ],
    "maxHours": 72.5
}
```

---

### B. Time Bucketing Logic

#### Parse Entry Date
```python
def parse_entry_date(value: Optional[str]) -> Optional[datetime]:
    if not value:
        return None
    try:
        return datetime.strptime(value, "%Y-%m-%d")  # Expects YYYY-MM-DD
    except ValueError:
        try:
            return datetime.fromisoformat(value)  # ISO format fallback
        except ValueError:
            return None
```

#### Bucket Creation by Aggregation Type
```python
def bucket_for_date(moment: datetime, aggregation: str) -> str:
    if aggregation == "monthly":
        return f"{moment.year:04d}-{moment.month:02d}"      # "2025-10"
    
    if aggregation == "quarterly":
        quarter = (moment.month - 1) // 3 + 1
        return f"{moment.year:04d}-Q{quarter}"              # "2025-Q4"
    
    # Weekly returns ISO week format
    iso_week = moment.isocalendar()
    return f"{iso_week[0]}-W{iso_week[1]:02d}"              # "2025-W40"
```

---

## 3. **entryDate in KPI Statistics Reports**

### File: `get_kpi_stats.py`

#### Scan Projection
```python
# Only fetch necessary fields for efficiency
scan_kwargs = {
    "ProjectionExpression": "reportID, entryDate, travelMins, workHours",
}
response = time_entry_table.scan(**scan_kwargs)
```

#### Entry Filtering by Date Range
```python
# Filter entries by date scope (e.g., last 3 months)
for entry in report_entries:
    try:
        entry_dt = datetime.strptime(entry["entryDate"], "%Y-%m-%d")
    except Exception:
        continue
    
    # Only include entries within the requested date range
    if not (start_date.date() <= entry_dt.date() <= end_date.date()):
        continue
    
    # Filter by month if specified
    month_abbrev = entry_dt.strftime("%b").upper()
    if months_whitelist and month_abbrev not in months_whitelist:
        continue
    
    # Entry is valid - include in aggregation
    scoped_entry_found = True
    break
```

#### Week-Based Aggregation
```python
for entry in report_entries:
    try:
        # Extract year, month name, and ISO week number from entryDate
        year, month_name, week_num = get_week_info(entry["entryDate"])
        entry_dt = datetime.strptime(entry["entryDate"], "%Y-%m-%d")
    except Exception:
        continue
    
    # Create week key: (2025, 'October', 40)
    week_key = (year, month_name, week_num)
    
    # Extract hours for this entry
    travel_hours = safe_float(entry.get("travelMins")) / 60.0
    work_hours = float(entry.get("workHours", 0) or 0)
    
    # Add to week aggregation
    week_aggregation[week_key]["workHours"] += work_hours
    week_aggregation[week_key]["travelHours"] += travel_hours
    week_aggregation[week_key]["activity_hours"][activity_type] += work_hours
```

---

## 4. **entryDate Data Storage**

### Creation (when report is submitted)
```python
# From main.py lines 2125-2145
for entry in report_data.dateEntries:
    entry_id = str(uuid.uuid4())
    utc_tz = pytz.utc
    
    # Parse the date provided by frontend
    entry_date_parsed = parser.parse(entry.date).astimezone(utc_tz)
    
    # Store in YYYY-MM-DD format
    time_entry_table.put_item(Item={
        "entryID": entry_id,
        "reportID": report_id,
        "entryDate": entry_date_parsed.strftime("%Y-%m-%d"),  # Normalized format
        "startWork": parser.parse(entry.startWork).strftime("%H:%M"),
        "endWork": parser.parse(entry.endWork).strftime("%H:%M"),
        "travelMins": entry.travelHours,
        "workHours": entry.workHours,
        "totalHours": entry.totalHours,
        "workNote": entry.workNote
    })
```

---

## 5. **Key Characteristics**

| Aspect | Details |
|--------|---------|
| **Format** | ISO date string: `YYYY-MM-DD` (e.g., "2025-10-05") |
| **Table** | `time_entry_table` (not work_report_table) |
| **Multiplicity** | Multiple entries per report (one per work day) |
| **Parent** | Linked to report via `reportID` |
| **Range** | Should be ≤ `reportDate` (work occurs before submission) |
| **Usage** | Time-series aggregation, filtering, bucketing |
| **Validation** | Must be parseable as date; can't be in future |

---

## 6. **entryDate vs reportDate**

| Feature | entryDate | reportDate |
|---------|-----------|-----------|
| **Meaning** | Actual date work was performed | When report was submitted |
| **Table** | TimeEntry table | WorkReport table |
| **Multiplicity** | Multiple per report | One per report |
| **Used For** | Heatmap visualization | User activity tracking |
| **Granularity** | Daily | Report-level |
| **Aggregation** | By day/week/month/quarter | By month (for trends) |
| **Example** | "2025-10-05" (work date) | "2025-10-10" (submission date) |

### Real Example
```
Work was performed:  entryDate = "2025-10-05"  (Oct 5)
Report submitted:    reportDate = "2025-10-10" (Oct 10)
                     Gap of 5 days = report preparation time
```

---

## 7. **Frontend Integration (WorkReport.js)**

### Entry Date Display
```javascript
const entryDate = entry?.date ? new Date(entry.date) : null;
const formattedDate = entryDate && !Number.isNaN(entryDate.valueOf())
    ? entryDate.toLocaleDateString()
    : 'No date';
```

### Storage in Draft
```javascript
const entryDate = date ? new Date(date) : new Date();
// Stored in work draft context with startWork, endWork, workHours
```

---

## 8. **Significance Summary**

### Why entryDate Matters
1. **Heatmap Foundation**: Without `entryDate`, we can't create work hour visualizations
2. **Time-Series Analysis**: Enables tracking work patterns over days/weeks/months
3. **Historical Accuracy**: Records when work actually happened, not when it was reported
4. **KPI Calculations**: Aggregates hours by time periods for performance metrics
5. **Trend Visualization**: Powers the monthly/quarterly heat map display
6. **Date Range Filtering**: Allows scoping data to specific date windows
7. **Work Type Distribution**: Enables analysis of work types by time period

### Heatmap Creation Flow
```
entryDate values → Parse & validate → Group by bucket (month/week/quarter)
      ↓
Extract workHours + travelMins → Adjust > 2hr travel rule
      ↓
Aggregate by (cluster, workType, bucket)
      ↓
Build matrix (workTypes × time buckets)
      ↓
Render heatmap image with color intensity = hours worked
      ↓
Return to frontend for Stats tab display
```

---

## 9. **Data Quality Impact**

### Missing entryDate
- Entry won't be included in heatmap
- Work hours won't be aggregated
- Reports appear "empty" in statistics

### Invalid entryDate Format
- `parse_entry_date()` returns None
- Entry is skipped in aggregation
- Partial data loss in visualizations

### Future entryDate
- Entries with `entry_date > now` are excluded
- Prevents incorrect forward-dated entries

---

## Summary
`entryDate` is the **temporal axis** of all heatmap and work statistics generation. It transforms raw work hour data into meaningful time-series visualizations, showing **when work actually happened** rather than when it was reported. Without accurate `entryDate` values stored in YYYY-MM-DD format, the entire analytics layer (aggregation, bucketing, visualization) fails to produce reliable insights.
