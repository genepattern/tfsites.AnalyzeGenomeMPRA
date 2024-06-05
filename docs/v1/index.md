# tfsites.AnalyzeGenomeMPRA v1

**Author(s):** Joe Solvason  

**Contact:** Joe Solvason (solvason@eng.ucsd.edu)

**Adapted as a GenePattern Module by:** Ted Liefeld (jliefeld@cloud.ucsd.edu)

**Task Type:** Transciption factor analysis

**LSID:**  urn:lsid:genepattern.org:module.analysis:00454


## Introduction

`AnalyzeGenomeMPRA` examines single-nucleotide variants (SNVs) from massively parallel reporter assay (MPRA) data and reports their effects on transcription factor binding sites. Possible mutation effects include increasing (or optimizing) the affinity of a binding site, decreasing (or sub-optimizing) the affinity of a binding site, deleting a binding site, or creating a binding site. This analysis is performed on one transcription factor. If BED files are given as an input, it reports whether overlap exists between the genomic intervals in each BED file and the mutations from the MPRA data. 


## Methodology

For each SNV in the MPRA data, we determine its effect, if any, on any binding sites that exist in the reference genome. These are the possible effects of a SNV on a binding site: 
- `inc`
    - The affinity/score of the binding site increases
    - The affinity/score fold change from the reference binding site to the alternate binding site is greater than 1
- `dec`
    - The affinity/score of the binding site decreases
    - The affinity/score fold change from the reference binding site to the alternate binding site is less than 1
- `denovo`
    - A binding site is created
    - The reference binding site didn't follow the binding site definition, but the alternate binding site follows it 
- `del`
    - A binding site is deleted
    - The reference binding site followed the binding site definition, but the alternate binding site didn’t follow it
  
If an optimization threshold is provided by the user, then we report only the binding sites that have a fold change greater than or equal to the threshold. Similarly, if a sub-optimization threshold is provided, then we report only the binding sites that have a fold change less than or equal to the threshold. 

If input BED files are provided, the MPRA file is converted from TSV to BED format, so that each mutation in the MPRA file is denoted as a genomic interval. The tool `bedtools intersect` is used to search for overlap between the genomic intervals in the converted MPRA BED file and the intervals in each input BED file provided. For each input BED file, there is one column appended to the original MPRA file that indicates whether or not overlap exists.


## Parameters

<span style="color: red;">*</span> indicates required parameter

### Inputs and Outputs

- <span style="color: red;">*</span>**MPRA data (.tsv)**
    - File containing a list of mutations from an MPRA experiment, along with their genomic position. It can also contain other information, such as the statistical association between the mutation and phenotype. 
- <span style="color: red;">*</span>**PBM or PFM reference data (.tsv)**
    - File containing the normalized PBM or PFM data file for a transcription factor obtained from `DefineTfSites`. It provides the relative affinity/score for every possible k-mer of a length k.
- **BED genomic interval data (.bed)**
    - File containing a list of genomic intervals of interest. 
- <span style="color: red;">*</span>**reference genome (.pkl)**
    - File containing the reference genome that corresponds to the genomic coordinates used in the input MPRA and BED files. 
- <span style="color: red;">*</span>**SNV effects of MPRA data output filename(.tsv)**
    -  Name of the output file containing the original MPRA input file with appended columns containing information about the effect of each SNV.

 ### Other Parameters
    
