# Guide to prepare the uploaded files for scSVAS online applications

## Summary of input files
The following table lists scSVAS applications' input files and demo datasets. `<file>` and `[file]` refers to compulsory and optional input file, respectively. Aberrations: Triple-negative breast cancer (TNBC); Acute myeloid leukaemia (AML); High grade serous ovarian cancer (HGSOC); Prostate cancer (PC); Lung cancer (LC).

|scSVAS Applications| Input Files | Demo Datasets |
|--|--|--|
|CNV View| <cnv.csv>, [cut.json], [meta.csv] | TNBC_T10,T16 [1,2], 10x_COLO829[3] |
|CNV Heatmap | <cnv.csv>, [cut.json], [meta.csv], [gene_cnv.csv], [target_region.bed] | TNBC_T10,T16 [1,2], 10x_COLO829[3]|
|Cell Phylogeny | <cut.json>, [meta.csv], [gene_cnv.csv] | TNBC_T10,T16 [1,2], 10x_COLO829[3]|
|Ploidy Stairstep | <cnv.csv>, [meta.csv] | TNBC_T10,T16 [1,2], 10x_COLO829[3]|
|Ploidy Distribution | <cnv.csv>, [meta.csv] |TNBC_T10,T16 [1,2], 10x_COLO829[3]|
|Embedding Map | <meta.csv>, [gene_cnv.csv] |TNBC_T10,T16 [1,2], 10x_COLO829[3]|
|Time Lineage | <clonal_edges.csv], <clonal_prev.csv> | AML [4,5], HGSOC_P7 [5,6]|
|Space Lineage | <clonal_edges.csv>, <clonal_prev.csv>, [space_edges.csv], [image.png] | HGSOC_P1,P7 [5,6], PC_A21 [5,7] |
|Space Prevalence | <clonal_edges.csv>, <clonal_prev.csv>, <space_edges.csv> | HGSOC_P1,P7 [5,6], PC_A21 [5,7] |
|Clonal Lineage | <evo.json>, [target_anno.tsv] | TNBC_T10,T16 [1,2], 10x_COLO829[3]|
|Recurrent Event | <recurrent.json>, [recurrent.tsv], [target_anno.tsv], [sample_meta.csv] | LC |


## Run scSVAS offline scripts 


### `scSVAS.py`

To get the uploaded files for applications CNV View, CNV Heatmap, Cell Phylogeny, Ploidy Stairstep, Ploidy Distribution, and Clonal Lineage, run `scSVAS.py` with the following command: 
```
python3 scSVAS.py cnv --cnv_fn=IN_FILE [--meta_fn=IN_FILE] [--nwk_fn=IN_FILE] [--target_gene_fn=IN_FILE] [--k=INT] [--out_prefix=STR] [--ref=STR]
    
Options:
    -h --help                   Show this screen.
    --version                   Show version.
    --cnv_fn=IN_FILE            Path of SCYN format cnv file. <required>
    --meta_fn=IN_FILE           Path of SCYN format meta file. [optional]
    --nwk_fn=IN_FILE            Path of build tree, scSVAS will build one if not supplied. [optional]
    --target_gene_fn=IN_FILE    Path of SCYN format meta file. [optional]
    --k=INT                     Number of clusters in the tree at the cut point. [optional, default: 50]
    --out_prefix=STR            Path of out file prefix, [optional, default: ./phylo]
    --ref=STR                   Reference version, [optional, default: hg38]
    
Outputs:
    scSVAS.nwk                  Stores the dendrogram tree build by hierarchy clustering.
    meta_cssvas.csv             Stores the clustering and embedding results for all single cells.
    cut.json                    Stores cut-dendrogram for all single cells.
    evo.tsv                     Stores the detail of clonal lineages result in tsv format.
    evo.json                    Stores the detail of clonal lineages result in json format.
    hcluster_cnv.csv            Stores the averaging copy number profiles of hcluster groups.
    gene_cnv.csv                Stores  the copy number profiles of detected driver gene.
```

### `recurrent.py`

