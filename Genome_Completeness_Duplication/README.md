Determining Percent Completeness and Duplication for New Genomes
================================================================
This method was used in “Comparative genomics of planktonic Flavobacteriaceae from the Gulf of Maine using metagenomic data” (Tully et al 2014, Microbiome).

By determining a core set of functions/genes for a group of related organisms, it is possible to determine percent completeness and duplication of a new genome for that group.

###Dependencies

* [Biopython](http://biopython.org/wiki/Download)

* [HMMER3](http://hmmer.janelia.org/)

* [TIGRFAM database - Version 14.0](ftp://ftp.jcvi.org/pub/data/TIGRFAMs/)

* [BLAST+](http://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download)

##Procedure
1.  Gather protein FASTA files of the desired group of related organisms - for simplicity, you may want to use only organisms with a single genomic scaffold or be sure to combine multiple scaffolds into a single file. Remove non-chromosomal scaffolds
	a. One possible way to create appropriate protein FASTA file would be to use FullGenbank_to_MultipleFASTA.py provided in this git
2.  Run each protein FASTA against the TIGRFAM 14.0 database using HMMER3
```
hmmscan -o <name>.out --cpu <#> --noali -E 0.00001 --tblout <name>.tbl /location/of/TIGRFAMs_14.0_HMM.LIB <input PROTEIN FAA>
```
3.  HMM_tbl_parse.py will parse through the TBL output format of a HMMER3 comparison between TIGRFAM and proteome of multiple genomes. The output will be a list of TIGRFAMs that represent the 'core' genome of the group and their representative counts.
	a. Be sure to execute HMM_tbl_parse.py in a directory with TBL output(s)
```
python HMM_tbl_parse.py
```
4.  Compare the proteins of new genome to TIGRFAM 14.0 using HMMER3 with the same parameters above
5.  HMM_percent_complete.py will use coregenome_tigrfam.txt generated by HMM_tbl_parse.py and the TBL format of the new genome to provide a estimate of the % completeness of the genome.
```
python HMM_percent_complete.py coregenome_tigrfam.txt <target>.tbl
```
6.  ConservedSingleCopyGenes.py will take a set of protein FASTA files and perform iterative BLASTP searches to determine the conserved single copy genes of a group.
	a. The % amino acid cutoff is user determined, but the % sequence alignment is fixed at 80%
	b. This must be executed in a folder with the target protein FASTA files in the format *.protein.faa
	c. A single FASTA file will be produced called singlecopygenes.faa
	d. It is recommended to determine which genome represent the "most central" genome of the group of interest. This can be be done by creating a phylogenetic tree using a marker gene from each genome.
```
python ConservedSingleCopyGenes.py <phylogenetically central genome>.faa <percent AAID cutoff>
```
7.  Estimate_Percent_Duplication.py uses singlecopygenes.faa generated using ConservedSingleCopyGenes.py to determine the % duplication of a new genome.
	a. Especially if the duplication is low, manual inspection will likely reveal that some or
all of the duplicate genes can be explained as the result of assembly errors/duplications.
```
python Estimate_Percent_Duplication.py <CSCGs | singlecopygenes.faa> <target genome>.faa
```