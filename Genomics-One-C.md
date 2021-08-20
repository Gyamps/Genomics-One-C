[toc]

# Identification of somatic and germline variants from tumor and normal sample pairs

## Introduction

Cancer cells have deviated from the organism's normal (germline) genome by acquiring and selecting for a set of mutations that allow them to grow rapidly and invasively, resist regulation, and possibly metastasize. These changes can range from simple single-base mutations to more complex events involving genomic gain, loss, or structural change. Modifying the function of a protein (e.g., disabling a tumor suppressor gene or activating an oncogene), silencing a gene's transcription, or affecting a gene's transcriptional affinity can then initiate the cancer process.

Many studies have chosen to sample and sequence both tumor tissue and normal genomic profiles from the same individual in order to distinguish acquired (somatic) mutations from malignant tissue. However, this solution is insufficient and as simple as the comparison being performed because there are many thousands of variants in healthy tissues compared to the reference genome. The reason for this is that each person inherits a unique pattern with many variants from their parents.

> The primary distinction between ***germline variants*** and ***somatic variants*** is that germline variants are inherited and present in the carrier's germline, whereas somatic variants are acquired from the environment and thus cannot be passed down to the offspring.

If this is the case, healthy and tumor tissues may both have germline variants, while tumor tissues are the only ones with somatic variants. This is why comparing a normal cell to a tumor cell is the only way to tell the difference between the two types of variant.

Furthermore, in addition to acquiring somatic variants, tumor tissues may gain chromosomal copies of heterozygous germline variants, i.e. homozygous state (duplication of one allelic copy accompanied by loss of the other), or lose, i.e. hemizygous state (if one allele is simply dropped).

> **Note: Only one of the two original alleles persists in the tumor.**
>
> This is known as *loss of heterozygozity* (LOH). Again, the detection of LOH events is based on a comparison of tumor and normal tissue data.

> **Genomics-One-C's** tutorial is set out to:
>
> - Identify somatic and germline variants, as well as LOH variants, in a tumor and a normal sample from the same patient.
> - Annotate the variant sites and the genes affected by them with content from general human genetic and cancer-specific databases.



We hope that this will provide insight into the genetic events that are driving tumor formation and growth in patients, as well as have prognostic and even therapeutic value by revealing variants known to affect drug resistance/sensitivity, tumor aggressiveness, *etc*.

> **Galaxy Server GEMINI tool(s) issue**
>
> We realized that the use of the *GEMINI tool*(s) which you will encounter as you get to the **Variant annotation and reporting** section was not available on the [usegalaxy.org](usegalaxy.org) server but instead on the [usegalaxy.eu](usegalaxy.eu) server hence, you will have to perform you tutorial on the latter server.



#### **Agenda**

