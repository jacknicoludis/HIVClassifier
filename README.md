# HIVClassifier
Insight Health Data Science Project on predicting HIV infection state from microbiome data.

The goal of this project was to use microbiome data to predict HIV infection state using machine learning classifiers. 

# Project Overview

HIV/AIDS is a global health concern with ~1 million people dying of the infection and its complications every year. A pain point for the eradication of the disease is diagnosing it early enough to prevent transmission to new people. About 25% of people with HIV do not know they have it. The most common (and inexpensive) test for HIV is a antibody-based test, which can detect the disease 4-6 weeks after infection. However, at this point, the person is already infectious. Thus, my project focused on detecting HIV early from microbiome data.

<p align="center">
  <img src="https://github.com/jacknicoludis/HIVClassifier/blob/master/project_overview_files/HIV-acquisition_timeline.png?raw=true" />
</p>

### Workflow
To determine HIV infections from microbiome data, I used the following steps to build a classifier based on compositional difference in the microbiome between people who were HIV negative and those that had recently acquired HIV:

1. Process microbiome data from raw sequencing reads to microbiome composition.
2. Normalize composition across microbiome samples.
3. Select important features for classification
4. Implement classifier

### Results at a glance
I built a logistic regression classifier to determine HIV infection state. For comparing HIV negative and HIV position people, I could distinguish samples with a recall of 68%. For HIV negative and recently HIV infected people, I could distinguish samples with a recall of 73%. Despite the small sample sizes (65 HIV-negative, 98 HIV-positive, 148 recently infected), the training and validation sets converge in learning curves for both classifiers, indicating that there is high variance in the samples but little bias. These results derive from patient samples in sub-Saharan Africa and would need to be retrained for populations in different parts of the world due to differences in baseline microbiomes based on diet and lifestyle, but this example suggests that determining HIV infection state from microbiome data may be possible. With the growth of microbiome sequencing and the decrease of sequencing costs, microbiome profiling may be a new tool for health diagnostics.

# Step 1. Process microbiome sequencing data into compositional data

### An explanation of microbiome profiling
The goal of microbiome profiling is to determine the composition of the microbes (bacteria, fungi, viruses) from a specific site of the body. This composition is dynamic and responds to environmental changes. For example, someone's diet, lifestyle or a disease they have can impact the types of microbes and their composition. To profile this composition, samples of the microbiome are sequenced. For a broad overview of the composition, this is typically (and most inexpensively) done using amplicon sequencing. In this approach, a specific marker gene that is found in all the microbes you'd like to profile (say, bacteria) is amplified using PCR (a method that can replicate a specific DNA strand of interest). The marker gene is chosen such that it is conserved in sequence enough that all the microbes of interest have it, but that there is significant variation between bacteria to differentiate them. Next Next-Gen Sequencing (NGS) is used to sequence the replicated marker genes. Using reference collections of bacterial marker genes, we can determine the species/taxon from which each marker gene originated to get the composition (the type and abundance of each bacteria) of the microbiome. 

### Data set: Cohort from Mozambique
The data set used in this project was from in ["Evolution of the gut microbiome following acute HIV-1 infection,"](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6511141/pdf/40168_2019_Article_687.pdf) by Rocafort et al. (2019) in *Microbiome*. The data includes 65 samples from HIV negative people, 98 samples from people chronically infected with HIV, and 148 samples from people who recently acquired HIV. The recently acquired HIV population were people who tested negative using a rapid antibody-based test and tested positive using a more expensive HIV test that directly detects the virus. 

