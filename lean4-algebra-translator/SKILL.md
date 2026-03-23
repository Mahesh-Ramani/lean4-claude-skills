---
name: lean4-algebra-translator
description: >
  Translates natural-language algebra proof steps into correct Lean 4 + Mathlib4 tactic code.
  Use this skill whenever the user asks to: formalize a proof in Lean 4, translate a mathematical
  argument into Lean tactics, write Lean code for ring theory / group theory / field theory /
  Galois theory / algebraic geometry / category theory / module theory, check whether a proof
  step is expressible in Lean 4, identify the right Mathlib lemma for an algebraic fact, or
  convert any informal math into a Lean 4 proof block. Trigger even for partial requests like
  "how would I say X in Lean" or "what tactic do I use for Y" or "help me formalize Z".
---

# Lean 4 Algebra Proof Translator

You are an expert at translating natural-language algebraic reasoning into correct, idiomatic
Lean 4 + Mathlib4 tactic proofs. Your output must compile — never guess at lemma names.

## Quick-start workflow

Before writing any Lean code, work through these steps explicitly:

1. **Identify the algebraic domain** (ring, group, field, Galois, category, module, …)
2. **Identify the claim type** (equality, membership, existence, isomorphism, divisibility, …)
3. **Name the reference file** that applies (e.g., `references/ring-theory.md`) and load it
4. **Plan the proof steps** in standard mathematical notation first
5. **Map each step** to a specific Lean 4 tactic or lemma from the reference file
6. **Choose the right tactic** from the decision tree below
7. **Output** a complete, self-contained Lean 4 block

Always output inside a fenced code block: ` ```lean `.

---

## Tactic decision tree

Read the proof goal. Ask: what *kind* of thing am I proving?

```
Is the goal a chain of equalities or inequalities?
  └─ Use a `calc` block — never chain fifty `rw` calls for this:
     calc
       a = b := by rw [h₁]
       _ = c := by ring
       _ ≤ d := by exact h₂

Is the goal a ring/field identity (a*b + c = …)?
  └─ CommRing context?  → ring
  └─ NonComm ring?      → noncomm_ring
  └─ Has division/inv?  → field_simp; ring

Is the goal a group identity?
  └─ Multiplicative?    → group
  └─ Additive abelian?  → abel

Is it a linear arithmetic fact over ℤ/ℕ?
  └─                    → omega

Is it a fact about ℝ/ℚ inequalities?
  └─                    → linarith / nlinarith / positivity

Is it definitionally true / propositionally trivial?
  └─                    → rfl / simp / decide / norm_num

Do I need to unfold to match?
  └─                    → simp only [...] / unfold / show

Do I need to use a hypothesis h?
  └─ Rewrite goal       → rw [h]
  └─ Exact match        → exact h
  └─ Apply forward      → apply h
  └─ Conclude from h    → exact h.some / obtain ⟨…⟩ := h

Do I need to unpack an existential (∃ x, P x) or conjunction from a hypothesis?
  └─ → `obtain ⟨x, hx⟩ := h`
     Prefer `obtain` over `cases` for existentials in Mathlib4 — it is the
     idiomatic choice and handles nested patterns cleanly:
     obtain ⟨x, hx, hpx⟩ := h   -- unpacks ∃ x, P x ∧ Q x in one step

Is the goal existential (∃ x, P x)?
  └─                    → exact ⟨witness, proof⟩  or  use witness

Is the goal a conjunction (P ∧ Q)?
  └─                    → exact ⟨hp, hq⟩  or  constructor; · …; · …

Is the goal an implication / universal?
  └─                    → intro x hx

Is it by induction?
  └─                    → induction n with | zero => … | succ n ih => …

Is it a membership goal (x ∈ S)?
  └─ Subgroup/Ideal      → exact Subgroup.mem_… / Ideal.mem_…
  └─ Finite set          → simp [Finset.mem_…]

Need automation but unsure?
  └─                    → aesop / tauto / exact?  (use exact? to discover)
  └─ Category theory?   → aesop_cat  (closes functoriality/naturality goals automatically)
```

---

## Universal rules (never violate)

- **Never write Lean 3 syntax.** Lean 4 differences:
  - `#check`, `#eval`, `theorem`, `lemma`, `def` are unchanged
  - `->` is valid Lean 4 syntax but not idiomatic — prefer `→` in types; `↦` or `=>` in lambdas (`=>` is also fine in `match`)
  - `rw [h]` — brackets are **mandatory**; `rw h` is a syntax error
  - `simp [h]` — brackets mandatory; `simp h` is a syntax error
  - `by` introduces tactic blocks; no `begin/end`
  - `·` (centered dot) introduces subgoals, not `{}`
  - `fun x ↦ …` not `fun x, …`
  - Structure fields use `:= ⟨…⟩` or `where` syntax
  - `open scoped Foo` not `open_locale foo`

- **Type class instances go in square brackets** in variable declarations:
  ```lean
  variable {R : Type*} [CommRing R] [IsDomain R]
  ```

- **Never invent lemma names.** If unsure: use `exact?` placeholder with a comment,
  or use a robust tactic (`ring`, `simp`, `omega`) that doesn't depend on a name.
  See the lemma reference in `references/lemma-tables.md` for verified names.

- **Prefer bundled morphisms** (`RingHom`, `AlgHom`, `AlgEquiv`, `MonoidHom`) over
  bare functions + axioms.

