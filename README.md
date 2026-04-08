# DEEPLIFE 2026
This repository provides data and supporting code for two DEEPLIFE 2026 projects. Both projects utilize the recently published dataset of cryptic binding sites (CBS) named [CryptoBench](academic.oup.com/bioinformatics/article/41/1/btae745/7927823). 

![](https://gregoiresu.github.io/deeplife4eu.github.io/images/DL_logo_2026.png)

## Projects
1. **Cryptic sites ensemble**: Detecting CBS by sampling protein conformational space such as [BioEmu](https://www.science.org/doi/10.1126/science.adv9817) and applying a structure-based predictor [P2Rank](jcheminf.biomedcentral.com/articles/10.1186/s13321-018-0285-8).
2. **Cryptic sites pLM**: Developing clustering strategies for [finetuned ESM2 model](https://dl.acm.org/doi/10.1145/3765612.3767221) outputs to predict CBS.

## Structure
1. `cryptic-sites-ensemble/` & `cryptic-sites-plm/`: Project-specific instructions. 
2. `data/`: Preprocessed train/test subsets of CryptoBench.  
3. `src/`: 
   - `DCC.py`, `RRO.py`: Evaluation metrics
   - `example.ipynb`: Metric calculation walkthrough
   - `run-model.ipynb`: Inference guide for the fine-tuned ESM2 model. 

## Data and data format
CIF files for the PDB structures can be downloaded from [this link](https://cunicz-my.sharepoint.com/:u:/g/personal/99575159_cuni_cz/IQDMhZp4eYdDSLxviELorPf7AfGqT3yJ1HS-zFWVDjMXXe4?e=KUeRMM).


Data entries follow a semicolon-delimited format:
```
PDB_ID;Chain;Ligand_IDs;Residue_Indices
```

If interested in any aditional information regarding the CryptoBench dataset, you can download the dataset from the [OSF repository](https://osf.io/pz4a9/files/osfstorage), though this step should not be neccesary for the scope of this project (read the description of the OSF's repository how to deal with a certain front-end bug in OSF).

### Example 
```
4pfs;B;ANP ADP;19 20 21 22 23 24 25 26 48 135 192 220 221 222 223
```
defines a pocket in *4pfs* (chain *B*) which can bind *ANP* and *ADP*.

- **Identifiers**: Uses `auth_asym_id` (Chain) and `auth_seq_id` (Residues). See [PDB Identifiers](https://www.rcsb.org/docs/general-help/identifiers-in-pdb) for details.
- **Multiple Pockets**: Separate lines for the same PDB+Chain indicate multiple distinct cryptic pockets, like this (two pockets, one binding *NAP* and *NDP* and other binding *DA3* and *2NP*):
```
5loc;A;NAP NDP;11 12 13 14 15 16 36 37 38 66 67 68 69 72 89 91 118 119 120 121 122 151 152 155 158 159 243 271 275
5loc;A;DA3 2NP;91 120 121 145 149 150 151 152 153 170 196 245 271
```
- *Note: For simplicity, both the test and train subsets exclude pockets spanning multiple chains.*

## Smaller ESM2 model *(for **Cryptic sites pLM** project)*
We provide the 3B parameter fine-tuned ESM2 model for consistency with [our preprint]([/home/skrhakv/cryptoshow-analysis](https://www.biorxiv.org/content/10.64898/2026.01.28.702257v1.full.pdf)). A smaller 650M parameter version is available upon request if hardware constraints arise.

## Trouble-shooting
If you encounter any problems with the data or running the models, please create a [GitHub Issue](https://github.com/cusbg/deeplife-2026/issues)! It is possible that your colleagues will find the same obstacles.

## License
This code is licensed under the [MIT license](https://github.com/skrhakv/deeplife-2026/blob/master/LICENSE).