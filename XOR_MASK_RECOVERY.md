# Recovering an XOR Mask from a Moving Integer Field

**A method for stripping fixed XOR whitening off an obscured numeric field -
and, where the whitening is generated, off the entire frame around it.**

---

## The problem

A numeric field in some data stream is obscured by XOR against a fixed
unknown mask. You can observe the masked values changing over time; you
cannot observe the mask, and you have no access to the encoder.

```
observed  =  true value  XOR  mask
```

This is the ordinary situation with whitened radio frames, scrambled
telemetry, and lightly obfuscated protocol fields. The mask is constant, so
every sample is corrupted the same way - which sounds like it should make the
mask invisible, and does make it invisible to any single observation.

It is not invisible across observations, provided the underlying value
*moves*.

---

## Why it is possible at all

Masking is XOR. The signal's structure is arithmetic. **These two do not
commute**, and the whole method lives in that gap.

XOR treats every bit independently - no carries, no interaction between
positions. Addition does the opposite: adding to a low bit can propagate all
the way up. So when a masked value changes by an arithmetic step, the
*pattern* of bits that flip depends on the true value, not the masked one.
This pattern leaks the mask.

If the field were random, or changed by XOR rather than arithmetic, none of
this would work. What is required is only that the value behaves like a
*number* - that it goes up and down by amounts small relative to its range.
It does not need to be a counter, and it does not need a schedule.

---

## The one free observable

Take any two samples and XOR them:

```
(V₁ XOR M)  XOR  (V₂ XOR M)  =  V₁ XOR V₂
```

The mask appears twice and annihilates itself. **Every comparison between
samples is mask-free**, even though no individual sample is.

So before knowing anything about the mask, you can already see exactly which
bit positions changed between any two readings. That is the entire input to
what follows.

This cancellation is the same one behind the classical two-time-pad attack
and behind differential cryptanalysis: a difference passes through
XOR-with-a-constant untouched.

---

## The lever: boundary crossings

Here is the fact that turns "which bits flipped" into "what the mask is".

Consider bit `k` of a number that is drifting up and down. **Bit `k` flips
only when the value crosses a multiple of `2^k`.** That is not a statistical
tendency; it is what binary place value means.

And crossing a boundary constrains the value tightly. Just below a multiple
of `2^k`, the low `k` bits are near all-ones. Just above, they are near
all-zeros:

```
                    ... 0 1 1 1 1 1 1        just below 2^k
   crossing  ─────────────────────────
                    ... 1 0 0 0 0 0 0        just above
```

So the moment you observe bit `k` flip, you know the true value's low bits at
that moment - to within the size of the step that carried it across.

Combine with the free observable, and the mask's low bits fall out:

```
mask, low k bits  =  observed value, low k bits  XOR  true value, low k bits
                     └── you have this ──┘        └── the crossing tells you this ──┘
```

No search, no key guessing, no known plaintext. The arithmetic hands it over.

### The exact case: unit steps

When the value moves by exactly ±1, the constraint is not approximate but
exact, and the observation is unmistakable.

A ±1 step flips a contiguous run of low bits and nothing else, so the XOR of
two consecutive samples is always of the form `2^m − 1` - that is, 1, 3, 7,
15, 31, 63, … and never a scattered pattern. Seeing such a value identifies
both that a crossing occurred and how deep it went. Call `m` the **depth**.

A depth-`m` crossing can only have come from one low-bit pattern, so:

```
mask, low m bits  =  observed value, low m bits  XOR  (2^(m−1) − 1)
```

One subtraction per event. Each crossing hands over the bottom `m` bits of
the mask outright, and - importantly for what follows - hands them over
**consecutively**, with no gaps.

### The general case: bounded, irregular, bidirectional steps

Unit steps are the clean case, not the necessary one. Relax to a signal whose
steps are merely *bounded* by some `D`, varying in size, going both
directions, on no schedule at all.

The logic survives with one change: instead of pinning the low bits to a
single value, a crossing pins them to a **narrow set**. Crossing upward
through a multiple of `2^k` with a step of size `d ≤ D` means the value
before the step was within `D` below the boundary; crossing downward means it
was within `D` above. So each crossing yields at most `2D` candidate values
for the mask's low `k` bits - the doubling because you generally cannot tell
up from down.

Then intersect. Every crossing at level `k` produces such a set, and the true
mask lies in all of them. Successive crossings arrive with different offsets
from the boundary, so the intersection collapses quickly - typically to a
handful of candidates after a modest number of events, and the residue is
usually the up/down pair rather than genuine uncertainty.

