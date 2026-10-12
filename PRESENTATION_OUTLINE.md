# Presentation outline — APMA 4903 seminar talk

**Format:** 60 min talk + 15 min questions
**Audience:** Applied Math juniors, Columbia. Assume multivariable calculus,
linear algebra, a first probability course. Assume **no** biology, **no** machine
learning.

| block | presenter | content | nugget |
|---|---|---|---|
| 1 | **Vignesh** | intro, background, Reg3A | — |
| 2 | **Vignesh** | Boltz-Prot-1 (and submit the live job) | — |
| 3 | **Jun** | ESM-C | **N1** derivation: bidirectional encoding |
| 4 | **Parsa** | the SAE + training results | **N2** derivation: TopK SAE |
| 5 | **Vignesh** | interpretability results + design demo | **N3** demo: generate → embed → FVE |

**Steering is cut from the talk.** It is a real negative result but it competes
with the finding for airtime. Keep **one backup slide** (spec at the end) for
the Q&A question "did you try *using* the features?"

> **This file is a spec, not slides.** Each slide gives (a) what appears on it,
> (b) what the speaker says, (c) what the presenter must be able to defend.
> Hand it to a design agent as-is.

---

## Hard constraints from the course guidelines

| rule | compliance |
|---|---|
| Slide 1 = presenter names + **why we got interested** | Slide 1, scripted personal opening |
| Slide 2 = 1–3 primary references, full citations | Slide 2; extended list Slide 3 |
| Websites cited properly, never bare URLs | all author/year/title/venue |
| No unattributed graphics | every figure generated from our own data; attribution line per slide |
| **One tech nugget per presenter** | N1 Jun, N2 Parsa, N3 Vignesh |
| **Complete sentences rarely** | slide text is fragments; prose lives in speaker notes |
| **No slop** | "Must defend" block on every slide |
| **No karaoke** | speaker notes deliberately differ from slide text |
| Nugget = derivation **or** demo + pseudocode | N1, N2 derivations; N3 demo with pseudocode |
| Pseudocode matches code run live | N3; notebook spec at the end |
| AI use disclosed | Slide 4 |

### Timing

| block | slides | min | cumulative |
|---|---|---|---|
| 1 Intro / Reg3A | 1–8 | 9 | 9 |
| 2 Boltz-Prot-1 | 9–12 | 7 | 16 |
| 3 ESM-C + **N1** | 13–19 | 14 | 30 |
| 4 SAE + **N2** | 20–27 | 15 | 45 |
| 5 Results + **N3** demo | 28–34 | 13 | 58 |
| Close | 35 | 2 | **60** |

Cut order if running long: slide 16, then 11, then 31. **Never cut a nugget.**

---

# BLOCK 1 — Intro and background (slides 1–8, 9 min, Vignesh)

### Slide 1 — Title
**On slide:** title; three names; one render of a designed binder docked to
Reg3A. Caption: "our data, rendered in PyMOL".

**Speaker (scripted — guidelines require opening with why):**
> "Last year I was on an iGEM team designing proteins to stick to a target. We
> could generate thirty-six thousand candidates in an afternoon. What bothered
> me is that we had no idea *why* any of them were supposed to work — you get a
> sequence and a confidence score and nothing else. This talk is about opening
> that box using a tool borrowed from language-model interpretability."

**Must defend:** which chain is which in the render; what "binding" means
physically.

### Slide 2 — Primary references
**On slide, verbatim:**
- Gao, L., la Tour, T. D., Tillman, H., Goh, G., Troll, R., Radford, A.,
  Sutskever, I., Leike, J., Wu, J. (2024). *Scaling and evaluating sparse
  autoencoders.* arXiv:2406.04093.
- Bricken, T., Templeton, A., Batson, J., Chen, B., Jermyn, A., et al. (2023).
  *Towards monosemanticity: decomposing language models with dictionary
  learning.* Transformer Circuits Thread.
- Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N.,
  Kaiser, Ł., Polosukhin, I. (2017). *Attention is all you need.* NeurIPS 30.