> In this tutorial, we will cover:
>
> 1. [Data Preparation](#data-preparation)
>    1. [Get data](#get-data)
> 2. [Quality control and mapping of NGS reads](#quality-control-and-mapping-of-NGS-reads)
>    1. [Quality control](#quality-control)
>    2. [Read trimming and filtering](#read-trimming-and-filtering)
>    3. [Read mapping](#read-mapping)
> 3. [Mapped reads postprocessing](#mapped-reads-postprocessing)
>    1. [Filtering on mapped reads properties](#filtering-on-mapped-reads-properties)
>    2. [Removing duplicate reads](#removing-duplicate-reads)
>    3. [Left-align reads around indels](#left-align-reads-around-indels)
>    4. [Recalibrate read mapping qualities](#recalibrate-read-mapping-qualities)
>    5. [Refilter reads based on mapping quality](#refilter-reads-based-on-mapping-quality)
> 4. [Variant calling and classification](#variant-calling-and-classification)
> 5. [Variant annotation and reporting](#variant-annotation-and-reporting)
>    1. [Get data](#get-data)
>    2. [Adding annotations to the called variants](#adding-annotations-to-the-called-variants)
>    3. [Reporting selected subsets of variants](#reporting-selected-subsets-of-variants)
>    4. [Generating reports of genes affected by variants](#generating-reports-of-genes-affected-by-variants)
>    5. [Adding additional annotations to the gene-centered report](#adding-additional-annotations-to-the-gene-centered-report)



# Data Preparation

We must first upload and prepare some input data for analysis. The sequencing reads we'll be looking at come from real-world data from a cancer patient's tumor and normal tissue samples. However, for the sake of analysis speed, the original data has been downsampled to include only reads from human chromosomes 5, 12, and 17.



## Get data

### Hands-on: Data upload

1. Create a new history for this tutorial and give it a meaningful name. eg. *Identification of somatic and germline variants from tumor and normal sample pairs (Genomics-One-C)*

    > :bulb:**Tip: Creating a new history**
    >
    > Click the :heavy_plus_sign: icon at the top of the history panel to the right of your screen.
    >
    > If the :heavy_plus_sign: is missing:
    >
    > 2. Click on the :gear: icon (**History options**) on the top of the history panel
    > 2. Select the option **Create New** from the menu

   > :bulb:**Tip: Renaming a history**
   >
   > 1. Click on **Unnamed history** (or the current name of the history)
   >
   >    (**Click to rename history**) at the top of your history panel
   >
   > 2. Type the new name
   >
   > 3. Press **`Enter`**

2. Import the following four files from [Zenodo](https://zenodo.org/record/2582555):

   >```
   >https://zenodo.org/record/2582555/files/SLGFSK-N_231335_r1_chr5_12_17.fastq.gz
   >https://zenodo.org/record/2582555/files/SLGFSK-N_231335_r2_chr5_12_17.fastq.gz
   >https://zenodo.org/record/2582555/files/SLGFSK-T_231336_r1_chr5_12_17.fastq.gz
   >https://zenodo.org/record/2582555/files/SLGFSK-T_231336_r2_chr5_12_17.fastq.gz
   >```

   where the first two files represent data from a patient's normal tissue's forward and reverse reads sequences, and the last two files represent data from the same patient's tumor tissue.

   Alternatively, the same files may be available on your Galaxy server via a shared data library (ask your instructor), in which case you should import the data directly from there.

   >  :bulb:**Tip: Importing via links**
   >
   > * ​	Copy the link location
   > * Open the Galaxy Upload Manager (on the top-left of the tool panel)
   > * Select _**Paste/Fetch Data**_
   > * Paste the link into the text field
   > * Change _**Type (set all)**_: from “Auto-detect” to <span style='color:red'>`fastqsanger.gz`</span>
   > * Press _**Start**_
   > * _**Close**_ the window
   > * By default, Galaxy uses the URL as the name, so rename the files with a more useful name.

   > :bulb:**Tip: Importing data from a data library**
   >
   > As an alternative to uploading the data from a URL or your computer, the files may also have been made available from a shared data _library_:
   >
   > * Go into _**Shared data**_ (top panel) then _**Data libraries**_
   > * Find the correct folder (ask your instructor)
   > * Select the desired files
   > * Click on the _**To History**_ button near the top and select _**as Datasets**_ from the dropdown menu
   > * In the pop-up window, select the history you want to import the files to (or create a new one)
   > * Click on _**Import**_

   

3. Check that all newly created datasets in your history have the correct datatypes assigned, and correct any missing or incorrect datatype assignment.

   > :bulb:**Tip: Changing the datatype**
   >
   > * Click on the _**pencil icon**_ for the dataset to edit its attributes
   > * In the central panel, click on the galaxy-chart-select-data _**Datatypes**_ tab on the top
   > * Select <span style='color:red'>`fastqsanger.gz`</span>
   > * Click the _**Change datatype**_ button



4. Rename the datasets and tag them appropriately.

   When you upload a dataset via a link, Galaxy will use the link address as the dataset name, which you should probably shorten to just the file names

   > :bulb:**Tip: Renaming a dataset**
   >
   > * Click on the galaxy-pencil _**pencil icon**_ for the dataset to edit its attributes
   > * In the central panel, change the _**Name**_ field
   > * Click the _**Save**_ button

   A large portion of the analysis in this tutorial will consist of identical steps performed in parallel on normal and tumor tissue data.

   Galaxy supports dataset tags to make it easier to remember which dataset represents which branch of an analysis in a linear history. In particular, if you add a tag beginning with <span style='color:red'>`#`</span> to any dataset, that tag will automatically propagate to any new dataset derived from the tagged dataset.

   > :bulb:**Tip: Adding a tag**
   >
   > * Click on the dataset
   > * Click on **Edit dataset tags**
   > * Add a tag starting with <span style='color:red'>`#`</span>
   > * Tags starting with <span style='color:red'>`#`</span> will be automatically propagated to the outputs of tools using this dataset.
   > * Check that the tag is appearing below the dataset name

   Before we begin our analysis, it is a good idea to tag the two fastq datasets representing normal tissue with, for example, <span style='color:red'>`#normal`</span> and the two datasets representing tumor tissue with, for example, <span style='color:red'>`#tumor`</span>.



5. Import the reference genome with the <span style='color:red'>`hg19`</span> version of the sequences of human chromosomes 5, 12 and 17:

   > ```english
   > https://zenodo.org/record/2582555/files/hg19.chr5_12_17.fa.gz
   > ```

   In the import dialog, make sure to select <span style='color:red'>`fasta`</span>  as the datatype.

   > **Shortcut**
   >
   > You can skip this step if the Galaxy server you're using has a hg19 version of the human reference genome with prebuilt indexes for bwa-mem and samtools (check the tools Map with BWA-MEM tool and VarScan Somatic tool to see if a hg19 version is listed as an option under "(Using) reference genome").

   Alternatively, load the dataset from a shared data library.

   

6. Rename the reference genome

   The reference genome you imported above came as a compressed file, but Galaxy unpacked it to plain <span style='color:red'>`fasta`</span>  format based on your datatype selection. You might want to remove the <span style='color:red'>`.gz`</span>  suffix from the dataset name now.

   

# Quality control and mapping of NGS reads

Prior to beginning the analysis, it is critical to ensure that the data used for analysis is of high quality. The quality control phase entails identifying and trimming low-quality parts as well as ditching low-quality reads that may be present in the input data to avoid spurious variant calls. In this step, we ensure that all sequencing reads used meet some basic quality criteria.

> ##### More on quality control and mapping
>
> If you want to learn more about quality control and mapping, check out the [Quality Control](https://training.galaxyproject.org/training-material/topics/sequence-analysis/tutorials/quality-control/tutorial.html) and [Mapping](https://training.galaxyproject.org/training-material/topics/sequence-analysis/tutorials/mapping/tutorial). We will only show the concrete steps required for quality control and read mapping of our specific datasets in this section.



## Quality control

#### Hands-on: Quality Control of the input datasets

1. Run **FastQC** on each of your four fastq datasets.

   * *"Short read data from your current history"*: all 4 FASTQ datasets selected with **Multiple datasets**

     > Tip: Select multiple datasets
     >
     > 1. Click on **Multiple datasets**
     > 2. Select several files by keeping the <span style='color:black'>Ctrl</span> (or <span style='color:BLACK'>`COMMAND`</span>) key pressed and clicking on the files of interest

     _8 new datasets (one with the calculated raw data, another one with an html report of the findings for each input dataset) wil get added to your history after you run this job._

2. Use **MultiQC** to aggregate the raw **FastQC** data of all four input datasets into one report

   * In *"Results"*
     * *"Which tool was used to generate logs?"*: <span style='color:red'>`FastQC`</span>
     * In *"FastQC output"*
       * *"Type of FastQC output?"*: <span style='color:red'>`Raw data`</span>
       * *"FastQC output"*: all four *RawData* outputs of **FastQC** 

3. Inspect the *Webpage* output produced by the tool

## Read trimming and filtering

Despite the relatively good overall quality of the raw reads, we will apply read trimming and filtering to see if we can improve things further, but also to demonstrate the general concept.

#### **Hands-on: Read trimming and filtering of the normal tissue reads**

1. Run the **Trimmomatic** tool to trim and filter the normal tissue reads

   - *“Single-end or paired-end reads?”*: <span style='color:red'>`Paired-end (two separate input files)`</span>

     This makes the tool treat the forward and reverse reads simultaneously.

     - *“Input FASTQ file (R1/first of pair)”*: the forward reads (r1) dataset of the normal tissue sample

     - *“Input FASTQ file (R2/second of pair)”*: the reverse reads (r2) dataset of the normal tissue sample

   - *“Perform initial ILLUMINACLIP step?”*: <span style='color:red'>`Yes`</span>

     - *Select standard adapter sequences or provide custom?”*: <span style='color:red'>`Standard`</span>

       - *"Adapter sequences to use”*: <span style='color:red'>`TrueSeq3 (paired-ended, for MiSeq and HiSeq)`</span>

     - *“Maximum mismatch count which will still allow a full match to be  performed”*: <span style='color:red'>`2`</span>

     - *“How accurate the match between the two ‘adapter ligated’ reads must  be for PE palindrome read alignment”*: <span style='color:red'>`30`</span>

     - *“How accurate the match between any adapter etc. sequence must be  against a read”*: <span style='color:red'>`10`</span>

     - *“Minimum length of adapter that needs to be detected (PE specific/  palindrome mode)”*: <span style='color:red'>`8`</span>

       *“Always keep both reads (PE specific/palindrome mode)?”*: <span style='color:red'>`Yes`</span>

   These parameters are used to cut ILLUMINA-specific adapter sequences from the reads.

   - In *“Trimmomatic Operation”*
     - In *“1: Trimmomatic Operation”*
       - *“Select Trimmomatic operation to perform”*: <span style='color:red'>`Cut the specified number of bases from the start of the read (HEADCROP)`</span>
         - *“Number of bases to remove from the start of the read”*: <span style='color:red'>`3`</span>
     - “Insert Trimmomatic Operation”*
     - In *“2: Trimmomatic Operation”*
       - *“Select Trimmomatic operation to perform”*: <span style='color:red'>`Cut bases off the end of a read, if below a threshold quality (TRAILING)`</span>
         - *“Minimum quality required to keep a base”*: <span style='color:red'>`10`</span>
     - “Insert Trimmomatic Operation”*
     - In *“3: Trimmomatic Operation”*
       - *“Select Trimmomatic operation to perform”*: <span style='color:red'>`Drop reads below a specified length (MINLEN)`</span>
         - *“Minimum quality required to keep a base”*: <span style='color:red'>`25`</span>

   These three trimming and filtering operations will be applied to the reads in the given order after adapter trimming.

   > Running this job will generate four output datasets:
   >
   > - *two for the trimmed forward and reverse reads with a proper mate in the other dataset*
   > - *two more datasets of orphaned forward and reverse reads, for which the corresponding mate was dropped due to insufficient length after trimming; however, when you inspect these two files, you should find that they are empty because none of our relatively high quality reads were excessively trimmed. You can delete the two datasets to make your history smaller*.



## Read Mapping

#### Hands-on: Read Mapping

1. Use **Map with BWA-MEM** Tool: to map the reads from the **normal tissue** sample to the reference genome      

   - *“Will you select a reference genome from your history or use a built-in index?”*:  <span style='color:red'>`Use a built-in genome`</span>

     - *“Using reference genome”*: <span style='color:red'>`Human: hg19`</span> (or a similarly named option)

       >### Using the imported <span style='color:red'>`hg19`</span> sequence
       >
       >In the event you have imported the <span style='color:red'>`hg19`</span> sequence as a fasta dataset into your history, you could instead:
       >
       >* *"Will you select a reference genome from your history or use a built-in genome?"*: <span style='color:red'>`Use a genome from the history`</span> 
       >
       >* *"reference genome"*: your imported <span style='color:red'>`hg19`</span> fasta dataset.


​     

   - “Single or Paired-end reads”*: <span style='color:red'>`Paired`</span> 

     * *“Select first set of reads”*: the trimmed  forward reads (r1) dataset of the **normal tissue** sample; output of **Trimmomatic**
     * *“Select second set of reads”*: the trimmed reverse reads (r2) dataset of the **normal tissue** sample; output of **Trimmomatic**

   - *“Set read groups information?”*: <span style='color:red'>`Set read groups (SAM/BAM specification)`</span>
     - *“Auto-assign”*: <span style='color:red'>`No`</span>
       - *“Read group identifier (ID)”*: <span style='color:red'>`231335`</span> (this value being taken from the original name of the normal tissue input files)
     - *“Auto-assign”*: <span style='color:red'>`No`</span>
       - *“Read group sample name (SM)”*: <span style='color:red'>`Normal`</span>

   > **Tip**: *In general, you are free to choose your own ID and SM values, but the ID should unambiguously identify the sequencing run that generated the reads, while the SM value should identify the biological sample.*

2. Use **Map with BWA-MEM** to map the reads from the **tumor tissue** sample,      

   - *“Will you select a reference genome from your history or use a built-in index?”*: <span style='color:red'>`Use a built-in genome`</span>

     - *“Using reference genome”*: <span style='color:red'>`Human: hg19`</span> (or a similarly named option)

     Adjust these settings as before if you are using the imported reference genome.

   - *“Single or Paired-end reads”*: <span style='color:red'>`Paired`</span>

     - *“Select first set of reads”*: the trimmed forward reads (r1) dataset of the **tumor tissue** sample; output of **Trimmomatic**
     - *Select second set of reads”*: the reverse reads (r2) dataset of the **tumor tissue** sample; output of **Trimmomatic**

   - *“Set read groups information?”*: <span style='color:red'>`Set read groups (SAM/BAM specification)`</span>

     - *“Auto-assign”*: `No`
       - *Read group identifier (ID)”*: `231336` (this value, again, being taken from the original name of the tumor tissue input files)
     - *“Auto-assign”*: <span style='color:red'>`No`</span>
       * *"Read group sample name (SM)"*: <span style='color:red'>`Tumor`</span>





# Mapped reads postprocessing

We will then perform some postprocessing to ensure that our variant analysis is based solely on unambiguous, high-quality read mappings.

## Filtering on mapped reads properties

To generate new filtered BAM datasets with only reads retained that have been successfully mapped to the reference, have a minimum mapping quality of 1, and have the mate read also mapped:

### Hands-on: Filtering for mapping status and quality

1. Run **Filter BAM datasets on a variety of attributes** with the following parameters:

   -  *“BAM dataset(s) to filter”*: mapped reads datasets from the normal *and* the tumor tissue data, outputs of **Map with BWA-MEM** 

   -  In *“Condition”*:

   -  In *“1: Condition”*:

      - In *“Filter”*:

        - In *“1: Filter”*: 

          - *“Select BAM property to filter on”*: <span style='color:red'>`mapQuality`</span>
        - ''*Filter on read mapping quality (phred scale)”*: <span style='color:red'>`>=1`</span>
        - In *“2: Filter”*:
             - *“Select BAM property to filter on”*: <span style='color:red'>`isMapped`</span> 
               - *“Selected mapped reads”*: <span style='color:red'>`Yes`</span>
        - Click on *“Insert Filter”*
        - In *“3: Filter”*:                  
             - *“Select BAM property to filter on”*: <span style='color:red'>`isMateMapped`</span>
               - *“Select reads with mapped mate”*: <span style='color:red'>`Yes`</span>

When multiple filters are configured within a single condition, reads must pass *all* of the filters in order to be retained in the output. As a result, the above settings retain only read pairs for which both mates are mapped.

It should be noted that filtering for the best mapping quality is not strictly necessary. Most variant callers (including **VarScan somatic**, which we will use later) allow us to use only reads with mapping quality above a certain threshold. However, in this section, we will process the retained reads in greater depth, so it will pay off in terms of performance to eliminate reads we do not intend to use at an early stage.

*“Would you like to set rules?”*: <span style='color:red'>`No`</span>

> This will result in two new datasets, one for each of the normal and tumor data.

> **Tools for filtering BAM datasets** 
>
> Under the hood, **Filter SAM or BAM, output SAM or BAM** tool uses the <span style='color:red'>`samtools view`</span> command line tool, while **Filter BAM datasets on a variety of attributes** tool uses <span style='color:red'>`bamtool filter`</span>. Whatever the BAM filtering task, you should be able to perform it in Galaxy with one of these two tools.



## Removing duplicate reads

### Hands-on: Remove duplicates

1. Run **RmDup** Tool with the following parameters:
   - *“BAM file”*: filtered reads datasets from the normal *and* the tumor tissue data; the outputs of **Filter SAM or BAM**
   - *“Is this paired-end or single end data”*: <span style='color:red'>`BAM is paired-end`</span>
   - *“Treat as single-end”*: <span style='color:red'>`No`</span>

> Again, this will produce two new datasets, one for each of the normal and tumor data.

## Left-align reads around indels

### Hands-on: Left-align

1. Run **BamLeftAlign** Tool with the following parameters:
   - *“Choose the source for the reference genome”*: <span style='color:red'>`Locally cached`</span> 
     - *“BAM dataset to re-align”*: your filtered and deduplicated reads datasets from the normal *and* the tumor tissue data; the outputs of **RmDup**    
     - *“Using reference genome”*: <span style='color:red'>`Human: hg19`</span> (or a similarly named choice)
   - *“Maximum number of iterations”*: <span style='color:red'>`5`</span>

> **Using the imported `hg19` sequence**
>
> If you have imported the <span style='color:red'>`hg19`</span> sequence as a fasta dataset into your history instead:
>
> - *“Choose the source for the reference genome”*: <span style='color:red'>`History`</span>
>   - *“Using reference file”*: your imported <span style='color:red'>`hg19`</span> fasta dataset

> As before, this will generate two new datasets, one for each of the normal and tumor data.



## Recalibrate read mapping qualities

### Hands-on: Recalibrate read quality scores

1. Run **CalMD** Tool with the following parameters:

   - *“BAM file to recalculate”*: the left-aligned datasets from the normal and the tumor tissue data; the outputs of **BamLeftAlign**

   - *“Choose the source for the reference genome”*: <span style='color:red'>`Use a built-in genome`</span>

     - *“Using reference genome”*: <span style='color:red'>`Human: hg19`</span> (or a similarly named choice)

       > **Using the imported <span style='color:red'>`hg19`</span> sequence    **
       >
       > If you have imported the <span style='color:red'>`hg19`</span> sequence as a fasta dataset into your history instead:
       >
       > - *“Choose the source for the reference genome”*: <span style='color:red'>`Use a genome from the history`</span>
       >   - *“Using reference file”*: your imported <span style='color:red'>`hg19`</span> fasta dataset.

       

   - *“Do you also want BAQ (Base Alignment Quality) scores to be calculated?”*: <span style='color:red'>`No`</span>

     The *VarScan* somatic tool, which we will use to call variants in the next step, is typically used in conjunction with unadjusted base quality scores because it is widely assumed that the base quality downgrades performed by *CalMD* and other tools in the *samtools* suite of tools are too severe for *VarScan* to retain good sensitivity. In this tutorial, we will continue with this practice.

     > **Using adjusted base quality scores**
     >
     > - If you want to experiment with adjusted base quality scores for your own data, keep in mind that *VarScan somatic* will only use them if they are incorporated directly into the read base qualities of a BAM input dataset, not if they are written to the dataset separately. Hence, should you ever decide to use:
     >
     >   - *“Do you also want BAQ (Base Alignment Quality) scores to be calculated?”*: <span style='color:red'>`Yes, run BAQ calculation`</span> 
     >
     >     and you want this setting to affect downstream variant calling with *VarScan somatic*, make sure you also set then:
     >
     >     - *“Use BAQ to cap read base qualities”*: <span style='color:red'>`Yes`</span>
     >
     >   Please keep in mind that BAQ scores are quite expensive to calculate, so expect a significant (up to 10x!) increase in job run time when you enable it.
     >
     
     

   - *“Additional options”*: <span style='color:red'>`Advanced options`</span>

     - *“Change identical bases to ‘=’“*: <span style='color:red'>`No`</span>

     - *“Coefficient to cap mapping quality of poorly mapped reads”*: <span style='color:red'>`50`</span>

       This last option is the main reason we're using CalMD at this point. The mapping quality of reads mapped with *bwa* should be capped in this way before variant calling, according to empirical but well-established findings.

> This will, once more, produce two new datasets, one for each of the normal and tumor data.



## Refilter reads based on mapping quality

**CalMD** may have set some mapping quality scores to 255 during read mapping quality recalibration. This special value is reserved for *undefined* mapping qualities and is used by the tool when a recalibrated mapping quality is less than zero. In other words, a value of 255 indicates a really bad mapping score, not a particularly good one. To remove such reads from the data, do the following:

### Hands-on: Eliminating reads with undefined mapping quality

1. Run **Filter BAM datasets on a variety of attributes** Tool: with the following parameters:

   - *“BAM dataset(s) to filter”*: the recalibrated datasets from the normal and the tumor tissue data; the outputs of **CalMD**

   - In *“Condition”*:

   - In *“1: Condition”*:

     - In *“Filter”*:

       - *“Select BAM property to filter on”*: <span style='color:red'>`mapQuality`</span>
- *“Filter on read mapping quality (phred scale)”*: <span style='color:red'>`>=254`</span> 

# Variant Calling and Classification

We can now proceed to variant calling after we have generated a high-quality set of mapped read pairs. We use the **VarScan somatic** tool on the galaxy server, which is a dedicated solution for somatic variant calling.

### Hands-on: Variant Calling and Classification

1. Run **VarScan somatic** with the following parameters:

   * *"Will you select a reference genome from your history or use a built-in genome?"* :  <span style='color:red'>`Use a built-in genome`</span> 

     * *"reference genome"*: <span style='color:red'>`Human: hg19`</span> (or a similarly named choice)

     
   
     > **Using the imported <span style='color:red'>`hg19`</span> sequence**
     >
     > In the event you have imported the <span style='color:red'>`hg19`</span> sequence as a fasta dataset into your history, you could instead:
     >
     > * *"Will you select a reference genome from your history or use a built-in genome?"*: <span style='color:red'>`Use a genome from the history`</span> 
     >
     > * *"reference genome"*: your imported <span style='color:red'>`hg19`</span> fasta dataset.

   * *"aligned reads from normal sample"*: the mapped and fully post-processed normal tissue dataset; one of the two outputs of filtering the **CalMD** outputs

   * *"aligned reads from tumor sample"*: the mapped and fully post-processed tumor tissue dataset; the other output of filtering the **CalMD** outputs.

   * *"Estimated purity (non-tumor content) of normal sample"*: <span style='color:red'>`1`</span>

   * *"Estimated purity (tumor content) of tumor sample"*: <span style='color:red'>`0.5`</span> 

   * *"Generate separate output datasets for SNP and indel calls?"*: <span style='color:red'>`No`</span> 

   * *"Settings for Variant Calling"*: <span style='color:red'>`Customize settings`</span> 

     * *"Minimum base quality"*: <span style='color:red'>`28`</span> 

       We discovered that our sequencing data is of very high quality during the quality control step, and we chose not to downgrade base qualities during the quality scores recalibration step above, so that we can increase the base quality required at any given position without discarding too much of our data.

     * *"Minimum mapping quality"*: <span style='color:red'>`1`</span> 

       We filtered our reads for those with a mapping quality of at least one during post-processing, but **CalMD** may have reduced some mapping qualities to zero afterwards.

     Leave all other settings in this section at their default values.
   
   * *"Settings for Posterior Variant Filtering"*: <span style='color:red'>`Use default values`</span> 



# Variant annotation and reporting

This tutorial will make use of variant and gene annotations from a variety of sources. The tools we'll be using in this section will handle the majority of these for us, but we'll need to import data from four different sources into Galaxy separately:

* variant annotations from [Cancer Hotspots](https://www.cancerhotspots.org/)
* variant and gene information from the [Cancer Biomarkers database](https://www.cancergenomeinterpreter.org/biomarkers) of the Cancer Genome Interpreter (CGI) project
* variant and gene information from the [CIViC](https://civicdb.org/) database
* variant annotations from [dbSNP](https://www.ncbi.nlm.nih.gov/snp)
* lists of genes annotated with the keywords *proto-oncogene* or *tumor suppressor* at [UniProt](https://www.uniprot.org/)

Each of these annotation sets has been released in the public domain or under a free data license, allowing you to use them not only for this tutorial but also for other purposes.

We generated a set of new data files tailored to the requirements of the workflow of this tutorial using the data downloaded from each of these sites and made them available through Zenodo, once again under a free data license.

## Get data

#### Hands-on: Data upload

1. Import the following **variant annotation files** from [Zenodo](https://zenodo.org/record/2581873):

   > ```
   > https://zenodo.org/record/2581873/files/hotspots.bed
   > https://zenodo.org/record/2581873/files/cgi_variant_positions.bed
   > https://zenodo.org/record/2581873/files/01-Feb-2019-CIVic.bed
   > https://zenodo.org/record/2582555/files/dbsnp.b147.chr5_12_17.vcf.gz
   > ```

   Make sure you select <span style='color:red'>`bed`</span> as the datatype for the first three files and <span style='color:red'>`vcf`</span> for the last file.

   Alternatively, add the files from a shared data library on your Galaxy server instance.

2. Import some **gene-level annotation files** from [Zenodo](https://zenodo.org/record/2581881):

   > ```
   > https://zenodo.org/record/2581881/files/Uniprot_Cancer_Genes.13Feb2019.txt
   > https://zenodo.org/record/2581881/files/cgi_genes.txt
   > https://zenodo.org/record/2581881/files/01-Feb-2019-GeneSummaries.tsv
   > ```

   and make sure you select <span style='color:red'>`tabular`</span> as their datatype, or add them from the shared data library.

3. Download **SnpEff functional genomic annotations**

   > **Shortcut**
   >
   > If the Galaxy server you're using has <span style='color:red'>`Homo sapiens: hg19`</span> as a locally installed snpEff database, you can skip this step. To see if this is the case, use the **SnpEff eff tool** under Genome source.

   Use **SnpEff Download tool** to download genome annotation database <span style='color:red'>`hg19`</span>.

4. Check that all new datasets' datatypes have been correctly set, and change them if necessary. You may also want to shorten the names of some of the datasets.

## Adding annotations to the called variants

### Adding functional genomic annotations

Certainly, not all variations are created equal. Many may be silent mutations with no effect at the amino acid level, while a few may disrupt a protein's coding sequence by introducing a premature stop codon or a frameshift. Of course, knowing which gene is affected by a variant is also important. Functional genomic annotations of this type can be added to a VCF dataset of variants with *SnpEff*.

#### Hands-on: Adding annotations with SnpEff

1. Run **SnpEff eff:wrench::gear:** with the following parameters:

   * *"Sequence changes (SNPs, MNPs, InDels)"*: the output of **VarScan somatic:wrench:**

   * *"Input format"*: <span style='color:red'>`VCF`</span>

   * *"Output format"*: <span style='color:red'>`VCF (only if input is VCF)`</span>

   * *"Genome source"*: <span style='color:red'>`Locally installed reference genome`</span>

     * *"Genome"*: <span style='color:red'>`Homo sapiens: hg19`</span> (or a similarly named option)

     
   
     > **Using the imported <span style='color:red'>`hg19`</span> SnpEff genome database**
     >
     > If you have imported the <span style='color:red'>`hg19`</span> SnpEff genome database into your history instead:
     >
     > * *"Genome source"*: <span style='color:red'>`Downloaded snpEff database in your history`</span>
     >   * *"SnpEff4.3 Genome Data"*: your imported <span style='color:red'>`hg19`</span> SnpEff dataset.
   
   * *"Produce summary Stats"*: <span style='color:red'>`No`</span>

### Adding genetic and clinical evidence-base annotations

Other interesting aspects of a variant include whether the variant has previously been observed in the human population and, if so, at what frequency. We would also like to know if a variant is known to be associated with specific diseases. To move forward with this type of genetic and clinical evidence-based annotations, we will convert our list of variants into a database that can be processed more efficiently than a VCF dataset. We will use the GEMINI tool suite for this task as well as any future work with variants.

#### Hands-on: Creating a GEMINI database from a variants dataset

1. Run **GEMINI load :wrench::gear:** with the following parameters:

   * *"VCF dataset to be loaded in the GEMINI database"*: the output of **SnpEff eff:wrench:**

   * *"The variants in this input are"*: <span style='color:red'>`annotated with snpEff`</span>

   * *"This input comes with genotype calls for its samples"*: <span style='color:red'>`Yes`</span>

     Calling sample genotypes was part of what we used VarScan somatic for.

   * *"Choose a gemini annotation source"*: select the latest available annotations snapshot (most likely, there will be only one)

   * *"Sample and family information in PED format"*: <span style='color:red'>`Nothing selected`</span>

   * *"Load the following optional content into the database"*

     * - [x] "GERP scores"*
     * - [x] *"CADD scores"*
     * - [x] *"Gene tables"*
     * - [x] *"Sample genotypes"*
     * - [x] *"variant INFO field"*

     Be careful to leave **unchecked**:

     * *"Genotype likelihoods (sample PLs)"*

       VarScan somatic does not generate these values

     * *"only variants that passed all filters"*

       It is simple and more flexible to filter for this property later

GEMINI already (silently) adds an impressive number of annotations it knows about to our variants during the database's creation (including, e.g., the frequency at which every variant has been observed in large human genome sequencing projects). All of these were obtained for free simply by converting the variants to a GEMINI database!

GEMINI also extracts a significant amount of information from the VCF input dataset for us (such as the functional genomic annotations that we added with SnpEff).

However, there are often additional annotations from other sources (not included in GEMINI) that would be useful to include. Furthermore, GEMINI is not equipped to extract some non-standard information from VCF datasets, such as some critical bits added by VarScan somatic.

The **GEMINI annotate tool** is intended to assist you with these rather common issues. It allows you to add additional annotations to variants in an already loaded GEMINI database.

As a first step, we will use the tool to add to the database some critical information generated by VarScan somatic but not recognized by GEMINI load. We are particularly interested in three values calculated by VarScan somatic for each detected variant:

* *Somatic status (SS)*

  This is a simple numeric code in which a value of <span style='color:red'>`1`</span> indicates a germline variant, a value of <span style='color:red'>`2`</span> indicates a somatic variant, and a value of <span style='color:red'>`3`</span> indicates a LOH event.

* *Germline p-value (GPV)*

  This is the error probability associated with variants that have a somatic status of 1.

* *Somatic p-value (SPV)*

  This is the error probability for status calls of 2 and 3. (somatic and LOH calls).

These values are encoded in the INFO column of the VarScan-generated VCF dataset, which we will extract and add to the GEMINI database.

#### Hands-on: Making variant call statistics accessible

1. Run **GEMINI annotate:wrench::gear:**:

   * *"GEMINI database"*: output of **GEMINI load**

   * *"Dataset to use as the annotation source"*: output of **VarScan somatic**

   * *"Strict variant-identity matching of database and annotation records (VCF format only)"*: <span style='color:red'>`Yes`</span>

     This setting is unimportant in this case because you built the GEMINI database from the same set of variants from which we are now retrieving annotations and because VarScan somatic does not call multiple alleles at a single site. However, matching on variant-identity is the desired behavior, so we might as well be explicit about it.

   * *"Type of information to add to the database variants"*: <span style='color:red'>`Specific values extracted from matching records in the annotation source (extract)`</span>

     * In *"1: Annotation extraction recipe"*:

       * *"Elements to extract from the annotation source"*: <span style='color:red'>`SS`</span>
       * *"Database column name to use for recording annotations"*: <span style='color:red'>`somatic_status`</span>
       * *"What type of data are you trying to extract?"*: <span style='color:red'>`Integer numbers`</span>
       * *"If multiple annotations are found for the same variant, store..."*: <span style='color:red'>`the first annotation found`</span>

       This is the recipe for extracting the VarScan-generated *SS* field and adding it to the GEMINI database as a new column *somatic status*.

     * :heavy_plus_sign:*"Insert Annotation extraction recipe"*

     * In *"2: Annotation extraction recipe"*:

       * *"Elements to extract from the annotation source"*: <span style='color:red'>`GPV`</span>
       * *"Database column name to use for recording annotations"*: <span style='color:red'>`germline_p`</span>
       * *"What type of data are you trying to extract?"*: <span style='color:red'>`Numbers with decimal precision`</span>
       * *"If multiple annotations are found for the same variant, store...": <span style='color:red'>`the first annotation found`</span>

       This is the recipe for extracting the VarScan-generated *GPV* field and adding it to the GEMINI database as a new column *germline_p*.

     * :heavy_plus_sign:*"Insert Annotation extraction recipe"*

     * In *"3: Annotation extraction recipe"*:

       * *"Elements to extract from the annotation source"*: <span style='color:red'>`SPV`</span>
       * *"Database column name to use for recording annotations"*: <span style='color:red'>`somatic_p`</span>
       * *"What type pf data are you trying to extract"*: <span style='color:red'>`Numbers with decimal precision`</span>
       * *"If multiple annotations are found for the same variant, store..."*:<span style='color:red'>`the first annotation found`</span>

       This is the recipe for extracting the VarScan-generated *SPV* field and adding it to the GEMINI database as a new column *somatic_p*.

Then, in addition to the annotations obtained directly from GEMINI or the input VCF dataset, we will add additional annotations. We would like to specifically add:

* more information from **dbSNP**

  GEMINI already checks all variants to see if they exist in dbSNP and, if so, stores their dbSNP IDs as part of the database creation process. As a result, we only need to extract some additional relevant information.

* information from **Cancer Hotspots**

* links to the **CiViC** database

* information from the **Cancer Genome Interpreter (CGI)**

#### Hands-on: Adding further annotations

1. Run **GEMINI annotate:wrench::gear:** to add further annotations from **dbSNP**

   * *"GEMINI database"*: the output of the last **GEMINI annotate:wrench:** run 

   * *"Dataset to use as the annotation source"*: the imported <span style='color:red'>`dbsnp.b147.chr5_12_17.vcf`</span>

   * *"Strict variant-identity matching of database and annotation records (VCF format only)"*: <span style='color:red'>`Yes`</span>

     dbSNP stores information about specific SNPs found in human populations, and we'd like to see if any of the same SNPs are found among our variants.

   * *"Type of information to add to the database variants"*: <span style='color:red'>`Specific values extracted from matching recors in the annotation source (extract)`</span>

     * In *"1: Annotation extraction recipe"*:

       * *"Elements to extract from the annotation source"*: <span style='color:red'>`SAO`</span>
       * *"Databse column name to use for recording annotations"*: <span style='color:red'>`rs_ss`</span>
       * *"What type of data are you trying to extract?"*: <span style='color:red'>`Integer numbers`</span>
       * *"If multiple annotations are found for the same variant, store..."*: <span style='color:red'>`the first annotation found`</span>

       This recipe extracts the dbSNP field and adds it as *rs_ss* to the GEMINI database.

2. Run **GEMINI annotate:wrench::gear:** to add further annotations from **cancerhotspots**

   * *"GEMINI database"*: the output of the last **GEMINI annotate:wrench:** run 

   * *"Dataset to use as the annotation source"*: the imported <span style='color:red'>`cancerhotspots_v2.bed`</span>

   * *"Strict variant-identity matching of database and annotation records (VCF format only)"*: <span style='color:red'>`Yes`</span> (with input in BED format this settign will be ignored)

     We will simply record the best q-value associated with any cancerhotspots variant that overlaps one of our variant sites for the cancerhotspots data.

   * *"Type of information to add to the database variants"*: <span style='color:red'>`Specific values extracted from matching records in the annotation source (extract)`</span>

     * In *"1: Annotation extraction recipe"*:

       * *"Elements to extract from the annotation source"*: <span style='color:red'>`5`</span>

         The q-values are stored in the fifth column of the BED dataset.

       * *"Database column name to use for recording annotations"*: <span style='color:red'>`hs_qvalue`</span>

       * *"What type of data are you trying to extract?"*: <span style='color:red'>`Numbers with decimal precision`</span>

       * *"If multiple annotations are found for the same variant, store..."*: <span style='color:red'>`the smallest of the (numeric) values`</span>

       This is the procedure for extracting the q-values of overlapping cancerhotspots sites and adding them to the GEMINI database as hs qvalue.

3. Run **GEMINI annotate:wrench::gear:** to add links to **CiViC**

   * *"GEMINI database"*: the output of the last **GEMINI annotate:wrench:** run

   * *"Dataset to use as the annotation source"**: the imported <span style='color:red'>`CiViC.bed`</span>

   * *"Strict variant-identity matching of database and annotation records (VCF format only)"*: <span style='color:red'>`Yes`</span> (with input in BED format this setting will be ignored)

     We will record the hyperlink to any variant found in the CIViC database that overlaps one of our variant sites for the CIViC data.

   * *"Type of information to add to the database variants"*: <span style='color:red'>`Specific values extracted from matching records in the annotation source (extract)`</span>

     * In *"1: Annotation extraction recipe"*:

       * *"Elements to extract from the annotation source"*: <span style='color:red'>`4`</span>

         The hyperlinks are stored in the fourth column of the BED dataset

       * *"Database column name to use for recording annotations"*: <span style='color:red'>`overlapping_civic_url`</span>

       * *"What type of data are you trying to extract?"*: <span style='color:red'>`Text (text)`</span>

       * *"If multiple annotations are found for the same variant, store..."*: <span style='color:red'>`a comma-separated list of non-redundant (text) values`</span>

         This is the recipe for extracting the hyperlinks of overlapping CIViC sites and adding them to the GEMINI database as a list of overlapping civic urls.

4. Run **GEMINI annotate:wrench::gear:** to add information from the **Cancer Genome Interpreter (CGI)**

   * *"GEMINI database"*: the output of the last **GEMINI annotate:wrench:** run

   * *"Dataset to use as the annotation source"*: the imported <span style='color:red'>`cgi_variant)positions.bed`</span>

   * *"Strict variant-identity matching of database and annotation records (VCF format only)"*: <span style='color:red'>`Yes`</span> (with input in BED format this setting will be ignored)

     For the CGI data, we will note whether any of our variant sites overlap with a variant in the CGI biomarkers database.

   * *"Type of information to add to the database variants"*: <span style='color:red'>`Binary indicator (1=found, 0=not found) of whether the variant had any match in the annotation source (boolean)`</span>

     * *"Databse column name to use for ecording annotations"*: <span style='color:red'>`in_cgidb`</span>

## Reporting selected subsets of variants

Now that we've built our GEMINI database and added annotations to it, it's time to explore the wealth of information it contains. The goal is to generate filtered variant reports that include specific classes (somatic, germline, and LOH) of high-quality variants as well as their most relevant annotations. This is accomplished through GEMINI queries that specify:

1. filters we want to apply to the variant list stored in the database
2. the pieces of information about the filtered variants that we would like to retrieve

> **The GEMINI query language**
>
> GEMINI queries are extremely flexible, allowing users to express a wide range of ideas in order to explore the variant space; however, the complete query syntax can be intimidating for beginners.
>
> In this section, we'll start with a simple query and work our way up to a more complex query in the following section.
>
> Consult the [query section of the GEMINI documentation](https://gemini.readthedocs.io/en/latest/content/querying.html) for a more detailed explanation of the query syntax. Because the GEMINI query syntax is based on the SQLite SQL dialect, the SQLite documentation, particularly its chapter on [SQLite core functions](https://sqlite.org/lang_corefunc.html), is an excellent resource for understanding more complex queries.

Let's begin by configuring a simple query to generate a list of genuine somatic variants. Our plan for retrieving them is as follows:

1. rely on the *somatic status* of the variants called by **VarScan somatic:wrench:** 
2. Discard any variants for which a significant number of supporting sequencing reads are also found in normal tissue data, or which are only supported by a very small fraction of the reads from the tumor sample.

#### Hands-on: Querying the GEMINI database for somatic variants

1. Run **GEMINI query:wrench::gear:** with:

   * *"GEMINI database"*: the fully annotated database created in the last **GEMINI annotate:wrench:** step

   * *"Build GEMINI query using"*: <span style='color:red'>`Basic variant query constructor`</span>

     * :heavy_plus_sign:*"Insert Genotype filter expression"*

       * *"Restrictions to apply to gentype values"*: <span style='color:red'>`gt_alt_freqs.NORMAL <= 0.05 AND gt_alt_freqs.TUMOR >= 0.10`</span>

       We retain only those variants supported by less than 5% of the reads in the normal sample but more than 10% of the reads in the tumor sample using this genotype-based filter.

     * *"Additional constraints expressed in SQL syntax"*: <span style='color:red'>`somatic_status = 2`</span>

       VarScan somatic has called for each variant's somatic status, which is stored in the GEMINI database (remember we used GEMINI annotate to add it). With the condition <span style='color:red'>`somatic_status = 2`</span>, we keep only the variants that passed the genotype filter above **and** were classified as somatic by the variant caller.

     * In *"Output format options"*

       * *"Type of report to generate"*: <span style='color:red'>`tabular (GEMINI default)`</span>
         * *"Add a header of column names to the output"*: <span style='color:red'>`Yes`</span>
         * *"Set of columns to include in the variant report table"*: <span style='color:red'>`Custom (report user-specified columns)`</span>
           * In *"Choose columns to include in the report"*:
             * - [x] *"chrom"*
             * - [x] *"start"*
             * - [x] *"ref"*
             * - [x] *"alt"*
           * *"Additional columns (comma-separated)"*: <span style='color:red'>`gene, aa_change, rs_ids, hs_qvalue, cosmic_ids`</span>

       In this section, we specify which columns (from the GEMINI database's variants table) we want to include in a <span style='color:red'>`tabular`</span> variant report, in the order specified.

> **How am I supposed to know these column names?**
>
> Obviously, you must know the names of the columns (in the GEMINI database tables) in order to include them in the report, but how are you supposed to know them?
>
> The [GEMINI documentation](https://gemini.readthedocs.io/en/latest/content/database_schema.html) lists the standard ones (added by GEMINI load when building the database). The non-standard columns (the ones you added with GEMINI annotate) are labeled with the names you assigned to them when you added them.
>
> Alternatively, to obtain the tables and column names of a specific database, use the **GEMINI database info tool** as follows:
>
> * *“GEMINI database”*: the database you want to explore
> * *“Information to retrieve from the database”*: <span style='color:red'>`Names of database tables and their columns`</span>

What about more sophisticated filtering?

#### Hands-on: More complex filter criteria

1. Run **GEMINI query:wrench::gear:** with the exact same settings as before, but:

   * *"Additional constraints expressed in SQL syntax"*: <span style='color:red'>`somatic_status = 2 AND somatic_p <= 0.05 AND filter IS NULL`</span>

   This translates to "variants classified as somatic with a p-value <= 0.05, but not flagged as likely false-positives".

> **SQL keywords**
>
> SQL keywords are given in uppercase in the condition above. This is not required, but it makes the syntax easier to understand.
>
> <span style='color:red'>`IS NULL`</span> can be used to determine whether a cell in a data table is empty or contains a value. <span style='color:red'>`IS NOT NULL`</span> can be used to determine whether a cell in a data table contains a value. You can use <span style='color:red'>`AND`</span> and <span style='color:red'>`OR`</span>, as well as parentheses to group conditions, to logically combine different filter criteria.

If you followed all of the steps exactly, running this job should result in a tabular dataset of 43 variants, and with the annotations in the report, it should be relatively easy to pick out a few interesting ones. However, before we get into the report's content, we could improve the report's format a little more.

#### Hands-on: SQL-based output formatting

1. Run **GEMINI query:wrench::gear:** with the exact same settings as in the last example, but:
   * In *"Output format options"*
     * *"Additional columns (comma-separated)"*: <span style='color:red'>`type, gt_alt_freqs.TUMOR, gt_alt_freqs.NORMAL, ifnull(nullif(round(max_aaf_all,2),-1.0),0) AS MAD, gene, impact_so, aa_change, ifnull(round(cadd_scaled,2), '.') AS cadd_scaled, round(gerp_bp_score,2) AS gerp_bp, ifnull(round(gerp_element_pval,2),'.') AS gerp_elemt_pval, ifnull(round(hs_qvalue,2),'.') AS hs_qvalue, in_omim, ifnull(clinvar_sig,'.') AS clinvar, ifnull(clinvar_disease_name,'.') AS clinvar_disease_name, ifnull(rs_ids,'.') AS dbsnp_ids, rs_ss, ifnull(cosmic_ids,'.') AS cosmic_ids, ifnull(overlapping_civic_url,'.') AS overlapping_civic_url, in_cgidb`</span>

This final query adds a lot more annotations to the report and also shows how to use the <span style='color:red'>`AS`</span> keyword to rename columns and some [SQLite functions](https://sqlite.org/lang_corefunc.html) to clean up the output.

> **Limitations of genotype column queries**
>
> If you're wondering why the above query doesn't use rounding on the samples' alternate allele frequencies, i.e., <span style='color:red'>`gt_alt_freqs.TUMOR`</span> and <span style='color:red'>`gt_alt_freqs.NORMAL`</span>, or why it doesn't rename these columns, it's because it would break the query.
>
> As a general rule, all columns in the variants table beginning with <span style='color:red'>`gt`</span> (the genotype columns calculated from the genotype fields of a VCF dataset) cannot be used as regular SQLite columns and must be parsed separately by GEMINI. As a result, you cannot mix them with regular SQLite elements such as functions and alias specifications when using <span style='color:red'>`AS`</span>. You may have also noticed that <span style='color:red'>`gt_alt_freqs.TUMOR`</span> and <span style='color:red'>`gt_alt_freqs.NORMAL`</span> columns do not follow the query's column order specification and end up as the last columns of the tabular report. This is yet another example of GEMINI's unique treatment of <span style='color:red'>`gt`</span> columns.

## Generating reports of genes affected by variants

Finally, using the same somatic variants we picked earlier, let's try to construct a gene-centered report.

Annotations that pertain to a full gene affected by a variant rather than the variant itself would be included in a gene-centered report. Known synonyms of an afflicted gene, its NCBI entrez number, the ClinVar phenotype linked with the gene, if any, a reference to the gene's page at CIViC.org, and so on are examples of such annotations.

Some of this data is included in every GEMINI database, but it's stored in a distinct table called <span style='color:red'>`gene_detailed`</span>, whereas we've only utilized and queried data from the <span style='color:red'>`variants`</span> table thus far.

We must join the <span style='color:red'>`variants`</span> and <span style='color:red'>`gene_detailed`</span> tables in order to retrieve information from both tables in the same query. Such an operation is not possible with the <span style='color:red'>`Basic query constructor`</span> we've been using so far, and needs building the query in advanced mode.

#### Hands-on: Turning query results into gene-centered reports

1. Run **GEMINI query:wrench::gear:** in advanced mode by choosing

   * *"Build GEMINI query using"*: <span style='color:red'>`Advanced query constructor`</span>

   * *"The query to be issued to the database"*: <span style='color:red'>`SELECT v.gene, v.chrom, g.synonym, g.hgnc_id, g.entrez_id, g.rvis_pct, v.clinvar_gene_phenotype FROM variants v, gene_detailed g WHERE v.chrom = g.chrom AND v.gene = g.gene AND v.somatic_status = 2 AND v.somatic_p <= 0.05 AND v.filter IS NULL GROUP BY g.gene`</span>

     > **Elements of the SQL query**
     >
     > The part between <span style='color:red'>`SELECT`</span> and <span style='color:red'>`FROM`</span> specifies which columns from which database tables we want to retrieve, while the part between <span style='color:red'>`FROM`</span> and <span style='color:red'>`WHERE`</span> specifies the database tables that need to be consulted and gives them simpler aliases (<span style='color:red'>`v`</span> becomes an alias for the <span style='color:red'>`variants`</span> table, <span style='color:red'>`g`</span> for the <span style='color:red'>`gene_detailed`</span> table), which we can then use throughout the query.
     >
     > The filter criteria we want to use are listed after <span style='color:red'>`WHERE`</span>. These criteria are nearly identical to those used in our previous somatic variants query, but because we are working with two tables instead of just one, we must specify which table the filter columns come from using table prefixes. Hence, <span style='color:red'>`somatic_status`</span> becomes <span style='color:red'>`v.somatic_status`</span>, and so on. Furthermore, we want to report corresponding information from the two tables, which is ensured by the additional criteria <span style='color:red'>`v.chrom = g.chrom`</span> and <span style='color:red'>`v.gene = g.gene`</span> (the SQL terminology is: we want to join the <span style='color:red'>`variants`</span> and <span style='color:red'>`gene_detailed`</span> tables on their <span style='color:red'>`chrom`</span> and <span style='color:red'>`gene`</span> columns).
     >
     > Finally, the <span style='color:red'>`GROUP BY`</span> clause specifies that we want to combine records affecting the same gene into one.

   * *"Genotype filter expression"*: <span style='color:red'>`gt_alt_freqs.NORMAL <= 0.05 AND gt_alt_freqs.TUMOR >= 0.10`</span>

     This remains unchanged from the previous somatic variants query.

## Adding additional annotations to the gene-centered report

Unfortunately, GEMINI annotate only allows you to add columns to a GEMINI database's variants table; there is no easy way to enrich the <span style='color:red'>`gene_detailed`</span> table with additional annotations. That is why, using more general-purpose Galaxy tools, we will now add such additional annotations to the tabular gene-centered report.

Annotating the GEMINI tabular gene report is a two-step process in which we first combine the report and tabular annotation sources into a larger tabular dataset, from which we then remove redundant and unwanted columns while *rearranging* the remaining ones.

**Step 1** consists of three separate *join* operations that pull in the annotations found in the three gene-based tabular datasets that you imported in the *Get Data* step of this section in a sequential fashion. To to this, we used the **Join two file** tool to add the information such as **UniProt cancer genes**, **CGI biomarkers** and **CIViC**.

#### Hands-on: Join

1. Use **Join two files:wrench::gear:** to add **UniProt cancer genes** information

   * *“1st file”*: the GEMINI-generated gene report from the previous step

   * *“Column to use from 1st file”*: <span style='color:red'>`Column: 1`</span>

   * *“2nd file”*: the imported <span style='color:red'>`Uniprot_cancer_Genes`</span> dataset

   * *“Column to use from 2nd file”*: <span style='color:red'>`Column: 1`</span>

   * *“Output lines appearing in”*: <span style='color:red'>`Both 1st & 2nd file, plus unpairable lines from 1st file. (-a 1)`</span>

     We obviously want to include a variant-affected gene in the final report even if it is not listed as a Uniprot cancer gene.

   * *“First line is a header line”*: <span style='color:red'>`Yes`</span>

   * *“Ignore case”*: <span style='color:red'>`No`</span>

   * *“Value to put in unpaired (empty) fields”*: <span style='color:red'>`0`</span>

     If you look at the <span style='color:red'>`Uniprot_Cancer_Genes`</span> dataset, you'll notice that it has two annotation columns: one that indicates whether a given gene is a proto-oncogene or not using <span style='color:red'>`1`</span> and <span style='color:red'>`0`</span>, and another that indicates tumor suppressor genes in the same way. To indicate the common case that a gene affected by a variant is neither a known proto-oncogene nor a tumor suppressor gene, we want to fill the corresponding two columns of the join result with <span style='color:red'>`0`</span>.

2. Use **Join two files:wrench:**:gear: to add **CGI biomarkers** information

   - *“1st file”*: the partially annotated dataset from the previous
   - *“Column to use from 1st file”*: <span style='color:red'>`Column: 1`</span>
   - *“2nd file”*: the imported <span style='color:red'>`cgi_genes`</span> dataset
   - *“Column to use from 2nd file”*: <span style='color:red'>`Column: 1`</span>
   - *“Output lines appearing in”*: <span style='color:red'>`Both 1st & 2nd file, plus unpairable lines from 1st file. (-a 1)`</span>
   - *“First line is a header line”*: <span style='color:red'>`Yes`</span>
   - *“Ignore case”*: <span style='color:red'>`No`</span>
   - *“Value to put in unpaired (empty) fields”*: <span style='color:red'>`0`</span>

   Inspect the input and the result dataset to make sure you understand what happened at this step.

3. Use **Join two files:wrench:**:gear: to add gene information from **CIViC**

   - *“1st file”*: the partially annotated dataset from step 2

   - *“Column to use from 1st file”*: <span style='color:red'>`Column: 1`</span>

   - *“2nd file”*: the imported <span style='color:red'>`GeneSummaries`</span> dataset

   - *“Column to use from 2nd file”*: <span style='color:red'>`Column: 3`</span>

     The gene column in the CIViC gene summaries annotation dataset is *not* the first one!

   - *“Output lines appearing in”*: <span style='color:red'>`Both 1st & 2nd file, plus unpairable lines from 1st file. (-a 1)`</span>

   - *“First line is a header line”*: <span style='color:red'>`Yes`</span>

   - *“Ignore case”*: <span style='color:red'>`No`</span>

   - *“Value to put in unpaired (empty) fields”*: <span style='color:red'>`.`</span>

   Inspect the input and the result dataset to make sure you understand what happened at this step.

Looking at the output datasets, the gene columns from each of the *join* operations was preserved from both input datasets. Furthermore, we had no control over the order in which columns were added to the report, and we couldn't exclude any columns.

In **Step 2** of the report preparation process, we will address all of these issues and rearrange the data to produce a fully annotated gene report. The tool used is **Column arrange by header name**.

#### Hands-on: Rearrange to get a fully annotated gene report

1. Run **Column arrange by header name:wrench:**:gear: configured like this:
   - *“file to rearrange”*: the final **Join** result dataset from step 3
   - In *“Specify the first few columns by name”*
     - In *“1: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`gene`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“2: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`chrom`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“3: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`synonym`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“4: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`hgnc_id`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“5: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`entrez_id`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“6: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`rvis_pct`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“7: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`is_OG`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“8: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`is_TS`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“9: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`in_cgi_biomarkers`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“10: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`clinvar_gene_phenotype`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“11: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`gene_civic_url`</span>
     - :heavy_plus_sign:*“Specify the first few columns by name”*
     - In *“12: Specify the first few columns by name”*
       - *“column”*: <span style='color:red'>`description`</span>
   - *“Discard unspecified columns”*: <span style='color:red'>`Yes`</span>

> **Alternative tool suggestion**
>
> If your Galaxy server doesn't have the Column arrange tool, it almost certainly has the **Cut columns from a table tool**, which can be used as a stand-in. This tool, however, expects a comma-separated list of column indexes, such as <span style='color:red'>`c1,c2`</span> for the first and second columns, so you must first determine the column numbers of your columns of interest.
>
> Keep in mind that the suggested tool should not be confused with *Cut columns from a table (cut)*, which does not allow you to change the order of columns!

# Conclusion

In addition to simply calling variants, *somatic variant calling* attempts to distinguish *somatic mutations*, which are unique to tumor tissue, from *germline* mutations, which are shared by tumor and healthy tissue, and *loss-of-heterozygosity* events, which involve the loss of one of two alleles found at a biallelic site in healthy tissue, from tumor tissue.

Dedicated somatic variant callers can perform this classification based on statistics, but the interpretation of any list of variants (somatic, germline, or LOH) is also critically dependent on rich genetic and cancer-specific variant and gene annotations.

> Some **Key points** to consider:
>
> * To reduce false-positive variant calls, use best practices for read mapping, quality control, and mapped reads post-processing.
> * Call variants and classify them statistically into somatic, germline, and LOH event variants using a dedicated somatic variant caller.
> * Annotations and queries based on variant properties improve the usefulness of variant and gene reports.
> * **GEMINI** is a very useful framework for managing, annotating, and querying lists of variants in a flexible manner.
> * To encourage reproducibility and information sharing, prefer public, free annotation sources.