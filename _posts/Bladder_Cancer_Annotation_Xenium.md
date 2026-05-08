# Comparison of Cell Type Annotation methods in Muscle invasive Bladder cancer
> *This post consists on the most important part of my master's thesis*

---
## 1. What is Xenium?
> *The annotation strategy will depend on the method used to extract the RNA data*
Xenium is a spatial transcriptomics technology, very recent (2022) and has huge potential as it has subcellular resolution.
>
> **Let me explain myself**, transcrips are mRNA molecules (very tiny pieces of DNA) that float arround our cells to finally become proteins and have a purpose. Reading the mRNA transcripts is a way to know what the cell wanted to do or what is currently doing ! And the spatial part does not refer to outer space, is about where in the tissue was that mRNA found. We can know what each cell is doing without breaking down the tissue gaining the capability of reading the interactions between cells !
---
## 2. Why is cell type annotation so important?
> *The reference dataset will depend on the type of tissue we are dealing with*
---
Cell type annotation is crucial because it translates raw single-cell RNA sequencing (scRNA-seq) data into meaningful biological insights, allowing researchers to identify, classify, and understand the diverse cell populations driving biological processes in health and disease. (I'm choosing to talk about scRNA-seq here because in the cell annotation process we don't care about the spatial axis, we are looking at each cell individually)

## 3. Problem description
> *The aim here is to annotate cell types in the Xenium data. Xenium can allow us to take the position of the gene by limiting the variaety of genes it can read, making cell annotation extra hard compared to single cell analysis.*
---

After quality control and preprocessing (sketch, PCA, etc.), the first key step is **clustering**. The rest of the annoation pipeline will depend on these clusters, the best algorithm for this task with biological datasets is the **Leiden** algorithm, , offering faster computation and guaranteeing better-connected communities.  

In Seurat, Leiden is algorithm = 4 in `FindClusters()`

From here, we can have supervised or unsupervised methods. Supervised methods outperform the unsupervised methods, except for the identification of unknown cell types. This is particularly true when the supervised methods use a reference dataset with high informational sufficiency, low complexity and high similarity to the query dataset. However, such outperformance could be undermined by some undesired dataset properties investigated in this study, which lead to uninformative and biased reference datasets. In these scenarios, unsupervised methods could be comparable to supervised methods. Although supervised methods are useful in some cases, they cannot be applied to all cases because reference data sets are not available for most organs, tissues, and conditions. [2]

In our case, we do not have a reference dataset specific to our tissue and patients so we are constraint to use a general pure cell type reference dataset called “BlueprintEncodeData”. Using the supervised method *SingleR*, which was stated to be the best method for cell annotation in [1], we could get some annotations with a relatively high relative abundance (see the following image) but there are still some clusters that need further refinement and confirmation.
![SingleRAnnotations](images/Leiden_SingleR_together.png)

Now we are left with unnspecified clusters with the top genes per each and lots of information on the internet so let's clear all that up in this post.

## 4. Practical strategies for cluster annotation in SingleCell [3]
(the referece also mentions SingleR but as we have already used it we skip it)

- Identify key marker genes associated with specific cell types and compare them to the genes defining each cluster in your scRNA-seq dataset. (biology-first approach)
- Large-scale reference atlases, such as the Human Cell Atlas and Azimuth, enable label transfer by mapping scRNA-seq clusters to well-characterized datasets. When batch effects are minimized, this approach provides reliable cell-type annotations based on transcriptional similarity.
- For standardized classification, cell ontologies like the Cell Ontology (CL) define cell types hierarchically based on function and molecular identity. Integrating ontologies with label transfer enhances annotation consistency and facilitates cross-study comparisons.
- Finally, confirm or refine labels by returning to a biology-first approach. Examine top genes in each cluster and assess consistency with published markers.

Althought the reference was usefull to get a the whole picture of how the analysis should be made, it's still single cell, not spatial. In our spatial transcriptomic experiment the amount of genes that we can read is ongly 377 while in typical single-cell RNA sequencing (scRNA-seq) experiments, the number of genes detected per cell usually ranges from 1,000 to 5,000. So probably the single cell methods will not be as usefull in our data as the specificity he have per cell is much lower in gene terms.

## 5. Which markers should I use for Muscle Invasive Bladder Cancer (MIBC) and how ?
In [4] we have general curated markers for each cell type, but it's still better to be further concrete and use gene we now are present in MIBC.

The only paper on the internet that uses Xenium in MIBC is the DUTRENEO paper, which the one we are working on.
If we amplify the search criteria to spatial transcriptomics technologies in MIBC we have [Wahafu et. al 2025](https://www.sciencedirect.com/science/article/pii/S2666168323001313?ref=pdf_download&fr=RR-2&rr=9debb7123f256e11) which uses Visium.

In the end, each cell type has a set of concrete genes that are unique to themselves. Those are called marker genes, and thank to them we can make differential expression analysis with `FindAllMarkers` function that also give us metrics to make a threshold of which cell type is each cell and which cells have unconcrete signatures (lacks high expression of a concrete cell type). Then to assign a lable to those unconcrete cells, there's `AddModuleScore` function to give us an score of its average expression levels. 

## 6. Do papers actually reclusterize ?
Reclustering is a used resource inside our lab. However, it is used only when other methods have failed before. In our sample right now we have 20 clusters that have an unconcrete mix of cell types, that elevated number of cell types is too high to consider SingleR + the original cluster a good first approach. Right now it might be the sample or that other algorithm like BANKSY should be tested in order to get better results.

**CORRECTION:** In the end, I used SingleR and double checked its results with FindAllMarkers, AddMdouleScore and FindMarkers functions.

# 8. Current AI methods ? Are they used ?
They exist but current bioinformaticians seem to be reluctant to use them. With Xenium transcriptomics data there are few papers that use a neural network that have not been made by the authors. However, Machine Learning methods are widely used but benchmarking should that SingleR is the best for Xenium sptaila transcriptomics data, and it can not be considered a machine learning algorithm. 

# Citations
*For clustering methods:*
[1] Sun X, Lin X, Li Z, Wu H. A comprehensive comparison of supervised and unsupervised methods for cell type identification in single-cell RNA-seq. Brief Bioinform. 2022 Mar 10;23(2):bbab567. doi: 10.1093/bib/bbab567. PMID: 35021202; PMCID: PMC8921620. https://pubmed.ncbi.nlm.nih.gov/35021202/ 
[2] Li D, Ding J, Bar-Joseph Z. Unsupervised cell functional annotation for single-cell RNA-seq. Genome Res. 2022 Sep 27;32(9):1765-1775. doi: 10.1101/gr.276609.122. PMID: 35764397; PMCID: PMC9528981. https://pmc.ncbi.nlm.nih.gov/articles/PMC9528981
*Singel cell strategies:*
[3] https://www.nygen.io/resources/blog/scrna-seq-cluster-annotation#:~:text=Once%20preprocessed%2C%20the%20data%20often,inflate%20differences%20across%20cell%20populations.
*General cell type markers:*
[4] https://www.celltypist.org/encyclopedia/Immune/v2
