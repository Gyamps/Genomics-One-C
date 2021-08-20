# Variant annotation and reporting

Variant annotation is a very important step in the analysis of genome sequencing data. Functional annotation results can have a strong influence on the ultimate conclusions of disease studies. Incorrect or incomplete annotations can cause researchers both to overlook potentially disease-relevant DNA variants and to dilute interesting variants in a pool of false positives.

> Variant annotation is the process of assigning functional information to DNA variants. There are many different types of information that could be associated with variants, from measures of sequence conservation to predictions about the effect of a variant on protein structure and function 

As seen earlier, variant calling is the process by which we identify variants from sequence data (Figure 1). It involves carrying out whole genome or whole exome sequencing to create **FASTQ** files, aligning the sequences to a reference genome, creating Align the sequences to a reference genome, creating BAM or CRAM files. or **CRAM** files. identifying where the aligned reads differ from the reference genome and write to a **VCF** file.

#### Somatic versus germline variant calling

In germline variant calling, the reference genome is the standard for the species of interest. This allows us to identify genotypes. As most genomes are diploid, we expect to see that at any given locus, either all reads have the same base, indicating homozygosity, or approximately half of all reads have one base and half have another, indicating heterozygosity. An exception to this would be the sex chromosomes in male mammals.

In somatic variant calling, the reference is a related tissue from the same individual. Here, we expect to see mosaicism between cells.

#### Importing Data

For this tutorial we are going to use variant and gene annotations from many different sources. Most of these are handled for us by the tools we are going to use in this section, but we need to take care of importing the data from four sources into Galaxy separately:

