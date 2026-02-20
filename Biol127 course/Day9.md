# Day 9 Workflow

## Aim 
Today, you will learn how a typical virome analysis for a metagenomic dataset looks like. The general steps are:

   * Read pre-processing and assembly (either virus-specific assembly or standard metagenomic assembly)
   * Viral identification
   * Virus-specific quality control
   * Viral clustering
   * Viral binning
   * Abundance estimation
   * Virus-specific annotation
   * Host prediction

Additionally to the steps we will discuss today, it is possible to add steps to the workflow as needed. Often added steps are:

   * Viral taxonomy prediction
   * Viral Lifecycle prediction
   * Virus-specific metabolic analyses

## Commands
### MVP:
```
module load gcc12-env/12.1.0
module load micromamba/1.3.1
micromamba activate MVP

cd $WORK/MVP_test

mvip MVP_00_set_up_MVP -i ./WORKING_DIRECTORY/ -m input_file_timeseries_final.csv  --genomad_db_path ./WORKING_DIRECTORY/00_DATABASES/genomad_db/ --checkv_db_path ./WORKING_DIRECTORY/00_DATABASES/checkv-db-v1.5/

mvip MVP_00_set_up_MVP -i ./WORKING_DIRECTORY/ -m input_file_timeseries_final.csv  --skip_install_databases

mvip MVP_01_run_genomad_checkv -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_02_filter_genomad_checkv -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_03_do_clustering -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_04_do_read_mapping -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv --delete_files

mvip MVP_05_create_vOTU_table -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_06_do_functional_annotation -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv

mvip MVP_07_do_binning -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv --force_outputs

mvip MVP_100_summarize_outputs -i WORKING_DIRECTORY/ -m input_file_timeseries_final.csv
```
### iPHoP:
```
module load miniconda3/4.12.0
conda activate GTDBTk

export GTDBTK_DATA_PATH=./GTDB_db/GTDB_db


gtdbtk de_novo_wf --genome_dir fa_all/ --bacteria --outgroup_taxon p__Patescibacteria --out_dir output/ --cpus 12 --force --extension fa

gtdbtk de_novo_wf --genome_dir fa_all/ --archaea --outgroup_taxon p__Altarchaeota --out_dir output/ --cpus 12 --force --extension fa


module load micromamba/1.3.1
micromamba activate iphop_env

iphop add_to_db --fna_dir fa_all/ --gtdb_dir ./output/ --out_dir ./MAGs_iPHoP_db --db_dir iPHoP_db/

iphop predict --fa_file ./MVP_07_Filtered_conservative_Prokaryote_Host_Only_best_vBins_Representative_Unbinned_vOTUs_Sequences_iPHoP_Input.fasta --db_dir ./MAGs_iPHoP_db --out_dir ./iphop_output -t 12
```

## Questions
### Virus identification and quality control
1. How many free viruses are in the BGR_140717 sample?
    * Command: `grep ">" -c 01_GENOMAD/BGR_140717/BGR_140717_Viruses_Genomad_Output/BGR_140717_modified_summary/*_virus.fna`
    * Answer: 846 
2. How many proviruses are in the BGR_140717 sample?
    * Command: `grep ">" -c 01_GENOMAD/BGR_140717/BGR_140717_Proviruses_Genomad_Output/proviruses_summary/*_virus.fna`
    * Answer: 11
3. How many Caudoviricetes viruses are in all samples together? (Use the filtered version)
    * Command: `grep -c "Caudoviricetes" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv`
    * Answer: 614 + 684 + 850 + 954 + 849 + 1079 + 609 + 368 + 575 + 717 + 468 + 358 + 527 + 559 + 337 + 476 + 401 + 339 = 10 764
4. How many unclassified viruses are in all samples together? (Use the filtered version)
    * Command:`grep -c "Caudoviricetes" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv`
    * Answer: 10 + 9 + 10 + 14 + 11 + 14 + 7 + 4 + 4 + 8 + 4 + 4 + 7 + 9 + 4 + 8 + 8 + 10 = 145
