
# Data Preparation
>First we need to upload and prepare some input data to analyze. The sequencing reads we are going to analyze are from real-world data from a cancer patient’s tumor and normal tissue samples. For the sake of an acceptable speed of the analysis, the original data has been downsampled though to include only the reads from human chromosomes 5, 12 and 17.

## Get data
### Hands-on: Data upload
1. Create a new history for this tutorial and give it a meaningful name
2. Import the following four files from Zenodo:
> https://zenodo.org/record/2582555/files/SLGFSK-N_231335_r1_chr5_12_17.fastq.gz
> 
> https://zenodo.org/record/2582555/files/SLGFSK-N_231335_r2_chr5_12_17.fastq.gz
> 
> https://zenodo.org/record/2582555/files/SLGFSK-T_231336_r1_chr5_12_17.fastq.gz
> 
> https://zenodo.org/record/2582555/files/SLGFSK-T_231336_r2_chr5_12_17.fastq.gz

> where the first two files represent the forward and reverse reads sequence data from a patient’s normal tissue, and the last two represent the data of the same patient’s tumor tissue.

> Alternatively, the same files may be available on your Galaxy server through a shared data library (your instructor may tell you so), in which case you may prefer to import the data directly from there.
  
  > * Tip: Importing via links
  > *   Copy the link location
  > *   Open the Galaxy Upload Manager (galaxy-upload on the top-right of the tool panel)
  > *   Select **Paste/Fetch Data**
  > *   Paste the link into the text field
  > *   Change **Type (set all)**: from “Auto-detect” to fastqsanger.gz
  > *   Press **Start**
  > *   **Close** the window
  > *   By default, Galaxy uses the URL as the name, so rename the files with a more useful name.
  
  > *Tip: Importing data from a data library   
  > As an alternative to uploading the data from a URL or your computer, the files may also have been made available from a shared data library:
   > *  Go into **Shared data** (top panel) then **Data libraries**
   > *  Find the correct folder (ask your instructor)
   > *  Select the desired files
   > *  Click on the **To History** button near the top and select **as Datasets** from the dropdown menu
   > *  In the pop-up window, select the history you want to import the files to (or create a new one)
   > *  Click on **Import**
3. Check that all newly created datasets in your history have their datatypes assigned correctly, and fix any missing or wrong datatype assignment
   > * Tip: Changing the datatype
   > *   Click on the galaxy-pencil **pencil icon** for the dataset to edit its attributes
   > *   In the central panel, click on the galaxy-chart-select-data **Datatypes** tab on the top
   > *   Select fastqsanger.gz
   > *   Click the **Change datatype** button

4. Rename the datasets and add appropriate tags to them

> For datasets that you upload via a link, Galaxy will pick the link address as the dataset name, which you will likely want to shorten to just the file names.
   > * Tip: Renaming a dataset
   > *   Click on the galaxy-pencil **pencil icon** for the dataset to edit its attributes
   > *   In the central panel, change the **Name** field
   > *   Click the **Save** button

> Large parts of the analysis in this tutorial will consist of identical steps performed on the normal and on the tumor tissue data in parallel.

> To make it easier to keep track of which dataset represents which branch of an analysis in a linear history, Galaxy supports dataset tags. In particular, if you attach a tag starting with # to any dataset, that tag will automatically propagate to any new dataset derived from the tagged dataset.

> * Tip: Adding a tag
> *   Click on the dataset
> *   Click on galaxy-tags Edit dataset tags
> *   Add a tag starting with #
> *   Tags starting with # will be automatically propagated to the outputs of tools using this dataset.
> *   Check that the tag is appearing below the dataset name\

5. Import the reference genome with the hg19 version of the sequences of human chromosomes 5, 12 and 17:

6. Rename the reference genome

> The reference genome you have imported above came as a compressed file, but got unpacked by Galaxy to plain fasta format according to your datatype selection. You may now wish to remove the .gz suffix from the dataset name.

# Quality control and mapping of NGS reads

It is imperative to ensure that the data used for analysis is of good quality prior to the commencement of the analysis. The quality control phase involves the identification and trimming of low-quality parts and/or ditching of poor quality reads that may be present in the input data so as to avoid spurious variant calls. Here, we ensure that all the sequencing reads used meet some minimal quality criteria.

