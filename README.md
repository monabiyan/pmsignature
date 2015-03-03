# pmsignature

The R package *pmsignature* is developed 
for efficiently extracting characteristic mutation patterns (mutation signatures) 
from the set of mutations collected typically from cancer genome sequencing data.

For extracting mutation signatures, 
principal component analysis or nonnegative matrix factorization have been popular.
Compared to these existing approaches, the `pmsignature` has following advantages:
  
  
1. *pmsignature* can perform robust estimation of mutation signatures even taking account for many mutation features such as two bases 5' and 3' to the mutated sites.
2. *pmsignature* provides intuitively interetable visualization of mutation signatures, which is reminicent of sequencing logos.

Currently, *pmsignature* can only accept tab delimited text files with specialized format. 
We will improve the program so that it can accept VCF format files.

## Paper

> Shiraishi et al. Extraction of Latent Probabilistic Mutational Signature in Cancer Genomes, submitted.

## Input data

For input data, we need mutation feature data for each sample and mutation.
Here, **mutation features** are elements used for categorize the mutations such as: 
  
  * 6 substitutions (C>A, C>G, C>T, T>A, T>C and T>G)
* 5’ and 3’ flanking bases (A, C, G and T)
* transcription direction.

Currently, *pmsignature* can accept following two formats of tab-delimited text file.


### Mutation Position Format

|&nbsp;|&nbsp;|&nbsp;|&nbsp;|&nbsp;|
|---------|------|-----|---|---|
| sample1 | chr1 | 100 | A | C |
| sample1 | chr1 | 200 | A | T |
| sample1 | chr2 | 100 | G | T |
| sample2 | chr1 | 300 | T | C |
| sample3 | chr3 | 400 | T | C |
  
  * The 1st column shows the name of samples 
* The 2nd column shows the name of chromosome 
* The 3rd column shows the coordinate in the chromosome
* The 4th column shows the reference base (A, C, G, or T).
* The 5th colum shows the alternate base (A, C, G, or T).


### Mutation Feature Vector Format

|&nbsp;|&nbsp;|&nbsp;|&nbsp;|&nbsp;|&nbsp;|&nbsp;|
|----|---|---|---|---|---|---|
|  1 | 4 | 4 | 4 | 3 | 3 | 2 |
|  2 | 4 | 3 | 3 | 1 | 1 | 2 |
|  3 | 4 | 4 | 3 | 2 | 2 | 2 |
|  4 | 3 | 3 | 2 | 3 | 3 | 1 |
|  5 | 3 | 4 | 2 | 4 | 4 | 2 |
|  6 | 4 | 1 | 4 | 2 | 1 | 2 |
|  3 | 2 | 1 | 1 | 1 | 1 | 2 |
|  7 | 4 | 2 | 2 | 4 | 3 | 2 |
  
  * The 1st column shows the name of samples 
* From the 2nd to the last column show the value of mutation features.
* In this example, substitution patterns (1 to 6 values, C>A, C>G, C>T, T>A, T>C and T>G), two 5' and 3' flanking bases (1 to 4 values for each 4 site, A, C, G and T)
and transcription direction (1 to 2 values, + and -) are considered.


## Workflow

### Install the package

First, the R packages *VariantAnnotation* and *BSgenome.Hsapiens.UCSC.hg19*,
which *pmsignature* depends has to be installed.
Also, *devtools* may be necessary for ease of installation.

```
source("http://bioconductor.org/biocLite.R")
biocLite(c("VariantAnnotation", "BSgenome.Hsapiens.UCSC.hg19"))
install.packages("devtools")
```

The easiest way for installing *pmsignature* is to use the package *devtools*:
  
  ```
library(devtools)
devtools::install_github("friend1ws/pmsignature")
library(pmsignature)
```



### Prepare input data

First, create the input data from your mutation data.

After installing *pmsignature*,
you can find the above example file at the directory where pmsignature is installed.

* Mutation Position Format
```
inputFile <- system.file("extdata/Nik_Zainal_2012.mutationPositionFormat.txt", package="pmsignature");
print(inputFile);
```

* Mutation Feature Vector Format
```
inputFile <- system.file("extdata/Hoang_MFVF.ind.txt", package="pmsignature");
print(inputFile);
```


### Read input data

Type the following commands:
  
  * Mutation Position Format
```
G <- readMPFile(inputFile, numBases = 5);
```
Here, *inputFile* is the path for the input file. *numBases* is the number of flanking bases to consider including the central base (if you want to consider two 5' and 3' bases, then set 5).
You can format the data as the full model by typing 
```
G <- readMPFile(inputFile, numBases = 5, type = "full");
```
Also, you can add transcription direction information by typing 
```
G <- readMPFile(inputFile, numBases = 5, trDir = TRUE);
```

* Mutation Feature Vector Format
```
G <- readMFVFile(inputFile, numBases = 5, type="independent", trDir=TRUE)
```

### Estimate the parameters


When you want to set the number of mutation signature as 3, type the following command:
  
  ```
Param <- getPMSignature(G, K = 3);
```

If you want to add the background signature, then after obtaining the background probability, perform the estimation.
Currently, we only provide the background data for the "independent" and "full" model with 3 and 5 flanking bases.

```
BG_prob <- readBGFile(G);
Param <- getPMSignature(G, K = 3, BG = BG_prob);
```

In default, we repeat the estimation 10 times by changing the initial value,
and select the parameter with maximum likelihood.
If you want to changet the repeat number, then

```
Param <- getPMSignature(G, K = 3, numInit=20);
```


### Visualing the mutation signatures and memberships

You can check the mutation signature by typing

```
visPMSignature(Param, 1)
visPMSignature(Param, 2)
visPMSignature(Param, 3)
```