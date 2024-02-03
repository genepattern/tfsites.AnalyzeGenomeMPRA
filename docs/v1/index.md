# tfsites.AnalyzeGwas v1

**Author(s):** Joe Solvason  

**Contact:** Joe Solvason (solvason@eng.ucsd.edu)

**Adapted as a GenePattern Module by:** Ted Liefeld (jliefeld@cloud.ucsd.edu)

**Task Type:** Transciption factor analysis

**LSID:**  urn:lsid:genepattern.org:module.analysis:00454


## Introduction

`analyzeGwas` examines mutations from genome-wide association studies (GWAS) data and reports the effect of all the mutations that are single-nucleotide variants (SNVs). Possible mutation effects include increasing (or optimizing) the affinity of a binding site, decreasing (or sub-optimizing) the affinity of a binding site, deleting a binding site, or creating a binding site. 
This analysis is performed on one transcription factor. For any input BED files, we will report whether overlap exists between the genomic intervals in each BED file and mutations from the GWAS data. 


## Methodology

For each SNV mutation in the input GWAS data, we determine its effect, if any, on any binding sites that exist in the reference genome. These are the possible effects of a SNV on a binding site: 
- `inc`
    - The affinity of the binding site increases
    - The affinity fold change from the reference binding site to the alternate binding site is greater than 1
- `dec`
    - The affinity of the binding site decreases
    - The affinity fold change from the reference binding site to the alternate binding site is less than 1
- `denovo`
    - A binding site is created
    - The reference binding site didn't follow the IUPAC definition, but the alternate binding site follows it 
- `del`
    - A binding site is deleted
    - The reference binding site followed the IUPAC definition, but the alternate binding site didn’t follow it
  
If an optimization threshold is provided by the user, then we report only the binding sites that have a fold change greater than or equal to the threshold. Similarly, if a sub-optimization threshold is provided, then we report only the binding sites that have a fold change less than or equal to the threshold. 

If input BED files are provided, the GWAS file is converted from TSV to BED format, so that each mutation in the GWAS file is denoted as a genomic interval. Bedtools intersect is used to search for overlap between the genomic intervals in the converted GWAS bed file and the intervals in each input BED file provided. For each input BED file, there is one column appended to the original GWAS file that indicates whether or not overlap exists.


## Parameters

<span style="color: red;">*</span> indicates required parameter

### Inputs and Outputs

- <span style="color: red;">*</span>**GWAS Data Input (.tsv)**
    - This file contains a list of mutations from a genome-wide association studies (GWAS) experiment, along with their genomic position. It can also contain other information, such as the statistical association between the mutation and phenotype. 
- <span style="color: red;">*</span>**Relative Affinity PBM Data (.tsv)**
    - This file is the normalized PBM data file for a transcription factor obtained from `defineTfSites`. It contains the relative affinity for every possible k-mer.
- **BED Genomic Interval Data (.bed)**
    - This file contains a list of genomic intervals of interest. 
- <span style="color: red;">*</span>**Reference Genome (.pkl)**
    - This file is the reference genome that corresponds to the genomic coordinates used in the input GWAS and BED files. 
- <span style="color: red;">*</span>**SNV Effects of GWAS Data (.tsv)**
    -  Name of the output file containing the original GWAS input file with appended columns containing information about the effect of each mutation.

 ### Other Parameters
    
