# Extracting common chimeras

The ExtractCommonChimeras pipeline allows the extraction of chimeras common to several samples according to different selection criteria detailed below.


## Installation

### Requirements

* Perl5 distribution

* CracTools-core perl package. It will be automatically installed by cpanm along with all other CPAN dependancies.

### Install from sources


## Documentation

### DESCRIPTION

The extractCommonChimeras pipeline is a tool to extract common chimeras from a list of tsv files generated by `chimCT` (crac-tools). By default, it only considers redundant chimeras, i.e. found at least in 2 different samples, but the threshold can be modified.

#### ChimCT tsv output details

```
col 1  Id                 - A Uniq Id for each chimera (sample_name:chimeric_junction)
col 2  Name               - Fusion gene names (symbols) separated by three dashes (NAME1---NAME2 or N/A---N/A if unknown)
col 3  Chr1               - Chr number of chimera 5' part
col 4  Pos1               - Genomic position of chimera 5' part
col 5  Strand1            - Strand of chimera 5' part
col 6  Chr2               - Chr number of chimera 3' part
col 7  Pos2               - Genomic position of chimera 3' part
col 8  Strand2            - Strand of chimera 3' part
col 9  ChimValue          - Score that takes into account the methodology parameters + ambiguities (read mapping, read coverage)
col 10 Spanning junction  - Chimeric junction coverage (normalized per billion of reads/10^9)
col 11 Spanning PE        - PE reads coverage (normalized per billion of reads/10^9)
col 12 Class              - Chimeric class from 1 to 4.
	                          1: exons localized on 2 different chromosomes 
	                          2 : colinear exons but usually belonging to 2 different genes
	                          3 : exons on the same chromosome and same strand but not in the same order as on the genome
	                          4 : exons on the same chromosome but 2 different strands
col 13 Comments           - Chimeric reads characteristics
col 14 Others             - Chhimeras annotations
	                          key:value
	                          Nb_spanning_reads= nb of raw reads at the junction
	                          Nb_spanning_PE= nb of PE raw reads
```

#### schematic view of a chimeric RNA

![](chimct.png)

### USAGE

```

extractCommonChimeras.pl [-h] [-v] [--min-ranking <score>] [--threshold <min_redundancy>] [--min-cover <min_coverage>] [--summary <output_file>] [--output <output_file>] [--Rdata <output_file>] [--filter <filter_file>] <file1.csv> <file2.csv> ... <filen.csv>

Mandatory: at least two csv files (generated by the chimera pipeline).

 --output        filename to write the results (default: stdout)
 --min-ranking   minimal rank (score between 0 and 100) to consider the
                 chimera (50 by default)
 --threshold     minimal redundancy to consider a redundant chimera,
                 i.e. the min number of different samples where a same
                 chimera is found (2 by default)
 --min-cover     minimal span junction raw reads number (default: 1)
 --summary       make some statistics in the output_file if the argument
                 is defined
 --Rdata         format the statistics for R graphics in the output_file
                 if the argument is defined
 --filter        a file containing chimeras id (one name by line) to be
                 filtered-out (chimeras black list)
 -h --help       output this help and exit
 -v --version    output version information and exit

```

### OPTIONS

```
    -h, --help
    -v, --version
    --min-ranking=I<score>
    --threshold=I<min_redundancy>
    --min-cover=I<min_coverage>
    --summary=I<output_FILE>
    --Rdata=I<output_FILE>

```

### FILTERING

- delete chimeras when : 
	* comment ~ pseudogene
	* comment ~ anchored **and** absolute spanJ > 10  (artefact PCR)
	* comment ~ PairedEndChimeras=strange_paired_end_support
	* comment ~ GSNAPMapping (spicing events)
- delete chimeras corresponding to **HLA genes**
- delete chimeras corresponding to **N/A---N/A genes**
- delete chimeras with cover **> 3000** (superfamily genes)

### SUBCLASSES ANNOTATIONS

* Class 1 + exon1end<5 + exon2end<5 --> **exon_borders**
* Class 2 + [pos2-pos1] < 1000 kb --> **potential readthrough**
* Class 3:
	* FusionDistance = **Overlap**
    * FusionDistance = **short_distance(XX)**
* Class 4 + (pos2 - pos1) <= 0 --> **overlap**
    

### OUTPUT FILES

Descrition of the fields of the output file format. 
Each line corresponds to a redundant chimera identified and annotated by the chimeraPipeline. Chimeras are ordered by Score, Class and number of redundant chimeras.

```
col 1  Id                 - A Uniq Id for each chimera.
col 2  name               - Fusion gene names (symbols) separated by three dashes ('---')
col 3  samples            - List of samples containing this chimera.
col 4  class              - Chimeric class from 1 to 4.
col 5  subclass           - Subclass inference. Example: overlap, short_distance, etc.
col 6  mean_score         - Mean score of the chimera across <samples> (between 0 and 100).
                            The rank is based on confidence about chimera's positivity.
col 7  count              - It correponds to the number of different sample where the chimera is found.
col 8  comments           - Several comments about the chimera and the rank computed.
col 9  read_seq           - Sequence of a read containing this chimera junction.
col 10 pos_junction       - Position of the chimeric junction in the read.
col 11 profil_support     - Support profile
col 12 profil_location    - Location profile 

```

