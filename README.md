# DSSC Dye Descriptor Toolkit

A growing set of small, self-contained calculators that turn raw DFT/TD-DFT output
into quantitative structural, electronic, and optical descriptors for dye-sensitized
solar cell (DSSC) sensitizers. Built one descriptor category at a time.

**Live site:** `https://<your-username>.github.io/<this-repo>/`

---

## Status

| Category | Tool | Status |
|---|---|---|
| Geometrical | [Coplanarity Descriptor](coplanarity.html) | ✅ Live |
| Electronic | HOMO/LUMO, ΔGinj/ΔGreg, hardness/softness, electrophilicity | 🔜 Planned |
| Optical | Oscillator strength, CT character, absorption onset | 🔜 Planned |

---

## Tool 1 — Coplanarity Descriptor

Takes a list of dihedral angles (from a DFT-optimized geometry) and returns a single
quantitative coplanarity value — plus a separate, complementary whole-molecule
**planarity** check (MPP/SDP) for when full atomic coordinates are available.

### Terminology
- **Coplanarity** — the term consistently used in DSSC/D–π–A dye papers for
  dihedral-angle-based assessments of whether linked donor/bridge/acceptor units
  share a plane. This is what the dihedral-list input below computes.
- **Planarity** — reserved here for whole-molecule, atom-position-based metrics
  (MPP/SDP, Mode C), which answer a different question (does the *entire* structure
  sit on one plane, not just whether two linked rings agree with each other).

### The formula
```
per junction:        c = cos²(φ)
single dihedral:      C = cos²(φ)                         (your typical case)
multiple junctions:    C = [ c₁·c₂·...·cₙ ]^(1/n)           (geometric mean)
```

**Why cos²(φ), not a linear average.** A naive `average(φ)/180°` formula has two real
bugs: dihedral angles are circular (−178° and +179° describe nearly the same geometry
but average to ~0°, not ~179°), and it scores 0°/180° — both physically planar — at
opposite ends of the scale. cos²(φ) fixes both for free: cos²(0°) = cos²(180°) = 1,
and cosine has no wraparound discontinuity.

**This isn't an arbitrary fix** — it's grounded in real physics and has real
literature precedent:
- The overlap between two π-systems joined by a single bond scales as cos²(φ) of the
  dihedral between them, a direct consequence of the p-orbital overlap integral.
  Confirmed in STM break-junction conductance measurements of torsion-controlled
  biphenyl-dithiols and in nonlinear-optical response measurements of
  torsion-restricted push–pull biarylenes.
- This exact functional form has been published as a planarity index, **⟨cos²φ⟩**, by
  **Che, Y. & Perepichka, D. F., *Quantifying Planarity in the Design of Organic
  Electronic Materials*, Angew. Chem. Int. Ed. 2021, 60, 1364–1373**, and used
  identically in at least one high-profile conjugated-polymer study (planarity index
  reaching 0.93 for the best-planarized building block reported there).
- Their index is technically a Boltzmann-weighted average over a full torsional
  potential-energy scan (accounting for thermal population away from equilibrium).
  What this tool computes — cos² of a single DFT-optimized dihedral angle — is the
  practical, single-point (0 K) approximation of that same index, consistent with how
  the rest of the dye literature already treats individual equilibrium dihedral angles.

**Why geometric mean for multiple junctions** (this combination step is a reasoned
extension proposed here, not itself a published formula — no paper was found
combining several distinct backbone torsions into one number). Conjugation must pass
through every junction in sequence, so one badly-twisted junction should dominate the
whole-molecule score the way a weak link dominates a chain. Geometric mean does
exactly that — a single 90° junction collapses the whole product to 0 — whereas an
arithmetic mean would just dilute it. It reduces exactly to cos²(φ) when there's only
one torsion, which is the typical case for these dyes today.

Legacy reference values (`p_linear`, the corrected linear-deviation average, and
`p_cos`, the arithmetic mean of `|cos φ|`) are still shown alongside `C` for
transparency, so you can see how the recommendation differs from a simple average.

### Mode C — whole-molecule Planarity (MPP/SDP)
Fits a least-squares plane through a set of atoms and measures RMS deviation (MPP)
and total spread (SDP) from that plane — see [Lu, T., *J Mol Model* 27, 263
(2021)](http://sobereva.com/multiwfn/) for the original metric. Useful when you have
full 3D coordinates and want a planarity check that doesn't require picking which
dihedral matters. The in-browser 3×3 eigensolver was cross-checked against NumPy's
SVD on synthetic test cases (flat point sets, the same set rigidly rotated, a
perturbed set, a unit cube) and matches to machine precision in every case.

**Built-in example:** N719's ground-state DFT geometry (CAM-B3LYP/genecp, 59 atoms),
extracted from a Gaussian log file. Whole-molecule MPP ≈ 2.04 Å — deliberately large,
since N719 is an octahedral Ru(II) polypyridyl complex, not a flat conjugated system.
Use the atom-range field to restrict the fit to one conjugated wing for a more
chemically meaningful comparison.

No geometry or dihedral data is preloaded yet for FPCN/FTCN/FFCN — paste your own
DFT-optimized coordinates or dihedral angles via "+ Custom molecule" and they'll be
computed and compared alongside N719.

### Input formats accepted (Mode C, auto-detected)
- Plain `Element  x  y  z` lines
- A full `.xyz` file (atom-count + comment header, then the same lines)
- A raw Gaussian "Standard orientation" / "Input orientation" table, pasted directly —
  including text headers and dashed separator lines; these are skipped automatically.

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
├── coplanarity.html    ← Tool 1: coplanarity (dihedral) + planarity (MPP/SDP) descriptors
└── README.md           ← this file
```

Each tool is a single HTML file (HTML + CSS + JS, Chart.js from CDN) — no build step,
no install. New descriptor categories will be added as additional standalone HTML
files, linked from `index.html`, as they're validated against real DFT/TD-DFT data.

---

## License

MIT — free to use, modify, and share.
