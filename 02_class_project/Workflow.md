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
--o-representative-sequences rep-seqs.qza \
--o-denoising-stats dada2_stats.qza \
--o-table cow_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \
--m-input-file cow_dada2_stats.qza \
--o-visualization cow_dada2_stats.qzv

qiime feature-table summarize \
--i-table cow_table_dada2.qza \
--m-sample-metadata-file ../metadata/cow_metadata.txt \
--o-visualization cow_table_dada2.qzv

qiime feature-table tabulate-seqs \
--i-data cow_seqs_dada2.qza \
--o-visualization cow_seqs_dada2.qzv
```