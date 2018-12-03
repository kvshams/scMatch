# scMatch: a single-cell gene expression profile annotation tool using reference datasets

Single-cell RNA sequencing (scRNA-seq) measures gene expression at the resolution of individual cells. Massively multiplexed single-cell profiling has enabled large-scale transcriptional analyses of thousands of cells in complex tissues. In most cases, the true identity of individual cells is unknown and needs to be inferred from the transcriptomic data. Existing methods typically cluster (group) cells based on similarities of their gene expression profiles and assign the same identity to all cells within each cluster using the averaged expression levels. However, scRNA-seq experiments typically produce low-coverage sequencing data for each cell, which hinders the clustering process. We introduce scMatch, which directly annotates single cells by identifying their closest match in large reference datasets. We used this strategy to annotate various single-cell datasets and evaluated the impacts of sequencing depth, similarity metric and reference datasets. We found that scMatch can rapidly and robustly annotate single cells with comparable accuracy to another recent cell annotation tool (SingleR), but that it is quicker and can handle considerably larger reference datasets. We demonstrate how scMatch can handle large customized reference gene expression profiles that combine data from multiple sources, thus empowering researchers to identify cell populations in any complex tissue with the desired precision.

scMatch is maintained by Rui Hou [rui.hou@research.uwa.edu.au]

## Download and Installation
```bat
   git clone https://github.com/asrhou/scMatch.git
```
A truncated FANTOM5 reference dataset can be downloaded from https://github.com/asrhou/scMatch/tree/master/refDB/FANTOM5. The compressed files need to be decompressed before being used as the reference database. We also merged it with reference datasets from [SingleR](https://www.biorxiv.org/content/early/2018/03/22/284604), which can be downloaded from https://figshare.com/s/efd2969ce20fae5c118f.

This tool provides command line utilities only for now.

## Command Line Utilities

This tool can annotate the given transcriptome data with sample names or ontology terms. It can analyze data from multiple platforms. The required user inputs are a raw count matrix file in CSV format, or an mtx file and two tsv files generated by CellRanger.

### scMatch: Annotate the given transcriptome data using human and/or mouse expression data from given reference dataset.

```
scMatch.py [-h] [--refType REFTYPE] [--testType TESTTYPE] --refDS REFDS
                  [--dFormat DFORMAT] --testDS TESTDS
                  [--testMethod TESTMETHOD] [--testGenes TESTGENES]
                  [--keepZeros KEEPZEROS] [--coreNum CORENUM]

optional arguments:
  -h, --help            show this help message and exit
  --refType REFTYPE     human (default) | mouse | both
  --testType TESTTYPE   human (default) | mouse | rat | chimpanzee |zebrafish
  --refDS REFDS         path to the folder of reference dataset(s)
  --dFormat DFORMAT     10x (default) | csv
  --testDS TESTDS       path to the folder of test dataset if dtype is 10x,
                        otherwise, the path to the file
  --testMethod TESTMETHOD
                        s[pearman] (default) | p[earson] | both
  --testGenes TESTGENES
                        optional, path to the csv file whose first row is the
                        genes used to calculate the correlation coefficient
  --keepZeros KEEPZEROS
                        y[es] (default) | n[o]
  --coreNum CORENUM     number of the cores to use, default=1

```

The raw output, generated by scMatch is an excel file showing the correlation coefficient of each reference sample with test transcriptome. 

To annotate a scRNA-seq dataset 'GSE81861_Cell_Line_COUNT.csv' from [Li *et al.*, (Nature Genetics, 2017)](https://www.nature.com/articles/ng.3818), which were sequenced by SMARTer Ultra-Low RNA Kit and can be found at [NCBI GEO:GSE81861](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE81861), use the following code
```bat
   python scMatch.py --refDS refDB/FANTOM5 --dFormat csv --testDS GSE81861_Cell_Line_COUNT.csv
```
A snippet of the output for a test transcriptome looks like this:

| sample name | Spearman correlation coefficient |
| --- | --- |
|lung adenocarcinoma cell line:A549.CNhs11275.10499-107C4 | 0.583425488 |
|bile duct carcinoma cell line:TFK-1.CNhs11265.10496-107C1 | 0.551511709 |
|hepatoma cell line:Li-7.CNhs11271.10484-107A7 | 0.545313045 |

### toTerms: Transfer the original sample name annotation vectors to the vectors of ontology terms.

```
toTerms.py [-h] --splF SPLF --refDS REFDS [--coreNum CORENUM]

optional arguments:
  -h, --help         show this help message and exit
  --splF SPLF        path to the original sample annotation folder which
                     contains original sample annotation data
  --refDS REFDS      path to the folder of reference dataset(s)
  --coreNum CORENUM  number of the cores to use, default is 1
```

The output of toTerms includes an excel file showing the averaged correlation coefficient of each ontology terms with test transcriptome and a csv file showing the most related ontology term for each test cell.

To transfer the annotation results of 'GSE81861_Cell_Line_COUNT.csv' to the vectors of ontology terms, use the following command
```bat
   python toTerms.py --splF GSE81861_Cell_Line_COUNT/annotation_result_keep_all_genes --refDS FANTOM5
```
A snippet of the output for a test transcriptome looks like this:

| Ont Term | Avg Score |
| --- | --- |
| DOID:3910 ! lung adenocarcinoma | 0.54626792 |
| DOID:4897 ! bile duct carcinoma | 0.538693164 |
| DOID:4556 ! lung large cell carcinoma | 0.532440558 |