5. What other taxonomies are there across all samples? 
    * Command: `grep -v "Caudoviricetes" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv |grep -v "Unclassified" |grep -v "Sample" > 5.csv`

       | Taxonomy | Genome type | Host type |
        | --- | --- | --- |
        | Mimiviridae | dsDNA | Eukaryote |
        | Phycodnaviridae | dsDNA | Eukaryote |
        | Viruses | Unknown | Unknown |
        | Autolykiviridae | dsDNA | Prokaryote |
        | Retroviridae | ssRNA-RT | Eukaryote |
        | Microviridae | dsDNA | Prokaryote |
6. How many High-quality and Complete viruses are in all samples together? (Use the filtered version and focus on the CheckV quality)
    * Command: ` cut -f 8 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -e "High-quality" -e "Complete" |grep -c "" `
    * Answer: 5
7. Create a table based on the CheckV quality with the columns Sample, Low-quality, Medium-quality, High-quality, Complete. Fill it with the amount of viruses for each of the categories.
    * Command:  for x in BGR_130305  BGR_130527  BGR_130708  BGR_130829  BGR_130925  BGR_131021  BGR_131118  BGR_140106  BGR_140121  BGR_140221  BGR_140320  BGR_140423  BGR_140605  BGR_140717  BGR_140821  BGR_140919  BGR_141022  BGR_150108; 
    do echo $x; echo  "Low-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Low-quality" ; echo  "Medium-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Medium-quality"; echo  "High-quality"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "High-quality"; echo  "Complete"; cut -f 8 02_CHECK_V/"$x"/MVP_02_"$x"_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv | grep -c "Complete"; 
    done > 7.csv 

    
| Sample      | Complete | High_Quality | Medium_Quality | Low_Quality |
|-------------|----------|--------------|----------------|-------------|
| BGR_130305  | 0        | 0            | 0              | 625         |
| BGR_130527  | 0        | 0            | 1              | 693         |
| BGR_130708  | 0        | 0            | 1              | 860         |
| BGR_130829  | 0        | 0            | 3              | 964         |
| BGR_130925  | 0        | 0            | 3              | 856         |
| BGR_131021  | 1        | 0            | 3              | 1091        |
| BGR_131118  | 0        | 0            | 3              | 613         |
| BGR_140106  | 0        | 0            | 0              | 372         |
| BGR_140121  | 1        | 0            | 0              | 578         |
| BGR_140221  | 0        | 1            | 1              | 724         |
| BGR_140320  | 0        | 0            | 1              | 471         |
| BGR_140423  | 0        | 0            | 0              | 362         |
| BGR_140605  | 0        | 0            | 2              | 531         |
| BGR_140717  | 1        | 0            | 3              | 564         |
| BGR_140821  | 0        | 1            | 1              | 338         |
| BGR_140919  | 0        | 0            | 1              | 485         |
| BGR_141022  | 0        | 0            | 0              | 409         |
| BGR_150108  | 0        | 0            | 1              | 348         |

