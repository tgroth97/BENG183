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

### Remove Duplicates
This step also follows the classic NGS analysis. The main purpose of removing duplicates is to mitigate the effects of PCR amplification bias introduced during library construction. Furthermore, one gains the computational benefit of reducing the number of reads to be processed in downstream steps. Tools for removing duplicates include Picard, Samtools and many others. In MARGI, fastuniq is used. In CHAR-seq, Super-Deduper is used. In GRID-seq it uses the barcode sequence and a custom script to identify duplicates.

### Separate DNA and RNA
One of the key steps in each different protocol is separating RNA and DNA within the chimeric sequence. For pair-end sequencing like MARGI and GRID-seq, the information has been identified though linker design, which naturally separates RNA and DNA information into two reads. However, with single-end sequencing (i.e. in CHAR-seq), this done by finding the bridge signal in the sequence.

CHAR-seq: RNA and DNA portions of each read are identified and split by a custom python script that finds the bridge, uses the bridge polarity to identify which side of the read is RNA or DNA, and verifies that there is only a copy of the bridge. Any reads that do not contain exactly one bridge sequence signal were removed. Only reads that contained sequence on both sides of the bridge (RNA and DNA) were processed further.

GRID-seq: To precisely remove the linker sequence from each read, MmeI motifs were used for defining linker boundaries. Linker orientation also dictated whether a given read at each end was originated from RNA or genomic DNA. raw reads clipped at MmeI motifs to produce paired DNA and RNA read mates ranging from 18 bp to 23 bp.

MARGI: The linker sequence was designed so that the DNA and RNA portions of the reads were on the appropriate sides according to a specfic primer of the linker. In the middle of the linker sequence is a BamHI restriction site that helps differentiate the two sides of the linker sequence. Using the restriction site and the primers that the RNA and DNA are associated, the reads can be separated into their according libraries.

### Map to the Reference
When mapping back the generated sequences, the RNA and DNA sequences must be dealt with separately. DNA can be mapped normally to the genome as would be done when working with Hi-C data. RNA on the other hand could have gone through post-translational modifications and is dervied from mRNA so must be mapped against the transcriptome. Due to these differences the tools mentioned have different work arounds for these. CHAR-seq and GRID-seq both use bowtie2 as their aligner. CHAR-seq focuses on forcing sense strandness to preserve RNA information. GRID-seq uses the -local flag with bowtie2 to map short reads which will return a large amount of mapping but loses out in specficity. MARGI uses STAR which is an aligner that is aware of splicing and can align to the transcriptome accordingly. 

![Figure1](https://github.com/tgroth97/BENG183/blob/master/mapping.png)

A brief diagram to show the general mapping process


### Generating RNA-DNA Contact Information
Generating and visualizing the RNA-DNA contact information has a very similar pipeline to that of Hi-C which generates data for DNA-DNA interactions. We need to begin with normalizing our data so that when we visualize it, the correct values are being displayed. The first step is to generate the observed contact matrix O[i,j] where every entry in the matrix contains the observed read counts between region i and region j. Next the correction matrix E[i,j] is made, where each entry contains the sum of corrections for the read pairs between entry i and j. Using these two matrices we can generate our final normalized matrix N[i,j] which is calculated N[i,j] = O[i,j]/E[i,j].

Once our data is normalized and contains accurate values, we are ready to visualize the data. The most common form of visualization for RNA-DNA contacts is a heat map where each bin N[i,j] is colored more red depending on how large the value is. Once the heatmap has been generated, it is very easy to identify possible topological domains (TADs). On the heatmap these TADs appear as red triangles but their exact beginning and end can be hard to decipher. Computationally the beginning and end of the TADs can be identified using a combination of a directionality index and a hidden markov model.

![Figure2](https://github.com/tgroth97/BENG183/blob/master/contact_matrix.png)

An example of what a RNA-DNA heatmap looks like compared to a DNA-DNA heatmap generated from Hi-C data.



Sources:

https://europepmc.org/articles/PMC5953555

https://cdn.elifesciences.org/articles/27024/elife-27024-v2.pdf

https://www.cell.com/current-biology/fulltext/S0960-9822(17)30011-8
