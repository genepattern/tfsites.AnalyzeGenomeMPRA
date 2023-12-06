# tfsites.AnalyzeGiwas v1

**Author(s):** Joe Solvason  

**Contact:** Joe Solvason (solvason@eng.ucsd.edu)

**Adapted as a GenePattern Module by:** Ted Liefeld (jliefeld@cloud.ucsd.edu)

**Task Type:** Transciption factor analysis

**LSID:**  urn:lsid:genepattern.org:module.analysis:00454


## Introduction

tfsites.AnalyzeGwas finds all effects of SNVs from GWAS data..


## Functionality

TBD

## Methodology

TBD

## Parameters

<span style="color: red;">*</span> indicates required parameter

- **input data**<span style="color: red;">*</span>
    - This is a [ state what the format and content is supposed to be] list of SNVs to be analyzed.
- **normpbm**
    - Normalized PBM created with the tfsites.defineTfSites module.
- **genome**
    - .Genome as pickled file (output from function tfsites.LoadGenome)
- **IUPAC**<span style="color: red;">*</span>
    - IUPAC DNA definition of the transcription factor site 
- **zero.position**<span style="color: red;">*</span>
    -  TRUE/FALSE, genomic coordinates are 0-indexed 
- **mutations**<span style="color: red;">*</span>
    - List of comma-seperated mutation effects to be analyzed (e.g. "inc,del,denovo") or "all".
- **out filename**<span style="color: red;">*</span>
    - Out file name for the annotated PBM data
- **bed file**
    - [Optional] Bed file to overlap with genomic coordinates in input file.
- ** upper threshold**
    - [Optional] Upper threshold for reporting affinity optimizing SNVs
- **bottom threshold**
    -[Optional] Bottom threshold for reporting affinity sub-optimizing SNVs'''


## Input Files

1.  input data.  Tab delimited file [ define format and contents in detail ] 
    
2. normpbm file. Normalized PBM created with the tfsites.defineTfSites module. [ define format and contents in detail  ]

3. genome file. Pickle file output from module tfsites.LoadGenome.

4. bed file. Bed file to overlap with genomic coordinates in input file. 

       
## Output Files

  1.PBM: <output prefix>.pbm.tsv.  Tab-separated text file TBD.
    e.g. 
```
seq     rel_aff
AAAAAAAA        0.147
AAAAAAAC        0.107
AAAAAAAG        0.13
AAAAAAAT        0.125
AAAAAACA        0.123

```
    
  
## Example Data

[Example input data is available on github](https://github.com/genepattern/tfsites.annotateTfSites/data)
    
## References

    
## Version Comments

- **1.0.0** (2023-11-27): Initial draft of document scaffold.
