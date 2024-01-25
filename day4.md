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

module load samtools
samtools view -bS SAMPLE_BGR_130527.sam > SAMPLE_BGR_130527.bam

module load samtools
samtools view -bS SAMPLE_BGR_130708.sam > SAMPLE_BGR_130708.bam
```
Then, for contigs data preperation:
```
anvi-gen-contigs-database -f contigs.anvio.fa -o contigs.db -n 'biol217'
```


anvi-run-hmms -c contigs.db