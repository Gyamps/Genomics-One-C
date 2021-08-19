[toc]


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

# Variant Calling and Classification

Now, having generated a high-quality set of mapped read pairs, we can proceed to variant calling. On the galaxy server we employ the use of the **VarScan somatic** tool, which is a dedicated solution for somatic variant calling.

### Hands-on: Variant Calling and Classification

1. Run **VarScan somatic** with the following parameters:

   * *"Will you select a reference genome from your history or use a built-in genome?"* :  <span style='color:red'><mark>**Use a built-in genome**</mark></span> 

     * *"reference genome"*: <span style='color:red'><mark>**Human: hg19**</mark></span> (or a similarly named choice)

     
   
     > ### Using the imported <span style='color:red'><mark>**hg19**</mark></span> sequence
     >
     > In the event you have imported the <span style='color:red'><mark>**hg19**</mark></span> sequence as a fasta dataset into your history, you could instead:
     >
     > * *"Will you select a reference genome from your history or use a built-in genome?"*: <span style='color:red'><mark>**Use a genome from the history**</mark></span> 
     >
     > * *"reference genome"*: your imported <span style='color:red'><mark>**hg19**</mark></span> fasta dataset.

   * *"aligned reads from normal sample"*: the mapped and fully post-processed normal tissue dataset; one of the two outputs of filtering the **CalMD** outputs

   * *"aligned reads from tumor sample"*: the mapped and fully post-processed tumor tissue dataset; the other output of filtering the **CalMD** outputs.

   * *"Estimated purity (non-tumor content) of normal sample"*: <span style='color:red'>**1**</span> 

   * *"Estimated purity (tumor content) of tumor sample"*: <span style='color:red'>**0.5**</span> 

   * *"Generate separate output datasets for SNP and indel calls?"*: <span style='color:red'>**No**</span> 

   * *"Settings for Variant Calling"*: <span style='color:red'>**Customize settings**</span> 

     * *"Minimum base quality"*: <span style='color:red'>**28**</span> 

       We have seen, at the quality control step, that our sequencing data is of really good quality, and we have chosen not to downgrade base qualities at the quality scores recalibration step above, so we can increase the base quality required at any given position without throwing away too much of our data.

     * *"Minimum mapping quality"*: <span style='color:red'>**1**</span> 

       During post-processing, we have filtered our reads for ones with a mapping quality of at least one, but **CalMD** may have lowered some mapping qualities to zero afterwards.

     Leave all other settings in this section at their default values.
   
   * *"Settings for Posterior Variant Filtering"*: <span style='color:red'>**Use default values**</span> 

# Conclusion

In addition to merely calling variants, *somatic variant calling* tries to distinguish *somatic mutations*, which are private to tumor tissue, from *germline* mutations, that are shared by tumor and healthy tissue, and *loss-of-heterozygosity* events, which involve the loss, from tumor tissue, of one of two alleles found at a biallelic site in healthy tissue.

Dedicated somatic variant callers can perform this classification on statistical grounds, but the interpretation of any list of variants (somatic, germline or LOH) also depends crucially on rich genetic and cancer-specific variant and gene annotations.

Some **Key points** to consider:

* Follow best practices for read mapping, quality control and mapped reads post-processing to minimize false-positive variant calls.
* Use a dedicated somatic variant caller to call variants and to classify them into somatic, germline and LOH event variants on statistical grounds.
* Annotations and queries based on variant properties add relevance to variant and gene reports.
* A framework like **GEMINI** is very helpful for managing, annotating and querying lists of variants in a flexible way.
* Prefer public, free annotation sources to foster reproducibility and information sharing.