A signal wandering irregularly with steps of ±1 to ±9, no periodicity, over a
span of roughly 22,000, narrows most levels to **two** candidates - the
direction ambiguity, nothing more.

The general recipe, then:

1. XOR consecutive samples to get the flip pattern (mask-free).
2. The highest flipped bit identifies which boundary was crossed.
3. Convert that to a candidate set for the mask's low bits.
4. Intersect across all crossings at that level.
5. Repeat per level; deeper levels give more bits.

Unit steps are simply the case where `D = 1` and step 3 returns a single
candidate instead of a set.

---

## What governs how much you recover

Not the number of samples. **The range the signal traverses.**

You learn mask bit `k` only if the value actually crosses a multiple of
`2^k`. A signal that jitters within a narrow band, however long you watch it,
never crosses the high boundaries and never reveals the corresponding mask
bits. A signal that sweeps a wide range exposes them quickly, even from few
samples.

As a rule of thumb, a field that traverses a span `S` gives you roughly
`log₂(S)` mask bits. Watching ten times longer without moving further gets
you nothing extra; moving twice as far gets you one more bit.

This has a practical consequence worth internalising: **the useful field is
the one with the widest excursion, not the one with the most samples or the
most regular behaviour.** A slow sensor that drifts across its whole range
over a week beats a fast one that hovers.

For a field that never varies, the count is zero - no crossings, no
information, nothing recoverable. That is the wall the next section goes
around.

---

## Extending to the whole frame

Everything so far recovers the mask *for one field*. If the whitening is
generated rather than arbitrary, one field is enough for all of them.

Communication systems rarely store a mask; they generate one, almost always
with a linear-feedback shift register - PN9 in sub-GHz radio, PN15 and
similar elsewhere, energy-dispersal sequences in broadcast standards. The
frame is XORed against the generator's output, so a field's mask is a
**window onto a keystream**, not an independent constant.

Two properties then matter:

**A register of degree `L` is pinned by `L` consecutive output bits.** Those
bits *are* its internal state as they shift out. From them the sequence can
be run forward to the end of the frame and backward to its start - backward
included, because a valid generator polynomial has a non-zero constant term,
which makes the state transition reversible.

**With `2L` consecutive bits, the polynomial itself can be derived**, via
Berlekamp-Massey, so prior knowledge of the generator is not required.

The join with the previous section is what makes this useful: boundary
crossings return **consecutive** mask bits, which is exactly the shape an
LFSR needs. Recover enough of one field's mask and you obtain the whitening
for the entire frame - including fields that never change at all, and which
no direct method could ever reach.

For PN9 (`L = 9`), nine consecutive bits suffice with the polynomial known;
eighteen suffice without it. Both are modest.

---

## What you cannot recover

Three limits. The first two are permanent; the third is what the extension
exists to defeat.

**The complement is invisible to the signal alone.** If a mask `M` explains
the data, so does its bitwise complement - the decoded signal simply becomes
its own mirror image, `(2ⁿ−1) − V`. Every difference reverses sign, so every
measure built on step *magnitudes* - total variation, step histograms,
smoothness of any kind - is exactly unchanged. This is not a matter of
insufficient data; it is a symmetry, and it holds for every signal and every
observation length.

Breaking it requires information from outside the signal's own shape. Either
a **directional** fact - that the value genuinely rose between two identified
moments - or, far more conveniently, a frame checksum computed before
whitening, which rejects the complement outright. See *Confirming the
result*.

**The top bit may be invisible.** For a field that wraps, flipping the
highest mask bit is indistinguishable from adding half the range - it shifts
absolute values but changes no difference between them. For a field that
stays within a sub-range and never wraps, the same flip would tear the signal
across the range boundary, and *is* detectable.

**Bits above the signal's excursion are invisible.** As above: no crossing, no
information. Under a generated mask this ceases to bind, since those bits are
computed from the recurrence rather than observed. Under a stored mask it is
final.

---

## Confirming the result

Two internal checks come free, and - in the common case where the frame
carries a checksum - one external check that is far stronger than either.

**Shallow crossings must nest inside deep ones.** A depth-12 event and a
depth-4 event both constrain the mask's low 4 bits, and they must agree. With
many events at every depth this becomes a strong consistency argument. Treat
a single disagreement as a stop condition rather than noise; it usually means
the bit ordering is wrong or a sample is corrupt, and averaging over it
destroys the signal.