### Slide 3 — Secondary references
**ACTION REQUIRED — verify each before the talk. Do not present a citation you
have not opened.**
- **Boltz-Prot-1** — the design model. Find and cite the primary paper.
- **ESM-C** (`biohub/ESMC-300M`) — cite the primary paper.
- **Besag, J. (1975).** *Statistical analysis of non-lattice data.* The
  Statistician, 24(3), 179–195. — pseudo-likelihood, needed for N1.
- CAPRI/PISA 5 Å heavy-atom contact convention — cite a structural source.

### Slide 4 — Generative AI disclosure
**On slide (fragments):** "Pipeline code + analysis: written with Claude Code" /
"Experimental design + hypotheses: ours" / "All results reproducible from the
repo" / repo URL as text.

**Speaker:** 20 seconds, state it plainly, move on.

### Slide 5 — Proteins as sequences
**On slide:** a short sequence in monospace; "20 amino acids" / "sequence →
3D fold → function".

### Slide 6 — The target: Reg3A
**On slide:** Reg3A render; "138 residues, signal peptide removed" /
"chain A in every complex".
**TODO (Vignesh):** one fragment on *why Reg3A*. This will be asked.

### Slide 7 — What a binder is, and why design is hard
**On slide:** cartoon — target surface with a patch marked "hotspot"; a small
binder docking onto it. Fragments: "bind a chosen patch" / "sequence space is
20^65".

**Speaker:** the search space number makes the point that this is a generative
problem, not an enumeration problem.

### Slide 8 — The gap this talk addresses
**On slide:** three fragments — "36,710 candidates" / "one score each" / "no
account of *why*".

**Speaker:** rank-and-test tells you *which* designs work, never *what makes*
them work — so nothing transfers to the next target. Pose the question: **can we
find human-readable structure inside the model's own representation?**

---

# BLOCK 2 — Boltz-Prot-1 (slides 9–12, 7 min, Vignesh)

> **Accuracy note.** Boltz-Prot-1 is a hosted service: we see inputs, outputs
> and reported metrics only. Make no claim about its architecture, objective or
> loss. It is a separate model from ESM-C.

### Slide 9 — What it does
**On slide:** box diagram — target structure + hotspot residues → [Boltz-Prot-1
v1.0] → 100 binder sequences + predicted complexes.

### Slide 10 — **Submit the live job here**
**On slide:** the job spec as fragments — "target Reg3A" / "hotspot 112–130" /
"100 designs, length 50–80" / "~13 min".

**Speaker:** submit it on screen, say explicitly: *"I'm submitting this now; we
come back to it at the end."* Then explain **why 112–130** is deferred — "the
analysis in block 5 chose this patch, and I'll show you why."

**Must defend:** what a hotspot constraint does to the generative process.

### Slide 11 — What comes back per design *(CUT SECOND)*
**On slide:** four fragments + one small complex render — predicted complex
(mmCIF) / PAE matrix / ipTM / binder sequence.

### Slide 12 — Our corpus, and the quality filter
**On slide:** the campaign table (10 campaigns, 36,710 designs, 50–139 aa) →
"2,387,652 amino acids"; ipTM histogram with the 0.7 cut marked;
"12,060 of 36,710 = 32.9% pass".

**Speaker:** two points. The amino acid count is the real sample size, because
everything downstream is per-residue. And the ipTM filter is about
**measurement validity**, not cherry-picking — a design scoring 0.2 has a
structure that is probably wrong, so its "interface" is not an interface and
anything computed on it is noise.

**Must defend:** roughly what ipTM is (0–1 predicted interface confidence); why
filter at all; why 0.7.

---

# BLOCK 3 — ESM-C, with NUGGET 1 (slides 13–19, 14 min, Jun)

**N1 type: derivation.**
**Start:** we want a representation of residue $i$ that depends on the whole
sequence, in both directions.
**End:** the masked-language-model objective, shown to be a stochastic estimator
of the pseudo-log-likelihood — and the reason bidirectionality is what makes our
binder+target trick possible at all.
**Tools:** linear algebra (attention), probability (conditional distributions,
pseudo-likelihood).

