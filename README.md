# ATAC-seq
***Pipeline of analysing ATAC-seq data***    
=============================================   

1.trim.data    # To cut adapt and fastqc your data    
`TrimGalore-0.5.0/trim_galore --fastqc --nextera --paired R1-3_HJNKHCCXY_L5_1.fq.gz R1-3_HJNKHCCXY_L5_2.fq.gz`    
</br> 
2.bowtie2.index     
`bowtie2-build ~/goat_pan/my_clean_data/bwa_mem/ASM.fa goat.bowtie2.index`    
</br>   
3.bowtie2.mapping   
`bowtie2 -x ../1.2.bowtie.index.goat/goat.bowtie2.index -X 2000 -1 ../01.cut-adapt/Rumen2_1_val_1.fq.gz -2 ../01.cut-adapt/Rumen2_2_val_2.fq.gz -S Rumen2.goat.sam`   
`samtools view -bS Rumen2.goat.sam >Rumen2.goat.bam`    
`samtools view -q 20 -u Rumen2.goat.bam >Rumen2.goat.uniq20.bam`  #Pick unique reads    
`samtools sort -o Rumen2.goat.uniq20.sort.bam -@ 8 -O bam Rumen2.goat.uniq20.bam`     
`samtools index Rumen2.goat.uniq20.sort.bam`    
</br> 
`java -Xmx50g -Djava.io.tmpdir=/stor9000/apps/users/NWSUAF/2016050001/tmp -jar ~/../2015060152/bin/picard-tools-2.1.1/picard.jar MarkDuplicates INPUT=Rumen2.goat.uniq20.sort.bam OUTPUT=Rumen2.goat.uniq20.sort.dedup.bam METRICS_FILE=Rumen2_dedup REMOVE_DUPLICATES=true CREATE_INDEX=true ASSUME_SORTED=true VALIDATION_STRINGENCY=LENIENT MAX_FILE_HANDLES=2000`   
</br> 
`samtools view -h Rumen2.goat.uniq20.sort.dedup.bam |awk '{if($3 != ''NC_005044.2''){print $0}}' |samtools view -Sb - >Rumen2.goat.uniq20.sort.dedup.noMT.bam`  #Remove mitochondria reads      
