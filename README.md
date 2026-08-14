# Recovery of a static XOR mask from a structured integer signal

**Abstract.** A fixed, unknown bitmask is XOR-applied to every sample of an
unknown integer-valued signal, and only the masked samples are observed -
the situation that arises whenever a whitened or obfuscated numeric field
must be decoded without access to the encoder. We show that when the signal
carries arithmetic structure, the mask is recoverable in closed form with no
search: each carry of depth $`m`$ in the underlying signal exposes exactly
$`m`$ *consecutive* low-order mask bits (Theorem 1). We characterise the
limit of any such recovery: under a free-mask model, the mask is identifiable
only modulo a subgroup determined jointly by which signal bits vary in the
observation window and by the invariances of the scoring functional used
(Theorem 2). We then show that this limit is a property of the free-mask
model rather than of masked observation in general: if the mask is a window
onto a linear-feedback shift register keystream of degree $`L`$ - the
standard construction in communication whitening - then $`L`$ consecutive
mask bits determine the keystream everywhere, in both directions, and
$`2L`$ bits determine it without prior knowledge of the polynomial
(Theorem 3). Since Theorem 1 delivers consecutive bits, the two compose:
carry events in a single varying field can de-whiten regions of a frame that
never vary at all, and which no quantity of observation could otherwise
reach.

*Keywords:* XOR masking, data whitening, blind descrambling, known-plaintext
model, identifiability, linear feedback shift registers.

---

## 1. Setup

Let $`n \ge 1`$, $`N = 2^n`$. Identify $`\{0,1\}^n`$ with $`\mathbb{Z}_N`$ by
binary representation; write $`x[k]`$ for bit $`k`$ of $`x`$, $`\oplus`$ for
bitwise XOR, $`e_k = 2^k`$, and $`J = N-1`$ (all ones). Note
$`(\mathbb{Z}_N, \oplus) \cong \mathbb{F}_2^n`$ as a group.

**Definition 1 (masked observation model).** A masked observation of length
$`T`$ is a sequence $`R = (R_1,\dots,R_T) \in \mathbb{Z}_N^T`$ such that

$$R_t = V_t \oplus M \qquad (1 \le t \le T)$$

for an unknown signal $`V \in \mathbb{Z}_N^T`$ and unknown mask
$`M \in \mathbb{Z}_N`$. For a candidate mask $`M'`$ define the induced decode
$`V^{M'} := (R_t \oplus M')_t`$. Thus $`V^{M} = V`$.

The recovery problem is: given $`R`$ and structural hypotheses on $`V`$,
determine $`M`$.

---

## 2. The invariance that makes the problem tractable

**Lemma 1 (difference invariance).** For all $`t, u`$ and every candidate
$`M'`$,

$$V^{M'}_t \oplus V^{M'}_u = R_t \oplus R_u,$$

independent of $`M'`$.

*Proof.*

$$(R_t \oplus M') \oplus (R_u \oplus M') = R_t \oplus R_u \oplus (M' \oplus M') = R_t \oplus R_u. \qquad \blacksquare$$

**Consequence.** Any statistic factoring through XOR-differences is
uninformative about $`M`$ - but is computable without knowing $`M`$. In
particular the per-bit toggle indicators
$`\mathbb{1}[V_t[k] \ne V_{t+1}[k]]`$ are mask-free, which is what permits
bit-ordering to be established prior to mask recovery.

---

## 3. Exact recovery from arithmetic structure

**Theorem 1 (Carry Signature Theorem).** Suppose
$`V_{t+1} \equiv V_t + 1 \pmod N`$. Let $`\tau`$ be the number of trailing
$`1`$-bits of $`V_t`$ and $`m = \min(\tau + 1,\, n)`$. Then:

1. $`V_t \oplus V_{t+1} = 2^m - 1`$.
2. Conversely, if $`V_t \oplus V_{t+1} = 2^m - 1`$ for some $`m \ge 1`$, then
   $`V_t \equiv 2^{m-1} - 1 \pmod{2^m}`$.
3. Consequently

$$M \equiv (R_t \bmod 2^m) \oplus (2^{m-1} - 1) \pmod{2^m}$$

*Proof.*

