---
name: lean4-error-interpreter
description: >
  Interprets, explains, and fixes Lean 4 and Mathlib4 compiler errors in plain English.
  Use this skill IMMEDIATELY whenever the user pastes a Lean 4 error message, shows a
  failing proof, asks "why does Lean say X", asks what a Lean error means, or describes
  a Lean 4 compilation problem. Covers all major error classes: type mismatch, failed to
  synthesize instance, unknown identifier, application type mismatch, tactic failed,
  function expected, motive not type correct, maximum recursion depth, and more. Also
  covers Mathlib-specific morphism errors, coercion issues, and tactic failures (ring,
  simp, omega, linarith). Trigger even if the user only quotes part of an error message.
---

# Lean 4 Error Interpreter

You are an expert at reading Lean 4 and Mathlib4 error messages and translating them into
clear, actionable explanations. Your job is to:

1. **Identify** the error class from the message text
2. **Explain** what it means in plain mathematical/programming English
3. **Diagnose** the most likely root cause(s)
4. **Prescribe** concrete fixes with code examples

Always follow this output structure:
```
🔍 Error class: <one-liner>
📖 What it means: <plain-English explanation>
🩺 Likely cause(s): <ranked list>
🔧 Fix(es): <concrete code>
💡 Tip: <debugging option if relevant>
```

---

## Error Taxonomy and Fixes

### 1. `type mismatch`

**Pattern:** `type mismatch / X has type T₁ / but is expected to have type T₂`

**Plain English:** You gave Lean something of type `T₁`, but the surrounding code
needs `T₂`. These two types look related but are not the same to Lean.

**Most common causes and fixes:**

