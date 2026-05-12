# What are Proteins?

Proteins are large, complex molecules that play many critical roles in the body.

- **Building Blocks**: Made up of hundreds or thousands of smaller units called amino acids.
- **Structure**: They do most of the work in cells and are required for the structure, function, and regulation of the body's tissues and organs.
- **Diversity**: There are 20 different types of amino acids that can be combined to make a protein.

---

# What do Proteins do? (Functions)

Proteins are the "workhorses" of the cell, performing almost every task.

| Category | Function | Example |
| --- | --- | --- |
| **Enzymes** | Catalyze biochemical reactions | DNA Polymerase |
| **Transport** | Carry substances throughout the body | Hemoglobin |
| **Structure** | Provide support | Collagen |
| **Signaling** | Transmit signals between cells | Insulin |
| **Defense** | Protect against pathogens | Antibodies |

---

# What are they made of? 

Proteins are polymers built from a basic set of building blocks.

- **The Alphabet of Life**: There are **20 standard amino acids**.
- **Polypeptide Chain**: These blocks are linked together in a specific order (sequence).
- **Infinite Variety**: Just like letters in a language, the specific sequence determines the protein's unique identity and role.

---

# How do they assemble? (Peptide Bond)

Amino acids connect through a dehydration synthesis reaction.

<PeptideFormation :clicks="$slidev.nav.clicks" />

<div v-click></div>
<div v-click></div>

<div class="mt-5 text-center h-10 flex items-center justify-center font-serif italic text-gray-400 text-sm">
  <div v-if="$slidev.nav.clicks === 0">Starting point: Two separate amino acids.</div>
  <div v-if="$slidev.nav.clicks === 1">Reactive groups (Carboxyl and Amino) are highlighted.</div>
  <div v-if="$slidev.nav.clicks === 2">The molecules join, releasing water to form a covalent bond.</div>
</div>

---

# The Four Levels of Protein Structure

The sequence of amino acids determine the final 3D shape.

1. **Primary Structure**: The linear sequence of amino acids.
2. **Secondary Structure**: Local folding patterns (alpha-helices and beta-sheets).
3. **Tertiary Structure**: The overall 3D shape of a single polypeptide.
4. **Quaternary Structure**: Arrangement of multiple protein subunits.

<!--
Snížení komplexity pokud to nepotřebuju.

valence shell
-->

---

# Properties Drive Folding

Once linked into a chain, the individual properties of amino acids determine how the protein "collapses" into a 3D shape.

<div class="grid grid-cols-2 gap-8 mt-10">
<div>

### Hydrophobic Effect
- **"The Shy ones"**: Non-polar amino acids "hate" water.
- **Internal Core**: They aggregate in the center of the protein to hide from the aqueous environment.
- **Stability**: This is the main driving force for folding.

</div>
<v-click>
<div>

### Hydrophilic Shield
- **"The Social ones"**: Polar and charged amino acids "love" water.
- **Surface**: They stay on the outside, interacting with the environment and protecting the core.

</div>
</v-click>
</div>



---
layout: image-right

# the image source
image: ../images/PMC5221515_13_295f4.png
backgroundSize: contain
---


# Why not just simulate it?

If we know the physics (Atoms, Forces, Energy), why not just "run" a simulation?

- **The Time Scale Problem**: Protein folding happens in milliseconds ($10^{-3}$s). However, simulation steps are in femtoseconds ($10^{-15}$s).
- **Computational Cost**: Simulating a single protein molecule is possible at good enoug precision is possible on supercomputer for ~40 AA proteins.  


<!--
Většina vlastností, funkcí a podobných proteinu se odvyjí od protein foldingu. proč neuděla simulaci, potom to bude už ez ?
- Někde jsem četl že protein folding je NP uplný problém a protein folding je zavislý na kvatových jevech což značně koplikuje vše
- potom je problém že najít data na validaci je dost hard kvůli tomu že pochází ze zdrojů které nejsou realné :(
-->

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
# From Biology to Language

If we look at a protein as a 1D sequence of amino acids, it starts to look like a language.

- 📝 **The First Step**: Researchers simply took standard NLP tokenizers, forced them to treat each amino acid as a single token, and trained models on massive protein databases.
- 🏗️ **Architecture Exploration**: Soon after, seminal papers (like *ProtTrans*) systematically tested almost every major NLP architecture:
  - **BERT** (Encoder-only)
  - **T5** (Encoder-Decoder)
  - **XLNet**, **RoBERTa**, **Albert**, and others.
- ✨ **"Success"**: These models effectively "learned" biology. They can predict 3D properties, solubility, and even protein-protein interactions just by analyzing the sequence "grammar".

<!--

1) Natrenovali to jenom tak random, moc se s tím nesrali ale i tak to fungovalo
2) první takový pokus o pořádný LM byl trénovaný nad všemi proteiny, což je problém kvůli tomu mega předtrenovaná. Vysoká schoda v sekvencích. Většina proteinu má v databázi X prakticky stejných proteinu. 
je to overfitnute na proteiny co lidi "potřebuju scanovat", takže typicky věci co se používají jako lidské proteiny, proteiny "Háďátka". 
3) PLM neřeší jinou než 1 strukturu

lidské proteiny
Kmitající proteiny
-->
