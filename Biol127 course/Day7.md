# Day 7 Workflow

## Download the data
```
curl -L https://ndownloader.figshare.com/files/28965090 -o V_jascida_genomes.tar.gz
tar -zxvf V_jascida_genomes.tar.gz
ls V_jascida_genomes
```

## Create contigs.dbs from .fasta files
```
cd $WORK/pangenomics/V_jascida_genomes
ls *fasta | awk 'BEGIN{FS="_"}{print $1}' > genomes.txt
```
### remove all contigs <2500 nt
```
for g in $(cat genomes.txt)
do 
    anvi-script-reformat-fasta ${g}_scaffolds.fasta --min-len 2500 --simplify-names -o ${g}_scaffolds_2.5k.fasta
done
```
### generate contigs.db
```
for g in $(cat genomes.txt)
do  
    anvi-gen-contigs-database -f ${g}_scaffolds_2.5k.fasta -o V_jascida_${g}.db --num-threads 4 -n V_jascida_${g}
done
```
### annotate contigs.db
```
for g in *.db
do
    anvi-run-hmms -c $g --num-threads 4
    anvi-run-ncbi-cogs -c $g --num-threads 4
    anvi-scan-trnas -c $g --num-threads 4
    anvi-run-scg-taxonomy -c $g --num-threads 4
done
```

## Visualize contigs.db
```
anvi-display-contigs-stats $WORK/pangenomics/V_jascida_genomes/*db
```
[Bacteria](../Images/Visualization_contigs_bacteria_day7.png)
[Archaea](../Images/Visualization_contig_archaea_day7.png)
[Protista](../Images/Visualization_contigs_protista_day7.png)

## Create external genomes file
```
anvi-script-gen-genomes-file --input-dir $WORK/pangenomics/V_jascida_genomes -o external-genomes.txt
```

## Investigate contamination
```
anvi-estimate-genome-completeness -e external-genomes.txt
```
[Contamination](../Images/Investigate_Contamination.png)

## Visualise contigs for refinement
```
anvi-interactive -c V_jascida_52.db -p V_jascida_52/PROFILE.db
```
[Visualise Contig Refinement](../Images/Visualise_contamination_day7.png)

## Splitting the genome on our good bins
Seperate the good bins stored in "default" from unwanted bins
```
anvi-split -p V_jascida_52/PROFILE.db -c V_jascida_52.db -C default -o V_jascida_52_SPLIT
sed 's/V_jascida_52.db/V_jascida_52_SPLIT\/Clean\/CONTIGS.db/g' external-genomes.txt > external-genomes-final.txt
```

## Estimate completeness of split vs. unsplit genome
```
anvi-estimate-genome-completeness -e external-genomes-final.txt -o genome_completeness.txt
```
[Split_Contamination](../Images/Completeness_split.png)

## Compute the pangenome
### generate a genome storage database
```
anvi-gen-genomes-storage -e external-genomes-final.txt -o V_jascida-GENOMES.db
```
### calculate the pangenome
```
anvi-pan-genome -g V_jascida-GENOMES.db --project-name V_jascida --num-threads 8   
```
### calculate the ANI 
```
anvi-compute-genome-similarity -e external-genomes-final.txt -o ani -p V_jascida/V_jascida-PAN.db  -T 8                                      
```

## Display the pangenome
```
anvi-display-pan -p V_jascida/V_jascida-PAN.db -g V_jascida-GENOMES.db
```
[Pangenome](../Images/Pangenome.png)

### Questions
* Are genes clustered based on sequence similarity or functional annotation?
    * Genes are clustered based on sequence similarity --> Functional annotation is assigned after clustering
* How do you spot a "bad" genome, or "bad" bin in a genome?
    * Low completeness and high contamination
* Use the search function to assign all gene clusters into the following bins: Core genome, Accessory genome, Singletons and Single Copy core genes (SCGs). Include a screenshot of your pangenome into the protocol 
    * [Pangenome](../Images/Pangenome_edit_day7.png)
* If you add more genomes to the pangenome, what would happen to the number of gene clusters in the Core genome and in SCGS?
    * Core genome size decreases --> Fewer genes are shared by all genomes
    * Number of Single Copy core Genes (SCGs) decreases --> higher chance of gene absence or duplication
    * Accesory genome and singleton counts increases --> new genomes introduce novel genes
* Based on the ANI, would you say all genomes belong to the same species?
    * Based on the ANI, all genomes belong to the same species --> ANI is above 0.95 for all genomes.