### Details on microbiome sequencing data processiong
For this project I used [HiMAP](https://www.biorxiv.org/content/10.1101/565572v1) to do these first processing steps. [HiMAP GitHub Repository](https://github.com/taolonglab/himap). These initial steps include merging reads, removing PCR primers, trimming read-lengths, removing chimeras, denoising (using dada2), align sequences against the reference database (the HiMAP database) to obtain abundances. In HiMAP, these are called OSUs (operational strain unit) as opposed to OTUs (operational taxonomic units) to reflect the higher resolution of the HiMAP reference database.

# Step 2. Normalization of compositional data across patients
Normalization of compositional data between samples is an important step in microbiome compositional comparisons because the sequencing depth (the number of sequenced reads) differ between samples as well as the compositional variation. There are many types of normalization procedures for microbiome compositional data. For this project, I implemented `Total Sum Scaling`, `Cumulative Sum Scaling`, and `Centered-Log Ratio` to compare different normalization methods. I settled on using centered-log ratio (CLR) for its strength in removing compositional artifacts and its performance in my classification. For CLR normzalization, I used a pseudocount of 0.5 for all OSUs with no reads.

<p align="center">
  <img src="https://github.com/jacknicoludis/HIVClassifier/blob/master/project_overview_files/normalization.png?raw=true" />
</p>

# Step 3. Feature selection
Because the microbiome is diverse and consists of hundreds to thousands of different organisms, microbiome compositional data has many features (OSUs) and is sparse. To deal with this, I tried several ways to reduce the number of features (OSUs) and select those features that contribute to the variance in the samples. I used dimensionality reduction (in the form of `SVD`) as one method. I also selected the OSUs with the highest Z-score (difference in mean abundance between labels over the standard deviation in abundance), as well as the OSUs that correlate the most with the labels. From these, I chose `SVD` to reduce the number of features to 20, which captured ~46% of the total variance in the samples, and was the least biased way to select the features.

As an additional feature, I allow for the reduction of features by combining similar taxonomic groups. OSUs can be combined based on their taxonomy to a certain level (genus, family, order, etc.). This type of merging OSUs is common in analysis of microbiome data and genus-level compositions are frequently used.

<p align="center">
  <img src="https://github.com/jacknicoludis/HIVClassifier/blob/master/project_overview_files/SVD.png?raw=true" />
</p>

# Step 4. Implementation of classifier
I then used `Logistic Regression` to classify samples. I made two classifiers: one that classified recently HIV-infected people from HIV negative people and one that classified HIV positive and HIV negative people. I chose a binary classification approach to simulate a clinical environment where people would be tested if they suspected they had been infected recently (and present to a clinic with flu-like symptoms) versus were not infected (as this is the situation under which the data was collected). Because a false negative diagnosis is worse than a false positive in a clinical context of HIV treatment (often secondary tests are done if someone is found to be HIV positive) I optimized the recall of my classifier.

# Results
For my classifier distinguishing HIV negative and recently infected people, I obtained a recall of 65%, a precision of 66% and an F1 score of 65%.
<p align="center">
  <img src="https://github.com/jacknicoludis/HIVClassifier/blob/master/project_overview_files/LG_CHI_NEG.png?raw=true" />
</p>
For my classifier distinguishing HIV negative and chronic HIV positive people, I obtained a recall of 69%, a precision of 70% and an F1 score of 69%. This classification is on par with a classification in the [literature](https://www.ebiomedicine.com/article/S2352-3964(16)30028-7/fulltext).
<p align="center">
  <img src="https://github.com/jacknicoludis/HIVClassifier/blob/master/project_overview_files/LG_RHI_NEG.png?raw=true" />
</p>
I used learning curves to determine whether their was bias in the data and whether more samples would improve the model. In the case of chronically HIV postive vs. HIV negative people, the training and cross-validation sets are starting to converge, but more samples may improve model performance.

<p align="center">
  <img src="https://github.com/jacknicoludis/HIVClassifier/blob/master/project_overview_files/LC_CHI_NEG.png?raw=true" />
</p>

In the case of recently HIV infected people, the models have converged, indicate the poor model performance is likely due to a biological property of the samples. This makes sense since people who have recently been infected with HIV likely have not experienced large changes in immune system function and have not started antiretroviral treatment (ARTs), both of which are known to alter the structure of the microbiome.

<p align="center">
  <img src="https://github.com/jacknicoludis/HIVClassifier/blob/master/project_overview_files/LC_RHI_NEG.png?raw=true" />
</p>

# Project in context
Microbiome sequencing is becoming more common in clinical settings and an increasing number of studies have found links to a variety of disease and environmental factors and the structure of the microbiome. As sequencing costs continue to decrease, we can only expect that microbiome sequencing will be a new diagnostic tool and we need to explore how different diseases might impact the structure of the microbiome. In this project I used a dataset of recently-HIV infected individuals to determine whether early HIV infection could be detected by microbiome data. I found that I could predict early HIV infection with better than even odds. My classification performed better at distinguishing HIV negative and chronically HIV postive people. 

While performance of the model was moderate, there are more than 150K people in the US that don't know they have HIV. Most of these people have likely had HIV for a long time but do not seek out HIV testing due to stigma against HIV and lack of medical access. If microbiome diagnostics become more common (say, as common as at-home ancestry tests of which 8% of the US population has taken part), then 7-8K of the 150K who don't know they are HIV positive would find out their HIV positive status.


### Project Files

Project files can be found in the **project_files** directory. To run the Classifier, open the **HIVClassifier.ipynb** notebook and select the appropriate options. 

**Note:** This project was designed to run output files from HiMAP and use labels from a metadata file called HIV_acquisition_metadata.csv. To adjust for new datasets, one should carefully look at the file format of the osu_abundances.txt and osu_taxonomy.txt files as well as the metadata file to adjust accordingly. Further details on file format for input files can be found in **join_and_normalize.ipynb**.

[Project slides can be found here](https://docs.google.com/presentation/d/1UP908EJqk5WD5XU99uxpahnpLAGg9AXD-Qj_7fn7iZY/edit?usp=sharing).
