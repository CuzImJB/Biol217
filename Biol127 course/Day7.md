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
[Phylogramm](../Images/Visualization_Vjascida_phylo.png)
[Phylogramm_circle](../Images/Visualization_Vjascida_circle.png)

## Create external genomes file
```
anvi-script-gen-genomes-file --input-dir $WORK/pangenomics/V_jascida_genomes -o external-genomes.txt
```

## Investigate contamination
```
anvi-estimate-genome-completeness -e external-genomes.txt
```
