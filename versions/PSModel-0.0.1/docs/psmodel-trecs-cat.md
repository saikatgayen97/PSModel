# T-RECS Catalogue Generator Guide (`psmodel-trecs-cat`)

The CLI tool [`scripts/psmodel-trecs-cat`](../scripts/psmodel-trecs-cat) simulates realistic radio source populations using **T-RECS** (Tiered Radio Extragalactic Continuum Simulation).

All source properties—including cosmological redshifts, luminosity functions, spectral energy distributions (SEDs), active galactic nuclei (AGN) vs. star-forming galaxy (SFG) populations, morphologies (major/minor axes and position angles), and polarization—are drawn directly from the physical cosmological models in the T-RECS database (`~/DATA/TRECS_Inputs`).

---

## 1. Directory & File Conventions

In strict adherence to the `PSModel` developer rules:
* **Catalogues (`.csv`)**: Saved in [`catalogue/`](../catalogue/).
* **Execution & Logs**: All runs must be executed from [`workdir/`](../workdir/), with logs saved in [`workdir/logs/`](../workdir/logs/).
* **Scripts**: Located in [`scripts/`](../scripts/).
* **Documentation**: Located in [`doc/`](../doc/).
* **Agent Lock (`agent.lock`)**: `agent.lock` must be `true` when idle and `false` during active file modifications.

---

## 2. Common Sky & Multi-Frequency Architecture

When multiple frequencies are requested (e.g. `--freq-MHz 400,610`):
1. **Single Sky Realization**: The sky is simulated **once** over the maximum bounding field of view across all requested bands. All astronomical sources are assigned persistent unique integer `id` numbers.
2. **Consistent Real-Sky Cross-Matching**: Any source visible in multiple frequency catalogues shares the **exact same `id`, coordinates (`ra`, `dec`), morphology (`bmaj`, `bmin`, `pa`), cosmological redshift, and radio class**.
3. **Per-Frequency FOV & Flux Limits**: Each frequency can have its own field of view (`--FOV-arcmin`) and minimum flux cutoff (`--min-flux-mjy`). A source outside a higher frequency's narrower beam or below its sensitivity limit is naturally excluded from that band while remaining present in the lower frequency band.
4. **Per-Frequency Fluxes**: Source flux densities are evaluated at each target frequency according to each source's intrinsic SED (and attenuated by the beam if `--pb-effect` is used).
5. **Local Spectral Index & Curvature**: For each frequency $\nu$, the script evaluates fluxes on a tight symmetric 3-point grid ($\nu_- = 0.97\nu$, $\nu_0 = \nu$, $\nu_+ = 1.03\nu$) and computes the exact local spectral parameters:
   * **Local Spectral Index**:
     $$\alpha(\nu) = \left.\frac{d \ln S}{d \ln \nu}\right|_\nu$$
   * **Local Spectral Curvature**:
     $$q(\nu) = \left.\frac{d^2 \ln S}{d (\ln \nu)^2}\right|_\nu$$
   where $S(\nu)$ follows the local expansion $\ln S(\nu) \approx \ln S(\nu_0) + \alpha \ln(\nu/\nu_0) + \frac{1}{2} q \ln^2(\nu/\nu_0)$.
6. **Automatic Antenna Beam Scaling**: When `--ant-dia-met` is specified and `--FOV-arcmin` is omitted, each frequency band automatically receives its primary beam radius at that frequency ($\theta \propto \lambda / D \propto 1/\nu$).

---

## 3. Command-Line Options Reference

### Field Centre & Pointing
| Option | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `--field-centre` | `str` (or list) | ELAIS-N1 (`242.580444 55.048777`) | Field centre pointing coordinates. Accepts multiple flexible coordinate formats. |

#### Supported Formats for `--field-centre`:
* **Decimal degrees**: `--field-centre 242.5804 55.0488` or `--field-centre "242.5804, 55.0488"`
* **Sexagesimal HMS / DMS**: `--field-centre "16h10m19.3s +55d02m55.6s"`
* **Colon-separated**: `--field-centre "16:10:19.3 +55:02:55.6"`
* **Space-separated 6 values**: `--field-centre "16 10 19.3 +55 02 55.6"`

---

### Frequencies & Multi-Frequency Outputs
| Option | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `--freq-MHz` | `float` (list) | `[400.0]` | Central frequencies in **MHz**. Accepts comma- or space-separated lists (e.g. `400,610` or `150 325 610`). Produces one dedicated CSV catalogue file per frequency. |

---

