# Seoul Bike Sharing – System Model Specification (Week 3)

**System:** Seoul bike-sharing service  
**Data:** `SeoulBikeData.csv`  
**Time step:** 1 hour

## 1. Purpose and system boundary

We compare simulated hourly bicycle rentals with real historical observations. The system includes estimated rental demand, service availability and completed rentals. The external inputs are hour of day, temperature (°C), humidity (%), rainfall (mm) and wind speed (m/s). The observed output is `Rented Bike Count` (bicycles/hour).

## 2. Model variables

| Category | Symbol | Variable |
|---|---|---|
| State (conceptual) | B(t) | Estimated hourly rental demand |
| Control | u(t) | `Functioning Day`: 1 if Yes, 0 if No |
| Disturbances | d(t) | Hour, temperature, humidity, rainfall, wind |
| Parameters | θ | Month-hour averages and training-derived weather effects |
| Observation | y(t) | Completed hourly rentals (`Rented Bike Count`) |

The **Week 2 conceptual dynamic model** is `B(t+1) = f(B(t), u(t), d(t), θ)` with `y(t) = u(t) × B(t)`. The current Week 3 implementation is a **simpler static approximation**: it does not propagate the previous demand state.

## 3. Real data and simulation

The notebook loads historical observations, combines `Date` and `Hour` into timestamps, checks missing values and duplicate timestamps, and plots measured rentals. It keeps observations from non-functioning days.

Within each calendar month, the first approximately 80% of **whole days** are training data; the rest are test data. This ensures that the same months appear in both sets. It tests held-out days **within known months**, not forecasting completely new months.

From **training data only**, the model calculates:

1. Average rentals for each month and hour, using operating days.
2. Average temperature, humidity, rainfall and wind speed for each month.
3. Simple weather multipliers: for each variable, compare observations below and at/above their monthly weather mean. Divide rentals by their month-hour baseline first, then average these relative rentals in each weather category.

For a test row, simulated rentals are its **month-hour baseline multiplied by the four weather-category factors**. If the service is unavailable, simulated completed rentals are zero. **Test rental counts are never inputs to the simulation.**
These multipliers are descriptive associations rather than proven causal effects. Weather factors may overlap, and the model is deliberately simple.

## 4. Simulation versus observation

The notebook plots measured and simulated rentals together for the same test-week timestamps and units (bicycles/hour). It also calculates mean absolute error over all test observations. Differences can result from omitted holidays, within-month changes, correlated weather variables and the use of averages. No automatic synchronization is implemented.

## 5. Digital Twin architecture

**Physical System → Historical Observations → Data Processing → Simulation Model → Model Outputs**

- **Physical system:** Seoul bike-sharing service.
- **Historical observations:** hourly time, weather, operating status and completed rentals from CSV.
- **Data processing:** timestamp creation, data checks, monthly train/test split and averages.
- **Simulation model:** month-hour baseline with training-derived weather adjustments.
- **Outputs:** simulated rentals, comparison plot and MAE.

**Proposed synchronization:** new hourly rental observations could update the estimated demand state in a future dynamic model. This connection is shown in the architecture diagram but is **not implemented**.

## 6. Preparation for Week 4

1. **Measured variable:** `Rented Bike Count` (bicycles/hour).
2. **Update frequency:** Hourly.
3. **Next-step prediction:** In a future dynamic model, use the newest observation to update estimated demand and calculate the next hour using updated state and expected weather inputs.

**Limitation:** Completed rentals are not identical to latent demand. During a service shutdown, observed rentals are zero even if potential demand exists.
