"Genomic Stability and Genetic Defense Systems in *Dolosigranulum
pigrum* a Candidate Beneficial Bacterium from the Human Microbiome"
Supplemental Methods
================

# PPanGGOLiN ANALYSIS

[PPanGGOLiN v1.1.141](https://github.com/labgem/PPanGGOLiN/releases)
([Gautreau et al. 2020](#ref-gautreau2020); [Bazin et al.
2020](#ref-bazin2020)) was
[installed](https://github.com/labgem/PPanGGOLiN/wiki/Installation) in a
Python environment called `PPanGGOLiN.`

## Importing Prokka annotations and Anvi’o 7 clusters

In order to import the Anvi’o clustering into PPanGGOLiN we needed:

1.  A .tsv file listing in the first column the gene family names, and
    in the second column the gene ID that is used in the annotation
    files.
2.  The annotated genomes with gene IDs that match the ones listed in
    the previous .tsv file.

Using the `anvi-summarize` output
`analysis_Anvio7/Pangenomic_Results_Dpig/PAN_DPIG_prokka_gene_clusters_summary.txt.gz`
described before we can create the .tsv file:

``` r
Dpig_Anvio7 <- read_delim("analysis_Anvio7/Pangenomic_Results_Dpig/PAN_DPIG_prokka_gene_clusters_summary.txt.gz", "\t")
Dpig_Anvio7 <- Dpig_Anvio7 %>% 
  unite(new_id, genome_name:gene_callers_id, sep = "___", remove = FALSE)
Clusters_Dpig_Anvio7 <- select(Dpig_Anvio7, c(gene_cluster_id, new_id))
```

``` r
write_delim(Clusters_Dpig_Anvio7, "analysis_PPanGGOLiN_Anvio7/Clusters_Dpig_Anvio7.tsv", col_names=FALSE)
```

**IMPORTANT:** The gene\_callers\_id provided by Anvi’o don’t match the
original ones in the Prokka annotation. The Prokka annotated genomes
were parsed into two text files, one for gene calls and one for
annotations, with the script `gff_parser.py`. By default Prokka
annotates also tRNAs, rRNAs and CRISPR regions. However, `gff_parser.py`
will only utilize open reading frames reported by Prodigal in the Prokka
output in order to be compatible with the pangenomic Anvi’o pipeline.
While parsing new gene\_callers\_id are generated only for the ORFs that
will be imported into Anvi’o. Fortunately
`anvi-get-sequences-for-gene-calls` can be used to export new .gff files
with only the ORFs and these can be used for PPanGGOLiN, instead of the
original Prokka .gff files:

``` bash
#conda activate anvio-7
mkdir -p "analysis_Anvio7/Exported_gffs"

path_f="analysis_Anvio7/Contigs_db_prokka_Dpig"
path_o="analysis_Anvio7/Exported_gffs"

for file in $path_f/*.db; do
    FILENAME=`basename ${file%.*}`
    anvi-get-sequences-for-gene-calls -c $file --export-gff3 \
                                      -o $path_o/Anvio7_$FILENAME.gff
      
done
```

We created lists with the file names for both the Anvi’o exported .gff
(`analysis_PPanGGOLiN_Anvio7/Anvio7GenomesExported.gff.txt`) and the
renamed .fasta files
(`analysis_PPanGGOLiN_Anvio7/Anvio7GenomesReformatted.fa.txt`) and use
them to run the `annotate` PPanGGOLiN subcommand. Then, the `cluster`
PPanGGOLiN subcommand was run using the .tsv file created before from
the `anvi-summarize` output.

``` bash
#conda activate PPanGGOLiN
ppanggolin annotate --anno analysis_PPanGGOLiN_Anvio7/Anvio7GenomesExported.gff.txt --fasta analysis_PPanGGOLiN_Anvio7/Anvio7GenomesReformatted.fa.txt -o analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7 --basename FromAnvio7

ppanggolin cluster -p analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7/FromAnvio7.h5 --clusters analysis_PPanGGOLiN_Anvio7/Clusters_Dpig_Anvio7.tsv --infer_singletons
```

## Graphing and Partitioning

The PPanGGOLiN`graph` subcommand has only a single other option, which
is ‘-r’ or ‘–remove\_high\_copy\_number.’ If used, it will remove the
gene families that are too duplicated in your genomes. This is useful if
you want to visualize your pangenome afterward and want to remove the
biggest hubs to have a clearer view. It can also be used to limit the
influence of very duplicated genes such as transposase or ABC
transporters in the partition step.

We ran the PPanGGOLiN `graph` and `partition` subcommands with default
parameters.

``` bash
#conda activate PPanGGOLiN
ppanggolin graph -p analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7/FromAnvio7.h5

ppanggolin partition -p analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7/FromAnvio7.h5
```

## Writing outpus

For details on PPanGGOLiN outputs see:
<https://github.com/labgem/PPanGGOLiN/wiki/Outputs>

``` bash
#conda activate PPanGGOLiN
ppanggolin write -p analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7/FromAnvio7.h5 -o analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7 --light_gexf --gexf  --csv --Rtab --stats --partitions --projection --families_tsv -f
```

## Finding Regions of Genome Plasticity

For details on RGPs see:
<https://github.com/labgem/PPanGGOLiN/wiki/Regions-of-Genome-Plasticity>

``` bash
#conda activate PPanGGOLiN
ppanggolin rgp -p analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7/FromAnvio7.h5

ppanggolin spot -p analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7/FromAnvio7.h5 --label_priority ID --draw_hotspots -o analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7/spots_ID -f

ppanggolin write -p analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7/FromAnvio7.h5 -o analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7 --regions --spots -f
```

## Summary of used parameters

The PPanGGOLiN `info` subcommand indicates, for each steps of the
analysis, the PPanGGOLiN parameters that were used and the source of the
data if appropriate.

``` bash
#conda activate PPanGGOLiN
ppanggolin info -p analysis_PPanGGOLiN_Anvio7/OutputFromAnvio7/FromAnvio7.h5 --parameters
```

The output for this command was:

-   <u>**annotation**</u>

    -   `read_annotations_from_file`: True

-   <u>**cluster**</u>

    -   `read_clustering_from_file`: True
    -   `infer_singletons`: True

-   <u>**graph**</u>

    -   `removed_high_copy_number_families`: False

-   <u>**partition**</u>

    -   `beta`: 2.5
    -   `free_dispersion`: False
    -   `max_node_degree_for_smoothing`: 10
    -   `computed_K`: True
    -   `K`: 3

-   <u>**RGP**</u>

    -   `persistent_penalty`: 3
    -   `variable_gain`: 1
    -   `min_length`: 3000
    -   `min_score`: 4
    -   `dup_margin`: 0.05

-   <u>**spots**</u>

    -   `set_size` : 3
    -   `overlapping_match` : 2
    -   `set_size`: 3
    -   `overlapping_match`: 2
    -   `exact_match`: 1

# <u>REFERENCES</u>

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-bazin2020" class="csl-entry">

Bazin, Adelme, Guillaume Gautreau, Claudine Médigue, David Vallenet, and
Alexandra Calteau. 2020. “panRGP: A Pangenome-Based Method to Predict
Genomic Islands and Explore Their Diversity.” *Bioinformatics* 36
(Supplement\_2): i651–58.
<https://doi.org/10.1093/bioinformatics/btaa792>.

</div>

<div id="ref-gautreau2020" class="csl-entry">

Gautreau, Guillaume, Adelme Bazin, Mathieu Gachet, Rémi Planel, Laura
Burlot, Mathieu Dubois, Amandine Perrin, et al. 2020. “PPanGGOLiN:
Depicting Microbial Diversity via a Partitioned Pangenome Graph.” Edited
by Christos A. Ouzounis. *PLOS Computational Biology* 16 (3): e1007732.
<https://doi.org/10.1371/journal.pcbi.1007732>.

</div>

</div>
