# Minnesota Case Study

This folder contains the code for the Minnesota case study presented in:
> **"Impact of Integrating Use Stage into Network Level Life Cycle Planning"**

## Overview

This notebook evaluates agency and user costs for Minnesota's state highway
network over a 10-year analysis period (2025–2034).

**Agency costs** are derived from MnDOT's Capital Highway Investment Plan
(CHIP) schedule, which includes planned maintenance and rehabilitation
treatments and their associated expenditures.

**User costs** are quantified as excess fuel consumption (EFC) attributable
to pavement roughness. Fuel consumption is estimated using the Roughness Speed
Impact (RSI) model (Ziyadi et al., 2018), which computes energy consumed per
vehicle mile traveled as a function of vehicle speed and pavement roughness
(IRI). IRI values are derived from MnDOT's projected Riding Quality Index
(RQI) values. Four vehicle classes are considered: passenger vehicles, small
trucks, medium trucks, and large trucks, with fuel prices based on 2025.

## Contents

| File | Description |
|------|-------------|
| `minnesota_case_study.ipynb` | Main analysis notebook |

## Data Availability

The MnDOT dataset was provided for research purposes only and cannot be
publicly shared.