- <span style="color: red;">*</span>**binding site definition (string)**
    - IUPAC definition of the core transcription factor binding site (see [here](https://www.bioinformatics.org/sms/iupac.html)).
- <span style="color: red;">*</span>**zero-indexed genomic coordinates (boolean)**
    - If `True`, the genomic coordinates in the input MPRA file are 0-indexed (sequence numbering starts at 0). If `False`, they are 1-indexed (sequence numbering starts at 1). 
- **SNV effects to report (string)**
    - `Default = all`
    - Specify one or more mutation types to analyze. Mutations can either increase (optimize) or decrease (sub-optimize) the affinity, delete a binding site, or create a binding site. Therefore, the possible mutation types are `inc`, `dec`, `denovo`, and `del`. By default, this option takes the value `all` if the user would like to analyze all of the listed mutation types.
- **optimization threshold (float)**
    - `Default = 1`
    - Fold change threshold for mutations that increase the affinity/score. Only SNVs with fold change above this threshold will be reported. By default, all SNVs will be reported.
- **sub-optimization threshold (float)**
    - `Default = 1`
    - Fold change threshold for mutations that decrease the affinity/score. Only SNVs with fold change below this threshold will be reported. By default, all SNVs will be reported.


## Warnings Printed:
1. We report the percentage of mutations whose reference nucleotide did not match the existing nucleotide at its position in the genome. Using lifted over coordinates may lead to the incorrect genomic position. 


## Input Files

1. MPRA data (.tsv)
- Can optionally contain other columns, but the first 4 columns below must be listed first
- Columns:
    - `Chromosome:` name of the chromosome
    - `Position:` genomic position of the SNV
    - `Ref:` reference nucleotide
    - `Alt:` alternate nucleotide
    - `P-value:` statistical association between the mutation and a phenotype

```
Chromosome   Position     Ref     Alt    P-value
chr1         1255143      C       T      1.43e-08
chr1         1258205      T       A      1.24e-09
chr1         1259091      C       A      1.75e-08
chr1         1259093      T       A      1.05e-08
chr1         1259424      T       C      3.17e-09
```

2. PBM or PFM reference data (.tsv)
- Can provide more than one file 
- Columns
    - `PBM Kmer:` the sequence of every possible k-mer
    - `PBM Relative Affinity:` the relative affinity of the k-mer normalized to the to the k-mer with the highest MFI

```
PBM Kmer     PBM Relative Affinity
AAAAAAAA     0.15
AAAAAAAC     0.11
AAAAAAAG     0.13
AAAAAAAT     0.13
AAAAAACA     0.12
```

3. BED genomic interval data (.tsv)
- Can provide more than one file 
- Columns
    - `Chromosome:` name of chromosome
    - `Start:` start coordinate of genomic interval
    - `End:` end coordinate of genomic interval

file1
```
Chromosome   Start       End   
chr1         1281500    1281700
chr1	     1259080	 1259099
chr1	     1259410	 1259420
chr1	     1281630	 1281645
chr1	     1351740	 1351750
chr1         1309990     1310000
```

file2
```
Chromosome    Start      End
chr1          1281252    1281389
chr1          1307300    1307450
chr1          1309980    1310000
chr1          1281630    1281645
chr1          1337700    1337950
chr2          158364697  158365864
```
       
## Output Files

1. SNV effects of MPRA data (.tsv)
- Columns
    - `Chromosome:` name of the chromosome
    - `Position:` position of the SNV
    - `Ref:` reference nucleotide
    - `Alt:` alternate nucleotide
    - `P-value:` statistical association between genotype and phenotype
    - `Overlap-{name of BED file #1}:` whether overlap exists between the MPRA data and BED file #1
    - `Overlap-{name of BED file #2}:` whether overlap exists between the MPRA data and BED file #2
    - `Reference Kmer:` reference binding site
    - `Alternate Kmer:` alternate binding site
    - `Site Direction:` direction of the binding site (+ if it follows the given binding site definition or - if it follows the reverse complement of the binding site definition)
    - `Reference Affinity/Score:` the affinity of the reference binding site
    - `Alternate Affinity/Score:` the affinity of the alternate binding site
    - `Fold Change:` the ratio between ref_aff and alt_aff
    - `SNV Effect:` the type of SNV effect

```
Chromosome    Position    Ref   Alt    P-value    Overlap-file1.bed    Overlap-file2.bed    Reference Kmer  Alternate Kmer   Site Direction   Reference Affinity   Alternate Affinity   Fold Change    Mutation Type
chr1          1281643     G     C      1.53e-09   1                    0                    GGCAAGG         GGGAAGG          -                NaN                  0.120                NaN            denovo          
chr1          1307327     A     G      3.93e-09   0                    1                    TTGGAAAC        CTGGAAAC         -                0.097                0.118                1.216          inc
chr1          1309994     G     A      8.83e-11   1                    1                    GGGATTC         GAGGATTC         +                0.091                0.093                1.022          inc
chr1          1325256     A     C      5.08e-10   0                    0                    CGGTAACA        CGGGAACA         -                NaN                  0.130                NaN            denovo
chr1          1337900     A     G      3.52e-12   0                    1                    GTGGAACT        GTGGAGCT         +                0.107                NaN                  NaN            del
```
    
  
## Example Data

[Example input data is available on github](https://github.com/genepattern/tfsites.analyzeGenomeMPRA/data)
    
    
## Version Comments

- **1.0.0** (2023-11-27): Initial draft of document scaffold.
- **1.0.1** (2024-02-02): Draft completed.
