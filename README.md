# DSSC Dye Descriptor Toolkit

A growing set of small, self-contained calculators that turn raw DFT/TD-DFT output
into quantitative structural, electronic, and optical descriptors for dye-sensitized
solar cell (DSSC) sensitizers. Built one descriptor category at a time.

**Live site:** `https://<your-username>.github.io/<this-repo>/`

---

## Status

| Category | Tool | Status |
|---|---|---|
| Geometrical | [Coplanarity / Planarity Descriptor](coplanarity.html) | ✅ Live |
| Electronic | HOMO/LUMO, ΔGinj/ΔGreg, hardness/softness, electrophilicity | 🔜 Planned |
| Optical | Oscillator strength, CT character, absorption onset | 🔜 Planned |

---

## Tool 1 — Coplanarity / Planarity Descriptor

Quantifies how planar a dye's conjugated backbone is, two independent ways:

### Mode A — corrected dihedral-angle averaging
```
δᵢ = distance of φᵢ from the nearest of {0°, 180°}      (handles angle wraparound + 0°/180° symmetry)
p_linear = 1 − (1/n)Σ(δᵢ/90°)                            1 = planar, 0 = maximally twisted
p_cos    = (1/n)Σ|cos φᵢ|                                 smooth/differentiable variant
```
This replaces a naive `average(φ)/180°` formula, which has two real bugs: dihedral
angles are circular (−178° and +179° describe almost the same geometry but average to
~0°, not ~179°), and it scores 0° and 180° — both physically planar — at opposite ends
of the scale.

### Mode C — Molecular Planarity Parameter (MPP) / Span of Deviation from Plane (SDP)
```
Fit a least-squares plane through a set of atoms (eigenvector of smallest variance
of the centered coordinate covariance matrix).
MPP = RMS(dᵢ)                where dᵢ = signed distance of atom i from that plane
SDP = max(dᵢ) − min(dᵢ)
```
This is the literature-standard metric (Lu, T. *Simple, reliable, and universal
metrics of molecular planarity*, J Mol Model 27, 263 (2021); also computable via the
free [Multiwfn](http://sobereva.com/multiwfn/) package). It sidesteps the core problem
with dihedral-based metrics — for a general molecule it's often genuinely ambiguous
*which* dihedral(s) should define "the" backbone torsion — by working directly on
atomic coordinates instead.

**Validation:** the in-browser 3×3 eigensolver (used to find the best-fit plane) was
cross-checked against NumPy's SVD on several synthetic test cases (a flat point set,
the same set rigidly rotated in 3D, a perturbed set, and a unit cube) and matches to
machine precision in every case.

**Built-in example:** N719's ground-state DFT geometry (CAM-B3LYP/genecp, 59 atoms),
extracted directly from a Gaussian log file. Loaded whole-molecule, MPP ≈ 2.04 Å —
deliberately large, because N719 is an octahedral Ru(II) polypyridyl complex and is
*not* a flat conjugated system by design. This is a useful worked example for why
whole-molecule MPP isn't always the meaningful number: use the atom-range field to
restrict the fit to one conjugated wing (e.g. one bipyridyl–NCS donor–acceptor arm)
for a chemically meaningful planarity comparison.

No geometry or dihedral data is preloaded yet for FPCN/FTCN/FFCN — paste your own
DFT-optimized coordinates or dihedral angles via "+ Custom molecule" and they'll be
computed and compared alongside N719.

### Input formats accepted (Mode C, auto-detected)
- Plain `Element  x  y  z` lines
- A full `.xyz` file (atom-count + comment header, then the same lines)
- A raw Gaussian "Standard orientation" / "Input orientation" table, pasted directly —
  including the text headers and dashed separator lines; these are skipped automatically.

---

## How to deploy on GitHub Pages

1. Create a new repository (or use this one).
2. Add `index.html` and `coplanarity.html` to the repo root.
3. Settings → Pages → Source: `main` branch, `/root`.
4. Visit `https://<username>.github.io/<repo>/`.

---

## Repository structure

```
dssc-descriptor-toolkit/
├── index.html         ← hub/landing page, links to each descriptor tool
├── coplanarity.html    ← Tool 1: geometrical / planarity descriptors
└── README.md           ← this file
```

Each tool is a single HTML file (HTML + CSS + JS, Chart.js from CDN) — no build step,
no install. New descriptor categories will be added as additional standalone HTML
files, linked from `index.html`, as they're validated against real DFT/TD-DFT data.

---

## License

MIT — free to use, modify, and share.
