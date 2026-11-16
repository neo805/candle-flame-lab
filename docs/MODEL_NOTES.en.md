# Model notes: understanding the overflow diagnostic

> [中文](MODEL_NOTES.zh-CN.md) · English

## It is not a mass-leakage measurement

The fixed-volume cells use this diagnostic:

```text
O = 100 × (nRT / (P_atm V_cell) − 1)
```

`nRT/P_atm` is the equivalent volume of the gas at 1 atm. Thus `O` can be read as an equivalent-volume deviation or as a relative pressure deviation at fixed cell volume. The model has no pressure-Poisson solve, divergence projection, or per-step isobaric renormalization.

The diagnostic is pedagogically useful, but it does not measure how much mass leaks from a real flame. Color-bar saturation changes only the display, not the underlying field.

## Why it departs from zero

1. **Exchange scales on a non-uniform mesh.** Without velocity, the original donor-cell face flux is proportional to `n_i = c_i V_i`. If adjacent cells have different volumes, opposite fluxes need not cancel even when `c_i = c_j`. Pairwise bookkeeping can conserve the global total while the local field remains physically inconsistent.
2. **Heating at fixed volume.** If the molar amount temporarily remains at its 300 K value, heating to 1000, 2000, or 2700 K alone gives approximately 233%, 567%, or 800%.
3. **Reaction-driven changes in gas moles.** `C25H52(g) → 12.5 C2H2 + 13.5 H2` converts 1 mol of gas into 26 mol. Together with heating, this can amplify the diagnostic without violating elemental mass conservation.

Cold tests with fuel and reactions disabled still develop large deviations on the original graded mesh, while a fully uniform mesh remains at zero. Adjusting only the color scale or temperature therefore cannot address the cause.

## Optional improvements

### A. Rename and explain the diagnostic only

Use “equivalent-volume deviation at 1 atm” or “relative pressure deviation at fixed cell volume,” and display temperature and gas moles alongside it. This leaves every trajectory unchanged but does not improve the numerical field.

### B. Use a shared-face transport scale (preferred first enhancement)

For example:

```text
V* = 2 V_i V_j / (V_i + V_j)
F  = 4 D q c_i V*
```

Without advection, opposite fluxes cancel exactly for a uniform concentration field, and the formula reduces to the original form on a uniform mesh. A complete implementation should treat both axial and radial graded interfaces.

This route **does not require changing the three classroom tasks or their code structure**. It does change reference trajectories and figures, so it is better suited to a post-lesson version or a separate experiment branch.

### C. Add optional isobaric relaxation after B

Relax toward `n_iso = P_atm V_cell / (RT)` after reactions and source terms, while keeping a per-species virtual-outflow or entrainment ledger. Silently rescaling every cell would hide, rather than solve, the accounting problem.

### D. Use a full low-Mach pressure solve

Pressure-Poisson, divergence, face-flux, and boundary-work terms would be more complete, but the complexity approaches a full CFD solver and exceeds the teaching goal.

## Recommendation

Keep deterministic code fixes separate from these modelling choices. Use A when the original classroom trajectory must be preserved. For an enhanced post-lesson version, validate B first and then decide whether C is necessary. Retain uniform-field, soot-conservation, temperature/species-extension, and reference-trajectory regression tests with every route.
