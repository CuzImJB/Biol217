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
