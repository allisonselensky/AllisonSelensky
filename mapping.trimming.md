## mapping - do the reference before putting the files - make sure to put $OUT before the files and before .sam
```
tmux
> for i in *.lite.trim.1_1.fastq
> do
> OUT=${i%.lite.trim.1_1.fastq}
> bwa mem -t 10 /storehouse/visitor/reference/dmel-all-chromosome-r6.69.fasta $OUT.lite.trim.1_1.fastq $OUT.lite.trim.1_2.fastq > $OUT.sam
> done
```

## Trimming
```
tmux
for i in *.lite.1_1.fastq
do
OUT=${i%.lite.1_1.fastq}
fastp -i $OUT.lite.1_1.fastq etc. 
done
```
