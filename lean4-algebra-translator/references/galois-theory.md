# Field Theory & Galois Theory — Lean 4 / Mathlib4 Reference

## Imports

```lean
import Mathlib.FieldTheory.Galois.Basic
import Mathlib.FieldTheory.SplittingField.Construction
import Mathlib.FieldTheory.Adjoin
import Mathlib.FieldTheory.IntermediateField.Basic
import Mathlib.FieldTheory.Separable
-- or: import Mathlib
```

---

## Field extensions

### Basic setup

```lean
-- F ≤ K (field extension)
variable {F K : Type*} [Field F] [Field K] [Algebra F K]

-- Degree of extension [K : F]
FiniteDimensional.finrank F K    -- ℕ, requires [FiniteDimensional F K]

-- Finite extension
[FiniteDimensional F K]

-- Algebraic extension
Algebra.IsAlgebraic F K   -- every element of K is algebraic over F
IsAlgebraic F x           -- x : K is algebraic over F

-- Minimal polynomial
minpoly F x               -- the minimal polynomial of x over F
-- Type: F[X]
minpoly.irreducible        -- it is irreducible
minpoly.aeval_eq_zero      -- x is a root: aeval x (minpoly F x) = 0
minpoly.degree_dvd         -- degree divides [K : F]
```

### Adjoining elements

```lean
-- F(α) as an intermediate field
F⟮α⟯                           -- IntermediateField.adjoin F {α}
IntermediateField.adjoin F {α}

-- Degree formula: [F(α) : F] = deg(minpoly F α)
IntermediateField.adjoin.finrank  -- finrank F F⟮α⟯ = natDegree (minpoly F α)

-- Primitive element theorem
Field.exists_primitive_element   -- ∃ α, F⟮α⟯ = ⊤ (separable extensions)
```

### Tower law

```lean
-- [L : F] = [L : K] * [K : F] for F ≤ K ≤ L
FiniteDimensional.finrank_mul_finrank  -- finrank F L = finrank K L * finrank F K
```

### Algebraic closure

```lean
[IsAlgClosed K]           -- K is algebraically closed
IsAlgClosed.exists_aeval_eq_zero  -- every poly of degree ≥ 1 has a root
```

---

## Splitting fields

```lean
-- The splitting field of a polynomial
Polynomial.SplittingField p   -- as a type
Polynomial.splits             -- p splits over K ↔ all roots in K
Polynomial.Splits.of_degree_le

-- Constructing the splitting field
IsSplittingField F K p    -- K is *a* splitting field of p over F
IsSplittingField.adjoin_rootSet  -- K = F adjoin (roots of p)
```

---

## Galois extensions

### The key class

```lean
[IsGalois F K]    -- K/F is Galois (= separable + normal)

-- Equivalently:
IsGalois.separable   -- K/F separable
IsGalois.normal      -- K/F normal (= splitting field of sep poly)
```

### The Galois group

```lean
-- The Galois group Gal(K/F)
K ≃ₐ[F] K           -- AlgEquiv F K K — type of F-algebra automorphisms of K

-- As a group (with composition)
-- Note: the group structure is via MulEquiv

-- Key facts
IsGalois.card_aut_eq_finrank  -- |Gal(K/F)| = [K : F]
-- i.e., Fintype.card (K ≃ₐ[F] K) = FiniteDimensional.finrank F K
```

### Galois correspondence

```lean
-- Fixed field of a subgroup
IntermediateField.fixedField H    -- H : Subgroup (K ≃ₐ[F] K)

-- Fixing subgroup of an intermediate field
IntermediateField.fixingSubgroup E  -- E : IntermediateField F K

-- The correspondence (as an order isomorphism)
IsGalois.intermediateFieldEquivSubgroup  
-- IntermediateField F K ≃o (Subgroup (K ≃ₐ[F] K))ᵒᵈ

-- Fixed field of fixing subgroup = original field
IsGalois.fixedField_fixingSubgroup  
-- fixedField (fixingSubgroup E) = E

-- Fixing subgroup of fixed field = original subgroup
IsGalois.fixingSubgroup_fixedField  
-- fixingSubgroup (fixedField H) = H
```

