## getting into the class portal
```
ssh visitor@134.129.113.23
cd /storehouse/visitor -> /table_1 -> /pigmentation

root-> home -> visitor
    ->storehouse -> visitor
```

## Trim adapters and quality check reads 
```
for i in *.lite.1-1.fastq
do
OUT=${i%.lite.1_1.fastq}
fastp -i $OUT.lite.1_1.fastq -o $OUT.lite.trim.1_1.fastq -O $OUT.lite.trim.1_2.fastq
done
```

## BWA INDEX THE REFERENCE
```
bwa index bbc.fasta
bwa mem -t 10 bbc.fasta SRR31835375.lite.trim.1_1.fastq SRR31835375.lite.trim.1_2.fastq > SRR31835375.sam (10 is # of threads, which files we want, putting those files in the .sam file)
```

## expand a variable without any whitespace to its right
```
foo=abc
echo foo is $foo ($ is calling for that variables value)
foo is abc (what it will say after pressing enter)
$ echo foo is ${foo}EFG
foo is abcEFG (what it says after pressing enter)
```

## use variables when you need to use files with different endings for the bwa command
```
for i in *.trim.1_1.fastq
do
OUT=${i%.lite.trim.1_1.fastq}
echo $OUT
done
```

## use samtools to convert to a more compressed file type called bam, then we sort the reads by location
```
samtools view -b file.sam -o file.bam (The -b flag tells the program to output as a bam file)
samtools sort file.bam -o file.sorted.bam
```

## Remember the steps we took to process our file:
```
fastp
BWA index
BWA mem
samtools view
samtools sort
```