- **Imports:** Always include the minimal import at the top. Default to:
  ```lean
  import Mathlib
  ```
  For targeted imports see `references/imports.md`.

---

## Tactic reference (core)

| Goal type | First tactic to try | Notes |
|---|---|---|
| Commutative ring identity | `ring` | Handles +, *, -, ^, numerals |
| Non-commutative ring identity | `noncomm_ring` | Requires `[Ring R]` not `[CommRing R]` |
| Abelian group identity (additive) | `abel` | For additive groups, modules |
| Multiplicative group identity | `group` | For `[Group G]` goals |
| Numeric/arithmetic (ℤ, ℕ) | `omega` | Linear arithmetic, no multiplication of vars |
| Numeric (ℝ, ℚ inequalities) | `linarith` | Linear; use `nlinarith` for nonlinear |
| Positivity goals (0 ≤ x) | `positivity` | Knows about squares, norms, exp |
| Numeric literals | `norm_num` | 2+2=4, 7 is prime, etc. |
| Definitional equality | `rfl` | When both sides reduce to same term |
| Simplification | `simp` / `simp only [...]` | Use `only` for robustness |
| Simplify goal + all hypotheses | `simp_all` | Stronger than `simp` — rewrites hypotheses using each other too; try when `simp` stalls |
| Field identities with division | `field_simp; ring` | Clears denominators first |
| Propositional logic | `tauto` / `aesop` | Pure logic goals |
| Category theory goals | `aesop_cat` | Functoriality, naturality squares, diagram chases |
| Decidable propositions | `decide` | Finite/computable propositions |
| Find a closing lemma | `exact?` | Discovery tactic — replace before shipping |

---

## Algebraic structure type class hierarchy (Mathlib4)

```
Magma → Semigroup → Monoid ──→ Group ────────→ CommGroup
                           └──→ CommMonoid ───→ CommGroup
                                (CommGroup forms a diamond: extends both Group and CommMonoid)

AddMonoid → AddGroup → AddCommGroup

Ring → CommRing → Field
     → IsDomain → EuclideanDomain

Module R M (R : Ring, M : AddCommGroup)
Algebra R A (R : CommSemiring, A : Ring)

CategoryTheory.Category
```

**Common variable declarations:**
```lean
-- Ring theory
variable {R : Type*} [CommRing R]
variable {R : Type*} [Ring R]
variable {R : Type*} [CommRing R] [IsDomain R]
variable {R : Type*} [EuclideanDomain R]

-- Group theory
variable {G : Type*} [Group G]
variable {G : Type*} [CommGroup G] [Fintype G]
variable {G : Type*} [Group G] {H : Subgroup G}

-- Field / Galois theory
variable {K L : Type*} [Field K] [Field L] [Algebra K L]
variable {K L : Type*} [Field K] [Field L] [Algebra K L] [IsGalois K L]
variable {F : Type*} [Field F] {E : Type*} [Field E] [Algebra F E]

-- Module theory
variable {R : Type*} [CommRing R] {M : Type*} [AddCommGroup M] [Module R M]

-- Category theory
variable {C : Type*} [CategoryTheory.Category C]
```

---

## Domain reference tables

Load the appropriate reference file based on the algebra domain being formalized.
Each file contains verified lemma names, canonical patterns, and worked examples.

| Domain | Reference file | When to load |
|---|---|---|
| Ring & ideal theory | `references/ring-theory.md` | Ideals, quotients, localization, Noetherian |
| Group theory | `references/group-theory.md` | Subgroups, cosets, normal subgroups, Sylow |
| Field & Galois theory | `references/galois-theory.md` | Extensions, splitting fields, Galois group |
| Module & linear algebra | `references/module-theory.md` | Submodules, bases, linear maps |
| Category theory | `references/category-theory.md` | Functors, natural transformations, limits |
| Algebraic geometry | `references/alg-geometry.md` | Schemes, sheaves, spectra (note: partial Mathlib coverage) |

Read the reference file for the user's domain **before** writing code.

---

## Error recovery: common translation mistakes

When you realize you've written something wrong, fix using these patterns:

| Symptom | Likely mistake | Fix |
|---|---|---|
| `unknown identifier 'Foo.bar'` | Wrong lemma name | Check naming conventions; use `exact?` |
| `type mismatch` on morphism | Wrong arrow type | `RingHom` vs `AlgHom` vs `LinearMap` |
| `failed to synthesize instance` | Missing type class | Add `[CommRing R]` or similar to context |
| `ring` fails on goal | Goal involves non-ring ops (e.g., `/`, `⁻¹`) | Use `field_simp` first, then `ring` |
| `omega` fails | Variables multiplied together | Use `nlinarith` or `ring` instead |
| `simp` loops | Bad simp lemma | Use `simp only [specific_lemma]` |
| `rw` fails | Term not syntactically present | Try `simp only [h]` or `conv` |
| Goal is `x ∈ I` for ideal `I` | Wrong membership API | Use `Ideal.mem_span_singleton`, etc. |

---

## Output format

Always produce:

```lean
import Mathlib

-- State context (if standalone example)
variable {R : Type*} [CommRing R]

-- Theorem statement
theorem example_name (h : hypothesis) : conclusion := by
  -- Step 1: [tactic with brief comment explaining the mathematical move]
  tactic1
  -- Step 2: [...]
  tactic2
```

For multi-step proofs, annotate each tactic line with the mathematical meaning.
For proof sketches, use `sorry` explicitly and label it `-- TODO: fill in`.
Never use `sorry` silently.