To get the uploaded files for applications Recurrent Event, run `recurrent.py` with the following command: 
```
recurrent.py cnv --cnv_fns=IN_FILES --samples=STR [--target_gene_fn=IN_FILE] [--out_prefix=STR] [--ref=STR]
recurrent.py -h | --help

Options:
    -h --help                   Show this screen.
    --version                   Show version.
    --cnv_fns=IN_FILES          List of SCYN format cnv file path, seperated by comma ',' <required>
    --samples=STR               List of sample names, seperated by comma ',' <required>
    --target_gene_fn=IN_FILE    Path of SCYN format meta file. [optional]
    --out_prefix=STR            Path of out file prefix, [optional, default: ./recurrent]
    --ref=STR                   Reference version, [optional, default: hg38]
    
Outputs:
    recurrent.tsv               Stores the detail of detected recurrent events in tsv format.
    recurrent.json              Stores the detail of detected recurrent events in json format.
```

## Input files for scSVAS scripts

### `cnv.csv` 

`cnv.csv` stores the copy number profiles of cells, with cell as row and bin region as column. The first column should be the cell IDs, and the first row should be bin region, e.g. `chr1:1-512000`. User can directly use the `cnv.csv` file generated by SCYN. Please note that the column bin regions are recommended to be equal size for best heatmap visualization effect.

<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_cnv_csv.png" style="width:50%" class="center">

*Screenshot of demo file T10_cnv.csv*

### `meta.csv`

`meta.csv` stores the meta information of cells, with cell as row, and meta information as column. User can directly use the `meta.csv` file generated by SCYN, or manually one. The following are examples of meta fields:

 + `c_gini`: stores the gini coeficient of each cell.
 + `c_ploidy`: stores the mean ploidy of each cell, it is calculated from `prefix_cnv.csv` (not the one SCOPE provide).

After running `scSVAS.py`, user can get `prefix_meta_scvar.csv` with additional meta fields:

Prefix `c` here denotes numeric continuous value. The absence of prefix `c` denotes category meta information like 'group' or 'cluster'. Prefix `e` refers to embedding and dimension reduction. User can manually add extra cell meta information like 'cell_type' and 'group' for downstream analysis:

 + `cell_type`: predefined cell types.
 + `group`: predefined cell groups.

<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_meta_csv.png" style="width:50%" class="center">

*Screenshot of demo file T10_meta_scsvas.csv*

### `nwk_fn`

