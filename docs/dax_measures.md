# DAX Measures: Dare Careers Student Dashboard

All measures live in the `Measures (2)` table. Calculated columns and the `Weeks` table are listed at the end.

## Assumptions

- **Dropout** = a learner whose Graduation Status is `Non Graduate`. The data has no separate dropout column.
- **Completed lab** = a lab with a score above 0 (a score of 0 means nothing was submitted).
- **Attended** = the learner stayed **more than 30 minutes** in the Zoom session (`DurationMinutes > 30`), as required by the brief.
- **Cohort** = each track (Power BI, AWS Cloud) is treated as its own cohort, as the data has no cohort column.

## Learner counts and rates

```dax
Total Learners =
// Number of distinct learners in the current filter (track, status...).
DISTINCTCOUNT ( Learners[Email] )
```

```dax
Total Graduations =
// Learners whose graduation status is "Graduate".
CALCULATE ( COUNTROWS ( 'Status' ), 'Status'[Graduation Status] = "Graduate" )
```

```dax
Total Certifications =
// Learners whose certification status is "Certified".
CALCULATE ( COUNTROWS ( 'Status' ), 'Status'[Certification Status] = "Certified" )
```

```dax
Total Dropouts =
// Learners who did not graduate are counted as dropouts.
CALCULATE ( COUNTROWS ( 'Status' ), 'Status'[Graduation Status] = "Non Graduate" )
```

```dax
Graduation Rate =
// Graduates as a share of all learners. DIVIDE avoids a divide-by-zero error.
DIVIDE ( [Total Graduations], [Total Learners] )
```

```dax
Certification Rate =
// Certified learners as a share of all learners.
DIVIDE ( [Total Certifications], [Total Learners] )
```

```dax
Dropout Rate =
// Dropouts as a share of all learners.
DIVIDE ( [Total Dropouts], [Total Learners] )
```

## Attendance

```dax
Days Attended =
// Total sessions where a learner stayed more than 30 minutes.
SUM ( Attendance[Attended] )
```

```dax
Class Days =
// Number of distinct class dates in the current week/month filter.
// REMOVEFILTERS on Learners stops a single learner's missing days from lowering the count.
CALCULATE ( DISTINCTCOUNT ( Attendance[Date] ), REMOVEFILTERS ( Learners ) )
```

```dax
Attendance % =
// Days attended as a share of all possible learner-days.
DIVIDE ( [Days Attended], [Total Learners] * [Class Days] )
```

```dax
Total Hours in Class =
// All time spent in Zoom sessions, converted from minutes to hours.
SUM ( Attendance[DurationMinutes] ) / 60
```

## Participation

```dax
Days Participated =
// Number of learner-days with a participation record.
COUNTROWS ( Participation )
```

```dax
Participation Rate =
// Days participated as a share of all possible learner-days (same logic as Attendance %).
DIVIDE ( [Days Participated], [Total Learners] * [Class Days] )
```

## Assessments

```dax
Average Lab Score =
// Mean of all weekly lab scores in the current filter.
AVERAGE ( Labs[Score] )
```

```dax
Average Quiz Score =
// Mean of all weekly quiz scores in the current filter.
AVERAGE ( Quizzes[Score] )
```

```dax
Average Assessment Score =
// Mean across all lab AND quiz scores together (total points / total scores).
DIVIDE (
    SUM ( Labs[Score] ) + SUM ( Quizzes[Score] ),
    COUNTROWS ( Labs ) + COUNTROWS ( Quizzes )
)
```

```dax
Count of Labs =
// Labs completed: a lab counts only if its score is above 0.
CALCULATE ( COUNTROWS ( Labs ), Labs[Score] > 0 )
```

```dax
Average Labs Completed =
// Completed labs per learner.
DIVIDE ( [Count of Labs], [Total Learners] )
```

## Weeks table and calculated columns

```dax
Weeks =
// One row per program week (1-10).
// Week 1 starts on Monday 5 Aug 2024; each later week starts 7 days after the previous one.
ADDCOLUMNS (
    SELECTCOLUMNS ( GENERATESERIES ( 1, 10, 1 ), "Week", [Value] ),
    "Start Date", DATE ( 2024, 8, 5 ) + ( [Week] - 1 ) * 7,
    // Each week takes the month it starts in, so week 9 (30 Sep - 4 Oct) counts as September.
    "Month", FORMAT ( DATE ( 2024, 8, 5 ) + ( [Week] - 1 ) * 7, "MMMM" ),
    "Month No", MONTH ( DATE ( 2024, 8, 5 ) + ( [Week] - 1 ) * 7 )
)
```

```dax
Week (Attendance) =
// Whole weeks since Monday 5 Aug 2024, plus 1, so 5-9 Aug is week 1.
INT ( ( Attendance[Date] - DATE ( 2024, 8, 5 ) ) / 7 ) + 1
```

```dax
Week (Participation) =
// Same rule as Attendance, so both tables use the same week numbers.
INT ( ( Participation[Date] - DATE ( 2024, 8, 5 ) ) / 7 ) + 1
```

In the model both calculated columns are named `Week`.

## Relationships

| From (many) | To (one) |
|---|---|
| Attendance[Email] | Learners[Email] |
| Labs[email] | Learners[Email] |
| Quizzes[email] | Learners[Email] |
| Status[email] | Learners[Email] (one-to-one) |
| Participation[Name] | Learners[Name] |
| Attendance[Week] | Weeks[Week] |
| Participation[Week] | Weeks[Week] |
| Labs[Week] | Weeks[Week] |
| Quizzes[Week] | Weeks[Week] |