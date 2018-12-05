# 4.2 Computational Analysis

RNA-DNA interactions are acquired from chimeric reads which contain RNA on one strand and DNA on the other. To understand the raw data right after sequencing, next we'll show the basic analysis pipeline. Both MARGI and GRID-seq use a pair-end sequencing strategy, so they follow a very similar pipeline. However, CHAR-seq uses single-end sequencing, so it differs a little in some details.

### Pipeline References
* MARGI portal: http://systemsbio.ucsd.edu/margi/
* CHAR-seq: https://github.com/straightlab/flypipe/
* GRID-seq: https://www.nature.com/articles/nbt.3968#methods

## Pipeline

### Trim Artificial Bases
Different linker designs may contain some artificial sequences. For the MARGI protocol, two random bases are added to the 5’ end that need to be removed and for GRID-seq this involves removal of 3 random bases. 

### Trim Adaptors
Removal of adapter sequences in a process called read trimming, or clipping, is one of the first steps in analyzing NGS data. Before mapping, one needs to trim the sequencing adaptors which were added for library preparation. These adaptors were used for holding barcoding sequences, forward/reverse primers (for paired-end sequencing) and the important binding sequences for immobilizing the fragments to the flowcell and allowing bridge-amplification. There are a lot of tools in the toolkit to remove adaptors; in MARGI portal FastX is used, in CHAR-seq Trimmomatic is used, and GRID-seq uses its own custom script.
