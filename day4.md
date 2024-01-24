DAY 4 NOTES
=
module load samtools
samtools view -bS SAMPLE_BGR_130305.sam > SAMPLE_BGR_130305.bam

module load samtools
samtools view -bS SAMPLE_BGR_130527.sam > SAMPLE_BGR_130527.bam

module load samtools
samtools view -bS SAMPLE_BGR_130708.sam > SAMPLE_BGR_130708.bam

anvi-gen-contigs-database -f contigs.anvio.fa -o contigs.db -n 'biol217'

anvi-run-hmms -c contigs.db