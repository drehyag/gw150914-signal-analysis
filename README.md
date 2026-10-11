# GW150914 Signal Analysis

Recovering the first detected gravitational wave, a binary black hole merger, from public LIGO strain data in Python.

The signal is buried below the detector noise in the raw data. This notebook measures the noise, removes it, and uses a matched filter to recover the event in both LIGO detectors, then checks the result two independent ways.

## Pipeline

1. **Measure the noise.** Estimate each detector's noise spectrum with Welch's method.
2. **Whiten.** Divide each frequency by its noise level so the noise is flat.
3. **Bandpass.** Keep 43 to 300 Hz with a zero-phase filter, where the detector is quiet and the signal has power.
4. **Matched filter.** Correlate the data against a template waveform, weighted by the noise, to get SNR over time.
5. **Validate.** Check detector coincidence and compare the event to a measured noise background.

## Results

| Measurement | Value |
| --- | --- |
| Peak SNR, Hanford (H1) | 24.25 |
| Peak SNR, Livingston (L1) | 17.92 |
| Arrival time vs. published merger time | within 0.4 ms |
| H1 minus L1 arrival time | 7.32 ms (light-travel limit: 10 ms) |
| Loudest noise-only SNR in the same record | 5.67 |

The recovered SNRs run higher than LIGO's published values (about 20 for H1 and 13 for L1). Likely causes: a single fixed template with no search over parameters, a noise spectrum estimated from data that includes the event, and a different frequency band than LIGO's production pipeline.

## A bug worth noting

The first whitening attempt gave a noise variance about 6 times too high. Running the same code on synthetic white noise returned 0.998 (expected 1.0), which ruled out the whitening code. The real cause was the FFT treating the 32-second record as periodic, so the jump between the last and first samples acted like a step and added power at every frequency. Trimming 4 seconds from each end fixed it. Tapering the ends with a window would be the better fix.

## Possible improvements

- Estimate the noise spectrum from data away from the event
- Use overlapping windowed segments instead of trimming
- Add a chi-squared test to reject glitches
- Search a bank of templates and keep the best SNR

## Running it

```
pip install -r requirements.txt
jupyter notebook GW150914_signal_analysis.ipynb
```

The first cell downloads the data files (about 2 MB) into a `data/` folder.

## Data and credits

Strain data and template from the [Gravitational Wave Open Science Center](https://gwosc.org), downloaded through LIGO's [open data event tutorial](https://github.com/losc-tutorial/LOSC_Event_tutorial). The analysis follows LIGO's published method in Abbott et al., Phys. Rev. Lett. 116, 061102 (2016), and builds on that tutorial.

Author: Andreh Yaghoubian
