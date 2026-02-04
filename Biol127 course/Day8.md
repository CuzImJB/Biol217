# Day 8 Workflow

## Example 1 --> For an Overview, not done by us
### set proxy environment to download the data and use the internet in the backend
```
#export http_proxy=http://relay:3128
#export https_proxy=http://relay:3128
#export ftp_proxy=http://relay:3128
```

### create folders
```
reademption create --project_path READemption_analysis_1 --species salmonella="Salmonella Typhimurium"
```

### download reference genome sequence files (genome and 3 plasmids)
```
FTP_SOURCE=ftp://ftp.ncbi.nih.gov/genomes/archive/old_refseq/Bacteria/Salmonella_enterica_serovar_Typhimurium_SL1344_uid86645/
wget -O READemption_analysis/input/salmonella_reference_sequences/NC_016810.fa $FTP_SOURCE/NC_016810.fna
wget -O READemption_analysis/input/salmonella_reference_sequences/NC_017718.fa $FTP_SOURCE/NC_017718.fna
wget -O READemption_analysis/input/salmonella_reference_sequences/NC_017719.fa $FTP_SOURCE/NC_017719.fna
wget -O READemption_analysis/input/salmonella_reference_sequences/NC_017720.fa $FTP_SOURCE/NC_017720.fna
```

### rename the files similar to the genome naming
```
sed -i "s/>/>NC_016810.1 /" READemption_analysis_1/input/salmonella_reference_sequences/NC_016810.fa
sed -i "s/>/>NC_017718.1 /" READemption_analysis_1/input/salmonella_reference_sequences/NC_017718.fa
sed -i "s/>/>NC_017719.1 /" READemption_analysis_1/input/salmonella_reference_sequences/NC_017719.fa
sed -i "s/>/>NC_017720.1 /" READemption_analysis_1/input/salmonella_reference_sequences/NC_017720.fa
```

### download gene annotations and unzip it
```
wget -O READemption_analysis_1/input/salmonella_annotations/GCF_000210855.2_ASM21085v2_genomic.gff.gz https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/210/855/GCF_000210855.2_ASM21085v2/GCF_000210855.2_ASM21085v2_genomic.gff.gz
gunzip READemption_analysis_1/input/salmonella_annotations/GCF_000210855.2_ASM21085v2_genomic.gff.gz
```
### download RNA-seq reads
```
wget -O READemption_analysis_1/input/reads/InSPI2_R1.fa.bz2 http://reademptiondata.imib-zinf.net/InSPI2_R1.fa.bz2
wget -O READemption_analysis_1/input/reads/InSPI2_R2.fa.bz2 http://reademptiondata.imib-zinf.net/InSPI2_R2.fa.bz2
wget -O READemption_analysis_1/input/reads/LSP_R1.fa.bz2 http://reademptiondata.imib-zinf.net/LSP_R1.fa.bz2
wget -O READemption_analysis_1/input/reads/LSP_R2.fa.bz2 http://reademptiondata.imib-zinf.net/LSP_R2.fa.bz2
```

### align reads to reference
```
reademption align -p 4 --poly_a_clipping --project_path READemption_analysis_1
```

### calculate read coverage
```
reademption coverage -p 4 --project_path READemption_analysis_1
```

### quantify gene expression
```
reademption gene_quanti -p 4 --features CDS,tRNA,rRNA --project_path READemption_analysis_1
```

### calculate differential expression using DESeq2
```
reademption deseq -l InSPI2_R1.fa.bz2,InSPI2_R2.fa.bz2,LSP_R1.fa.bz2,LSP_R2.fa.bz2 -c InSPI2,InSPI2,LSP,LSP -r 1,2,1,2 --libs_by_species salmonella=InSPI2_R1,InSPI2_R2,LSP_R1,LSP_R2 --project_path READemption_analysis_1
```

### visualization
```
reademption viz_align --project_path READemption_analysis_1
reademption viz_gene_quanti --project_path READemption_analysis_1
reademption viz_deseq --project_path READemption_analysis_1
```

## Example 2
In this example you will run the anaylsis yourself