### Slide 13 — Protein language model, by analogy
**On slide:** two parallel rows — "text: the cat sat on the ___" / "protein:
M K T A Y I A ___ Q R". Fragment: "trained by masking and predicting".

### Slide 14 — What we want, stated as probability
**On slide:**

Autoregressive (GPT-style) factorization — **left context only**:
$$\log P(x) \;=\; \sum_{i=1}^{L} \log P\!\left(x_i \mid x_{<i}\right)$$

What a *representation* of residue $i$ should condition on — **both sides**:
$$P\!\left(x_i \mid x_{-i}\right), \qquad x_{-i} = (x_1,\dots,x_{i-1},x_{i+1},\dots,x_L)$$

**Speaker:** in a protein, residue 10 is constrained by residue 60 — they may
touch in the fold. A left-to-right factorization structurally cannot express
that for residue 10. So we need a different objective.

### Slide 15 — Mechanism: attention without a causal mask
**On slide:**
$$\mathrm{Attn}(Q,K,V) = \mathrm{softmax}\!\left(\frac{QK^\top}{\sqrt{d}}\right)V, \qquad \text{so} \qquad \mathbf{out}_i = \sum_{j=1}^{L} a_{ij}\,v_j, \quad a_{ij} = \frac{\exp\!\big(q_i\!\cdot\! k_j/\sqrt{d}\big)}{\sum_{j'}\exp\!\big(q_i\!\cdot\! k_{j'}/\sqrt{d}\big)}$$

Causal (GPT): add $M_{ij} = -\infty$ for $j>i$ $\Rightarrow a_{ij}=0$ for $j>i$.
Bidirectional (ESM-C): **no mask** $\Rightarrow a_{ij}>0\ \forall j$.

**Speaker:** $\mathbf{out}_i$ is a convex combination of *all* positions' values
— the weights are positive and sum to one. Bidirectionality is not an extra
mechanism; it is what you get by *not* adding the causal mask.

### Slide 16 — Why $\sqrt{d}$ *(CUT FIRST)*
**On slide:** if $q,k$ have i.i.d. zero-mean unit-variance components,
$$q\cdot k=\sum_{m=1}^{d} q_m k_m, \qquad \mathbb{E}[q\cdot k]=0, \qquad \mathrm{Var}(q\cdot k)=\sum_{m=1}^d \mathrm{Var}(q_mk_m)=d$$
so $\mathrm{sd}=\sqrt{d}$; dividing by $\sqrt{d}$ restores unit variance.

**Speaker:** without it, dot products grow like $\sqrt{d}$, the softmax
saturates, and gradients vanish. A one-line variance argument fixes it.

### Slide 17 — Objective: MLM as pseudo-likelihood *(the derivation payoff)*
**On slide:**

Pseudo-log-likelihood (Besag 1975):
$$\mathrm{PLL}(x) \;=\; \sum_{i=1}^{L} \log P\!\left(x_i \mid x_{-i}\right)$$

MLM: sample a mask set $M \subseteq \{1,\dots,L\}$, maximize
$$\mathcal{L}_{\mathrm{MLM}} \;=\; \mathbb{E}_{M}\left[\;\sum_{i \in M} \log P\!\left(x_i \mid x_{\setminus M}\right)\right]$$

**Single-token masking** ($|M|=1$, uniform) gives it *exactly*:
$$\mathbb{E}_{M}\big[\log P(x_i \mid x_{-i})\big] \;=\; \frac{1}{L}\sum_{i=1}^{L}\log P\!\left(x_i \mid x_{-i}\right) \;=\; \frac{\mathrm{PLL}(x)}{L}$$

**15% masking** conditions on $x_{\setminus M}$, which is *smaller* than
$x_{-i}$ → an approximation, bought for ~$0.15L$ terms per forward pass instead
of one.

**Speaker:** walk the three lines. Then the honest caveat: $\prod_i P(x_i\mid
x_{-i})$ is **not** a normalized distribution over sequences — the conditionals
need not be consistent with any joint. So a bidirectional model gives you no
exact sequence likelihood, only a pseudo-likelihood. This is a real limitation,
not a technicality.