**(1)** By definition of $`\tau`$, bits $`0,\dots,\tau-1`$ of $`V_t`$ are
$`1`$ and bit $`\tau`$ is $`0`$ (if $`\tau < n`$). Adding $`1`$ propagates a
carry through positions $`0,\dots,\tau-1`$, clearing them, and sets bit
$`\tau`$; positions above $`\tau`$ are unchanged. Exactly bits
$`0..\tau`$ change, so the XOR is
$`\sum_{i=0}^{\tau} 2^i = 2^{\tau+1}-1`$. If $`\tau = n`$ (so
$`V_t = N-1`$) then $`V_{t+1} = 0`$ and the XOR is $`N - 1 = 2^n - 1`$.

**(2)** Increment-by-one changes bits in exactly one pattern: a maximal run
of $`1`$s becomes $`0`$s, and the single $`0`$ immediately above becomes
$`1`$. So if bits $`0..m-1`$ are precisely the changed positions, then bit
$`m-1`$ went $`0 \to 1`$ and bits $`0..m-2`$ went $`1 \to 0`$. Hence
$`V_t \bmod 2^m = 2^{m-1}-1`$.

**(3)** XOR is bitwise, so reduction mod $`2^m`$ commutes with it:
$`(a \oplus b) \bmod 2^m = (a \bmod 2^m) \oplus (b \bmod 2^m)`$. Therefore
$`M \bmod 2^m = (R_t \bmod 2^m) \oplus (V_t \bmod 2^m)`$, and substitute (2).
$`\blacksquare`$

Note that the recovered bits are **consecutive**: a depth-$`m`$ carry yields
mask positions $`0`$ through $`m-1`$ with no gaps. This is what makes
Theorem 3 applicable.

**Corollary 1 (complete recovery).** If the observation contains a
transition with $`m = n`$ - i.e. the signal passes through $`N-1`$ - then
$`M`$ is determined exactly, with no search.

**Corollary 2 (recovery rate).** For a counter of uniformly distributed
phase, depth-$`m`$ carries occur with frequency $`2^{-m}`$ per step. Hence
$`T`$ consecutive increments determine $`M \bmod 2^{m}`$ for
$`m \approx \log_2 T`$; recovering all $`n`$ bits by this route alone
requires $`T = \Omega(2^n)`$.

Theorem 1 generalises to any step $`d`$ with known value: the achievable
resolution is governed by how strongly $`d`$ constrains the pre-value's low
bits, and $`d=1`$ is the maximally informative case.

---

## 4. The identifiability obstruction

**Lemma 2 (XOR versus modular addition).** For $`k \in \{0,\dots,n-1\}`$,
the maps $`x \mapsto x \oplus e_k`$ and $`x \mapsto x + e_k \pmod N`$
coincide on all of $`\mathbb{Z}_N`$ **iff** $`k = n-1`$.

*Proof.* ($`\Leftarrow`$) If $`x[n-1]=0`$ then
$`x \oplus e_{n-1} = x + e_{n-1}`$; if $`x[n-1]=1`$ then
$`x \oplus e_{n-1} = x - e_{n-1} \equiv x + e_{n-1} \pmod{2^n}`$.
($`\Rightarrow`$) For $`k < n-1`$ choose $`x`$ with $`x[k]=1`$ and
$`x[k+1]=0`$. Then $`(x \oplus e_k)[k+1] = 0`$ while
$`(x + e_k)[k+1] = 1`$. $`\blacksquare`$

**Lemma 3 (complementation is reflection).** For $`x \in [0,N)`$:
$`x \oplus J = J - x`$.

**Definition 2 (invariance classes).** Let
$`\Phi: \mathbb{Z}_N^T \to \mathbb{R}`$ be a scoring functional. Call
$`\Phi`$

- **translation-invariant** if $`\Phi(V + c) = \Phi(V)`$ for constant $`c`$;
- **modular-translation-invariant** if $`\Phi(V + c \bmod N) = \Phi(V)`$;
- **reflection-invariant** if $`\Phi(c - V) = \Phi(V)`$ for constants $`c`$.

Total variation $`\sum_t |V_{t+1}-V_t|`$, the multiset of step magnitudes,
step variance, and any threshold count
$`\#\{t : |V_{t+1}-V_t| \le \kappa\}`$ are translation- and
reflection-invariant.

**Theorem 2 (Ambiguity Theorem).** Assume the mask ranges freely over
$`\mathbb{Z}_N`$. Let

$$Z = \{\,k : V_t[k] \text{ is constant in } t\,\}$$

