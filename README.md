# pileup2var

[![release](https://img.shields.io/badge/release-v1.1.0-orange.svg)](https://img.shields.io/badge/release-v1.1.0-orange.svg)
[![license](https://img.shields.io/badge/license-GPLv3-green.svg)](https://img.shields.io/badge/license-GPLv3-green.svg)

pileup2var is a variant calling tool for RNA-based next-generation sequencing data. Unlike VarScan and GATK, pileup2var can operate strandness of directional library types. Moreover, pileup2var is able to count reads from data of different RNA sequencing technologies whose methods focus on inducing base mutation or conversion. It can be used in RNA-seq, PAR-CLIP, m<sup>6</sup>A-SAC-seq, eTAM-seq data and so on. Therefore, pileup2var is qualified to be applied in various RNA study fields like the identification of RNA editing sites, RBP binding sites, RNA modification sites, etc. Thus, pileup2var serves as a general upstream analysis tool for current or developing new RNA sequencing techonologies that are based on base misincorporation or conversion.  

## Dependencies:

- **Samtools**, >= 1.14
- **Parallel::ForkManager**

## Installation:

Add paths of executable programs of Samtools and pileup2var to PATH.

Install the Perl module `Parallel::ForkManager` if parallel data processing is performed:

```bash
# In case that one does not have the root privileges.

# 1. Install cpanm and local::lib
wget -O- http://cpanmin.us | perl - -l ~/perl5 App::cpanminus local::lib
# 2. Setup the appropriate environment variables
eval `perl -I ~/perl5/lib/perl5 -Mlocal::lib`
# 3. Install Parallel::ForkManager
cpanm Parallel::ForkManager
```

## Usage:

### 1. Available options in pileup2var

```
Options:
  -v/--version              Print the version number.
  -h/--help                 Print brief help messages.
  --man                     Full documentation.
  -b/--bam <file>           A bam format file of reads mapping. (Requried)
  -g/--genome <file>        A fasta format file of genome sequences. (Required)
  -o/--out <file>           A TSV file to output variant counts. (Required)
  -t/--thread <int>         Number of CPU threads to run pileup2var. (Default: 1)
  -m/--mrd <int>            Read maximally <int> reads at a position. Set 0 to remove the limit. (Default: 0)
  -p/--mpq <int>            Minimum read mapping quality to be considered valid. (Default: 0)
  -q/--mbq <int>            Minimum base quality for a valid read base. See the document for details. (Default: 0)
  -c/--coverage <int>       Minimum read coverage to output a position record. (Default: 1)
  -a/--base [ATCGatcg]      If defined, only output positions of the defined reference base. (Default: off)
  -i/--inscount <int>       Minimum insertion count to output a position record. (Default: 0)
  -d/--delcount <int>       Minimum deletion count to output a position record. (Default: 0)
  -n/--failedcount <int>    Output counts <= <int> unconverted base counts at a position. (Default: -1)
  -y/--tagYfZf              Output unconverted and converted counts at a position. (Default: off)
  -f/--flag <int>           Ignore reads with the defined SAM flag value. (Default: 1804)
  -s/--strandness [FR]      Either F for forward or R for reverse stranded library type. (Default: R)
  -6/--phred64              Required if the base quality encoding is the Illumina 1.3+. (Default: off)
  -r/--region <string>      If defined, only output positions in the <string> region. (Default: off)
  -e/--bed <file>           If defined, only output positions in the regions from <file>. (Default: off)
```

#### `-b/--bam <file>`

A sorted bam file generated from Bowtie, Bowtie2, Hisat2, STAR, Hisat-3n or other mapping tools.

#### `-o/--out <file>`

The default output is a tab-separetd and 13-column plain file.

| Chrom   | Position  | Strand  | Ref   | Cov   | A     | T     | C     | G   | Ins   | Del   | InsBase   | DelBase   |
|-------  |---------- |-------- |-----  |-----  |-----  |-----  |-----  |---  |-----  |-----  |---------  |---------  |
| chr1    | 631284    | +       | A     | 332   | 332   | 0     | 0     | 0   | 0     | 0     | NA        | NA        |
| chr1    | 631285    | +       | T     | 333   | 0     | 333   | 0     | 0   | 0     | 0     | NA        | NA        |
| chr1    | 631286    | +       | A     | 327   | 327   | 0     | 0     | 0   | 0     | 0     | NA        | NA        |
| chr1    | 631287    | +       | C     | 328   | 0     | 0     | 328   | 0   | 0     | 0     | NA        | NA        |
| chr1    | 631288    | +       | C     | 325   | 1     | 0     | 324   | 0   | 0     | 0     | NA        | NA        |
| chr1    | 631289    | +       | C     | 317   | 0     | 1     | 316   | 0   | 0     | 0     | NA        | NA        |
| chr1    | 631290    | +       | A     | 324   | 324   | 0     | 0     | 0   | 0     | 0     | NA        | NA        |
| chr1    | 631291    | +       | T     | 315   | 0     | 315   | 0     | 0   | 0     | 0     | NA        | NA        |
| chr1    | 631292    | +       | C     | 313   | 0     | 0     | 313   | 0   | 0     | 0     | NA        | NA        |
| chr1    | 631293    | +       | A     | 311   | 311   | 0     | 0     | 0   | 0     | 0     | NA        | NA        |

Note that `Ref` can be a, c, t or g, which probably indicate that the genome sequences used are soft-masked. If any, `InsBase` and `DelBase` list insertion base(s) and deletion base(s), repectively. For example, `C,C` in `DelBase` show that there are two deletion reads, both of whom are C deleted compared with the reference genome. Note that insertion and deletion (InDel) are assigned to the "left" position, i.e., InDel is recorded at upstream 1st position if it occurs at "+" strand or dowmstream 1st position if at "-" strand.
If `-n/failedcount` or (and) `-y/tagYfZf` is used, extra columns are added after `DelBase` (See `-n` and `-y` for details).

#### `-t/--thread <int>`

If `<int>` is larger than one, pileup2var will process data using multiple threads, which is highly recommended when the read depth is high and variant calling is performed in multiple chromosomes.

#### `-m/--mrd <int>`

The option is the same to `-d` in `samtools mpileup`. pipeup2var reads maximally `<int>` reads at a position even if the number of reads at the position are more than the value. Note that setting `-m 0` removes the read depth limit (Default).

#### `-p/--mpq <int>`

The option is the same to `-q` in `samtools mpileup`. pileup2var ignores reads with mapping quality scores less than `<int>`.

#### `-q/--mbq <int>`

pileup2var ignores reads with base quality scores less than `<int>`. Unlike `-Q` in `samtools mpileup`, pileup2var enforces read-pair overlap detection even if `-q 0` is set. This behaviour is different from GATK.

#### `-c/--coverage <int>`

pileup2var ignores positions with read coverage less than `<int>`.

#### `-a/--base [ATCGatcg]`

By default, pileup2var writes all positions covered by reads. This option makes pileup2var output positions of the user-defined base (case insensitive).

#### `-i/--inscount <int>`

pileup2var ignores positions with counts of reads containing insertion less than `<int>`. Setting `-i 0` disables insertion detection.

#### `-d/--delcount <int>`

pileup2var ignores positions with counts of reads containing deletion less than `<int>`. Setting `-d 0` disables deletion detection.

#### `-n/--failedcount <int>`

This option only can be used when the input bam file are generated by Hisat-3n. If defined, an extra column after `DelBase` is added to record counts of reads with unconverted base counts no larger than `<int>`. Setting `-n -1` disables the output.

#### `-y/--tagYfZf`

This option only can be used when the input bam file are generated by Hisat-3n. If defined, two extra columns after `DelBase` are added to record unconverted and converted base counts at a position, respectively.

#### `-f/--flag <int>`

The option is the same to `--ff` in `samtools mpileup`. pileup2var skips reads with the SAM flag value set (see [here](https://broadinstitute.github.io/picard/explain-flags.html) for decoding SAM flags).

#### `-s/--strandness [FR]`

The possible value in the option is either `F` or `R`. `F` means the library type is forward stranded, i.e., the first read of the read pair (or the only read in single-end) is from the transcript strand. `R` means the library type is reverse stranded, i.e., the first read of the read pair (or the only read in single-end) is from the opposite strand of the transcript.

#### `-6/--phred64`

The option is the same to `-6` in `samtools mpileup`. If defined, pileup2var treats the Phred quality score encoding in fastq files as 1.3+ or Phred+64 (See [here](https://en.wikipedia.org/wiki/FASTQ_format#Encoding) for the encoding history).

#### `-r/--region <string>`

With the option defined, pileup2var only writes positions in the region of `<string>`. The possible format is `chr`, `chr:start` or `chr:start-end`. Note that the option is disabled if multiple threads are used.

#### `-e/--bed <file>`

The option is the same to `-l` in `samtools mpileup`. With the option defined, pileup2var only writes positions in the regions from `<file>`. Note that the option is disabled if multiple threads are used.

### 2. Usage examples

#### RNA editing

Mutation calling is performed from RNA-seq. One may consider positions of all bases in order to check the count distribution of different mutation types.

```bash
# Extra filtering steps like PCR deduplication or even multiple-loci read removal can be done before pileup2var.
pileup2var -t 24 -f 1804 -p 20 -q 25 -c 10 -s R -g genome.fa -b in.bam -o out.txt
```

#### RBP binding

Specific mutation type (T to C transition) is called from PAR-CLIP. One may focus on positions of all Ts in the transcriptome.

```bash
# Extra filtering steps like PCR deduplication or even multiple-loci read removal can be done before pileup2var.
pileup2var -t 6 -f 1804 -a T -c 5 -s F -g genome.fa -b in.bam -o out.txt
```

#### m<sup>6</sup>A modification

Conversion calling is performed from eTAM-seq. One may would like to obtain extra information of read counts with a user-defined cutoff for successfully treated reads and counts of surrounding unconverted and converted bases at a target position.

```bash
# Extra filtering steps like PCR deduplication or even multiple-loci read removal can be done before pileup2var.
pileup2var -t 24 -n 5 -y -f 1804 -a A -c 1 -s F -g genome.fa -b in.bam -o out.txt
```

## Citation

If you use **pileup2var** in published research, please cite:

Xiao, Y., Liu S., Ge R., et al. Transcriptome-Wide Profiling and Quantification of *N*<sup>6</sup>-methyladenosine by Enzyme-Assisted Adenosine Deamination.
