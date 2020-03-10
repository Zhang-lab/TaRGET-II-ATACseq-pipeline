# TaRGET-II-ATACseq-pipeline

### Overview
The TaRGET II ATAC-seq pipeline is designed for standardized data processing of mouse ATAC-seq samples. It incorporates automatic quality control, generates user friendly files for computational analysis and outputs genome browser tracks for data visualization. To ensure consistent and reproducible data processing, the entire workflow, associated software and libraries are built into a singularity image, which can be run on computational clusters with job submission as well as on stand-alone machines. Pipeline installation requires minimal user input. All the software and genome references used for TaRGET II ATAC-seq data processing are included in the pipeline image. The pipeline supports both single- and paired-end data, it accepts FASTQ files, performs alignments, peak calling and signal track generation. 
### Software Used in the Pipeline
**cutadapt (v1.16)** was used to find and remove adapter sequence, and other types of unwanted sequence from high-throughput sequencing reads.

**fastqc (v0.11.7)** was used to provide quality control checks on raw sequence data.

**bwa (v0.7.16a)** was used to map sequence reads against the mouse reference genome.

**methylQA (v0.1.9)** was used to generate the reads density and reads mapping statistics.

**macs2 (v2.2.5)** was used for peak calling process.

**samtools (v1.9)** and **bedtools (v2.29)** were used to process the sequence alignments and genome features.

### Genomic References Used in the Pipeline 
The following mouse genome and gene references were built into the singularity image and used for TaRGET ATAC-seq data processing:
1)	BWA indexes of mouse genome (mm10)
2)	Chromosome size file of mm10
3)	Promoter region file of genes from GENCODE vM20 annotation file


# Usage:

**2-step guideline for pipeline usage:**

**step.1** Download the singularity image [here](http://regmedsrv1.wustl.edu/Public_SPACE/bmiao/Public_html/TaRGET_II_pipeline/ATACseq):
```
# download image from local server:
wget http://regmedsrv1.wustl.edu/Public_SPACE/bmiao/Public_html/TaRGET_II_pipeline/ATACseq/ATAC_mm10_target_181103.simg  
```
**step.2** Process ATAC-seq data with the image: 

Please Run at same directory with your data OR the soft link of your data
```
singularity run -B ./:/process <path-to-image> -r <SE/PE> -g <mm10 > -o <read_file1> -p <read_file2>
```
**soft link introduction:** For soft link of data, need to add one bind option for singularity, which is ```-B <full-path-of-original-position>:<full-path-of-original-position>```. 
For example:
soft link the data from /scratch to run on folder /home/example. **Please make sure you use the absolute path.**
```
ln -s /scrach/mydata.fastq.gz /home/example;
cd /home/example
singularity run -B ./:/process -B /scratch:/scratch /home/image/ATAC_mm10_target_181103.simg -r PE -g mm10 -o read1.fastq.gz -p read2.fastq.gz
```
**parameters:**

`-h`: help information.

`-r`: SE for single-end, PE for paired-end.

`-g`: genome reference, one simg is designed for ONLY one species due to the file size. For now the supported genoms are mm10.

`-o`: reads file 1 or the SE reads file, must be ended by .fastq or .fastq.gz or .sra (for both SE and PE). 

`-p`: reads file 2 if input PE data, must be ended by .fastq or .fastq.gz.

`-c`: (optional) specify read length minimum cutoff for methylQA filtering, default 38.

`-t`: (optional) specify number of threads to use, default 24.

`-i`: (optional) insertion free region finding parameters used by Wellington Algorithm (Jason Piper etc. 2013), see documentation for more details.

# Output files

After running the pipeline, there will be a folder called Processed_${name}, all intermediate files and final output files are stored there. The major outputs produced by the pipeline are:
   1)	JSON file with quality control measurement of user supplied dataset. Quality control measurement of ENCODE dataset is  provided for references. 
   2)	Log file with processing status of each step 
   3)	Processed files after alignment (.bam) and peak calling (.narrowPeak) 
   4)	Bigwig file for visualization purpose

# Quailty Control Parameters
The JSON output file contains the QC parameters that were used to check the quality of ATAC-seq data.

**Key QC parameters:**
 
**single_end**: Useful single ends: the total number of useful single ends. This is each end of a non-redundant uniquely mapped read pair.

**enrp**: The promoters of active genes provide a positive control for open chromatin regions. The ATAC-seq useful ends (E) enriched on detected promoters (overlapping ATAC-seq peaks) are used as a QC metric to measure the signal enrichment.

**enrs**: The ATAC-seq signal (useful ends) enriched on the detected ATAC-seq peaks is used as a QC metric to measure the signal enrichment at the genome-wide level. To avoid sequencing-depth bias, 10 million useful ends (E) are sampled from the complete dataset, and peak calling is performed to identify the open chromatin regions. enrs is calculated after 10 million pseudo counts are added into the calculation as background, which can avoid calculation failure caused by the low sequencing depth of testing ATAC-seq library.

**rup**: The percentage of all useful ends that fall into the called peak regions with at least 50% overlap. 

**bk**: Fifty thousand genomic regions (500bp each) are randomly selected from the genome outside of ATAC-seq peaks. The ATAC-seq signal in each region is calculated as reads per kilobase per million mapped reads (RPKM). The percentage of all such regions with the ATAC-seq signal over the theoretical threshold (RPKM=0.377) is considered high-background and used as a QC metric to indicate the background noise

|QC parameter|Value|QC score|
| :---: | :---: | :---: |
| **single end** | >=40000000 | 2 |
| **single end** | \[25000000, 40000000\) | 1 |
| **single end** | <25000000 | -1 |
| **rup** | >=0.2 | 2 |
| **rup** | \[0.12, 0.2\) | 1 |
| **rup** | <0.12 | -1 |
| **enrp** | >=11 | 2 |
| **enrp** | \[7, 11\) | 1 |
| **enrp** | <7 | -1 |
| **enrs** | >=18 | 2 |
| **enrs** | \[15, 18\) | 1 |
| **enrs** | <15 | -1 |
| **bk** | <=10 | 2 |
| **bk** | \(10, 20\] | 1 |
| **bk** | >20 | -1 |

The QC score used to check the quality of data:
| Type |QC cutoff| >= cutoff | < cutoff |
| :---: | :---: | :---: | :---: |
| **Sum of QC score** | 5 | good quality | bad quaility |