### Field of View & Antenna Beam Modes
| Option | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `--FOV-arcmin` / `--FOV-arc-min` | `float` (single or list) | `30.0` (or from antenna) | Direct field radius in **arcminutes** (e.g. `35.0` for all bands, or multiple values `46.0, 100.0` matching `--freq-MHz`). Sets the spatial boundary where sources are generated. |
| `--antenna-dia-met` / `--ant-dia-met` | `str` / `float` (single or list) | `None` | Antenna diameter in **meters** (e.g. `68.` for GMRT, `13.5` for MeerKAT). Accepts either a single diameter (applied to all bands) or multiple diameters matching `--freq-MHz` (e.g. `68., 45.`). Aliases: `--antenna-dia`, `--ant-dia`, `--antenna-beam-met`, `--ant-beam-met`, `--ant-beam`. When used alongside `--FOV-arcmin` and `--pb-effect`, sources are generated across the specified FOV and attenuated using this antenna beam response. If `--FOV-arcmin` is omitted, automatically sets FOV to the antenna beam radius $\theta(\nu) \propto 1/\nu$. |
| `--null-factor` / `--beam-factor` | `float` | `1.0` | Multiplier for the calculated beam radius. |

#### Beam Modes in `--antenna-dia-met`:
For dish diameter $D$ at frequency $\nu$ with wavelength $\lambda = c / \nu$:
* `null1`: **1st Airy Null** (default)
  $$\theta = 1.220 \frac{\lambda}{D}\ \text{rad} = 1.220 \frac{c}{\nu D} \times \frac{10800}{\pi}\ \text{arcmin}$$
* `null2`: **2nd Airy Null**
  $$\theta = 2.233 \frac{\lambda}{D}\ \text{rad} = 2.233 \frac{c}{\nu D} \times \frac{10800}{\pi}\ \text{arcmin}$$
* `fwhm`: **Full Width at Half Maximum (FWHM)**
  $$\theta = 1.029 \frac{\lambda}{D}\ \text{rad} = 1.029 \frac{c}{\nu D} \times \frac{10800}{\pi}\ \text{arcmin}$$
* `fwhmb2`: **FWHM by 2 (Half Width at Half Maximum / Half-Power Radius)**
  $$\theta = 0.5145 \frac{\lambda}{D}\ \text{rad} = 0.5145 \frac{c}{\nu D} \times \frac{10800}{\pi}\ \text{arcmin}$$

Any mode can be paired with a numeric multiplier $M$ (e.g. `--antenna-dia-met 45 null1 1.3` or `--antenna-dia-met 45 fwhmb2 1.2`), making the radius $R = M \times \theta$. Multiple diameters can also be passed directly matching the frequencies, e.g. `--antenna-dia-met 68., 45.`.

---

### Flux Limits, Source Caps & Outputs
| Option | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `--min-flux-mjy` | `float` (single or list) | `None` | Optional minimum flux density threshold in **mJy** (e.g. `0.11` for all bands, or multiple values `0.11, 0.25` matching `--freq-MHz`). If a single value is given, it is applied to all frequencies. |
| `--pb-effect` / `-pb-effect` | `flag` | `False` | Apply primary beam attenuation $PB(\theta, \nu) = [2 J_1(x)/x]^2$ to source fluxes and filter out sources whose attenuated flux falls below `--min-flux-mjy`. `total_flux_mjy` records the attenuated flux. |
| `--max-sources` | `int` | `None` | Optional maximum number of sources to retain in each catalogue (keeps the top $N$ brightest sources). |
| `--catalogue-name`| `str` | `psmodel_trecs_cat` | Base filename for the output catalogues. |
| `--dry-run` | `flag` | `False` | Displays expected source counts, expected **minimum and maximum flux** (formatted on separate lines per band), peak RAM, disk space, and runtime estimates without running T-RECS. |
| `--seed` | `int` | `666` | Random seed for simulation reproducibility. |

---

## 4. Output Catalogue Schema

Each generated `.csv` catalogue in [`catalogue/`](../catalogue/) contains **strictly the following 12 columns**:

| Column Name | Units | Description |
| :--- | :--- | :--- |
| `id` | `int` | Master unique integer source identifier assigned from the master simulation. |
| `ra` | `deg` | Right Ascension (J2000) in decimal degrees. |
| `dec` | `deg` | Declination (J2000) in decimal degrees. |
| `separation_arcmin` | `arcmin` | Radial angular distance from the field centre pointing. |
| `total_flux_mjy` | `mJy` | Total integrated flux density at this catalogue's frequency. |
| `spectral_index` | `dimensionless` | Local spectral index $\alpha(\nu) = d\ln S / d\ln \nu$ at frequency $\nu$. |
| `spectral_curvature` | `dimensionless` | Local spectral curvature $q(\nu) = d^2\ln S / d(\ln \nu)^2$ at frequency $\nu$. |
| `bmaj` | `arcsec` | Intrinsic major axis FWHM in arcseconds (`-100` for unresolved/point sources). |
| `bmin` | `arcsec` | Intrinsic minor axis FWHM in arcseconds (`-100` for unresolved/point sources). |
| `pa` | `deg` | Position angle in degrees east of north (`-100` for unresolved/point sources). |
| `redshift` | `dimensionless` | Cosmological redshift $z$. |
| `radioclass` | `code` | Radio morphology class code (`0` = SFG, `1` = Starburst, `2` = AGN FRI, `3` = AGN FRII, etc.). |

---

## 5. Usage Examples

> [!IMPORTANT]
> Always execute commands from the `workdir/` directory.

