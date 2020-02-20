[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)

# agro932-homework1

## Job content
Simulate an NGS dataset for two populations with a small reference genome (< 1 Mb) of yourchoice, each population with 10 diploid individuals, to address a hypothesis about populationdifferentiation.

- Establish a version-controlled directory system to host the project.
- Describe your simulation strategy and the hypothesis to test (positive, negative, or neutral selection).
- Use ANGSD to calculate thetas for each population and Fst between the twopopulations.
- Breakdown the theta ratios and Fst values into different genomic features (i.e., genic and intergenic regions) according to the general feature format (GFF) file for the referencegenome of your choice.
- Visualize and interpret your results and report them in a reproducible manner.

## What I do

### Establish a version-controlled directory system to host the project.
- Establish project **agro932-homework1**: *git@github.com:lijinlong1991/agro932-homework1.git*
- This is [directory system](http://projecttemplate.net/architecture.html)

### Describe your simulation strategy and the hypothesis to test (positive, negative, or neutral selection).
- simulation strategy hypothesis
two population
positive selection: result in very large differences in allele frequencies.


### 
#### download the reference genome 
[link](ftp://ftp.ensemblgenomes.org/pub/plants/release-46/fasta/zea_mays/dna/Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa.gz)
#### then unzip it
gunzip Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa.gz

#### simulate 10 individals
for i in {1..10}
do
   wgsim Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa -e 0 -d 500 -N 50000 -1 100 -2 100 -r 0.1  -R 0 -X 0 l$i.read1.fq l$i.read2.fq
done
 
#### check
wc -l l1.read1.fq 


#### Align the NGS reads to the reference genome
module load bwa samtools
bwa index Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa    #index the reference genome # using bwa mem to align the reads to the reference genome
#bwa mem Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa l1.read1.fq l1.read2.fq | samtools view -bSh - > l1.bam  # => samtools to convert into bam file


#### Do alignment for 2 individuals using bash loop
for i in {1..10}; do bwa mem Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa l$i.read1.fq l$i.read2.fq | samtools view -bSh - > l$i.bam; done # alignment
for i in *.bam; do samtools sort $i -o sorted_$i; done # sort
for i in sorted*.bam; do samtools index $i; done # index them

#### Check mapping statistics
samtools flagstat sorted_l1.bam


#### write the bam files to a txt file
mkdir bam_files
mv sorted*.bam bam_files
cd bam_files


angsd -bam bam.txt -doSaf 1 -anc ../Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa -GL 1  -out out 
# use realSFS to calculate sfs
/home/jyanglab/jli89/bin/angsd/misc/realSFS out.saf.idx > out.sfs
cp out.sfs ../../cache/

angsd -bam bam.txt -out out -doThetas 1 -doSaf 1 -pest out.sfs -anc ../Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa -GL 1
/home/jyanglab/jli89/bin/angsd/misc/thetaStat print out.thetas.idx > theta.txt
## cp theta to the cache/ folder
cp theta.txt ../../cache/

#### pop1&pop2
#### first calculate per pop saf for each populatoin
angsd -b pop1.txt -anc ../Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa -out pop1 -dosaf 1 -gl 1
angsd -b pop2.txt -anc ../Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa -out pop2 -dosaf 1 -gl 1
# calculate the 2dsfs prior
/home/jyanglab/jli89/bin/angsd/misc/realSFS pop1.saf.idx pop2.saf.idx > pop1.pop2.ml
# prepare the fst for easy window analysis etc
/home/jyanglab/jli89/bin/angsd/misc/realSFS fst index pop1.saf.idx pop2.saf.idx -sfs pop1.pop2.ml -fstout out
# get the global estimate
/home/jyanglab/jli89/bin/angsd/misc/realSFS fst stats out.fst.idx 
# below is not tested that much, but seems to work
/home/jyanglab/jli89/bin/angsd/misc/realSFS fst stats2 out.fst.idx -win 500 -step 100 > fst_win.txt

#### cp sfs to the cache/ folder
cp fst_win.txt ../../cache/


## Breakdown the theta ratios and Fst values into different genomic features (i.e., genic andintergenic regions) according to the general feature format

wget ftp://ftp.ensemblgenomes.org/pub/plants/release-46/fasta/zea_mays/dna/Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa.gz
gunzip Zea_mays.B73_RefGen_v4.dna.chromosome.Mt.fa.gz

- then work in the R



## Visualize and interpret your results and report them in a reproducible manner.

- Repot in the profilling foder


## License
This is an ongoing research project. It was intended for internal lab usage. It has not been extensively tested. Use at your own risk.
It is a free and open source software, licensed under [GPLv3](LICENSE).
