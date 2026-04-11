# Load QIIME2
```
sinteractive --time=02:00:00 --partition=amilan --nodes=1 --ntasks=6 --qos=normal

module purge

module load qiime2/2024.10_amplicon
```
# Import files
```
qiime tools import \
--type "SampleData[PairedEndSequencesWithQuality]" \
--input-format PairedEndFastqManifestPhred33V2 \
--input-path manifest/manifest_run2_16S.tsv \
--output-path demux/demux.qza
```
# Check data quality
```
cd demux

qiime demux summarize \
--i-data demux.qza \
--o-visualization demux.qzv
```
# Denoise
## Slurm script contents
```
#!/bin/bash
#SBATCH --job-name=denoise
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2024.10_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils/dada2

# Denoise with DADA2
qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux.qza \
--p-trunc-len-f 250 \
--p-trunc-len-r 250 \
--p-n-threads 12 \
--o-representative-sequences rep_seqs.qza \
--o-denoising-stats dada2_stats.qza \
--o-table table.qza

#Visualize the denoising results:
qiime metadata tabulate \
--m-input-file dada2_stats.qza \
--o-visualization dada2_stats.qzv

qiime feature-table summarize \
--i-table table.qza \
--m-sample-metadata-file ../metadata/metadata.tsv \
--o-visualization table.qzv

qiime feature-table tabulate-seqs \
--i-data rep_seqs.qza \
--o-visualization rep_seqs.qzv
```
## Run Slurm script
```
sbatch denoise.sh
```
# Filter out large ASVs (off-target taxa)
## Slurm script contents
```
#!/bin/bash
#SBATCH --job-name=filter
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2024.10_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils/dada2

# Filter out large ASVs (off-target taxa)
qiime feature-table filter-seqs \
--i-data rep_seqs.qza \
--m-metadata-file rep_seqs.qza \
--p-where 'length(sequence) < 300' \
--o-filtered-data rep_seqs_filtered300.qza

qiime feature-table tabulate-seqs \
--i-data rep_seqs_filtered300.qza \
--o-visualization rep_seqs_filtered300.qzv

qiime feature-table filter-features \
--i-table table.qza \
--m-metadata-file rep_seqs_filtered300.qza \
--o-filtered-table table_filtered300.qza
  
qiime feature-table summarize \
--i-table table_filtered300.qza \
--m-sample-metadata-file ../metadata/metadata.tsv \
--o-visualization table_filtered300.qzv
```
# Taxonomic classification
## Download pre-trained Greengenes2 classifier
```
mkdir taxonomy

cd taxonomy

wget --no-check-certificate https://ftp.microbio.me/greengenes_release/2024.09/2024.09.backbone.v4.nb.qza
```
## Taxonomic classification sbatch script
```
#!/bin/bash
#SBATCH --job-name=filter
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2024.10_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils/taxonomy

# Classify using Greengenes2 pre-trained classifier
qiime feature-classifier classify-sklearn \  
--i-reads ../dada2/rep_seqs_filtered300.qza \  
--i-classifier 2024.09.backbone.v4.nb.qza \  
--o-classification taxonomy_gg2_filtered300.qza

# Generate taxonomy visualization
qiime metadata tabulate \  
--m-input-file taxonomy_gg2_filtered300.qza \  
--o-visualization taxonomy_gg2_filtered300.qzv
```
Run sbatch script
```
sbatch t
```