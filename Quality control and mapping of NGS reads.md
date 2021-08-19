# Quality control and mapping of NGS reads

Prior to the analysis, it is good to be certain that the input data is of good quality, i.e., there wasn’t any major issues during DNA preparation, exon capture, or during actual sequencing. To avoid false variant calls due to low input quality, we can ensure that all sequencing reads used in the analysis meet some minimal quality criteria by trimming low-quality parts of the ends of reads and/or discarding reads of poor quality altogether. The resulting set of polished reads then needs to be mapped to the human reference genome because knowing the genomic positions that the bases of a read provide evidence for is, of course, a prerequisite for variant calling.

### Quality Control 

**Hands-on: Quality control of the input datasets**

1. Run **FastQC** Tool: on each of your four fastq datasets

   - “Short read data from your current history”: all 4 FASTQ datasets selected with **Multiple datasets** 

   **N.B** *When you start this job, eight new datasets (one with the calculated raw data, another one with an html report of the findings for each input dataset) will get added to your history.*

2.  Use **MultiQC** Tool: to aggregate the raw **FastQC** data of all four input datasets into one report 

   - In "*Results*" 
     - *“Which tool was used generate logs?”*: `FastQC`
     - In *“FastQC output”*
       - *“Type of FastQC output?”*: `Raw data`
       - *“FastQC output”*: all four *RawData* outputs of **FastQC**

3. Inspect the *Webpage* output produced by the tool

### Read trimming and filtering

Although the raw reads used in this tutorial are of relatively good overall quality already, we will apply read trimming and filtering to see if we can improve things still a bit, but also to demonstrate the general concept.

**Hands-on: Quality control of the input datasets**

1. Run **Trimmomatic** Tool: to trim and filter the normal tissue reads

   - *“Single-end or paired-end reads?”*: `Paired-end (two separate input files)`

     This makes the tool treat the forward and reverse reads simultaneously.

     - *“Input FASTQ file (R1/first of pair)”*: the forward reads (r1) dataset of the normal tissue sample

     - *“Input FASTQ file (R2/second of pair)”*: the reverse reads (r2) dataset of the normal tissue sample

   - *“Perform initial ILLUMINACLIP step?”*: `Yes`

     - *Select standard adapter sequences or provide custom?”*: `Standard`

       - "*Adapter sequences to use”*: `TruSeq3 (paired-ended, for MiSeq and HiSeq)`

     - *“Maximum mismatch count which will still allow a full match to be  performed”*: `2`

     - *“How accurate the match between the two ‘adapter ligated’ reads must  be for PE palindrome read alignment”*: `30`

     - *“How accurate the match between any adapter etc. sequence must be  against a read”*: `10`

     - *“Minimum length of adapter that needs to be detected (PE specific/  palindrome mode)”*: `8`

       *“Always keep both reads (PE specific/palindrome mode)?”*: `Yes`

   These parameters are used to cut ILLUMINA-specific adapter sequences from the reads.

   - In *“Trimmomatic Operation”*
     - In *“1: Trimmomatic Operation”*
       - *“Select Trimmomatic operation to perform”*: `Cut the specified number of bases from the start of the read (HEADCROP)`
         - *“Number of bases to remove from the start of the read”*: `3`
     - “Insert Trimmomatic Operation”*
     - In *“2: Trimmomatic Operation”*
       - *“Select Trimmomatic operation to perform”*: `Cut bases off the end of a read, if below a threshold quality (TRAILING)`
         - *“Minimum quality required to keep a base”*: `10`
     -  “Insert Trimmomatic Operation”*
     - In *“3: Trimmomatic Operation”*
       - *“Select Trimmomatic operation to perform”*: `Drop reads below a specified length (MINLEN)`
         - *“Minimum quality required to keep a base”*: `25`

   These three trimming and filtering operations will be applied to the reads in the given order after adapter trimming.

   **N.B** **Running this job will generate four output datasets:**
   
   - *two for the trimmed forward and reverse reads that still have a proper mate in the other dataset*.
   
   -  *two more datasets of orphaned forward and reverse reads, for which the corresponding mate got dropped because of insufficient length after trimming; when you inspect these two files, however, you should find that they are empty because none of our relatively high quality reads got trimmed that excessively. You can delete the two datasets to keep your history more compact*.

### Read Mapping

**Hands-on:** **Read Mapping**

