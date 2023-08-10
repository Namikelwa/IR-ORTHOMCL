# IR-ORTHOMCL
## Ortholog Detection with OrthoMCL 

### OrthoMCL is an algorithm and a set of tools that can be used for the identification of orthologous genes within a set of genomes. An overview of OrthoMCL is as follows.

![image](https://user-images.githubusercontent.com/85280529/259699946-5cdcd857-33ca-49f6-b217-b6fdaf940f83.png)

### The input to OrthoMCL is a set of genes, as protein sequences, belonging to a set of genomes. The genomes are processed by OrthoMCL, which produces, as output, a file listing which genes within which genomes are the most likely to be orthologs or paralogs.

![image](https://user-images.githubusercontent.com/85280529/259698156-fe1c0d5d-6391-41a3-992c-55adbca900e4.png)

### The first step involves performing BLAST search of every gene against every other gene. The reciprocal best matches between genes are found. These are sent through further processing and used to generate a graph of all genes linked up by their similarity scores. The graph is sent through the software, MCL (Markov Cluster Algorithm), which detects clusters of similar scoring genes within the graph.

![image](https://user-images.githubusercontent.com/85280529/259701824-5e81b87f-278d-4bb1-a7a7-56b1eee856b0.png)

### The clusters detected by MCL are printed out to a file and can be interpreted as the most likely sets of orthologs or paralogs.

### OrthoMCL is only one such set of tools to identify orthologs, with an alternative being OMA. This lab will focus on ortholog identification with OrthoMCL, and in particular the usage of the OrthoMCL Pipeline which will help automate running OrthoMCL.
