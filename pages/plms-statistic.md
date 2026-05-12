---
layout: center
---

# Protein language models learn evolutionary statistics of interacting sequence motifs
##### Zhidian Zhang, Hannah K. Wayment-Steele, Garyk Brixi, and Sergey Ovchinnikov 





---
layout: center
---

# Do MSA-free models learn the physics of folding?

Single-sequence structure predictors like **OmegaFold** and **ESMFold** do not use MSA as an input.

**The central question:** 
Have these protein language models (pLMs) learned the intrinsic biophysics of folding a single amino acid sequence, or are they implicitly looking up stored evolutionary information?

<!--
Přecházíme k hlavní otázce tohoto článku. 
Nové modely jako ESMFold nebo OmegaFold už na vstupu MSA nepotřebují. Znamená to, že se jazykové modely konečně naučily skutečnou fyziku skládání proteinů? Nebo jen dělají nějaký skrytý trik a "vytahují" si z paměti evoluční data, která viděly při tréninku?
-->

---
layout: default
---

# Three Hypotheses for pLM Functionality

The authors proposed three distinct hypotheses for how ESM-2 predicts protein structure:

1. **Hypothesis 1:** Fold protein based on physics (energy landscape).
2. **Hypothesis 2:** Look up complete folds (stores a separate coevolution model for each full protein family).
3. **Hypothesis 3:** Look up segment pairings (stores small, independent models for pairs of interacting fragments/motifs).

<!--
K zodpovězení této otázky si autoři stanovili tři hypotézy.
První: Model se naučil skutečnou fyziku a minimalizuje energii.
Druhá: Model si pamatuje celé tvary (folds) pro každou proteinovou rodinu a jen sekvenci přiřadí ke správné rodině.
Třetí: Model si pamatuje statistiky pro malé interagující segmenty (motivy) a ty k sobě modulárně skládá.
-->

---
layout: default
---

# Debunking Physics: The Isoform Trap

To test Hypothesis 1, using the **protein isoforms** (sequences formed from alternative splicing that disrupt ordered domains).

- Isoform sequences are missing parts of the full structure and are generally unfolded in reality.
- **AlphaFold2, OmegaFold, and ESMFold** predict them as rigid, structured fragments.
- These models leave nonphysical patches of hydrophobic residues completely exposed (high SAP scores).

<!--
Aby otestovali první hypotézu o fyzice, použili chytrý chyták – izoformy. 
Jde o proteiny, u kterých evoluce nebo sestřih vyřízly část sekvence. V reálné biologii by se takový nekompletní protein vůbec nesložil.
Ale AF2 i ESMFold takový protein nesmyslně poskládají jako by byl kompletní, a nechají na povrchu odhalené hydrofobní vrstvy, což fyzikálně nedává smysl. Modely tedy fyzice nerozumí.
-->

---
layout: default
---

# From 4D Tensor to 2D Contact Map

How do we convert the massive $L \times 20 \times L \times 20$ Categorical Jacobian into a clean $L \times L$ contact map?

**Step 1: The Frobenius Norm (Collapse the chemistry)**
For each pair of positions $(i, j)$, we have a $20 \times 20$ table of interaction weights. We collapse this into a single "interaction strength" score by taking the square root of the sum of squared weights:
$$m_{ij} = \sqrt{\sum_{n=1}^{20} \sum_{m=1}^{20} W[i, n, j, m]^2}$$

**Step 2: Average Product Correction (APC)**
The raw $L \times L$ map contains background noise (e.g., highly reactive exposed residues). We subtract this systemic noise using APC:
$$APC(i, j) = m_{ij} - \frac{\sum_{i'} m_{i'j} \sum_{j'} m_{ij'}}{\sum_{i',j'} m_{i'j'}}$$

**Result:** A highly accurate 2D contact map (Precision @ L/2 = 0.80 for ESM-2 3B).

<!--
Přesouváme se k tomu, jak z toho gigantického 4D tenzoru uděláme naši finální 2D mapu. 

První krok je tzv. Frobeniusova norma [1]. Pro každou myslitelnou dvojici pozic v proteinu máme tabulku 20x20 hodnot – to jsou všechny chemické kombinace aminokyselin. Abychom z toho udělali jedno číslo, zkrátka všechny tyto hodnoty umocníme na druhou, sečteme je a uděláme z nich odmocninu [1]. Získáme tak hrubou sílu vazby mezi pozicí "i" a pozicí "j".

Druhý krok je APC, neboli Average Product Correction [1, 2]. Ta hrubá mapa má totiž problém – některé aminokyseliny, typicky ty na povrchu, tvoří spoustu "šumu", protože se zdají reagovat se vším. APC funguje tak, že pro každou buňku spočítá průměrný šum jejího řádku a sloupce a tento šum matematicky odečte [2].

Výsledkem je čistá 2D mapa kontaktů. A jak už jsme si řekli dříve, přesnost této mapy spočítané z ESM-2 překonává starší lineární modely a dosahuje skóre 0.80 [3, 4].
-->


