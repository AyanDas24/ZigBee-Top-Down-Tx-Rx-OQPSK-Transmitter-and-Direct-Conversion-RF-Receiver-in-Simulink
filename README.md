# ZigBee Top-Down Tx/Rx: OQPSK Transmitter and Direct-Conversion RF Receiver in Simulink

> Simulink models of an IEEE 802.15.4 (ZigBee) 2.45 GHz link, from baseband OQPSK transmitter to RF direct-conversion receiver and chip error rate measurement.

A Simulink project that models an IEEE 802.15.4 (ZigBee) 2.45 GHz link, from a baseband OQPSK transmitter through an RF direct-conversion receiver and back to a bit-error-rate measurement. It follows a top-down design flow: the system is first specified at the behavioral (baseband) level, then the receiver is refined into an RF circuit-envelope model so RF impairments can be evaluated against end-to-end performance.

## Contents

| File | Description |
|------|-------------|
| `TopDownTxRx.slx` | Top-level system model: transmitter, RF receiver, baseband demodulator, and BER/error measurement. |
| `TopDownRFReceiverDesignLib.slx` | Library holding the 802.15.4 transmitter subsystem and the circuit-envelope model of the direct-conversion receiver. Referenced by `TopDownTxRx.slx`. |

Both files are saved in MATLAB/Simulink **R2021b**. Keep them in the same folder so the library references resolve.

## Requirements

- MATLAB and Simulink R2021b or later
- Communications Toolbox
- DSP System Toolbox
- RF Blockset (formerly SimRF), which provides the circuit-envelope blocks
- Signal Processing Toolbox (spectrum analyzer)

## Getting Started

1. Put both `.slx` files in the same folder and add that folder to the MATLAB path (or make it the current folder).
2. Open the top-level model:
   ```matlab
   open_system('TopDownTxRx')
   ```
3. Run the simulation. The model workspace variables below are set automatically by the model's `PreLoadFcn` callback when it opens.
4. Inspect results in the display blocks (**ChER**, **ADC Output Power (dBm)**, **Display**) and the **Rx Spectrum** scope.

The library can be opened separately to browse or modify the subsystems:

```matlab
open_system('TopDownRFReceiverDesignLib')
```

## System Overview

### Signal chain (`TopDownTxRx.slx`)

```
Bernoulli Binary Generator
        -> Bits to Symbols (4 bits/symbol)
        -> Symbol to Chips (32 chips/symbol)
        -> OQPSK Modulator (baseband)
        -> Power Calibration to -100 dBm
        -> Circuit Envelope Model of Direct Conversion Receiver  (RF)
        -> Real-Imag to Complex
        -> Correct for SAW Filter Phase Rotation
        -> AGC -> Bipolar ADC
        -> OQPSK Demodulator (baseband)
        -> Find Delay / Error Rate Calculation  -> ChER display
```

Supporting blocks: **Power Meter** and **Mean** (ADC output power in dBm), **Rx Spectrum** (spectrum analyzer).

### RF receiver (`TopDownRFReceiverDesignLib.slx`)

The *Circuit Envelope Model of Direct Conversion Receiver* subsystem contains:

- Three cascaded RF amplifiers (noise figures 4 dB, 12 dB, 12 dB, 50 Ω input impedance)
- S-parameter block (front-end filtering, e.g. the SAW filter)
- IQ demodulator (noise figure 9 dB) producing separate **I Out** and **Q Out** ports
- Thermal noise source at 290 K
- RF Blockset Configuration block with carrier power normalization enabled

The 802.15.4 Transmitter subsystem contains the Bernoulli source, bit-to-symbol conversion, symbol-to-chip spreading, OQPSK baseband modulator, and a power calibration stage that sets the received signal level to -100 dBm.

## Key Parameters

Defined in the `PreLoadFcn` callback of `TopDownTxRx.slx`:

| Variable | Value | Meaning |
|----------|-------|---------|
| `bitRate` | 250e3 | Data rate (bit/s) |
| `bps` | 4 | Bits per ZigBee symbol |
| `cps` | 32 | Chips per ZigBee symbol |
| `chipRate` | `bitRate * cps / bps` = 2 Mchip/s | Spreading chip rate |
| `sps` | 4 | Samples per OQPSK symbol |
| `spf` | 4 | Samples per frame |
| `carrierFreqs` | 2.45e9 | RF carrier frequency (Hz) |
| `harmonicOrder` | 3 | Harmonics simulated in the circuit envelope |
| `oversampling` | 1 | No out-of-band signal modeled |
| `loToRFIsolation` | inf | LO-to-RF isolation (dB), ideal |
| `demodIP2` | inf | Demodulator IP2 (dBm), ideal |
| `computationDelay` | 0 | Delay compensation for BER calculation |

## Things to Try

- Change `loToRFIsolation` or `demodIP2` from `inf` to finite values to observe DC offset and second-order distortion effects on the chip error rate.
- Vary amplifier noise figures or the IQ demodulator gain/phase mismatch in the library to see their impact on ChER.
- Adjust the input power calibration (default -100 dBm) to explore receiver sensitivity.
- Increase `harmonicOrder` or `oversampling` to include more RF nonlinear and out-of-band effects (at the cost of simulation speed).
