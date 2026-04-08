# Project: Prediction of Cryptic Binding Sites Using Protein Language Models

## Introduction

Accurate prediction of protein-ligand binding sites is crucial for drug discovery, protein engineering, and understanding biological functions at the molecular level. Binding site prediction can be performed based on either the three-dimensional structure of a protein or its sequence. Traditionally, structure-based methods have been more commonly used, as binding is governed by the structural and physico-chemical complementarity between the protein and its small molecule binding partner.

However, structure-based approaches face inherent challenges because they are typically trained and evaluated on holo (ligand-bound) structures. In these cases, the ligand is removed, leaving a clear cavity in the protein structure, which is then detected by the prediction method. This creates a bias, as methods trained on such data often struggle to predict pockets in apo (ligand-free) protein forms, where the binding site may be obscured or, conversely, too open for the ligand to bind. Binding sites that exhibit different conformations in apo and holo forms are referred to as cryptic binding sites.

Most existing protein-ligand binding site prediction methods are biased toward detecting holo-binding sites because current benchmarks and training datasets predominantly consist of holo structures. As a result, these methods are often ineffective at identifying cryptic binding sites. To address this limitation, a novel dataset and benchmark called [CryptoBench](https://osf.io/pz4a9/) has been introduced, built upon the [AHoJ-DB](https://apoholo.cz/db) — a comprehensive database of apo-holo protein conformations. CryptoBench focuses on apo-holo protein pairs with significant structural rearrangements in their binding sites, and evaluates methods at the level of individual **binding residues** — i.e., predicting which residues in the apo structure are part of a cryptic binding site.

While residue-level predictions are a natural starting point, many downstream tasks — such as docking a small molecule into a predicted pocket, or scoring the druggability of a site — require the identification of coherent, spatially compact **binding regions** rather than isolated residue labels. This is the gap this project sets out to address.

One promising family of approaches for binding residue prediction uses pretrained protein language models (PLMs), such as [ESM-2](https://github.com/facebookresearch/esm), which produce rich per-residue sequence embeddings. These embeddings can be leveraged either in a transfer learning setting — training a lightweight prediction head on top of frozen PLM embeddings — or through fine-tuning the PLM itself to directly predict whether a residue belongs to a cryptic binding site. One such approach was explored in the CryptoBench paper and shown to outperform existing state-of-the-art methods on the benchmark.

## Goal

The goal of this project is to take per-residue CBS predictions from a fine-tuned PLM (provided by the organizers) and develop a method that **clusters predicted binding residues into discrete binding sites**. The predicted sites should then be evaluated against the true binding sites obtained from the holo forms of the same proteins.

Concretely, the pipeline you will build consists of the following steps:

1. Develop and optimize a clustering strategy using the **train set**: given per-residue binding site probabilities produced by the CBS model, group high-scoring residues into spatially coherent binding site predictions and tune your approach against the ground truth annotations.
2. Once the pipeline is finalized, evaluate it on the **test set** to obtain unbiased performance estimates.

**How clustering is performed is entirely up to your team.** Some directions worth exploring:

- *Geometry-based clustering*: cluster residues based on their 3D Cα distances (e.g., DBSCAN, hierarchical clustering), using the apo structure as input.
- *ML-based clustering*: train a classifier that, given a pair of residues and their features (PLM embeddings, spatial proximity, predicted scores), decides whether they belong to the same binding site. This can be framed as a graph problem — construct a residue graph and perform community detection or learned edge prediction.
- *Hybrid approaches*: combine geometric constraints with learned scoring.

## Evaluation

Predicted binding sites will be evaluated using the following criteria (consistent with the CryptoBench benchmark):

- **DCC (Distance from the Center of the Closest Binding Site)**: measures whether the center of a predicted pocket falls within a given distance threshold of the true binding site center. While 4 Å is commonly used, the [LIGYSIS paper](https://academic.oup.com/nar/article/53/D1/D545/7905468) suggests 12 Å as a more permissive and practical threshold — we adopt 12 Å as our recommended criterion.

  Since a protein can contain multiple binding sites, the DCC criterion is evaluated in a **top-N** or **top-N+2** setting: if a protein has N true binding sites, one tests how many of them are correctly recovered (i.e., matched within the DCC threshold) among the top N or top N+2 predicted pockets, ranked by prediction confidence. This rewards methods that prioritize true sites highly without penalizing reasonable additional predictions.

- **Relative Residue Overlap (RRO)**: measures the fraction of true binding site residues that are covered by the predicted site, normalized by the size of the true site.

Both metrics should be computed per structure and aggregated across the dataset (e.g., success rate at the DCC threshold, mean RRO).

## Optional Extension: Visualization with PyMOL

As an optional output, teams are encouraged to produce visualizations of predicted pockets directly on the apo protein structures using [PyMOL](https://pymol.org/). Visualizing predicted vs. true sites side-by-side is a powerful way to interpret and communicate results. PyMOL can be scripted in Python, making it straightforward to integrate into the prediction pipeline.

## Can You Beat Us?

Our team has recently developed a pipeline tackling exactly this problem — predicting and clustering cryptic binding sites from PLM-based residue scores. The approach and results are described in [our preprint](https://www.biorxiv.org/content/10.64898/2026.01.28.702257v1.full.pdf). We encourage you to read it, understand what we did, and then try to beat us. 🙂

## Data and Models Provided

The following resources will be provided by the organizers:

- **Train and test splits** from the CryptoBench benchmark, containing PDB IDs of apo-holo structure pairs along with the corresponding binding site residue annotations (ground truth labels). The full dataset is also available from the [CryptoBench OSF project site](https://osf.io/pz4a9/).
- **CBS model**: a fine-tuned protein language model that outputs per-residue probabilities of belonging to a cryptic binding site, along with instructions for running inference.
- **Embedding extraction script**: a script demonstrating how to obtain raw PLM residue embeddings from the CBS model, in case your pipeline makes use of these embeddings (e.g., as features for an ML-based clustering step). The underlying ESM-2 embedding generation code is available at [this repository](https://github.com/skrhakv/esm2-generator).

## Useful Resources

- Protein structure handling: BioPython's [Bio.PDB package](https://biopython.org/docs/1.75/api/Bio.PDB.html) — a short tutorial is available [here](https://biopython.org/DIST/docs/tutorial/Tutorial.html#sec240) | [Biotite](https://www.biotite-python.org/latest/)
- CryptoBench benchmark and evaluation: [Paper](https://academic.oup.com/bioinformatics/article/41/1/btae745/7927823) | [OSF dataset](https://osf.io/pz4a9/).
- AHoJ-DB (apo-holo structure pairs): [apoholo.cz/db](https://apoholo.cz/db).
- PyMOL scripting documentation: [PyMOL wiki](https://pymolwiki.org/index.php/Main_Page).

## How to run the finetuned ESM2 model
Download the model from [this link](https://cunicz-my.sharepoint.com/:u:/g/personal/99575159_cuni_cz/IQDTozFMT5pMQoWIaHzqQmQ5ASbN36kTO4OUv2GFxuu2pno?e=iOakb4). 

We have tested installation pipeline using CONDA environment:
```
conda create -n "run_esm2" python=3.12.12
conda activate run_esm2
python3 -m pip install -r requirements.txt
```
Nevertheless, fresh environment with Python3.12 should also work without problems.

