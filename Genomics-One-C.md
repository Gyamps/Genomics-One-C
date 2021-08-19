[toc]

# Identification of somatic and germline variants from tumor and normal sample pairs

## Introduction

Cancer cells have deviated from the normal (germline) genome of the organism by acquiring and selecting for a set of mutations which enable them to grow rapidly and invasively, to resist regulation and/or possibly to metastasize. These changes can be simple single-base mutations to more complex genomic gain, loss or structural change events. The changes can then trigger the cancer process by modifying the function of a protein (e.g. disabling a tumor suppressor gene, or activating an oncogene), silencing a gene’s transcription or affecting a gene’s transcriptional affinity.

Many studies have elected to sample and sequence both the tumor tissue with a normal genomic profile from the same individual in order to separate the acquired (somatic) mutations from the malignant tissue. However, this solution is not adequate and simple as comparison being done because even in healthy tissues, there are many thousands of variants compared to the reference genome. The reason for this being that every individual inherit  a unique pattern of many variants from their parents.

> A major difference between ***germline variants*** and ***somatic variants*** is that: germline variants is inherited and present in the carrier's germline while the somatic variants are acquired from the environment, hence cannot be transmitted to the offspring.

If this is the case, it can be said that healthy and tumor tissues could possess germline variants while tumor tissues are the only ones with the somatic variants. This is why comparison between a normal cell and a tumor cell is the only way to distinguish between the two types of variant.

Furthermore, besides acquiring somatic variants, tumor tissues also could gain chromosomal copies of heterozygous germline variants  i.e. homozygous state (in the case of a duplication of one allelic copy accompanied by loss of the other) or lose i.e. hemizygous state (if one allele is simply dropped)

**Note: only one of the two original alleles persists in the tumor.**

This concept is called *loss of heterozygosity* (LOH). The detection of LOH events, again, is dependent on a comparison of tumor and normal tissue data.

**Genomics-One-C** is set out to:

- Identify somatic and germline variants, as well as variants affected by LOH, from a tumor and a normal sample of the same patient.
- Report the variant sites, and the genes affected by them, annotated with the content of general human genetic and cancer-specific databases.

We are hoping this provide the insight into the genetic events driving tumor formation and growth in patient, and might be of prognostic and even therapeutic value by revealing variants known to affect drug resistance/sensitivity, tumor aggressiveness, *etc*.

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
> 5. Variant annotation and reporting
>    1. Get data
>    2. Adding annotations to the called variants
>    3. Reporting selected subsets of variants
>    4. Generating reports of genes affected by variants
>    5. Adding additional annotations to the gene-centered report



# Data Preparation

> We must first upload and prepare some input data for analysis. The sequencing reads we'll be looking at come from real-world data from a cancer patient's tumor and normal tissue samples. However, for the sake of analysis speed, the original data has been downsampled to include only reads from human chromosomes 5, 12, and 17.



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
   >    	As an alternative to uploading the data from a URL or your computer, the files may also have been made available from a shared data _library_:
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
   > ```
   > You can skip this step if the Galaxy server you're using has a hg19 version of the human reference genome with prebuilt indexes for bwa-mem and samtools (check the tools Map with BWA-MEM tool and VarScan Somatic tool to see if a hg19 version is listed as an option under "(Using) reference genome").
   > ```

   Alternatively, load the dataset from a shared data library.

   

6. Rename the reference genome

   The reference genome you imported above came as a compressed file, but Galaxy unpacked it to plain <span style='color:red'>`fasta`</span>  format based on your datatype selection. You might want to remove the <span style='color:red'>`.gz`</span>  suffix from the dataset name now.

   

# Quality control and mapping of NGS reads

It is imperative to ensure that the data used for analysis is of good quality prior to the commencement of the analysis. The quality control phase involves the identification and trimming of low-quality parts and/or ditching of poor quality reads that may be present in the input data so as to avoid spurious variant calls. Here, we ensure that all the sequencing reads used meet some minimal quality criteria.

