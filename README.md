# TRS-398 Calculator — PTW 30013 Farmer Chamber (6 MV & 18 MV)

A Python tool that computes the theoretical ionization current and absorbed dose to water for a PTW 30013 Farmer-type ionization chamber, using cavity theory and explicit mass energy-absorption coefficients, for 6 MV and 18 MV photon beams following the IAEA **TRS-398** protocol.

## Overview

The calculator combines:

- **Bragg-Gray cavity theory** to convert a photon fluence spectrum into a theoretical ionization current
- **Explicit mass energy-absorption coefficients** (μ_en/ρ) for air components (N₂, O₂, Ar) and water, interpolated from NIST XCOM data using log-log interpolation
- **TRS-398 correction factors** (k_TP, k_elec, k_pol, k_s, k_Q) to convert a raw chamber reading into absorbed dose to water

Core physics formula:

```
I_theo = (V_cav × ρ_air × e / W_air) × ∫ Φ(E) × E × (μ_en/ρ)_air(E) dE × k_corr
```

### Beam quality characteristics

| Beam | TPR₂₀,₁₀ | Mean energy | Peak fluence energy |
|------|----------|--------------|----------------------|
| 6 MV | ≈ 0.670–0.685 | ≈ 2.0 MeV | ≈ 0.5 MeV |
| 18 MV | ≈ 0.770–0.790 | ≈ 4.5 MeV | ≈ 1.5 MeV |

## Features

- Analytical photon fluence spectrum generators for 6 MV and 18 MV (based on published spectral shapes)
- Full step-by-step TRS-398 calculation workflow: fluence integral → theoretical current → raw reading → corrected reading → k_Q → absorbed dose → clinical output (cGy/MU)
- Air density correction from measured temperature and pressure
- Uncertainty budget estimation
- Side-by-side comparison mode for 6 MV vs 18 MV results

## Requirements

- Python 3.8+
- [NumPy](https://numpy.org/)

```bash
pip install numpy
```

## Usage

Run the script directly to execute both individual beam calculations and a side-by-side comparison:

```bash
python py_code000.py
```

### Using it as a module

```python
from py_code000 import TRS398_6MV_18MV, generate_6MV_spectrum

calc = TRS398_6MV_18MV()
spectrum = generate_6MV_spectrum(dose_rate_Gy_min=3)

results = calc.run_beam_calculation(
    beam_quality="6MV",
    spectrum=spectrum,
    T=20.7,             # measured temperature, °C
    P=89.96,             # measured pressure, kPa
    k_elec=1.000,
    k_pol=1.0005,
    k_s=1.0018,
    N_Dw_Q0=5.362e7,     # chamber calibration factor at Co-60, Gy/C
    t_irr=40,            # irradiation time, s
    n_MU=200,            # monitor units delivered
    dose_rate_target_Gy_min=3,
    output_units="cGy_per_MU"
)

print(results["cGy_per_MU"])
```

### Comparison mode

```python
from py_code000 import run_comparison

run_comparison(T=22.5, P=100.8, n_MU=200)
```

## Chamber specifications

Defaults are set for the **PTW 30013 Farmer chamber**:

- Nominal volume: 0.6 cm³
- Cavity radius: 3.05 mm, cavity length: 23.0 mm
- Wall: 0.335 mm PMMA + 0.09 mm graphite
- Central electrode: aluminum, 1.15 mm diameter

These can be overridden by passing a custom `ChamberSpecs` instance to `TRS398_6MV_18MV(chamber=...)`.

## Notes on accuracy

The bundled 6 MV and 18 MV fluence spectra are **analytical approximations** intended for demonstration and educational use. For research-grade or clinical accuracy:

- Replace the analytical spectra with Monte Carlo phase-space data (e.g., the [IAEA Phase-Space Database](https://www-nds.iaea.org/phsp))
- Score spectra at the chamber's reference point (SSD + depth in water)
- Apply field-size corrections for fields other than 10×10 cm²
- Add PDD/TMR/OAR factors for patient-specific QA

## References

- IAEA TRS-398 (2000): *Absorbed Dose Determination in External Beam Radiotherapy*
- [NIST XCOM](https://physics.nist.gov/PhysRefData/Xcom) — mass energy-absorption coefficients
- [PTW 30013 Farmer Chamber specifications](https://www.ptwdosimetry.com/fileadmin/user_upload/PTW/30013_en.pdf)
- [IAEA Phase-Space Database](https://www-nds.iaea.org/phsp)
- Sheikh-Bagheri & Rogers, *Med. Phys.* 29(3), 2002 — reference beam spectra

## Disclaimer

This tool is provided for educational and research purposes. It is **not** validated for clinical dosimetry decisions. Any clinical use must follow institutional QA procedures and be independently verified against measured chamber calibration data.

## License

This project is licensed under the [MIT License](LICENSE).
