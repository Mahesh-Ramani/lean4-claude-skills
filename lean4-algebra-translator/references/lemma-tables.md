# Verified Lemma Tables — Lean 4 / Mathlib4

All entries in this file correspond to actual Mathlib4 lemma names.
They are organized by the mathematical statement shape.

## Table 1: Ring / Field identities (use `ring` instead when possible)

When `ring` doesn't work (e.g., involves conditionals, non-ring ops), use these:

| Statement | Lemma name |
|---|---|
| `a * 0 = 0` | `mul_zero` |
| `0 * a = 0` | `zero_mul` |
| `a * 1 = a` | `mul_one` |
| `1 * a = a` | `one_mul` |
| `a + 0 = a` | `add_zero` |
| `0 + a = a` | `zero_add` |
| `a + b = b + a` | `add_comm` |
| `a * b = b * a` | `mul_comm` |
| `a * (b + c) = a*b + a*c` | `mul_add` |
| `(a + b) * c = a*c + b*c` | `add_mul` |
| `a * (b * c) = (a * b) * c` | `mul_assoc` (reversed) |
| `-(a + b) = -a + -b` | `neg_add_rev` |
| `a - b = a + (-b)` | `sub_eq_add_neg` |
| `a ^ (m + n) = a^m * a^n` | `pow_add` |
| `a ^ (m * n) = (a^m)^n` | `pow_mul` |

## Table 2: Group / Monoid lemmas

| Statement | Lemma name |
|---|---|
| `a * a⁻¹ = 1` | `mul_inv_cancel` |
| `a⁻¹ * a = 1` | `inv_mul_cancel` |
| `(a * b)⁻¹ = b⁻¹ * a⁻¹` | `mul_inv_rev` |
| `(a⁻¹)⁻¹ = a` | `inv_inv` |
| `1⁻¹ = 1` | `inv_one` |
| `orderOf g ∣ n ↔ g^n = 1` | `orderOf_dvd_iff_pow_eq_one` |
| `g ^ orderOf g = 1` | `pow_orderOf_eq_one` |
| `orderOf g ∣ Fintype.card G` | `orderOf_dvd_card` |
| `Fintype.card H * H.index = Fintype.card G` | `Subgroup.card_mul_index` |

## Table 3: Divisibility

| Statement | Lemma name |
|---|---|
| `a ∣ a` | `dvd_refl` |
| `a ∣ b → b ∣ c → a ∣ c` | `dvd_trans` |
| `a ∣ b → a ∣ b * c` | `Dvd.dvd.mul_right` |
| `a ∣ b → a ∣ c * b` | `Dvd.dvd.mul_left` |
| `a ∣ b → a ∣ c → a ∣ b + c` | `dvd_add` |
| `a ∣ b → a ∣ -b` | `dvd_neg` |
| `a * b ∣ a * c ↔ b ∣ c` (a ≠ 0) | `mul_dvd_mul_iff_left` |

## Table 4: Injectivity / surjectivity

| Statement | Lemma name |
|---|---|
| `f injective ↔ ker = {0}` (linear) | `LinearMap.injective_iff_ker_eq_bot` |
| `f injective ↔ ker = {1}` (group hom) | `MonoidHom.injective_iff_ker_eq_bot` |
| `f surjective ↔ range = ⊤` | `LinearMap.range_eq_top` |
| `Function.Injective f → f a = f b → a = b` | `Function.Injective.eq_iff` |

## Table 5: Ideal membership

| Statement | Lemma name |
|---|---|
| `x ∈ Ideal.span {a} ↔ a ∣ x` | `Ideal.mem_span_singleton` |
| `a ∈ I → a * b ∈ I` | `Ideal.mul_mem_right I b ha` |
| `a ∈ I → b ∈ I → a + b ∈ I` | `Ideal.add_mem` |
| `0 ∈ I` | `Ideal.zero_mem` |
| `I ≤ J ↔ ∀ x ∈ I, x ∈ J` | `Ideal.le_def` (via `SetLike.le_def`) |
| `I.IsPrime ↔ …` | `Ideal.isPrime_iff` |

