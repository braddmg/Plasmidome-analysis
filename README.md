# Pipeline for plasmidome analysis
This pipeline analyzes plasmid sequences to uncover their functional characteristics and genomic relationships. It was used in the publication: [Unraveling the plasmidome of Helicobacter pylori: an unexplored source of potential pathogenicity](https://www.biorxiv.org/content/10.1101/2025.01.06.631533v1)

## MobMess networks
Using MobMess, plasmid networks are generated and classified into backbone, compound, fragment, and maximal clusters. For more information about MobMess see the [Paper](https://www.nature.com/articles/s41564-024-01610-3):

The following command processes the plasmid.fasta file containing all plasmid sequences obtained from [PlasMidScope](https://plasmid.deepomics.org/database/plasmid), filters contigs < 1000 bp, annotates them with Anvi’o (using COG14 and Pfam databases), and creates MobMess networks.
Files used in the analysis can be found in the repository
```bash
anvi-script-reformat-fasta plasmids.fasta \
                           -o plasmids.fa \
                           -l 1000 --seq-type NT
    anvi-gen-contigs-database -f plasmids.fa -o plasmids.db
    anvi-run-hmms -c plasmids.db
    anvi-export-gene-calls --gene-caller prodigal -c plasmids.db -o plasmids-gene-calls.txt

anvi-run-ncbi-cogs -T 64 --cog-version COG14 --cog-data-dir /work/databases/anvio/COG_2014 -c plasmids.db
anvi-run-pfams -T 64 --pfam-data-dir /work/bmendoza/Tesis/Data/plasmids/anvio/Pfam_v32 -c plasmids.db
anvi-export-functions --annotation-sources COG14_FUNCTION,Pfam -c plasmids.db -o plasmids-cogs-and-pfams.txt

mobmess systems \
    --sequences plasmids.fa \
    --complete plasmids.txt \
    --output mobmess \
    --threads 64

```
## Alignment and visualization of specific plasmid networks
To visualize specific plasmids in the network, we used the next MobMess function:
```bash
contigs=IMGPR_plasmid_2563367159_000001,COMPASS_NC_014556_1,IMGPR_plasmid_2617271162_000001,IMGPR_plasmid_2619619205_000001,COMPASS_NC_019561_1,IMGPR_plasmid_2619619172_000001
mobmess visualize -s plasmids.fa -a plasmids-cogs-and-pfams.txt -g plasmids-gene-calls.txt -o figura/ -T 32 --contigs $contigs --align-blocks-height 1

```

## Mash distance-based dendogram 
Non-redundant representative plasmids were separated with the next command: 

```bash
seqtk subseq plasmids.fa representative.txt > nrplasmids.fasta
```

We then split plasmid sequences into multiple FASTA files using a Python script available in the repository:
```bash
python sep.py nrplasmids.fasta
```
Generate the dendrogram using [Mashtree](https://github.com/lskatz/mashtree):
```bash
mashtree --mindepth 0 *.fasta --outtree H_pylori.tree
```
## Enrichment analysis
Enrichment analysis followed this [tutorial](https://merenlab.org/2016/11/08/pangenomics-v2/). Import sequences individually to Anvio and annotate them as described in the MobMess section. Then, perform pangenomic and enrichment analysis:

```bash
#Perform pangenome analysis
anvi-gen-genomes-storage -e external-genomes.txt -o Filo-GENOMES.db 
anvi-pan-genome -g Filo-GENOMES.db --project-name Filo --num-threads 32 

#Import layers
anvi-import-misc-data groupos.txt \
                      -p Filo/Filo-PAN.db \
                      --target-data-table layers

#Enrichment analysis
anvi-compute-functional-enrichment -p Filo/Filo-PAN.db\
                                   -g Filo-GENOMES.db \
                                   -o functional-enrichment-txt \
                                   --category-variable group \
                                   --annotation-source COG14_FUNCTION
```
## Coding Sequences annotation with COG classifier
Amino acid sequences from MobMess were divided into Group 1 and Group 2 plasmids (available in the repository). Each file was processed with the  [COGclassifier](https://github.com/moshi4/COGclassifier) tool. 

```bash
COGclassifier -i G1.faa -o G1/
```
