# BAM-match #

A tool for determining whether two BAM files were sequenced from the same sample or patient. 

## Installation ##

```
cd /directory/path/where/bam-matcher/is/to/be/installed/
git clone https://bitbucket.org/sacgf/bam-matcher.git
```

Either include the path to BAM-matcher to the environment variable PATH, or move ```bam-matcher.py```, ```bam-matcher.conf``` and ```bam_matcher_html_template``` to a directory that is in the PATH already.


## Dependencies ##

**Python** 

(version 2.7)

**Python libraries**

* PyVCF
* HTSeq
* ConfigParser
* Cheetah

**Variant Callers**

(Require at least one)

* GATK (requires Java)
* VarScan2 (requires Java and Samtools)
* Freebayes

## Configuration ##

BAM-matcher requires a configuration file. If the default configuration file is missing, a template can be generated by the ```--generate-config (-G)``` function. 

```
BAM-matcher.py --generate-config path_to_file_to_be_generated
```


The configuration file contains these sections:

```
[VariantCallers]
# file paths to variant callers and other binaries
GATK:      GenomeAnalysisTK.jar
freebayes: freebayes
samtools:  samtools
varscan:   VarScan.jar
java:      java

[ScriptOptions]
DP_threshold:    15
filter_VCF:      False
number_of_SNPs:  1500
# enable --targets option for Freebayes, faster but more prone to Freebayes errors
# set to False will use --region, each variant is called separately
fast_freebayes: True
VCF_file: variants_noX.vcf

[VariantCallerParameters]
# GATK memory usage in GB
GATK_MEM: 4
# GATK threads (-nt)
GATK_nt:  1
# VarScan memory usage in GB
VARSCAN_MEM: 4

[GenomeReference]
# default reference fasta file
REFERENCE: hg19.fasta

# Reference fasta file, with no chr in chromosome name (e.g. Broad19.fasta)
REF_noChr: Broad19.fasta

# Reference fasta file with 'chr' in chromosome names
REF_wChr:  genome.fa

[BatchOperations]
CACHE_DIR:  cache_dir

[Miscellaneous]
```

Most configuration settings can also be overridden at run time.

**[VariantCallers]**

Paths to variant callers and their required components (Java, Samtools)

**[ScriptOptions]**

Settings for BAM-matcher comparison.

* **DP_threshold**: the minimum read depth required for both BAM files to make a genotype comparison at any given site. (Recommended: 15 for WES-WES comaprison).

* **filter_VCF**: When set to True, will filter out variants which are not SNPs and have 1KG_AF values outside of 0.45-0.55. (Recommended: False. Generally, easier to just pre-select the variants positions to be compared, and not use this option).

* **number_of_SNPs**: The maximum number of variant sites to compare, even if the the input VCF file contains more variants. 

* **fast_freebayes**: When using Freebayes for genotype calling, by default, each position is called separately (with --region). This is less efficient, but as Freebayes' --targets sometimes fails in our testing, this is a safer option. Set this option to "True" will enable using "--targets" during when running Freebayes. (Recommended: False. Slower but safer).

* **VCF_file**: If you are using the same VCF file most of the time, then just set this option here, then you won't need to specify the VCF path every time you run BAM-matcher.



**[VariantCallerParameters]**

Set memory requirements (Java VM) and number of processing threads (GATK only).

**[GenomeReference]**

Set path(s) to the genome reference file. **REFERENCE** should always be set. **REF_noChr** and **REF_wChr** is to deal with the GRCh37 situation where two compatible assemblies are available, but one uses 'Chr' in chromosome names and the other does not. By setting these paths, BAM-matcher can compare BAM files that were mapped to different references.


**[BatchOperations]**

BAM-matcher caches the genotype data of BAM files that it has processed previously.  **CACHE_DIR** sets the location of the cache directory. 


## Running BAM-matcher ##

If the configuration file is set up with paths to REFERENCE and VCF file, then you can run a comparison by:

```
bam-matcher.py -B1 BAM_FILE_1 -B2 BAM_FILE_2 
```

This assumes that the configuration is in the same directory as bam-matcher.py and is called bam-matcher.conf.

As no output options are specified, the output is written to standard-output, and also to a text file in current working directory. 

Run ```bam-matcher.py -h```  to see the full help message.

```
REQUIRED:
  --bam1 BAM1, -B1 BAM1
                        First BAM file
  --bam2 BAM2, -B2 BAM2
                        Second BAM file
```

Minimum required input, if the configuration file is set up.


