_Updated: Aug 21, 2023_
# Biastools: Measuring, visualizing and diagnosing reference bias

This github is originally forked from https://github.com/sheila12345/biastools

## Prerequisite programs:
- samtools=v1.11
- bcftools=v1.9
- bedtools=v2.30.0
- gzip=v1.9
- tabix=v1.9
- bowtie2=v2.4.2
- bwa=v0.7.17
- mason_simulator=v2.0.9 (SeqAn=v2.4.0)
  
Not all the programs are necessary for every stage. Users can use `--force` option to disable the prerequisite checking when running biastools

## Install
- [pip](https://pypi.org/project/biastools/)
```
pip install biastools
```
- [Github](https://github.com/maojanlin/biastools.git)
```
git clone https://github.com/maojanlin/biastools.git
cd biastools
```
Though optional, it is a good practice to setup a virtual environment to manage the dependancies:

```
python -m venv venv
source venv/bin/activate
```
Now a virtual environment (named venv) is activated:

```
python setup.py install
```


## Usage:

### Simulation, plotting, and analysis
```
$ biastools --simulate --align --analyze -o <work_dir> -g <ref.fa> -v <vcf> -s <sample_name> -r <run_id>
```

With the example command, biastools will 
- [1] Simulate reads based on <ref.fa> and <vcf>, generating pair-end .fq.gz files for both haplotypes (work_dir/sample_name.hapA/B_1/2.fq.gz). 
- [2] Then align the reads to the reference <ref.fa>, generating bam file with phasing information (work_dir/sample_name.run_id.sorted.bam).
- [3] Analyze the bam file with context-aware asignment method, generating bias reports and plots.

#### Other aligners
Biastools support the alignment with `Bowtie 2` and `BWA MEM`. Additional alignment method can be performed on the simulated reads and feed in the analysis with command
```
$ biastools --analyze -o <work_dir> -g <ref.fa> -v <vcf> -s <sample_name> -r <run_id>
```

Noted that the alignment file should be named with <work_dir/sample_name.run_id.sorted.bam> and tag with haplotype information.


#### Real data
Context-aware assignment can also analyze real data with `-R` option, but only generate the plot without simulation information (sample_id.real.indel_balance.pdf).
```
$ biastools --analyze -o <work_dir> -g <ref.fa> -v <vcf> -s <sample_name> -r <run_id> -R
```


#### Multiple indel plots
Multiple analysis result can be combined into one single indel-balance plot.
```
$ biastools --analyze -o <work_dir> -g <ref.fa> -v <vcf> -s <sample_name> -r <run_id> \
                      -lr file1.bias file2.bias file3.bias... \
                      -ld run_id1 run_id2 run_id3...
```

The product file `sample_name.combine.sim.indel_balance.pdf` will merge the indel information of the bias reports after the `-l` option with the simulated balance information. For real bias report only, the `-R` option will generate combined plot file `sample_name.combine.real.indel_balance.pdf` without the simulated balance.


### Bias prediction from bias report
#### Real data
Biastools can predict if a variant is bias or not based on the context-aware assignment:
```
$ biastools --predict -o <work_dir> -g <ref.fa> -v <vcf> -s <sample_name> -r <run_pd_id> -pr <path_to_bias_report>
```
With the example command, biastools will 
- [4] The command will generate two files: `sample_name.real.pd_id_bias.tsv` and `sample_name.real.pd_id_suspicious.tsv`. The `bias.tsv` report contains all the sites predicted to be biased by the model. The `suspicious.tsv` file contains the sites which suspicious of lacking enough information from the vcf file. In another word, the reads align to the site shows different pattern to the haplotype indicated by the vcf file. 

#### Simulated guided prediction
```
$ biastools --predict -o <work_dir> -g <ref.fa> -v <vcf> -s <sample_name> -r <run_pd_id> \
                      -pr <path_to_bias_report> \
                      -ps <path_to_simulated_bias_report>
```
If the report of the sample based on simulated data is presented, biastools can generate cross prediction experiment result. In the experiment, the ground truth bias sites are based on simulation data.


### Scanning bias without vcf information
#### Scanning
```
$ biastools_scan --scan -o <work_dir> -g <ref.fa> -s <sample_name> -r <run_id> -i <path_to_target.bam>
```

Biastools will transform the <path_to_target.bam> into mpileup format, and then scanning the mpileup and generate the `sample_name.run_id.bias.bed` and `sample_name.run_id.suspicious.bed`


#### Compare two bam files with common baseline
```
$ biastools_scan --compare_bam -o <work_dir> -g <ref.fa> -s <sample_name> -r <run_id> \
                               -i  <path_to_target.bam> \
                               -i2 <path_to_second.bam> \
                               -m  <path_to_target.mpileup> \
                               -m2 <path_to_second.mpileup>
```
Biastools will generate a common baseline from `path_to_target.bam` and `path_to_second.bam`, and use the new common baseline to recalculate the bias regions based on the two mpileup files. The mpileup files can be generated by running **scanning** first, or directly run the **bcftools consensus**.



#### Directly compare two bias reports
User can also generate the comparison of the bias reports without a common baseline (not recommended):
```
$ biastools_scan --compare_rpt -o <work_dir> -s <sample_name> -r <run_id> \
                               -b1 <path_to_target_bias.bed> \
                               -b2 <path_to_improved_bias.bed> \
                               -l2 <path_to_improved_lowRd.bed>
```




