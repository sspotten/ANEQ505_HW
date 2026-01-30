During our **Friday tutorial time only**, we can use this node by first running the following command:
```
sinteractive --reservation=aneq505 --time=01:00:00 --partition=amilan --nodes=1 --ntasks=2 --qos=normal
```

Create tutorial directory:
```
# replace "username" with YOUR CSU EID
cd /scratch/alpine/c829737140@colostate.edu  
  
# make a directory using the mkdir command
mkdir decomp_tutorial  
  
# move into that directory using cd
cd decomp_tutorial
```

## Import decomp tutorial metadata and turn on QIIME2
```
# we first purge any loaded modules from the node we are on, this ensures no conflicting modules are "on"
# modules are preloaded packages of things people commonly use  
module purge
  
# we can turn "on" or "load" qiime2
module load qiime2/2024.10_amplicon
```

Copy metadata from the decomp project into a new metadata folder:
```
# make a directory for the metadata using the mkdir command while you are INSIDE of the decomp_tutorial directory
mkdir metadata
  
# move into that directory using cd
cd metadata
```

```
# Copy metadata file from project folder
cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/metadata_q2_workshop.txt .

# Rename metadata file
mv metadata_q2_workshop.txt metadata.txt
```

Visualize the metadata file:
```
qiime metadata tabulate \
	--m-input-file metadata.txt \
	--o-visualization metadata.qzv
```