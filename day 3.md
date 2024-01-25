DAY 3 NOTES
=
number of contigs in our final assembly file:
```
grep -c ">" final.contigs.fa
```
result= 558356

then, we ran
```
megahit_toolkit contig2fastg 99 final.contigs.fa > final.contigs.fastg                   
```
"To visualize contig graph in Bandage, the first step is to convert the fasta file(s) intermediate_contigs/k{kmer_size}.contigs.fa into SPAdes-like FASTG format. The following code shows the translation from NAME.contigs.fa into NAME.fastg."

metaquast -t 6 -o ? -m 1000 ? was written as:

(anvio-8) [sunamxxx@caucluster2 3_metaquast_out]$ metaquast -t 6 -o . -m 1000 ../3_coassembly/final.contigs.fa

I didnt run the code in an emacs script, which I had to but didnt.
In order to visualize it, I opened another terminal and ran:


scp sunamxxx@caucluster.rz.uni-kiel.de:/work_beegfs/sunam227/Metagenomics/3_metaquast_out/report.html .
```
report.html                                                                                                                                                 100%  482KB  24.8MB/s   00:00    
```

I opened the folder from comp
```
#N50	                   3014
#number of contigs         55836
#number of misassemblies   97
#assembled contigs         55739
#total length of contigs   142642416 bp
```

# BINNING
I created 3_binning under Metagenomics with mkdir. Then,
```
anvi-script-reformat-fasta final.contigs.fa -o ../3_binning/contigs.anvio.fa --min-len 1000 --simplify-names --report-file name_conversion.txt
```
emacs for binning is binning.sh
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH --job-name=reformat
#SBATCH --output=reformat.out
#SBATCH --error=reformat.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

#anvi-script-reformat-fasta final.contigs.fa -o ../3_binning/contigs.anvio.fa --min-len 1000 --simplify-names --report-file name_conversion.txt

module load bowtie2
cd ../3_binning
bowtie2-build contigs.anvio.fa contigs.anvio.fa.index

#BGR130305#
#module load bowtie2
bowtie2 --very-fast -x contigs.anvio.fa.index -1 ../2_fastp/BGR_130305_mapped_R1_clean.fastq.gz -2 ../2_fastp/BGR_130305_mapped_R2_clean.fastq.gz -S SAMPLE_BGR_130305.sam

#BGR130527#
#module load bowtie2
bowtie2 --very-fast -x contigs.anvio.fa.index -1 ../2_fastp/BGR_130527_mapped_R1_clean.fastq.gz -2 ../2_fastp/BGR_130527_mapped_R2_clean.fastq.gz -S SAMPLE_BGR_130527.sam

#BGR130708#
#module load bowtie2
bowtie2 --very-fast -x contigs.anvio.fa.index -1 ../2_fastp/BGR_130708_mapped_R1_clean.fastq.gz -2 ../2_fastp/BGR_130708_mapped_R2_clean.fastq.gz -S SAMPLE_BGR_130708.sam

```
