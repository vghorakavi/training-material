---
layout: tutorial_hands_on
topic_name: metagenomics
tutorial_name: shotgun
---

# Introduction
{:.no_toc}

In metagenomics, information about micro-organisms in an environment can be extracted with two main techniques:

- Amplicon sequencing, which sequence only on the rRNA/rDNA of organisms
- Shotgun sequencing, which sequence full genomes of the micro-organisms in the environment

Data generated from these two techniques must be treated differently. In this tutorial, we will focus on the analysis of whole-genome sequencing. 

> ### {% icon comment %} Comments
> If you want to learn how to analyze amplicon data, please check our dedicated tutorials
{: .comment}

From both amplicon and shotgun metagenomics raw data, we can extract information about which micro-organisms are present in the studied environment. But, contrary to amplicon, shotgun metagenomics data contain also full genome information about the micro-organisms. It is then possible to identify genes associated to functions, to reconstruct metabolic pathways and then determine which functions are done by the micro-organisms in the studied environment. It is even possible to go further and determine which micro-organisms are involved in a given function or pathways.

> ### Agenda
>
> However, extraction of useful information from raw shotgun metagenomics sequences is a complex process
with numerous bioinformatics steps and tools to use. These steps can be get together in 4 main steps we will deal with in the following tutorial:
>
> 1. TOC
> {:toc}
>
{: .agenda}