### Example 1: Multi-Frequency Common Sky (400 MHz & 610 MHz, 1024 Sources)
Simulate both frequencies together with identical sources in both files:
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
../scripts/psmodel-trecs-cat \
    --FOV-arcmin 30.0 \
    --freq-MHz 400,610 \
    --max-sources 1024 \
    --catalogue-name elaisn1_dualband
```
**Outputs created:**
* [`catalogue/elaisn1_dualband_400MHz.csv`](../catalogue/) (1,024 sources)
* [`catalogue/elaisn1_dualband_610MHz.csv`](../catalogue/) (1,024 sources, exact same IDs and coordinates)
* `logs/elaisn1_dualband_<timestamp>.log`

---

### Example 2: Automatic FOV using Beam Modes (`fwhmb2`, `fwhm`, `null1`, `null2`)
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
# Using half-power beam radius (FWHM/2):
../scripts/psmodel-trecs-cat \
    --field-centre "16h10m19.3s +55d02m55.6s" \
    --ant-dia-met 45 fwhmb2 \
    --freq-MHz 400,610 \
    --min-flux-mjy 1.0 \
    --catalogue-name gmrt_halfpower
```
* Calculates field radius to the half-power point (FWHM/2) at $\nu_{\max} = 610$ MHz: $R \approx 19.3\ \text{arcmin}$.
* Retains sources with flux $\ge 1.0\ \text{mJy}$.

---

### Example 3: Dry-Run Resource & Flux Range Estimation
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
../scripts/psmodel-trecs-cat \
    --ant-dia-met 45 null1 1.3 \
    --freq-MHz 400,610 \
    --min-flux-mjy 0.5 \
    --dry-run
```
Outputs estimated source counts, expected minimum and maximum flux across bands (displayed on separate lines), peak RAM, disk space, and simulation runtime.

---

### Example 4: Primary Beam Attenuation (`--pb-effect`)
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
../scripts/psmodel-trecs-cat \
    --ant-dia-met 68. \
    --freq-MHz 400 \
    --min-flux-mjy 0.11 \
    --pb-effect \
    --catalogue-name gmrt_pb_attenuated
```
* Multiplies each candidate's intrinsic flux by the circular aperture Airy primary beam factor $PB(\theta, \nu) = \left[\frac{2 J_1(x)}{x}\right]^2$, where $x = \frac{\pi D \theta}{\lambda}$.
* Filters out sources whose attenuated (apparent) flux falls below $0.11\ \text{mJy}$.
* Writes the resulting apparent flux into `total_flux_mjy` in [`catalogue/gmrt_pb_attenuated_400MHz.csv`](../catalogue/).

---

### Example 5: Independent FOV & Flux Limits per Frequency
Simulate multiple bands where each frequency has its own field radius and sensitivity cutoff:
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
../scripts/psmodel-trecs-cat \
    --freq-MHz 400,610 \
    --FOV-arcmin 45.0, 30.0 \
    --min-flux-mjy 0.11, 0.25 \
    --catalogue-name dualband_multidim
```
* **400 MHz catalogue**: Sources within $45.0'$ having flux $\ge 0.11\ \text{mJy}$.
* **610 MHz catalogue**: Sources within $30.0'$ having flux $\ge 0.25\ \text{mJy}$.
* Sources appearing in both catalogues share the exact same `id`, `ra`, `dec`, morphology, and redshift.

---

### Example 6: Generating to Specified FOVs and Correcting with Antenna Beam
Simulate sources out to custom field radii (e.g. into the outer beam or sidelobes) while correcting source flux densities using the physical antenna dish diameter:
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
../scripts/psmodel-trecs-cat \
    --freq-MHz 400,610 \
    --FOV-arc-min 46.,100. \
    --antenna-dia-met 68. \
    --min-flux-mjy 0.11,0.14 \
    --pb-effect \
    --seed 134 \
    --catalogue-name trecs-400-fov-46-mf-0.11-pbeffect
```
* **Field Generation**: Sources are generated out to $46.0'$ at 400 MHz and $100.0'$ at 610 MHz.
* **Beam Attenuation**: Flux densities in both bands are attenuated using the 68-meter circular aperture Airy pattern $PB(\theta, \nu) = [2 J_1(x)/x]^2$, where $x = \frac{\pi D \theta}{\lambda}$ with $D = 68.0\ \text{m}$.
* Sources whose attenuated flux falls below the respective cutoff ($0.11\ \text{mJy}$ at 400 MHz, $0.14\ \text{mJy}$ at 610 MHz) are excluded.

---

### Example 7: Multiple Antenna Diameters per Frequency
Specify different physical dish diameters for each frequency band:
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
../scripts/psmodel-trecs-cat \
    --freq-MHz 400,610 \
    --FOV-arcmin 46.,100. \
    --antenna-dia-met 68., 45. \
    --min-flux-mjy 0.11, 0.14 \
    --pb-effect \
    --catalogue-name multidia_demo
```
* 400 MHz sources attenuated by $D = 68.0\ \text{m}$ beam pattern.
* 610 MHz sources attenuated by $D = 45.0\ \text{m}$ beam pattern.