## Table 6: Subgroup / Submodule membership

| Statement | Lemma name |
|---|---|
| `1 ∈ H` (subgroup) | `Subgroup.one_mem` |
| `h ∈ H → h⁻¹ ∈ H` | `Subgroup.inv_mem` |
| `h k ∈ H → h * k ∈ H` | `Subgroup.mul_mem` |
| `0 ∈ N` (submodule) | `Submodule.zero_mem` |
| `n m ∈ N → n + m ∈ N` | `Submodule.add_mem` |
| `n ∈ N → r • n ∈ N` | `Submodule.smul_mem` |

## Table 7: Finrank / dimension

| Statement | Lemma name |
|---|---|
| `finrank F L = finrank K L * finrank F K` | `FiniteDimensional.finrank_mul_finrank` |
| `finrank K N ≤ finrank K M` (N ≤ M) | `Submodule.finrank_le` |
| `finrank (range f) + finrank (ker f) = finrank M` | `LinearMap.finrank_range_add_finrank_ker` |
| `finrank K (M × N) = finrank K M + finrank K N` | `FiniteDimensional.finrank_prod` |
| `|Gal(K/F)| = [K:F]` | `IsGalois.card_aut_eq_finrank` |

## Table 8: Polynomial facts

| Statement | Lemma name |
|---|---|
| `minpoly F x` is irreducible | `minpoly.irreducible` |
| `aeval x (minpoly F x) = 0` | `minpoly.aeval` |
| `degree (minpoly F x) ≠ 0` | `minpoly.degree_pos` |
| `p.degree = n → p has n roots` | `Polynomial.card_roots_le_degree` |
| `p splits in K` | `Polynomial.Splits` |

## Table 9: Category theory

| Statement | Lean name |
|---|---|
| `𝟙 X ≫ f = f` | `Category.id_comp` |
| `f ≫ 𝟙 Y = f` | `Category.comp_id` |
| `(f ≫ g) ≫ h = f ≫ (g ≫ h)` | `Category.assoc` |
| `F.map (𝟙 X) = 𝟙 (F.obj X)` | `Functor.map_id` |
| `F.map (f ≫ g) = F.map f ≫ F.map g` | `Functor.map_comp` |
| `α.naturality f` (naturality square) | `NatTrans.naturality` |

---

## Naming convention cheatsheet

Mathlib4 names follow predictable patterns:

| Pattern | Rule | Example |
|---|---|---|
| `A_of_B` | A follows from B | `Prime.irreducible` (prime of_irreducible) |
| `A_iff_B` | A ↔ B | `Ideal.isPrime_iff` |
| Conclusion first | Named for what it proves | `orderOf_dvd_card` |
| Snake case for lemmas | | `mul_inv_cancel` not `mulInvCancel` |
| UpperCamelCase for types/structures | | `IsGalois`, `CommRing`, `Subgroup` |
| `mem_X` for membership | | `Ideal.mem_span_singleton` |
| `X_le_Y` for ordering | | `Submodule.finrank_le` |
| `card_X` for cardinalities | | `IsGalois.card_aut_eq_finrank` |

## Discovery tactics (use when name is uncertain)

```lean
-- Find a lemma that closes the current goal
exact?

-- Find lemmas you can apply to make progress
apply?

-- Search by type signature (in VS Code / command line)
#check @Ideal.mem_span_singleton

-- Moogle / LeanSearch: natural language search
-- → https://www.moogle.ai / https://leansearch.net
```

**When to use `exact?`:** Whenever you would otherwise guess a lemma name,
write `exact? -- looking for: [description]` as a placeholder. This is correct
behavior — the prover will find the lemma when connected to Lean.
