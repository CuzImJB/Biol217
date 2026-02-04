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
    *
* ```methanosarcina_viz_align```
    *
* ```methanosarcina_viz_deseq```
    *
* ```methanosarciana_viz_gene_quanti```
    *
* ```read_lengths_viz_align```
    *

### What now?
The most important files for further analysis lie in the methanosarcina_deseq folder. Here you can find the tables that show the results of the differential expression analysis (log2 FC, p-values etc.), for both the raw data (deseq_raw) and with annotations (deseq_with_annotations). The annotations are crucial, because without them we don't know which genes are actually differentially expressed.

Think about what you could do with these results: