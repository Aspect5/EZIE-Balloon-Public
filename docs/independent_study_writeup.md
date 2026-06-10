# Reconstructing the Balloon's Lost GPS Track

*Edited excerpt from my independent research journal for AS.171.501 at JHU (fall 2023), supervised by Professors Reid Mumford and Daniel Reich. The full journal also covers outreach and professional development work that isn't included here.*

## The problem

Before this semester I designed, built, tested, and launched an EZIE-Mag-based scientific weather balloon payload via the University of Maryland Aerospace Engineering Department's Nearspace program. Upon recovery, I realized there was a 20 mile gap in the GPS data. According to my calculations using the Haversine formula, the balloon traveled a substantial distance during this time: between points 6874 (39.8105, -78.2149) and 6875 (39.5425, -77.9305), 23.9 miles.

The goal of the independent study was to reconstruct the lost position information, compare it with UMD's GPS readings, and formalize the work into an academic paper. Over the semester the codebase grew from about 2,100 to 3,500 lines.

## Kalman filtering research

My summer code attempted to incorporate readings from the accelerometer, magnetometer, and gyroscope into a 12x12 matrix that went through a Kalman filter. That model was overly complex, so I restricted the new attempts to the accelerometer measurements plus altitude estimates derived from UMD's pressure data.

I worked through the standard references (Welch and Bishop's "An Introduction to the Kalman Filter" and Maybeck's "Stochastic models, estimation, and control") and built out the measurement noise covariance matrix R and process noise covariance matrix Q from the sensor datasheets: accelerometer RMS noise of 1.8 mg and noise density of 0.0018 g/sqrt(Hz) (LSM6DSOX), gyroscope RMS noise of 75 mdps, GPS positional accuracy of about 3 m CEP (PA1616D), with sensor bandwidths around 18.5 Hz, half the 37-40 Hz sampling rate per Nyquist.

## Merging and cleaning the data

The `align_data` function merges the datasets from UMD's payload and my retrofitted EZIE-Mag. The improved version marks UMD data with a `U_` prefix, scans for repeated altitude readings (which indicate lost GPS measurements) and fills the corresponding latitude and longitude with NaN values to be estimated later, and is more careful about which columns get merged.

Since noise in the acceleration readings would substantially affect double-integrated position estimates, I wrote two outlier removal passes, one using the interquartile range and one using Z-scores. These were very effective for computing altitude from UMD's pressure data, bringing it to within 8-10% maximum error.

## Position estimation, attempt 1: double integration

The first algorithm fills the gap assuming motion from the last known state: set the initial conditions to the last valid point before the gap, convert geodetic coordinates to ECEF, compute an initial ECEF velocity from the last known ground speed, heading, and ascent rate, then iterate over the gap integrating acceleration (rotated from ENU to ECEF) into velocity and position, converting back to geodetic at each step.

Getting the coordinate handling right was harder than expected. Rather than a naive conversion factor I used pyproj to do proper cartographic projections and coordinate transformations between geodetic and ECEF frames, which accounts for Earth's curvature and the balloon's altitude. I validated the transformers with round-trip conversions.

The double integration output something, but far from ideal, so I switched to UMD's ground speed measurements so the algorithm only had to integrate once.

## Position estimation, attempt 2: single integration

The second algorithm updates the movement parameters dynamically instead of holding the initial velocity constant: at each step it recomputes ground speed, ascent rate, and heading, updates the ECEF velocity, and advances the position with midpoint integration (averaging the current and previous velocity). It also added debugging checks on the heading data and velocity vectors.

This output was a lot more promising. It seemed to have the correct distance on visual inspection, but drifted off the proper axis. To quantify this I built test harnesses that copy the dataset and punch artificial gaps of 5 to 1,000 samples into it, run the estimator, and score it with RMSE against the known track, plus a validation script that bounds-checks coordinates and flags large gaps.

## What I learned

The estimates became less accurate over time, as expected, with drift causing substantial deviation in direction though not necessarily in magnitude. A question at the physics poster session made the root cause click: the accelerometer's sampling rate was too low relative to the rate of gyrations and wind perturbations on the payload, so the acceleration record undersamples the actual motion.

My advisor and I concluded the reconstructed positions weren't accurate enough to publish. Professor Brown from the math department pointed out there are better methods than naive integration for estimating future states of a dynamical system, and capping the velocity would keep the error from growing unbounded. The plan from here is to work with APL experts on proper sensor fusion.
