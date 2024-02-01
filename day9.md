DAY 9 NOTES
=
first, activate the environment:
```
module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate reademption
```
THEN, WE CREATE A NEW FOLDER.
```
[sunam227@caucluster1 ~]$ cd $WORK
[sunam227@caucluster1 sunam227]$ cd $WORK/RNAseq
[sunam227@caucluster1 RNAseq]$ reademption create --project_path READemption_analysis_KAREN2 --species metanosarcina="Methanosarcina mazei Go1"
-bash: reademption: command not found
[sunam227@caucluster1 RNAseq]$ module load gcc12-env/12.1.0
[sunam227@caucluster1 RNAseq]$ module load miniconda3/4.12.0
[sunam227@caucluster1 RNAseq]$ conda activate reademption
(reademption) [sunam227@caucluster1 RNAseq]$ 
```
--------------
I ran the grep commands again because I accidentally deleted the things.
--------------
    2. copy the reference genome and annotation files to the input folder

    3. copy the raw_reads to the READemption_analysis_KAREN2/input/reads folder

4. Run the following commands after optimization for complete analysis in the backend.
--------------
we will update trc.sh script. its on github:

```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=64G
#SBATCH --time=0-04:00:00
#SBATCH --job-name=rna_seq_methanosarcina
#SBATCH --output=rna_seq_methanosarcina.out
#SBATCH --error=rna_seq_methanosarcina.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate reademption

## 1. create a directory for the analysis
# reademption create --project_path READemption_analysis_KAREN2 \
# 	--species metanosarcina="Methanosarcina mazei Gö1"

#2- copy the sequences and files in respective directories
# download the sequences from the NCBI database or github folder named "genome_input"

#3- Processing and aligning the reads
reademption align --project_path READemption_analysis_KAREN2 \
	--processes 32 --segemehl_accuracy 95 \
	--poly_a_clipping \
	--fastq --min_phred_score 25 \
	--progress

#4- Coverage
reademption coverage --project_path READemption_analysis_KAREN2 \
	--processes 32

#5- Performing gene wise quantification
reademption gene_quanti --project_path READemption_analysis_KAREN2 \
	--processes 32 --features CDS,tRNA,rRNA 

#6- Performing differential gene expression analysis 

####NOTE:: Change the names according to your file names in the READemption_analysis_KAREN2/input/reads/ directory
reademption deseq --project_path READemption_analysis_KAREN2 \
	--libs mut_R1.fastq.gz,mut_R2.fastq.gz,wt_R1.fastq.gz,wt_R2.fastq.gz \
	--conditions mut,mut,wt,wt --replicates 1,2,1,2 \
	--libs_by_species metanosarcina=mut_R1,mut_R2,wt_R1,wt_R2

#7- Create plots 
reademption viz_align --project_path READemption_analysis_KAREN2
reademption viz_gene_quanti --project_path READemption_analysis_KAREN2
reademption viz_deseq --project_path READemption_analysis_KAREN2

# The whole command will take around 2 hours to run.
conda deactivate
module purge
jobinfo
```
we created a new directory but the new file is KAREN2 again.

"what is tnoar?"
a question you should answer in the protocol

WE HAVE GOT THE RNASEQ RESULTS FROM MUHAMMAD.
screenshots of the possible upregulated and downregulated genes were taken.

R:


###GGPLOT2
install.packages("ggplot2")
# Library
library(ggplot2)
library(hrbrthemes)
library(plotly)

# Dummy data
x <- LETTERS[1:20]
y <- paste0("var", seq(1,20))
data <- expand.grid(X=x, Y=y)
data$Z <- runif(400, 0, 5)

# new column: text for tooltip:
data <- data %>%
  mutate(text = paste0("x: ", x, "\n", "y: ", y, "\n", "Value: ",round(Z,2), "\n", "What else?"))

# classic ggplot, with text in aes
p <- ggplot(data, aes(X, Y, fill= Z, text=text)) + 
  geom_tile() +
  theme_ipsum()

ggplotly(p, tooltip="text")

# save the widget
# library(htmlwidgets)
# saveWidget(pp, file=paste0( getwd(), "/HtmlWidget/ggplotlyHeatmap.html"))


