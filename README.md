## GSSG : Gene Set and SNP-Gene strategies

Codes for combining gene sets (programs) with S2G strategies to generate annotations 
Also check out some relevant gene sets explored in our paper.

## Citation

If you use the data or code from this repository, please cite these following papers

<ins> **blood specific gene programs** </ins>
Dey, K.K. et al bioRxiv. 2020. Unique contribution of enhancer-driven and master-regulator genes to autoimmune disease revealed using functionally informed SNP-to-gene linking strategies. [Link](https://www.biorxiv.org/content/10.1101/2020.09.02.279059v1)

<ins> **single-cell gene programs (sc-linker)** </ins>
Jagadeesh, K.J.*, Dey, K.K.* et al bioRxiv. 2021. Identifying disease-critical cell types and cellular processes across the human body by integration of single-cell profiles and human genetics.(sc-linker) [Link]()

If you use the ABC S2G strategies please cite Nasser, J., Engreitz, J. et al. unpublished data. 2020. and [Fulco et al, 2019](https://www.nature.com/articles/s41588-019-0538-0). If you use PC-HiC S2G strategies, please cite [Javierre et al 2016 Cell](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5123897/). If you use ATAC (Yoshida), cite [Yoshida et al 2019 Cell](https://www.cell.com/cell/pdf/S0092-8674(18)31650-7.pdf). If you use eQTL S2G strategies, cite [Hormozdiari et al 2018 NG](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6030458/). If you use Roadmap S2G links, cite the references [here](https://ernstlab.biolchem.ucla.edu/roadmaplinking/). 


## Code Tutorial


code/GeneSet_toS2G :  Here is a demo of how to run the codes to combine gene set with S2G strategy to create annotation

Take a geneset file ( see demo ~/code/GeneSet_toS2G/Test_GeneSets/geneset_example.txt )

```head -n 5 geneset_example.txt ```

```
RITA1	0
FAM225B	0.00320672144802546
TBC1D23	0.0716823315695179
SIRPG	0
L3MBTL4-AS1	0
```

It is a 2 column file with the first column being the names of genes (HGNC names). If the gene names are in Emntrez or
Ensembl, please convert them first to HGNC. 

### Gene Set -> Annotation

We first run the script that converts the gene set file to probabilistic .bed format files (bedgraph) for different S2G strategies.

First go to the folder

```
cd ~/GSSG/code/GeneSet_toS2G

```
For **blood specific data (Dey et al 2020)** with 10 different S2G strategies, run the following script

```
geneset_dir=Test_GeneSets
geneset=geneset_example
bed_dir=Test_Beds
bash geneset_to_bed.sh $geneset_dir $bed_dir $geneset
```

For **sc-linker (Jagadeesh, Dey et al 2021)** with Roadmap-U-ABC enhancer-gene strategy for a tissue-specific enhancer,
please run the following script

```
enhancer_tissue="BLD" (for blood, can also be "BRN", "GI", "LNG", "LIV", "KID", "SKIN",
                      "FAT", "HRT" for brain, intestine, lung, liver, kidney, skin, adipose and heart)
bash geneset_to_bed_sclinker.sh $geneset_dir $bed_dir $geneset $enhancer_tissue
```

We postprocess the created bedgraph files by removing overlapping intervals. The user needs BEDTOOLS and BEDOPS for the
following script. Check `clean_bed.sh` for details.

```
bash clean_bed.sh $bed_dir/$geneset
```

Next we use a .bim file for a list of variants to annotate. We here use the .bim files widely used for S-LDSC analysis.

```
bimfile_path="~/GSSG/processed_data/BIMS"
annot_path=Test_Annots
bash bed_to_annot.sh $bed_dir $bimfile_path $annot_path $geneset
```

If you processed the bed files with **all blood S2G strategies Dey et al 2020)** using `geneset_to_bed.sh`, then for the `geneset_example.txt` file, the above codes will create a folder structure of the form 

```
- Test_Annots
  - geneset_example 
    - 100kb
    - 5kb
    - ABC
    - FinemapBloodeQTL (eQTL S2G in paper)
    - PCHiC
    - Roadmap_Enhancer
    - Yoshida (ATAC S2G in paper)
```

This includes 7 of 10 S2G strategies discussed in the manuscript. For the other strtegies, we look at TSS, Promoter and Coding regions for each gene set using the UCSC nomenclature and the already set-up baseline-LD model. For this, download the baseline-LD annotations from here `https://data.broadinstitute.org/alkesgroup/LDSCORE/`. Say these baseline-LD annotations are in a directory called `baseline_cell`.

Then run the following script to generate the S2G annotations for the remaining 3 strategies.

```
bash geneset_coding_tss_promoter.sh $annot_path $geneset $baseline_cell
```

If you processed the bed files with **tissue-specific enhancer-gene S2G strategies (Jagadeesh, Dey et al 2021)** using `geneset_to_bed_sclinker.sh`, then for the `geneset_example.txt` file, the above codes will create a folder structure 
of the form 

```
- Test_Annots
  - geneset_example 
    - ABC_Road_{enhancer_tissue}
    - 100kb
```


### Gene score analysis and S-LDSC

1. Check `GSSG/code/calc_gene_scores/calc_gene_scores_enhancers.R` and `GSSG/code/calc_gene_scores/calc_gene_scores_masterreg.R`
for how the gene sets in `GSSG/genesets` were computed.

2. Check `GSSG/code/ppi_RWR.R` and `GSSG/code/ppi_string_RWR.R` for functions to compute PPI-informed combination of multiple 
   gene scores either using gene network or using the STRING PPI network. Also check `GSSG/code/example.R` for a demo.
   
3. For running the S-LDSC analysis, go to `GSSG/code/ldsc`. 
    - create_ldscores.sh : Create LD-scores from a reference panel
	  - run_ldsc_reg.sh: Run regression model in S-LDSC
	  - ldsc_postprocess_taustar.R: calculate tau-star metric for S2G annotations of a gene set
	  - ldsc_postprocess_enrichment.R : calculate the enrichment metric for S2G annotations of a gene set
	  - ldsc_postprocess_joint_taustar.R : Joint S-LDSC analysis tau-star
	  - ldsc_postprocess_joint_enrichment.R : Joint S-LDSC analysis enrichment 
	  - ldsc_postprocess_combined_taustar.R : Compute combined tau-star metric for the annotations 
    

## Gene Sets

Probabilistic gene sets (.txt files with 2 columns, first column gene symbol HGNC, the second symbol
the probabilitic membership of the gene in the gene set. Equals to 1 for binary gene sets. Genes with 0 weight
can be ignored)

- Enhancer-driven gene scores: ABC9_G_top10_0_015.txt (ABC-distal), EDS_Binary_top10.txt (EDS-binary),
                              eQTL_CTS_prob.txt (eQTL-CTS), Expecto_MVP.txt (ExPecto-MVP), HOMOD_prob.txt (ATAC-distal)
                              PCHiC_binary.txt (PC-HiC-binary), SEG_GTEx_top10.txt (SEG-blood)
                              
- Master-regulator gene scores:  master_regulator.txt (Trans-master), TF_genes_curated.txt (TF-genes)

- PPI-informed gene scores:  PPI_Enhancer.txt (PPI-enhancer), PPI_Master.txt (PPI-master),
                                    PPI_All.txt (PPI-all), PPI_control.txt (PPI-control)
                                    
- Additional gene scores: pLI_genes2.txt (pLI), RegNet_Enhancer.txt (RegNet-Enhancer), Trans_Reg_genes.txt (Trans-regulated)

## Annotations

All gene set S2G annotations studied in the companion paper can be found at `https://alkesgroup.broadinstitute.org/LDSCORE/Dey_Enhancer_MasterReg/`. The Enhancer-Promoter processed prediction files can be found at 
`https://alkesgroup.broadinstitute.org/LDSCORE/Dey_Enhancer_MasterReg/processed_data/`. Some of the large data files needed to run some of the codes above are not provided with this repo. The user can fetch them (by `wget`) from `https://alkesgroup.broadinstitute.org/LDSCORE/DeepLearning/Dey_DeepBoost_Imperio/data_extra/` and add them to the `processed_data/` directory.


## How to use these annotations?

1) Download the LDSC from git (https://github.com/bulik/ldsc/wiki/Partitioned-Heritability)
2) Download the baselineLD_v2.1 annotations from Broad webpage (https://data.broadinstitute.org/alkesgroup/LDSCORE/)
3) Download deep learning predictive annotations (see above) from https://data.broadinstitute.org/alkesgroup/LDSCORE/Dey_DeepLearning
4) Use your GWAS summary statistics formatted in LDSC details is available (https://github.com/bulik/ldsc/wiki/Summary-Statistics-File-Format)
5) Download the baseline frq file and weights for 1000G available (https://data.broadinstitute.org/alkesgroup/LDSCORE/)
6) Run S-LDSC with these annotations conditional on baselineLD_v2.1 (see https://github.com/bulik/ldsc/)

```
ANNOT FILE header (*.annot):

CHR -- chromosome
BP -- physical position (base pairs)
SNP -- SNP identifier (rs number)
CM -- genetic position (centimorgans)
all additional columns -- Annotations
```

NOTE: Although one would expect the genetic position to be non-negative for all 1000G SNPs, we have checked that
in fact the genetic position is negative for a handful of 1000G SNPs that have a physical position that is smaller
than the smallest physical position in the genetic map. The genetic positions were obtained by running PLINK on
the Oxford genetic map (http://www.shapeit.fr/files/genetic_map_b37.tar.gz).

MORE DETAIL CAN BE OBTAINED FROM https://github.com/bulik/ldsc/wiki/LD-File-Formats


```
LD SCORE FILE header (*.l2.ldscore):

CHR -- chromosome
BP -- physical position (base pairs)
SNP -- SNP identifier (rs number)
all additional columns -- LD Scores

```

## Contact 

In case of any questions, please open an issue or send an email to me at `kdey@hsph.harvard.edu`.









