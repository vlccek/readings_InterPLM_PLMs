---
layout: center
---
# InterPLM: Discovering Interpretable Features in Protein Language Models via Sparse Autoencoders

<!--
Jde o to že udělali framework na interpretaci Features z PLM. 

-->
---


## Introduction & Motivation
*   **The "Black Box" Problem:** Protein language models (PLMs) like ESM-2 achieve impressive performance in predicting protein structure and function, but their internal mechanisms are poorly understood. Furthermore, I'm suggesting that interpreting a language we don't know is even harder than with natural language (NL). 
*   **Research Goal:** To create a systematic framework using Sparse Autoencoders (SAEs) to decompose these complex, tangled representations into clear, understandable biological features.



<!--

Jde o model ESM-2, což je aktuálně asi nejlepší free to use model. 
- stejný problém jak klasick LLM
- SAE
-->

---

## Methodology: Sparse Autoencoders (SAEs)
*   **The Tool:** SAEs are trained on the hidden amino acid embeddings of the ESM-2 model.
*   **Experiments:**
    - **`ESM2-8M` (6L):** 320 neurons expanded into 10,420 latent features (32x)
    - **`ESM2-650M` (33L):** Same 10,422 latent features (8x expansion)


---
layout: center
---

# Interpretability: ESM-2-8M (Neurons vs. SAE)

<img src="/images/esm28-f1.png" class="h-90 mx-auto rounded shadow" />

<div class="mt-4 text-center text-sm opacity-60">
  <span class="text-green-500 font-bold">Green:</span> Random Weights | 
  <span class="text-blue-500 font-bold">Blue:</span> Neurons | 
  <span class="text-pink-500 font-bold">pink:</span> SAE Features
</div>

<!--

Zelene je RND vahy ESM, modrá je přímo interpretovatelnost z neuronu a potom 
F1>0.7
-->
---
layout: center
---

# Interpretability: ESM-2-650M

<img src="/images/esm2650-f1.png" class="h-90 mx-auto rounded shadow" />

<div class="mt-4 text-center text-sm opacity-60">
  <span class="text-blue-500 font-bold">Blue:</span> Neurons | 
  <span class="text-pink-500 font-bold">pink:</span> SAE Features
</div>

<!--

Zelene je RND vahy ESM, modrá je přímo interpretovatelnost z neuronu a potom 
-->
---

## SAEs vs. Neurons

*   Standard neurons identified **only 15** distinct biological concepts.
*   SAE features on the same model successfully isolated **143 concepts**.
*   The larger ESM-2-650M model captured significantly more concepts (over **420**), proving that larger models learn richer representations.

<!--
F1>0.7

1. Roztřídili je do proteinových rodin podle databáze InterPro. Následoval tento proces:

    Pro každý objevený rys se podívali na sekvenci, která u něj vyvolává úplně nejvyšší aktivaci, a zjistili, do jaké "InterPro" rodiny tato sekvence patří.
    Následně zkusili rys použít jako binární klasifikátor a ptali se: "Lze na základě toho, jak moc se tento rys aktivuje, s jistotou určit, že protein patří právě do této rodiny?".
    Algoritmus postupně testoval různé hranice aktivace (od 0,1 do 0,9) a měřil úspěšnost pomocí takzvaného F1 skóre.
    Pokud byla předpovědní schopnost rysu velmi vysoká a dosáhla F1 skóre většího než 0,7, autoři jej oficiálně označili jako "specifický pro danou rodinu" (family-specific).

-->

---
layout: image-right
image: /images/umap-sae.png
backgroundSize: contain
---

##  Functional Clustering
*  **UMAP & HDBSCAN** applied on top of the SAE features.
*  Features with similar dictionary vectors naturally cluster together by biological function.

<!--
Exponenciální růst objemu, vzdálenosti mezi všemi body začínají vyrovnávat, 
-->
---

## Automated Annotation via LLMs
*  Less than **20%** of discovered features perfectly match existing Swiss-Prot database labels.
*  Used **Claude-3.5 Sonnet** to automatically generate descriptions for discovered features based on protein metadata and activation patterns.

---
layout: center
---

# LLM for Interpretation of SAE Features

<img src="/images/llm_pipline.png" class="mx-auto rounded shadow" />

<!--
1) Dáme LLM 40 proteinu u kterýh nějákou SAE featura je aktivovana na různých mírách. Dostaneme popis s tím co tato feature koduje
2) Dáme LLM popis generovany, a chceme popis SAE pro ten protein
3) porovná se pSEA a SAE co dal plm

-->
---


## LLM as Interpreter of SAE Features
* Many apparent "false positive" activations were actually correct model predictions uncovering missing labels in human databases. 



<!--
- u rysu co víme že dokáže PLM detekovat tak víme že je detekuje hodně dobře, často to nechází chyby v anotacích

-->

---

# Steering
- encoder model, and steering? 
*   **Targeted Steering:** By manually manipulating feature activations, researchers can causally influence the model's sequence generation (e.g., successfully forcing the generation of periodic glycines in collagen-like patterns).

<!--
Hodně PoC, ale jde vlastně o to že mohu dělat mírné změny i v enkoderu ale jsem limitován tím že musím mít stejné delku atd. a moc to reálně nejde 
- ale JDE TO PYCE
-->

---