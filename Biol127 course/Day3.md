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
bowtie2 -1 fastp/BGR_130305_mapped_R1.fastq.gz -2 fastp/BGR_130305_mapped_R2.fastq.gz -x anvi_contigs/contigs.anvio.fa.index -S BGR_130305.sam 
bowtie2 -1 fastp/BGR_130527_mapped_R1.fastq.gz -2 fastp/BGR_130527_mapped_R2.fastq.gz -x anvi_contigs/contigs.anvio.fa.index -S BGR_130527.sam 
bowtie2 -1 fastp/BGR_130708_mapped_R1.fastq.gz -2 fastp/BGR_130708_mapped_R2.fastq.gz -x anvi_contigs/contigs.anvio.fa.index -S BGR_130708.sam 
```