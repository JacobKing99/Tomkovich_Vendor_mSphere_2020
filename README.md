Download the [latest release](https://github.com/SchlossLab/new_project/releases/latest) to the directory and decompress


## Initial gut microbiota influences Clostridioides difficile clearance after clindamycin perturbation in mice

ABSTRAAAAAAAAAAAAACT




### Overview

	project
	|- README          # the top level description of content (this doc)
	|- CONTRIBUTING    # instructions for how to contribute to your project
	|- LICENSE         # the license for this project
	|
	|- submission/
	| |- study.Rmd    # executable Rmarkdown for this study, if applicable
	| |- study.md     # Markdown (GitHub) version of the *.Rmd file
	| |- study.tex    # TeX version of *.Rmd file
	| |- study.pdf    # PDF version of *.Rmd file
	| |- header.tex   # LaTeX header file to format pdf version of manuscript
	| |- references.bib # BibTeX formatted references
	| |- XXXX.csl     # csl file to format references for journal XXX
	|
	|- data           # raw and primary data, are not changed once created
	| |- references/  # reference files to be used in analysis
	| |- raw/         # raw data, will not be altered
	| |- mothur/      # mothur processed data
	| +- process/     # cleaned data, will not be altered once created;
	|                 # will be committed to repo
	|
	|- code/          # any programmatic code
	|
	|- results        # all output from workflows and analyses
	| |- tables/      # text version of tables to be rendered with kable in R
	| |- figures/     # graphs, likely designated for manuscript figures
	| +- pictures/    # diagrams, images, and other non-graph graphics
	|
	|- exploratory/   # exploratory data analysis for study
	| |- notebook/    # preliminary analyses
	| +- scratch/     # temporary files that can be safely deleted or lost
	|
	+- Makefile       # executable Makefile for this study, if applicable


### How to regenerate this repository

#### Dependencies and locations
* Gnu Make should be located in the user's PATH
* mothur (v1.43.0) should be located in the user's PATH
* FFmpeg should be located in the user's PATH
* R (v. 3.5.2) should be located in the user's PATH
* R packages:
    * broom v0.5.0
    * tidyverse_1.3.0
    * cowplot v0.9.3
    * magick v2.0
    * vegan v2.5-2
    * reshape2 v1.4.3
    * knitr v1.20
    * rmarkdown v1.10
    * gtools v3.8.1
    * ggpubr v.0.2.4
    * ggforce v0.1.3
    * gganimate v1.0.5
* Analysis assumes the use of 8 processors  


#### Running analysis
Download 16S rRNA sequencing dataset from the NCBI Sequence Read Archive (BioProject Accession no. PRJNA608529).
```
git clone https://github.com/SchlossLab/Tomkovich_vendor_difs_XXXX_2020
make write.paper
```
Transfer 16S rRNA sequencing fastq.gz files into Tomkovich_vendor_difs_XXX_2020/data/raw
```
cd Tomkovich_vendor_difs_XXXX_2020
```
Obtain the SILVA reference alignment from version 132 described at http://blog.mothur.org/2018/01/10/SILVA-v132-reference-files/. We will use the SEED v. 132, which contain 12,083 bacterial sequences. This also contains the reference taxonomy. We will limit the databases to only include bacterial sequences.
```
wget -N https://mothur.org/w/images/7/71/Silva.seed_v132.tgz
tar xvzf Silva.seed_v132.tgz silva.seed_v132.align silva.seed_v132.tax
mothur "#get.lineage(fasta=silva.seed_v132.align, taxonomy=silva.seed_v132.tax, taxon=Bacteria);degap.seqs(fasta=silva.seed_v132.pick.align, processors=8)"
mv silva.seed_v132.pick.align data/references/silva.seed.align
rm Silva.seed_v132.tgz silva.seed_v132.*
#Narrow to v4 region
mothur "#pcr.seqs(fasta=data/references/silva.seed.align, start=11894, end=25319, keepdots=F, processors=8)"
mv data/references/silva.seed.pcr.align data/references/silva.v4.align
```
Obtain the RDP reference taxonomy. The current version is v11.5 and we use a "special" pds version of the database files, which are described at http://blog.mothur.org/2017/03/15/RDP-v16-reference_files/.
```
wget -N https://www.mothur.org/w/images/c/c3/Trainset16_022016.pds.tgz
tar xvzf Trainset16_022016.pds.tgz trainset16_022016.pds
mv trainset16_022016.pds/* data/references/
rm -rf trainset16_022016.pds
rm Trainset16_022016.pds.tgz
```
Obtain the Zymo mock community data; note that Zymo named the 5 operon of Salmonella twice instead of the 7 operon.
```
wget -N https://s3.amazonaws.com/zymo-files/BioPool/ZymoBIOMICS.STD.refseq.v2.zip
unzip ZymoBIOMICS.STD.refseq.v2.zip
rm ZymoBIOMICS.STD.refseq.v2/ssrRNAs/*itochondria_ssrRNA.fasta #V4 primers don't come close to annealing to these
cat ZymoBIOMICS.STD.refseq.v2/ssrRNAs/*fasta > zymo_temp.fasta
sed '0,/Salmonella_enterica_16S_5/{s/Salmonella_enterica_16S_5/Salmonella_enterica_16S_7/}' zymo_temp.fasta > zymo.fasta
mothur "#align.seqs(fasta=zymo.fasta, reference=data/references/silva.v4.align, processors=12)"
mv zymo.align data/references/zymo_mock.align
rm -rf zymo* ZymoBIOMICS.STD.refseq.v2* zymo_temp.fasta
```
Run the 16S rRNA sequnce data through mothur.
Create vendors.files:
```
mothur code/make_file_fix_ids.batch
```
Fix the sample ids in vendor.files before proceeding with the rest of the analysis by doing the following:
Make a copy of data/raw/vendors.files and save as a .csv file (data/raw/vendors.files.csv)
Then run the following rscript to fix the ids:
```
Rscript code/fix_ids.R
```
Open up data/raw/vendors.files.fixed.csv and data/raw/vendors.files. Paste rhe fix_id column (select all 439 samples, starting at row 2) from data/raw/vendors.files.fixed.csv over the 1st column with dashes in the group name in data/raw/vendors.files and save data/raw/vendors.files. Now that the sample ids have been corrected, the analysis can proceed.
```
mothur code/get_good_seqs.batch
mothur code/get_error.batch
mothur code/get_shared_otus.batch
mothur code/get.otu.batch
```
Copy files generated by mothur analysis and place into data/process while shortening file names at the same time.
```
bash rename_mothur_outputs
```
Perform *C. difficile* CFU and mouse weight over time analysis and create plots.
```
Rscript code/cfu_plot.R
Rscript code/weight_plot.R
```
Perform alpha diversity analysis and create plots.
```
Rscript code/diversity.R
```
Use code/subset_analysis.R to generate lists of IDs specific to each sample day and pasted them into the following scripts to create a bray-curtis distance matrix and PCoA specific for a single timepoint. Also used to create PCoAs of baseline communities of each mouse colony source.
```
mothur code/d-1_dist_PCoA
mothur code/d0_dist_PCoA
mothur code/d1_dist_PCoA
mothur code/d-1_vendors_dist_PCoA
```
Perform PCoA analysis and create plots.
```
Rscript code/pcoa_stats.R
```
Perform taxonomic analysis and create plots.
```
Rscript code/taxa_stats.R
```
For L2 Logistic regression analysis, use the following script to generate the input data at the OTU and family levels:
```
Rscript code/l2_classification_family_input_data.R
Rscript code/l2_classification_input_data.R
```
Use [ML_pipeline_microbiome repository](https://github.com/SchlossLab/ML_pipeline_microbiome) to perform L2 Logistic regression analysis. Modify model_pipeline.R to specify outcomes and a 60:40 data split for cross-validation and testing steps. Acccess [modified version of repository here](https://github.com/tomkoset/ML_pipeline_microbiome). Move Tomkovich_vendor_difs_XXXX_2020/data/process/classification_input_*data.csv files into ML_pipeline_microbiome/test/data (this will capture input data at both OTU and family level). Run the pipeline as arrayed jobs using batch scripts formatted for Slurm (or whatever scheduler your HPC uses), merge files in between with bash scripts, otherwise they will be overwritten in temp folder:
```
mkdir data/process/dayn1 data/process/day0 data/process/day1
sbatch code/slurm/L2_log_Regression_dn1.sh
bash code/bash/dn1_cat_csv_files.sh
sbatch code/slurm/L2_log_Regression_d0.sh
bash code/bash/d0_cat_csv_files.sh
sbatch code/slurm/L2_log_Regression_d1.sh
bash code/bash/d1_cat_csv_files.sh
mkdir data/process/dayn1_family data/process/day0_family data/process/day1_family
sbatch code/slurm/L2_log_Regression_dn1_family.sh
bash code/bash/family_dn1_cat_csv_files.sh
sbatch code/slurm/L2_log_Regression_d0_family.sh
bash code/bash/family_d0_cat_csv_files.sh
sbatch code/slurm/L2_log_Regression_d1_family.sh
bash code/bash/family_d1_cat_csv_files.sh

```
Copy the 6 folders containing outputs from Logistic Regression classification analysis (there should be 5 files in each folder) and paste into Tomkovich_vendor_difs_XXXX_2020/data/process/classification/
ML_pipeline_microbiome/data/process/dayn1
ML_pipeline_microbiome/data/process/day0 
ML_pipeline_microbiome/data/process/day1
ML_pipeline_microbiome/data/process/dayn1_family
ML_pipeline_microbiome/data/process/day0_family
ML_pipeline_microbiome/data/process/day1_family 

Rename classification output files to indicate which day of the experiment relative abundances from either the OTU or family level were used to train the models to predict *C. difficile* colonization status on day 7.
```
bash code/rename_classification_outputs_60
bash code/rename_classification_outputs_60_family
```
Make plots comparing cross-validation and test AUROCs for classification models generated from 3 different types of input data (OTUs from Day -1, 0, and 1) and make plots to interpret which OTUs are contributing to the 3 models.
```
Rscript code/class._60-40_analysis.R
Rscript code/class_interpretation.R
```
Make plots comparing cross-validation and test AUROCs for classification models generated from 3 different types of input data (bacterial families from Day -1, 0, and 1) and make plots to interpret which families are contributing to the 3 models.
```
Rscript code/class._family_analysis.R
Rscript code/class_interpretation_family.R
```
Perform statistical analysis of 6 classification models.
```
Rscript code/compare_models.R
```

Make figures for the paper.
```
Rscript code/figure_1.R
Rscript code/figure_2.R
Rscript code/figure_3.R
Rscript code/figure_4.R
Rscript code/figure_5.R
Rscript code/figure_6.R
Rscript code/figure_7.R
Rscript code/figure_8.R
Rscript code/figure_S1.R
Rscript code/figure_S2.R
Rscript code/figure_S3.R
Rscript code/figure_S4.R
Rscript code/figure_S5.R
Rscript code/figure_S6.R
Rscript code/figure_S7.R
```

Make supplemental movie for the paper. Note conversion of gif generated from code/pcoa_stats.R requires [FFmpeg](http://ffmpeg.org/). Install FFmpeg using homebrew.
```
brew install ffmpeg
```
Convert gif into .mov file and simultaneously move and rename file into submission directory. 
```
ffmpeg -i results/pcoa_over_time.gif -movflags faststart -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" results/pcoa_over_time.mov
cp results/pcoa_over_time.mov submission/movie_S1.mov
```

Generate paper.
```
make write.paper
```