In this tutorial, we will work on a sample from Arctic Ocean (at 451 m), sequenced with Illumina MiSeq. This sample has already been analyzed with the [EBI Metagenomics' pipeline](https://www.ebi.ac.uk/metagenomics/pipelines/3.0), which uses slightly different tools than the ones we will use here. But, we could compare our results with the ones obtained with EBI Metagenomics.

# Pretreatments

Before any extraction of information about the community, raw sequences have to be pre-processed with quality control of the raw sequences and sequence sorting. But, first, we need in get our data in Galaxy.

## Data upload

The original data are available at EBI Metagenomics under run number [ERR1855251](https://www.ebi.ac.uk/metagenomics/projects/ERP015773/samples/ERS1569001/runs/ERR1855251/results/versions/3.0).

> ### {% icon hands_on %} Hands-on: Data upload
>
> 1. Import the FASTQ file pair from [Zenodo]() or from the data library
>
>    > ### {% icon tip %} Tip: Importing data via links
>    >
>    > * Copy the link location
>    > * Open the Galaxy Upload Manager
>    > * Select **Paste/Fetch Data**
>    > * Paste the link into the text field
>    > * Press **Start**
>    {: .tip}
>
>    > ### {% icon tip %} Tip: Importing data from a data library
>    >
>    > * Go into "Shared data" (top panel) then "Data libraries"
>    > * Click on "Training data" and then "WGS input data"
>    > * Select both files
>    > * Click on "Import selected datasets into history"
>    > * Import in a new history
>    {: .tip}
>
>    As default, Galaxy takes the link as name, so rename them.
>
{: .hands_on}

## Quality control and treatment

For quality control, we use similar tools as described in [the Quality Control tutorial](../../NGS-QC/tutorials/dive_into_qc): [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) and [Trim Galore!](http://www.bioinformatics.babraham.ac.uk/projects/trim_galore/).

> ### {% icon hands_on %} Hands-on: Quality control
>
> 1. **FastQC** {% icon tool %} on both FastQ files to control the quality of the reads
> 2. **MulitQC** {% icon tool %} with
>    - "Software name" to `FastQC`
>    - "Result file" to the raw data generated with FastQC
>
>    > ### {% icon question %} Questions
>    >
>    > 1. What can we say about the quality of the sequences in both sequence files?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li>The quality of the sequence decrease a lot at the end of sequences for both datasets. We will need to trim them</li>
>    >    </ol>
>    >    </details>
>    {: .question}
>
> 2. **Trim Galore** {% icon tool %} with
>    - "Is this library paired- or single-end?" to `Paired-end`
>    - "Reads in FASTQ format" to the input datasets with first the forward (ending with `_1`) and then the reverse (ending with `_2`)
>    - "Trim Galore! advanced settings" to `Full parameter list`
>    - "Trim low-quality ends from reads in addition to adapter removal" to `20`
>    - "Discard reads that became shorter than length N" to `60`
>    - "Generate a report file" to `Yes`
>
> 3. **MulitQC** {% icon tool %} with
>    - "Software name" to `Cutadapt`
>    - "Result file" to the report file generated with Trim Galore!
>
>    > ### {% icon question %} Questions
>    >
>    > 1. How much of the sequences have been trimmed?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li>6.7% and 18.3% of the bases for the forward reads and reverse reads (respectively)</li>
>    >    </ol>
>    >    </details>
>    {: .question}
{: .hands_on}

One sequence file in Fasta is expected for the next steps. We need then to assemble the paired sequences and convert them to Fasta.

> ### {% icon hands_on %} Hands-on: Paired-end join
>
> 1. **fastq-join** {% icon tool %} with
>    - "Dataset type" to `Paired-end`
>    - "Read 1 Fastq" to the forward trimmed reads 
>    - "Read 2 Fastq" to the reverse trimmed reads
> 2. **FASTQ to FASTA** {% icon tool %} with
>    - "FASTQ Library to convert" to the joined FastQ file
>    - "Discard sequences with unknown (N) bases" to `no`
>    - "Rename sequence names in output file" to `no`
{: .hands_on}

## Dereplication

During sequencing, one sequence must have been added to the dataset in multiple exact copy. Removing such duplicates reduce the size of the dataset without loosing information, with the dereplication (identification of unique sequences in a dataset).

> ### {% icon hands_on %} Hands-on: Dereplication
>
> 1. **VSearch dereplication** {% icon tool %} with
>    - "Select your FASTA file" to the Fasta file generated previously
>    - "Strand specific clustering" to `Both strand`
>
>    > ### {% icon question %} Questions
>    >
>    > 1. How many sequences are removed with the dereplication?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li>42,363 on 43,145</li>
>    >    <li></li>
>    >    </ol>
>    >    </details>
>    {: .question}
>
{: .hands_on}

## Sequence sorting

With shotgun metagenomics data, full genome information can be accessed: information corresponding to CDS of the micro-organisms, sequences corresponding to ribosomal sequences (rDNA or rRNA) of the micro-organisms, ... Useful functional information are present in sequences corresponding to CDS, and some taxonomic information in sequences corresponding to ribosomomal sequences (like the amplicon). To reduce the dataset size for the extraction of functional information, we can remove rRNA/rDNA sequences from the original dataset. 

This task is also useful to inspect the rRNA/rDNA sequences. And as in EBI Metagenomics' pipeline, these sequences can be used for taxonomic analyses as any amplicon data

> ### {% icon comment %} Comments
> If you want to learn how to analyze amplicon data, please check our dedicated tutorials
{: .comment}

For this task, we use SortMeRNA ([Kopylova et al, 2012](https://academic.oup.com/bioinformatics/article-abstract/28/24/3211/246053)). This tool filter RNA sequences based on local sequence alignment (BLAST) against 8 rRNA databases (2 Rfam databases for 5.8S and 5S eukarya sequences and 6 SILVA datasets for 16S (archea and bacteria), 18S (eukarya), 23S (archea and bacteria) and 28S (eukarya) sequences.

> ### {% icon hands_on %} Hands-on: Sequence sorting
>
> 1. **SortMeRNA** {% icon tool %} with
>    - "Querying sequences" to the dereplicated dataset
>    - "Sequencing type" to `Reads are not paired`
>    - "Which strands to search" to `Search both strands`
>    - "Databases to query" to `Public pre-indexed ribosomal databases`
>    - "rRNA databases" to select all
>    - "Include aligned reads in FASTA/FASTQ format?" to `Yes`
>    - "Include rejected reads file?" to `Yes`
>    - "Include alignments in SAM format?" to `No`
>    - "Include alignments in BLAST-like format?" to `No`
>
> 2. (Optional) **SortMeRNA** {% icon tool %} Run SortMeRNA on the assigned rRNA sequences with the selection of 16S rRNA databases
> 3. (Optional) **SortMeRNA** {% icon tool %} Run SortMeRNA on the assigned rRNA sequences with the selection of 18S rRNA databases
{: .hands_on}

> ### {% icon question %} Questions
>
> 1. Which percentage of the original data are assigned to rRNA/rDNA sequences?
> 2. How can you explain with low percentage?
>
>    <details>
>    <summary>Click to view answers</summary>
>    <ol type="1">
>    <li>1,988 over 42,363 are aligned on rRNA databases so 4.7%</li>
>    <li>Shotgun metagenomics data with few rRNA genes</li>
>    </ol>
>    </details>
{: .question}

The low percentage of sequences assigned to rRNA genes show that using only these sequences can biais the found taxons. But if you want, you can still using the extracted 16S rRNA and/or 18S rRNA sequences with tools like QIIME for the taxonomic information. Here, we will use a different approach.

# Extraction of taxonomic information

The first important information to extract from any metagenomics sample is which micro-organisms are present in the sequenced sample and in which proportion are they present. So we want to extract information about the structure of the community of micro-organisms in the environment.

To identify the community structure, several approaches can be used. With amplicon or rRNA data, the sequences are clustered into Operational Taxonomic Units (OTU) and one representative sequence of each OTU is assigned to the most plausible microbial lineage. This approach is possible because of rRNA data: data that evolved quite slowly compared to other part of genomes, rRNA sequences data (particularly 16S and 18S) are then well conserved and good taxonomic markers.

However, for shotgun data, applying such approaches implies using a really small proportion of sequences for the taxonomic assignation, which can induce statistical bias. Other approaches have been developed to cope with shotgun data. For example, MetaPhlAn2 ([Truong et al, 2015](https://www.nature.com/nmeth/journal/v12/n10/full/nmeth.3589.html)) uses a database of ~1M unique clade-specific marker genes (not only the rRNA genes) identified from ~17,000 reference (bacterial, archeal, viral and eukaryotic) genomes.

## Taxonomic assignation

> ### {% icon hands_on %} Hands-on: Taxonomic assignation
>
> 1. **MetaPhlAN2** {% icon tool %} with
>    - "Input file" to the dereplicated sequences
>    - "Database with clade-specific marker genes" to `Locally cached`
>    - "Cached database with clade-specific marker genes" to `MetaPhlAn2 clade-specific marker genes`
>    - "Type of analysis to perform" to `Profiling a metagenomes in terms of relative abundances`
>    - "Taxonomic level for the relative abundance output" to `All taxonomic levels`
>
{: .hands_on}

3 files are generated by MetaPhlAn2:

- A tabular file with the community structure

    ```
    #SampleID   Metaphlan2_Analysis
    k__Bacteria 100.0
    k__Bacteria|p__Proteobacteria   86.20712
    k__Bacteria|p__Actinobacteria   13.79288
    k__Bacteria|p__Proteobacteria|c__Gammaproteobacteria    86.20712
    k__Bacteria|p__Actinobacteria|c__Actinobacteria 13.79288
    ```

    Each line contains a taxa and its relative abundance found for our sample. The file starts with high level taxa (kingdom: `k__`) and go to more precise taxa.


- A BIOM file with the same information as the previous file but in BIOM format

    It can be used then by mothur and other tools requiring community structure information in BIOM format

- A SAM file with the results of the mapping of the sequences on the reference database

> ### {% icon question %} Questions
>
> 1. What is the most precise level we have access to with MetaPhlAn2?
> 2. What are the two orders found in our sample?
> 3. What is the most abundant family in our sample?
>
>    <details>
>    <summary>Click to view answers</summary>
>    <ol type="1">
>    <li>We have access to species level</li>
>    <li>Pseudomonadales and Solirubrobacterales are found in our sample</li>
>    <li>The most abundant family is Pseudomonadaceae with 86.21 % of the assigned sequences</li>
>    </ol>
>    </details>
{: .question}

## Community structure visualization

The generated files remains difficult to inspect. Visualization would help. Here, we will look at 3 tools that can be used to visualize the community structure in our sample.

Krona ([Ondov et al, 2011](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-12-385)) is a visualization tool for intuitive exploration of relative abundances of taxonomic classifications. It produces an interactive HTML file

> ### {% icon hands_on %} Hands-on: Interactive visualization with KRONA
>
> 1. **Format MetaPhlAn2 output for Krona** {% icon tool %} with
>    - "Input file" to `Community profile` output of `MetaPhlAn2`
>
> 2. **KRONA pie chart** {% icon tool %} with
>    - "What is the type of your input data" as `MetaPhlan`
>    - "Input file" to the output of `Format MetaPhlAn2`
{: .hands_on}

> ### {% icon question %} Questions
>
> 1. What are the main species found for the bacteria?
>
>    <details>
>    <summary>Click to view answers</summary>
>    <ol type="1">
>    <li></li>
>    </ol>
>    </details>
{: .question}

[GraPhlAn](https://bitbucket.org/nsegata/graphlan/wiki/Home>) is a tool for producing circular static representation of taxonomic analyses, easily exportable.

> ### {% icon hands_on %} Hands-on: Static visualization with GraPhlAn
>
> 1. **Export to GraPhlAn** {% icon tool %} with
>    - "Input file" to `Community profile` output of `MetaPhlAn2`
>    - "Use a LEfSe output file as input?" to `No`
>    - "List which levels should be annotated in the tree" to `5`
>    - "List which levels should use the external legend for the annotation" to `6,7`
>    - "Title font size" to `15`
>    - "Default size for clades not found as biomarkers" to `10`
>    - "Minimum value of clades that are biomarkers" to `0`
>    - "Maximum value of clades that are biomarkers" to `250`
>    - "Font size for the annotation legend" to `11`
>    - "Minimun abundance value for a clade to be annotated" to `0`
>    - "Number of clades to highlight" to `100`
>
>    > ### {% icon comment %} Comments
>    > We decide to display the maximum of clade (100, here). If you want more or less, you can modulate the number of clades to highlight. And if you want to change displayed annotations, you can change levels to annotate.
>    {: .comment}
>
> 2. **Generation, personalization and annotation of tree for GraPhlAn** {% icon tool %} with
>    - "Input tree" to the tree generated by the previous tool
>    - "Annotation file" to the annotation file generated by the previous tool
> 3. **GraPhlAn** {% icon tool %} with
>    - "Input tree" to the tree generated in the last step
>    - "Output format" to `PNG`
>
{: .hands_on}

With our dataset, we obtain a nice graphical representation of taxonomic diversity inside our sample, with circle radius being proportional to relative abundance of the corresponding clade.

> ### {% icon question %} Questions
>
> 1. What are the main species found for the bacteria?
> 2. Is the main species the same as the one observed with KRONA? 
>
>    <details>
>    <summary>Click to view answers</summary>
>    <ol type="1">
>    <li></li>
>    </ol>
>    </details>
>
{: .question}

[Phinch](http://phinch.org/) is an open-source framework for visualizing biological data. 

Galaxy runs an instance of Phinch which is directly accessible for any BIOM file.

> ### {% icon hands_on %} Hands-on: Visualisation with Phinch
>
> 1. Click on the BIOM file generated by MetaPhlAn2
> 2. Click on the "Visualize" icon and then on "Phinch"
>   
>    It will lead you to the Phinch website, which will automatically load in your file, and where you can several interactive visualisations:
>
{: .hands_on}

# Functional analyses

Investigation of the structure composition gives an insight on "What organisms are present in our sample". We now want to know "What are they doing in that environment?". 

With shotgun data, we have full genome information and particularly the sequence of genes. To determine which functions are done by the micro-organisms in the studied environment, we need then to identify genes, associate them to functions, combine such information to reconstruct metabolic pathways, ...

The first step is then to identify sequences and affiliate them to a known genes, using available database. [HUMAnN2](http://huttenhower.sph.harvard.edu/humann2) is a tool to profile the presence/absence and abundance of gene families and microbial pathways in a community from metagenomic or metatranscriptomic sequencing data.

> ### {% icon hands_on %} Hands-on: Metabolism function identification
>
> 1. **HUMAnN2** {% icon tool %} with
>    - "Input sequence file" to the dereplicated sequences
>    - "Use a custom taxonomic profile?" to `Yes`
>    - "Taxonomic profile file" to the community profile file generated by MetaPhlAn2
>    - "Nucleotide database" to `Locally cached`
>    - "Nucleotide database" to `Full`
>    - "Software to use for translated alignment" to `Diamond`
>    - "Protein database" to `Locally cached`
>    - "Protein database" to `Demo`
>    - "Search for uniref50 or uniref90 gene families?" to `uniref50`
>    - "Database to use for pathway computations" to `MetaCyc`
>
{: .hands_on}

HUMAnN2 generates 3 files

- A file with the abundance of each gene family in the community

    Gene families are groups of evolutionarily-related protein-coding sequences that often perform similar functions. Gene family abundance at the community level is stratified to show the contributions from known and unknown species. Individual species' abundance contributions sum to the community total abundance.

    Gene family abundance is reported in RPK (reads per kilobase) units to normalize for gene length. It reflects the relative gene (or transcript) copy number in the community.

    "UNMAPPED" value is the total number of reads which remain unmapped after both alignment steps (nucleotide and translated search). Since other gene features in the table are quantified in RPK units, "UNMAPPED" can be interpreted as a single unknown gene of length 1 kilobase recruiting all reads that failed to map to known sequences.

- A file with the coverage of pathways

    Pathway coverage provides an alternative description of the presence (1) and absence (0) of pathways in a community, independent of their quantitative abundance.

- A file with the abundance of pathways

> ### {% icon question %} Questions
>
> 1. How many gene families are found? And pathways?
>
>    <details>
>    <summary>Click to view answers</summary>
>    We can not access directly this information: in the generated, the gene families and pathways are repeated for the different species contributing to these gene families. We then need to use a tool to extract only the lines without information about the involved species. We found then that 29,661 gene families and 333 pathways are found.
>    </details>
{: .question}

The RPK for the gene families are quite difficult to interpret in term of relative abundance. We decide then to normalize the values

> ### {% icon hands_on %} Hands-on: Normalize the gene family abundances
>
> 1. **Renormalize a HUMAnN2 generated table** {% icon tool %} with
>    - "Gene/pathway table" to the gene family table generated with `HUMAnN2`
>    - "Normalization scheme" to `Relative abundance`
>    - "Normalization level" to `Normalization of all levels by community total`
>
>  > ### {% icon question %} Questions
>  >
>  > 1. Which percentage of sequences has not be assigned to a gene family?
>  > 2. What is the most abundant gene family?
>  > 3. Which genus is the most involved in "UniRef50_P77221"?
>  >
>  >    <details>
>  >    <summary>Click to view answers</summary>
>  >    <ol type="1">
>  >    <li>10.42% of the sequences has not be assigned to a gene family</li>
>  >    <li>The most abundant gene family with 0.1% of sequences is a unknown UniRef50 gene families. </li>
>  >    <li>Salmonella</li>
>  >    </ol>
>  >    </details>
>  {: .question}
{: .hands_on}

With the HUMAnN2 output, we have access to UniRef50 gene families. However, the names can remains cryptic and sometimes we would like a more general view about the functions. HUMAnN proposes a tool to regroup the gene families into different meta-groups: GO (Gene Ontology), EC, etc.

> ### {% icon hands_on %} Hands-on: Regroup the gene families into GO terms
>
> 1. **Regroup a HUMAnN2 generated table by features** {% icon tool %} with
>    - "Gene/pathway table" to the gene family table generated with `HUMAnN2`
>    - "How to combine grouped features?" to `Sum`
>    - "Use built-in grouping options?" to `Yes`
>    - "Gene family type" to `UniRef50 gene families`
>    - "Grouping options" to `UniRef50 gene families into GO`
>
> 2. **Renormalize a HUMAnN2 generated table** {% icon tool %} with
>    - "Gene/pathway table" to the gene family table generated with `Regroup`
>    - "Normalization scheme" to `Relative abundance`
>    - "Normalization level" to `Normalization of all levels by community total`
>
> 3. **Select lines that match an expression**  {% icon tool %} with
>    - "Select lines from" to the renormalized regrouped table
>    - "that" to `NOT Matching`
>    - "the pattern" to `\|`
>
>  > ### {% icon question %} Questions
>  >
>  > 1. What is the abundance of the first GO term?
>  > 2. What is related to in [Gene Ontology](http://www.geneontology.org/)?
>  >
>  >    <details>
>  >    <summary>Click to view answers</summary>
>  >    <ol type="1">
>  >    <li>GO:0000015 (found after sorting) with 0.0025%</li>
>  >    <li>It seems to correspond to "phosphopyruvate hydratase complex"</li>
>  >    </ol>
>  >    </details>
>  {: .question}
>
{: .hands_on}

The 3 generated files give detailed insights into gene families and pathways. This is interesting when we want to look to a particular pathway or to check abundance of a given gene families. However, when we want a broad overview of metabolic processes in a community, we need tools to regroup gene families or pathways into global categories.

To get global categories from HUMAnN2 outputs, we decide to use the [Gene Ontology](http://geneontology.org/>) to describe the gene families in terms of their associated biological processes, cellular components and molecular functions.

# Combination of taxonomic and functional results

With the previous analyses, we can now give some answers to the questions "Which micro-organims are present in my sample?" and "What function are done by the micro-organisms in my sample?". One remaining question stays unanswered: "Which micro-organisms are implied in the realization of a given function?".

To answer this question, we need to relate generated taxonomic and functional results. 

> ### {% icon hands_on %} Hands-on: Combination of taxonomic and functional results
>
> 1. **Combine... ** {% icon tool %} Run **Combine** on the gene family abundance generated with HUMAnN2 and the MetaPhlAn2 output
> 
>    > ### {% icon question %} Questions
>    >
>    > 1. 
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li></li>
>    >    <li></li>
>    >    </ol>
>    >    </details>
>    {: .question}
>
> 2. 
{: .hands_on}

# Conclusion
{:.no_toc}

Conclusion about the technical key points. And then relation between the technics and the biological question to end with a global view.