`nwk_fn` is the standard [newick format](https://en.wikipedia.org/wiki/Newick_format) for the phylogeny tree of single cells. You can use any other tools to build the tree. scSVAS will build one if not supplied. For 10x data, user can be obtained the `nwk_fn` by running `process_10x_h5.py`.

<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_nwk.png" style="width:50%" class="center">

*Screenshot of demo file demo1.nwk*

### `target_gene_fn`

`target_gene_fn` is a list of genes file the user interested in to present in the Cell Phylogeny.

<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_target_gene_txt.png" style="width:50%" class="center">

*Screenshot of demo file demo_target_gene.txt*

## Output files for scSVAS.py

### `scSVAS.nwk`

`scSVAS.nwk` stores the dendrogram tree build by hierarchy clustering in [newick format](https://en.wikipedia.org/wiki/Newick_format). In dendrogram, the leaf nodes are single cells.

### `cut.json`

`cut.json` stores cut-dendrogram for all nodes.

```
   "node_name": {
      "dist_to_root": number,    
      "parent": string,  // parent node name
      "newick": string,  // cutted dendrogram in newick format
      "leafs": list of string  // list of cell names included in current node
   }
```
<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_cut_json.png" style="width:50%" class="center">

*Screenshot of demo file T10_cut5.json*

### `meta_scsvas.csv`

After running `scSVAS.py`, user can get `meta_scsvas.csv` with single cell as row, and following meta field as column:

 + `hcluster`: the hierachy clustering result.
 + `e_PC1`: the first principle component of cells after PCA.
 + `e_PC2`: the second principle component of cells after PCA.
 + `e_TSNE1`: the first dimension of cells after T-distributed Stochastic Neighbor Embedding (t-SNE).
 + `e_TSNE2`: the second dimension of cells after T-distributed Stochastic Neighbor Embedding (t-SNE).
 + `e_UMAP1`: the first dimension of cells after Uniform Manifold Approximation and Projection (UMAP).
 + `e_UMAP2`: the second dimension of cells after Uniform Manifold Approximation and Projection (UMAP).

Prefix `c` denotes numeric continuous value. The absence of prefix `c` denotes category meta information like `group` or `cluster`. Prefix `e` refers to embedding/dimension reduction methods.

<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_meta_csv.png" style="width:50%" class="center">

*Screenshot of demo file T10_meta_scsvas.csv*

### `evo.tsv`

`evo.tsv` stores the detail of clonal lineages results. The file header are listed as follow:

 + `category_label`: categorical meta label, e.g. `hcluster`
 + `parent`: parent node name, e.g. `c1`
 + `child`: child node namem, e.g. `c2`
 + `amp/loss`: amplification or deletion, e.g.  `amp`
 + `region`: bin region, e.g. `chr10:10240001-15360000`
 + `cytoband`: cytoband, e.g. `10p13`
 + `parent_cnv`: CN of parent node in specified region, e.g. `1.93` 
 + `child_cnv`: CN of child node in specified region, e.g. `3.42`
 + `shift`: The CN changes between `parent_cnv` and `child_cnv`, e.g. `1.48`
 + `gene`: genes in specified region, only genes in `target_gene_fn` and gene set will be shown, 
   e.g. `PNISR,CCNC,MMS22L,ASCC3`

<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_evo_tsv.png" style="width:50%" class="center">

*Screenshot of demo file T10_evo.tsv*

### `evo.json`

`evo.json` stores the all information needed for "Clonal Lineage'' application. It includes lineage tree structure and all information of `evo.tsv`.
<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_evo_json.png" style="width:50%" class="center">

*Screenshot of demo file T10_evo.json*


### `hcluster_cnv.csv`

`hcluster_cnv.csv` stores the averaging copy number profiles of hcluster groups, with hcluster group as row and bin region as column. The first column stores the hcluster group IDs, and the first row stores bin region, e.g. `chr1:1-512000`. The bin regions are the same with input file `cnv.csv`.

<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_hcluster_cnv_csv.png" style="width:50%" class="center">

*Screenshot of demo file T10_hcluster_cnv.csv*


### `gene_cnv.csv`

`gene_cnv.csv` stores  the copy number profiles of detected driver gene, with single cell as row and driver gene as column. The first column stores the single cell IDs, and the first row stores driver gene names.

<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_gene_cnv_csv.png" style="width:50%" class="center">

*Screenshot of demo file T10_gene_cnv.csv*



## Output files for recurrent.py

### `recurrent.tsv` 

File `recurrent.tsv` stores the detail of detected recurrent events. The file header are listed as follow:

+ `sample_subclone`: categorical meta label, e.g. `LC009T_c4`
+ `amp/del`: amplification or deletion, e.g.  `del`
+ `region`: bin region, e.g. `chr10:10240001-15360000`
+ `cytoband`: cytoband, e.g. `10p13`
+ `cnv`: CN of current sample subclone in specified region, e.g. `3.42`
+ `shift`: The CN change of the `sample_subclone` compared to diploid cell, e.g. `1.48`
+ `gene`: genes in specified region, only genes in `target_gene_fn` and gene set will be shown, e.g. `PNISR,CCNC,MMS22L,ASCC3`

<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_recurrent_tsv.png" style="width:50%" class="center">
*Screenshot of demo file LC_recurrent.tsv*
  

### `recurrent.json` 

`recurrent.json` stores the all information needed for "CNV Recurrent Event'' application. It includes all information of `recurrent.tsv`.
<img src="https://raw.githubusercontent.com/paprikachan/scSVAS/master/webserver/fig/demo_recurrent_json.png" style="width:50%" class="center">

*Screenshot of demo file LC_recurrent.json*


## User customized scSVAS input files

### `target_gene_sets.csv`

File `target_gene_sets.csv` specified the molecular signatures database (MSigDB) gene sets and user self-defined gene sets for annotation. A demo file is shown in below:

```
MSigDB,KEGG_NON_SMALL_CELL_LUNG_CANCER,KEGG_SMALL_CELL_LUNG_CANCER
self-defined gene set 1,NOTCH2,TP53,ALK,SOX2
self-defined gene set 2,NOTCH2,TP53
```` 
The first line with starts with `MSigDB`, then follows with names of gene sets defined in MsigDB,  e.g. `KEGG_NON_SMALL_CELL_LUNG_CANCER`. Next, each subsequent line stores a customized gene set. The gene set starts with the name of customized gene set, e.g. `self-defined gene set 1`, then follows with the name of genes in customized gene set, e.g. `NOTCH2,TP53,ALK,SOX2`.

### `clonal_edges.csv`

The format of file `clonal_edges.csv` is the same with input format TimeScape, like in demo file 1, except we can accept extra edge distance like in demo file 2.

```
# demo file 1
source,target
n1,n2
n1,n3
n3,n4
n4,n5
```

```
# demo file 2
source,target,dist
n1,n2,1.5
n1,n3,1.5
n3,n4,1
n4,n5,2
````

The file headers are as follow:

 + `source`: The source/parent node of an edge/branch, e.g. `n1`.
 + `target`: The target/child node of an edge/branch, e.g. `n2`.
 +  `dist`: *Optional*, the distance of an edge/branch. e.g. `1.5`.

### `clonal_prev.csv`

The format of file `clonal_prev.csv` is the same with input format TimeScape, such as the following demo file:

```
timepoint,clone_id,clonal_prev
Diagnosis,n1,0.1274
Diagnosis,n2,0.5312
Diagnosis,n3,0.2904
Diagnosis,n4,0.0510
Relapse,n5,1.00
```

The file headers are as follows:

 + `timepoint`: The timepoint in the history of tumor evolution, e.g. `Diagnosis`
 + `clone_id`: The node ID in the history of clonal evolution, e.g. `n1`
 + `clonal_prev`: The node prevalence of one subclone at specified timepoint, e.g. `0.1274`.

### `target_reigon.bed`

File `target_reigon.bed` stores user interested local genome regions in BED format. Please check the demo file as follow:

```
chrom   start   end
chr2    140924620       145924620
chr3    140924620       145924620
```

## Reference

[1] Navin, N., Kendall, J., Troge, J., Andrews, P., Rodgers, L., McIndoo, J., Cook, K., Stepansky, A., Levy, D., Esposito, D., Muthuswamy, L., Krasnitz, A., McCombie, W. R., Hicks, J., & Wigler, M. (2011). Tumour evolution inferred by single-cell sequencing. *Nature*, 472(7341), 90???94. https://doi.org/10.1038/nature09807

[2] Feng, X., Chen, L., Qing, Y., Li, R., Li, C., & Li, S. C. (2020). SCYN: Single cell CNV profiling method using dynamic programming. *bioRxiv*. https://doi.org/10.1101/2020.03.27.011353

[3] Velazquez-Villarreal, E. I., Maheshwari, S., Sorenson, J., Fiddes, I. T., Kumar, V., Yin, Y., ... & Craig, D. W. (2020). Single-cell sequencing of genomic DNA resolves sub-clonal heterogeneity in a melanoma cell line. *Communications biology*, 3(1), 1-8. https://www.nature.com/articles/s42003-020-1044-8

[4] Ding, L., Ley, T. J., Larson, D. E., Miller, C. A., Koboldt, D. C., Welch, J. S., Ritchey, J. K., Young, M. A., Lamprecht, T., McLellan, M. D., McMichael, J. F., Wallis, J. W., Lu, C., Shen, D., Harris, C. C., Dooling, D. J., Fulton, R. S., Fulton, L. L., Chen, K., Schmidt, H., ??? DiPersio, J. F. (2012). Clonal evolution in relapsed acute myeloid leukaemia revealed by whole-genome sequencing. *Nature*, 481(7382), 506???510. https://doi.org/10.1038/nature10738

[5] Smith, M. A., Nielsen, C. B., Chan, F. C., McPherson, A., Roth, A., Farahani, H., Machev, D., Steif, A., & Shah, S. P. (2017). E-scape: interactive visualization of single-cell phylogenetics and cancer evolution. *Nature methods*, 14(6), 549???550. https://doi.org/10.1038/nmeth.4303

[6] McPherson, A., Roth, A., Laks, E., Masud, T., Bashashati, A., Zhang, A. W., Ha, G., Biele, J., Yap, D., Wan, A., Prentice, L. M., Khattra, J., Smith, M. A., Nielsen, C. B., Mullaly, S. C., Kalloger, S., Karnezis, A., Shumansky, K., Siu, C., Rosner, J., ??? Shah, S. P. (2016). Divergent modes of clonal spread and intraperitoneal mixing in high-grade serous ovarian cancer. *Nature genetics*, 48(7), 758???767. https://doi.org/10.1038/ng.3573

[7] Gundem, G., Van Loo, P., Kremeyer, B., Alexandrov, L. B., Tubio, J., Papaemmanuil, E., Brewer, D. S., Kallio, H., H??gn??s, G., Annala, M., Kivinummi, K., Goody, V., Latimer, C., O'Meara, S., Dawson, K. J., Isaacs, W., Emmert-Buck, M. R., Nykter, M., Foster, C., Kote-Jarai, Z., ??? Bova, G. S. (2015). The evolutionary history of lethal metastatic prostate cancer. *Nature*, 520(7547), 353???357. https://doi.org/10.1038/nature14347