> ##### More on quality control and mapping
>
> If you would like to explore the topics of quality control and read mapping in detail, you should take a look at the separate [Quality Control](https://training.galaxyproject.org/training-material/topics/sequence-analysis/tutorials/quality-control/tutorial.html) and [Mapping](https://training.galaxyproject.org/training-material/topics/sequence-analysis/tutorials/mapping/tutorial.html) tutorials. Here, we will only illustrate the concrete steps necessary for quality control and read mapping of our particular datasets.



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

We will apply read trimming and filtering albeit the relatively good overall quality of the raw reads to see if we can improve things still a bit, but also to demonstrate the general concept.

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
   > - *two for the trimmed forward and reverse reads that still have a proper mate in the other dataset*.
   > - *two more datasets of orphaned forward and reverse reads, for which the corresponding mate got dropped because of insufficient length after trimming; when you inspect these two files, however, you should find that they are empty because none of our relatively high quality reads got trimmed that excessively. You can delete the two datasets to keep your history more compact*.



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

   > **Tip**: *In general, you are free to choose ID and SM values to your liking, but the ID should unambiguously identify the sequencing run that produced the reads, while the SM value should identify the biological sample.*

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

To ensure that we base our variant analysis only on unambiguous, high-quality read mappings we will do some postprocessing next.

## Filtering on mapped reads properties

To produce new filtered BAM datasets with only those reads retained that have been mapped to the reference successfully, have a minimal mapping quality of 1, and for which the mate read has also been mapped:

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

When you configure multiple filters within one condition, reads have to pass *all* the filters to be retained in the output. The above settings, thus, retain only read pairs, for which both mates are mapped.

Note that filtering for a minimal mapping quality is not strictly necessary. Most variant callers (including **VarScan somatic**, which we will be using later) have an option for using only reads above a certain mapping quality. In this section, however, we are going to process the retained reads further rather extensively so it pays off in terms of performance to eliminate reads we do not plan to use at an early step.

*“Would you like to set rules?”*: <span style='color:red'>`No`</span>

> This will result in two new datasets, one for each of the normal and tumor data.

> **Tools for filtering BAM datasets** 
>
> Under the hood, **Filter SAM or BAM, output SAM or BAM** tool uses the <span style='color:red'>`samtools view`</span> command line tool, while **Filter BAM datasets on a variety of attributes** tool uses <span style='color:red'>`bamtool filter`</span>. Whatever the BAM filtering task, you should be able to perform it in Galaxy with one of these two tools.



## Removing duplicate reads

### Hands-on: Remove duplicates

1. Run **RmDup** Tool: with the following parameters:
   - *“BAM file”*: filtered reads datasets from the normal *and* the tumor tissue data; the outputs of **Filter SAM or BAM**
   - *“Is this paired-end or single end data”*: <span style='color:red'>`BAM is paired-end`</span>
   - *“Treat as single-end”*: <span style='color:red'>`No`</span>

> Again, this will produce two new datasets, one for each of the normal and tumor data.

## Left-align reads around indels

### Hands-on: Left-align

1. Run **BamLeftAlign** Tool: with the following parameters:
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

1. Run **CalMD** Tool:with the following parameters:

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

     The *VarScan somatic* tool that we are going to use for calling variants at the next step is typically used in combination with unadjusted base quality scores because the general opinion is that the base quality downgrades performed by *CalMD* and other tools from the *samtools* suite of tools are too severe for *VarScan* to retain good sensitivity. We are sticking with this practice in this tutorial.

     > **Using adjusted base quality scores**
     >
     > If, for your own data, you would like to experiment with adjusted base quality scores, it is important to understand that *VarScan somatic* will only make use of the adjusted scores if they are incorporated directly into the read base qualities of a BAM input dataset, but not if they are written to the dataset separately.
     >
     > - Hence, should you ever decide to use:
     >
     >   - *“Do you also want BAQ (Base Alignment Quality) scores to be calculated?”*: <span style='color:red'>`Yes, run BAQ calculation`</span> 
     >
     >     and you want this setting to affect downstream variant calling with *VarScan somatic*, make sure you also set then:
     >
     >     - *“Use BAQ to cap read base qualities”*: <span style='color:red'>`Yes`</span>
     >
     >   Please also note that BAQ scores are quite expensive to calculate so be prepared to see a substantial (up to 10x!) increase in job run time  when enabling it.
     >

     

   - *“Additional options”*: <span style='color:red'>`Advanced options`</span>

     - *“Change identical bases to ‘=’“*: <span style='color:red'>`No`</span>

     - *“Coefficient to cap mapping quality of poorly mapped reads”*: <span style='color:red'>`50`</span>

       This last setting is the real reason why we use CalMD at this point. It is an empirical, but well-established finding that the mapping quality of reads mapped with *bwa* should be capped this way before variant calling.

> This will, once more, produce two new datasets, one for each of the normal and tumor data.



## Refilter reads based on mapping quality

During recalibration of read mapping qualities **CalMD** may have set some mapping quality scores to 255. This special value is reserved for *undefined* mapping qualities and is used by the tool when a recalibrated mapping quality would drop below zero. In other words, a value of 255 does not indicate a particularly good mapping score, but a really poor one. To remove such reads from the data:

### Hands-on: Eliminating reads with undefined mapping quality

1. Run **Filter BAM datasets on a variety of attributes** Tool: with the following parameters:

   - *“BAM dataset(s) to filter”*: the recalibrated datasets from the normal and the tumor tissue data; the outputs of **CalMD**

   - In *“Condition”*:

   - In *“1: Condition”*:

     - In *“Filter”*:

       - *“Select BAM property to filter on”*: <span style='color:red'>`mapQuality`</span>
- *“Filter on read mapping quality (phred scale)”*: <span style='color:red'>`>=254`</span> 

# Variant Calling and Classification

Now, having generated a high-quality set of mapped read pairs, we can proceed to variant calling. On the galaxy server we employ the use of the **VarScan somatic** tool, which is a dedicated solution for somatic variant calling.

### Hands-on: Variant Calling and Classification

1. Run **VarScan somatic** with the following parameters:

   * *"Will you select a reference genome from your history or use a built-in genome?"* :  <span style='color:red'>`Use a built-in genome`</span> 

     * *"reference genome"*: <span style='color:red'>`Human: hg19`</span> (or a similarly named choice)

     
   
     > ### Using the imported <span style='color:red'>`hg19`</span> sequence
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

       We have seen, at the quality control step, that our sequencing data is of really good quality, and we have chosen not to downgrade base qualities at the quality scores recalibration step above, so we can increase the base quality required at any given position without throwing away too much of our data.

     * *"Minimum mapping quality"*: <span style='color:red'>`1`</span> 

       During post-processing, we have filtered our reads for ones with a mapping quality of at least one, but **CalMD** may have lowered some mapping qualities to zero afterwards.

     Leave all other settings in this section at their default values.
   
   * *"Settings for Posterior Variant Filtering"*: <span style='color:red'>`Use default values`</span> 

# Conclusion

In addition to merely calling variants, *somatic variant calling* tries to distinguish *somatic mutations*, which are private to tumor tissue, from *germline* mutations, that are shared by tumor and healthy tissue, and *loss-of-heterozygosity* events, which involve the loss, from tumor tissue, of one of two alleles found at a biallelic site in healthy tissue.

Dedicated somatic variant callers can perform this classification on statistical grounds, but the interpretation of any list of variants (somatic, germline or LOH) also depends crucially on rich genetic and cancer-specific variant and gene annotations.

> Some **Key points** to consider:
>
> * Follow best practices for read mapping, quality control and mapped reads post-processing to minimize false-positive variant calls.
> * Use a dedicated somatic variant caller to call variants and to classify them into somatic, germline and LOH event variants on statistical grounds.
> * Annotations and queries based on variant properties add relevance to variant and gene reports.
> * A framework like **GEMINI** is very helpful for managing, annotating and querying lists of variants in a flexible way.
> * Prefer public, free annotation sources to foster reproducibility and information sharing.