# Hot Axle Box Detection (HABD)

Wayside infrared monitoring system for early detection of overheating axle bearings on railway rolling stock, enabling predictive maintenance and prevention of axle-bearing failure (hot box) derailments.

## Overview

Hot Axle Box Detection (HABD), also known internationally as Wheel Temperature Detection (WTD) or Hot Box Detector (HBD), is a trackside system that measures the infrared thermal signature of axle bearings and wheel treads as a train passes, flagging bearings operating above safe temperature thresholds before catastrophic bearing seizure and axle failure can occur.

This repository implements a sensor-fusion-based HABD system combining infrared pyrometry, wheel-detection/axle-counting, and train consist correlation to produce per-axle temperature reports and alarm classifications in real time.

## Problem Statement

Axle bearing overheating due to lubrication failure, bearing wear, or mechanical damage is a leading cause of freight and passenger derailments worldwide. Traditional trackside HABD installations are spaced 20–40 km apart, which:

- Allows a developing hot bearing to progress significantly between detection points
- Produces single-point temperature readings without trend/rate-of-rise data
- Struggles with false positives from ambient conditions (sun-loading, rain, snow) and false negatives from sensor misalignment or dirty optics

This system targets closer detection intervals, rate-of-rise trending across consecutive sites, and multi-sensor fusion to reduce both missed detections and nuisance alarms.

## System Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌───────────────────┐
│  Wayside Sensor  │────▶│  Edge Processing  │────▶│  Central Monitoring│
│      Head        │     │      Unit         │     │      System        │
└─────────────────┘     └──────────────────┘     └───────────────────┘
   IR pyrometers            Signal processing         Fleet-wide DB
   Wheel sensors             Axle correlation          Alarm dispatch
   Axle counter              Alarm classification       Trend analysis
```

### 1. Wayside Sensor Head

| Component | Specification |
|---|---|
| IR detector | Dual-channel HgCdTe or InGaAs pyrometer, 8–14 µm waveband |
| Temperature range | -40°C to +350°C, ±2°C accuracy (or ±2% of reading, whichever greater) |
| Sampling rate | ≥ 10 kHz per channel (to resolve bearing signature at line speeds up to 160 km/h) |
| Field of view | 3–5 mrad, mechanically aligned to bearing cup centerline ±5 mm |
| Wheel/axle sensor | Inductive proximity sensor pair, redundant, for speed and axle-position triggering |
| Enclosure rating | IP66/IP67, heated optics window (anti-condensation, -25°C rated) |
| Mounting | Trackside pedestal, gauge-corner offset per rail authority standard (e.g., 75 mm from rail head) |

### 2. Edge Processing Unit (EPU)

- Real-time signal conditioning: bandpass filtering to isolate bearing-cup thermal pulse from wheel-tread and rail-return signal
- Axle-by-axle windowing using axle counter triggers to gate the IR capture window
- Per-axle peak, mean, and rise-rate computation
- Local alarm classification against three-tier thresholds (see below)
- Store-and-forward buffering for network outage resilience
- Typical hardware: industrial PC or embedded ARM/x86 controller, deterministic RTOS or PREEMPT_RT Linux

### 3. Alarm Classification Thresholds (illustrative — tune to rail authority standard, e.g., AAR, UIC 812-3, or RDSO specifications)

| Level | Criterion | Action |
|---|---|---|
| Warm (Level 1) | ΔT vs. train average > 20°C, or absolute > 70°C | Log, no immediate action |
| Hot (Level 2) | ΔT > 40°C, or absolute > 90°C, or rise rate > 2°C/km | Alert to control center, advisory speed restriction |
| Critical (Level 3 / Dragging Equipment tier) | ΔT > 60°C, absolute > 110°C, or asymmetric bearing pair differential > 30°C | Immediate stop order, dispatch to nearest siding |

Differential (bearing-to-bearing on same axle, and axle-to-train-average) analysis is weighted more heavily than absolute temperature, since ambient/solar loading affects all bearings similarly but a developing fault does not.

### 4. Data Correlation & Fleet Integration

- Axle position correlated to wagon/coach ID via consist manifest (from TMS/ICMS) or axle-counting + known consist length
- Time-series storage per axle-bearing across multiple wayside sites to compute inter-site rise-rate trending
- Integration point for a broader Wagon/Rolling-Stock Monitoring platform (fleet-wide historical trending, maintenance work-order generation)

## Repository Structure

```
.
├── firmware/           # Sensor head control and signal acquisition
├── edge-processing/    # EPU signal processing, axle correlation, alarm logic
├── central-monitoring/ # Backend services, alarm dispatch, dashboards
├── docs/               # System design docs, threshold calibration references
└── tests/              # Unit/integration tests, synthetic thermal signature datasets
```

## Standards & References

- UIC 812-3 — Hot axle box detectors, technical specification
- AAR Manual of Standards and Recommended Practices, Section G-II (Wayside detector systems)
- RDSO specifications for Hot Axle Box Detection (Indian Railways context)
- EN 50126 / EN 50129 — RAMS and safety-related electronic systems for railway applications (for SIL classification of the alarm chain)

## Status

Early-stage design/prototype repository. Contributions and threshold-calibration data from field deployments are welcome.

## License

TBD