---

# The "Categorical Jacobian"

How do we extract coevolutionary signal from ESM-2 in a completely *unsupervised* way?

The **Categorical Jacobian** measures how a change in the input amino acid at all position affects the output logit at all position :

**Aggregate**: The result is an $L \times 20 \times L \times 20$ tensor capturing all pairwise dependencies.

<!--
Když tedy modely nepočítají fyziku, jak z nich dostat to, co si skutečně myslí o evoluci? Autoři vyvinuli metodu "Kategorický jakobián".
Místo složitých animací je tu čistá matematika: Jakobián je v podstatě matice parciálních derivací. Protože jsou aminokyseliny diskrétní, počítá se to jako rozdíl v logitech (probabilities) před a po mutaci.
Získáme tak mapu, která říká: "Když změním tohle tady, co se stane s tamtím támhle?".
-->

---
layout: default
---

# ESM-2 vs. Linear Models

The Categorical Jacobian allows direct comparison between the non-linear ESM-2 and classic linear models (Markov Random Fields / Multivariate Gaussian).

- **High Correlation:** The underlying weight matrices between linear models and ESM-2 show striking visual similarities and high correlation.
- **Beyond MSA:** Unlike standard MSAs which lack data for unobserved mutations, pLMs infer pairwise coupling for *every* residue type at *every* position.
- **Accuracy:** The Jacobian approach on ESM-2 outperformed classic linear models in long-range contact accuracy (0.80 vs 0.67).

<!--
Proč to dělali? Tento Jakobián jim umožnil přímo srovnat ESM-2 s klasickými lineárními modely, jako jsou Markovova náhodná pole, která se standardně používají na MSA.
Zjistili, že ESM-2 dělá v podstatě to samé, obě mapy si jsou vizuálně velmi podobné. ESM-2 je ale lepší, protože si dokáže "domyslet" i závislosti pro mutace, které ve skutečných datech MSA nikdy nebyly pozorovány.
-->

---
layout: default
---

# Testing Context: The Masking Experiment

To differentiate between Hypothesis 2 (full folds) and Hypothesis 3 (segment pairings), the authors masked the sequence.

- Masked the whole sequence except for two 11-aa interacting segments.
- **Strategy A:** Unmask local flanking regions.
- **Strategy B:** Unmask random regions globally.
- **Result:** ESM-2 recovers contacts roughly 3 times more effectively when unmasking local flanking regions compared to random global unmasking.

<!--
Jak rozhodnout mezi tím, jestli model hledá celé rodiny (Hypotéza 2), nebo jen malé kousky (Hypotéza 3)?
Zamaskovali celý protein a nechali jen dva malé interagující kousky. Pak zkoušeli odkrývat buď lokální sousedství těchto kousků, nebo náhodně odkrývat zbytek proteinu.
Zjištění bylo jasné: Model mnohem lépe obnovil predikci kontaktu, když viděl lokální kontext (přilehlé aminokyseliny), než když viděl zbytek globální struktury.
-->

---
layout: center
---

# The Step-Function "Jump"

- ESM-2 exhibited a striking step-function behavior in contact recovery.
- E.g., for the SusD protein, using 13 flanking residues yielded zero contact probability. Unmasking exactly the 14th residue caused complete contact recovery.
- **Conclusion:** Validates **Hypothesis 3**. pLMs need a specific, discrete window of context to recognize interacting sequence motifs.

<!--
Tohle je naprosto fascinující experimentální důkaz. U mnoha kontaktů autoři pozorovali doslova skokové chování.
Třeba u proteinu SusD: Odkryli 13 aminokyselin kolem motivu a model kontakt neviděl. Odkryli jednu další (čtrnáctou) a jistota modelu okamžitě vyskočila na maximum. 
To jasně potvrzuje Hypotézu 3 – model si pamatuje přesně ohraničené motivy a potřebuje vidět jejich kompletní okraje, aby je mohl propojit.
-->

---
layout: default
---

# Conclusion: The Compression Engine

- **Not physics oracles:** Single-sequence pLMs approximate structures using stored evolutionary statistics, not physics calculations.
- **Motif-based compression:** Storing coevolution for all UniProt families would require ~261 billion parameters.
- By segmenting proteins into common modular motifs, ESM-2 (at 3 billion parameters) highly compresses this evolutionary data.

<!--
Závěrem, tento článek ukazuje, že single-sequence modely nejsou fyzikální orákula.
Místo toho fungují jako geniální kompresní algoritmy. Kdyby si měly pamatovat evoluci úplně pro každou proteinovou rodinu, potřebovaly by přes 260 miliard parametrů.
Tím, že si místo celých proteinů pamatují jen menší společné motivy a to jak se tyto motivy párují, dokáže se to vše ESM-2 naučit "jen" do 3 miliard parametrů.
-->