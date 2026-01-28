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
    * 55835
* What is the total lenght of the contigs?
    * 142641998 bp

## Mapping Sequencing Reads to Assembled Contigs