1. Use **Map with BWA-MEM** Tool: to map the reads from the **normal tissue** sample to the reference genome      

   - *“Will you select a reference genome from your history or use a built-in index?”*: **`YES`** `Use a built-in genome index`

     - *“Using reference genome”*: `Human: hg19` (or a similarly named option)

       ### Using the imported `hg19` sequence

       If you have imported the `hg19` sequence as a fasta dataset into your history instead:

       - *“Will you select a reference genome from your history or use a built-in index?”*: `Use a genome from history and build index`
         - *“Use the following dataset as the reference sequence”*: your imported `hg19` fasta dataset.

   - *“Single or Paired-end reads”*: `Paired`

     - *“Select first set of reads”*: the trimmed  forward reads (r1) dataset of the **normal tissue** sample; output of **Trimmomatic**
     - *“Select second set of reads”*: the trimmed reverse reads (r2) dataset of the **normal tissue** sample; output of **Trimmomatic**

   - *“Set read groups information?”*: `Set read groups (SAM/BAM specification)`

     - *“Auto-assign”*: `No`
       - *“Read group identifier (ID)”*: `231335` (this value being taken from the original name of the normal tissue input files)
     - *“Auto-assign”*: `No`
       - *“Read group sample name (SM)”*: `Normal`

   **Tips** *In general, you are free to choose ID and SM values to your liking, but the ID should unambiguously identify the sequencing run that produced the reads, while the SM value should identify the biological sample.*

2. Use **Map with BWA-MEM** Tool: toolshed.g2.bx.psu.edu/repos/devteam/bwa/bwa_mem/0.7.17.1  to map the reads from the **tumor tissue** sample,      

   - *“Will you select a reference genome from your history or use a built-in index?”*: `Use a built-in genome index`

     - *“Using reference genome”*: `Human: hg19` (or a similarly named option)

     Adjust these settings as before if you are using the imported reference genome.

   - *“Single or Paired-end reads”*: `Paired`

     - *“Select first set of reads”*: the trimmed forward reads (r1) dataset of the **tumor tissue** sample; output of **Trimmomatic**
     - *Select second set of reads”*: the reverse reads (r2) dataset of the **tumor tissue** sample; output of **Trimmomatic**

   - *“Set read groups information?”*: `Set read groups (SAM/BAM specification)`

     - *“Auto-assign”*: `No`
       - *Read group identifier (ID)”*: `231336` (this value, again, being taken from the original name of the tumor tissue input files)
     - *“Auto-assign”*: `No`
       - *“Read group sample name (SM)”*: `Tumor`



# Mapped reads postprocessing

To ensure that we base our variant analysis only on unambiguous, high-quality read mappings we will do some postprocessing next.

### Filtering on mapped reads properties

To produce new filtered BAM datasets with only those reads retained that have been mapped to the reference successfully, have a minimal mapping quality of 1, and for which the mate read has also been mapped:

**Hands-on:** **Filtering for mapping status and quality**

1. Run **Filter BAM datasets on a variety of attributes** with the following parameters:

   -  *“BAM dataset(s) to filter”*: mapped reads datasets from the normal *and* the tumor tissue data, outputs of **Map with BWA-MEM** 

   - In *“Condition”*:

   - In *“1: Condition”*:

     - In *“Filter”*:

       - In *“1: Filter”*: 

         - *“Select BAM property to filter on”*: `mapQuality` 

           - ''*Filter on read mapping quality (phred scale)”*: `>=1`
- In *“2: Filter”*:
         - *“Select BAM property to filter on”*: `isMapped` 
    - *“Selected mapped reads”*: `Yes`
       - Click on *“Insert Filter”*
- In *“3: Filter”*:                  
         - *“Select BAM property to filter on”*: `isMateMapped`
    - *“Select reads with mapped mate”*: `Yes`

When you configure multiple filters within one condition, reads have to pass *all* the filters to be retained in the output. The above settings, thus, retain only read pairs, for which both mates are mapped.

Note that filtering for a minimal mapping quality is not strictly necessary. Most variant callers (including **VarScan somatic**, which we will be using later) have an option for using only reads above a certain mapping quality. In this section, however, we are going to process the retained reads further rather extensively so it pays off in terms of performance to eliminate reads we do not plan to use at an early step.

*“Would you like to set rules?”*: `No`

This will result in two new datasets, one for each of the normal and tumor data.

###### **Tools for filtering BAM datasets** 

Under the hood, **Filter SAM or BAM, output SAM or BAM** tool uses the `samtools view` command line tool, while **Filter BAM datasets on a variety of attributes** tool uses `bamtools filter`. Whatever the BAM filtering task, you should be able to perform it in Galaxy with one of these two tools.

