---
title: "RNAseq analysis methods"
output: 
  html_notebook:
      toc: true
      toc_float: true
---

## Part2

1.	Open R (I use R studio, you can also use the command line) 
2.	Install edgeR. 

```{r, eval=FALSE}
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("edgeR")
```

3.	Load the edgeR package. We will use edgeR to identify significantly differentially expressed genes between our two groups based on counts of gene expression for each gene. 

```{r, results='hide'}
library(edgeR)
```

Set working directory
```{r}
setwd("/Users/jlkelley/OneDriveWSU/OneDrive - Washington State University (email.wsu.edu)/ConGen2019/analyses")
```

Read in gene count matrix 
```{r}
counts = read.csv("gene_count_matrix.csv", header = T, row = 1)
```

Look at the beginning of the file (note, there are lots of zeros)
```{r}
head(counts)
```

Make sure the number of genes and samples is expected
```{r}
dim(counts)
```


Create map of samples to treatments 
```{r}
samples = colnames(counts)
group = c(rep("A", 10),rep("B",9))
sample.info = data.frame(cbind(samples,group))
head(sample.info)
```

Remove genes with zero counts

```{r}
length(which(rowSums(counts) == 0))
gene.counts = counts[rowSums(counts) != 0,]
```

### Determine the number of genes left after removing zeros
```{r}
dim(gene.counts)
```


### Create a DGEList of the data
```{r}
y <- DGEList(counts=gene.counts,group=sample.info$group)

```

Calculate the normalization factors (from the edgeR manual: The calcNormFactors function normalizes for RNA composition by finding a set of scaling factors for the library sizes that minimize the log-fold changes between the samples for most genes. The default method for computing these scale factors uses a trimmed mean of M-values (TMM) between each pair of samples [30]. We call the product of the original library size and the scaling factor the effective lib)
```{r}
y <- calcNormFactors(y)
```

Look at y
```{r}
y
```

## Visualize the data in y, for example using an MDS plot
```{r}
plotMDS(y)
```

Design matrix
```{r}
design <- model.matrix(~group)
design
```

Estimate dispersion 
```{r}
y <- estimateDisp(y,design) 
```

perform quasi-likelihood F-tests:
```{r}
fit <- glmQLFit(y,design)
qlf <- glmQLFTest(fit,coef=2)
topTags(qlf)
```

summarize the results
```{r}
summary(de <- decideTestsDGE(qlf, p.value = 0.05))
```

### Plot the results

```{r}
detags <- rownames(y)[as.logical(de)]
plotSmear(qlf, de.tags=detags)
abline(h = c(-1, 1), col = "blue")
```
