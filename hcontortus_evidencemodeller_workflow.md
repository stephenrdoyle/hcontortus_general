
## Commands used to run EvidenceModeller for the Haemonchus genome annotation
- basic workflow for running EvidenceModeller,
- a little cleaned up to remove Sanger-specfic info, eg. we use bsub for job submission, so I have removed this, but have included the rough run time and requirements that were used.
- I didn't write it up as a proper workflow at the time as I was naively working through it step by step, so not very pretty, but did work ok
- the output of this was sent back into PASA for a second round update as recommended
- let me know if any of it is not clear, or if you get stuck


```bash
# get the input data from ftp
wget ftp://ngs.sanger.ac.uk/production/pathogens/sd21/hcontortus_genome/evidencemodeller_troubleshooting/*

# input files:
#--- AUGUSTUS_JN_CURATED.exonerate.renamed.gff - a set of ~110 manually curated genes
#--- braker.renamed.gff3 - braker output
#--- ce_2_v4.exonerate.renamed.gff3 - C.elegans proteins mapped to H. contortus genome using exonerate
#--- exonerate.V1_2_V4.renamed.gff - H. contortus V1 proteins mapped to H. contortus genome using exonerate
#--- HAEM_V4_final.chr.fa - genome
#--- pasa.renamed.gff3 - output of PASA containing IsoSeq predicted models
#--- weights.txt - differential weighting scheme for each piece of evidence



# Step 1 - partition inputs
#- ~1.5h, < 1 Gb mem

EVidenceModeler-1.1.1/EvmUtils/partition_EVM_inputs.pl \
     --genome HAEM_V4_final.chr.fa \
     --gene_predictions braker.renamed.gff3 \
     --transcript_alignments pasa.renamed.gff3 \
     --protein_alignments exonerate.V1_2_V4.renamed.gff \
     --protein_alignments ce_2_v4.exonerate.renamed.gff3 \
     --protein_alignments AUGUSTUS_JN_CURATED.exonerate.renamed.gff \
     --segmentSize 100000 \
     --overlapSize 10000 \
     --partition_listing partitions_list.out


# Step 2 - write commands
#- < 1 min, < 100 Mb mem

EVidenceModeler-1.1.1/EvmUtils/write_EVM_commands.pl \
     --weights weights.txt \
     --genome HAEM_V4_final.chr.fa \
     --gene_predictions braker.renamed.gff3 \
     --transcript_alignments pasa.renamed.gff3 \
     --protein_alignments exonerate.V1_2_V4.renamed.gff \
     --protein_alignments ce_2_v4.exonerate.renamed.gff3 \
     --protein_alignments AUGUSTUS_JN_CURATED.exonerate.renamed.gff \
     --output_file_name evm.out \
     --partitions partitions_list.out > commands.list



# Step 3 - run EVM
#- ~ 4h, < 100 Mb mem

chmod a+x commands.list
./commands.list



# Step 4 - recombine partitions
#- ~ 5 min, < 100 Mb mem

EVidenceModeler-1.1.1/EvmUtils/recombine_EVM_partial_outputs.pl \
     --partitions partitions_list.out \
     --output_file_name evm.out


# Step 5 - make a GFF3
#- < 1 min, < 100 Mb mem

EVidenceModeler-1.1.1/EvmUtils/convert_EVM_outputs_to_GFF3.pl \
     --partitions partitions_list.out \
     --output evm.out \
     --genome HAEM_V4_final.chr.fa
```


## Reloading PASA with evidence modeller output, and iteratively reloading isoseq data as per manual
```bash
# step 1: load annotations
PASApipeline-pasa-v2.2.0/scripts/Load_Current_Gene_Annotations.dbi -c alignAssembly.config -g HAEM_V4_final.chr.fa -P HC_V4_evm.gff


# step 2: compare - note used both HQ and LQ isoseq data in reload
PASApipeline-pasa-v2.2.0/scripts/Launch_PASA_pipeline.pl -c annotCompare.config -A -g HAEM_V4_final.chr.fa -t all_isoseq.renamed.fasta --CPU 7
#> this produced a GFF called "sd21_pasa_HcV4_2.gene_structures_post_PASA_updates.6815.gff3"

# step 3: load_pasaR1_annotations - see filename with new suffix 6815
PASApipeline-pasa-v2.2.0/scripts/Load_Current_Gene_Annotations.dbi -c alignAssembly.config -g HAEM_V4_final.chr.fa -P sd21_pasa_HcV4_2.gene_structures_post_PASA_updates.6815.gff3


# step 4: pasa_compare_2_reloaded_annotations
PASApipeline-pasa-v2.2.0/scripts/Launch_PASA_pipeline.pl -c annotCompare.config -A -g HAEM_V4_final.chr.fa -t all_isoseq.renamed.fasta --CPU 7

#> this produced a GFF called "sd21_pasa_HcV4_2.gene_structures_post_PASA_updates.31804.gff3" which I made some modifications (renamed gene ids etc ) / cleaned up / sorted to produce "HCON_V4.renamed.gff3".


```
