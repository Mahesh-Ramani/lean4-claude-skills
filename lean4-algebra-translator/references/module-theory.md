# Module Theory & Linear Algebra — Lean 4 / Mathlib4 Reference

## Imports

```lean
import Mathlib.LinearAlgebra.Basic
import Mathlib.LinearAlgebra.Basis.Basic
import Mathlib.LinearAlgebra.Dimension.Basic
import Mathlib.LinearAlgebra.TensorProduct.Basic
-- or: import Mathlib
```

---

## Type classes

```lean
-- R-module M
variable {R : Type*} [CommRing R] {M : Type*} [AddCommGroup M] [Module R M]

-- Vector space (module over a field)
variable {K : Type*} [Field K] {V : Type*} [AddCommGroup V] [Module K V]

-- Finite-dimensional vector space
[FiniteDimensional K V]
```

---

## Submodules

```lean
-- Type
N : Submodule R M

-- Constructors
Submodule.span R S        -- span of a set S ⊆ M
Submodule.map f N         -- image of N under f : M →ₗ[R] M'
Submodule.comap f N       -- preimage

-- Membership
x ∈ N
Submodule.mem_span_iff    -- x ∈ span R S ↔ ...

-- Operations
N ⊓ P                    -- intersection
N ⊔ P                    -- sum
⊤                        -- whole module
⊥                        -- zero submodule

-- Quotient module
M ⧸ N                    -- Submodule.Quotient N (also written M ⧸ N)
Submodule.Quotient.mk    -- the quotient map
```

---

## Linear maps

```lean
-- Type
f : M →ₗ[R] N           -- LinearMap R M N

-- Kernel and range
LinearMap.ker f           -- Submodule R M
LinearMap.range f         -- Submodule R N

-- Properties
LinearMap.injective_iff_ker_eq_bot  -- injective ↔ ker = ⊥

-- Isomorphisms
e : M ≃ₗ[R] N            -- LinearEquiv R M N

-- Standard maps
LinearMap.id              -- identity
LinearMap.comp f g        -- composition

-- First isomorphism theorem
LinearMap.quotKerEquivRange  -- M ⧸ ker f ≃ₗ[R] range f
```

---

## Basis and dimension

```lean
-- A basis
b : Basis ι R M          -- ι indexes the basis vectors

-- Constructors
Basis.ofLinearIndependent    -- from linearly independent set
Basis.mk                     -- from injective linear map

-- Rank / dimension
Module.rank R M              -- as Cardinal
FiniteDimensional.finrank R M -- as ℕ (requires FiniteDimensional)

-- Key lemma: finrank of a subspace ≤ finrank of ambient
Submodule.finrank_le         -- finrank R N ≤ finrank R M

-- Rank-nullity theorem
LinearMap.finrank_range_add_finrank_ker  
-- finrank R (range f) + finrank R (ker f) = finrank R M

-- Dimension of product
FiniteDimensional.finrank_prod  -- finrank (M × N) = finrank M + finrank N
```

---

## Tensor products

```lean
import Mathlib.LinearAlgebra.TensorProduct.Basic

-- Type
M ⊗[R] N                 -- TensorProduct R M N

-- Elements
m ⊗ₜ n                   -- TensorProduct.tmul m n

-- Universal property: R-bilinear map → linear map on tensor product
TensorProduct.lift        -- TensorProduct.lift f : M ⊗[R] N →ₗ[R] P

-- Associativity
TensorProduct.assoc       -- (M ⊗ N) ⊗ P ≃ₗ M ⊗ (N ⊗ P)
```

---

## Free modules and finitely generated modules

```lean
-- Free module: R^n
Fin n → R                -- as a type, with Module R (Fin n → R)

-- Finitely generated module
Module.Finite R M         -- M is finitely generated as R-module

-- Nakayama's lemma (commutative algebra version)
Submodule.eq_top_of_smul_eq  -- Nakayama: if I * M ⊆ N and M f.g., then M = N under conditions
```

---

## Worked examples

### Rank-nullity

```lean
example {K V W : Type*} [Field K] [AddCommGroup V] [Module K V]
    [AddCommGroup W] [Module K W] [FiniteDimensional K V]
    (f : V →ₗ[K] W) :
    FiniteDimensional.finrank K (LinearMap.range f) +
    FiniteDimensional.finrank K (LinearMap.ker f) =
    FiniteDimensional.finrank K V :=
  LinearMap.finrank_range_add_finrank_ker f
```

### Subspace dimension bound

```lean
example {K V : Type*} [Field K] [AddCommGroup V] [Module K V]
    [FiniteDimensional K V] (N : Submodule K V) :
    FiniteDimensional.finrank K N ≤ FiniteDimensional.finrank K V :=
  Submodule.finrank_le N
```

### First isomorphism theorem for modules

```lean
example {R M N : Type*} [CommRing R] [AddCommGroup M] [Module R M]
    [AddCommGroup N] [Module R N] (f : M →ₗ[R] N) :
    M ⧸ LinearMap.ker f ≃ₗ[R] LinearMap.range f :=
  f.quotKerEquivRange
```
