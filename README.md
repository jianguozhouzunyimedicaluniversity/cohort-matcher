[![Build Status](https://jenkins-ci.pri.bms.com:8443/job/cohort-matcher/statusbadges-build/icon)](https://jenkins-ci.pri.bms.com:8443/job/cohort-matcher)
[![Code Grade](https://jenkins-ci.pri.bms.com:8443/job/cohort-matcher/statusbadges-grade/icon)](https://jenkins-ci.pri.bms.com:8443/job/cohort-matcher)
[![Coverage](https://jenkins-ci.pri.bms.com:8443/job/cohort-matcher/statusbadges-coverage/icon)](https://jenkins-ci.pri.bms.com:8443/job/cohort-matcher)

# cohort-matcher #

A simple tool for determining whether two cohorts of [BAM files](https://samtools.github.io/hts-specs/SAMv1.pdf) contain reads sequenced from the same samples or patients by counting genotype matches at common SNPs.  Cohort-matcher is built on BAM-matcher.

BAM-matcher is most useful at comparing whole-genome-sequencing (WGS), whole-exome-sequencing (WES) and RNA-sequencing (RNA-seq) human data, but can also be customised to compare panel data or non-human data.

To compare two cohorts, run:
```
/ngs/apps/Python-2.7.8/bin/python /ngs/apps/cohort-matcher/cohort_matcher.py \
        --set1 cohort1.txt --set2 cohort2.txt \
        --cache-dir `pwd`/cache --scratch-dir /scratch \
        --caller freebayes \
        --vcf /ngs/apps/cohort-matcher/hg19.exome.highAF.7550.vcf \
        --reference /ngs/reference/hg19/hg19.fa \
        --freebayes-path /ngs/apps/freebayes/bin/freebayes \
        --aws /usr/bin/aws \
        --Rscript /ngs/apps/R-3.2.2/bin/Rscript \
        --samtools /ngs/apps/samtools-0.1.19/samtools \
        --output-dir output
```

which will output a series of files indicating sample similarity include:
cohort-matcher-results.txt
cohort-matcher-results.pdf
topmatches.txt
meltedResults.txt

# Docker #

Cohort matcher is available in Docker via AWS Batch.   Use the following arguments to run the AWS Batch cohort-matcher job: 

--set1_s3_path
--set2_s3_path
--set1_reference [hg19/GRCh37]
--set2_reference [hg19/GRCh37]
--s3_output_folder_path

## Variant Callers ##

(Require at least one)

* GATK (requires Java)
* VarScan2 (requires Java and Samtools)
* Freebayes

Note: Cohort-matcher only supports Freebayes at this time.

## Installation ##

```
git clone https://github.com/golharam/cohort-matcher
```

The repository includes 3 VCF files which can be used for comparing human data (hg19/GRCh37). 

These VCF files also contain variants extracted from 1000 Genomes project which are all exonic and have high likelihood of switching between REF and ALT alleles (global allele frequency between 0.45 and 0.55). The only difference between them is the number of variants contained within.

The repository also includes several BAM files which can be used for testing (under test_data directory), as well as the expected results for various settings.

Cohort-matcher adds unit tests to test the python code.

# LICENCE #

The code is released under the Creative Commons by Attribution licence (http://creativecommons.org/licenses/by/4.0/). You are free to use and modify it for any purpose (including commercial), so long as you include appropriate attribution. 

# Citation #

*BAM-matcher: a tool for rapid NGS sample matching*

Paul P.S. Wang; Wendy T. Parker; Susan Branford; Andreas W. Schreiber
Bioinformatics 2016

[doi: 10.1093/bioinformatics/btw239](http://bioinformatics.oxfordjournals.org/content/early/2016/05/01/bioinformatics.btw239.abstract)


# Who do I talk to? #

Paul (paul.wang @ sa.gov.au)