**Must defend:**
- Derive the single-token line on the spot.
- Why 15% masking is an approximation and what is traded for what.
- Why there is no exact likelihood (consistency of conditionals).

### Slide 18 — What we take from ESM-C
**On slide:** fragments — "trained by masked-residue prediction" / "we read the
hidden state at layer 23" / "960 numbers per residue". Set apart: "the objective
is why that state sees both directions".

**Speaker:** slides 14–17 established the objective that produced this model.
What we take from it is not a prediction — it is the internal state at layer 23,
960 numbers per residue, which we hand to the autoencoder. The
pseudo-likelihood objective matters to us for exactly one reason: it is why that
state is conditioned on context from both directions rather than one.

**Must defend:** the distinction between a training objective and a learned
representation — why an objective you never invoke at inference still determines
what the representation contains.

**Two accuracy points, for questions rather than the slide:**
- ESM-C *does* have a masked-token prediction head (`EsmcMaskedLMHead`,
  960 → 64 vocab); given a masked position it returns a distribution over amino
  acids. So "ESM-C is solely an embedding model" is not strictly correct. What
  it does **not** have is any structure-prediction head. Avoid the phrase
  "prediction head" on the slide, since it reads as structural.
- This project uses only the hidden states. The token head was used once, in the
  steering experiments, which are cut from the talk.

> **Earlier draft error, recorded so it is not reintroduced.** This slide
> previously claimed Boltz-Prot-1's design loss contains an ESM-C
> pseudo-perplexity term. That is **false**. That regularizer lives in unrelated
> upstream design code in the iGEM repository which was **never used to generate
> any data in this project.** Our designs came from `engine: boltzprot v1.0`, a
> hosted service whose internals we cannot observe. Boltz-Prot-1 and ESM-C are
> independent models; neither contains the other. Make no claim about what is
> inside Boltz-Prot-1.

### Slide 19 — Why bidirectionality makes OUR method possible
**On slide:** the token strip `binder + | + target`, binder span boxed, target
span greyed. Fragments: "chain-break token, id 31" / "keep binder vectors only"
/ "960 numbers per residue, layer 23 of 30".

**Speaker:** the punchline of the nugget. We place the binder **first**, then a
chain-break token, then the target. Under a **causal** mask, a binder residue
could attend only to earlier *binder* residues — it could never see the target
at all. Bidirectional attention is exactly what lets every binder residue's
vector depend on the target in a single forward pass. We then discard the
target's vectors and keep the binder's: target-aware representations of binder
residues.

**Must defend:**
- Why co-attention would be impossible with a causal model (position ordering).
- Why we discard the target's vectors (we want per-binder-residue features, and
  the target is identical across all 36,710 designs, so it carries no
  discriminating signal).
- What "layer 23 of 30" means, and that the choice is provisional — our own
  layer sweep could not discriminate layers ($R^2\approx0.03$ everywhere,
  spread smaller than fold-to-fold noise), because it pooled ~65 residue
  vectors into one and asked for a single whole-complex number.

---

# BLOCK 4 — The SAE, with NUGGET 2 (slides 20–27, 15 min, Parsa)

**N2 type: derivation.**
**Start:** dense, polysemantic 960-dimensional vectors.
**End:** the constrained optimization problem we solve, why each constraint
exists, and the update rule the constraints force.
**Tools:** linear algebra, constrained optimization, projection geometry.

### Slide 20 — The problem, precisely
**On slide:** fragments — "dense: nearly all 960 nonzero" / "polysemantic: no
coordinate = one concept" / "so what do we do?"

### Slide 21 — Superposition
**On slide:** 2D cartoon — 2 dimensions, 5 near-orthogonal labelled directions,
one active at a time. Fragment: "more concepts than dimensions".

**Speaker:** if concepts are rarely simultaneously active, you can pack many
more than $d$ into $\mathbb{R}^d$ at small interference cost. That predicts
recovery needs a dictionary **larger** than $d$ — which is the next slide.

