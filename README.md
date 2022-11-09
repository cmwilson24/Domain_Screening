# Domain Screening
Gilbert Lab domain screening pipeline

PacBio Demultiplexing with _lima_
  You will need a barcode fasta file that will look something like this: 
  
  <<Insert screenshot here >>
  
```
lima --ccs --same --split-named --dump-clips -j 10 --log-level=INFO input.ccs.bam barcodes.fa output.ccs.demux.bam
```
Trim demultiplexed files
```
seqtk trimfq -b 16 -e 16 input.fq > trimmed.fq
```




First install 3 fusions from JH. dCas9-fusions is private, so you will need an authentication token from your github account.
```
git clone git@github.com:jeffhussmann/hits.git
git clone git@github.com:jeffhussmann/knock-knock.git
git clone git@github.com:jeffhussmann/dCas9-fusions.git
```
