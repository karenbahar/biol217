DAY 4 NOTES
=
First, we run sunam227.sh:
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH --job-name=sambam
#SBATCH --output=sambam.out
#SBATCH --error=sambam.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

module load samtools
samtools view -bS SAMPLE_BGR_130305.sam > SAMPLE_BGR_130305.bam
samtools view -bS SAMPLE_BGR_130527.sam > SAMPLE_BGR_130527.bam
samtools view -bS SAMPLE_BGR_130708.sam > SAMPLE_BGR_130708.bam
```
Then, for contigs data preperation (inside sunam227.sh):
```
anvi-gen-contigs-database -f contigs.anvio.fa -o contigs.db -n biol217
```
HMM SEARCH ON CONTIGS:
```
anvi-run-hmms -c contigs.db
```
(didn"t work, so recieved the pdf from outside)
Next, we ran this for binning with anvio, after `scp` ing the .bam extension files into 5_anvio... folder:
```
for i in *.bam; do anvi-init-bam $i -o "$i".sorted.bam; done
```
Then, for anvio profiles, we ran these commands(under profiling.sh):
"anvi-profile -i YOUR_SORTED.bam -c contigs.db --output-dir OUTPUT_DIR"
so, the commands are:
```
anvi-profile -i BGR_130305.bam.sorted.bam -c contigs.db --output-dir anvioprof_output
anvi-profile -i BGR_130527.bam.sorted.bam -c contigs.db --output-dir anvioprof_output
anvi-profile -i BGR_130708.bam.sorted.bam -c contigs.db --output-dir anvioprof_output
```
After this, we have ran metabat.sh:
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH --job-name=metabat2
#SBATCH --output=metabat2.out
#SBATCH --error=metabat2.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-merge ./anvioprof_output/BGR_130305/PROFILE.db ./anvioprof_output/BGR_130527/PROFILE.db ./anvioprof_output/BGR_130708/PROFILE.db -o ./metabat_output -c . --enforce-hierarchical-clustering
```
