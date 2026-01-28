# Day 2 Workflow

## Quality Control of raw reads

Before proceeding with the analysis of the raw reads, you need to assess the quality of the reads. Further, the raw reads are processed and filtered. This is done with two commands: ```fastqc``` and ```fastp```

### fastqc
Fastqc is used to evaluate the quality of the raw reads. It provides basic quality control metrics like Phred scores, which gives you an idea of how accurate each base call was.
<br> ```fastqc 0_raw_reads/*.fastq.gz -o fastqc``` 
* ```0_raw_reads/*.fastq.gz``` --> all fastq.gz files in the directory 0_raw_reads are evaluated
* ```-o fastqc``` --> output folder

### fastp
Fastp is used to process and filter the reads. As we have paired-end reads (forward and backward primer), we need to specify two different input files: R1 (forward) and R2 (backward)
<br> ```fastp -i 0_raw_reads/BGR_130305_mapped_R1.fastq.gz -I 0_raw_reads/BGR_130305_mapped_R2.fastq.gz -o fastp/BGR_130305_mapped_R1.fastq.gz -O fastp/BGR_130305_mapped_R2.fastq.gz -t 6 -q 20 -h "fastp_BGR_130305.html" -R fastp/"fastp_BGR_130305"```
<br>```fastp -i 0_raw_reads/BGR_130527_mapped_R1.fastq.gz -I 0_raw_reads/BGR_130527_mapped_R2.fastq.gz -o fastp/BGR_130527_mapped_R1.fastq.gz -O fastp/BGR_130527_mapped_R2.fastq.gz -t 6 -q 20 -h "fastp_BGR_130527.html" -R fastp/"fastp_BGR_130527"```
<br>```fastp -i 0_raw_reads/BGR_130708_mapped_R1.fastq.gz -I 0_raw_reads/BGR_130708_mapped_R2.fastq.gz -o fastp/BGR_130708_mapped_R1.fastq.gz -O fastp/BGR_130708_mapped_R2.fastq.gz -t 6 -q 20 -h "fastp_BGR_130708.html" -R fastp/"fastp_BGR_130708"```
<br> this step must be done for each pair of reads (*_R1_fastq.gz would not work)
* ```-i 0_raw_reads/BGR_130305_mapped_R1.fastq.gz``` --> Input of the forward read
* ```-I 0_raw_reads/BGR_130305_mapped_R2.fastq.gz``` --> Input of the reverse read
* ```-o fastp/BGR_130305_mapped_R1.fastq.gz``` --> Output of the processed forward read
* ```-O fastp/BGR_130527_mapped_R2.fastq.gz``` --> Output of the processed backward read
* ```-t 6``` --> Bases trimmed from the tail of the forward reads (Adapter)
* ```-q 20``` --> Phred score treshold for filtering bases (anything lower than 20 is cut)
* ```-h "fastp_BGR_130305.html"``` --> Name of the html format output file
* ```-R fastp/"fastp_BGR_130305"``` --> Name of the final output file

## Assembly
Once the samples are evaluated, processed and filtered, the genomes are assembled. We are using ```megahit```, an ultra-fast and memory-efficient NGS assembler. It is optimized for metagenomic coassembly and multiple samples.
<br> ```megahit -1 fastp/BGR_130305_mapped_R1.fastq.gz -1 fastp/BGR_130527_mapped_R1.fastq.gz -1 fastp/BGR_130708_mapped_R1.fastq.gz -2 fastp/BGR_130305_mapped_R2.fastq.gz -2 fastp/BGR_130527_mapped_R2.fastq.gz -2 fastp/BGR_130708_mapped_R2.fastq.gz -o assembly --min-contig-len 1000 --presets meta-large -m 0.85 -t 12```
* ```-1 fastp/BGR_130305_mapped_R1.fastq.gz``` --> Input of all forward reads (multiple)
* ```-2 fastp/BGR_130305_mapped_R2.fastq.gz``` --> Input of all reverse reads (multiple)
* ```-o assembly``` --> output directory
* ```--min-contig-len 1000``` --> minimum lenght of the assembled genomes
* ```--presets meta-large``` --> Defines minimum and maximum kmer sizes 
* ```-m 0.85``` -->  max memory in byte to be used in SdBG construction
* ```-t 12``` --> number of CPU threads

 During the assembly process, ```megahit``` tried out several k-mer sizes and wrote the best assembly to final.contigs.fa. All assemblies from different k-sizes are found in outdir/intermediate_contigs/k{N}.contigs.fa.
<br> To visualize the assembled contigs in Bandage, you need to convert the plain-text sequence file into fasta-like graph. 
<br> ``` megahit_toolkit contig2fastg 99 final.contigs.fa > final.contigs.fastg```
<br> The result is loaded in Bandage and looked at.

[Assembly](../Images/Assemlby_Day2.png)

Only Contigs are created, the next step would be Metagenomic Binning to create MAGs

