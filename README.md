# ReporTree

<p align="center">
  <img width="300" height="190" src=https://user-images.githubusercontent.com/19263468/154529278-2447e132-c903-46ec-86ae-467c5c72b6c8.png>
</p>

**Genomics-informed pathogen surveillance** strengthens public health decision-making, thus playing an important role in infectious diseases’ prevention and control. A pivotal outcome of genomics surveillance is the **identification of pathogen genetic clusters/lineages and their characterization in terms of geotemporal spread or linkage to clinical and demographic data**. This task usually relies on the visual exploration of (large) phylogenetic trees (e.g. Minimum Spanning Trees (MST) for bacteria or rooted SNP-based trees for viruses). As this may be a non-trivial, non-reproducible and time consuming task, we developed **ReporTree, a flexible pipeline that facilitates the detection of genetic clusters and their linkage to epidemiological data**. 


_ReporTree can help you to:_      
- obtain **genetic clusters at all partition level(s)** of a newick tree (e.g. SNP-scaled tree) or an MST derived from cg/wgMLST allele matrix
- obtain **summary reports with the statistics/trends** (e.g. timespan, location range, cluster/group size and composition, age distribution etc.) for the derived genetic clusters or for any other provided grouping variable (e.g. clade, lineage, ST, vaccination status, etc.)
- obtain **count/frequency matrices** for the derived genetic clusters or for any other provided grouping variable
- identify regions of **cluster stability** (i.e. cg/wgMLST partition/threshold ranges in which cluster composition is similar)

In summary, ReporTree facilitates and accelerates the production of surveillance-oriented reports, thus contributing to a sustainable and efficient public health genomics-informed pathogen surveillance.

_Note: this tool relies on the usage of programs/modules of other developers. DO NOT FORGET TO ALSO CITE THEM!_


## Implementation