### Key Galois theorems

```lean
-- A subextension is Galois iff corresponding subgroup is normal
IsGalois.normal_iff_isGalois_fixedField  
-- H.Normal ↔ IsGalois F (fixedField H)

-- Degree formula via Galois correspondence
-- [K : fixedField H] = |H|
IsGalois.card_fixingSubgroup_eq_finrank
-- Fintype.card H = finrank (fixedField H) K
```

---

## Characteristic and Frobenius

```lean
-- Characteristic
CharP R p                   -- R has characteristic p
CharP.cast_eq_zero          -- (p : R) = 0
ringChar R                  -- the characteristic as ℕ

-- Characteristic zero
CharZero R                  -- char 0
-- Implies: (n : R) ≠ 0 for n : ℕ, n ≠ 0

-- Frobenius endomorphism (char p)
frobenius R p               -- x ↦ x^p
frobenius_def               -- frobenius R p x = x ^ p

-- Perfect fields
[PerfectField K]            -- Frobenius is bijective
```

---

## Separability

```lean
-- A polynomial is separable
Polynomial.Separable p    -- separable ↔ gcd(p, p') = 1

-- An element is separable over F
IsSeparable F x           -- minpoly F x is separable

-- An extension is separable
Algebra.IsSeparable F K   -- all elements of K are separable over F
```

---

## Worked examples

### Degree of extension equals Galois group order

```lean
example {F K : Type*} [Field F] [Field K] [Algebra F K]
    [FiniteDimensional F K] [IsGalois F K] :
    Fintype.card (K ≃ₐ[F] K) = FiniteDimensional.finrank F K :=
  IsGalois.card_aut_eq_finrank F K
```

### Fixed field of whole Galois group is the base field

```lean
-- In Mathlib: IsGalois.fixedField_top
example {F K : Type*} [Field F] [Field K] [Algebra F K] [IsGalois F K] :
    IntermediateField.fixedField (⊤ : Subgroup (K ≃ₐ[F] K)) = ⊥ :=
  IsGalois.fixedField_top F K
```

### Tower law application

```lean
example {F K L : Type*} [Field F] [Field K] [Field L]
    [Algebra F K] [Algebra K L] [Algebra F L] [IsScalarTower F K L]
    [FiniteDimensional F K] [FiniteDimensional K L] :
    FiniteDimensional.finrank F L =
    FiniteDimensional.finrank K L * FiniteDimensional.finrank F K :=
  FiniteDimensional.finrank_mul_finrank F K L
```

### Splitting field of x^n - a

```lean
-- Proving a splitting field exists
example (p : Polynomial ℚ) (hp : p.degree ≠ 0) :
    ∃ K : Type _, ∃ (_ : Field K) (_ : Algebra ℚ K), IsSplittingField ℚ K p :=
  ⟨p.SplittingField, inferInstance, inferInstance, inferInstance⟩
```

---

## Naming pattern guide (Galois / field theory)

Lean's naming convention for this domain:

| Mathematical concept | Lean name pattern | Example |
|---|---|---|
| is Galois | `IsGalois` | `IsGalois F K` |
| Galois group element | `K ≃ₐ[F] K` | `σ : K ≃ₐ[F] K` |
| fixed field | `fixedField H` | `IntermediateField.fixedField H` |
| intermediate field | `IntermediateField F K` | |
| minimal polynomial | `minpoly F x` | `(minpoly F x).degree` |
| splitting field | `SplittingField p` / `IsSplittingField` | |
| algebraic element | `IsAlgebraic F x` | |
| separable | `Polynomial.Separable` / `Algebra.IsSeparable` | |
| char p | `CharP R p` | `[CharP K p]` |
| degree of ext | `finrank F K` | `FiniteDimensional.finrank F K` |