### Download the sequence data you want to analyze
For downloading the data we will use```grabseqs```, a tool that allows downloading of sequence data from various repositories including SRA.

#### use micromamba to activate grabseq
```
module load micromamba/1.4.2
eval "$(micromamba shell hook --shell=bash)"
micromamba activate $WORK/.micromamba/envs/10_grabseqs
mkdir fastq_raw
cd fastq_raw
```
```
grabseqs sra -t 4 -m ./metadata.csv SRR4018514
grabseqs sra -t 4 -m ./metadata.csv SRR4018515
grabseqs sra -t 4 -m ./metadata.csv SRR4018516
grabseqs sra -t 4 -m ./metadata.csv SRR4018517
```
Note: Rename each SRR file according to the sample name. For example, SRR4018514 to wt_R1.fastq.gz, SRR4018515 to wt_R2.fastq.gz, SRR4018516 to mut_R1.fastq.gz, and SRR4018517 to mut_R2.fastq.gz.

### Create the redemption folder structure
```
reademption create --project_path READemption_analysis_2 --species methanosarcina="Methanosarcina mazei Gö1"
```

### Download the reference genome and annotation files and move them to the input folder

#### download reference genome
```
wget -O READemption_analysis_2/input/methanosarcina_reference_sequences/GCF_000007065.1_ASM706v1_genomic.fna.gz https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/065/GCF_000007065.1_ASM706v1/GCF_000007065.1_ASM706v1_genomic.fna.gz
```

#### download annotation
```
wget -O READemption_analysis_2/input/methanosarcina_annotations/GCF_000007065.1_ASM706v1_genomic.gff.gz https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/065/GCF_000007065.1_ASM706v1/GCF_000007065.1_ASM706v1_genomic.gff.gz
```

#### unzip them
```
gunzip READemption_analysis_2/input/methanosarcina_reference_sequences/GCF_000007065.1_ASM706v1_genomic.fna.gz
gunzip READemption_analysis_2/input/methanosarcina_annotations/GCF_000007065.1_ASM706v1_genomic.gff.gz
```

Copy the raw_reads to the READemption_analysis/input/reads folder

### Run READemption

#### align reads to reference
```
reademption align -p 4 --poly_a_clipping --project_path READemption_analysis_2 --fastq
```

#### calculate read coverage
```
reademption coverage -p 4 --project_path READemption_analysis_2
```

#### quantify gene expression
```
reademption gene_quanti -p 4 --features CDS,tRNA,rRNA --project_path READemption_analysis_2
```

#### calculate differential expression using DESeq2
```
reademption deseq -l mut_R1.fastq.gz,mut_R2.fastq.gz,wt_R1.fastq.gz,wt_R2.fastq.gz -c mut,mut,wt,wt -r 1,2,1,2 --libs_by_species methanosarcina=mut_R1,mut_R2,wt_R1,wt_R2 --project_path READemption_analysis_2
```

#### visualization
```
reademption viz_align --project_path READemption_analysis_2
reademption viz_gene_quanti --project_path READemption_analysis_2
reademption viz_deseq --project_path READemption_analysis_2
```

