
# Quality Control

It is imperative to ensure that the data used for analysis is of good quality prior to the commencement of the analysis. The quality control phase invoves the identification and exclusion of various errors that may e present in the sample nucleic acid sequences. 

##  Hands-on: Quality Control

1. ### Inspect the raw sequence files;
* We shall proceed to inspect the FASTQ files by clicking on the eye icon of the galaxy platform. Ideally, each read in our file should have **four** lines.
*  The first lines which begins with "**@**" symbol has a description of the read, while the second line had the actual nucleic acid sequence. 
*  The third and fourth lines contained a "**+**" symbol and a string of characters representation of the quality scores associated with each base of the nucleic sequence, respectively.

2. ### After the manual inspection of sample sequences, run [FastQC Tool](toolshed.g2.bx.psu.edu/repos/devteam/fastqc/fastqc/0.72+galaxy1) on each of the fastq datasets
* _param-files “Short read data from your current history”_: all 4 FASTQ datasets selected with **Multiple datasets**
* This step will result in eight new datasets-hich will now be available at the history section of the galaxy platform.

3. ### Use [**MultiQC Tool**](toolshed.g2.bx.psu.edu/repos/iuc/multiqc/multiqc/1.8+galaxy0) to collate the raw raw **FastQC **data of all four input datasets into one report
   * In “Results”
        ** “_Which tool was used generate logs_?”: FastQC
         ** In “FastQC output”
                *** “Type of _FastQC output_?”: Raw data
                *** param-files “_FastQC output_”: all four RawData outputs of **FastQC** tool)

3. ### Inspect the _Webpage_ output produced by the tool
