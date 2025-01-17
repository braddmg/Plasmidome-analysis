# Pipeline for plasmidome analyses
In this pipeline we analyze a set of related plasmid sequences to unraveling their functional characteristics and genomic relationships. 

There are multiple tools available for identifying plasmidic contigs in metagenomic data. In this workflow, we will use PlasX, a machine learning-based software designed for plasmid detection:
[PlasX GitHub Repository](https://github.com/michaelkyu/PlasX)
The following command processes all samples in FASTA format, filters out contigs shorter than 500 bp, annotates them using Anvi’o with the COG14 and Pfam databases, and assigns a PlasX score to each contig: 
```bash
for i in `ls *fasta | awk 'BEGIN{FS=".fasta"}{print $1}'`
do
anvi-script-reformat-fasta $i.fasta \
                           -o $i.fa \
                           -l 500 --seq-type NT --simplify-names --prefix $i
anvi-gen-contigs-database -f $i.fa -o $i.db
anvi-run-hmms -c $i.db
anvi-export-gene-calls --gene-caller prodigal -c $i.db -o $i-gene-calls.txt
done

for i in `ls *db | awk 'BEGIN{FS=".db"}{print $1}'`
do
anvi-run-ncbi-cogs -T 32 --cog-version COG14 --cog-data-dir /work/databases/anvio/COG_2014 -c $i.db
anvi-run-pfams -T 32 --pfam-data-dir /work/bmendoza/Tesis/Data/plasmids/anvio/Pfam_v32 -c $i.db
anvi-export-functions --annotation-sources COG14_FUNCTION,Pfam -c $i.db -o $i-cogs-and-pfams.txt
done

for i in `ls *fasta | awk 'BEGIN{FS=".fasta"}{print $1}'`
do
plasx search_de_novo_families \
    -g $i-gene-calls.txt \
    -o $i-de-novo-families.txt \
    --threads 32 \
    --splits 32 \
    --overwrite

plasx predict \
    -a $i-cogs-and-pfams.txt $i-de-novo-families.txt \
    -g $i-gene-calls.txt \
    -o $i-scores.txt \
    --overwrite
done
```
## Detecting antibiotic resistance genes (ARGs)
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