- <span style="color: red;">*</span>**IUPAC Definition (string)**
    - IUPAC definition of core transcription factor binding site (see [here](https://www.bioinformatics.org/sms/iupac.html)).
- <span style="color: red;">*</span>**Zero Index Genomic Coordinates (boolean)**
    - If `True`, the genomic coordinates in the input GWAS file are 0-indexed (sequence numbering starts at 0). If `False`, they are 1-indexed (sequence numbering starts at 1).
- **SNV Effect (string)**
    - `default = None`
    - Specify one or more mutation types to analyze. SNV mutations can either increase (optimize) or decrease (sub-optimize) the affinity, delete a binding site, or create a binding site. Therefore, the possible mutation types are `inc`, `dec`, `denovo`, and `del`. This option also takes the value `all` if the user would like to analyze all of the listed mutation types.
- **Optimization Threshold (float)**
    - `default = 1`
    - Affinity fold-change threshold for affinity-increasing mutations. Only SNVs with fold change above this threshold will be reported. By default, all SNVs will be reported.
- **Sub-Optimization Threshold (float)**
    - `default = 1`
    - Affinity fold-change threshold for affinity-decreasing mutations. Only SNVs with fold change below this threshold will be reported. By default, all SNVs will be reported.


## Warnings Printed:
1. We report the percentage of mutations whose reference nucleotide did not match the existing nucleotide at its position in the genome. Using lifted over coordinates may lead to the incorrect genomic position. 


## Input Files

1. GWAS Data Input (.tsv)
- Can optionally contain other columns, but the five columns below must be listed first
- Columns:
    - `Chrom:` name of the chromosome
    - `Pos:` genomic coordinate indicating the position of the mutation
    - `Ref:` the wildtype sequence 
    - `Alt:` the mutated sequence
    - `P_value:` statistical association between the mutation and a phenotype

```
chrom    pos      ref  alt  p_value
chr1     1255143  C    T    1.43e-08
chr1     1258206  AT   A    1.24e-09
chr1     1259090  ACG  A    1.75e-08
chr1     1259093  T    A    1.05e-08
chr1     1259424  T    C    3.17e-09
```

2. Relative Affinity PBM Data (.tsv)
    - Columns
        - `Seq:` the sequence of every possible k-mer
        - `Rel_aff:` the relative affinity of the k-mer normalized to the max IUPAC k-mer

```
seq             rel_aff
AAAAAAAA        0.147
AAAAAAAC        0.107
AAAAAAAG        0.13
AAAAAAAT        0.125
AAAAAACA        0.123
```

       
## Output Files

1. SNV Effects of GWAS Data (.tsv)
- Columns
    - `Chrom:` name of the chromosome
    - `Pos-1idx:` position of the SNV, 1-indexed
    - `Ref:` reference nucleotide
    - `Alt:` alternate nucleotide
    - `P_value:` statistical association between genotype and phenotype
    - `Ref_kmer:` reference binding site
    - `Alt_kmer:` alternate binding site
    - `Site_direction:` direction of the binding site (+ if it follows the given IUPAC or - if it follows the reverse complement of the IUPAC)
    - `Ref_aff:` the affinity of the reference binding site
    - `Alt_aff:` the affinity of the alternate binding site
    - `Fold_change:` the ratio between ref_aff and alt_aff
    - `Mut_type:` the type of SNV effect

```
chrom    pos      ref  alt  p_value   ref_kmer  alt_kmer   site_direction  ref_aff  alt_aff   fold_change   mut_type

chr1     1255143  C    T    1.43e-08  GGCAAGG   GGGAAGG    -               NaN      0.120     NaN           denovo
chr1     1258206  AT   A    1.24e-09  TTGGAAAC  CTGGAAAC   -               0.097    0.118     1.216         inc
chr1     1259090  ACG  A    1.75e-08  GGGATTC   GAGGATTC   +               0.091    0.093     1.022         inc
chr1     1259093  T    A    1.05e-08  CGGTAACA  CGGGAACA   -               NaN      0.130     NaN           denovo
chr1     1259424  T    C    3.17e-09  GTGGAACT  GTGGAGCT   +               0.107    NaN       NaN           del
```
    
  
## Example Data

[Example input data is available on github](https://github.com/genepattern/tfsites.analyzeGwas/data)
    
    
## Version Comments

- **1.0.0** (2023-11-27): Initial draft of document scaffold.
- **1.0.1** (2024-02-02): Draft completed.
