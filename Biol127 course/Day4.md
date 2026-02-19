# Day 4 Worklflow
## Detecting chimeras in MAGs
To check for chimeras and potential contaminations, ```gunc``` is used. Chimeric genomes are genomes wrongly assembled out of two or more genomes coming from seperate organisms. This is repeated for every single MAG; or every archaeal MAG in our case.
```
gunc run -i refine/METABAT__14/METABAT__14-contigs.fa -r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd --out_dir refine/METABAT__14/METABAT__14_gunc_out --detailed_output --threads 12
gunc run -i refine/METABAT__32/METABAT__32-contigs.fa -r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd --out_dir refine/METABAT__32/METABAT__32_gunc_out --detailed_output --threads 12
gunc run -i refine/METABAT__34/METABAT__34-contigs.fa -r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd --out_dir refine/METABAT__34/METABAT__34_gunc_out --detailed_output --threads 12
```
* ```-i refine/METABAT__14/METABAT__14-contigs.fa``` --> Input of the genome/MAG as file.fasta
* ```-r $WORK/databases/gunc/gunc_db_progenomes2.1.dmnd``` --> Reference database to identify taxa in the sample
* ```--out_dir refine/METABAT__14/METABAT__14_gunc_out``` --> Output directory
* ```--detailed_output``` --> Write output in details
* ```--threads 12``` --> Number of threads used for computation

## Creating interactive plots of the chimeras
After running the chimera detection, the results can be visualized in a plot.
```
gunc plot -d refine/METABAT__14/METABAT__14_gunc_out/iamond_output/METABAT__14-contigs.diamond.progenomes_2.1.out -g refine/METABAT__14/METABAT__14_gunc_out/gene_calls/gene_counts.json --out_dir METABAT__14_gunc_out
gunc plot -d refine/METABAT__32/METABAT__32_gunc_out/iamond_output/METABAT__32-contigs.diamond.progenomes_2.1.out -g refine/METABAT__32/METABAT__32_gunc_out/gene_calls/gene_counts.json --out_dir METABAT__32_gunc_out
gunc plot -d refine/METABAT__34/METABAT__34_gunc_out/iamond_output/METABAT__34-contigs.diamond.progenomes_2.1.out -g refine/METABAT__34/METABAT__34_gunc_out/gene_calls/gene_counts.json --out_dir METABAT__34_gunc_out
```
* ```-d refine/METABAT__14/METABAT__14_gunc_out/iamond_output/METABAT__14-contigs.diamond.progenomes_2.1.out``` --> Database result file from ```diamond```
* ``` -g refine/METABAT__14/METABAT__14_gunc_out/gene_calls/gene_counts.json``` --> Gene count file from ```gunc run```
* ```--out_dir METABAT__14_gunc_out``` --> Output directory

<br>[Bin_14_plot](../Images/Bin_14_plot_chimera.png)
<br>[Bin_32_plot](../Images/Bin_32_plot_chimera.png)
<br>[Bin_34_plot](../Images/Bin_34_plot_chimera.png)


#### Questions
* Do you get ARCHAEA bins that are chimeric`
    * METABAT__14: Maybe chimeric (clean from kingdom to genus; species-level failure suggest strain-level heterogeneity)
    * METABAT__32:Not chimeric (CSS=0 and PASS.GUNC=True across all rank; likely a clean bin)
    * METABAT__34: Chimeric (high CSS and PASS.GUNC=False at finer taxonomic levels indicate mixed genomes)
* In your own words, briefly explain what a chimeric bin is?
    * A chimeric bin is a genome that accidentally contains DNA fragments from more than one organism. Instead of representing a single, clean genome, it´s a mosaic, usually because contigs from related taxa were incorrectly grouped during binning.
    * --> One bin, multiple biological sources.

## Manuel bin refinement
As large metagenome assemblies can result in hundreds of bins, pre-select some of the better onesfor manual refinement, e.g. >70% completeness.
```
anvi-refine -c $WORK/metagenomics/anvi_contigs/contigs.db -p refine/PROFILE_refined.db --bin-id METABAT__14 -C METABAT2 
anvi-refine -c $WORK/metagenomics/anvi_contigs/contigs.db -p refine/PROFILE_refined.db --bin-id METABAT__32 -C METABAT2 
anvi-refine -c $WORK/metagenomics/anvi_contigs/contigs.db -p refine/PROFILE_refined.db --bin-id METABAT__34 -C METABAT2 
```
<br> [Bin_14_plot_refined](../Images/Bin_14_plot_refined.png)
<br> [Bin_32_plot_refined](../Images/Bin_32_plot_refined.png)
<br> [Bin_34_plot_refined](../Images/Bin_34_plot_refined.png)
#### Questions
* How much could you improve the quality of your ARCHAEA?
    * MEGABAT__14: 48.68% Completion and 9.21% Redundancy --> 48.68% Completion and 9.21% Redundancy
    * MEGABAT__32: 98.68% Completion and 2.63% Redundancy --> 98.68% Completion and 2.63% Redundancy
    * MEGABAT__34: 38.16% Completion and 0.00% Redundancy --> 38.16% Completion and 0.00% Redundancy

## Visualizing Coverage
Checking how abundant Archaea MAGs are.
```
anvi-interactive -p refine/PROFILE_refined.db -c $WORK/metagenomics/anvi_contigs/contigs.db -C METABAT2
```
[Visualization_coverage](../Images/Visualization_abundance.png)

#### Questions
* How abundant (relatively) are the Archaea bins (bin 32) in the 3 samples?
    * BGR_130305: 8.11
    * BGR_130527: 5.29
    * BGR_130708: 3.52

