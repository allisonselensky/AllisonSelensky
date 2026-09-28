## pipe command - It allows you to direct data to a new command without writing it into a file.
'''
cat bbc.fasta | sed '1d'
'''

## trying to get bcftools running without delving into what is next, but our system is outdated, Conda is an environment manager
'''
conda activate bio_env
bcftools
'''

## We are going to use two commands to call alleles in our bam files
'''
mpileup
call
'''

## put all the files (we did files with .sorted.bam for this one) where there is the ...
'''
bcftools mpileup -Ou -f bbc.fasta SRR10729165.sorted.bam SRR10729166.sorted.bam... | bcftools call -mv -Ov -o body_size.vcf
'''

## open the file
'''
nano body_size.vcf
'''
