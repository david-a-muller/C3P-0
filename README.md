# C3P-0

**Claude's CTF Code for Ptychography in the weak-phase limit**

C3P-0 is an interactive, browser-based simulator of the contrast transfer function of 4D-STEM data for a weak phase object. It shows the transfer function *T*(**K**<sub>f</sub>, **Q**<sub>p</sub>) as slices in detector space and in virtual-bright-field (vBF) space. It also compares how much of that transfer each reconstruction method recovers, as a noise-normalized CTF (NN-CTF) and a DQE.

It runs entirely in the browser as a single `index.html` with no dependencies or installation, so it works on any operating system. It was written for class demonstrations and for making figures.

**Live version:** <https://david-a-muller.github.io/C3P-0/>

## Using it

Use the live version, or open `index.html` locally in any modern browser.

The sidebar sets the optics: beam energy, convergence semi-angle α, and aberrations up to C5 in Krivanek notation, with presets. It also sets the partial coherence: a Gaussian focal spread and a source size. Hover over any plot to read values. Clicking a vBF or 2D NN-CTF panel adds that **Q**<sub>p</sub> to the detector-space series, and clicking a detector-space panel adds that pixel **K**<sub>f</sub> to the vBF series.

The page shows four views:

- **Noise-normalized CTF (NN-CTF).** Line or radially averaged profiles of η(**Q**<sub>p</sub>) and DQE(**Q**<sub>p</sub>) = η²/4, plotted for each reconstruction:
  - acBF
  - SSB
  - tcBF
  - iCoM/DPC
  - BF, with an adjustable detector radius β
  - ABF

  A dotted line marks the self-luminous limit, the upper bound for acBF.
- **2D NN-CTF η(Q<sub>p</sub>).** The same quantity over the full **Q**<sub>p</sub> plane, which shows anisotropy from astigmatism or coma.
- **Detector space.** *T*(**K**<sub>f</sub>) at a series of fixed **Q**<sub>p</sub>, showing the double- and triple-overlap regions of the bright-field disk with the disks shifted to ±**Q**<sub>p</sub>.
- **vBF space.** *T*(**Q**<sub>p</sub>) for a series of fixed detector pixels **K**<sub>f</sub>. Each panel is the CTF of that pixel's virtual bright-field image, with sidebands centered at ±**K**<sub>f</sub>.

*T* can be displayed as complex color (hue = phase), real or imaginary part, magnitude or phase. Either sideband term can also be shown on its own.

### Export

Every figure exports as:

- **PNG** at 4× resolution
- **SVG**, with vector text and outlines and the maps embedded as images
- **PDF**, with vector text and line art in the standard Helvetica and Symbol fonts

**Export all figures (PDF)** writes every figure as a multi-page PDF. The NN-CTF and DQE profiles also export as CSV. Raise *Samples per panel* and *K-sampling* before exporting final figures.

## Theory and conventions

The code uses the Nellist notation: **K**<sub>f</sub> is detector space, **Q**<sub>p</sub> is the Fourier transform of probe position, and K<sub>max</sub> = α/λ is the aperture cutoff. The probe is *A*(**K**) = *A*<sub>p</sub>(**K**) exp[−iχ(**K**)].

The transfer function of a weak phase object, *t* = 1 + iσφ, is

*G*(**K**<sub>f</sub>, **Q**<sub>p</sub>) = |*A*(**K**<sub>f</sub>)|² δ(**Q**<sub>p</sub>) + σ φ̃(**Q**<sub>p</sub>) *T*(**K**<sub>f</sub>, **Q**<sub>p</sub>),

where

*T*(**K**<sub>f</sub>, **Q**<sub>p</sub>) = i [ *A*\*(**K**<sub>f</sub>) *A*(**K**<sub>f</sub> − **Q**<sub>p</sub>) − *A*(**K**<sub>f</sub>) *A*\*(**K**<sub>f</sub> + **Q**<sub>p</sub>) ].

The combined transfer of a reconstruction with pixel weights *w* is the noise-normalized η = |Σ *w T*| / √(*N* Σ|*w*|²), summed over the *N* bright-field pixels at equal dose. The methods differ only in their weights:

| Method | Weights |
|---|---|
| BF | 1 inside a detector of radius β |
| ABF | 1 on an annulus |
| iCoM/DPC | **K**<sub>f</sub>·**Q̂**<sub>p</sub> |
| tcBF | exp[−i∇χ·**Q**<sub>p</sub>] |
| SSB | exp[−i arg *T*] on the double-overlap lunes only |
| acBF | exp[−i arg *T*] |

Three noise models set which pixels add noise at each **Q**<sub>p</sub>:

- **None: flat NPS** (default, as in Ma et al. [1]): every weighted pixel's vBF image carries white Poisson noise, so Σ|*w*|² runs over all weighted pixels. The self-luminous limit is then η = 2𝔸(**Q**<sub>p</sub>), the ADF-like aperture autocorrelation, and tcBF peaks reach it.
- **vBF mask, single + double sideband**: each pixel's image is first masked to its two sideband disks, so only pixels in the overlap support at that **Q**<sub>p</sub> add noise. This raises tcBF and acBF above α by removing noise from pixels that carry no signal there.
- **vBF mask, single sideband only**: each pixel's image keeps only the regions where one sideband is present. These are exactly the double-overlap lunes in detector space, so acBF with this mask is identical to SSB.

Detector space and vBF space are slices of the same 4D support: single sideband ↔ double overlap, double sideband ↔ triple overlap. The noise model applies to tcBF and acBF. BF, ABF and iCoM/DPC integrate detector signals, and SSB always uses only the double-overlap lunes, so the switch does not change them.

An ideal Zernike phase plate has η = 2, so DQE = η²/4. The theory section on the page gives the full derivations and the self-luminous limit.

Partial coherence is treated exactly for a Gaussian focal spread (an analytic damping envelope on each term) and for a Gaussian source (an envelope in **Q**<sub>p</sub>).

### Numerics

The combined transfer sums over a pixel grid of up to 256 px per K<sub>max</sub>, with **Q**<sub>p</sub> sampled at up to 64 per K<sub>max</sub>. The sums run in parallel Web Workers. The status line warns when the aberration phase is under-sampled.

## Credits

Developed by **David A. Muller** and **Claude** (Anthropic). Based on the information analysis of Ma et al. [1] and the Pelz lab's interactive [ptychographic CTF app](https://pelzlab.science/marimo/apps/ptychographic_ctf.html).

### Background literature

1. D. Ma, G. Li, D. A. Muller, S. E. Zeltmann (2026). Information in 4D-STEM: Where it is, and How to Use it. *Ultramicroscopy* **283**, 114351. <https://doi.org/10.1016/j.ultramic.2026.114351>
2. F. Bennemann, A. I. Kirkland, D. A. Muller, P. D. Nellist (2026). Detective Quantum Efficiency-Based Comparison of HRTEM and Ptychography Phase Imaging. *Microscopy and Microanalysis* **32**(2), ozag018. <https://doi.org/10.1093/mam/ozag018>
3. G. Varnavides, J. M. Bekkevold, S. M. Ribet, A. R. C. McCray, M. C. Scott, L. Jones, C. Ophus (2026). Beyond Contrast Transfer: Spectral SNR as a Finite-Dose Metric for STEM Phase Retrieval. *Microscopy and Microanalysis* **32**(2), ozag005. <https://doi.org/10.1093/mam/ozag005>

## License

MIT. See [LICENSE](LICENSE).
