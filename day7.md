DAY7NOTES
=
``````
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=128G
#SBATCH --time=5:00:00
#SBATCH --job-name=anvio_pangenomics
#SBATCH --output=anvio_pangenomics.out
#SBATCH --error=anvio_pangenomics.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

# create new folder
mkdir $WORK/pangenomics/02_anvio_pangenomics

(anvio-8) [sunam227@caucluster1 V_jascida_genomes]$ anvi-estimate-genome-completeness -e external-genomes.txt
+---------------+----------+--------------+----------------+----------------+--------------+----------------+
| genome name   | domain   |   confidence |   % completion |   % redundancy |   num_splits |   total length |
+===============+==========+==============+================+================+==============+================+
| V_jascida_12  | BACTERIA |            1 |            100 |           5.63 |          282 |        5814827 |
+---------------+----------+--------------+----------------+----------------+--------------+----------------+
| V_jascida_13  | BACTERIA |            1 |            100 |           5.63 |          283 |        5812569 |
+---------------+----------+--------------+----------------+----------------+--------------+----------------+
| V_jascida_14  | BACTERIA |            1 |            100 |           5.63 |          290 |        5861661 |
+---------------+----------+--------------+----------------+----------------+--------------+----------------+
| V_jascida_47  | BACTERIA |            1 |            100 |           5.63 |          282 |        5821176 |
+---------------+----------+--------------+----------------+----------------+--------------+----------------+
| V_jascida_52  | BACTERIA |            1 |            100 |          21.13 |          362 |        6303185 |
+---------------+----------+--------------+----------------+----------------+--------------+----------------+
| V_jascida_53  | BACTERIA |            1 |            100 |           5.63 |          289 |        5853972 |
+---------------+----------+--------------+----------------+----------------+--------------+----------------+
| V_jascida_55  | BACTERIA |            1 |            100 |           5.63 |          289 |        5857980 |
+---------------+----------+--------------+----------------+----------------+--------------+----------------+
| V_jascida_ref | BACTERIA |            1 |            100 |           5.63 |          298 |        5989523 |
+---------------+----------+--------------+----------------+----------------+--------------+----------------+

* The 'domain' shown in this table is the domain anvi'o predicted for your contigs
  in a given bin with the amount of confidence for that call in the 'domain'
  column. If the domain is 'mixed', it means it is very likely you have contigs
  in your bin that spans accross multiple domains of life (maybe it is worth a
  Nobel, but more likely it is just garbage). If the domain is 'blank', it means
  anvi'o did not find enough signal from single-copy core genes to determine
  what domain it should use to estimate the completion and redundancy of this
  bin accurately. You can get much more information about these estimations by
  running the same command line with the additinal flag `--debug`.