### Slide 22 — The construction
**On slide:**
$$z = \mathrm{TopK}_k\!\left(W_{\mathrm{enc}}^\top x + b_{\mathrm{enc}}\right), \qquad \hat{x} = W_{\mathrm{dec}}^\top z + b_{\mathrm{dec}}$$
$$\min_{W_{\mathrm{enc}},\,W_{\mathrm{dec}},\,b}\ \mathbb{E}_x\big\|x-\hat x\big\|_2^2 \quad\text{s.t.}\quad \|z\|_0=k,\qquad \big\|W_{\mathrm{dec}}^{(i)}\big\|_2=1\ \ \forall i$$
Shapes: $x\in\mathbb{R}^{960}$, $W_{\mathrm{dec}}\in\mathbb{R}^{16384\times960}$,
$k=64$.

**Speaker:** read it left to right — encode, keep the $k$ largest, decode from
those alone. Name both constraints and promise to justify each.

### Slide 23 — Why TopK, not $\ell_1$
**On slide:** two columns. $\ell_1$: "convex" / "shrinks the survivors" /
"sparsity only indirect". TopK: "exact $\|z\|_0=k$" / "no shrinkage" /
"non-convex; gradient only through selected coordinates".

**Speaker:** the classical move is relaxing $\ell_0\to\ell_1$ for convexity. But
$\ell_1$ biases the coefficients it keeps toward zero, and sparsity is
controlled only indirectly through the penalty weight. TopK enforces the
constraint exactly; the price is a non-smooth, non-convex objective.

**Must defend:** what shrinkage bias is; why unselected coordinates get no
gradient.

### Slide 24 — Why the decoder rows are normalised
**On slide:** the rescaling degeneracy; the constraint `‖w_i‖ = 1`; the two-step
update (drop the component along `w_i`, then renormalise); diagram F11.

**Speaker:** the heart of the nugget. The rescaling leaves both the
reconstruction *and* the sparsity count untouched, so an unconstrained optimizer
can drive the loss down by inflating decoder norms rather than learning
anything. Fixing `‖w_i‖ = 1` kills that. But then a plain gradient step pushes
`w_i` off the unit sphere, and renormalising afterwards silently undoes part of
the step — so first drop the component of the gradient along `w_i`, then step,
then renormalise.

**Must defend:**
- Show `⟨g̃_i, w_i⟩ = 0` in one line (expand, use `‖w_i‖ = 1`).
- What breaks without the projection step.
- Why the norm is per **row** of `W_dec`, not per column. (A real bug in
  inherited code took it along the wrong axis — normalising per output
  coordinate across features, which does not prevent the degeneracy.)

> **Deliberately cut:** an earlier draft framed the constraint set as a product
> of 16,384 copies of `S^959` and used the words *tangent space* and *retract*.
> That notation was removed: it conveys nothing beyond "each row has unit norm",
> and it oversells the method, since no Riemannian machinery is used — no
> metric, no geodesics, no exponential map. The F11 diagram carries the geometry
> more honestly. Do not reintroduce it; an applied-maths audience is exactly the
> one likely to ask what the manifold structure is buying.

### Slide 25 — Dead features, and the complete objective
**On slide:** the dead-feature problem; k-annealing as a fragment pair; then the
AuxK equations and the full objective (see `SLIDE_BUILD_SPEC.md` for LaTeX).

**Speaker:** with `k = 64` of 16,384, whichever features win the early race while
weights are still random is close to arbitrary — and a loser never recovers.
k-annealing solves an easier, less sparse problem first and deforms it to the
hard one. AuxK attacks the same problem from the other side: take the features
that have not fired in 400,000 residues, and ask *them* to reconstruct the
residual the live features left behind. A dead feature useful for that gets
gradient again. The stop-gradient matters — the residual is a fixed target, so
the auxiliary term cannot degrade the main reconstruction to make its own job
easier.

**Must defend:**
- Why the slide-22 objective was incomplete, and that **this** is the loss we
  trained.
- What `sg[·]` does and why it is needed.
- Where `1/32` comes from (Gao et al., §A.2).
- What happens to the term when nothing is dead.

### Slide 26 — Training results
**On slide:** FVE vs epoch with a second axis for dead-feature count, annotating
the **2,810 spike at epoch 10** and recovery to **0**. Table: FVE **0.9687** /
0 dead / $\|z\|_0=64$ exactly / 3 h 02 min on a laptop GPU.

