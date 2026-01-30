# Day 5 Workflow

## Taxonomic Assignment
Taxonomic annotations are added to the MAGs. ```anvi-run-scg-taxonomy``` associates single-copy core genes in the genes of contigs.db with taxonomy information.
```
anvi-run-scg-taxonomy -c contigs.db -T 20 -P 2
```

Now ```anvi-estimate-scg-taxonomy``` is run: This programm makes quick taxonomy estimates for genomes, metagenomes or bins stored in contigs.db using single-copy core genes
```
anvi-estimate-scg-taxonomy -c contigs.db -p $WORK/metagenomics/anvi_contigs/BGR_130305_profile/PROFILE.db --metagenome-mode --compute-scg-coverages --update-profile-db-with-taxonomy > temp_BGR_130305.txt
anvi-estimate-scg-taxonomy -c contigs.db -p $WORK/metagenomics/anvi_contigs/BGR_130527_profile/PROFILE.db --metagenome-mode --compute-scg-coverages --update-profile-db-with-taxonomy > temp_BGR_130527.txt
anvi-estimate-scg-taxonomy -c contigs.db -p $WORK/metagenomics/anvi_contigs/BGR_130708_profile/PROFILE.db --metagenome-mode --compute-scg-coverages --update-profile-db-with-taxonomy > temp_BGR_130708.txt
```
Lastly, one final summary is done to get comprehensive info about your METABAT2 bins.
```
anvi-summarize -p $WORK/metagenomics/anvi_contigs/BGR_merge/merged_PROFILE.db -c contigs.db -o $WORK/day5/summary_metabat2 -C METABAT2
```

#### Questions
* Did you get a species assignment to the ARCHAEA bins previously identified?
    * METABAT__14: Methanoculleus thermohydrogenotrophicum --> low quality
    * METABAT__32: Methanoculleus sp012797575 --> high quality
    * METABAT__34: Methanosarcina flavescens --> low quality
* Does the HIGH-QUALITY assignment of the bin need revision?
    * No
        * The genome is nearly complete  
        * The taxonomy is coherent
        * The markers agree
        * The only uncertainty is species naming, which is normal and expected

## Genome dereplication (BONUS)
Genome dereplication will be done using ```anvi-dereplicate-genome```
```
anvi-dereplicate-genomes -i file.csv --program fastANI --similarity-threshold 0.95 -o ANI95 --force-overwrite --log-file log_ANI -T 10
anvi-dereplicate-genomes -i file.csv --program fastANI --similarity-threshold 0.90 -o ANI90 --force-overwrite --log-file log_ANI -T 10
anvi-dereplicate-genomes -i file.csv --program fastANI --similarity-threshold 0.80 -o ANI80 --force-overwrite --log-file log_ANI -T 10
```

#### Questions
* How many species do you have in the dataset? 
    * 95% (default)	45	Every genome is unique → species-level resolution
    * 90%	45	Still all genomes are singletons; no merging yet → species-level distinction still maintained
    * 80%	43	Some genomes merged → broader, genus-level similarity
* Try to dereplicate again at 90% identity then at 80% identity. In you own words, explain the differences between the different % identities.
    * 95% ANI (default):
        * Captures species-level diversity.
        * Each genome is unique → no merging.
        * Useful when you want to know how many distinct species are in your dataset.
    * 90% ANI:
        * Still mostly species-level, but could start grouping very closely related species (some genera).
        * In the dataset, all genomes remained singletons → species are still distinct.
    * 80% ANI:
        * Captures broader taxonomic relationships (possibly genus or family level).
        * Merges genomes that are less similar but still related.
        * Example: METABAT12 + METABAT5 → now considered part of the same broader group.
    * --> Lower ANI thresholds merge more genomes into clusters, reducing redundancy but also reducing taxonomic resolution. Your dataset is highly diverse, so even at 80% ANI, most genomes remain separate.
    * --> Binning looks at kmer-sequences and GC-content, but not at the nucleotide identity; chimera; this results in possible mis-binning and bins the same species onto two different bins. This can be found out through Genome dereplication, by looking at the nucleotide identity.