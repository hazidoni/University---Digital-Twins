
# Seoul Bike Sharing – System Model Specification

**Project:** Digital Twins – Week 3  
**System:** Seoul Bike Sharing Service  
**Dataset:** Seoul Bike Sharing Demand  
**Time Step:** 1 hour

## 1. Modelling Purpose

The purpose of this project is to simulate hourly bicycle rental demand in Seoul and compare the simulated results with real observations.

The main question is:

**How closely can a simple simulation reproduce observed hourly bicycle rentals under different weather conditions?**

The model considers the hour of the day, temperature, humidity, rainfall, wind speed and service availability.

Possible what-if scenarios include:
- Increased rainfall
- Higher temperature
- Morning rush hour versus nighttime
- Service shutdown

The simulation may help understand how rental demand changes under different conditions.

## 2. System Boundary

The system represents hourly bicycle rental demand and completed rentals in Seoul.

### Inside the System

- Estimated bicycle rental demand
- Changes in rental demand
- Service operating status
- Completed bicycle rentals

### Outside the System

- Hour of the day
- Temperature
- Humidity
- Rainfall
- Wind speed

These external variables influence rental demand but are not controlled by the bike-sharing operator.

## 3. Variable Classification

| Category | Symbol | Variable |
|---|---|---|
| State | B(t) | Estimated hourly rental demand |
| Control | u(t) | Functioning Day |
| Disturbances | d(t) | Hour, Temperature, Humidity, Rainfall, Wind Speed |
| Parameters | θ | Hourly baseline and weather-effect coefficients |
| Observation | y(t) | Rented Bike Count |

### State

B(t) represents the estimated hourly bicycle rental demand.

The conceptual dynamic model assumes that the current demand level can influence demand in the next hour.

### Control

u(t) represents service availability:

- u(t) = 1: Service operating
- u(t) = 0: Service unavailable

### Disturbances

The disturbance vector is:

d(t) = [Hour, Temperature, Humidity, Rainfall, Wind Speed]

These variables affect rental demand but are external to the system.

### Parameters

The parameters describe the hourly rental baseline and the assumed effects of weather conditions.

### Observation

The measured output is `Rented Bike Count`, representing completed bicycle rentals per hour.

## 4. Dynamic System Mapping

The conceptual dynamic model defined in Week 2 is:

B(t+1) = f(B(t), u(t), d(t), θ)

The observation equation is:

y(t) = u(t) × B(t)

This distinguishes estimated demand from completed rentals.

When the service is unavailable, completed rentals are zero.

The current Week 3 implementation uses a simpler approximation and does not yet update the demand state from the previous hour.

## 5. Data and Simulation

The model uses historical observations from `SeoulBikeData.csv`.

The dataset contains:
- Date and Hour
- Temperature (°C)
- Humidity (%)
- Rainfall (mm)
- Wind speed (m/s)
- Functioning Day
- Rented Bike Count (bicycles/hour)

The data is sorted chronologically and divided into:

- 80% training period
- 20% evaluation period

The training period is used to calculate average bicycle rentals for each hour of the day, using observations from operating days.

For each evaluation hour, the simulation uses the hourly average as a baseline.

The baseline is adjusted using weather-effect coefficients:

factor = max(
    0,
    1
    - 0.015 × (Temperature - 15)
    - 0.004 × (Humidity - 60)
    - 0.06 × Rainfall
    - 0.03 × Wind Speed
)

Simulated Rentals = round(
    Functioning Day × Hourly Average × factor
)

The coefficients are illustrative assumptions carried over from the Week 2 scenario model. They have not been calibrated using the dataset.

The actual rental counts from the evaluation period are not used to generate simulated values.

## 6. Simulation and Observation Comparison

The simulation produces an estimated number of rented bicycles for each hour in the evaluation period.

These simulated values are compared with the real `Rented Bike Count` observations.

Both are represented using the same timestamps and units.

The comparison includes:
- A time-series plot of measured and simulated rentals
- Mean Absolute Error (MAE)
- A short discussion of differences

Possible differences may result from simplified assumptions, seasonal changes and omitted factors.

Parameter calibration is not required at this stage.

## 7. Digital Twin Architecture

The proposed architecture consists of five components:

Physical System
    ↓
Historical Observations
    ↓
Data Processing
    ↓
Simulation Model
    ↓
Model Outputs

The physical system is the Seoul bike-sharing service.

Historical observations are stored in the CSV dataset.

Data processing includes timestamp creation, data quality checks and preparation of model inputs.

The simulation model estimates hourly bicycle rentals using the hourly baseline, weather conditions and service availability.

The model outputs include simulated rentals and their comparison with real observations.

A future synchronization connection could update the model's estimated demand state using newly received observations.

Automatic synchronization is not implemented in Week 3.

## 8. Preparation for Week 4

**1. Which measured variable should be used for synchronization?**

Rented Bike Count (bicycles/hour).

**2. How frequently should the Digital Twin be updated?**

Hourly, matching the dataset's time resolution.

**3. How would the next time step be predicted after synchronization?**

After receiving a new observation, the model could update its estimated demand state and calculate the next hour using the updated state and expected external inputs.

Completed rentals are only a proxy for potential demand. In particular, zero rentals during service shutdown do not imply zero demand.

## 9. Limitations

The current simulation is an hourly baseline model rather than a fully dynamic Digital Twin.

It does not include automatic synchronization, parameter calibration or future weather forecasting.

These features may be introduced in later project stages.
