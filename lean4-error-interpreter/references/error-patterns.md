# Lean 4 Error Patterns — Extended Reference

Verbatim error texts with annotated diagnoses and fixes.

---

## Pattern 1 — Bundled morphism vs bare function

```
type mismatch
  myFunc
has type
  G → H : Type u
but is expected to have type
  G →* H : Type u
```

**What happened:** `myFunc` is a plain Lean function `fun g => ...`. Mathlib
expects a `MonoidHom`, which is a *structure* that also carries proofs of `map_one`
and `map_mul`.

**Fix:**
```lean
-- Option A: use `where` syntax (cleanest)
def myHom : G →* H where
  toFun    := myFunc
  map_one' := by simp [myFunc]
  map_mul' := fun x y => by simp [myFunc]

-- Option B: anonymous constructor
def myHom : G →* H :=
  { toFun    := myFunc
    map_one' := by simp [myFunc]
    map_mul' := fun x y => by simp [myFunc] }
```

---

## Pattern 2 — Wrong morphism level

```
type mismatch
  f
has type
  G →+* H : Type u        -- RingHom
but is expected to have type
  G →* H : Type u         -- MonoidHom
```

**Fix:** Coerce down: `(f : G →+* H).toMonoidHom`

---

## Pattern 3 — Missing CommRing instance

```
failed to synthesize instance
  CommRing R
```

**Context:**
```lean
-- You wrote:
theorem foo {R : Type*} (x y : R) : x * y = y * x := by ring
-- Lean can't find CommRing R because you didn't declare it.
```

**Fix:**
```lean
theorem foo {R : Type*} [CommRing R] (x y : R) : x * y = y * x := by ring
--                       ^^^^^^^^^^^^
```

---

## Pattern 4 — Instance buried under def

```
failed to synthesize instance
  Inhabited (MySet α)
```

**Context:**
```lean
def MySet (α : Type u) := α → Prop
-- Lean sees MySet α as an opaque type, not α → Prop
```

**Fix:**
```lean
instance : Inhabited (MySet α) :=
  inferInstanceAs (Inhabited (α → Prop))
```

---

## Pattern 5 — ring fails on field goal

```
tactic 'ring' failed
  goal involves division '/'
```

**Context:**
```lean
example (x y : ℝ) (hy : y ≠ 0) : (x / y) * y = x := by
  ring   -- ❌ ring doesn't handle /
```

**Fix:**
```lean
example (x y : ℝ) (hy : y ≠ 0) : (x / y) * y = x := by
  field_simp   -- rewrites / using hy ≠ 0
  ring         -- closes the polynomial goal
```

---

## Pattern 6 — omega on real-valued goal

```
omega could not close goal
⊢ 0 < x * x + 1
```

`omega` only handles `ℤ`/`ℕ` linear arithmetic. For `ℝ`:

```lean
example (x : ℝ) : 0 < x * x + 1 := by
  positivity   -- knows x^2 ≥ 0, so x^2 + 1 > 0
```

---

## Pattern 7 — simp made no progress

```
simp made no progress
```

**Likely cause:** The lemma you expected is not tagged `@[simp]`, or the term doesn't
match the LHS syntactically.

**Diagnostic workflow:**
```lean
-- Step 1: ask simp what it can do
simp?
-- Step 2: be explicit
simp only [Nat.add_zero, Nat.zero_add]
-- Step 3: check the lemma fires manually
#check @Nat.add_zero   -- confirm the statement
rw [Nat.add_zero]      -- try rewriting directly
```

---

## Pattern 8 — type mismatch from universe confusion

```
type mismatch
  f
has type
  F A : Type (u+1)
but is expected to have type
  F A : Type u
```

**Fix options:**
1. Use `Type*` (auto-bound universe polymorphism) in your variable declarations
2. Explicitly match universe levels with `{u : Level}` annotations
3. Use `ULift` to change universes: `ULift.{v} A`

---

## Pattern 9 — unknown identifier after missing import

```
unknown identifier 'ℝ'
-- or
unknown identifier 'Real.sqrt'
```

**Fix:**
```lean
import Mathlib.Data.Real.Basic         -- for ℝ, Real
import Mathlib.Analysis.SpecialFunctions.Pow.Real  -- for Real.sqrt, rpow
-- Or just:
import Mathlib  -- imports everything (slower compile)
```

---

## Pattern 10 — application type mismatch with induction

```
application type mismatch
  @Fin.instOfNat n inst✝
argument
  inst✝ has type NeZero n✝
but function has type
  [inst : NeZero n] → {i : Nat} → OfNat (Fin n) i
```

**What happened:** The `induction` tactic changed `n` to a new variable `n✝`, but
still passed the original `NeZero n` instance to `Fin.instOfNat n✝`. This is a known
Lean 4 elaboration subtlety.

**Fix:**
```lean
-- Revert the instance-dependent variable before induction
intro h
revert h
induction n with
| zero => intro h; ...
| succ n ih => intro h; ...
```

---

## Pattern 11 — `rw` fails silently or errors

```
tactic 'rewrite' failed, did not find instance of the pattern in the goal
```

**Causes:**
- The term to rewrite is not syntactically present (maybe hidden under a definition)
- Implicit arguments differ

**Fixes:**
```lean
-- Try simp only instead:
simp only [h]

-- Or unfold first, then rw:
unfold myDef
rw [h]

-- Or use conv to focus on a subterm:
conv_lhs => rw [h]

-- Or change to put the goal in the right form first:
change expr_matching_lhs_of_h
rw [h]
```

---

## Pattern 12 — `exact?` / `apply?` finds nothing

This is not an error but a sign you need a different approach:

1. Check your goal with `#check` — maybe it's not the right statement
2. Try `simp?` — sometimes the goal is provable by simp
3. Search Mathlib docs manually: `leanprover-community.github.io/mathlib4_docs`
   — use the search box with keywords from your goal
4. Ask on the Lean 4 Zulip: `leanprover.zulipchat.com`