**Speaker:** define FVE explicitly —
$$\mathrm{FVE} \;=\; 1-\frac{\big\|x-\hat x\big\|^2}{\big\|x-\bar x\big\|^2}$$
where $\bar x$ is the **training-set mean**, not zero. Then the spike: it lands
exactly where $k$ finishes annealing 256→64, thousands of features lose their
slot at once, and AuxK brings them back. The mechanism and its remedy, caught in
the act.

**Must defend:** what FVE compares against and why not zero (ESM-C activations
have large near-constant coordinates, so a zero baseline inflates FVE for free);
what the 2,810 spike is.

### Slide 27 — So what did we buy
**On slide:** three fragments — "64 of 16,384 numbers reconstruct 96.9%" /
"every dictionary entry used" / "uniform across campaigns, 0.94–0.99".

**Speaker:** that is only possible if the representation genuinely decomposes
into sparse parts. Hand off: *are those parts about anything we care about?*

---

# BLOCK 5 — Results and the design demo (slides 28–34, 13 min, Vignesh)

### Slide 28 — Defining the interface
**On slide:** cartoon — target surface, binder residue, a 5 Å arrow. Fragments:
"min heavy-atom distance to the target" / "≤ 5 Å = contact" / "CAPRI/PISA".
Numbers: "164,285 contact residues / 771,228 = 21.3%", "0 designs skipped".

**Speaker:** note we use real geometry, not the model's PAE confidence — PAE
says how *sure* the model is about a residue pair, a different axis from whether
they are close.

### Slide 29 — Enrichment
**On slide:**
$$E_f \;=\; \frac{P\big(f\ \text{fires}\mid \text{contact}\big)}{P\big(f\ \text{fires}\mid \text{far}\big)}$$
Interpretation: 1 = indifferent, 5 = interface-specific, 0.2 = interface-avoiding.
Measured: **median 1.009, max 5.49, over 13,399 features**.

**Speaker:** a ratio of conditional probabilities, estimated by counting. The
median sitting at 1.00 is the sanity check — most features *should* have no
interface preference, and they don't.

**Must defend:** how each probability is estimated from counts; why a median of
~1 supports the method rather than undermining it.

### Slide 30 — A feature you can read
**On slide:** feature 14364's examples, monospaced, firing residue bracketed:
```
QVKEKIEE[L]TGIAPEDQ    TYR124, VAL128          pLDDT 95.6
DVKKKIEE[L]TGLDPAEQ    TYR124, VAL128, ARG129
QVKEKISE[K]TGLDPKYQ    TYR124, VAL128
EVKERIEE[L]TGIAPEDQ    TYR124, VAL128, ARG129
```
Fragments: "amphipathic helix" / "invariant TG" / "11/15 → same pocket".

**Speaker:** the `TG` is invariant even where the firing residue itself varies.
Glycine breaks helices. So the feature reads as *helix ending in a TG turn,
docked on the Tyr124/Val128 pocket* — a structural hypothesis pulled out of a
black box.

**Must defend:** why Gly breaks a helix; what pLDDT is and why 95.6 matters.

### Slide 31 — A statistical trap *(CUT THIRD)*
**On slide:** naive vs support-filtered top features —
| ranking | top feature | max activation | support |
|---|---|---|---|
| naive | 5403 | 40.8 | **2 residues** |
| filtered | 14364 | 16.5 | 8,052 residues |

**Speaker:** a maximum is an **extreme-value statistic**. Over 16,384 features,
"which has the largest value anywhere" finds whichever had a fluke, not
whichever is strongest typically. Two residues of 164,285 is not a describable
feature. Fix: require ≥50 support before ranking.

### Slide 32 — The surprise *(the headline finding)*
**On slide:** the per-campaign contact table, targeted window vs
actually-contacted windows in different colours:

| campaign | 25–33 | 45–57 | 58–70 | 112–130 |
|---|---|---|---|---|
| `final_25_33` | **17.9%** | 1.6% | 20.4% | 26.2% |
| `final_45_57` | 1.2% | **11.7%** | 18.1% | 37.6% |

