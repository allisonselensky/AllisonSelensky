## Starting point
'''
bcftools mpileup -Ou -f bbc.fasta \
    SRR10729165.sorted.bam SRR10729166.sorted.bam SRR10729566.sorted.bam SRR10733526.sorted.bam \
    SRR31835375.sorted.bam SRR31835473.sorted.bam SRR31835482.sorted.bam SRR31835573.sorted.bam \
    | bcftools call -mv -Ov -o body_size.vcf
'''

## Step 1: Check your sample names - bcftools mpileup names samples after the BAM file paths by default, so confirm what they look like before splitting into groups:
'''
conda activate bio_env
bcftools query -l body_size.vcf
'''

## Step 2: Compress and index the VCF - The next steps need a compressed, indexed VCF (your -Ov output is plain text):
'''
bgzip body_size.vcf
bcftools index body_size.vcf.gz
'''

## Step 3: Define your two groups - Make two plain text files, one sample name per line, matching exactly what bcftools query -l printed:
'''
(#group1.txt - doesn't actually get typed - just telling that these .bam files go into group 1)
SRR10729165.sorted.bam
SRR10729166.sorted.bam
SRR10729566.sorted.bam
SRR10733526.sorted.bam

(#group2.txt)
SRR31835375.sorted.bam
SRR31835473.sorted.bam
SRR31835482.sorted.bam
SRR31835573.sorted.bam
'''

## Step 4: Split the VCF by group
'''
bcftools view -S group1.txt body_size.vcf.gz -Oz -o group1.vcf.gz
bcftools view -S group2.txt body_size.vcf.gz -Oz -o group2.vcf.gz
'''

## Step 4.5 - filter each group's VCF to strictly biallelic SNPs first
'''
bcftools view -m2 -M2 -v snps group1.vcf.gz -Oz -o group1_biallelic.vcf.gz
bcftools view -m2 -M2 -v snps group2.vcf.gz -Oz -o group2_biallelic.vcf.gz
'''

## Step 5: Calculate allele frequency within each group - bcftools +fill-tags recalculates INFO tags (including AF, allele frequency) based on only the samples currently in the file — which is exactly why we split into two files first. Run it separately on each group:
'''
bcftools +fill-tags group1_biallelic.vcf.gz -Oz -o group1_af.vcf.gz -- -t AF
bcftools +fill-tags group2_biallelic.vcf.gz -Oz -o group2_af.vcf.gz -- -t AF
'''

## Step 6: Pull out just CHROM, POS, and AF
'''
bcftools query -f '%CHROM\t%POS\t%INFO/AF\n' group1_af.vcf.gz > group1_af.tsv
bcftools query -f '%CHROM\t%POS\t%INFO/AF\n' group2_af.vcf.gz > group2_af.tsv
(#Each file now has one row per SNP: chromosome, position, and that group's allele frequency (0 to 1) at that site)
'''

## check your group files
'''
nano group1_af.tsv
nano group2_af.tsv
'''

## go into R
'''
R 
'''

## Step 7: Merge, compute the difference, and plot in R
'''
g1 <- read.table("group1_af.tsv", col.names = c("CHROM", "POS", "AF1"))
g2 <- read.table("group2_af.tsv", col.names = c("CHROM", "POS", "AF2"))

(#merge on shared sites only)
merged <- merge(g1, g2, by = c("CHROM", "POS"))
(#drop any sites where AF couldn't be calculated in one group)
(#(e.g. no called genotypes in that subset))

merged <- na.omit(merged)
merged$CHROM (#to see it)
merged$AF_diff <- merged$AF1 - merged$AF2
merged$AF_diff (#to see it)
'''

## make a plot of allele frequency differences
'''
pdf('merged.pdf')

plot(merged$POS, merged$AF_diff,
     pch = 19, col = "steelblue",
     xlab = "Position in gene", ylab = "Allele frequency difference (Group1 - Group2)",
     main = "Allele frequency difference along bbc")
abline(h = 0, lty = 2, col = "grey40")

dev.off() (#we are done with this file-closes the pdf file) 
'''

## How do you look at your pdf? - Navigate to someone on YOUR computer where you will be able to find a file using cd
'''
(#go into a new terminal on Ubuntu and make sure its on YOUR computer, then type in this code)
scp -r visitor@134.129.113.23:/storehouse/visitor/table_/pigmentation/merged.pdf . (#space and . at the end)
(#it will ask for the passord which is temp) 
'''

## group1.txt - Zambia (warm population)
## group2.txt - Ethiopia (cold population)
## AF1 - Zambia
## AF2 - Ethiopia
## Zambia - Ethiopia (actually subtracting them) -> if Zambia has higher frequency we'll get a positive number, If Zambia has a smaller frequency we'll get a negative number
