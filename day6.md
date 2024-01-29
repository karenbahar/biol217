DAY 6 NOTES
=
we open the genomics directory under the $WORK.
then, we run fastqc.sh:
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=128G
#SBATCH --time=5:00:00
#SBATCH --job-name=01_fastqc
#SBATCH --output=01_fastqc.out
#SBATCH --error=01_fastqc.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2
micromamba activate 01_short_reads_qc

## 1.1 fastqc raw reads
mkdir -p $WORK/genomics/1_short_reads_qc/1_fastqc_raw
for i in $WORK/genomics/0_raw_reads/short_reads/*.gz; do fastqc $i -o $WORK/genomics/1_short_reads_qc/1_fastqc_raw -t 32; done

## 1.2 fastp 
mkdir -p $WORK/genomics/1_short_reads_qc/2_cleaned_reads
fastp -i $WORK/genomics/0_raw_reads/short_reads/241155E_R1.fastq.gz \
 -I $WORK/genomics/0_raw_reads/short_reads/241155E_R2.fastq.gz \
 -R $WORK/genomics/1_short_reads_qc/2_cleaned_reads/fastp_report \
 -h $WORK/genomics/1_short_reads_qc/2_cleaned_reads/report.html \
 -o $WORK/genomics/1_short_reads_qc/2_cleaned_reads/241155E_R1_clean.fastq.gz \
 -O $WORK/genomics/1_short_reads_qc/2_cleaned_reads/241155E_R2_clean.fastq.gz -t 32 -q 25

## 1.3 fastqc cleaned
mkdir -p $WORK/genomics/1_short_reads_qc/3_fastqc_cleaned
for i in $WORK/genomics/1_short_reads_qc/2_cleaned_reads/*.gz; do fastqc $i -o $WORK/genomics/1_short_reads_qc/3_fastqc_cleaned -t 12; done
micromamba deactivate
echo "---------short read cleaning completed successfully---------"
```

Questions

    How Good is the read quality?
    How many reads do you had before trimming and how many do you have now?
    Did the quality of the reads improve after trimming?


Good quality
3.279098 M vs 3.230824 M
Yes

Then, we do nanoplot etc for long reads
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=128G
#SBATCH --time=5:00:00
#SBATCH --job-name=02_long_reads_qc
#SBATCH --output=02_long_reads_qc.out
#SBATCH --error=02_long_reads_qc.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
module load micromamba/1.4.2

echo "---------long reads cleaning started---------"
micromamba activate 02_long_reads_qc

## 2.1 Nanoplot raw
cd $WORK/genomics/0_raw_reads/long_reads/
mkdir -p $WORK/genomics/2_long_reads_qc/1_nanoplot_raw
NanoPlot --fastq $WORK/genomics/0_raw_reads/long_reads/*.gz \
 -o $WORK/genomics/2_long_reads_qc/1_nanoplot_raw -t 32 \
 --maxlength 40000 --minlength 1000 --plots kde --format png \
 --N50 --dpi 300 --store --raw --tsv_stats --info_in_report

## 2.2 Filtlong
mkdir -p $WORK/genomics/2_long_reads_qc/2_cleaned_reads
filtlong --min_length 1000 --keep_percent 90 $WORK/genomics/0_raw_reads/long_reads/*.gz | gzip > $WORK/genomics/2_long_reads_qc/2_cleaned_reads/241155E_cleaned_filtlong.fastq.gz

## 2.3 Nanoplot cleaned
cd $WORK/genomics/2_long_reads_qc/2_cleaned_reads
mkdir -p $WORK/genomics/2_long_reads_qc/3_nanoplot_cleaned
NanoPlot --fastq $WORK/genomics/2_long_reads_qc/2_cleaned_reads/*.gz \
 -o $WORK/genomics/2_long_reads_qc/3_nanoplot_cleaned -t 32 \
 --maxlength 40000 --minlength 1000 --plots kde --format png \
 --N50 --dpi 300 --store --raw --tsv_stats --info_in_report

micromamba deactivate
echo "---------long reads cleaning completed Successfully---------"

module purge
jobinfo
```
Questions

    How Good is the long reads quality?
    How many reads do you had before trimming and how many do you have now?

good
15963 vs 12446

ran the code that has been published(pipeline...): 
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=128G
#SBATCH --time=5:00:00
#SBATCH --job-name=pipeline_genome_assembly
#SBATCH --output=pipeline_genome_assembly.out
#SBATCH --error=pipeline_genome_assembly.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0 #for GTDB-tk database path
module load micromamba/1.4.2
export MAMBA_ROOT_PREFIX=$HOME/.micromamba
eval "$(micromamba shell hook --shell=bash)"


# 3 Assembly (1 hour)-----------------------------------------------------------
echo "---------Unicycler Assembly pipeline started---------"
micromamba activate 03_unicycler
cd $WORK/genomics
mkdir -p $WORK/genomics/3_hybrid_assembly
unicycler -1 $WORK/genomics/1_short_reads_qc/2_cleaned_reads/241155E_R1_clean.fastq.gz -2 $WORK/genomics/1_short_reads_qc/2_cleaned_reads/241155E_R2_clean.fastq.gz -l $WORK/genomics/2_long_reads_qc/2_cleaned_reads/241155E_cleaned_filtlong.fastq.gz -o $WORK/genomics/3_hybrid_assembly/ -t 32
micromamba deactivate
echo "---------Unicycler Assembly pipeline Completed Successfully---------"

# 4 Assembly quality-----------------------------------------------------------
echo "---------Assembly Quality Check Started---------"

## 4.1 Quast (5 minutes)
micromamba activate 04_checkm_quast
cd $WORK/genomics/3_hybrid_assembly
mkdir -p $WORK/genomics/3_hybrid_assembly/quast
quast.py $WORK/genomics/3_hybrid_assembly/assembly.fasta --circos -L --conserved-genes-finding --rna-finding \
 --glimmer --use-all-alignments --report-all-metrics -o $WORK/genomics/3_hybrid_assembly/quast -t 32
micromamba deactivate

## 4.2 CheckM
micromamba activate 04_checkm_quast
cd $WORK/genomics/3_hybrid_assembly
mkdir -p $WORK/genomics/3_hybrid_assembly/checkm
checkm lineage_wf $WORK/genomics/3_hybrid_assembly/ $WORK/genomics/3_hybrid_assembly/checkm -x fasta --tab_table --file $WORK/genomics/3_hybrid_assembly/checkm/checkm_results -r -t 32
checkm tree_qa $WORK/genomics/3_hybrid_assembly/checkm
checkm qa $WORK/genomics/3_hybrid_assembly/checkm/lineage.ms $WORK/genomics/3_hybrid_assembly/checkm/ -o 1 > $WORK/genomics/3_hybrid_assembly/checkm/Final_table_01.csv
checkm qa $WORK/genomics/3_hybrid_assembly/checkm/lineage.ms $WORK/genomics/3_hybrid_assembly/checkm/ -o 2 > $WORK/genomics/3_hybrid_assembly/checkm/final_table_checkm.csv
micromamba deactivate

# 4.3 Checkm2
# (can not work, maybe due to insufficient memory usage)
micromamba activate 05_checkm2
cd $WORK/genomics/3_hybrid_assembly
mkdir -p $WORK/genomics/3_hybrid_assembly/checkm2
checkm2 predict --threads 32 --input $WORK/genomics/3_hybrid_assembly/* --output-directory $WORK/genomics/3_hybrid_assembly/checkm2 
micromamba deactivate
echo "---------Assembly Quality Check Completed Successfully---------"

# 5 Annotate-----------------------------------------------------------
echo "---------Prokka Genome Annotation Started---------"

micromamba activate 06_prokka
cd $WORK/genomics/3_hybrid_assembly
# Prokka creates the output dir on its own
prokka $WORK/genomics/3_hybrid_assembly/assembly.fasta --outdir $WORK/genomics/4_annotated_genome --kingdom Bacteria --addgenes --cpus 32
micromamba deactivate
echo "---------Prokka Genome Annotation Completed Successfully---------"


# 6 Classification-----------------------------------------------------------
echo "---------GTDB Classification Started---------"
# (can not work, maybe due to insufficient memory usage increase the ram in bash script)
micromamba activate 07_gtdbtk
conda env config vars set GTDBTK_DATA_PATH="$WORK/Databases/GTDBTK_day6";
micromamba activate 07_gtdbtk
cd $WORK/genomics/4_annotated_genome
mkdir -p $WORK/genomics/5_gtdb_classification
echo "---------GTDB Classification will run now---------"
gtdbtk classify_wf --cpus 12 --genome_dir $WORK/genomics/4_annotated_genome/ --out_dir $WORK/genomics/5_gtdb_classification --extension .fna 
# reduce cpu and increase the ram in bash script in order to have best performance
micromamba deactivate
echo "---------GTDB Classification Completed Successfully---------"

# 7 multiqc-----------------------------------------------------------
echo "---------Multiqc Started---------"
micromamba activate 01_short_reads_qc
multiqc -d $WORK/genomics/ -o $WORK/genomics/6_multiqc
micromamba deactivate
echo "---------Multiqc Completed Successfully---------"


module purge
jobinfo

```

opened the .csv files on BANDAGE.

# GTDB

# MULTIQC

Questions

    How good is the quality of genome?
    Why did we use Hybrid assembler?
    What is the difference between short and long reads?
    Did we use Single or Paired end reads? Why?
    Write down about the classification of genome we have used here
GOOD
We used to assemble short and long reads to have (more accurate results)
Paired. To have more results
Bacteroides sp002491635
user_genome	classification	fastani_reference	fastani_reference_radius	fastani_taxonomy	fastani_ani	fastani_af	closest_placement_reference	closest_placement_radius	closest_placement_taxonomy	closest_placement_ani	closest_placement_af	pplacer_taxonomy	classification_method	note	other_related_references(genome_id,species_name,radius,ANI,AF)	msa_percent	translation_table	red_value	warnings
PROKKA_01282024	d__Bacteria;p__Bacteroidota;c__Bacteroidia;o__Bacteroidales;f__Bacteroidaceae;g__Bacteroides;s__Bacteroides sp002491635	GCF_004793475.1	95.0	d__Bacteria;p__Bacteroidota;c__Bacteroidia;o__Bacteroidales;f__Bacteroidaceae;g__Bacteroides;s__Bacteroides sp002491635	97.39	0.78	GCF_004793475.1	95.0	d__Bacteria;p__Bacteroidota;c__Bacteroidia;o__Bacteroidales;f__Bacteroidaceae;g__Bacteroides;s__Bacteroides sp002491635	97.39	0.78	d__Bacteria;p__Bacteroidota;c__Bacteroidia;o__Bacteroidales;f__Bacteroidaceae;g__Bacteroides;s__	taxonomic classification defined by topology and ANI	topological placement and ANI have congruent species assignments	GCA_025147485.1, s__Bacteroides uniformis, 95.0, 91.8, 0.63; GCF_000614125.1, s__Bacteroides rodentium, 95.0, 90.63, 0.62; GCA_905197435.1, s__Bacteroides sp905197435, 95.0, 86.29, 0.65; GCF_000195635.1, s__Bacteroides fluxus, 95.0, 81.59, 0.46; GCA_902388495.1, s__Bacteroides sp902388495, 95.0, 81.32, 0.42; GCF_001688725.2, s__Bacteroides caecimuris, 95.0, 80.86, 0.22; GCA_025147325.1, s__Bacteroides stercoris, 95.0, 80.7, 0.41; GCA_905203765.1, s__Bacteroides sp905203765, 95.0, 80.67, 0.49; GCA_910578895.1, s__Bacteroides sp910578895, 95.0, 80.63, 0.42; GCF_018390535.1, s__Bacteroides propionicigenes, 95.0, 80.45, 0.4; GCF_900129655.1, s__Bacteroides clarus, 95.0, 80.3, 0.39; GCF_900155865.1, s__Bacteroides bouchesdurhonensis, 95.0, 80.27, 0.15; GCF_000374365.1, s__Bacteroides gallinarum, 95.0, 80.26, 0.33; GCF_003865075.1, s__Bacteroides faecalis, 95.0, 80.23, 0.14; GCA_025146565.1, s__Bacteroides eggerthii, 95.0, 80.13, 0.37; GCF_000614145.1, s__Bacteroides faecichinchillae, 95.0, 80.03, 0.14; GCA_021203515.1, s__Bacteroides sp021203515, 95.0, 80.01, 0.37; GCF_000186225.1, s__Bacteroides helcogenes, 95.0, 79.89, 0.38; GCF_900241005.1, s__Bacteroides cutis, 95.0, 79.83, 0.35; GCF_024623065.1, s__Bacteroides acidifaciens, 95.0, 79.82, 0.21; GCF_900130125.1, s__Bacteroides congonensis, 95.0, 79.81, 0.21; GCF_000172175.1, s__Bacteroides intestinalis, 95.0, 79.79, 0.33; GCF_002998435.1, s__Bacteroides zoogleoformans, 95.0, 79.78, 0.42; GCF_000381365.1, s__Bacteroides salyersiae, 95.0, 79.71, 0.18; GCF_004342845.1, s__Bacteroides heparinolyticus, 95.0, 79.67, 0.41; GCA_905215345.1, s__Bacteroides sp905215345, 95.0, 79.58, 0.29; GCF_000315485.1, s__Bacteroides oleiciplenus, 95.0, 79.57, 0.35; GCF_900142015.1, s__Bacteroides stercorirosoris, 95.0, 79.55, 0.35; GCF_000156195.1, s__Bacteroides finegoldii, 95.8579, 79.48, 0.18; GCF_009193325.2, s__Bacteroides zhangwenhongii, 95.8579, 79.43, 0.21; GCF_000513195.1, s__Bacteroides timonensis, 95.0, 79.36, 0.31; GCF_000210075.1, s__Bacteroides xylanisolvens, 95.0, 79.32, 0.2; GCF_902364365.1, s__Bacteroides sp900556215, 95.2143, 79.29, 0.31; GCF_020091405.1, s__Bacteroides sp900552405, 95.0, 79.21, 0.32; GCA_945607605.1, s__Bacteroides sp945607605, 95.0, 79.18, 0.46; GCF_903181435.1, s__Bacteroides sp900765785, 95.0, 79.14, 0.22; GCF_900106755.1, s__Bacteroides faecis, 95.0, 79.12, 0.19; GCF_000158035.1, s__Bacteroides cellulosilyticus, 95.0, 79.11, 0.3; GCA_900755095.1, s__Bacteroides sp900755095, 95.0, 79.11, 0.13; GCF_003464595.1, s__Bacteroides intestinalis_A, 95.2143, 79.08, 0.31; GCA_014385165.1, s__Bacteroides sp014385165, 95.0, 79.06, 0.15; GCA_910586915.1, s__Bacteroides sp910586915, 95.0, 79.04, 0.27; GCA_000613465.1, s__Bacteroides nordii, 95.0, 78.99, 0.17; GCF_014334015.1, s__Bacteroides intestinigallinarum, 95.0, 78.93, 0.19; GCA_905207245.1, s__Bacteroides sp905207245, 95.0, 78.91, 0.3; GCA_023458215.1, s__Bacteroides sp023458215, 95.0, 78.91, 0.31; GCF_002849695.1, s__Bacteroides fragilis_A, 95.0, 78.89, 0.18; GCA_910585845.1, s__Bacteroides sp910585845, 95.0, 78.89, 0.34; GCF_000011065.1, s__Bacteroides thetaiotaomicron, 95.0, 78.83, 0.19; GCF_002222615.2, s__Bacteroides caccae, 95.0, 78.73, 0.18; GCA_900555635.1, s__Bacteroides sp900555635, 95.0, 78.73, 0.33; GCF_000499785.1, s__Bacteroides neonati, 95.0, 78.69, 0.12; GCF_009193295.2, s__Bacteroides luhongzhouii, 95.0, 78.58, 0.17; GCF_000025985.1, s__Bacteroides fragilis, 95.0, 78.36, 0.18; GCA_900547205.1, s__Bacteroides sp900547205, 95.0, 78.36, 0.17; GCF_001314995.1, s__Bacteroides ovatus, 95.0, 78.35, 0.18; GCF_014750685.1, s__Bacteroides sp014750685, 95.0, 78.33, 0.2; GCF_019583405.1, s__Bacteroides fragilis_B, 95.0, 78.31, 0.17; GCF_012113595.1, s__Bacteroides sp012113595, 95.0, 78.29, 0.19; GCA_905215555.1, s__Bacteroides sp905215555, 95.0, 78.28, 0.25; GCA_900761785.1, s__Bacteroides sp900761785, 95.0, 78.26, 0.19; GCA_902362375.1, s__Bacteroides sp902362375, 95.0, 78.18, 0.18; GCA_002293435.1, s__Bacteroides sp002293435, 95.0, 78.15, 0.23; GCA_944322345.1, s__Bacteroides sp944322345, 95.0, 78.12, 0.14; GCA_900557355.1, s__Bacteroides sp900557355, 95.0, 78.02, 0.16; GCA_009929715.1, s__Bacteroides sp009929715, 95.0, 78.01, 0.09; GCF_014196225.1, s__Bacteroides pyogenes_A, 95.0, 78.0, 0.16; GCA_934716785.1, s__Bacteroides sp934716785, 95.0, 77.88, 0.14; GCF_000428105.1, s__Bacteroides pyogenes, 95.0, 77.86, 0.16; GCA_002471185.1, s__Bacteroides sp002471185, 95.0, 77.77, 0.08; GCA_900766005.1, s__Bacteroides sp900766005, 95.0, 77.72, 0.19; GCA_017992695.1, s__Bacteroides sp017992695, 95.0, 77.65, 0.15; GCA_002471195.1, s__Bacteroides sp002471195, 95.0, 77.65, 0.07; GCA_007896885.1, s__Bacteroides sp007896885, 95.0, 77.62, 0.11; GCA_022648295.1, s__Bacteroides sp022648295, 95.0, 77.61, 0.12; GCA_900766195.1, s__Bacteroides sp900766195, 95.0, 77.56, 0.11; GCF_000517545.1, s__Bacteroides reticulotermitis, 95.0, 77.52, 0.11; GCA_900556625.1, s__Bacteroides sp900556625, 95.0, 77.49, 0.15; GCA_023426145.1, s__Bacteroides sp023426145, 95.0, 77.43, 0.11; GCA_019416685.1, s__Bacteroides sp900762525, 95.0, 77.29, 0.12; GCA_019412865.1, s__Bacteroides sp019412865, 95.0, 76.33, 0.07	97.48	11	N/A	N/A

