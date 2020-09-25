# hcontortus_amplicons





# MIPGEN
```bash
# working directory

mkdir -p /nfs/users/nfs_s/sd21/lustre118_link/hc/AMPLICONS/MIPs

# get refdata
ln -s /nfs/users/nfs_s/sd21/lustre118_link/hc/GENOME/REF/HAEM_V4_final.chr.fa REF.fa

ln -s /nfs/users/nfs_s/sd21/lustre118_link/hc/GENOME/TRANSCRIPTOME/TRANSCRIPTOME_CURATION/CURRENT/HCON_V4_curated_20200422_WBPS15.gff3 ANNOTATION.gff3

ln -s /nfs/users/nfs_s/sd21/lustre118_link/hc/XQTL/04_VARIANTS/PARENTS/XQTL_PARENTS.raw.snpeff.vcf.gz
ln -s /nfs/users/nfs_s/sd21/lustre118_link/hc/XQTL/04_VARIANTS/PARENTS/XQTL_PARENTS.raw.snpeff.vcf.gz.tbi


# make bwa index of REF.fa for MIPGEN

bsub.py 1 index_ref "bwa index REF.fa"


# get some targets - needs to be tab delimited file of:
# chromosome   start     stop gene_ID

# will try do this for one exon per gene across IVM region on chormosome 5

awk -F '[\t;]' '{if($1=="hcontortus_chr5_Celeg_TT_arrow_pilon" && $3=="CDS" && $4>=36000000 && $5<=38000000) print $1,$4,$5,$11}' OFS="\t" ANNOTATION.gff3 | sed 's/Parent=//g' > genes.bed

# check number of genes
cat genes.bed | cut -f4 | sort | uniq | wc -l
#> 190 genes

# filter out single exon genes and secondary transcripts.

cat genes.bed | cut -f4 | sort | uniq -c | awk '$2 ~ /-00001$/ && $1 > 1 {print $2}' > genes_to_keep.list

grep -f genes_to_keep.list genes.bed > genes.filtered.bed

cat genes.filtered.bed | cut -f4 | sort | uniq -c | wc -l
#> 152 genes



bsub.py 10 mipgen_test "/nfs/users/nfs_s/sd21/lustre118_link/software/AMPLICONS/MIPGEN/mipgen \
     -regions_to_scan genes.filtered.bed \
     -project_name hc_mipgene_test1 \
     -min_capture_size 150 \
     -max_capture_size 150 \
     -bwa_genome_index $PWD/REF.fa \
     -snp_file XQTL_PARENTS.raw.snpeff.vcf.gz"

```
