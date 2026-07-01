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

Takes a list of backbone dihedral angles (from DFT-optimized geometry) and returns a
three-descriptor coplanarity profile — plus a separate, complementary whole-molecule
**Planarity** check (MPP/SDP) for when full atomic coordinates are available.

### The new descriptor framework (Π_DA / C_avg / β_eff)

This version introduces a three-descriptor coplanarity framework derived from two
independently established physical theories:

**Step 1 — per-junction coupling retention (established physics):**
```
c_i = cos²(φ_i)
```
The fraction of π-orbital coupling retained at each rotatable junction scales as
cos²(φ) of the dihedral angle — a direct consequence of the p-orbital overlap
integral. Confirmed experimentally by Venkataraman et al. in single-molecule STM
break-junction conductance measurements of torsion-controlled biphenyldithiols, where
conductance G ∝ cos²(θ) across the full 0–90° range.[1,2]

**Step 2 — multi-junction combination (new application of established theory):**
```
Π_DA = c₁ × c₂ × ... × cₙ = ∏ᵢ cos²(φᵢ)       [new descriptor 1]
C_avg = Π_DA^(1/n)                                [new descriptor 2]
β_eff = −ln(C_avg)   (nats / junction)            [new descriptor 3]
```
The combination rule for Π_DA is derived from McConnell's superexchange model for
donor–bridge–acceptor electron transfer (J. Chem. Phys. 1961, 35, 508).[3] In that
model, the electronic coupling between a donor and an acceptor mediated by a chain of
bridge units factorizes as a **raw product** of the per-unit couplings — not an
n-th-root average. Applying the same factorization principle to dihedral-twist-induced
decoupling gives Π_DA as the physically meaningful quantity: the actual fraction of
donor→acceptor electronic coupling that survives transmission through the full
π-bridge.

**What each descriptor means:**

| Descriptor | Physical meaning | Good for |
|---|---|---|
| **Π_DA** | Total D→A coupling retained through the whole bridge | Absolute performance ranking within a series of dyes |
| **C_avg** = Π_DA^(1/n) | Per-junction coupling retention, length-normalized | Comparing dyes with different numbers of junctions fairly |
| **β_eff** = −ln(C_avg) | Decoupling exponent (nats/junction) | Analogous to the β constant for molecular-wire electron-transfer rates |

**Score bands for C_avg** (derived from the physical meaning of cos²φ as a
coupling-retention fraction, consistent with Che & Perepichka 2021[4]):

| Band | C_avg | Π_DA (n=2) | Physical meaning |
|---|---|---|---|
| **Planar** | ≥ 0.90 | ≥ 0.81 | ≥90% coupling retained per junction |
| **Moderate** | 0.50–0.90 | 0.25–0.81 | 50–90% retained; includes biphenyl at ~23° |
| **Twisted** | < 0.50 | < 0.25 | <50% retained; φ > 45° at any junction |

**What is new vs. established:**
- Per-junction cos²(φ) law: established (Venkataraman 2006, Che & Perepichka 2021)
- Product-combination rule for multi-junction backbones: adapted from McConnell
  superexchange theory (1961), but its **application as a DSSC coplanarity
  descriptor** (Π_DA, C_avg, β_eff) is proposed here for the first time
- The previous geometric-mean C = [∏cos²φ]^(1/n) (numerically identical to C_avg)
  is still correct as a per-junction average, but it lacked physical derivation;
  Π_DA now provides the physically motivated primary descriptor

Legacy reference values (p_cos = mean of |cos φ|, p_linear = corrected linear
average) are still shown alongside for transparency and comparison.

### Mode C — whole-molecule Planarity (MPP/SDP)

Fits a least-squares plane through a set of atoms and measures RMS deviation (MPP)
and total spread (SDP) — see Lu (2021)[5] for the original metric. The in-browser
3×3 eigensolver was cross-checked against NumPy's SVD and matches to machine
precision.

### References

[1] Venkataraman, L. et al. Dependence of single-molecule junction conductance on
    molecular conformation. *Nature* 2006, 442, 904–907.

[2] Mishchenko, A. et al. Influence of Conformation on Conductance of Biphenyl-
    Dithiol Single-Molecule Contacts. *Nano Lett.* 2010, 10, 156–163.

[3] McConnell, H. M. Intramolecular Charge Transfer in Aromatic Free Radicals.
    *J. Chem. Phys.* 1961, 35, 508–515.

[4] Che, Y. & Perepichka, D. F. Quantifying Planarity in the Design of Organic
    Electronic Materials. *Angew. Chem. Int. Ed.* 2021, 60, 1364–1373.

[5] Lu, T. Simple, reliable, and universal metrics of molecular planarity.
    *J Mol Model* 2021, 27, 263.

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
├── index.html          ← hub/landing page
├── coplanarity.html    ← Tool 1: Π_DA / C_avg / β_eff coplanarity + MPP/SDP planarity
└── README.md           ← this file
```

Each tool is a single HTML file (HTML + CSS + JS, Chart.js from CDN) — no build step.

---

## License

MIT — free to use, modify, and share.