Plus the contact-frequency bar chart over all 430,358 contact pairs.

**Speaker:** read it twice. First — the guidance *works*: 17.9 vs 1.6 is an ~11×
contrast, each campaign is enriched for its own hotspot. Second — it is a
*minority* of contacts. Both campaigns put most of their interface on the same
shared surface at 58–70 and 112–130. Most "interface features" therefore
describe that shared sticky epitope, not the hotspot anyone asked for.

**Must defend:**
- Could this be an artifact of the ipTM filter? (Honest: possibly. We gate on
  ipTM, and if designs hitting the intended hotspot score worse, gating would
  preferentially keep off-target binders. We have **not** tested the ungated
  distribution. Say so.)
- Why measuring geometry was necessary to see it at all.

### Slide 33 — **NUGGET 3: the demo** — generate, embed, reconstruct
**On slide, pseudocode (must match the notebook cell that runs):**

```
# submitted at minute 3 of this talk
job      <- BoltzProt1.design(target = Reg3A,
                              hotspot = residues 112..130,   # from slide 32
                              n = 100, length = 50..80)
designs  <- job.results()        # 100 binders the SAE has never seen

mu, s    <- SAE.checkpoint.mean, SAE.checkpoint.scale
for each design in designs:
    x     <- ESMC.hidden_states[23]( binder + "|" + target )[binder span]
    xp    <- (x - mu) * s                       # the SAE's input space
    z     <- TopK_64( W_enc^T xp + b_enc )
    xhat  <- W_dec^T z + b_dec
    FVE(design) <- 1 - ||xp - xhat||^2 / ||xp||^2

plot  histogram of FVE(designs)   against the held-out training FVE = 0.9687
report fraction with ipTM > 0.7   against 28.3% (25-33) and 37.6% (45-57)
```

**Live demo:** run the cell. Two numbers come out.

**Speaker:** this closes the loop. The SAE was trained on 36,710 designs
targeting *other* hotspots. These 100 are brand new, aimed at the patch our
analysis says designs actually prefer. Two questions: does the sparse
decomposition still hold on proteins it has never seen, and does targeting the
preferred patch actually work better?

**Must defend:**
- The FVE formula, including why the denominator is $\|x_p\|^2$ (because $x_p$
  is already centered, so that *is* the error of predicting the training mean).
- Why $\|z\|_0=64$ still holds on new data (TopK is applied at inference, not
  learned).
- **What this demo does NOT show:** it does not validate any feature's
  *interpretation*, and the ipTM comparison is **uncontrolled** — n=100, a
  different hotspot, possibly different settings from the historical campaigns.
  Suggestive, not conclusive. Say this before being asked.

### Slide 34 — What the demo returned
**On slide:** leave space for two live numbers — the FVE histogram with 0.9687
marked, and the ipTM pass rate next to 28.3% / 37.6%.

**Speaker:** interpret whatever came back honestly, including a null result. If
FVE holds near 0.97 the decomposition generalizes. If the ipTM rate does not
beat the baselines, say so — it is an uncontrolled comparison either way.

---

### Slide 35 — Conclusions (2 min, all three)
**On slide:** three fragments — "64 of 16,384 features reconstruct 96.9%" /
"features are legible — motifs docking specific pockets" / "and they point at an
epitope nobody targeted".

**Speaker:** one sentence each presenter on their own block.

---

## BACKUP SLIDE (not presented — for Q&A only)

**Trigger:** "did you try *using* the features to steer design?"

**On slide:** "Yes — it does not work (yet)". Then:
| | |
|---|---|
| chance expectation | 16.5 of 32 |
| observed mean rank of the correct feature | **15.72** (p = 0.41) |
| ranked first | 3 of 96 — chance is exactly 3.0 |

**Speaker:** inject a feature's direction at layer 23, then ask whether the
*correct* feature beats the other 31 at producing its own motif. It does not. A
real direction beats random noise (p = 3e-6) but so does any real direction —
that is on-manifold geometry, not feature identity. Six measurement artifacts
turned up along the way and **five of six initially pointed toward "steering
works"** — which is the real lesson. Scope: decoder directions, layers 23–29,
α from 2 to 150. Untested: encoder directions, optimizing in SAE space during
generation.

