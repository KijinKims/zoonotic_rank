# Zoonotic risk assement from viral genomes

Code and data used in Mollentze _et al._ (2020) "Identifying and prioritizing potential human-infecting viruses from their genome sequences".

For a list of priority categories and ranks for all viruses in the paper, see ___.


## Table of contents
- [Requirements](#requirements)
- [Ranking novel viruses](#ranking-novel-viruses)
- [Repeating all analyses](#repeating-all-analyses)
- [File structure](#file-structure)



## Requirements
- [R](https://www.r-project.org/) (tested using version 3.5.1)
   - Install required R libraries using `Rscript -e "renv::hydrate()"`
- [Python](https://www.python.org/) (version >=3.6)
   - [Biopython](https://biopython.org/)
   - [Pandas](https://pandas.pydata.org/)

If repeating all analyses (see below), the [BLAST+ suite of applications](https://www.ncbi.nlm.nih.gov/books/NBK279670/?report=classic) are also required (used for "phylogenetic neighbourhood" analyses and predictions, but not otherwise required). If your R session has trouble finding the BLAST+ executables, run `make update_path` and enter the path to  BLAST+ (e.g. `/usr/local/ncbi/blast/bin`). Making figure S9 requires [iqtree](http://www.iqtree.org/).


## Ranking novel viruses
Ranks for novel viruses can be generated by specifying the input format, paths to genome and metadata files, and a name for output files, e.g:
```
Scripts/PredictNovel.R fasta InternalData/example_files/genomes.fasta InternalData/example_files/metadata.csv example_1
```

For detailed instructions and further options, see
```
Scripts/PredictNovel.R --help
```

#### Input
The example files in `InternalData/example_files/` illustrate the required file format for the most complicated scenario, in which sequences are in fasta format and open reading frames are specified manually. In such cases, the metadata file should have the following form (excluding the "Comments" column):


Name                  | SequenceID  | CodingStart | CodingStop | Comments
----------------------|-------------|-------------|------------|----------------
Non-segmented virus 1 | NC_031751.1 | 107         | 1894       | Each line specfies an open reading frame (ORF)
Non-segmented virus 1 | NC_031751.1 | 2309        | 4252       | Name and SequenceID repeated to specify another ORF in the same sequence
Segmented virus 1     | NC_015451.1 | 20          | 6277       | Multiple viruses are distinguished using the Name column
Segmented virus 1     | NC_015450.1 | 24          | 3986       | Name repeated to specify a further SequenceID belonging to this virus
Segmented virus 1     | NC_015452.1 | 36          | 866        | The first of two genes on this segment
Segmented virus 1     | NC_015452.1 | 1778        | 1041       | Coordinates reversed to indicate that the ORF is on the complementary strand

If sequences are available in genbank format, only the first two columns are required, mapping each virus name to one or more sequence identifiers. This format allows more flexiblility in the specification of open reading frames (see the "FEATURES" section of https://www.ncbi.nlm.nih.gov/genbank/samplerecord).

#### Output
All files generated will be preceded by the output name specified ("example_1" in the example above).

File                         | Description
-----------------------------|----------------------------------------------------------------------------
`*.predictions.csv`          | Predicted scores, binary predictions of whether the virus is expected to infect humans, and priority categories.
`*.genome_features.csv`      | Calculated genome composition features used to make predictions.
`*.explanations.csv`         | Influence of individual features on the predicted scores of each virus (i.e., SHAP values).
`*.similar_explanations.csv` | Viruses in the training data which have similar explanations for their predicted scores. All viruses are clustered based on similarity in their SHAP values, using affinity propagation clustering to identify discrete clusters and the most central observation in each cluster (the exemplar). This output file details clusters containing at least one of the input viruses.
`*.all_cluster_members.csv`  | All clusters obtained during the above SHAP similarity-based clustering.
`*.similar_explanations.pdf` | Dendrogram illustrating the distances between clusters in `*.all_cluster_members.csv`. Red branches highlight clusters containing input viruses.
`*.feature_dendrogram.pdf`   | Hierarchical clustering of viruses by similarity in their genome features, giving an indication of how distant the input viruses are from the training data. Note that this clustering will not neccesarily reflect taxonomy.



## Repeating all analyses

Follow instructions below to repeat the analyses described in the manuscript. Note that these steps are _not_ needed to make predictions as described above (pre-trained models are included). Running all analyses takes ~3 weeks on a 4-core, 2.8 GHz i7 processor and requires ~5Gb of disk space.

#### Basic
These steps will download any missing source data and automatically create/update files as needed.

_Using Rstudio:_
1. Open `ZoonosisPredictor.Rproj` in RStudio
2. On the `Build` tab, select `More` > `Clean and Rebuild`

_Using the command-line:_
```
make clean all
```

#### Advanced options (command-line only)

- Use `make help` to see individual steps in the pipeline. Upstream steps are run automatically if needed. For example, using `make prepare` will run the data cleanup step, but also downloads the raw data if needed.
- `make <path to file>` runs all steps neccesary to produce/update the specified file (e.g. `make Plots/Figure1.pdf`).
- `make as_distributed` resets the project to the state in which it was distributed.
- `make clean` removes all run-related files, allowing a complete re-run (in contrast to `as_distributed`, this includes removing the pre-trained models required for prediction).


## File structure
(files/folders which will be created during a full run are indicated by `[*]`)

```
└─ZoonosisPredictor/
   ├─Makefile                                   Record of workflow and dependencies
   │                                            between files
   ├─options.config                             Runtime options (speciefies number
   │                                            of parrallel threads allowed and 
   │                                            the random seed)
   ├─InternalData/                              All data unique to this project
   │   ├─example_files/                           Example input files for 
   │   │                                          predicting novel viruses 
   │   ├─Shaw2017_raw/                            Raw ISG data from Shaw et al. 
   │   │                                          2017 (see https://isg.data.cvr.ac.uk/)
   │   ├─FinalData_Cleaned.csv                    Final dataset, as used for training. 
   │   │                                          Created by merging files below 
   │   │                                          (see Scripts/MergeAndCleanData.R)
   │   ├─AllInternalData_Checked.csv              Metadata for the viruses used 
   │   │                                          as training data
   │   ├─Final_Accessions_Unique_Spp.csv          Accession numbers of sequences 
   │   │                                          used for training (replaces 
   │   │                                          those in the metadata file)
   │   ├─NameMatches_All.csv                      Manually curated list used to 
   │   │                                          match virus names to unique 
   │   │                                          species across external datasets
   │   ├─SourcesOfZoonoses_BabayanZoonotic.csv    Additional zoonotic status data 
   │   │                                          for species not available in 
   │   │                                          external data sources
   │   └─Taxonomy_UnclassifiedViruses.csv         Taxonomic information for 
   │                                              unclassified viruses in the 
   │                                              metadata (unused)
   │
   ├─CalculatedData/                            [*] Intermediate calculations
   ├─ExternalData/                              [*] Data from external sources, 
   │                                            dowloaded as needed (see Makefile)
   ├─Misc/                                      Miscelaneous scripts to download 
   │                                            external data
   ├─Plots/                                     [*] Final plots generated
   ├─Predictions/                               [*] Predictions for case studies
   ├─renv/                                      Cache of R libraries required
   ├─RunData/                                   Trained models ([*], except for 
   │                                            files required by PredictNovel.R)
   ├─Scripts/                                   Main analysis, prediction, and 
   │                                            plotting scripts
   │   └─Plotting/                              Scripts to generate published plots
   ├─Tests/                                     Unit tests for basic functionality 
   │                                            of utility scripts
   └─Utils/                                     Utility functions and tools called 
                                                by other scripts
```
