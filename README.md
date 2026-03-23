# lean4-claude-skills

A pair of Claude skills for working with Lean 4 and Mathlib4. These skills give Claude structured, reference-backed knowledge for two complementary tasks: interpreting compiler errors and translating mathematical proofs into tactic code.

---

## Skills

### `lean4-error-interpreter`

Interprets Lean 4 and Mathlib4 compiler errors and returns plain-English explanations with ranked causes and concrete fixes.

**Covers 14 error classes:**
- `type mismatch` — including bundled morphism confusion and universe mismatches
- `failed to synthesize instance` — missing type class instances
- `unknown identifier` — wrong names, missing imports, namespace issues
- `application type mismatch` — induction + instance elaboration bugs
- `tactic 'ring' failed` — when to use `field_simp`, `abel`, `group`, `noncomm_ring`
- `omega` / `linarith` failed — arithmetic tactic selection guide
- `simp` failed / simp loops — targeted fixes including `simp only` and `conv`
- `function expected` — coercion and application issues
- `motive is not type correct` — dependent type case-split errors
- `unknown free variable` — tactic ordering / scoping issues
- `maximum recursion depth` — typeclass search limit fixes
- Morphism type confusion — full bundled morphism arrow reference table
- `unsolved goals` — reading residual goals and supplying side conditions
- `rewrite failed, pattern is a metavariable` — when to use `erw`, `simp only`, `conv`, explicit arguments

**Reference file:** `lean4-error-interpreter/references/error-patterns.md` — verbatim error texts with annotated diagnoses across 12 extended patterns.

---

### `lean4-algebra-translator`

Translates natural-language algebraic proof steps into correct, idiomatic Lean 4 + Mathlib4 tactic code.

**Workflow:** Before writing any code, the skill instructs Claude to identify the algebraic domain, name the applicable reference file, plan the proof in standard mathematical notation, and map each step to a tactic — before outputting any Lean.

**Tactic decision tree covers:**
- Chained equalities / inequalities → `calc` blocks
- Ring, field, group, abelian group identities
- Linear and nonlinear arithmetic
- Existential unpacking → `obtain`
- Induction, membership goals, propositional logic
- Category theory → `aesop_cat`
- Automation fallback → `aesop`, `exact?`

**Tactic reference table covers:** `ring`, `noncomm_ring`, `abel`, `group`, `omega`, `linarith`, `nlinarith`, `positivity`, `norm_num`, `rfl`, `simp`, `simp_all`, `field_simp`, `tauto`, `aesop`, `aesop_cat`, `decide`, `exact?`

**Reference files:**

| File | Coverage |
|---|---|
| `references/ring-theory.md` | Ideals, quotients, localization, Noetherian rings |
| `references/group-theory.md` | Subgroups, cosets, normal subgroups, Sylow theorems |
| `references/galois-theory.md` | Field extensions, splitting fields, Galois group |
| `references/module-theory.md` | Submodules, bases, linear maps |
| `references/category-theory.md` | Functors, natural transformations, limits |
| `references/alg-geometry.md` | Schemes, sheaves, spectra (partial Mathlib coverage) |
| `references/lemma-tables.md` | Verified lemma names across 9 domains with naming conventions |
| `references/imports.md` | Targeted import paths for common Mathlib modules |

---

## Repo Structure

```
lean4-error-interpreter/
├── SKILL.md
└── references/
    └── error-patterns.md

lean4-algebra-translator/
├── SKILL.md
└── references/
    ├── alg-geometry.md
    ├── category-theory.md
    ├── galois-theory.md
    ├── group-theory.md
    ├── imports.md
    ├── lemma-tables.md
    ├── module-theory.md
    └── ring-theory.md
```

---

## Usage

These are skills for [Claude](https://claude.ai). To use them, place the skill folders in your Claude skills directory and they will be triggered automatically on relevant queries:

- **Error interpreter** triggers on any pasted Lean 4 error message, failing proof, or question about what a Lean error means.
- **Algebra translator** triggers on any request to formalize a proof, write Lean code for an algebraic domain, or identify the right Mathlib lemma for a mathematical fact.

---

## Notes

- All tactic and lemma names target **Lean 4 + Mathlib4**. No Lean 3 syntax is used.
- Lemma names in `references/lemma-tables.md` are verified against Mathlib4.
- For lemma discovery when names are uncertain, the skills consistently direct to `exact?`, `apply?`, and [Mathlib4 docs](https://leanprover-community.github.io/mathlib4_docs).
- Community support: [Lean 4 Zulip](https://leanprover.zulipchat.com)
