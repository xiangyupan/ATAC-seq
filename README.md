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
4.call.peak   #Broad or Narrow  
`python2.7 /stor9000/apps/users/NWSUAF/2013110098/bin/python27-package/MACS2-2.1.1.20160309/bin/macs2 callpeak -t /stor9000/apps/users/NWSUAF/2016050001/Chip_RNA/ATAC-seq/1.3.bowtie2.alignment.goat/R1-3.goat.sort.uniqe20.dedup.noMT.bam --shift -37 --extsize 73 -f BAM -g 2500000000 -n R1-3_broad_bedgraph -q 0.05 -B --outdir R1-3_broad_bedgraph/ --broad`    

`python2.7 /stor9000/apps/users/NWSUAF/2013110098/bin/python27-package/MACS2-2.1.1.20160309/bin/macs2 bdgcmp -t ./R1-3_broad_bedgraph/R1-3_broad_bedgraph_treat_pileup.bdg -c ./R1-3_broad_bedgraph/R1-3_broad_bedgraph_control_lambda.bdg -o ./R1-3_broad_bedgraph/R1-3_broad_atac_FE.bdg -m FE`   
</br> 
5.wig    
 `faSize ASM.fa > goat.size`    
`sort -k1,1 -k2,2n /stor9000/apps/users/NWSUAF/2015050469/ChIPAnalysis/sh6/SD2_pe-5/SD2_H3K27AC_FE.bdg > SD2_H3K27AC_FE.bdg.sort`   
`bedGraphToBigWig SD2_H3K27AC_FE.bdg.sort /stor9000/apps/users/NWSUAF/2015060145/genomic_align/phyloP/goat.sizes SD2_H3K27AC_FE.bw`  
OR  
`les Rumen2_peaks.narrowPeak |awk '{print $1"\t"$2"\t"$3"\t"$9}' > Rumen2_peaks.narrowPeak.bedgraph`    
`bedGraphToBigWig Rumen2_peaks.narrowPeak.bedgraph ~/RSCE_Comparative_genome/Repeat_masker/Genome_sizes/goat.whole.sizes Rumen2_peaks.narrowPeak.bdgraph.bw`    

`bigWigToWig in.bigWig out.wig`   
6.
