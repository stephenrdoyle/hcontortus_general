
# Commands used to run EvidenceModeller for the Haemonchus genome annotation




```bash

wget



# Step 1 - partition inputs

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

chmod a+x commands.list

./commands.list



# Step 4 - recombine partitions

EVidenceModeler-1.1.1/EvmUtils/recombine_EVM_partial_outputs.pl \
     --partitions partitions_list.out \
     --output_file_name evm.out


# Step 5 - make a GFF3

EVidenceModeler-1.1.1/EvmUtils/convert_EVM_outputs_to_GFF3.pl \
     --partitions partitions_list.out \
     --output evm.out \
     --genome HAEM_V4_final.chr.fa

```
