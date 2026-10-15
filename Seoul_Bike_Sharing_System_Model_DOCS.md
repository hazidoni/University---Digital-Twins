
# Seoul Bike Sharing – System Modelling

## 1. Modelling Purpose

The purpose of this project is to model hourly bicycle rental demand in Seoul and understand how it changes under different weather conditions and throughout the day.

**Main question:**

How do the hour of the day, temperature, humidity, rainfall, and wind speed affect bicycle rental demand?

**What-if question:**

What happens to the number of bicycle rentals when weather conditions change or the bike-sharing service is unavailable?

**Potential decision:**

The model could help bike-sharing operators understand rental demand and decide when the service should be available.

## 2. System Boundary

The system represents the hourly bicycle rental demand of the Seoul bike-sharing service.

### Inside the System

- Hourly bicycle rental demand
- Changes in rental demand over time
- Bike-sharing service operating status

### Outside the System

- Hour of the day
- Temperature
- Humidity
- Rainfall
- Wind speed

These external factors influence the system but are not controlled by the bike-sharing operator.

### System Boundary Diagram

```text
             EXTERNAL INPUTS
        +-----------------------+
        | Hour                  |
        | Temperature           |
        | Humidity              |
        | Rainfall              |
        | Wind Speed            |
        +-----------------------+
                    |
                    v
    +--------------------------------+
    |    SEOUL BIKE SHARING SYSTEM   |
    |                                |
    |  Current Rental Demand         |
    |  Service Status (Control)      |
    |  Demand Update                 |
    +--------------------------------+
                    |
                    v
        +-----------------------+
        | OUTPUT                |
        | Hourly Bike Rentals   |
        +-----------------------+
```

## 3. Variable Classification

The model variables are classified into five categories.

| Symbol | Category | Variable | Description |
|---|---|---|---|
| x | State | Current hourly rental demand | Represents the system's current demand level |
| u | Control | Functioning Day | Indicates whether the bike-sharing service is operating |
| d | Disturbance | Hour, Temperature, Humidity, Rainfall, Wind Speed | External factors affecting rental demand |
| θ | Parameters | Hourly demand and weather-effect coefficients | Describe how external factors influence demand |
| y | Observation | Rented Bike Count | Actual number of bicycles rented per hour |

### State (x)

The state variable represents the current hourly bicycle rental demand.

The model uses the current demand to calculate the demand for the next hour.

### Control (u)

The control variable represents the operating status of the bike-sharing service.

It is based on the `Functioning Day` column in the dataset.

- **u = 1:** The bike-sharing service is operating.
- **u = 0:** The bike-sharing service is not operating.

In the model, the operator is assumed to be able to change the service status.

### Disturbances (d)

Disturbances are external variables that influence bicycle rental demand.

The disturbance vector is:

**d = [Hour, Temperature, Humidity, Rainfall, Wind Speed]**

The hour of the day is included because rental demand may differ significantly between morning, afternoon, and nighttime.

### Parameters (θ)

Parameters describe how strongly the different external factors affect rental demand.

The parameter vector is:

**θ = [α, β, γ, δ, ε]**

These parameters represent the effects of the hour of the day, temperature, humidity, rainfall, and wind speed.

### Observation (y)

The observation variable is the actual hourly rented bicycle count recorded in the dataset.

It represents the measurable output of the system.

## 4. State and Memory

The model treats hourly rental demand as a state variable because the current demand level is assumed to influence demand in the next hour.

For example, high rental demand during the morning commute may be followed by another period of high demand.

However, rental demand also depends on the hour of the day and weather conditions.

Therefore, the next state depends on both the previous state and the external inputs.

## 5. Dynamic System Mapping

The general dynamic system model is:

**x(t+1) = f(x(t), u(t), d(t), θ)**

The observation equation is:

**y(t) = g(x(t), u(t))**

### Mapping to the Seoul Bike Sharing System

**B(t+1) = f(B(t), u(t), [H(t), T(t), RH(t), R(t), W(t)], [α, β, γ, δ, ε])**

Where:

| Symbol | Meaning |
|---|---|
| B(t) | Current hourly rental demand |
| B(t+1) | Rental demand in the next hour |
| u(t) | Bike-sharing service status (0 or 1) |
| H(t) | Hour of the day |
| T(t) | Temperature |
| RH(t) | Humidity |
| R(t) | Rainfall |
| W(t) | Wind speed |
| α, β, γ, δ, ε | Model parameters |

### Observation Mapping

The model distinguishes between rental demand and completed rentals.

If the service is unavailable, no bicycle rentals can occur.

A simple observation relationship is:

**y(t) = u(t) × B(t)**

Therefore:

- If **u(t) = 0**, then **y(t) = 0**.
- If **u(t) = 1**, then **y(t) = B(t)**.

The actual `Rented Bike Count` from the dataset is the measured observation used to represent completed rentals.