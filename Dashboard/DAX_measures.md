1. Dim_Calendar = CALENDAR(DATE(2026,1,1), DATE(2026,3,31))




2.Actionable Leakage = 
SUMX(
    VALUES(Dim_Magistrate[Magistrate_ID]),
    VAR Total_Issued = CALCULATE(SUM(Fact_Inspections[Fine_Amount]))
    VAR Target_Expected = Total_Issued * 'Recovery Target %'[Recovery Target % Value]
    VAR Total_Collected = CALCULATE(SUM(Fact_Inspections[Fine_Amount]), Fact_Inspections[Fine_Status] = "Paid")
    VAR Leakage = Target_Expected - Total_Collected
    RETURN
    IF(Leakage > 0, Leakage, 0)
)





3. Selected Metric = 
SWITCH(
    SELECTEDVALUE('Commodity Selector'[Commodity]),
    "Onion Price Trend", AVERAGE(Fact_Market[Onion_Price_Dev_From_Trend]),
    "Potato Price Trend", AVERAGE(Fact_Market[Potato_Price_Dev_From_Trend]),
    "Tomato Price Trend", AVERAGE(Fact_Market[Tomato_Price_Dev_From_Trend]),
    AVERAGE(Fact_Market[Onion_Price_Dev_From_Trend])
)





4. Ghost Inspections = 
CALCULATE(
    COUNTROWS('Fact_Inspections'), 
    'Fact_Inspections'[Minutes_Since_Last_Inspection] < 8
)





5. Night Owl Inspections = 
CALCULATE(
    COUNTROWS('Fact_Inspections'), 
    HOUR('Fact_Inspections'[Time]) >= 22 || HOUR('Fact_Inspections'[Time]) < 5
)





6. Total Hoarding Anomalies = SUM('Fact_Market'[City_Daily_Hoarding_Risk])




7. Anomalies Last Month = CALCULATE([Total Hoarding Anomalies], DATEADD('Dim_Calendar'[Date], -1, MONTH))




8. MoM Hoarding Growth % = DIVIDE([Total Hoarding Anomalies] - [Anomalies Last Month],
                               [Anomalies Last Month],
                               0
)                               


9. Recovery Target % = GENERATESERIES(0.5, 1, 0.01)


10. Commodity Selector = 
DATATABLE(
    "Commodity", STRING,
    {
        {"Onion Price Trend"},
        {"Potato Price Trend"},
        {"Tomato Price Trend"}
    }
)