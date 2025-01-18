# Pipeline for plasmidome analyses
This pipeline analyze a set of related plasmid sequences to unraveling their functional characteristics and genomic relationships. It was used in the publication: [Unraveling the plasmidome of Helicobacter pylori: an unexplored source of potential pathogenicity](https://www.biorxiv.org/content/10.1101/2025.01.06.631533v1)

## MobMess networks
We will employ the MobMess software to create plasmid networks and classify them into backbone, compound, fragment and maximal clusters. For more information about MobMess review the [Paper](https://www.nature.com/articles/s41564-024-01610-3):

The following command processes the plasmid.fasta file containing all plasmid sequences obtained from [PlasMidScope](https://plasmid.deepomics.org/database/plasmid), filters out contigs shorter than 1000 bp, annotates them using Anvi’o with the COG14 and Pfam databases, and create the MobMess networks: 
Files used in the analysis can be found in the repository
```bash
anvi-script-reformat-fasta plasmids.fasta \
                           -o plasmids.fa \
                           -l 0 --seq-type NT
    anvi-gen-contigs-database -f plasmids.fa -o plasmids.db
    anvi-run-hmms -c plasmids.db
    anvi-export-gene-calls --gene-caller prodigal -c plasmids.db -o plasmids-gene-calls.txt

anvi-run-ncbi-cogs -T 64 --cog-version COG14 --cog-data-dir /work/databases/anvio/COG_2014 -c plasmids.db
anvi-run-pfams -T 64 --pfam-data-dir /work/bmendoza/Tesis/Data/plasmids/anvio/Pfam_v32 -c plasmids.db
anvi-export-functions --annotation-sources COG14_FUNCTION,Pfam -c plasmids.db -o plasmids-cogs-and-pfams.txt

plasx search_de_novo_families \
    -g plasmids-gene-calls.txt \
    -o plasmids-de-novo-families.txt \
    --threads 64 \
    --splits 64 \
    --overwrite

mobmess systems \
    --sequences plasmids.fa \
    --complete plasmids.txt \
    --output mobmess \
    --threads 64

```
## Alignment and visualization of specific related plasmids
We can visualize the alignment of specific plasmids in the network with the next command. 

```bash
contigs=IMGPR_plasmid_2563367159_000001,COMPASS_NC_014556_1,IMGPR_plasmid_2617271162_000001,IMGPR_plasmid_2619619205_000001,COMPASS_NC_019561_1,IMGPR_plasmid_2619619172_000001
mobmess visualize -s plasmids.fa -a plasmids-cogs-and-pfams.txt -g plasmids-gene-calls.txt -o figura/ -T 32 --contigs $contigs --align-blocks-height 1

```

## Mash distance-based dendogram 
Non-redundant representative plasmids were isoalted with the next command: 

```bash
seqtk subseq plasmids.fa representative.txt > nrplasmids.fasta
```
We then separated plasmid sequences using a python script available in the repository:
```bash
python sep.py nrplasmids.fasta
```
Mash distance-based dendogram was created with the [Mashtree](https://github.com/lskatz/mashtree) software with the next command:
```bash
mashtree --mindepth 0 *.fasta --outtree H_pylori.tree
```
## Enrichment analysis
Enrichment analysis was performed as indicated in the web tutorial: https://merenlab.org/2016/11/08/pangenomics-v2/
We imported sequences individually to Anvio and perform the COG and Pfam annotation as indicated above in the MobMess section. Then we performed a pangenomic analyses and ran the enrichment analysis with the next command: 

Then we used the next command to perform the analysis:
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
## Coding Sequences annotatgion with COG classifier

