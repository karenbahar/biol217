DAY5 NOTES
=
we ran `summarize.sh` for bin refinement:
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH --job-name=summarize
#SBATCH --output=summarize.out
#SBATCH --error=summarize.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-summarize -p ./5_anvio_profiles/merged_profiles/PROFILE.db -c ./4_mapping/contigs.db --list-collections
```
.SH CODE RAN ON TERMINAL. (ANVI-SUMMARIZE)

then:
```
anvi-summarize -c ./4_mapping/contigs.db -p ./5_anvio_profiles/merged_profiles/PROFILE.db -C METABAT2 -o SUMMARY_METABAT2 --just-do-it
```
then:
```
cd ./5_anvio_profiles/SUMMARY_METABAT2/bin_by_bin

mkdir ../../ARCHAEA_BIN_REFINEMENT

cp ./METABAT_23/*.fa ./ARCHAEA_BIN_REFINEMENT
cp ./METABAT_44*.fa ./ARCHAEA_BIN_REFINEMENT
cp ./METABAT_1/*.fa ./ARCHAEA_BIN_REFINEMENT

```
# GUNC
gunc.sh
```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH --job-name=gunc3
#SBATCH --output=gunc3.out
#SBATCH --error=gunc3.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate gunc

cd $WORK/Metagenomics/CC/anvio_testing_follow_up/5_anvio_profiles/ARCHAEA_BIN REFINEMENT

mkdir GUNC

for i in *.fa; do gunc run -i "$i" -r /work_beegfs/sunam227/Databases/gunc_db_progenomes2.1.dmnd --out_dir GUNC/"$i" --threads 10 --detailed_output; done

#gunc plot -d ./GUNC/diamond_output/METABAT###-contigs.diamond.progenomes_2.1.out -g ./GUNC/genes_calls/gene_counts.json
```
no gunc results obtained.