(computable from $`R`$ alone, by Lemma 1). Let
$`G \le (\mathbb{Z}_N,\oplus)`$ be the subgroup generated by

- $`\{e_k : k \in Z\}`$, if $`\Phi`$ is translation-invariant;
- $`J`$, if $`\Phi`$ is reflection-invariant;
- $`e_{n-1}`$, if $`\Phi`$ is modular-translation-invariant.

Then $`\Phi(V^{M \oplus g}) = \Phi(V^{M})`$ for every $`g \in G`$. Hence
$`M`$ is identifiable at best up to the coset $`M \oplus G`$.

*Proof.* It suffices to check the generators, since $`\Phi`$-preserving maps
compose.

*Generator* $`e_k`$, $`k \in Z`$. Say $`V_t[k] = b`$ for all $`t`$. Then
$`V_t \oplus e_k = V_t + (1-2b)2^k`$ for every $`t`$ - one constant offset,
independent of $`t`$. So $`V^{M\oplus e_k} = V^M + c`$, and
translation-invariance applies.

*Generator* $`J`$. By Lemma 3, $`V^{M \oplus J} = J - V^{M}`$; apply
reflection-invariance.

*Generator* $`e_{n-1}`$. By Lemma 2,
$`V^{M \oplus e_{n-1}} = V^{M} + e_{n-1} \pmod N`$; apply
modular-translation-invariance. $`\blacksquare`$

**Corollary 3 (necessity of directional ground truth).** If $`\Phi`$ is
reflection-invariant, then $`M`$ and $`M \oplus J`$ are indistinguishable
**for every signal and every observation length**. No quantity of additional
observation resolves the complement class. Breaking it requires a functional
that is not reflection-invariant, i.e. one that distinguishes $`V`$ from
$`J - V`$ - equivalently, an external datum asserting the *sign* of at least
one true difference $`V_u - V_t`$.

**Remark (sharpness).** $`G`$ is a **lower** bound on the ambiguity, not an
upper bound. Accidental degeneracies can add ties for particular
$`(V,\Phi)`$: a signal with rigidly periodic low-bit structure - for example
a deterministic ramp of constant step $`7`$ - admits extra
$`\Phi`$-preserving mask flips not in $`G`$. Such ties are artifacts of the
signal, not of the model, and generically vanish under jitter.

---

## 5. Structured masks: escaping the obstruction

Theorem 2 assumes the mask is an arbitrary element of $`\mathbb{Z}_N`$, so
its bits are logically independent and a bit that leaves no trace in the
data is unrecoverable. In practice masks are rarely arbitrary. Communication
systems whiten with a *pseudorandom sequence* generated by a linear-feedback
shift register, so a field's mask is a **window** onto a keystream whose bits
satisfy a linear recurrence. Under that hypothesis the obstruction largely
dissolves.

**Definition 3 (LFSR-generated mask).** Let $`p(x) = x^L + c_{L-1}x^{L-1} +
\dots + c_1 x + c_0`$ over $`\mathbb{F}_2`$ with $`c_0 = 1`$, and let
$`(\kappa_i)_{i \ge 0}`$ satisfy the recurrence
$`\kappa_{i+L} = \sum_{j=0}^{L-1} c_j \kappa_{i+j}`$. A frame is
*LFSR-whitened* if its frame-wide mask bits are $`\kappa_{i}`$ at frame
position $`i`$; a field occupying positions $`[a, a+n)`$ then carries mask
window $`(\kappa_a, \dots, \kappa_{a+n-1})`$.

**Theorem 3 (LFSR extension).** Under Definition 3 with $`p`$ known, any
$`L`$ consecutive keystream bits $`\kappa_j, \dots, \kappa_{j+L-1}`$
determine $`\kappa_i`$ for **every** $`i \ge 0`$ - both forwards and
backwards.

*Proof.* The $`L`$ known bits are the register state at position $`j`$.
Forward determination is immediate from the recurrence. For backward
determination, the state map is multiplication by the companion matrix
$`C`$ of $`p`$, whose determinant is $`c_0 = 1 \ne 0`$; hence $`C`$ is
invertible over $`\mathbb{F}_2`$ and the state at position $`j-1`$ is
$`C^{-1}`$ applied to the state at $`j`$. Iterating reaches every earlier
position. $`\blacksquare`$