- variant annotations from [Cancer Hotspots](https://www.cancerhotspots.org/)

  ```
  https://zenodo.org/record/2581873/files/hotspots.bed
  https://zenodo.org/record/2581873/files/cgi_variant_positions.bed
  https://zenodo.org/record/2581873/files/01-Feb-2019-CIVic.bed
  https://zenodo.org/record/2582555/files/dbsnp.b147.chr5_12_17.vcf.gz
  ```

- variant and gene information from the [Cancer Biomarkers database](https://www.cancergenomeinterpreter.org/biomarkers) of the Cancer Genome Interpreter (CGI) project

- variant and gene information from the [CIViC](https://civicdb.org/) database

- variant annotations from [dbSNP](https://www.ncbi.nlm.nih.gov/snp)

- lists of genes annotated with the keywords *proto-oncogene* or *tumor suppressor* at [UniProt](https://www.uniprot.org/)

  ```
  https://zenodo.org/record/2581881/files/Uniprot_Cancer_Genes.13Feb2019.txt
  https://zenodo.org/record/2581881/files/cgi_genes.txt
  https://zenodo.org/record/2581881/files/01-Feb-2019-GeneSummaries.tsv
  ```

  > Note: While importing data files check that the datatypes of all new datasets have been set correctly, and change them if necessary.

#### Adding annotations to the called variants

Certainly, not all variants are equal. Many may just be silent mutations with no effect at the amino acid level, while a few others may be disrupting the coding sequence of a protein by introducing a premature stop codon or a frameshift. Of course, it is also important to know *which* gene is affected by a variant. Such functional genomic annotations can be added to a VCF dataset of variants with ***SnpEff***.

> **Understanding VCF files**
>
> **VCF is the standard file format** for storing variation data. It is used by large scale variant mapping projects such as [IGSR](http://www.internationalgenome.org/). It is also the standard output of variant calling software such as [GATK](https://software.broadinstitute.org/gatk/) and the standard input for variant analysis tools such as the [VEP](http://www.ensembl.org/info/docs/tools/vep/index.html) or for variation archives like [EVA](https://www.ebi.ac.uk/eva/).
>
> VCF is a preferred format because it is **unambiguous, scalable and flexible**, allowing extra information to be added to the info field. Many millions of variants can be stored in a single VCF file.![fig12](C:\Users\HP\Documents\Project C\fig12.png) 
>
> An example of a VCF file.

1. Run **SnpEff eff** Tool: toolshed.g2.bx.psu.edu/repos/iuc/snpeff/snpEff/4.3+T.galaxy1 with the following parameters:

   - param-file *“Sequence changes (SNPs, MNPs, InDels)”*: the output of **VarScan somatic** tool

   - *“Input format”*: `VCF`

   - *“Output format”*: `VCF (only if input is VCF)`

   - “Genome source”: `Locally installed reference genome`

      *“Genome”*: `Homo sapiens: hg19` (or a similarly named option)

     

#### Adding genetic and clinical evidence-based annotations

Other interesting pieces of information about a variant include aspects like whether the variant has been observed before in the human population and, if so, at which frequency. If a variant is known to be associated with specific diseases, we would also very much like to know that. To proceed with this kind of genetic and clinical evidence-based annotations, we are going to convert our list of variants into a database that can be handled more efficiently than a VCF dataset. We will use the *GEMINI* tool suite for this task and for all further work with the variants.

1. Run **GEMINI load** with the following parameters:

   - *“VCF dataset to be loaded in the GEMINI database”*: the output of **SnpEff eff** tool

   - *“The variants in this input are”*: `annotated with snpEff`

   - *“This input comes with genotype calls for its samples”*: `Yes`

     Calling sample genotypes was part of what we used VarScan somatic for.

   - *“Choose a gemini annotation source”*: select the latest available annotations snapshot (most likely, there will be only one)

   - *“Sample and family information in PED format”*: `Nothing selected`

   - “Load the following optional content into the database”

     - [ ] param-check *“GERP scores”*
     - [ ] param-check *“CADD scores”*
     - [ ] param-check *“Gene tables”*
     - [ ] param-check *“Sample genotypes”*
     - [ ] param-check *“variant INFO field”*

     Be careful to leave **unchecked**:

     - *“Genotype likelihoods (sample PLs)”*

       VarScan somatic does not generate these values

     - *“only variants that passed all filters”*

       It is simple and more flexible to filter for this property later

> During the creation of the database *GEMINI* already (silently) adds an impressive amount of annotations it knows about to our variants (including, *e.g.*, the frequency at which every variant has been observed in large human genome sequencing projects).
>
> *GEMINI* also extracts a lot of the information stored in the VCF input dataset for us (such as the functional genomic annotations that we added with ***SnpEff***.
>
> Although GEMINI has blessed us with all of these features, there are additional annotations from other sources that one might like to add. *GEMINI* is not prepared to extract some non-standard information from VCF datasets, including some important bits added by *VarScan somatic*.

As a first step, we are going to use the tool to add some crucial information generated by *VarScan somatic*, but not recognized by *GEMINI load*, to the database. Specifically, we are interested in three values calculated by *VarScan somatic* for each variant it detected:

- *Somatic status (SS)*

  This is a simple numeric code, in which a value of `1` indicates a germline variant, `2` a somatic variant and `3` an LOH event.

- *Germline p-value (GPV)*

  For variants with a somatic status of 1, this is the error probability associated with that status call.

- *Somatic p-value (SPV)*

  This is the error probability associated with status calls of 2 and 3 (somatic and LOH calls).

These values are encoded in the *INFO* column of the VarScan-generated VCF dataset and we are going to extract them from there and add them to the *GEMINI* database.



#### Making variant call statistics accessible

1. Run **GEMINI annotate** Tool:  *“GEMINI database”*: output of **GEMINI load**

   - param-file *“Dataset to use as the annotation source”*: output of **VarScan somatic**

   - *“Strict variant-identity matching of database and annotation records (VCF format only)”*: `Yes`

     This setting does not really matter here since you have built the GEMINI database from the exact same list of variants that we are now retrieving annotations from and because VarScan somatic does not call multiple alleles at single sites. Matching on variant-identity is the behavior we would like to see though, so we may as well be explicit about it.

   - *“Type of information to add to the database variants”*: `Specific values extracted from matching records in the annotation source (extract)`

     - In “1: Annotation extraction recipe”:

       - *“Elements to extract from the annotation source”*: `SS`
       - *“Database column name to use for recording annotations”*: `somatic_status`
       - *“What type of data are you trying to extract?”*: `Integer numbers`
       - *“If multiple annotations are found for the same variant, store …“*: `the first annotation found`

       This is the recipe for extracting the VarScan-generated *SS* field and adding it as a new column *somatic_status* to the GEMINI database.

     - *“Insert Annotation extraction recipe”*

     - In “2: Annotation extraction recipe”:

       - *“Elements to extract from the annotation source”*: `GPV`
       - *“Database column name to use for recording annotations”*: `germline_p`
       - *“What type of data are you trying to extract?”*: `Numbers with decimal precision`
       - *“If multiple annotations are found for the same variant, store …“*: `the first annotation found`

       This is the recipe for extracting the VarScan-generated *GPV* field and adding it as a new column *germline_p* to the GEMINI database.

     -  *“Insert Annotation extraction recipe”*

     - In “3: Annotation extraction recipe”:

       - *“Elements to extract from the annotation source”*: `SPV`
       - *“Database column name to use for recording annotations”*: `somatic_p`
       - *“What type of data are you trying to extract?”*: `Numbers with decimal precision`
       - *“If multiple annotations are found for the same variant, store …“*: `the first annotation found`

       This is the recipe for extracting the VarScan-generated *SPV* field and adding it as a new column *somatic_p* to the GEMINI database.

#### Adding additional annotations

Next, we are going to add additional annotations beyond the ones directly obtainable through *GEMINI* or from the input VCF dataset. Specifically we want to add:

- more information from ***dbSNP***

  ------

  Run **GEMINI annotate** Tool: to add further annotations from dbSNP

  - *“GEMINI database”*: the output of the last **GEMINI annotate** tool run

  - *“Dataset to use as the annotation source”*: the imported `dbsnp.b147.chr5_12_17.vcf`

  - *“Strict variant-identity matching of database and annotation records (VCF format only)”*: `Yes`

    dbSNP stores information about specific SNPs observed in human populations and we would like to record if any exact same SNPs are among our variants.

  - “Type of information to add to the database variants”:

    ```plaintext
    Specific values extracted from matching records in the annotation source (extract)
    ```

    - In “1: Annotation extraction recipe”:

      - *“Elements to extract from the annotation source”*: `SAO`
      - *“Database column name to use for recording annotations”*: `rs_ss`
      - *“What type of data are you trying to extract?”*: `Integer numbers`
      - *“If multiple annotations are found for the same variant, store …“*: `the first annotation found`

      This recipe extracts the dbSNP *SAO* field and adds it as *rs_ss* to the GEMINI database.

    ------

    

  > As part of the database creation process, GEMINI already checks all variants whether they occur in dbSNP and, if so, stores their dbSNP IDs. Hence, we only need to extract some additional information of interest.

  

- information from **Cancer Hotspots**

------

1. Run **GEMINI annotate** Tool: to add further annotations from **cancerhotspots**

   -  *“GEMINI database”*: the output of the last **GEMINI annotate** tool run

   -  *“Dataset to use as the annotation source”*: the imported `cancerhotspots_v2.bed`

   - *“Strict variant-identity matching of database and annotation records (VCF format only)”*: `Yes` (with input in BED format this setting will be ignored)

     For the cancerhotspots data, we are simply going to record the best q-value associated with any cancerhotspots variant overlapping one of our variant sites.

   - “Type of information to add to the database variants”:

     ```plaintext
     Specific values extracted from matching records in the annotation source (extract)
     ```

     - In “1: Annotation extraction recipe”:

       - *“Elements to extract from the annotation source”*: `5`

         The q-values are stored in the fifth column of the BED dataset.

       - *“Database column name to use for recording annotations”*: `hs_qvalue`

       - *“What type of data are you trying to extract?”*: `Numbers with decimal precision`

       - *“If multiple annotations are found for the same variant, store …“*: `the smallest of the (numeric) values`

       This is the recipe for extracting the *q-values* of overlapping **cancerhotspots** sites and adding them as *hs_qvalue* to the GEMINI database.

------



- links to the **CIViC** database

------

1. Run **GEMINI annotate** Tool: to add links to CIViC

   - *“GEMINI database”*: the output of the last **GEMINI annotate** tool run

   - *“Dataset to use as the annotation source”*: the imported `CIViC.bed`

   - *“Strict variant-identity matching of database and annotation records (VCF format only)”*: `Yes` (with input in BED format this setting will be ignored)

     For the CIViC data, we are going to record the hyperlink to any variant found in the CIViC database that overlaps one of our variant sites.

   - “Type of information to add to the database variants”: 

     ```plaintext
     Specific values extracted from matching records in the annotation source (extract)
     ```

     - In “1: Annotation extraction recipe”:

       - *“Elements to extract from the annotation source”*: `4`

         The hyperlinks are stored in the fourth column of the BED dataset.

       - *“Database column name to use for recording annotations”*: `overlapping_civic_url`

       - *“What type of data are you trying to extract?”*: `Text (text)`

       - *“If multiple annotations are found for the same variant, store …“*: `a comma-separated list of non-redundant (text) values`

       This is the recipe for extracting the hyperlinks of overlapping CIViC sites and adding them as a list of *overlapping_civic_urls* to the GEMINI database.

   ------

   

- information from the **Cancer Genome Interpreter (CGI)**

------

1. Run **GEMINI annotate** Tool: to add information from the Cancer Genome Interpreter (CGI)

   - *“GEMINI database”*: the output of the last **GEMINI annotate** tool run

   - *“Dataset to use as the annotation source”*: the imported `cgi_variant_positions.bed`

   - *“Strict variant-identity matching of database and annotation records (VCF format only)”*: `Yes` (with input in BED format this setting will be ignored)

     For the CGI data, we are going to record if any of our variant sites is overlapped by a variant in the CGI biomarkers database.

   - “Type of information to add to the database variants”:

     ```plaintext
     Binary indicator (1=found, 0=not found) of whether the variant had any match in the annotation source (boolean)
     ```

     - *“Database column name to use for recording annotations”*: `in_cgidb`





#### Reporting selected subsets of variants

After building an enriched GEMINI database, the goal is to produce filtered variant reports that list specific classes (somatic, germline, LOH) of high-quality variants together with their most relevant annotations. 

The major ways to achieve this is through GEMINI queries that specify: 

- filters we want to apply to the variant list stored in the database

- the pieces of information about the filtered variants that we would like to retrieve



#### Querying the GEMINI database for somatic variants

Lets start by configuring a simple query to obtain a report of *bona fide* somatic variants. Our strategy for retrieving them is to:

1. rely on the *somatic status* of the variants called by **VarScan somatic** tool
2. disregard questionable variants, for which either a non-negligible amount of supporting sequencing reads is also found in the normal tissue data, or which are only supported by a very small fraction of the reads from the tumor sample

------

1. Run **GEMINI query** Tool: with:

   - *“GEMINI database”*: the fully annotated database created in the last **GEMINI annotate** tool step

   - “Build GEMINI query using”: 

     ```plaintext
     Basic variant query constructor
     ```

     - “Insert Genotype filter expression”

       - *“Restrictions to apply to genotype values”*: `gt_alt_freqs.NORMAL <= 0.05 AND gt_alt_freqs.TUMOR >= 0.10`

       With this genotype-based filter, we retain only those variants that are supported by less than 5% of the reads of the normal sample, but by more than 10% of the reads of the tumor sample.

     - *“Additional constraints expressed in SQL syntax”*: `somatic_status = 2`

       Among the info stored in the GEMINI database is the somatic status VarScan somatic has called for every variant (remember we used GEMINI annotate to add it). With the condition `somatic_status = 2` we retain only those variants passing the genotype filter above **and** considered somatic variants by the variant caller.

     - In “Output format options”

       - “Type of report to generate”: 

         ```plaintext
         tabular (GEMINI default)
         ```

         - *“Add a header of column names to the output”*: `Yes`

         - “Set of columns to include in the variant report table”:

           ```plaintext
           Custom (report user-specified columns)
           ```

           - In “Choose columns to include in the report”:
             - [x] *“chrom”*
             - [x] *“start”*
             - [x] *“ref”*
             - [x] *“alt”*
           - *“Additional columns (comma-separated)”*: `gene, aa_change, rs_ids, hs_qvalue, cosmic_ids`

       Here we specify, which columns (from the *variants* table of the GEMINI database) we want to have included, in the specified order, in a `tabular` variant report.

------

> To know the column names you can use **GEMINI database info** tool like so:
>
> - *“GEMINI database”*: the database you want to explore
> - *“Information to retrieve from the database”*: `Names of database tables and their columns`



#### More complex filter criteria

Run **GEMINI query** : with the exact same settings as before, but:

- *“Additional constraints expressed in SQL syntax”*: `somatic_status = 2 AND somatic_p <= 0.05 AND filter IS NULL`

This translates into “variants classified as somatic with a p-value <= 0.05, which haven’t been flagged as likely false-positives”.

> In the condition above, SQL keywords are given in uppercase. This is not a requirement, but it makes it easier to understand the syntax.
>
> You can check whether any cell in a data table is empty with `IS NULL`, and whether it contains *any* value with `IS NOT NULL`. To combine different filter criteria logically, you can use `AND` and `OR`, and parentheses to group conditions if required.



#### Enhancing the report format

Up until now, running this job we have 43variants, and with the annotations in the report it is relatively easy to pick out a few interesting ones.

Run **GEMINI query**: with the exact same settings as in the last example, but:

- In “Output format options”

  - *“Additional columns (comma-separated)”*: `type, gt_alt_freqs.TUMOR, gt_alt_freqs.NORMAL, ifnull(nullif(round(max_aaf_all,2),-1.0),0) AS MAF, gene, impact_so, aa_change, ifnull(round(cadd_scaled,2),'.') AS cadd_scaled, round(gerp_bp_score,2) AS gerp_bp, ifnull(round(gerp_element_pval,2),'.') AS gerp_element_pval, ifnull(round(hs_qvalue,2), '.') AS hs_qvalue, in_omim, ifnull(clinvar_sig,'.') AS clinvar_sig, ifnull(clinvar_disease_name,'.') AS clinvar_disease_name, ifnull(rs_ids,'.') AS dbsnp_ids, rs_ss, ifnull(cosmic_ids,'.') AS cosmic_ids, ifnull(overlapping_civic_url,'.') AS overlapping_civic_url, in_cgidb`

    

> ##### A little limitation
>
> The above query did not use rounding alternate allele frequencies of the sample  `gt_alt_freqs.TUMOR` and `gt_alt_freqs.NORMAL`. From the tutorial it was stated that it would break the query.



#### Generating reports of genes affected by variants

This is the final step and we are going to generate a gene-centered report based on the same somatic variants we just selected above. The gene-centered report would include annotations that apply to a whole gene affected by a variant rather than to the variant itself.

To access information from the `variants` and the `gene_detailed` table in the same query we need to join the two tables. To do this, we have to use the advanced mode for composing the query.

Run **GEMINI query** in advanced mode by choosing

- *“Build GEMINI query using”*: `Advanced query constructor`
- *“The query to be issued to the database”*: `SELECT v.gene, v.chrom, g.synonym, g.hgnc_id, g.entrez_id, g.rvis_pct, v.clinvar_gene_phenotype FROM variants v, gene_detailed g WHERE v.chrom = g.chrom AND v.gene = g.gene AND v.somatic_status = 2 AND v.somatic_p <= 0.05 AND v.filter IS NULL GROUP BY g.gene`

- *“Genotype filter expression”*: `gt_alt_freqs.NORMAL <= 0.05 AND gt_alt_freqs.TUMOR >= 0.10`

  This remains the same as in the previous somatic variants query.



#### Adding additional annotations to the gene-centered report

We added extra annotations now to the tabular gene-centered report using more general-purpose Galaxy tools. It is a two-step process in which we first *join* the report and tabular annotation sources into a larger tabular dataset, from which we then eliminate redundant and unwanted columns, while *rearranging* the remaining ones.

The **first step**  consists of three separate *join* operations that sequentially pull in the annotations found in the three gene-based tabular datasets that was imported in the *Get Data* step of this section. To to this, we used the **Join two file** tool to add the information such as **UniProt cancer genes**, **CGI biomarkers** and **CIViC**. 

------

Following these procedures:

1. Use **Join two files** to add **UniProt cancer genes** information

   - *“1st file”*: the GEMINI-generated gene report from the previous step

   - *“Column to use from 1st file”*: `Column: 1`

   - *“2nd file”*: the imported `Uniprot_Cancer_Genes` dataset

   - *“Column to use from 2nd file”*: `Column: 1`

   - *“Output lines appearing in”*: `Both 1st & 2nd file, plus unpairable lines from 1st file. (-a 1)`

     If a variant-affected gene is not listed as a Uniprot cancer gene, then, obviously, we still want to have it included in the final report.

   - *“First line is a header line”*: `Yes`

   - *“Ignore case”*: `No`

   - *“Value to put in unpaired (empty) fields”*: `0`

     If you inspect the `Uniprot_Cancer_Genes` dataset, you will see that it has two annotation columns - one indicating, using `1` and `0`, whether a given gene is a *proto-oncogene* or not, the other one indicating *tumor suppressor* genes the same way. For genes missing from this annotation dataset, we want to fill the corresponding two columns of the join result with `0` to indicate the common case that a gene affected by a variant is neither a known proto-oncogene, nor a tumor suppressor gene.

2. Use **Join two files** to add **CGI biomarkers** information

   - *“1st file”*: the partially annotated dataset from the previous
   - *“Column to use from 1st file”*: `Column: 1`
   -  *“2nd file”*: the imported `cgi_genes` dataset
   - *“Column to use from 2nd file”*: `Column: 1`
   - *“Output lines appearing in”*: `Both 1st & 2nd file, plus unpairable lines from 1st file. (-a 1)`
   - *“First line is a header line”*: `Yes`
   - *“Ignore case”*: `No`
   - *“Value to put in unpaired (empty) fields”*: `0`

   Inspect the input and the result dataset to make sure you understand what happened at this step.

3. Use **Join two files** to add gene information from **CIViC**

   - *“1st file”*: the partially annotated dataset from step 2

   - *“Column to use from 1st file”*: `Column: 1`

   - *“2nd file”*: the imported `GeneSummaries` dataset

   - *“Column to use from 2nd file”*: `Column: 3`

     The gene column in the CIViC gene summaries annotation dataset is *not* the first one!

   - *“Output lines appearing in”*: `Both 1st & 2nd file, plus unpairable lines from 1st file. (-a 1)`

   - *“First line is a header line”*: `Yes`

   - *“Ignore case”*: `No`

   - *“Value to put in unpaired (empty) fields”*: `.`

   Inspect the input and the result dataset to make sure you understand what happened at this step.

------



> Looking at the output datasets, the gene columns from each of the join operations was preserved. Also, the order in which the columns got added to the report was not right.
>
> That's why the **Step 2** is here.

In this second step we addressed all of the issues and rearranged to get a fully annotated gene report. The tool used is **Column arrange by header name** . 

------

Following the procedure:

1. Run **Column arrange by header name**  configured like this:

   - *“file to rearrange”*: the final **Join** result dataset from step 3
   - In “Specify the first few columns by name”
     - In “1: Specify the first few columns by name”
       - *“column”*: `gene`
     - *“Specify the first few columns by name”*
     - In “2: Specify the first few columns by name”
       - *“column”*: `chrom`
     - *“Specify the first few columns by name”*
     - In “3: Specify the first few columns by name”
       - *“column”*: `synonym`
     - *“Specify the first few columns by name”*
     - In “4: Specify the first few columns by name”
       - *“column”*: `hgnc_id`
     - *“Specify the first few columns by name”*
     - In “5: Specify the first few columns by name”
       - *“column”*: `entrez_id`
     - *“Specify the first few columns by name”*
     - In “6: Specify the first few columns by name”
       - *“column”*: `rvis_pct`
     - *“Specify the first few columns by name”*
     - In “7: Specify the first few columns by name”
       - *“column”*: `is_OG`
     - *“Specify the first few columns by name”*
     - In “8: Specify the first few columns by name”
       - *“column”*: `is_TS`
     - *“Specify the first few columns by name”*
     - In “9: Specify the first few columns by name”
       - *“column”*: `in_cgi_biomarkers`
     - *“Specify the first few columns by name”*
     - In “10: Specify the first few columns by name”
       - *“column”*: `clinvar_gene_phenotype`
     - *“Specify the first few columns by name”*
     - In “11: Specify the first few columns by name”
       - *“column”*: `gene_civic_url`
     - *“Specify the first few columns by name”*
     - In “12: Specify the first few columns by name”
       - *“column”*: `description`
   - *“Discard unspecified columns”*: `Yes`

   ------

   