ReporTree is implemented in python 3.6 and comprises four modules available in standalone mode that are orchestrated by _reportree.py_ (see details in [ReporTree wiki](https://github.com/insapathogenomics/ReporTree/wiki/1.-Implementation#reportree-modular-organization)).


## Input files

Metadata table in .tsv format (column should not have blank spaces)           
**AND (optionally)**        

Newick tree which will be used to obtain genetic clusters       
**OR**      
Allele matrix which will be used to obtain genetic clusters from a MST      
**OR**     
Partitions table (i.e. matrix with genetic clusters) in .tsv format (columns should not have blank spaces)      


## Main output files

- metadata_w_partitions.tsv - initial metadata information with additional columns comprising information on the genetic clusters at different partitions

_TIP: Users can interactively visualize and explore the ReporTree derived clusters by uploading this metadata_w_partitions.tsv table together with either the original newick tree (e.g. rooted SNP-scaled tree) at [auspice.us](https://auspice.us) or the allele profile matrix (cg/wgMLST data) using [GrapeTree](https://github.com/achtman-lab/GrapeTree). With these tools your dataset is visualised client-side in the browser._

- partitions_summary.tsv - summary report with the statistics/trends (e.g. timespan, location range, cluster/group size and composition, age distribution etc.) for the derived genetic clusters present in partitions.tsv (note: singletons are not reported in this file but indicated in metadata_w_partitions.tsv)
- variable_summary.tsv - summary report with the statistics/trends (e.g. timespan, location range, cluster/group size and composition, age distribution etc.) for any (and as many) grouping variable present in metadata_w_partitions.tsv (such as, clade, lineage, ST, vaccination status, etc.)
- partitions.tsv - genetic clusters obtained for each user-selected partition threshold (only generated when a newick file or an allele matrix is provided)
- freq_matrix.tsv - frequencies of grouping variable present in metadata_w_partitions.tsv (e.g. lineage, ST, etc.) across another grouping variable (e.g. iso_week, country, etc.)
- count_matrix.tsv - counts of a grouping variable present in metadata_w_partitions.tsv (e.g. lineage, ST, etc.) across another grouping variable (e.g. iso_week, country, etc.)
- metrics.tsv - metrics resulting from the cluster congruence analysis, with indication of the Adjusted Wallace and the Ajusted Rand coefficients for each comparison of subsequent partitions, and the Simpson's Index of Diversity for each partition.
- stableRegions.tsv - partition ranges for which Adjusted Wallace coefficient is higher than the cut-off defined by the user (useful to study cluster stability and infer possible nomenclature) 


## Installation and dependencies

Dependencies:
- [TreeCluster](https://github.com/niemasd/TreeCluster) v1.0.3
- [A modified version of GrapeTree](https://github.com/insapathogenomics/GrapeTree)
- [Pandas](https://pandas.pydata.org)
- [Ete3](http://etetoolkit.org)

Installation:
```bash
conda create -n reportree -c etetoolkit -c anaconda -c bioconda ete3 scikit-learn pandas grapetree=2.1 treecluster=1.0.3 python=3.6
git clone https://github.com/insapathogenomics/ReporTree
cd ReporTree/scripts/
git clone https://github.com/insapathogenomics/GrapeTree
git clone https://github.com/insapathogenomics/ComparingPartitions
```


## Usage

```bash
  -h, --help            show this help message and exit

ReporTree:
  ReporTree input/output file specifications

  -m METADATA, --metadata METADATA
                        [MANDATORY] Metadata file in .tsv format
  -t TREE, --tree TREE  [OPTIONAL] Input tree
  -a ALLELE_PROFILE, --allele-profile ALLELE_PROFILE
                        [OPTIONAL] Input allele profile matrix
  -p PARTITIONS, --partitions PARTITIONS
                        [OPTIONAL] Partitions file in .tsv format -
                        'partition' represents the threshold at which
                        clustering information was obtained
  -out OUTPUT, --output OUTPUT
                        [OPTIONAL] Tag for output file name (default =
                        ReporTree)
  --list                [OPTIONAL] If after your command line you specify this
                        option, ReporTree will list all the possible columns
                        that you can use as input in '--
                        columns_summary_report'. NOTE!! The objective of this
                        argument is to help you with the input of '--
                        columns_summary_report'. So, it will not run
                        reportree.py main functions!!

Partition minimum unit:
  Minimum unit/distance between partition thresholds

  -d DIST, --dist DIST  Distance unit by which partition thresholds will be
                        multiplied (example: if -d 10 and -thr 5,8,10-30, the
                        minimum spanning tree will be cut at
                        50,80,100,110,120,...,300. If -d 10 and --method-
                        threshold avg_clade-2, the avg_clade threshold will be
                        set at 20). This argument is particularly useful for
                        non- SNP-scaled trees. Currently, the default is 1,
                        which is equivalent to 1 allele distance or 1 SNP
                        distance. [1.0]

Partitioning with TreeCluster:
  Specifications to cut the tree with TreeCluster [only if a tree file is provided]

  --method-threshold METHOD_THRESHOLD
                        List of TreeCluster methods and thresholds to include
                        in the analysis (comma-separated). To get clustering
                        at all possible thresholds for a given method, write
                        the method name (e.g. root_dist). To get clustering at
                        a specific threshold, indicate the threshold with a
                        hyphen (e.g. root_dist-10). To get clustering at a
                        specific range, indicate the range with a hyphen (e.g.
                        root_dist-2-10). Default: root_dist,avg_clade-1 (List
                        of possible methods: avg_clade, leaf_dist_max,
                        leaf_dist_min, length, length_clade, max, max_clade,
                        root_dist, single_linkage, single_linkage_cut,
                        single_linkage_union) Warning!! So far, ReporTree was
                        only tested with avg_clade and root_dist!
  --support SUPPORT     [OPTIONAL: see TreeCluster github for details] Branch
                        support threshold
  --root-dist-by-node   [OPTIONAL] Set only if you WANT to cut the tree with
                        root_dist method at each tree node distance to the
                        root (similar to root_dist at all levels but just for
                        informative distances)

Partitioning with GrapeTree:
  Specifications to get and cut minimum spanning trees derived from cg/wgMLST allele data [only if an allele profile file is provided]

  --method GRAPETREE_METHOD
                        "MSTreeV2" [DEFAULT] Alternative:"MSTree"
  --missing HANDLER     ONLY FOR MSTree. 0: [DEFAULT] ignore missing data in
                        pairwise comparison. 1: remove column with missing
                        data. 2: treat missing data as an allele. 3: use
                        absolute number of allelic differences.
  --wgMLST              [EXPERIMENTAL: see GrapeTree github for details] a
                        better support of wgMLST schemes
  --n_proc NUMBER_OF_PROCESSES
                        Number of CPU processes in parallel use. [5]
  -thr THRESHOLD, --threshold THRESHOLD
                        Partition threshold for clustering definition.
                        Different thresholds can be comma-separated (e.g.
                        5,8,16). Ranges can be specified with a hyphen (e.g.
                        5,8,10-20). If this option is not set, the script will
                        perform clustering for all the values in the range 1
                        to max
  --subset              Reconstruct the minimum spanning tree using only the
                        samples that correspond to the filters specified at
                        the '--filter' argument
  --matrix-4-grapetree  Output an additional allele profile matrix with the
                        header ready for GrapeTree visualization. Set only if
                        you WANT the file

ReporTree metadata report:
  Specific parameters to report clustering/grouping information associated to metadata

  --columns_summary_report COLUMNS_SUMMARY_REPORT
                        Columns (i.e. variables of metadata) to get statistics
                        for the derived genetic clusters or for other grouping
                        variables defined in --metadata2report (comma-
                        separated). If the name of the column is provided, the
                        different observations and the respective percentage
                        are reported. If 'n_column' is specified, the number
                        of the different observations is reported. For
                        example, if 'n_country' and 'country' are specified,
                        the summary will report the number of countries and
                        their distribution (percentage) per cluster/group.
                        Exception: if a 'date' column is in the metadata, it
                        can also report first_seq_date, last_seq_date,
                        timespan_days. Check '--list' argument for some help.
                        Default = n_sequence,lineage,n_country,country,n_regio
                        n,first_seq_date,last_seq_date,timespan_days [the
                        order of the list will be the order of the columns in
                        the report]
  --partitions2report PARTITIONS2REPORT
                        Columns of the partitions table to include in a joint
                        report (comma-separated). Other alternatives: 'all' ==
                        all partitions; 'stability_regions' == first partition
                        of each stability region as determined by
                        comparing_partitions_v2.py. Note: 'stability_regions'
                        can only be inferred when partitioning TreeCluster or
                        GrapeTree is run for all possible thresholds or when a
                        similar partitions table is provided (i.e. sequential
                        partitions obtained with the same clustering method)
                        [all]. Check '--list' argument for some help
  --metadata2report METADATA2REPORT
                        Columns of the metadata table for which a separated
                        summary report must be provided (comma-separated)
  -f FILTER_COLUMN, --filter FILTER_COLUMN
                        [OPTIONAL] Filter for metadata columns to select the
                        samples to analyze. This must be specified within
                        quotation marks in the following format 'column<
                        >operation< >condition' (e.g. 'country == Portugal').
                        When more than one condition is specified for a given
                        column, they must be separated with commas (e.g
                        'country == Portugal,Spain,France'). When filters
                        include more than one column, they must be separated
                        with semicolon (e.g. 'country ==
                        Portugal,Spain,France;date > 2018-01-01;date <
                        2022-01-01'). White spaces are important in this
                        argument, so, do not leave spaces before and after
                        commas/semicolons.
  --frequency-matrix FREQUENCY_MATRIX
                        [OPTIONAL] Metadata column names for which a frequency
                        matrix will be generated. This must be specified
                        within quotation marks in the following format
                        'variable1,variable2'. Variable1 is the variable for
                        which frequencies will be calculated (e.g. for
                        'lineage,iso_week' the matrix reports the percentage
                        of samples that correspond to each lineage per
                        iso_week). If you want more than one matrix you can
                        separate the different requests with semicolon (e.g.
                        'lineage,iso_week;country,lineage'). If you want a
                        higher detail in your variable2 and decompose it into
                        two columns you use a colon (e.g.
                        lineage,country:iso_week will report the percentage of
                        samples that correspond to each lineage per iso_week
                        in each country)
  --count-matrix COUNT_MATRIX
                        [OPTIONAL] Same as '--frequency-matrix' but outputs
                        counts and not frequencies
  --mx-transpose        [OPTIONAL] Set ONLY if you want that the variable1
                        specified in '--frequency-matrix' or in '--count-
                        matrix' corresponds to the matrix first column.

Stability regions:
  Congruence analysis of cluster composition at all possible partitions to determine regions of cluster stability (automatically run if you set --partitions2report 'stability_regions'). WARNING! This option is planned to handle sequential partitions obtained with the same clustering method, such as a partitions table derived from cg/wgMLST data (from 1 to max allele threshold). Use it at your own risk, if you provide your own partitions table.

  -AdjW ADJUSTEDWALLACE, --AdjustedWallace ADJUSTEDWALLACE
                        Threshold of Adjusted Wallace score to consider an
                        observation for method stability analysis [0.99]
  -n N_OBS, --n_obs N_OBS
                        Minimum number of sequencial observations that pass
                        the Adjusted Wallace score to be considered a
                        'stability region' (i.e. a threshold range in which
                        cluster composition is similar) [5]
  -o ORDER, --order ORDER
                        [Set only if you provide your own partitions table]
                        Partitions order in the partitions table (0: min ->
                        max; 1: max -> min) [0]
  --keep-redundants     Set ONLY if you want to keep all samples of each
                        cluster of the most discriminatory partition (by
                        default redundant samples are removed to avoid the
                        influence of cluster size in the identification of
                        stability regions)
```


#### Note on the '--method-threshold' argument

ReporTree uses [TreeCluster](https://github.com/niemasd/TreeCluster) to obtain clustering information from a SNP-distance tree. To provide some flexibility, it uses the argument '--method-threshold' to run this program for all the combinations of method-threshold that the user needs:
- Clustering at all possible thresholds of a single method (from 1 SNP to the maximum distance of the tree) -> set only the method (e.g. root_dist)
- Clustering at a specific threshold for a single method -> set the method and use a hyphen to indicate the threshold (e.g. root_dist-10)
- Clustering at all possible thresholds of a range for a single method -> set the method and use a hyphen to indicate the threshold range (e.g. root_dist-10-30)
- Clustering using two or more methods and/or different thresholds -> set the different requirements separated by a comma (e.g. root_dist,avg_dist-10,avg_dist-20-30)


#### Note on the '--partitions2report' argument

This argument is used to select the columns of the partitions table that will be incorporated into the metadata table. If you use ReporTree to obtain the partitions table, the column names specification follows the same rules as the '--method-threshold' or the '--threshold' argument, depending on whether you provided a newick tree or an allele matrix, respectively.


#### Note on the columns for summary reports

This argument is used to select the columns that will be provided in the summary report.
-	To take the most profit of ReporTree, we recommend that you include the column 'date' in your metadata. This column must follow the format YYYY-MM-DD. If you only provide YYYY or YYYY-MM, it will assume YYYY-01-01!
-	If a 'date' column is provided in the metadata, ReporTree will determine and provide in the new metadata table the columns:
    - iso_year
    - iso_week_nr 
    - iso_week
-	While for nominal or categorical variables ReporTree can provide in the summary report the number of observations (‘n_column’) or the frequency of each observation, for the 'date' column this script can provide:
    - first_seq_date
    - last_seq_date
    - timespan_days
-	The columns of the summary reports are defined by the ‘--columns_summary_report’ argument. To know the columns that you can include based on your metadata table and the outputs of ReporTree, write the full command line and add the argument ‘--list’ in the end. This will give you a list of the possible columns that your summary reports can include, i.e. that you can request in ‘--columns_summary_report’.


#### Note on the '--frequency-matrix' and '--count-matrix' arguments

These arguments take as input two variables separated by a comma (variable1,variable2). Frequencies will be calculated for variable1 and by default the different observations of this variable (e.g. different lineages if variable 1 == 'lineage') will correspond to different columns, while the observations of variable 2 will correspond to different rows. To transpose this you can use '--mx-transpose'.

_TIP: If you want, you can split variable 2 in up to two variables. To this end you can indicate them separated by colon (e.g. lineage,country:iso_week)_


### How to run ReporTree with a newick tree as input?

```bash
reportree.py -m metadata.tsv -t tree.nw -out output -d dist --method-threshold method1,method2-threshold --root-dist-by-node --columns_summary_report columns,summary,report --partitions2report all --metadata2report column1,column2 -f ‘column1 == observation;date > year-mm-dd’ --frequency-matrix variable1,variable2 --count-matrix variable1,variable2
```


### How to run ReporTree with an allele matrix as input?

```bash
reportree.py -m metadata.tsv -a allele_matrix.tsv -out output -d dist --method MSTreeV2 -thr 5,8,15 --matrix-4-grapetree --columns_summary_report columns,summary,report --partitions2report all --metadata2report column1,column2 -f ‘column1 == observation;date > year-mm-dd’ --frequency-matrix variable1,variable2 --count-matrix variable1,variable2 -AdjW adjusted_wallace -n n
```


## Examples

### Routine surveillance - viral pathogen (e.g. SARS-CoV-2) - [click here](https://github.com/insapathogenomics/ReporTree/wiki/4.-Examples#routine-surveillance---viral-pathogen-eg-sars-cov-2)

ReporTree is currently applied to generate weekly reports about SARS-CoV-2 variant circulation in Portugal (https://insaflu.insa.pt/covid19/). In [ReporTree wiki](https://github.com/insapathogenomics/ReporTree/wiki/4.-Examples#routine-surveillance---viral-pathogen-eg-sars-cov-2), we give some examples on how to rapidly generate key surveillance metrics taking as input metadata tables (tsv format) and rooted divergence (SNP) trees (newick format) provided for download in regular Nextstrain (auspice) builds, such as those maintained by the National Institute of Health Dr. Ricardo Jorge, Portugal (INSA) at https://insaflu.insa.pt/covid19/.


### Outbreak detection - bacterial foodborne pathogen (e.g. _Listeria monocytogenes_) - [click here](https://github.com/insapathogenomics/ReporTree/wiki/4.-Examples#outbreak-detection---bacterial-foodborne-pathogen-eg-listeria-monocytogenes)

ReporTree can facilitate the routine surveillance and outbreak investigation of bacterial pathogens, such as foodborne pathogens. In [ReporTree wiki](https://github.com/insapathogenomics/ReporTree/wiki/4.-Examples#outbreak-detection---bacterial-foodborne-pathogen-eg-listeria-monocytogenes), we provide a simple example of the usage of ReporTree to rapidly identify and characterize potential Listeriosis outbreaks. With a single command, ReporTree builds a MST from cgMLST data and **automatically extracts genetic clusters at three high resolution levels (<5, <8, <15 allelic differences)**, and provides comprehensive reports about the sample collection (e.g. ST sequence count/frequency per year, etc).  


### Large-scale genetic clustering and linkage to antibiotic resistance data (e.g. _Neisseria gonorrhoeae_) - [click here](https://github.com/insapathogenomics/ReporTree/wiki/4.-Examples#large-scale-genetic-clustering-and-linkage-to-antibiotic-resistance-data-eg-neisseria-gonorrhoeae)

ReporTree can enhance genomics surveillance and quickly identify/characterize genetic clusters from large datasets. In [ReporTree wiki](https://github.com/insapathogenomics/ReporTree/wiki/4.-Examples#large-scale-genetic-clustering-and-linkage-to-antibiotic-resistance-data-eg-neisseria-gonorrhoeae), with a single command line, we reproduce part of the extensive genomics analysis performed by [Pinto et al., 2021](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8208699/pdf/mgen-7-481.pdf) over 3,791 _N. gonorrhoeae_ genomes from isolates collected across Europe.


## Citation

If you run ReporTree, please do not forget to cite this repository.

Also, ReporTree relies on the work of other developers. So, depending on the functionalities you use, there are other tools that you must cite:     
- Ete3: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4868116/pdf/msw046.pdf (all tools)     
- Grapetree: http://www.genome.org/cgi/doi/10.1101/gr.232397.117 (if you provided an allele matrix)      
- TreeCluster: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6705769/pdf/pone.0221068.pdf (if you provided a newick tree)      
- ComparingPartitions: https://journals.asm.org/doi/10.1128/jcm.02536-05?permanently=true (if you requested "stability_regions")      