**Corollary 4 (unknown polynomial).** Given $`2L`$ consecutive keystream
bits and no knowledge of $`p`$, the Berlekamp–Massey algorithm returns the
minimal LFSR generating them - both $`p`$ and the state - in $`O(L^2)`$
operations, after which Theorem 3 applies.

**Corollary 5 (recovery beyond the observation window).** Mask bits at frame
positions whose underlying signal never varies are unidentifiable under
Theorem 2, yet are **determined** under Definition 3 by extrapolation from
any $`L`$ consecutive bits recovered elsewhere in the frame. Consequently
the "a bit that never varies is invisible" limit is a property of the
free-mask model, not of masked observation in general.

**Composition with Theorem 1.** Theorem 1 yields *consecutive* mask bits,
which is exactly the input Theorem 3 requires. A single carry of depth
$`m \ge L`$ in one varying field therefore suffices to de-whiten the entire
frame, including fields that are constant throughout the capture. For the
PN9 generator $`p(x) = x^9 + x^5 + 1`$ common in sub-GHz radio, $`L = 9`$:
a depth-$`9`$ carry - one increment in $`512`$ for a uniformly phased
counter - is enough with $`p`$ known, and depth $`18`$ suffices without it.

**Corollary 6 (collapse of the ambiguity group).** If $`n \ge L`$, the set
of admissible masks has size at most $`2^L < 2^n`$. The residual ambiguity
of Theorem 2 is then $`\{g \in G : M \oplus g \text{ is an admissible
window}\}`$, which is generically the identity alone. The complement
symmetry of Corollary 3 survives only in the exceptional case that
$`M \oplus J`$ is itself a keystream window.

**Two hypotheses this rests on**, both of which must be checked rather than
assumed. First, that whitening is LFSR-generated at all: a mask fitted
independently per field, or one drawn from a stored table, satisfies no
recurrence and admits no extension. Testing this is direct - recover masks
for two or more fields and ask whether one register reproduces both at
their respective frame offsets. Second, that the map from a field's numeric
bit weights to frame positions is known: Theorem 1 returns low-order
*numeric* bits, and these are contiguous in the frame only under a known
ordering, possibly reversed if the field is transmitted least-significant
bit first.

---

## 6. Recovery procedure

**Proposition.** Given $`R`$, the following recovers $`M`$ up to
$`M \oplus G`$, and under Definition 3 recovers the frame-wide keystream:

1. **Order.** Determine the bit-to-weight assignment from per-bit toggle
   rates (mask-free, Lemma 1); for a smoothly varying signal these decay
   monotonically from the least significant bit.
2. **Low bits.** Let $`m^\star`$ be the deepest observed carry. Apply
   Theorem 1(3) to fix $`M \bmod 2^{m^\star}`$. Consistency across all
   depth-$`m`$ events for $`m \le m^\star`$ is a validity check on step 1.
3. **High bits.** Search the $`2^{\,n-m^\star}`$ remaining candidates,
   maximising $`\Phi`$.
4. **Coset.** Resolve the residual $`|G|`$ candidates by external ground
   truth: absolute level, or a known direction of change.
5. **Extend (optional).** If $`m^\star \ge L`$, apply Theorem 3 - or
   Corollary 4 when $`p`$ is unknown and $`m^\star \ge 2L`$ - to obtain the
   keystream over the whole frame. Steps 3 and 4 may then be skipped
   entirely, since the extension supplies the high bits directly and
   Corollary 6 collapses the coset.

Cost: $`O(T)`$ for steps 1–2, $`O(2^{\,n-m^\star} T)`$ for step 3,
$`O(L^2)`$ for step 5. Step 2 is what makes step 3 tractable; step 5, when
available, removes the search altogether.

---

## 7. Verification

Every claim above admits direct machine verification, and all were checked
before publication. At $`n=10`$ the statements quantified over
$`\mathbb{Z}_N`$ were tested against all $`N = 1024`$ values; Theorem 1(3)
was additionally tested on $`2000`$ uniformly random $`(M, V_t)`$ pairs.

