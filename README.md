# CLIPseq-Analysis
## CLIPseq Overview
Paired-end Illumina Novaseq X Plus sequencing was performed to capture protein to RNA binding information. ** ADD **

## FASTQ to bigWig Pipeline
Raw reads were pre-processed to remove low quality bases, adapter sequences, and a stretch of at least 10 consecutive guanine (G) nucleotides at the 3’ end using cutadapt v 4.2. It was necessary to remove long stretches of G’s and ``` --nextseq-trim=10 ``` due to the sequencing instrument reporting high quality G’s in the absence of signal towards the end of the read.
```
cutadapt --match-read-wildcards --times 2 -e 0.1 -O 1 --nextseq-trim=10 -m 18 -a AGATCGGAAGAGCACACGTC -a G{10} -A AGATCGGAAGAGCGTCGTGT -A G{10} -o <OUTPUT R1> -p <OUTPUT R2> <FASTQ R1> <FASTQ R2>
```
FastQC v 0.11.5 was used as a quality control check to ensure adapter removal. 
```
fastqc --outdir=<OUTPUT> <INPUT>
```
The unique molecular identifier (UMI) is 10 random nucleotides and was extracted using UMI-tools v 1.1.5. 
```
umi_tools extract --extract-method=string --bc-pattern=NNNNNNNNNN --stdin=<FASTQ R1> --read2-in=<FASTQ R2> --stdout=<OUTPUT R1> --read2-out=<OUTPUT R2>
```
Reads were aligned to the human genome (hg38) using STAR v 2.7.9a in local alignment mode. 
```
STAR --runMode alignReads --genomeDir <INDEXED HG38 FASTA REFERENCE> --readFilesIn <FASTQ R1> <FASTQ R2> \
--outFilterMultimapNmax 10 --outFilterMultimapScoreRange 1 --outSAMprimaryFlag AllBestScore --outFileNamePrefix <OUTPUT BAM> \
--outSAMattributes All --readFilesCommand zcat --outStd BAM_Unsorted --outSAMtype BAM Unsorted --outFilterMismatchNmax 3 --outFilterType BySJout \
--outReadsUnmapped Fastx --outFilterScoreMin 10 --outSAMattrRGline ID:foo SM:<FILE NAME> \
--alignEndsType Local --limitOutSJcollapsed 2000000 > <OUTPUT BAM>
```
Separately, reads were also aligned to a small genome containing rRNA, tRNA, and snoRNA sequences using Bowtie2 v 2.5.4. 
```
bowtie2 -q --local -x <INDEXED RNA FASTA REFERENCE> -1 <FASTQ R1> -2 <FASTQ R2> -S <OUTPUT SAM>
```
All alignments were sorted and indexed using samtools v 1.6. Code varies slightly whether BAM or SAM is the input.
```
samtools view -bS <INPUT SAM> | samtools sort -o <OUTPUT SORTED BAM> -
samtools index <OUTPUT INDEXED BAM>
```
All alignments were deduplicated using UMI-tools v 1.1.5.
```
umi_tools dedup -I <BAM> --paired -S <DEDUPLICATED BAM>
```
**R1/R2** reads were extracted from the alignments using samtools to obtain reads containing information about the reverse transcription stop. 
```
ADD LATER
```
These alignments were converted to bigWig files normalized by reads per kilobase per million mapped reads (RPKM) using Deeptools v 3.5.1. 
```
bamCoverage -b <BAM> -o <OUTPUT BIGWIG> --outFileFormat bigwig --normalizeUsing RPKM --binSize 1
```
bigWig files were visualized using Integrative Genomics Viewer v 2.19.1 web interface.