### Analyze the results
* ```all_species_viz_align``` 
    * [All Species viz align](../Images/all_species_viz_align.png)
    * The stacked species box plot shows how many reads mapped to which species (or didn't map at all). In this case, most of the reads were aligned to Methanosarcina mazei Gö1 and a few could not be aligned to anything. Cross aligned reads are reads that are aligned to multiple genomes, which is not possible in our case.
* ```methanosarcina_viz_align```
    * [Methanosarcina viz align](../Images/methanosarcina_viz_align.png)
    * The aligned reads box plot shows the number of aligned reads and how they aligned. Uniquely aligned reads were mapped to only one location in the genome, while multiple aligned reads mapped to several locations. In this case, most reads aligned uniquely, only a few aligned to multiple locations. Split aligned reads are reads that span across multiple regions in the genome, which can happen in eukaryotic genomes with introns, but is less common in prokaryotes. Cross aligned reads align to multiple reference genomes, which is not relevant since we only have one genome. 
* ```methanosarcina_viz_deseq```
    * [MA Plot](../Images/methanosarcina_viz_deseq_MA_plot.png)
    * The MA plots visualize the differential expression analysis. Each point represents a gene, with the x-axis showing the average expression level and the y-axis showing the log2 fold change between conditions. Genes that are significantly differentltially expressed are shown as red dots, non-significant as black dots. A log2 fold change of above 0 means that the gene is upregulated in the first conditions compared to the second and vice-versa. In general, we see a mostly symmetric distribution around the log2 fold-change = 0, with some genes showing significant up- or down-regulation.
    * [volcano Plot](../Images/Methanosarcina_viz_deseq_volcano_plot.png)
    * The volcano plots also visualize the differential expression analysis. There are plots for both raw p-values and adjusted p-values. The adjusted p-value is more conservative than the raw p-value and therefore showns less false positives. In both cases the plot shows the log2 fold-changes on the x-axis, positive values indicate upregulation in the first condition compared to the second and vice-versa. The y-axis shows the -log10 of the p-value, meaning that the higher up a point is, the lower is its p-value and the more significant it is. The green dotted lines indicate the thresholds for significance (p-value < 0.05) and log2 fold-change (> 1 or <-1). 
* ```methanosarciana_viz_gene_quanti```
    * [Expression Scatter Plot 1-4](../Images/methanosarcina_viz_gene_quanti_expressionscatterplots_1-4.png)
    * [Expression Scatter Plot 5-8](../Images/methanosarcina_viz_gene_quanti_expressionscatterplots_5-8.png)
    * [Expression Scatter Plot 9-12](../Images/Methanosarcina_viz_gene_quanti_expressionscatterplots_9-12.png)
    * The expression scatter plots compare the expression of each gene in different conditions. Generally, we expect most genes to express similarly in all conditions. In that case most points should fall along the diagonal line and the R-value should be close to 1. This is true for our example, most points fall along the diagonal line, but a few here and there a further away from the line, suggesting different expression under different conditions. The values are close to 1.
    * [RNA Class sizes](../Images/Methanosarcina_viz_gene_quanti_rnaclasssize.png)
    * The RNA class box plots show the number of read counts for the different RNA classes (CDS, TRNA and rRNA) in each sample. This can help identify any biases in the data, for example if one RNA class is overpresented in a sample. In our analysis, CDS has the most counts and tRNA and rRNA have fewer, but it doesnt look overrepresented in a sample, since this spans over all samples.
* ```read_lengths_viz_align```
    * [Input read lenght distribution mut](../Images/read_lenghts_viz_align_input_mut.png)
    * [Input read lenght distribution wt](../Images/read_lenght_viz_align_input_wt.png)
    * [Processed read lenght distribution mut](../Images/read_lenght_viz_align_processed_mut.png)
    * [Processed read lenght distribution wt](../Images/read_lenght_viz_align_processed_wt.png)
    * The read lenght distribution plots compare the differences in read lenght before and after trimming and between samples. The read lenght is determined by the sequencing technology used. In our example the input reads all had a uniform lenght of 100 nt and after trimming the reads range from ca.10 to 100 nt. Only a few reads have a lenght of 10, while most of the reads have higher lenghts over 70 with most of them still having a lenght of 100 nt.


### What now?
The most important files for further analysis lie in the methanosarcina_deseq folder. Here you can find the tables that show the results of the differential expression analysis (log2 FC, p-values etc.), for both the raw data (deseq_raw) and with annotations (deseq_with_annotations). The annotations are crucial, because without them we don't know which genes are actually differentially expressed.

Think about what you could do with these results: 
<br> You can use the DESeq results to identify significantly up- and down-regulated genes and, with the annotated tables, interpret these changes biologically. The annotations allow grouping DE genes into functional categories and pathways, enabling enrichment and pathway analyses to reveal which cellular processes (e.g. metabolism, regulation, stress response) are affected. Overall, this turns statistical output (log2FC, p-values) into biologically meaningful insights about how Methanosarcina responds to the tested condition.