9. For the Complete viruses from all samples, extract all the lines (from the same output file you just used to answer previous questions) so we can take a closer look. Also add a header so we know what each column contains.
    * Command: `grep "Complete" 02_CHECK_V/BGR_*/MVP_02_BGR_*_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv > 8.csv"
    * Answer: 02_CHECK_V/BGR_131021/MVP_02_BGR_131021_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_131021	BGR_131021_NODE_96_length_46113_cov_32.412567	46113	No	53	12	2	Complete	High-quality	100.0	DTR (high-confidence)	1.0		11	0.9741	6	26.0838	Viruses;Duplodnaviria;Heunggongvirae;Uroviricota;Caudoviricetes	dsDNA	Prokaryote
    02_CHECK_V/BGR_140121/MVP_02_BGR_140121_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_140121	BGR_140121_NODE_54_length_34619_cov_66.823718	34619	No	47	24	0	Complete	High-quality	100.0	DTR (high-confidence)	1.0		11	0.9812	10	41.8446	Viruses;Duplodnaviria;Heunggongvirae;Uroviricota;Caudoviricetes	dsDNA	Prokaryote 
    02_CHECK_V/BGR_140717/MVP_02_BGR_140717_Filtered_Relaxed_Merged_Genomad_CheckV_Virus_Proviruses_Quality_Summary.tsv:BGR_140717	BGR_140717_NODE_168_length_31258_cov_37.020094	31258	No	44	21	0	Complete	High-quality	100.0	DTR (medium-confidence)	1.0		11	0.9818	10	40.2292	Viruses;Duplodnaviria;Heunggongvirae;Uroviricota;Caudoviricetes	dsDNA	Prokaryote
10. * In what samples were the complete viruses found?
        * Answer: BGR_131021, BGR_140121 and BGR_140717
    * Are they integrated proviruses?
        * Answer: No
    * How long are they?
        * Answer: 46113, 34619 and 31258 nucleotides
    * How many viral hallmark genes do they have?
        * Answer: 6, 10 and 10 
    * What percentage of the viral genes are viral hallmark genes?
        * Answer: 50%, 42% and 48%
    * Why are there more genes (gene_count) than viral genes and host genes combined?
        * Answer: Gene_count are all genes encoded in this sequence. We know for sure that some match known host and virus genes (viral_genes, host_genes). For the other genes, we simply don't know. They are just normal (viral) genes without any matches to the reference databases of geNomad and CheckV. 

### Clustering and Abundance 
10. Find the clustering output. What is the difference between the three .tsv files?
    * Answer: Unfiltered (All output), Filtered (Potential not virus sequences are filtered) and Representative (One representative of each bin is chosen)
11. How many cluster representatives are there?
    * Command: `grep -c "" 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv`
    * Answer: 5375 - 1(Header) = 5374
12. How many of the cluster representatives are proviruses?
    * Command: `cut -f 5 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv | grep "Yes" -c`
    * Answer: 91
13. What clusters do the complete viruses from 8. belong to? How large are the clusters?
    * Command: `grep -e "BGR_131021_NODE_96_length_46113_cov_32.412567" -e "BGR_140121_NODE_54_length_34619_cov_66.823718" -e "BGR_140717_NODE_168_length_31258_cov_37.020094" 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv`
    for x in BGR_131021_NODE_96_length_46113_cov_32.412567 BGR_140121_NODE_54_length_34619_cov_66.823718 BGR_140717_NODE_168_length_31258_cov_37.020094; 
    do echo $x; cut -f 2 03_CLUSTERING/MVP_03_All_Sample_Filtered_Relaxed_Merged_Genomad_CheckV_Representative_Virus_Proviruses_Quality_Summary.tsv | grep "$x" | grep "," -o |grep "" -c; 
    done
    * Answer:   BGR_131021_NODE_96_length_46113_cov_32.412567--> BGR_131021_NODE_96_length_46113_cov_32.412567 Members: 6
                BGR_140121_NODE_54_length_34619_cov_66.823718--> BGR_140121_NODE_54_length_34619_cov_66.823718 Members: 24
                BGR_140717_NODE_168_length_31258_cov_37.020094--> BGR_140717_NODE_168_length_31258_cov_37.020094 Members: 10
14. Now we want to look at abundances. The files can be found in 04_READ_MAPPING, split per sample. Open any of the CoverM files and ignore everything except the first two columns.
What does this file tell you, conceptionally? Meaning: What do the lines in this file represent (i.e. what kind of data is described in line 2, line 3, .....), who do the IDs listed in the second column belong to, what kind of data was combined to generate this file, and what can we learn from it? 
    * Answer: For this file, the entire reads of one sample were mapped against all 5374 viral cluster representatives. Column 2 contains the ID of the currently described cluster representative. From this file, we can learn how abundant each of the recovered viruses (vOTUs) is in each of the samples (each sample has its own file). 
15. If you scroll through the CoverM file you selected, you will come across lines where all of the metrics (except length) are zero. How could this have happened? Is it a bug or a feature?!
    * Answer: As you just learned, this file contains mapping information about all viruses but for only one sample. If you look back at your table from 7., not every sample contains 5000+ viruses. So the lines with only zeros are cluster representatives for viruses that a) weren't assembled from this particular sample and b) didn't even have any matching reads (if you remember from Alex' lectures: Not all reads become contigs, not all contigs become vMAGs). Those can be e.g. viruses that weren't present at this timepoint (e.g. died out), in this sample (they swam away when the technician came to collect), or that were in the sample but didn't get sequenced or the reads got filtered out in the read cleaning step.
16. Right now, the output for this module is spread across multiple files, which is inconvenient and happens a lot in bioinformatics. Luckily, each file has a column where the sample information is listed anyways, so we can merge them. Task: Merge all CoverM files for all the samples
    * Command: `grep --no-filename -v "Sample" 04_READ_MAPPING/BGR_*/*_CoverM.tsv > 04_READ_MAPPING/temp_CoverM_output.tsv ; grep "Sample" 04_READ_MAPPING/BGR_130305/BGR_130305_CoverM.tsv > 04_READ_MAPPING/temp.tsv ; cat 04_READ_MAPPING/temp.tsv 04_READ_MAPPING/temp_CoverM_output.tsv > 04_READ_MAPPING/merged_output.tsv ; rm 04_READ_MAPPING/temp*.tsv`
17. Using the file you just created, how abundant are the complete viruses in different samples (RPKM)?
    * Command: for x in "BGR_131021_NODE_96_length_46113_cov_32.412567" "BGR_140121_NODE_54_length_34619_cov_66.823718"  "BGR_140717_NODE_168_length_31258_cov_37.020094" ; 
    do grep "$x" 04_READ_MAPPING/merged_output.tsv | cut -f1,2,11 ;
    done > 04_READ_MAPPING/merged_viruses_RPKM.tsv
    * Answer: 
    
        | Sample | Contig | RPKM |
        | --- | --- | --- |
### Annotations
18. Find the annotation output from module 06. Based on the file naming alone, what is each of the files for and what is different between files with the same extension (file ending, like .tsv)?
    * Answer: The two tsv files contain gene annotation information for only the 5000+ viral cluster representatives. (GENOMAD, PHROGS and PFAM are the databases that were used for annotation.)
    The difference between them is, that one is based on something conservative and the other on something relaxed. Those are filtering parameters. If you count the amount of lines in each file, you can see that the conservative file (7749) is way shorter than the relaxed file (24648).
    The faa files contain the actual protein predictions; one for the cluster representatives and one for all viruses.
19. How many genes does the complete virus have?
    * Answer: BGR_131021_NODE_96_length_46113_cov_32.412567: 53 Genes --> DNA, RNA  and nucleotide metabolism, head and packaging, other and unknown
            BGR_140121_NODE_54_length_34619_cov_66.823718: 47 Genes --> connector, DNA, RNA and nucleotide metabolsim, head and packaging, lysis, other, tail and unknown
            BGR_140717_NODE_168_length_31258_cov_37.020094: 44 Genes --> connector, DNA, RNA and nucleotide metabolism, head and packaging, lysis, other, tail and unknown
20. What kind of metabolism are the viruses involved in?
    * Answer:  Selenocompound metabolism,  Pyrimidine metabolism, Sulfur metabolism, Nicotinate and nicotinamide metabolism,  Purine metabolism, Porphyrin and chlorophyll metabolism
21. Are there any toxin genes? Briefly look up the function of them (What do they do, where do they occur, what does it mean for this virus and its host?)
    * Answer: Zot-like toxin/ Zonular occludens toxin (Zot) --> Zot-like toxins are proteins encoded by filamentous bacteriophages in bacteria like Vibrio cholerae. They disrupt tight junctions in host epithelial cells, increasing permeability, which can contribute to diarrhea. For the phage, Zot also helps assemble and release virus particles. In the host bacterium, carrying Zot can increase virulence and spread via horizontal gene transfer.
### Binning
22. Find this table and open it: 07_BINNING/07C_vBINS_READ_MAPPING/MVP_07_Merged_vRhyme_Outputs_Unfiltered_best_vBins_Memberships_geNomad_CheckV_Summary_read_mapping_information_RPKM_Table.tsv
What does each line represent (focus on column 1,2,5)? What is the purpose of this table?
    * Answer: This table holds information for the different viral bins. Each line is one bin member, so one virus. By design of this pipeline, binning was performed on the vOTUs. So those viruses are the cluster representatives from module 03. This table aggregates most output from previous modules (like quality, taxonomy, abundance etc.). For each time point, we have the covered fraction (how much of the contig has reads mapped to it) and the RPKM ("Reads Per Kilobase Million", coverage of the contig, normalized by sequencing depth (explanation in words, explanation as formula)). So to put it simple: We can see the abundance of each cluster representative at different time points.
23. How many High-quality "viruses" are there after binning?
    * Answer: 9
24. ilter the results so you can only see vBin_16 (or just highlight the corresponding 5 lines). The metrics for all bin members are the same. But membership_provirus has different values. What does this tell you about the vBin/vMAG 16?
    * Answer: We do binning to generate vMAGs, viral metagenome assembled genomes. This means, all members of a vMAG or vBin should belong to the same virus. Since the virus 16 was detected both as an integrated provirus and a free virus, it is likely a virus that can enter both lytic and lysogenic cycle (assuming the binning is correct).
25. Are your complete viruses part of a bin? Why does this result make sense?
    * Answer: The complete viruses are not part of a bin. The purpose of binning is to restore a fragmented genome by grouping together the individual fragments based on coverage information. Since our complete viruses are, well, complete, we don't need to find their missing pieces.
### Host Prediction
26. Find this table and open it: 08_iPHoP/Host_prediction_to_genome_m90.csv. The input file to generate this table is 08_iPHoP/MVP_07_Filtered_conservative_Prokaryote_Host_Only_best_vBins_Representative_Unbinned_vOTUs_Sequences_iPHoP_Input_clean.fna. Based on the first column of the table and the name of the input file: What does each line represent? (Meaning: What kind of viruses are in this file?)
    * Answer: The input file contains the best vBins (written as vBin_number in the table, with vRhyme_number__sequenceID being individual sequences of that bin), and all unbinned vOTU representatives. Each line is one virus (unbinned vOTU representative, vMAG, or vBin member=binned vOTU representative), where a host could be predicted. Viruses without a prediction are not in the file. Hosts that start with BGR_* are MAGs from the dataset, while all other host identifyers are from reference databases.
27. What hosts were predicted for the complete viruses? From what habitat did the hosts come from?
    * Answer: d__Bacteria;p__Bacillota_A;c__Clostridia;o__Tissierellales;f__Peptoniphilaceae;g__;s__ Habitat: Biogasreactor
28. Find an example for a virus with more than one host prediction
    A. With closely related hosts:BGR_130305_NODE_1097_length_6048_cov_14.323544 --> d__Bacteria;p__Firmicutes;c__Bacilli;o__Bacillales;f__Bacillaceae_G;g__Bacillus_A;s__Bacillus_A tropicus and d__Bacteria;p__Firmicutes;c__Bacilli;o__Bacillales;f__Bacillaceae_G;g__Bacillus_A;s__Bacillus_A luti
        a. Biological reasoning: These hosts are closely related species within the same genus, so the virus may naturally infect multiple Bacillus species due to similar surface receptors or ecological niches.
        b. Prediction method: Likely based on sequence homology, CRISPR spacer matches, or k-mer similarity to host MAGs.
        c. Potential contamination of the host MAGs: Low; because the hosts are closely related, overlapping sequences might represent true multi-host infection rather than contamination.
    B. With distantly related hosts: BGR_130527_NODE_365_length_14183_cov_15.647721 --> d__Bacteria;p__Cloacimonadota;c__Cloacimonadia;o__Cloacimonadales;f__Cloacimonadaceae;g__UBA3900;s__ and d__Bacteria;p__Firmicutes_A;c__Clostridia;o__Tissierellales;f__Peptoniphilaceae;g__Levyella;s__Levyella massiliensis and d__Bacteria;p__Firmicutes_A;c__Clostridia;o__Tissierellales;f__Peptoniphilaceae;g__Anaerosphaera;s__Anaerosphaera multitolerans
        a. Biological reasoning: Infection of phylogenetically distant hosts is unusual and may indicate broad host range, horizontal gene transfer, or potential misassignment.
        b. Prediction method: Likely k-mer or sequence composition similarity, which can sometimes link unrelated hosts.
        c. Potential contamination of the host MAGs: High; distant host prediction could be an artifact due to chimeric MAGs or contamination during assembly.
        




