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
moved to CC folder and ran maxbin.sh:
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH --job-name=maxbin
#SBATCH --output=maxbin.out
#SBATCH --error=maxbin.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-cluster-contigs -p ./5_anvio_profiles/merged_profiles/PROFILE.db -c ./4_mapping/contigs.db -C MAXBIN2 --driver maxbin2 --just-do-it --log-file log-maxbin2

anvi-summarize -p ./5_anvio_profiles/merged_profiles/PROFILE.db -c ./4_mapping/contigs.db -o SUMMARY_MAXBIN2 -C MAXBIN2
```


# from download:
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
After this, we have ran merged the files:
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
moved to CC folder and ran maxbin.sh:
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH --job-name=maxbin
#SBATCH --output=maxbin.out
#SBATCH --error=maxbin.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-cluster-contigs -p ./5_anvio_profiles/merged_profiles/PROFILE.db -c ./4_mapping/contigs.db -C MAXBIN2 --driver maxbin2 --just-do-it --log-file log-maxbin2

anvi-summarize -p ./5_anvio_profiles/merged_profiles/PROFILE.db -c ./4_mapping/contigs.db -o SUMMARY_MAXBIN2 -C MAXBIN2
```
and metabat.sh:
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

anvi-cluster-contigs -p ./5_anvio_profiles/merged_profiles/PROFILE.db -c ./4_mapping/contigs.db -C METABAT --driver metabat2 --just-do-it --log-file log-metabat2

anvi-summarize -p ./5_anvio_profiles/merged_profiles/PROFILE.db -c ./4_mapping/contigs.db -o SUMMARY_METABAT -C METABAT
```

Results from the testing file (anvio_testing:)
```
archea from metabat: 48
archaea from maxbin: 69
```
# MAGS
Estimate your genomes completeness and contamination levels.
You can assess the quality of your bins by using(qualityest.sh):
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH --job-name=qe1
#SBATCH --output=qe1.out
#SBATCH --error=qe1.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-estimate-genome-completeness -c ./4_mapping/contigs.db -p ./5_anvio_profiles/merged_profiles/PROFILE.db -C METABAT
```
Which binning strategy gives you the best quality for the archaea
bins??
How many archaea bins do you get that are of High Quality? How many bins do you get that are of High Quality?
```
MAXBIN (from index.html)

MAXBIN__010 	maxbin2 	
1,045 	4,453 	59.15% 	
80.26% 	archaea

METABAT
bin name    | domain   |   confidence |   % completion |   % redundancy |   num_splits |   total length |
METABAT__23 | ARCHAEA  |          0.4 |             50 |           5.26 |          352 |        1348667 |
METABAT__44 | ARCHAEA  |          0.9 |          97.37 |           5.26 |          250 |        1847063 |
METABAT__1  | ARCHAEA  |          0.3 |          39.47 |              0 |          138 |         457387 |

Q1: METABAT
Q2: 1 AND 8
```

