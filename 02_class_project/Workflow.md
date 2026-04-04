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
--input-path manifest/manifest_run2_16S.csv \
--output-path demux/demux.qza
```