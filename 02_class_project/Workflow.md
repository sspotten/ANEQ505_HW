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
cd ../dada2

qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux.qza \
--p-trunc-len-f 250 \
--p-trunc-len-r 250 \
--p-n-threads 8 \
--o-table table.qza \
--o-representative-sequences seqs.qza \
--o-denoising-stats dada2_stats.qza
```