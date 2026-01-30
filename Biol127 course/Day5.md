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