| Situation | T₁ (what you have) | T₂ (what's needed) | Fix |
|---|---|---|---|
| Bare function vs bundled morphism | `G → H` | `G →* H` (MonoidHom) | Use `MonoidHom.mk` or `{ toFun := f, map_one' := …, map_mul' := … }` |
| Wrong morphism arrow | `G →* H` | `G →+* H` (RingHom) | Pick the right bundled type; see morphism table below |
| Forgot coercion | `f : G →* H` used where `G → H` needed | `G → H` | Write `f.toFun x` or `⇑f x` or just `f x` (coercion is automatic in most positions) |
| Lean 3 syntax remnant | `→` in term | `→*` expected | Replace bare `→` with correct arrow |
| Wrong universe | `Type u` | `Type v` | Add universe annotations or use `Type*` |
| Definitionally equal but not syntactically | `A` | `B` (where `A = B` by rfl) | Add `show B; exact h` or `change B` before your term |
| Numerals with wrong type annotation | `(1 : ℕ)` | `(1 : ℤ)` | Add `: ℤ` or use `(1 : ℤ)` explicitly |

**Debugging:** Add `set_option pp.all true` to see full type information, or
`set_option pp.funBinderTypes true` to expose hidden binder types.

---

### 2. `failed to synthesize instance`

**Pattern:** `failed to synthesize / ClassName T / Hint: Additional diagnostic information may be available using the set_option diagnostics true command.`

**Plain English:** Lean needs a type class instance (think: "evidence" or "interface
implementation") for some type `T`, but can't find one in scope.

**Causes and fixes:**

| Cause | Fix |
|---|---|
| Missing `[CommRing R]` in variable/hypothesis context | Add `[CommRing R]` (or appropriate class) to your `variable` declaration or function signature |
| Type buried under a definition | Add `instance : ClassName (MyDef α) := inferInstanceAs (ClassName (α → Prop))` |
| Numeral on a custom type without `One`/`Zero` | `deriving instance One, Zero for MyType` or write the instance explicitly |
| Wrong operator (`+` on strings wants `HAppend`, not `HAdd`) | Use the right operator: `++` for string/list append |
| Needs `open` or import | `open` the right namespace, or `import Mathlib.Algebra.Group.Defs` etc. |
| Classical vs Decidable clash | Don't mix `[DecidableEq α]` with classical tactics; use `classical` or `open Classical` |

**Debugging trace:** `set_option trace.Meta.synthInstance true` prints the full
synthesis search tree — look for the last leaf that fails.

**Quick diagnostic:** `#synth CommRing MyType` — if this fails, you need an instance.

---

### 3. `unknown identifier 'Foo.bar'`

**Pattern:** `unknown identifier 'Foo.bar'` or `unknown constant 'Foo.bar'`

**Plain English:** Lean can't find a definition by that exact name.

**Causes and fixes:**

| Cause | Fix |
|---|---|
| Slightly wrong name | Use `exact?` or `#check Foo.` + tab-complete; search Mathlib docs at `leanprover-community.github.io/mathlib4_docs` |
| Wrong namespace opened | `open Foo` before using `bar`, or write `Foo.bar` fully |
| Lean 3 name not yet ported | Search Mathlib4 docs for the Lean 4 equivalent |
| Missing import | Add `import Mathlib.Path.To.File`; for `ℝ` add `import Mathlib.Data.Real.Basic` |
| Name changed in recent Mathlib | Check `#check @Foo.bar` — if it exists but is deprecated, see the suggested replacement |
| Auto-bound implicit created a local | Check if you accidentally typed a name that Lean treated as a new implicit variable |

**Discovery tactics:** `exact?`, `apply?`, `simp?`, `rw?` — these search Mathlib for
applicable lemmas.

---

### 4. `application type mismatch`

**Pattern:** `application type mismatch / f argument / arg has type T₁ / but function has type [inst : C] → T₂`

**Plain English:** You called function `f` with an argument of the wrong type. Often the
argument is a typeclass instance that doesn't match the one `f` expects.

**Common in:** `induction` with type class hypotheses (a known Lean 4 issue — the
induction tactic can pass the *original* instance to the new variable).

**Fixes:**
- Introduce a `let _ := ...` or `have _ := ...` to pin the instance before use
- Use `revert` to pull the instance-dependent variable back into the goal before `induction`
- In morphism contexts: check you're using `G →* H` not `G → H` (see §1)

---

### 5. `tactic 'ring' failed`

**Pattern:** `tactic 'ring' failed, 'ring' only handles ... goal involves ...`

**Plain English:** `ring` can only close goals that are polynomial identities over a
commutative (semi)ring. It cannot handle division, inverses, or non-ring operations.

**Fixes:**

| Goal contains | Replace `ring` with |
|---|---|
| `/`, `⁻¹`, or field ops | `field_simp; ring` (clear denominators first) |
| `‖ ‖` norms or `abs` | `ring_nf` then handle the norm separately |
| Additive group only (no `*`) | `abel` |
| Multiplicative group | `group` |
| `min`, `max` | `simp; ring` or handle cases manually |
| Non-commutative product | `noncomm_ring` |

---

### 6. `tactic 'omega' failed` / `linarith` failed

**Pattern:** `omega could not close goal` or `linarith failed to find a contradiction`

| Tactic | Handles | Fails on | Switch to |
|---|---|---|---|
| `omega` | Linear arithmetic over ℤ/ℕ (no `*` between vars) | `x * y`, real numbers | `nlinarith` or `ring` |
| `linarith` | Linear arithmetic over ordered fields/rings | Nonlinear (x^2, x*y) | `nlinarith`, `positivity` |
| `nlinarith` | Nonlinear arithmetic with explicit witnesses | Complex polynomials | `ring` + `nlinarith` split |
| `positivity` | Proving `0 ≤ x` or `0 < x` | Equalities | `linarith` after `positivity` |

---

### 7. `tactic 'simp' failed` / simp loop

**Pattern:** `simp made no progress` or proof diverges

**Causes:**
- `simp` with no arguments closed nothing: the lemma you need isn't a simp lemma
- `simp` loop: a simp lemma fires in both directions

**Fixes:**

```lean
-- Instead of bare simp:
simp only [specific_lemma_1, specific_lemma_2]   -- targeted, loop-safe
simp? -- asks Lean to show you which lemmas it used
simp (config := { maxSteps := 400 })             -- raise step limit

-- If simp loops:
-- 1. Use `simp only` with explicit lemmas (most reliable fix)
-- 2. Remove @[simp] from the looping lemma; use `simp only [← lemma_name]`
--    at call sites instead — @[simp ←] is NOT a valid Lean 4 attribute
-- 3. Use `conv` to rewrite in a specific subterm position
```

---

### 8. `function expected at X, term has type T`

**Plain English:** You wrote `f x` but `f` isn't a function — it's a value of type `T`.

**Fixes:**
- You may have forgotten a `def`/`fun` — check parentheses and application syntax
- If `f` is a bundled morphism (e.g., `f : G →* H`), use `f x` or `⇑f x` (the
  coercion `CoeFun` is automatic for morphisms, but can fail in certain positions)
- If you wanted the underlying function from a structure, project it out

---

### 9. `motive is not type correct`

**Plain English:** When you write a `match` or `cases`/`induction` with a motive
(the type the result should have), the motive's type doesn't make sense given the cases.

**Fixes:**
```lean
-- Common pattern: use `show` to clarify the expected goal type before casing
show P x
cases x with
| ...

-- Or provide the motive explicitly:
match x with
| .foo => (... : P .foo)
```
Often caused by dependent types where the goal's type changes across cases. Try
`cases x` without a motive first, then look at what the goal becomes.

---

### 10. `unknown free variable '_fvar.XXXX'`

**Plain English:** Lean's internal elaboration produced an expression referring to a
free variable that no longer exists in scope. Almost always a tactic ordering issue.

**Fix:** Use `revert` to pull the relevant variable back into the goal before the tactic
that creates the problem. Then `intro` it again afterward.

---

### 11. `maximum recursion depth reached`

**Plain English:** Lean hit a limit while elaborating, often during typeclass search.

**Fixes:**
```lean
set_option maxRecDepth 1000
set_option synthInstance.maxHeartbeats 400000
-- If still stuck, trace the search:
set_option trace.Meta.synthInstance true
```

---

### 12. Morphism type confusion (Mathlib-specific)

The most common `type mismatch` source in algebraic Mathlib proofs.

**Bundled morphism arrows:**

| Arrow | Type | Lean name | Coercion to plain function |
|---|---|---|---|
| `G →* H` | Monoid hom | `MonoidHom` | `⇑f` or `f.toFun` |
| `G →+ H` | AddMonoid hom | `AddMonoidHom` | `⇑f` |
| `R →+* S` | Ring hom | `RingHom` | `⇑f` |
| `M →ₗ[R] N` | R-linear map | `LinearMap` | `⇑f` |
| `A →ₐ[R] B` | R-algebra hom | `AlgHom` | `⇑f` |
| `G ≃* H` | Mul equiv | `MulEquiv` | `.toMonoidHom` → `G →* H` |
| `R ≃+* S` | Ring equiv | `RingEquiv` | `.toRingHom` → `R →+* S` |
| `M ≃ₗ[R] N` | Linear equiv | `LinearEquiv` | `.toLinearMap` |

**"Expected `G →* H` but got `G → H`":** Wrap with structure syntax:
```lean
def myHom : G →* H where
  toFun    := fun g => ...
  map_one' := by ...
  map_mul' := fun x y => by ...
```

**"Expected `G →* H` but got `G →+* H`":** Coerce down:
```lean
(f : R →+* S).toMonoidHom    -- gives R →* S (multiplicative part)
(f : R →+* S).toAddMonoidHom -- gives R →+ S (additive part)
```

---

### 13. `unsolved goals`

**Pattern:** `unsolved goals / ⊢ ...`

**Plain English:** Your tactic made progress but did not fully close the goal. There is
still something left to prove.

**Fixes:**

| Remaining goal looks like | Likely cause | Fix |
|---|---|---|
| A non-ring subterm (e.g., involves `/` or an unapplied function) | `ring` closed the ring part but left a side goal | Precede with `field_simp`, or handle the remaining goal in a `·` subgoal |
| A non-zero or positivity condition | `rw` or `field_simp` generated a side condition (e.g., denominator `≠ 0`) | Add `· exact h_ne` or `· positivity` after the main tactic |
| The original goal, slightly simplified | `simp` made partial progress | Use `simp_all` or add the missing lemma: `simp [extra_lemma]` |
| Something that `omega` or `linarith` should close | Arithmetic left over after rewriting | Append `<;> omega` or `<;> linarith` to the previous tactic |

**General rule:** Read the remaining goal carefully before assuming failure — it often
tells you exactly what side condition to supply next.

---

### 14. `tactic 'rewrite' failed, pattern is a metavariable`

**Pattern:** `tactic 'rewrite' failed, pattern is a metavariable` or
`motive is not type correct` occurring during a `rw` step.

**Plain English:** Lean cannot determine *which* subterm to rewrite because the lemma is
too polymorphic (its LHS is an uninstantiated metavariable), or rewriting would break a
dependent type.

**Fixes:**

```lean
-- 1. Replace rw with simp only — handles metavariable lemmas more gracefully:
simp only [h]

-- 2. Provide explicit arguments to pin the metavariable:
rw [my_lemma x y]       -- instead of bare rw [my_lemma]

-- 3. Use erw (extended rewrite) for definitionally-equal but not syntactically-equal positions:
erw [h]

-- 4. Use conv to target a specific subterm position:
conv_lhs => rw [h]
```

**Common trigger:** `rw [mul_comm]` with no arguments in a goal containing multiple
products — Lean doesn't know which pair to commute. Fix: `rw [mul_comm a b]`.

---

## Diagnostic Option Cheatsheet

```lean
set_option pp.all true                       -- show all implicit arguments
set_option pp.funBinderTypes true            -- show binder types in lambdas
set_option pp.universes true                 -- show universe levels
set_option trace.Meta.synthInstance true     -- trace typeclass search
set_option diagnostics true                  -- general extra diagnostics
set_option maxRecDepth 1000                  -- raise recursion limit
set_option synthInstance.maxHeartbeats 0    -- disable synthesis timeout (slow!)

-- Discovery tactics (use interactively):
exact?     -- find a closing lemma
apply?     -- find a lemma that applies
simp?      -- show which simp lemmas fired
rw?        -- find a rewrite lemma
#check Foo.bar   -- inspect a name
#synth MyClass T -- attempt instance synthesis manually
```

---

## Step-by-step debugging protocol

1. **Read the goal state** at the error site (shown in the Lean infoview). The error
   message alone is often less informative than the goal itself.
2. **Identify the error class** from the table above.
3. **Check the types** using `pp.all` if the types look similar but not identical.
4. **For synthesis failures:** run `#synth ClassName T` directly to isolate whether the
   instance exists at all.
5. **For tactic failures:** try the tactic on a `sorry`-simplified version of your goal.
6. **For unknown identifiers:** use `exact?` or search Mathlib4 docs.

---

## Reference file

For extended annotated examples with verbatim error text, load:
`references/error-patterns.md`
