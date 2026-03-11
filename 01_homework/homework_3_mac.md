~={red}(1point)=~ for Alpha Rarefaction Plot
Run Core Metrics ~={red}(1 point; .25pts per line)=~
Make alpha diversity plots ~={red}(3points)=~
~={red}10 points=~ for the questions 

~={red}15 points total=~
------------------------------------------------------------------

Due: 

**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------
**Learning Objectives**
1. Practice recording commands and editing code to match your analysis.
2. Perform alpha rarefaction and determine an appropriate sequencing depth.
3. Run core metrics, generate plots for alpha and beta diversity
--------------------------------------------------

**Cow Site Data Workflow**, part 3

Load qiime2 in a terminal session after you go into the **cow** folder

```
# Insert the two commands to activate qiime2
module purge
module load qiime2/2024.10_amplicon
```

### Alpha Rarefaction Plot ~={red}(1 point)=~
- Chose the input sequencings depths (min and max) for generating the alpha rarefaction plot: 

```
#go to the cow directory

qiime diversity alpha-rarefaction \
--i-table dada2/cow_table_dada2_filtered300.qza \
--m-metadata-file metadata/cow_metadata.txt \
--o-visualization alpha_rarefaction_curves_16S.qzv \
--p-min-depth 3 \
--p-max-depth 11000
```


### Run Core Metrics ~={red}(1 point)=~

```
qiime diversity core-metrics-phylogenetic \
--i-table dada2/cow_table_dada2_filtered300.qza \
--i-phylogeny tree/tree_gg2.qza \
--m-metadata-file metadata/cow_metadata.txt \
--p-sampling-depth 1500 \
--output-dir core_metrics_results
```


### Visualize alpha diversity plots
- generate a plot to visualize the observed features ~={red}(1 point)=~
```
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/observed_features_vector.qza \
--m-metadata-file metadata/cow_metadata.txt \
--o-visualization core_metrics_results/observed_features_statistics.qzv
```

- generate a plot to visualize faith's PD ~={red}(2 points)=~
```
## insert the entire code chunk for generating this visualization
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/faith_pd_vector.qza \
--m-metadata-file metadata/cow_metadata.txt \
--o-visualization core_metrics_results/faiths_pd_statistics.qzv
```



## Homework questions ~={red}(10 points)=~

1. what is the name of the file you needed to use to figure out what min and max depths to use to generate the alpha rarefaction plot? (Hint: which file contains the sequencing depths for each sample)
	1. The name of the file we need to use to figure out the min and max sequencing depths to use to generate the alpha rarefaction plot is the feature table visualization `cow_table_dada2_filtered300.qzv`.
2. what did you choose for the rarefaction depth (the input for core metrics -p-sampling-depth flag)? why?
	1. I chose a sampling depth of 1,500 reads to try to balance retaining as many samples as possible (89%) with culling low-depth (and therefore low-information) samples. 
3. Which cow body location had more observed features? Which has the lowest?
	1. The fecal samples consistently had the most observed features. Aside from the negative extraction controls, the nasal samples had the lowest number of observed features.
4. What is the main difference between Faiths PD and Shannons alpha diversity metrics?
	1. The main difference between Faith's PD and Shannon diversity is that Faith's PD is phylogenetically-informed, while Shannon diversity is based on the richness and evenness of taxa in a sample. Faith's PD is calculated by summing the branch lengths connecting taxa in a sample, measuring the sample's diversity as a function of the evolutionary relationships within it.
5. Which diversity metrics produced by the core-metrics pipeline require phylogenetic information?
	1. The diversity metrics in the core-metrics pipeline that require phylogenetic information are:
		1. Faith's Phylogenetic Diversity
		2. Unweighted UniFrac
		3. Weighted UniFrac
6. Which two body sites have the highest Faiths PD alpha diversity?  Are the groups significantly different?
	1. The skin and fecal samples have the highest Faith's PD alpha diversity. With a q-value of 3.548956e-04 in the Kruskal-Wallis pairwise test, these two body sites are statistically significantly different from each other in their microbial community structures.
7. Does it seem like there are any groupings in the beta diversity? What are the groupings?
	1. There do appear to be some groupings in the beta diversity metrics (unweighted UniFrac and Bray-Curtis). The fecal samples are clustering together on their own, the skin and udder samples together form another grouping, and the nasal and oral samples form a third grouping, although it is much more loosely clustered than the first two.
8. Why do you think these samples are grouping together?
	1. These samples are likely grouping together because their host environments share similar characteristics and/or they are in close physical proximity to each other, leading to similarities in their microbiome communities (e.g. skin with udder, nasal with oral). The fecal samples are coming from a completely different host environment, the cow's gut, which does not share environmental conditions with any of the other sample types, hence why the fecal samples form their own cluster in the PCoA.
9. What test can you run to determine if the groups are significantly different?
	1. We can use a PERMANOVA (non-parametric multivariate analysis of variance) to test whether the distances within each group are different from differences between groups.
10. What command would you use to run that test?

```
#insert command for running the test you suggest from question 7

# Unweighted UniFrac PERMANOVA:
qiime diversity beta-group-significance \
--i-distance-matrix core_metrics_results/unweighted_unifrac_distance_matrix.qza \
--m-metadata-file metadata/cow_metadata.txt \
--m-metadata-column body_site \
--o-visualization core_metrics_results/unweighted_unifrac_distance_matrix.qzv

# Bray-Curtis PERMANOVA:
qiime diversity beta-group-significance \
--i-distance-matrix core_metrics_results/bray_curtis_distance_matrix.qza \
--m-metadata-file metadata/cow_metadata.txt \
--m-metadata-column body_site \
--o-visualization core_metrics_results/bray_curtis_distance_matrix.qzv
```