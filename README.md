# Domain Screening
Gilbert Lab domain screening analysis pipeline
 <img width="1738" alt="image" src="https://github.com/cmwilson24/Domain_Screening/assets/62605069/f33b2aa3-b780-4ad3-a743-e6c76f0b9b40">

PacBio Demultiplexing with _lima_
  You will need a barcode fasta file that will look something like this: 
  
```
lima --ccs --same --split-named --dump-clips -j 20 --log-level=INFO input.ccs.bam barcodes.fa output.ccs.demux.bam
```
Trim demultiplexed files - trimming is only necessary if sequencing on a SequelII/IIe. Otherwise skip. 
```
seqtk trimfq -b 16 -e 16 input.fq > trimmed.fq
```

Gzip your files
```
pigz -p 8 file.fq
```

Create a sample sheet for your experiments (.yaml file) - see example

**Alternatively, if you have a lot of samples, you can do the following set of commands (for batch/high throughput sample processing):***
#Start with conda environment needed to demux and trim files. If using PacBio REVIO - We don't need to do CCS because these are HIFI reads
conda activate PacBio
```python
lima --same --split-named --dump-clips -j 25 --log-level=INFO m84066_231208_174057_s1.hifi_reads.bam  DCF007_barcodes.fa ccs.demux.bam 
```
#Run lima to demux files based on the PCR barcoded primers specified in the barcodes.fa file.

#Use the ccs.demux.report.counts to get a sense of read numbers per demuxed samples
**If you used multiple flowcells and now need to combine samples**
```
find . -name "*.bam" -type f -exec basename {} \; | sort | uniq -c | awk '$1 == 3 {print $2}' > common_files.txt
```
**Merging samples that have the same file name (in this example we have 3 directories/flow cells)**
```
while read -r filename; do
    samtools merge -@ <#threads> combined_bams/"$filename" your_directory1/hifi_reads/"$filename" your_directory2/hifi_reads/"$filename" your_directory3/hifi_reads/"$filename"
done < common_files.txt
```
**Convert bam file to fastq**
```
for bam_file in combined_bams/*.bam; do
    output_prefix="fastq_output/$(basename "$bam_file" .bam)"
    samtools bam2fq -@ 25 "$bam_file" > "$output_prefix.fastq"
done
```
**Trimming fastqs and zipping**
```
conda actviate SeqTrim
mkdir trimmed_output  # Create a directory to store the trimmed FASTQ files

for fq_file in *.fastq; do
    output_file="trimmed_output_newbc/tr_${fq_file}"
    seqtk trimfq -b 16 -e 16 "$fq_file" > "$output_file"
done
#will need to gzip them all 
for trimmed_file in trimmed_output_newbc/*.fastq; do
    gzip "$trimmed_file"; done
```
**Time for alignments**
In a new conda environment, install the following: 
First install 3 fusions from JH. See dCas9-fusions for next steps. They will start with the following...
```
git clone git@github.com:jeffhussmann/hits.git
git clone git@github.com:jeffhussmann/knock-knock.git
git clone git@github.com:jeffhussmann/dCas9-fusions.git
#conda activate JH_hits_newer
count_dCas9_fusion_domains parallel --batch DCF007_updated_bc ./ 30
```