> ##### More on quality control and mapping
>
> If you would like to explore the topics of quality control and read mapping in detail, you should take a look at the separate [Quality Control](https://training.galaxyproject.org/training-material/topics/sequence-analysis/tutorials/quality-control/tutorial.html) and [Mapping](https://training.galaxyproject.org/training-material/topics/sequence-analysis/tutorials/mapping/tutorial.html) tutorials. Here, we will only illustrate the concrete steps necessary for quality control and read mapping of our particular datasets.

* We shall proceed to inspect the FASTQ files by clicking on the eye icon of the galaxy platform. Ideally, each read in our file should have **four** lines.
* The first lines which begins with "**@**" symbol has a description of the read, while the second line had the actual nucleic acid sequence. 
* The third and fourth lines contained a "**+**" symbol and a string of characters representation of the quality scores associated with each base of the nucleic sequence, respectively.

## Quality control

#### Hands-on: Quality Control of the input datasets

1. Run **FastQC** on each of your four fastq datasets.

   * *"Short read data from your current history"*: all 4 FASTQ datasets selected with **Multiple datasets**

     > Tip: Select multiple datasets
     >
     > 1. Click on **Multiple datasets**
     > 2. Select several files by keeping the **Ctrl** (or **COMMAND**) key pressed and clicking on the files of interest

     8 new datasets (one with the calculated raw data, another one with an html report of the findings for each input dataset) wil get added to your history after you run this job.

2. Use **MultiQC** to aggregate the raw **FastQC** data of all four input datasets into one report

   * In *"Results"*
     * *"Which tool was used to generate logs?"*: **FastQC**
     * In *"FastQC output"*
       * *"Type of FastQC output?"*: **Raw data**
       * *"FastQC output"*: all four *RawData* outputs of **FastQC** 

3. Inspect the *Webpage* output produced by the tool

## **Read trimming and filtering**

Although the raw reads used in this tutorial are of relatively good overall quality already, we will apply read trimming and filtering to see if we can improve things still a bit, but also to demonstrate the general concept.

### **Hands-on: Read trimming and filtering of the normal tissue reads**

Run  **Trimmomatic**   to trim and filter the normal tissue reads

- _&quot;__Single-end or paired-end reads?&quot;_: `Paired-end (two separate input files)`

This makes the tool treat the forward and reverse reads simultaneously.

  > -  _"Input FASTQ file (R1/first of pair)"_: the forward reads (r1) dataset of the normal tissue sample
  > -  _"Input FASTQ file (R2/second of pair)"_: the reverse reads (r2) dataset of the normal tissue sample
  > - _"Perform initial ILLUMINACLIP step?"_: `Yes`
  > - _"Select standard adapter sequences or provide custom?";_: `Standard`
  > - _"Adapter sequences to use&quot;"_: `TruSeq3 (paired-ended, for MiSeq and HiSeq)`
  > - _"Maximum mismatch count which will still allow a full match to be performed"_: `2`
  > - _"How accurate the match between the two &#39;adapter ligated&#39; reads must be for PE palindrome read alignment";_: `30`
  > - _"How accurate the match between any adapter etc. sequence must be against a read";_: `10`
  > - _"Minimum length of adapter that needs to be detected (PE specific/ palindrome mode)";_: `8`
  > - _"Always keep both reads (PE specific/palindrome mode)?";_: `Yes`

These parameters are used to cut ILLUMINA-specific adapter sequences from the reads.

- In _"Trimmomatic Operation";_
  > - In _"1: Trimmomatic Operation";_
    > - _"Select Trimmomatic operation to perform"_: `Cut the specified number of bases from the start of the read (HEADCROP)`
    > - _"Number of bases to remove from the start of the read"_: `3`
  > -  "Insert Trimmomatic Operation"
  > - In _"2: Trimmomatic Operation"_
  > - _"Select Trimmomatic operation to perform"_: `Cut bases off the end of a read, if below a threshold quality (TRAILING)`
  >   - _"Minimum quality required to keep a base"_: `10`
  > -  "Insert Trimmomatic Operation"
  > - In "Trimmomatic Operation"_
    > - _"Select Trimmomatic operation to perform"_: `Dropreadsbelow a specified length (MINLEN)`
      > - _"Minimum quality required to keep a base"_: `25`

These three trimming and filtering operations will be applied to the reads in the given order after adapter trimming

Running this job will generate four output datasets:

- two for the trimmed forward and reverse reads that still have a proper mate in the other dataset
- two more datasets of orphaned forward and reverse reads, for which the corresponding mate got dropped because of insufficient length after trimming; when you inspect these two files, however, you should find that they are empty because none of our relatively high quality reads got trimmed that excessively. You can delete the two datasets to keep your history more compact.

### **Hands-on: Read trimming and filtering of the tumor tissue reads**

1. Trim and filter the  **tumor tissue**  reads following the same steps as above, just change the two input datasets to treat the tumor tissue reads with identical settings.
2. Check that the two unpaired reads datasets are empty, and delete them.

### **Exercise: Quality control of the polished datasets**

Use  **FastQC**   and  **MultiQC**   like before, but using the four trimmed datasets produced by Trimmomatic as input
