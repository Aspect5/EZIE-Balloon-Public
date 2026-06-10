# EZIE-Balloon

In 2023 I designed, built, and launched a scientific weather balloon payload built around an EZIE-Mag, the magnetometer kit from the outreach program for [NASA's EZIE mission](https://science.nasa.gov/mission/ezie/) at JHU APL. The launch went through the University of Maryland's Nearspace program. After recovery I found a 20 mile gap in the GPS track, and I spent an independent study at JHU trying to reconstruct the missing positions from the accelerometer and ground speed data (Kalman filtering, double and single integration, geodetic/ECEF coordinate transforms).

## What's here

- `docs/` - the SmallSat 2024 paper on EZIE-Mag (I'm a co-author), my AIAA YPSE 2024 presentation, the ISEC poster, and the notebook from a classical mechanics final project that used the EZIE-Mag
- `data/smr.60s/` - 60 second averaged magnetometer products from the 2023 flights
- `data/logs/` - instrument and GPS logs
- `eclipse-2024/` - poster, analysis notebook, and map from deploying EZIE-Mags along the April 2024 total eclipse path

The raw magnetometer captures are too large for this repo. The analysis code lives on APL's internal GitLab.
