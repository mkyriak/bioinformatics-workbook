In this tutorial, we will learn about how to assemble the transcriptome from RNA-Seq reads and calculate differential expression.

Command to start Trinity


```
## Single-end Illumina
Trinity --seqType fq --max_memory 32G --CPU 16 --trimmomatic --normalize_by_read_set --output TrinityOut --single R1.gz &

##Paired-end Illumina or a combination of SE and PE
Trinity --seqType fq --max_memory 32G --CPU 16 --trimmomatic --normalize_by_read_set --output TrinityOut --left all_left_R1.gz --right all_right_R2.gz &

## To show all available options for running Trinity
Trinity --show_full_usage_info
```



--seqType Specify fq for fastq files or fa for fasta files
## FAQ
**
**Where is the output of the final assembly?
**What if I have a combination of single-end and paired-end sequencing data?
[https://github.com/trinityrnaseq/trinityrnaseq/wiki/Running-Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki/Running-Trinity)