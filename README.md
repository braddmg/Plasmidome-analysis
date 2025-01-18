# Pipeline for plasmidome analyses
This pipeline analyze a set of related plasmid sequences to unraveling their functional characteristics and genomic relationships. It was used in the publication: [Unraveling the plasmidome of Helicobacter pylori: an unexplored source of potential pathogenicity](https://www.biorxiv.org/content/10.1101/2025.01.06.631533v1)

## MobMess networks
We will employ the MobMess software to create plasmid networks and classify them into backbone, compound, fragment and maximal clusters. For more information about MobMess review the [Paper](https://www.nature.com/articles/s41564-024-01610-3):

The following command processes all samples in FASTA format, filters out contigs shorter than 1000 bp, annotates them using Anvi’o with the COG14 and Pfam databases, and create the MobMess networks: 
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
We can detect ARGs from fasta files with [ABRicate](https://github.com/tseemann/abricate)
In this example, we employ the [CARD](https://card.mcmaster.ca) database, ABRicate supports several other databases. 

```bash
for i in `ls -1 *.fa | sed 's/.fa//'`
do
abricate $i\.fa --db card > $i\.card
```

## Assigning taxonomy to metagenomic contigs
For assigning taxonomy to metagenomic contigs we will use [Kaiju](https://github.com/bioinformatics-centre/kaiju) tool. This tool assigns the closest taxonomy to each contig in a metagenomic assembly using protein-level classification and various databases. 
In this example we will employ the SwissProt database. For instructions on indexing or downloading databases, refer to the [Kaiju github repository](https://github.com/bioinformatics-centre/kaiju).
```bash
kaiju -z 32 -t database/swissprot_nodes.dmp -f database/kaiju_db_refseq.fmi -i metagenomes.fa -o kaiju.out -v
```