---

## Live demo notebook — build spec

`demo/seminar_demo.ipynb`. Every cell under ~10 s except the Boltz-Prot-1 job,
which is submitted in block 2 and read in block 5.

| cell | block | content | cached input |
|---|---|---|---|
| 1 | 2 | submit the Boltz-Prot-1 job, print the job id | API credentials |
| 2 | 4 | load SAE checkpoint, print config, show $\|z\|_0=64$ on random input | `best.pt` (362 MB) |
| 3 | 4 | FVE + dead-feature curve, epoch-10 spike annotated | `results/training_log_layer23_paired.csv` |
| 4 | 5 | print feature 14364's examples with contacts | `results/interface/top_examples.csv` |
| 5 | 5 | naive vs support-filtered ranking | `results/interface/feature_summary.csv` |
| 6 | 5 | per-campaign contact table + bar chart | cached contact counts |
| 7 | 5 | **N3** — fetch job, embed 100 designs, FVE histogram, ipTM rate | job id + `best.pt` + ESM-C weights |

**Pre-flight, day before:**
- run every cell top to bottom on the presentation machine, **offline**
- pre-download ESM-C weights (~1.3 GB) — do not rely on the venue network
- do one full rehearsal submission of the Boltz-Prot-1 job to confirm
  credentials, quota, and the ~13 min turnaround
- **pre-generate a fallback 100-design job** and cache its results locally; if
  the live job fails, use it and say so

**Fallback:** export every cell's output to a static PDF, open in another tab.

---

## Anti-slop rehearsal list

**Block 3 / N1 (Jun)**
1. Derive $\mathbb{E}_M[\log P(x_i\mid x_{-i})] = \mathrm{PLL}(x)/L$ for single-token masking.
2. Why is 15% masking an approximation, and what does it buy?
3. Why is there no exact sequence likelihood for a bidirectional model?
4. Derive $\mathrm{Var}(q\cdot k)=d$ and explain the $\sqrt{d}$.
5. Why could we not co-attend binder and target with a causal model?
6. We never use ESM-C's predictions — so why does its training objective matter
   to us at all?

**Block 4 / N2 (Parsa)**
7. Show $\langle \tilde g_i, w_i\rangle = 0$.
8. State the rescaling degeneracy and why the unit-norm constraint kills it.
9. What does TopK do to gradients of unselected coordinates?
10. What does FVE compare against, and why not zero?
11. What is k-annealing for? What is AuxK for? Write the full objective.
12. Why 16,384 and 64 — what would change at 4,096, or $k=16$?

**Block 5 (Vignesh)**
13. Estimate both probabilities in $E_f$ from counts, out loud.
14. Why is a maximum a bad ranking statistic across many features?
15. Why geometric distance rather than the model's own PAE?
16. Could the hotspot finding be an ipTM-filter artifact? (Yes, possibly —
    untested. Own it.)
17. Why does Gly break a helix?
18. What does the demo *not* show?

---

## Figures to generate

All from this repository's own data. Attribution line "our data" on each.

| # | figure | source |
|---|---|---|
| F1 | binder–target complex render | any gated design `.cif` |
| F2 | ipTM histogram, 0.7 cut marked | `results/manifest_combined.csv` |
| F3 | superposition cartoon (2D) | hand-drawn |
| F4 | FVE + dead-features vs epoch, spike annotated | `results/training_log_layer23_paired.csv` |
| F5 | 5 Å contact-definition cartoon | hand-drawn |
| F6 | feature 14364 example block | `results/interface/top_examples.csv` |
| F7 | naive vs filtered ranking table | `results/interface/feature_summary.csv` |
| F8 | per-residue contact-frequency bar chart | geometry cache |
| F9 | per-campaign contact table, coloured | README §6 |
| F10 | **live** FVE histogram of 100 new designs vs 0.9687 | demo cell 7 |
| F11 | cross-injection rank histogram (backup slide only) | `results/steering_cross/cross_injection.csv` |
