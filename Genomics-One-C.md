[toc]



# Variant Calling and Classification

Now, having generated a high-quality set of mapped read pairs, we can proceed to variant calling. On the galaxy server we employ the use of the **VarScan somatic** tool, which is a dedicated solution for somatic variant calling.

### Hands-on: Variant Calling and Classification

1. Run **VarScan somatic** with the following parameters:

   * *"Will you selct a reference genome from your history or use a built-in genome?"* :  <span style='color:red'>==Use a built-in genome==</span> 

     * *"reference genome"*: <span style='color:red'>==Human: hg19==</span> (or a similarly named choice)

     #### Using the imported <span style='color:red'>==hg19==</span> sequence

     In the event you have imported the <span style='color:red'>==hg19==</span> sequence as a fasta dataset into your history, you could instead:

     * *"Will you select a reference genome from your history or use a built-in genome?"*: <span style='color:red'>==Use a genome from the history==</span> 
       * *"reference genome"*: your imported <span style='color:red'>==hg19==</span> fasta dataset.

   * *"aligned reads from normal sample"*: the mapped and fully post-processed normal tissue dataset; one of the two outputs of filtering the **CalMD** outputs

   * *"aligned reads from tumor sample"*: the mapped and fully post-processed tumor tissue dataset; the other output of filtering the **CalMD** outputs.

   * *"Estimated purity (non-tumor content) of normal sample"*: <span style='color:red'>1</span> 

   * *"Estimated purity (tumor content) of tumor sample"*: <span style='color:red'>0.5</span> 

   * *"Generate separate output datasets for SNP and indel calls?"*: <span style='color:red'>No</span> 

   * *"Settings fro Variant Calling"*: <span style='color:red'>Customize settings</span> 

     * *"Minimum base quality"*: <span style='color:red'>28</span> 

       We have seen, at the quality control step, that our sequencing data is of really good quality, and we have chosen not to downgrade base qualities at the quality scores recalibration step above, so we can increase the base quality required at any given position without throwing away too much of our data.

     * *"Minimum mapping quality"*: <span style='color:red'>1</span> 

       During postpreocessing, we have filtered our reads for ones with a mapping quality of at least one, but **CalMD** may have lowered some mapping qualities to zero afterwards.

     Leave all other settings in this section at their default values.

   * *"Settings for Posterior Variant Filtering"*: <span style='color:red'>Use default values</span> 