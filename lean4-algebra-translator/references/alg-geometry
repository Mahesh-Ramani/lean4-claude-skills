# Algebraic Geometry — Lean 4 / Mathlib4 Reference

> **Coverage note:** Mathlib4's algebraic geometry formalization is substantial but not
> complete as of 2025. Schemes, sheaves, and morphisms of schemes are present; many
> advanced results (e.g., Riemann-Roch, étale cohomology) are not yet formalized.
> Always check https://leanprover-community.github.io/mathlib4_docs for the latest state.

## Imports

```lean
import Mathlib.AlgebraicGeometry.Scheme
import Mathlib.AlgebraicGeometry.Morphisms.Basic
import Mathlib.AlgebraicGeometry.StructureSheaf
import Mathlib.Topology.Sheaves.Sheaf
-- or: import Mathlib
```

---

## Spectra and affine schemes

```lean
open AlgebraicGeometry

-- Spectrum of a commutative ring
Spec R                  -- PrimeSpectrum R as a topological space
-- (also: AlgebraicGeometry.Spec.obj (CommRingCat.of R) as a Scheme)

-- Prime spectrum
PrimeSpectrum R         -- the set of prime ideals of R
PrimeSpectrum.asIdeal   -- the underlying ideal of a prime

-- The structure sheaf
Spec.structureSheaf R   -- the sheaf O_X on Spec R

-- Sections of the structure sheaf
(Spec.structureSheaf R).obj U  -- sections over an open U

-- Zariski topology
PrimeSpectrum.basicOpen f      -- D(f) = {p | f ∉ p}
PrimeSpectrum.isBasis_basic_opens  -- basic opens form a basis

-- The stalk at a prime p
(Spec.structureSheaf R).stalk p  ≅ Localization.AtPrime p.asIdeal
```

---

## Schemes

```lean
-- A scheme
X : Scheme

-- Underlying topological space
X.carrier           -- the underlying type
X.topologicalSpace  -- TopologicalSpace instance

-- Structure sheaf
X.presheaf          -- the structure presheaf
X.sheaf             -- the structure sheaf

-- Affine schemes
IsAffine X          -- X is affine
-- Affine ↔ X ≅ Spec R for some R

-- The global sections functor
Scheme.Γ.obj X      -- global sections Γ(X, O_X)
```

---

## Morphisms of schemes

```lean
-- A morphism of schemes
f : X ⟶ Y           -- in the category Scheme

-- The category of schemes
Scheme             -- CategoryTheory.Category Scheme

-- Special morphisms
IsOpenImmersion f   -- f is an open immersion
IsClosedImmersion f -- f is a closed immersion
IsAffineHom f       -- f is an affine morphism
IsFiniteType f      -- f is of finite type

-- Fiber products
pullback f g        -- X ×_Z Y where f : X → Z, g : Y → Z
```

---

## Varieties (classical algebraic geometry)

> Most "variety" results in Lean use the scheme language. Classical affine/projective 
> varieties are accessed via `AlgebraicGeometry.Scheme` and morphisms to Spec k.

```lean
-- Affine n-space
AlgebraicGeometry.Spec (MvPolynomial (Fin n) k)

-- Vanishing ideal of a set
-- (work directly with prime ideals and Spec)

-- Polynomial rings used for varieties
variable {k : Type*} [Field k]
MvPolynomial σ k     -- k[x₁,...,xₙ] where σ = Fin n typically
```

---

## Sheaf theory (general)

```lean
import Mathlib.Topology.Sheaves.Sheaf

-- A presheaf of rings on X
F : TopCat.Presheaf RingCat X

-- A sheaf
F : TopCat.Sheaf RingCat X

-- Stalks
F.stalk x            -- the stalk of F at x

-- Sheafification
sheafify F           -- the associated sheaf
```

---

## Worked examples

### Basic open in Spec R is Spec of a localization

```lean
-- The basic open D(f) ≅ Spec R[f⁻¹]
example {R : Type*} [CommRing R] (f : R) :
    IsAffineOpen (PrimeSpectrum.basicOpen f) := by
  exact PrimeSpectrum.basicOpen_isAffineOpen f
```

### Global sections of Spec R is R

```lean
example {R : Type*} [CommRing R] :
    (AlgebraicGeometry.Spec.obj (CommRingCat.of R)).presheaf.obj ⊤ = R := by
  -- This holds by definition / ring isomorphism
  rfl -- or: exact StructureSheaf.globalSections_eq R
```

---

## Naming patterns

| Concept | Lean name | Notes |
|---|---|---|
| Spectrum | `PrimeSpectrum R` | As a topological space |
| Affine scheme | `AlgebraicGeometry.Spec.obj` | As a Scheme |
| Structure sheaf | `Spec.structureSheaf` / `X.sheaf` | |
| Basic open D(f) | `PrimeSpectrum.basicOpen f` | |
| Stalk at p | `F.stalk p` | |
| Morphism of schemes | `X ⟶ Y` in `Scheme` category | |
| Affine morphism | `IsAffineHom f` | |
| Finite type | `AlgebraicGeometry.Morphisms.FiniteType` | |

---

## Warning: what is NOT in Mathlib (as of 2025)

- Riemann-Roch theorem
- Étale cohomology
- Intersection theory (Chow groups)
- Derived categories / derived algebraic geometry
- Motives
- Moduli spaces (general theory)
- Most of the theory of abelian varieties

For these topics, formalization will require `sorry`-filled placeholders.
Indicate this explicitly in output with `-- NOT IN MATHLIB: requires sorry`.