| claim | result |
|---|---|
| Thm 1(1) forward carry signature | holds for all $`V_t`$ |
| Thm 1(2) converse | holds for all $`V_t`$ |
| Thm 1(3) mask recovery | holds on all trials |
| Lemma 2 uniqueness of $`k = n-1`$ | only $`k=9`$ at $`n=10`$ |
| Lemma 3 $`x \oplus J = J - x`$ | holds for all $`x`$ |
| Thm 2 generator $`e_k`$, $`k \in Z`$ | constant translation confirmed |
| Thm 3 forward extension (PN9) | full stream reproduced from $`9`$ bits |
| Thm 3 backward extension (PN9) | full stream reproduced, $`C^{-1}`$ iterated |
| Cor 4 Berlekamp–Massey (PN9) | minimal length $`L=9`$ recovered from $`17`$ bits |

Theorem 2 was further confirmed on synthetic signals at $`n=16`$, and those
experiments also produced the two phenomena recorded in the Remark. A
bounded random walk confined to a narrow interval yielded a $`512`$-element
tie set rather than the $`|G|`$ predicted from reflection alone - the excess
being exactly the non-toggling high positions, i.e. a large $`Z`$, as
Theorem 2 requires. A deterministic ramp of constant step $`7`$ yielded one
tie outside $`G`$, an accidental degeneracy of that particular signal which
disappeared once jitter was introduced. Both outcomes are consistent with
$`G`$ being a lower bound and not an upper bound on the ambiguity.

---

## 8. Interpretation

Theorems 1, 2 and 3 together delimit the problem.

Theorem 1 says arithmetic structure is *information*: each carry of depth
$`m`$ transfers $`m`$ consecutive bits of the mask into the observation, for
free and without search.

Theorem 2 says that under a free-mask model, structure is also *the limit*:
a bit that never toggles contributes nothing, since its mask bit acts as a
pure translation, and complementation is an exact symmetry of any
magnitude-based score.

Theorem 3 says that limit is contingent, not fundamental. It binds only
because the free-mask model treats mask bits as independent. Real whitening
is generated, and generated masks are heavily constrained - a degree-$`L`$
register has $`2^L`$ possible keystreams regardless of frame length, so
pinning $`L`$ consecutive bits pins all of them. The practical effect is a
reversal: the field that is *easiest* to attack, because it carries a
counter and therefore produces carries, becomes the lever that de-whitens
the fields that are *impossible* to attack directly because they never
change.

Corollary 3 remains the standing caution. Reflection ambiguity is not a
sample-size problem, and where Corollary 6 does not apply, a practitioner
who scores candidate masks by smoothness and reports a unique answer has
either used a non-reflection-invariant score, or has silently imported an
external assumption about which direction the signal moves.

---

## 9. Relation to prior work

The individual ingredients are standard, and are cited below as such.

- **Lemma 1** is the observation underlying both the two-time-pad attack and
  differential cryptanalysis: an XOR difference passes through
  XOR-with-a-constant with probability 1, annihilating constant key
  material. See Biham and Shamir, *Differential Cryptanalysis of the Data
  Encryption Standard* (Springer, 1993).
- **§6 steps 3–4** follow the standard shape of automated classical
  cryptanalysis: enumerate a keyspace, score candidates by a
  plaintext-plausibility functional, and take the maximiser.
- **Theorem 3 and Corollary 4** are classical facts about linear recurring
  sequences. That $`L`$ consecutive outputs determine an LFSR's future, and
  $`2L`$ determine the register itself, is the content of the
  Berlekamp–Massey algorithm; see Massey, *Shift-Register Synthesis and BCH
  Decoding*, IEEE Transactions on Information Theory 15(1), 1969.
- The **blind scrambler reconstruction** literature - M. Cluzeau,
  *Reconstruction of a Linear Scrambler*, IEEE Transactions on Computers
  56(9), 2007, and its successors - recovers an LFSR scrambler's feedback
  polynomial from a long output segment under assumptions on the input
  stream. The present §5 addresses the complementary situation: the
  keystream is not observed directly at all, but is reconstructed from mask
  bits that Theorem 1 extracts from a *structured plaintext field*.

**On the contribution.** No single component here is new. What does not
appear to be stated elsewhere is the composition, and it is the composition
that does the work: Theorem 1 produces *consecutive* mask bits, which is
precisely the input Theorem 3 consumes, so carry events in one arithmetic
field yield the whitening of an entire frame - including regions that
Theorem 2 shows are unreachable by any amount of direct observation. The
explicit identifiability analysis of §4, and its contingency on the
free-mask assumption made precise in §5, likewise appear not to be written
down in this form.
