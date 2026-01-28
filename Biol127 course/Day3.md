# Day 3 Worklflow

## Quality Assessment of Assemblies

```quast``` is a Quality Assessment Tool to evaluate genome assembly. It is used to evaluate the results from ```megahit```.
```
metaquast /work_beegfs/sunam227/metagenomics/assembly/final.contigs.fa -o metaquast -m 1000 -t 6
```
* ``` /work_beegfs/sunam227/metagenomics/assembly/final.contigs.fa ``` --> Used file
* ``` -o metaquast ``` --> Output directory
* ``` -m 1000 ``` --> Minimum contig lenght
* ```-t 6 ``` --> Maximum number of CPU threads used

Questions:
* What is your N50 value? Why is this value relevant?
    * N50=3014. A higher N50 value indicates fewer, longer contigs, suggesting a more complete and less fragmented assembly.
* How many contigs are assembled?
    * 55.835
* What is the total lenght of the contigs?
    * 142.641.998 bp

## Mapping Sequencing Reads to Assembled Contigs
If you simply use the ```final.contigs.fa``` file for read mapping, you will run into issues such as:
* The contig names are too complicated and may contain special characters.
* Many contigs are too small for the mapping to be meaningful
* The contigs and nucleotide positions are random and not organized in a systematic manner.
* The fasta format usually cannot store many types of relevant information such as:
    * Where the genes are
    * Tetra-nucleotide
    * Which species a contig may belong to
    * Detailed information on individual nucleotides
* --> The assembly needs to be cleaned up

### Re-formatting the contigs
The contig sequence IDs are simplified and short contigs are cut.
```
anvi-script-reformat-fasta /work_beegfs/sunam227/metagenomics/assembly/final.contigs.fa -o anvi_contigs/contigs.anvio.fa --min-len 1000 --simplify-names --report-file names.txt
```
* ```assembly/final.contigs.fa``` --> Used file
* ```-o anvi_contigs/contigs.anvio.fa``` --> Write output to fasta.file
* ```--min-len 1000``` --> Minimum lenght of the contigs
* ```--simplify-names``` --> Names are simplified
* ```--report-file names.txt``` --> Old and new sequence IDs are wrote in a table

### Indexing the contigs
If your contigs are organized and indexed, the mapping will run much faster. To index the re-formatted reads, we will use a programm from the ```bowtie2``` suite.
```
bowtie2-build /work_beegfs/sunam227/metagenomics/anvi_contigs/contigs.anvio.fa /work_beegfs/sunam227/metagenomics/anvi_contigs/contigs.anvio.fa.index 
```
* ```/work_beegfs/sunam227/metagenomics/anvi_contigs/contigs.anvio.fa``` --> Input of re-formatted contigs
* ```/work_beegfs/sunam227/metagenomics/anvi_contigs/contigs.anvio.fa.index``` --> Output of the  index file 

### Mapping reads onto contigs
```bowtie2``` is used for the actual mapping. The output of read mapping is called a Sequence Alignment Map file (.sam).
```
bowtie2 -1 fastp/BGR_130305_mapped_R1.fastq.gz -2 fastp/BGR_130305_mapped_R2.fastq.gz -x anvi_contigs/contigs.anvio.fa.index -S BGR_130305.sam --very-fast
bowtie2 -1 fastp/BGR_130527_mapped_R1.fastq.gz -2 fastp/BGR_130527_mapped_R2.fastq.gz -x anvi_contigs/contigs.anvio.fa.index -S BGR_130527.sam --very-fast
bowtie2 -1 fastp/BGR_130708_mapped_R1.fastq.gz -2 fastp/BGR_130708_mapped_R2.fastq.gz -x anvi_contigs/contigs.anvio.fa.index -S BGR_130708.sam --very-fast
```
* ```-1 fastp/BGR_130305_mapped_R1.fastq.gz``` --> Input of forward read
* ```-2 fastp/BGR_130305_mapped_R2.fastq.gz``` --> Input of reverse read
* ``` -x anvi_contigs/contigs.anvio.fa.index``` --> Used contig index file
* ```-S BGR_130305.sam``` --> Output of .sam file
* ``` --very-fast``` --> Run mode. Faster, but less acurate

