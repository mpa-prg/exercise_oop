# Object-Oriented Programming in Python

Instead of implementing one algorithm end-to-end, you will incrementally
build a small set of **classes** that model genomic annotation records
(exons, genes, and variants) and their positions on a chromosome.

Structure of the session:
1. **Classes and objects** (theory) → **Task 1**
2. **Inheritance and polymorphism** (theory) → **Task 2**
3. **Task 3** - a longer task you work through
   independently, combining everything from Tasks 1 and 2.

Write your own code from scratch in a single file, `oop_exercise.py`. 
Fork this repository to your own GitHub account, clone your fork, and
commit and push your work to it as you go (e.g. one commit per finished
part).

---

## Task 1: `GenomicFeature` class

Implement a class `GenomicFeature` representing any annotated region on a
chromosome.

**Constructor** `__init__(self, chromosome, start, end, strand)`:
- `chromosome`: a string, e.g. `"chr1"`.
- `start`, `end`: positive integers, 1-based, inclusive; **must** satisfy
  `start <= end`.
- `strand`: must be `"+"` or `"-"`.
- If any of these constraints is violated, raise a `ValueError` with a short
  message explaining what is wrong.
- Store all four values as instance attributes.

**Methods:**
- `length(self)` → `int`: the number of bases covered, i.e. `end - start + 1`.
- `overlaps(self, other)` → `bool`: `True` if `self` and `other` are on the
  same chromosome and their `[start, end]` ranges intersect (touching at a
  single base counts as overlapping; strand is ignored for this check).
- `describe(self)` → `str`: a human-readable one-line summary, e.g.
  `"GenomicFeature chr1:1000-5000(+)"`. You are free to choose the exact
  wording, but it must include the class name (see hint below), chromosome,
  start, end and strand.

**Hint:** to make `describe()` automatically say the right class name
(`"GenomicFeature"`, later `"Exon"`, `"Gene"`, `"Variant"`) without
repeating it in every subclass, you can use `type(self).__name__` inside
the method.

Write a short `if __name__ == "__main__":` block that does
this and prints the results:
```python
a = GenomicFeature("chr1", 1000, 5000, "+")
b = GenomicFeature("chr1", 4800, 6000, "+")
c = GenomicFeature("chr2", 1000, 5000, "+")

print(a.describe())     # GenomicFeature chr1:1000-5000(+)
print(a.length())        # 4001
print(a.overlaps(b))     # True  (4800-5000 shared)
print(a.overlaps(c))     # False (different chromosome)
GenomicFeature("chr1", 5000, 1000, "+")  # should raise ValueError
```

---

## Task 2: a first subclass, `Exon`

Create a subclass `Exon(GenomicFeature)`:
- Extra attribute: `exon_number` (int).
- Must call `super().__init__(...)` to reuse the validation already written
  in `GenomicFeature` - do not re-implement it.
- Override `describe()` to also include the exon number, e.g.
  `"Exon chr1:1000-1200(+) exon #1"`.

Build a small list mixing plain `GenomicFeature` and
`Exon` objects, and call `describe()` on each in a loop:
```python
features = [
    GenomicFeature("chr1", 1000, 5000, "+"),
    Exon("chr1", 1000, 1200, "+", 1),
    Exon("chr1", 3000, 3300, "+", 2),
]
for feature in features:
    print(feature.describe())
```
Each object should print its *own* version of `describe()` - that's
polymorphism: the loop only ever calls `feature.describe()`, without
checking what type `feature` actually is.

---

## Task 3: `Exon`, `Gene`, `Variant` classes and a report

This is the main, longer task — work through it independently, combining
everything from Task 1 and 2.

Create (or extend, if you keep your `Exon` from Task 2) three
subclasses of `GenomicFeature`:

- **`Gene(GenomicFeature)`**
  - Extra attribute: `name` (string), plus an empty list `self.exons = []`.
  - Method `add_exon(self, exon)`: appends an `Exon` instance to `self.exons`.
  - Method `total_exon_length(self)` → `int`: the sum of `length()` over all
    exons added so far (use the `Exon` objects' own `length()` — don't
    recompute it from scratch).
  - Override `describe()` to also include the gene name and the number of
    exons added so far, e.g. `"Gene GENE_A chr1:1000-5000(+), 3 exon(s)"`.

- **`Exon(GenomicFeature)`** — extra attribute `exon_number` (int); override
  `describe()` to also include the exon number (same as Task 2).

- **`Variant(GenomicFeature)`**
  - Extra attributes: `ref_allele`, `alt_allele` (both strings).
  - Method `variant_type(self)` → `str`, returning:
    - `"SNP"` if `ref_allele` and `alt_allele` are both length 1,
    - `"insertion"` if `alt_allele` is longer than `ref_allele`,
    - `"deletion"` if `alt_allele` is shorter than `ref_allele`,
    - `"MNV"` otherwise (same length, but longer than 1 base).
  - Override `describe()` to also include `ref_allele > alt_allele` and the
    variant type, e.g. `"Variant chr1:3050-3050(+) C>T (SNP)"`.

All three subclasses must call `super().__init__(...)`.


The file `oop_data.tsv` contains a small set of records (tab-separated,
one per line): genes, their exons, and a handful of variants, all on
`chr1`. Columns are: `type`, `chromosome`, `start`, `end`, `strand`,
`field_a`, `field_b`, where the meaning of `field_a`/`field_b` depends on
`type`:

| `type`    | `field_a`            | `field_b`     |
|-----------|----------------------|---------------|
| `gene`    | gene name            | *(empty)*     |
| `exon`    | parent gene name     | exon number   |
| `variant` | ref allele           | alt allele    |

Each gene's row always comes before the rows of its own exons, so a single
top-to-bottom pass over the file works fine - you don't need to handle an
exon showing up before its gene.

Write code that:

1. Reads `oop_data.tsv` and builds the corresponding `Gene`, `Exon` and
   `Variant` objects (attach each `Exon` to its parent `Gene` via
   `add_exon()`, matched by gene name; you don't need to build a `Gene`
   lookup by anything fancier than a plain dict).
2. Prints a report by calling `describe()` **polymorphically** on every
   `Gene` and every `Variant` object (i.e. iterate over one combined list
   and just call `.describe()` - do not branch on type to decide what to
   print), and next to each gene also prints its `total_exon_length()`.
3. For every `Variant`, prints which `Gene`(s) it falls inside (using
   `overlaps()`), or `"intergenic"` if it falls inside none.

Expected result (verify your output matches this reasoning,
exact wording is up to you): 
* the variant at `3050` and the deletion at
`4900-4902` should be reported as inside `GENE_A` (and inside one of its
exons)
* the variant at `8100` inside `GENE_B` (and inside one of its
exons)
* the insertion at `2000` should be inside `GENE_A`'s span but
**not** inside any of its exons
* the variant at `6000` should come out as
`"intergenic"` (outside every gene).

> A variant can actually end up in one of three situations: outside 
> every gene ("intergenic"), inside a gene *and* inside one of its 
> exons, or inside a gene but **not** inside any of its exons (sitting 
> in an intron or UTR). Make sure your code actually distinguishes all 
> three - it's easy to accidentally collapse the last two into one.
