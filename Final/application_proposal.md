# Seoul Bike Digital Twin – Desktop Application Proposal

**Application name:** Seoul Bike Digital Twin

## 1. Main Purpose

The application helps a bike-sharing operations manager explore historical bicycle rental demand in Seoul and view model-based forecasts of future hourly demand.

## 2. Main User

Primary user: **Bike Sharing Operations Manager**. The user wants to identify busy periods, compare demand across weeks/months/seasons, and inspect expected demand for upcoming hours without writing Python code.

Secondary user: **Bicycle Rental Customer**. A person who wants to rent a bicycle in Seoul and use the application to decide when to go cycling or rent a bike, based on forecast weather, predicted citywide rental demand and historical busy periods. The app can suggest potentially less busy hours, but it cannot guarantee bicycle availability at a particular station because station-level inventory data is unavailable.

## 3. Objectives

- Forecast hourly bicycle rental demand using our existing calibrated and AI-enhanced models.
- Integrate forecast weather information through the Open-Meteo API.
- Compare the performance of different forecasting models on historical test data.
- Support operational planning through data-driven forecasts and simulations.

## 4. Specific Use Cases

### UC1 – View Annual Rental Statistics

- **Actor:** Operations Manager
- **Trigger:** Open *Historical Statistics* and select **Annual**.
- **Input:** An available year or the complete available 12-month observation period.
- **Processing:** Filter historical rows; calculate total rentals, average rentals per observed hour, and monthly rental totals. Identify the busiest month.
- **Output:** Summary numbers and a bar chart with 12 monthly values (or only months actually available in the selected calendar year).
- **Acceptance criteria:** The chart shows no months without observations as if they contained zero rentals; the UI shows the precise date range and number of covered hours.
- **Implementation:** `pandas` date filtering and `groupby` on year/month; Matplotlib chart.

### UC2 – View Monthly Rental Statistics

- **Actor:** Operations Manager
- **Trigger:** Select **Monthly** in *Historical Statistics*.
- **Input:** One available month (year + month).
- **Processing:** Calculate total rentals per day, overall monthly total, average hourly rentals and the highest-demand day.
- **Output:** Daily demand line/bar chart, monthly summary and busiest day.
- **Acceptance criteria:** Only observed days from the selected month are included; the date labels are readable.
- **Implementation:** Filter by year/month; `groupby` on calendar date.

### UC3 – View Weekly Rental Statistics

- **Actor:** Operations Manager
- **Trigger:** Select **Weekly** in *Historical Statistics*.
- **Input:** A date or an available ISO week.
- **Processing:** Find the Monday–Sunday week containing the date; calculate daily rental totals and average hourly rentals for covered days; mark the busiest day.
- **Output:** Seven-day bar chart where dates with missing observations are shown as missing, not zero.
- **Acceptance criteria:** The selected date range is visible and incomplete boundary weeks are clearly marked.
- **Implementation:** Compute week start/end; filter timestamps; group by date.

### UC4 – Compare Seasonal Rental Statistics

- **Actor:** Operations Manager
- **Trigger:** Open **Seasonal Comparison**.
- **Input:** The available historical period; optionally select one season.
- **Processing:** Group rows by the original `Seasons` field (Winter, Spring, Summer, Autumn); calculate total rentals and **average rentals per observed hour** for each season.
- **Output:** Four-category bar chart and a table identifying the highest-demand season.
- **Acceptance criteria:** The user can compare seasons even if observation counts differ; averages and sample counts are shown alongside totals.
- **Implementation:** `groupby('Seasons')` with `sum`, `mean` and `count`.

### UC5 – Forecast Future Hourly Rental Demand

- **Actor:** Operations Manager
- **Trigger:** Open **Future Forecast**, choose 24, 48 or 72 hours, and click **Generate Forecast**.
- **Input:** Forecast horizon, Open-Meteo forecast weather, saved model parameters, and a clearly stated initialization for lagged rental inputs where necessary.
- **Processing:** Retrieve forecast temperature/humidity/precipitation/cloud cover, align to `Asia/Seoul`, then run the selected existing demand model. For multistep lag-based forecasting, feed previous **predictions** into later steps; never feed unobserved future rentals.
- **Output:** Hourly demand line chart and tabular forecasts with dates and times.
- **Acceptance criteria:** The chart is labeled **unvalidated future estimate**; an API failure produces a visible error, not invented measurements; no future observed rentals are accessed.
- **Implementation:** Existing Week 5/7 forecasting functions + Open-Meteo Forecast API + Matplotlib.

### UC6 – Compare Calibrated and AI-Enhanced Models

- **Actor:** Data Analyst or Operations Manager
- **Trigger:** Open **Model Comparison**, select a historical held-out interval and choose models.
- **Input:** Historical observed rentals and predictions of Week 5 calibrated / Week 7 direct AI / Week 7 hybrid AI. Optionally include Week 6 context matching as a benchmark.
- **Processing:** Plot predictions for the **same timestamps**; calculate MAE and RMSE only where observed rentals exist. For methods with missing predictions, report coverage and evaluate their common predicted hours separately.
- **Output:** Observed-versus-predicted chart, MAE/RMSE table, and coverage information.
- **Acceptance criteria:** The hybrid model is compared directly with the *same baseline* it corrects; training never uses rows from the held-out evaluation period.
- **Implementation:** Reuse existing notebook outputs/model functions and `sklearn.metrics` or NumPy metrics.

### UC7 – Export Selected Results:

- **Actor:** Operations Manager or Data Analyst
- **Trigger:** Click **Export CSV** on a statistics, forecast or comparison screen.
- **Input:** Currently displayed table.
- **Processing:** Save the current result as CSV through a file-save dialog.
- **Output:** A CSV file with timestamps, values, units and model labels as appropriate.
- **Acceptance criteria:** The exported rows match the visible filters and period.
- **Implementation:** `DataFrame.to_csv()` and PySide6 file dialog.

### UC8 – Adjust Weather Effects (What-if Simulation)

**Primary Actor:** Bike Sharing Operations Manager

**Purpose:** Allow the user to manually modify weather conditions and observe how these changes affect predicted bicycle rental demand.

**Preconditions:**
- Default weather values are available.

**Main Flow:**
1. The user opens the **Weather Simulation** section.
2. The application displays the current or selected weather conditions.
3. The user adjusts one or more weather parameters:
   - Temperature (°C)
   - Humidity (%)
   - Rainfall (mm)
   - Wind speed (m/s)
   - Cloud cover (%)
4. The user clicks **Run Simulation**.
5. The application uses the modified weather values as inputs to the selected forecasting model.
6. The application displays the updated predicted bicycle rental demand.
7. The user compares the original prediction with the modified prediction on a graph.

**Expected Output:**
- Original predicted rental demand.
- Predicted rental demand under modified weather conditions.
- Difference between the two predictions, displayed as a number and percentage.
- Comparison chart.

**Objective:** Help the operations manager understand how different weather scenarios might influence bicycle rental demand and support operational planning.

**Priority:** Medium

**Complexity:** Easy–Medium