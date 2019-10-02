# HIVClassifier
Insight Health Data Science Project on predicting HIV infection state from microbiome data.

The goal of this project was to use microbiome data to predict HIV infection state using machine learning classifiers. The data from this project was previously published in ["Evolution of the gut microbiome following acute HIV-1 infection,"](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6511141/pdf/40168_2019_Article_687.pdf) by Rocafort et al. (2019) in *Microbiome*.

In addition, the raw sequencing reads from this study were processed by [HiMAP](https://www.biorxiv.org/content/10.1101/565572v1) into abundance files, following the instructions from the [HiMAP GitHub Repository](https://github.com/taolonglab/himap). 

### Project Files

Project files can be found in the **project_files** directory. To run the Classifier, open the **HIVClassifier.ipynb** notebook and select the appropriate options. 

**Note:** This project was designed to run output files from HiMAP and use labels from a metadata file called HIV_acquisition_metadata.csv. To adjust for new datasets, one should carefully look at the file format of the osu_abundances.txt and osu_taxonomy.txt files as well as the metadata file to adjust accordingly. Further details on file format for input files can be found in **join_and_normalize.ipynb**.

[Project slides can be found here](https://docs.google.com/presentation/d/1UP908EJqk5WD5XU99uxpahnpLAGg9AXD-Qj_7fn7iZY/edit?usp=sharing).

# Project Overview


