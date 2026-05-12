---
layout: center
---

# Protein language models learn evolutionary statistics of interacting sequence motifs
##### Zhidian Zhang, Hannah K. Wayment-Steele, Garyk Brixi, and Sergey Ovchinnikov 


---
layout: center
---

# Multiple Sequence Alignment (MSA)

<!--

Před tím si ale musíme vysvětli co to je MSA a proč je to tak důležité?
-->

---

# What is Multiple Sequence Alignment ?

MSA is the process of aligning *biological* sequences to identify regions of similarity that may be a consequence of evolutionary relationships.

<MsaVisualizer :clicks="$slidev.nav.clicks" />

<div v-click></div>

<!--
MSA nám umožňuje vidět evoluci v přímém přenosu. 
1) Nejdřív máme sekvence "vypadající" podobně, ale neshodují se v pozicích.
2) Po zarovnání vidíme vertikální sloupce stejných nebo podobných aminokyselin. 
To jsou ta místa, která jsou pro protein důležitá – pokud by tam došlo k mutaci, protein by přestal fungovat.
-->

---

# Why is MSA powerful for PLMs?

While single-sequence models (like ESM-2) must *infer* evolution, MSA-based models have it *explicitly*.

<div class="grid grid-cols-2 gap-4">
<div>

- **Input**: A stack of related sequences.
- **Context**: Co-evolutionary signals.
- **Accuracy**: Extremely High for contact prediction.
- **Models**: MSA Transformer, AlphaFold (?).

</div>
<div class="flex items-center justify-center">
  <img src="/images/MSA.png" class="rounded shadow-lg border border-gray-200" />
</div>
</div>

<!--
- MSA modely jako MSA Transformer berou jako vstup celou tuhle "kostku" dat. 
- To je ten důvod, proč byl AlphaFold tak úspěšný – MSA je extrémně silný prediktor struktury.
- ale je tady jedna nevýhoda: Model se neučí predikovat stukturu ale dělá "navlékaní", dekompozici
- jde vlastně o to že se naučí HMM vidím "X" to se vždycky složí na X
- Tento paper vlastně chce tak zhruba dokázat že model co bere jen jako sekvenci na vstupu dělá vlastně uvnitř MSA, a potom dělá to co MSA based model
-->



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

To test Hypothesis 1, the authors used **protein isoforms** (sequences formed from alternative splicing that disrupt ordered domains).

- Isoform sequences are missing parts of the full structure and are generally unfolded in reality.
- **AlphaFold2, OmegaFold, and ESMFold** predict them as rigid, structured fragments.
- These models leave nonphysical patches of hydrophobic residues completely exposed (high SAP scores).

**Conclusion:** pLMs do not simulate biophysics.

<!--
Aby otestovali první hypotézu o fyzice, použili chytrý chyták – izoformy. 
Jde o proteiny, u kterých evoluce nebo sestřih vyřízly část sekvence. V reálné biologii by se takový nekompletní protein vůbec nesložil.
Ale AF2 i ESMFold takový protein nesmyslně poskládají jako by byl kompletní, a nechají na povrchu odhalené hydrofobní vrstvy, což fyzikálně nedává smysl. Modely tedy fyzice nerozumí.
-->

---
layout: center
---

# The "Categorical Jacobian"

How do we extract coevolutionary signal from ESM-2 in a completely *unsupervised* way?

<JacobianVisualizer :clicks="$slidev.nav.clicks" />

<div v-click></div>
<div v-click></div>
<div v-click></div>

<!--
Když tedy modely nepočítají fyziku, jak z nich dostat to, co si skutečně myslí o evoluci? Autoři vyvinuli metodu "Kategorický jakobián".
Je to metoda bez učitele. Postupně zmutují každou pozici v proteinu na všech 20 aminokyselin a sledují, jak moc tato změna ovlivní zbytek proteinu. Získají tak obrovskou matici, která mapuje všechny párové závislosti.
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