### Removing duplicate reads

**Hands-on: Remove duplicates**

1. Run **RmDup** Tool: with the following parameters:
   - *“BAM file”*: filtered reads datasets from the normal *and* the tumor tissue data; the outputs of **Filter SAM or BAM**
   - *“Is this paired-end or single end data”*: `BAM is paired-end`
   - *“Treat as single-end”*: `No`

Again, this will produce two new datasets, one for each of the normal and tumor data.

### Left-align reads around indels

**Hands-on: Left-align**

1. Run **BamLeftAlign** Tool: with the following parameters:
   - *“Choose the source for the reference genome”*: `Locally cached` 
     - *“BAM dataset to re-align”*: your filtered and deduplicated reads datasets from the normal *and* the tumor tissue data; the outputs of **RmDup**    
     - *“Using reference genome”*: `Human: hg19` (or a similarly named choice)
   - *“Maximum number of iterations”*: `5`

**Using the imported `hg19` sequence**

If you have imported the `hg19` sequence as a fasta dataset into your history instead:

- *“Choose the source for the reference genome”*: `History`
  - *“Using reference file”*: your imported `hg19` fasta dataset

### Recalibrate read mapping qualities

**Hands-on: Recalibrate read quality scores**

1. Run **CalMD** Tool:with the following parameters:

   - *“BAM file to recalculate”*: the left-aligned datasets from the normal and the tumor tissue data; the outputs of **BamLeftAlign**

   - *“Choose the source for the reference genome”*: `Use a built-in genome`

     -  *“Using reference genome”*: `Human: hg19` (or a similarly named choice)

       **Using the imported `hg19` sequence    **

       If you have imported the `hg19` sequence as a fasta dataset into your history instead:

       - *“Choose the source for the reference genome”*: `Use a genome from the history`
         - *“Using reference file”*: your imported `hg19` fasta dataset.

   - *“Do you also want BAQ (Base Alignment Quality) scores to be calculated?”*: `No`

     The *VarScan somatic* tool that we are going to use for calling variants at the next step is typically used in combination with unadjusted base quality scores because the general opinion is that the base quality downgrades performed by *CalMD* and other tools from the *samtools* suite of tools are too severe for *VarScan* to retain good sensitivity. We are sticking with this practice in this tutorial.

     **Using adjusted base quality scores**

     If, for your own data, you would like to experiment with adjusted base quality scores, it is important to understand that *VarScan somatic* will only make use of the adjusted scores if they are incorporated directly into the read base qualities of a BAM input dataset, but not if they are written to the dataset separately.

     Hence, should you ever decide to use:

     - *“Do you also want BAQ (Base Alignment Quality) scores to be calculated?”*: `Yes, run BAQ calculation` 

       and you want this setting to affect downstream variant calling with *VarScan somatic*, make sure you also set then:

       - *“Use BAQ to cap read base qualities”*: `Yes

     Please also note that BAQ scores are quite expensive to calculate so be prepared to see a substantial (up to 10x!) increase in job run time  when enabling it.

   - *“Additional options”*: `Advanced options`

     - *“Change identical bases to ‘=’“*: `No`

     - *“Coefficient to cap mapping quality of poorly mapped reads”*: `50`

       This last setting is the real reason why we use CalMD at this point. It is an empirical, but well-established finding that the mapping quality of reads mapped with *bwa* should be capped this way before variant calling.

This will, once more, produce two new datasets, one for each of the normal and tumor data.

### Refilter reads based on mapping quality

During recalibration of read mapping qualities **CalMD** may have set some mapping quality scores to 255. This special value is reserved for *undefined* mapping qualities and is used by the tool when a recalibrated mapping quality would drop below zero. In other words, a value of 255 does not indicate a particularly good mapping score, but a really poor one. To remove such reads from the data:

**Hands-on: Eliminating reads with undefined mapping quality**

1. Run **Filter BAM datasets on a variety of attributes** Tool: with the following parameters:

   - *“BAM dataset(s) to filter”*: the recalibrated datasets from the normal and the tumor tissue data; the outputs of **CalMD**

   - In *“Condition”*:

   - In *“1: Condition”*:

     - In *“Filter”*:

       - *“Select BAM property to filter on”*: `mapQuality`

         - *“Filter on read mapping quality (phred scale)”*: `<=254` 

         ​                        









​                                  

