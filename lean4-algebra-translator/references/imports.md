# Mathlib4 Import Guide

Use `import Mathlib` as the default. For performance-sensitive use, use targeted imports below.

## Targeted imports by domain

### Ring theory
```lean
import Mathlib.RingTheory.Ideal.Basic
import Mathlib.RingTheory.Ideal.Quotient.Basic
import Mathlib.RingTheory.Ideal.Operations
import Mathlib.RingTheory.Noetherian.Basic
import Mathlib.RingTheory.PrincipalIdealDomain
import Mathlib.RingTheory.UniqueFactorizationDomain
import Mathlib.RingTheory.Localization.Basic
import Mathlib.RingTheory.Polynomial.Basic
```

### Group theory
```lean
import Mathlib.GroupTheory.Subgroup.Basic
import Mathlib.GroupTheory.QuotientGroup.Basic
import Mathlib.GroupTheory.GroupAction.Basic
import Mathlib.GroupTheory.Sylow
import Mathlib.GroupTheory.OrderOfElement
import Mathlib.GroupTheory.Coset.Basic
import Mathlib.GroupTheory.FiniteGroup
```

### Field and Galois theory
```lean
import Mathlib.FieldTheory.Galois.Basic
import Mathlib.FieldTheory.SplittingField.Construction
import Mathlib.FieldTheory.Adjoin
import Mathlib.FieldTheory.IntermediateField.Basic
import Mathlib.FieldTheory.Separable
import Mathlib.FieldTheory.PerfectClosure
import Mathlib.FieldTheory.IsAlgClosed.Basic
```

### Linear algebra / module theory
```lean
import Mathlib.LinearAlgebra.Basic
import Mathlib.LinearAlgebra.Basis.Basic
import Mathlib.LinearAlgebra.Dimension.Basic
import Mathlib.LinearAlgebra.TensorProduct.Basic
import Mathlib.LinearAlgebra.Quotient
import Mathlib.LinearAlgebra.FiniteDimensional.Basic
```

### Category theory
```lean
import Mathlib.CategoryTheory.Category.Basic
import Mathlib.CategoryTheory.Functor.Basic
import Mathlib.CategoryTheory.NatTrans
import Mathlib.CategoryTheory.Limits.HasLimits
import Mathlib.CategoryTheory.Limits.Shapes.Products
import Mathlib.CategoryTheory.Adjunction.Basic
import Mathlib.CategoryTheory.Equivalence
import Mathlib.CategoryTheory.Abelian.Basic
```

### Algebraic geometry
```lean
import Mathlib.AlgebraicGeometry.Scheme
import Mathlib.AlgebraicGeometry.Morphisms.Basic
import Mathlib.AlgebraicGeometry.StructureSheaf
import Mathlib.AlgebraicGeometry.AffineScheme
```

### Number theory
```lean
import Mathlib.NumberTheory.LegendreSymbol.QuadraticReciprocity
import Mathlib.NumberTheory.Cyclotomic.Basic
import Mathlib.RingTheory.DedekindDomain.Basic
```

### Commutative algebra
```lean
import Mathlib.RingTheory.Noetherian.Basic
import Mathlib.RingTheory.DedekindDomain.Basic
import Mathlib.RingTheory.IntegralClosure.Basic
import Mathlib.RingTheory.Polynomial.Cyclotomic.Basic
```

## Tactic imports (usually included automatically with domain imports)

```lean
import Mathlib.Tactic           -- all tactics
import Mathlib.Tactic.Ring      -- ring, ring_nf
import Mathlib.Tactic.Linarith  -- linarith, nlinarith
import Mathlib.Tactic.Omega     -- omega
import Mathlib.Tactic.Positivity -- positivity
import Mathlib.Tactic.NormNum   -- norm_num
import Mathlib.Tactic.FieldSimp -- field_simp
import Mathlib.Tactic.Abel      -- abel
import Mathlib.Tactic.Group     -- group
import Mathlib.Tactic.Ext       -- ext
import Mathlib.Tactic.Contrapose -- contrapose
```

## Universe polymorphism note

Mathlib uses universe polymorphism extensively. Most types are declared with `Type*` (automatically inferred universe) or explicit `Type u`. When declaring new types, use:

```lean
variable {α : Type*}   -- universe-polymorphic, preferred
variable {R : Type u}  -- explicit universe u, for multi-universe statements
```

The `*` in `Type*` is sugar for the universe being automatically inferred — it avoids having to explicitly quantify over universe levels in most use cases.