```
CONFIGURATION:
  --config CONFIG, -c CONFIG
                        Specify configuration file (default =
                        /dir/where/script/is/located/bam-matcher.conf)
  --generate-config GENERATE_CONFIG, -G GENERATE_CONFIG
                        Specify where to generate configuration file template
```

By default, BAM-matcher looks for the config file ("bam-matcher.conf") in the same directory as the script itself. The --config option can be used to specify a different config file. 


```
OUTPUT REPORT:
  --output OUTPUT, -o OUTPUT
                        Specify output report path (default =
                        /current/dir/bam_matcher.SUBFIX)
  --short-output, -so   Short output mode (tab-separated).
  --html, -H            Enable HTML output. HTML file name = report + '.html'
  --no-report, -n       Don't write output to file. Results output to command
                        line only.
  --scratch-dir SCRATCH_DIR, -s SCRATCH_DIR
                        Scratch directory for temporary files. If not
                        specified, the report output directory will be used
                        (default = /tmp/[random_string])
```

If no output settings are specified, BAM-matcher will print out results to standard output and write results to bam_matcher.SUBFIX in the current working directory, where SUBFIX includes the BAM file names and a random string.

The scratch directory is usually deleted at the end of a successful run, unless --debug option is set, then the temporary files will be kept. If you are using the --scratch-dir option, the specified path must not exist already (although its parent directory should exist).



```
VARIANTS:
  --vcf VCF, -V VCF     VCF file containing SNPs to check (default can be
                        specified in config file instead)
  --filter-vcf, -FT     Enable filtering of the input VCF file
```

Use --vcf to specify the variants to compare. This will override the setting in the config file.

--filter-vcf is the same as filter_VCF setting in config file.


```
CALLERS AND SETTINGS (will override config values):
  --caller {gatk,freebayes,varscan}, -CL {gatk,freebayes,varscan}
                        Specify which caller to use (default = 'gatk')
  --dp-threshold DP_THRESHOLD, -DP DP_THRESHOLD
                        Minimum required depth for comparing variants
  --number_of_snps NUMBER_OF_SNPS, -N NUMBER_OF_SNPS
                        Number of SNPs to compare.
  --fastfreebayes, -FF  Use --targets option for Freebayes.
  --gatk-mem-gb GATK_MEM_GB, -GM GATK_MEM_GB
                        Specify Java heap size for GATK (GB, int)
  --gatk-nt GATK_NT, -GT GATK_NT
                        Specify number of threads for GATK UnifiedGenotyper
                        (-nt option)
  --varscan-mem-gb VARSCAN_MEM_GB, -VM VARSCAN_MEM_GB
                        Specify Java heap size for VarScan2 (GB, int)
```

These are all the same as the settings in config file. Specifying values here will override config settings.



```
REFERENCES:
  --reference REFERENCE, -R REFERENCE
                        Default reference fasta file. Needs to be indexed with
                        samtools faidx
  --ref_noChr REF_NOCHR, -Rn REF_NOCHR
                        Reference fasta file, no 'chr' in chromosome names.
                        Needs to be indexed with samtools faidx
  --ref_wChr REF_WCHR, -Rw REF_WCHR
                        Reference fasta file, has 'chr' in chromosome names.
                        Needs to be indexed with samtools faidx
  --bam1-reference BAM1_REFERENCE, -B1R BAM1_REFERENCE
                        Reference fasta file for BAM1. Requires
                        --bam2-reference/-B2R, overrides other settings
  --bam2-reference BAM2_REFERENCE, -B2R BAM2_REFERENCE
                        Reference fasta file for BAM2. Requires
                        --bam1-reference/-B1R, overrides other settings
```

These are all the same as the settings in config file. Specifying values here will override config settings.


```
BATCH OPERATIONS:
  --do-not-cache, -NC   Do not keep variant-calling output for future
                        comparison. By default (False) data is written to
                        /bam/filepath/without/dotbam.GT_compare_data
  --recalculate, -RC    Don't use cached variant calling data, redo variant-
                        calling. Will overwrite cached data unless told not to 
                        (-NC)
  --cache-dir CACHE_DIR, -CD CACHE_DIR
                        Specify directory for cached data. Overrides
                        configuration
```

Alter caching parameters at run time.



```
optional arguments:
  -h, --help            show this help message and exit
  --debug, -d           Debug mode. Temporary files are not removed
  --verbose, -v         Verbose reporting. Default = False
```












### Who do I talk to? ###

* Repo owner or admin
* Other community or team contact