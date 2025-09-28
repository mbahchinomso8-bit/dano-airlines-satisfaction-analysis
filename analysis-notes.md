Analysis Notes – Airline Passenger Satisfaction
1. **Dataset**
Name in Power BI: airline_passenger_satisfaction
Size: ~120,000 rows (one row per passenger).
Columns:
Passenger info: Age, Gender, Customer Type, Type of Travel, Class.
Flight info: Flight Distance, Departure Delay, Arrival Delay.
Service ratings: Seat Comfort, Food & Drink, Inflight Wifi Service, and others (scores 1–5).
Outcome: Satisfaction (Satisfied or Neutral or Dissatisfied).
2. **Cleaning steps (done in Power Query)**
Removed rows that were completely empty (kept partial blanks).
Trimmed spaces in text fields (e.g. " Male " → "Male").
Fixed casing for categories (e.g. "LOYAL CUSTOMER" → "Loyal Customer").
Standardized Satisfaction values to two options: "Satisfied" and "Neutral or Dissatisfied".
Changed data types:
Age, Flight Distance, Departure Delay, Arrival Delay → Whole Number.
Service ratings → Whole Number (1–5).
Text fields → Text.
3. **Core DAX Measures**
**Passenger counts**
Total Passengers = COUNTROWS('airline_passenger_satisfaction')
Satisfied Passengers =CALCULATE([Total Passengers],'airline_passenger_satisfaction'[Satisfaction] = "Satisfied")
% Satisfied = DIVIDE([Satisfied Passengers], [Total Passengers], 0)
**Operational KPIs**
Avg Flight Distance = AVERAGE('airline_passenger_satisfaction'[Flight Distance])
Avg Departure Delay = AVERAGE('airline_passenger_satisfaction'[Departure Delay])
Avg Arrival Delay = AVERAGE('airline_passenger_satisfaction'[Arrival Delay])
On-time Rate % = VAR OnTime = CALCULATE(COUNTROWS('airline_passenger_satisfaction'),'airline_passenger_satisfaction'[Arrival Delay] <= 0) RETURN DIVIDE(OnTime, [Total Passengers], 0)
Delay >15 min Rate % = VAR Late15 = CALCULATE(COUNTROWS('airline_passenger_satisfaction'), 'airline_passenger_satisfaction'[Arrival Delay] > 15)
RETURN DIVIDE(Late15, [Total Passengers], 0) Service benchmarks
Avg Seat Comfort = AVERAGE('airline_passenger_satisfaction'[Seat Comfort])
Avg Food And Drink = AVERAGE('airline_passenger_satisfaction'[Food And Drink])
Avg Wifi = AVERAGE('airline_passenger_satisfaction'[Inflight Wifi Service])

4. **Buckets (calculated columns)**
Age Groups
Age Group = SWITCH(TRUE(), ISBLANK('airline_passenger_satisfaction'[Age]) || 'airline_passenger_satisfaction'[Age] < 0, "Unknown", 'airline_passenger_satisfaction'[Age] <= 20, "0–20",
'airline_passenger_satisfaction'[Age] <= 30, "21–30",
'airline_passenger_satisfaction'[Age] <= 40, "31–40",
'airline_passenger_satisfaction'[Age] <= 50, "41–50",
'airline_passenger_satisfaction'[Age] <= 60, "51–60",
'airline_passenger_satisfaction'[Age] <= 70, "61–70",
'airline_passenger_satisfaction'[Age] <= 80, "71–80","81+")
Age Group Sort = VAR A = 'airline_passenger_satisfaction'[Age]
RETURN
SWITCH(
    TRUE(),
    ISBLANK(A) || A < 0, 0,
    A <= 20, 1,
    A <= 30, 2,
    A <= 40, 3,
    A <= 50, 4,
    A <= 60, 5,
    A <= 70, 6,
    A <= 80, 7, 8)
   
Arrival Delay Buckets
Arrival Delay Bucket = VAR d = 'airline_passenger_satisfaction'[Arrival Delay]
RETURN
SWITCH(
    TRUE(),
    ISBLANK(d), "Unknown",
    d <= 0, "On time or early",
    d <= 15, "0–15 min late",
    d <= 60, "16–60 min late",
    d <= 180, "1–3 hours late", "3+ hours late")

Arrival Delay Bucket Sort =
VAR d = 'airline_passenger_satisfaction'[Arrival Delay]
RETURN
SWITCH(
    TRUE(),
    ISBLANK(d), 0,
    d <= 0, 1,
    d <= 15, 2,
    d <= 60, 3,
    d <= 180, 4,5)

Departure Delay Buckets
Departure Delay Bucket = VAR d = 'airline_passenger_satisfaction'[Departure Delay]
RETURN
SWITCH(
    TRUE(),
    ISBLANK(d), "Unknown",
    d <= 0, "On time or early",
    d <= 15, "0–15 min late",
    d <= 60, "16–60 min late",
    d <= 180, "1–3 hours late",
    "3+ hours late")

Departure Delay Bucket Sort =
VAR d = 'airline_passenger_satisfaction'[Departure Delay]
RETURN
SWITCH(
    TRUE(),
    ISBLANK(d), 0,
    d <= 0, 1,
    d <= 15, 2,
    d <= 60, 3,
    d <= 180, 4, 5)

5. **Dashboard Layout (single page)**

Top row (Cards):Total Passengers
% Satisfied
Avg Arrival Delay
On-time Rate %
Delay >15 min Rate %
Middle row (Charts – who is happy vs unhappy):
% Satisfied by Class (Eco, Eco Plus, Business)
% Satisfied by Customer Type (Loyal vs Disloyal)
% Satisfied by Type of Travel (Business vs Personal)
Bottom row (Charts – why they are unhappy):
Average ratings: Seat Comfort, Food & Drink, Inflight Wifi
% Satisfied by Arrival Delay Bucket
% Satisfied by Age Group
Slicers (side panel):Class, Customer Type, Type of Travel, Gender

**6. How to Read the Dashboard**
If delays increase, satisfaction drops, especially after 15 minutes.
Economy cabins drag the score due to seat comfort, food, and wifi.
Business class and loyal customers are steady, protect this base.
Service recovery for disloyal and delayed passengers is the fastest win.
