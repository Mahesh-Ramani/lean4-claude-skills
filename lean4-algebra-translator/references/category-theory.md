# Category Theory — Lean 4 / Mathlib4 Reference

## Imports

```lean
import Mathlib.CategoryTheory.Category.Basic
import Mathlib.CategoryTheory.Functor.Basic
import Mathlib.CategoryTheory.NatTrans
import Mathlib.CategoryTheory.Limits.HasLimits
import Mathlib.CategoryTheory.Adjunction.Basic
-- or: import Mathlib
```

**Important:** Always `open CategoryTheory` at the top of category theory proofs.

```lean
open CategoryTheory
```

---

## Categories

### Basic setup

```lean
variable {C : Type*} [Category C]
-- or equivalently: [CategoryTheory.Category C]

-- Objects: C
-- Morphisms: f : X ⟶ Y   (unicode arrow, typed as \hom or \-->)
-- Identity: 𝟙 X  (typed as \b1)
-- Composition: f ≫ g  (typed as \gg) — note: LEFT to RIGHT
```

### Key axioms / lemmas

```lean
Category.id_comp          -- 𝟙 X ≫ f = f
Category.comp_id          -- f ≫ 𝟙 Y = f
Category.assoc            -- (f ≫ g) ≫ h = f ≫ (g ≫ h)
```

### Isomorphisms

```lean
-- Type
e : X ≅ Y                -- Iso X Y

-- Components
e.hom : X ⟶ Y
e.inv : Y ⟶ X
e.hom_inv_id             -- e.hom ≫ e.inv = 𝟙 X
e.inv_hom_id             -- e.inv ≫ e.hom = 𝟙 Y

-- Constructors
Iso.mk (hom := f) (inv := g) (hom_inv_id := ...) (inv_hom_id := ...)
```

---

## Functors

```lean
-- Type
F : C ⥤ D               -- Functor C D  (typed as \functor or ⥤)

-- Components
F.obj : C → D            -- object map
F.map : (X ⟶ Y) → (F.obj X ⟶ F.obj Y)  -- morphism map

-- Axioms (automatically provided)
F.map_id                 -- F.map (𝟙 X) = 𝟙 (F.obj X)
F.map_comp               -- F.map (f ≫ g) = F.map f ≫ F.map g
```

### Natural transformations

```lean
-- Type
α : F ⟹ G               -- NatTrans F G

-- Components
α.app (X : C) : F.obj X ⟶ G.obj X

-- Naturality square: for f : X ⟶ Y
-- F.map f ≫ α.app Y = α.app X ≫ G.map f
α.naturality f

-- Vertical composition
NatTrans.vcomp α β       -- α : F ⟹ G, β : G ⟹ H  →  F ⟹ H
-- Horizontal composition: whiskering
F.whiskerLeft α          -- whiskerLeft F α : F ≫ G ⟹ F ≫ H (for α : G ⟹ H)
α.whiskerRight H         -- whiskerRight α H : F ≫ H ⟹ G ≫ H (for α : F ⟹ G)
```

### Natural isomorphisms

```lean
-- Type
α : F ≅ G               -- Iso F G in the functor category
-- Components are natural transformations

-- Key: isomorphism in functor category = pointwise isomorphism
NatIso.isIso_app_of_isIso   -- if α is NatIso, each α.app X is iso
NatIso.ofComponents         -- build NatIso from pointwise isos
```

---

## Limits and colimits

```lean
import Mathlib.CategoryTheory.Limits.HasLimits

open CategoryTheory.Limits

-- The limit of a diagram F : J ⥤ C
[HasLimit F]
limit F                       -- the limit object
limit.π F j                   -- projection to component j
limit.lift F s                -- unique map from a cone s

-- HasLimitsOfShape (all limits of shape J exist)
[HasLimitsOfShape J C]

-- Common limits
[HasTerminal C]               -- terminal object (limit of empty diagram)
⊤_ C                          -- the terminal object

[HasBinaryProducts C]         -- pairwise products
X ⨯ Y                         -- the product

[HasPullbacks C]              -- pullbacks
pullback f g                  -- the pullback of f, g

-- Colimits
[HasColimit F]
colimit F
colimit.ι F j                 -- coprojection
colimit.desc F s              -- unique map to a cocone s

[HasInitial C]                -- initial object
⊥_ C

[HasCoproducts C]             -- coproducts / coproduct
X ⨿ Y
```

---

## Adjunctions

```lean
import Mathlib.CategoryTheory.Adjunction.Basic

-- An adjunction F ⊣ G
adj : F ⊣ G               -- Adjunction F G

-- The unit and counit
adj.unit : 𝟭 C ⟹ G.comp F
adj.counit : F.comp G ⟹ 𝟭 D

-- Hom set bijection
adj.homEquiv X Y : (F.obj X ⟶ Y) ≃ (X ⟶ G.obj Y)
```

---

## Equivalences

```lean
-- Equivalence of categories
e : C ≌ D              -- CategoryTheory.Equivalence C D

-- Components
e.functor : C ⥤ D
e.inverse : D ⥤ C
e.unitIso : 𝟭 C ≅ e.functor ⋙ e.inverse
e.counitIso : e.inverse ⋙ e.functor ≅ 𝟭 D

-- Fully faithful + essentially surjective ↔ equivalence
CategoryTheory.Functor.isEquivalence_iff
```

---

## Common categories in Mathlib

```lean
-- Category of types
Type            -- has Category instance (morphisms = functions)

-- Category of groups
Grp             -- GroupCat or Grp

-- Category of rings
RingCat

-- Category of R-modules
ModuleCat R     -- ModuleCat.{v} R for universe v

-- Category of commutative rings
CommRingCat

-- Opposite category
Cᵒᵖ             -- CategoryTheory.Opposite C
op X            -- X as an object of Cᵒᵖ
unop X          -- back to C
```

---

## Worked examples

### Functoriality of a ring hom

```lean
open CategoryTheory

-- RingCat as a category with ring homomorphisms as morphisms
example (f : R →+* S) : CommRingCat.of R ⟶ CommRingCat.of S :=
  RingHom.toAlgebra f |>.toLinearMap -- or:
  CommRingCat.ofHom f
```

### Natural transformation check (naturality square)

```lean
-- Typically proved by: ext X; simp [...]
example {F G : C ⥤ D} (α : F ⟹ G) {X Y : C} (f : X ⟶ Y) :
    F.map f ≫ α.app Y = α.app X ≫ G.map f :=
  α.naturality f
```

### Constructing a limit cone

```lean
-- For binary products in a category with products
example [HasBinaryProducts C] (X Y : C) : X ⨯ Y ≅ Y ⨯ X :=
  Limits.prod.braiding X Y
```