**The extension is self-checking.** If a register recovered from one field
also reproduces the mask independently derived for a second field, at the
correct offset, that is demanding to pass by accident. Failure means the
generated-mask assumption is wrong.

### The frame may certify the mask for you

If the frame carries a CRC, and **the CRC was computed before whitening was
applied**, then a correct mask is self-certifying: de-whiten the frame and the
checksum simply validates.

That ordering is not exotic - it is the conventional arrangement, and the
default in several vendor stacks. Silicon Labs' EFR32 tooling, for instance,
computes the CRC over the un-whitened stream and then whitens payload and CRC
together. Where that holds, this is a categorically better test than anything
in the preceding sections:

- **Binary, not a ranking.** Smoothness scoring produces an ordering of
  candidates and an argument. A CRC produces pass or fail.
- **Independently repeatable.** Every captured frame is a fresh trial. A mask
  that validates across hundreds of frames is not a coincidence.
- **It resolves the complement ambiguity** - the one limit described above as
  permanent. A CRC is affine over GF(2), so complementing the whole frame
  changes the checksum by a fixed amount `L(J)` determined by the frame length
  and polynomial, while the complemented CRC field changes by all-ones. These
  agree only if `L(J)` happens to equal all-ones, which for ordinary
  polynomials and lengths it does not. The property is all-or-nothing per
  frame length rather than per message, so it is worth checking once for your
  specific geometry - but generically the complement fails the checksum and
  the ambiguity dissolves.
- **It identifies the checksum at the same time.** If the CRC algorithm is
  unknown too, search the pair jointly: candidate masks against a catalogue of
  standard CRCs. A combination that validates across many frames confirms both
  simultaneously - and confirming a *standard* algorithm is stronger evidence
  than confirming a bespoke one, since a catalogue entry has no free
  parameters left to absorb error.

The whole method then becomes a bootstrap. Boundary crossings in one moving
field give partial mask bits; the shift-register recurrence extends those
across the frame; the checksum then confirms the extension, resolves the
residual ambiguity, and names the checksum algorithm. Each stage validates the
one before it.

**The ordering caveat matters, and is checkable.** If instead the CRC is
computed *over* whitened data, it validates on the whitened form and
de-whitening breaks it - in which case the checksum says nothing about the
mask. The two cases are easy to distinguish: test the CRC before and after
de-whitening, and see which side it passes on.

Where no checksum exists, or it is computed post-whitening, the earlier
limits stand unchanged: internal consistency is all you have, and the
complement requires an external directional fact.

---

## Related work

The components are individually standard; the composition is what produces
the result on static fields.

- The **difference cancellation** is the basis of the two-time-pad attack and
  of differential cryptanalysis - a difference passes through
  XOR-with-a-constant unchanged. Biham and Shamir, *Differential
  Cryptanalysis of the Data Encryption Standard*, Springer, 1993.
- **Searching a keyspace and scoring candidates by plaintext plausibility**
  is the standard shape of automated classical cryptanalysis, and is what the
  general case reduces to when arithmetic structure is too weak to invert
  directly.
- **Recovery of a register from its output** is the classical theory of
  linear recurring sequences; the `2L` bound and the recovery procedure are
  Berlekamp-Massey. Massey, *Shift-Register Synthesis and BCH Decoding*, IEEE
  Transactions on Information Theory 15(1), 1969.
- **Blind scrambler reconstruction** - Cluzeau, *Reconstruction of a Linear
  Scrambler*, IEEE Transactions on Computers 56(9), 2007, and its successors
  - recovers a scrambler's polynomial from a long segment of observed output.
  The situation here is complementary: the keystream is never observed
  directly at all, and is instead reconstructed from mask bits extracted from
  a structured plaintext field.
- **Whitening in practice**: manufacturer notes such as TI's DN509 and NXP's
  AN5070 document the PN9 generator (`x⁹ + x⁵ + 1`) and the reason packet
  radios whiten - suppressing DC bias and spectral lines. Because whitening
  normally begins at a fixed frame offset, each field always meets the same
  segment of the keystream, which is precisely why a *fixed per-field mask*
  is the usual observable rather than an exotic one.

What does not appear to be written down elsewhere is the join: arithmetic
boundary crossings yield **consecutive** mask bits, which is exactly what a
shift register needs to be pinned, so a single moving field can de-whiten the
static ones around it. Each half is old. Together they reach fields that
neither half can.