To convert the .sam file into a file, which is readable for the computer, we need to convert them into .bam files.
```
samtools view -Sb anvi_contigs/BGR_130305.sam > anvi_contigs/BGR_130305.bam
samtools view -Sb anvi_contigs/BGR_130527.sam > anvi_contigs/BGR_130527.bam
samtools view -Sb anvi_contigs/BGR_130708.sam > anvi_contigs/BGR_130708.bam
```
* ```-Sb anvi_contigs/BGR_130708.sam > anvi_contigs/BGR_130708.bam``` --> Conversion of .sam to .bam

### Sorting mapped reads
Sorting the mapped reads speeds up data processing and allows downstream analysis such as visualization and variant calling.
```
anvi-init-bam anvi_contigs/BGR_130305.bam -o BGR_130305_sorted.bam
anvi-init-bam anvi_contigs/BGR_130527.bam -o BGR_130527_sorted.bam
anvi-init-bam anvi_contigs/BGR_130708.bam -o BGR_130708_sorted.bam
```
* ``` -o BGR_130305_sorted.bam``` --> Output of the sorted .bam file

## Binning reads
With the results from read mapping, we bin our contigs into individual genomes (MAGs) and start to figure out which microbes are present in the samples.

### Generating contigs database
The ```fasta``` formatting is limited and cannot store more complex types of information. The ```fasta``` file becomes a database that can store many types of informations that are of interest to us, such as the taxonomy of the genomes in the bins.
```
anvi-gen-contigs-database -f anvi_contigs/contigs.anvio.fa -o anvi_contigs/contigs.db -n biol217
```
* ```-f anvi_contigs/contigs.anvio.fa``` --> Input of reformatted fasta file
* ``` -o anvi_contigs/contigs.db``` --> Output of contigs database file
* ```-n biol217``` --> Project name

### Annotating ORFs
It is also a good idea to search for potential biological functions that the predicted ORFs may have. This information can come in handy when you want to study the metabolism of the species in the experiment. We will perform a HMM search against several collections of genes to see if the ORFs predicted are similar to any known genes.
```
anvi-run-hmms -c anvi_contigs/contigs.db --num-threads 4 
```
* ```-c anvi_contigs/contigs.db``` --> Input of the contig database
* ```--num-threads 4 ``` --> number of CPU threads used

### Visualizing the contigs database
An ```anvi'o``` profile is like an upgraded ```anvi'o``` database that can also store read mapping results and detailed per-nucleotide information.
```
anvi-profile -i anvi_contigs/BGR_130305_sorted.bam -c anvi_contigs/contigs.db --output-dir anvi_contigs/BGR_130305_profile
anvi-profile -i anvi_contigs/BGR_130527_sorted.bam -c anvi_contigs/contigs.db --output-dir anvi_contigs/BGR_130527_profile
anvi-profile -i anvi_contigs/BGR_130708_sorted.bam -c anvi_contigs/contigs.db --output-dir anvi_contigs/BGR_130708_profile
```
* ```-i anvi_contigs/BGR_130305_sorted.bam``` --> Input of sorted and indexed .bam file+
* ``` -c anvi_contigs/contigs.db``` --> Input of contigs database
* ``` --output-dir anvi_contigs/BGR_130708_profile``` --> Output directory

### Merging ```anvi'o``` profiles from all samples
To analyze and compare all samples together, we merge the profiles coming from the different samples into one profile.
```
anvi-merge anvi_contigs/BGR_130305_profile anvi_contigs/BGR_130527_profile anvi_contigs/BGR_130708_profile -o anvi_contigs/BGR_merge -c anvi_contigs/contigs.db --enforce-hierarchical-clustering
```
* ```anvi_contigs/BGR_130305_profile anvi_contigs/BGR_130527_profile anvi_contigs/BGR_130708_profile -o anvi_contigs/BGR_merge``` --> Profiles that will be merged
* ```-o anvi_contigs/BGR_merge``` --> Output directory
* ```-c anvi_contigs/contigs.db``` --> Contig database file
* ```

