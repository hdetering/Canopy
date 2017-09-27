## **How do I generate CNA input for Canopy?**
To generate allele-specific copy number calls, [Sequenza](https://CRAN.R-project.org/package=sequenza) or [FALCON](https://CRAN.R-project.org/package=falcon) can be used. Both methods require paired tumor normal samples as input.


### FALCON-X (recommended)

FALCON-X is for ASCN profiling with WES of one or more tumor samples, multiple (>=20) normal controls, which don't have to be matched.

[FALCON-X demo code](https://github.com/yuchaojiang/Canopy/blob/master/instruction/falconx_demo.R)


### FALCON (recommended)

FALCON is for ASCN profiling with WGS of paired tumor normal samples.

[FALCON demo code](https://github.com/yuchaojiang/Canopy/blob/master/instruction/falcon_demo.R)

Above is demo code for applying Falcon to paired normal, primary tumor, and relapse genome of a neuroblatoma patient from [Eleveld et al. (Nature Genetics 2015)](http://www.nature.com/ng/journal/v47/n8/abs/ng.3333.html), provided by Drs. Derek Oldridge, Sharon Diskin, and John Maris. FALCON processes each chromosome separately and here we focus on chr14 of the relapse genome, where a copy-neutral loss of heterozygousity (LOH) event has been previous reported in the relapse. After QC procedure for data input and curation of segmentation output, the final plot as well as table output is below.

<p align="center">
Falcon plot output chr14 of neuroblatoma relapse genome
  <img src='https://github.com/yuchaojiang/Canopy/blob/master/instruction/falcon.relapse.qc.jpg' width='500' height='700'>
</p>

<p align="center">
Falcon table output chr14 of neuroblatoma relapse genome
  <img src='https://github.com/yuchaojiang/Canopy/blob/master/instruction/falcon_table_output.jpg' width='600' height='66'>
</p>


### Sequenza
Sequenza estimates allele-specific copy numbers as well as tumor purity and ploidy using B-allele frequencies and depth ratios from paired tumor-normal sequencing data. Here are Sequenza outputs from WES of normal, primary tumor and relapse genome of a neuroblastoma patient from [Eleveld et al. (Nature Genetics 2015)](http://www.nature.com/ng/journal/v47/n8/abs/ng.3333.html).

   * [Primary tumor Sequenza segment output](https://github.com/yuchaojiang/Canopy/blob/master/instruction/primary.txt)
   * [Relapse genome Sequenza segment output](https://github.com/yuchaojiang/Canopy/blob/master/instruction/relapse.txt)

Data visualization and QC procedures on Sequenza's segmentation output are strongly recommended. Below is genome-wide view from Sequenza as a sanity check. The top panel is for the primary tumor and the bottom panel is for the relapse. The red and blue lines are for the major and minor copy numbers, respectively. We see that large chromosome-arm level deletions and duplications are fairly concordant between the primary tumor and the relapse genome (check!), while the relapse tumor gained additional CNAs at the second timepoint. The small CNA events, however, are most likely false positives and should be filtered out through QC procedures, which may include:
   * Have at least 100 heterozygous loci within each segment;
   * Are at least 5Mb long;
   * Don't have extreme estimated copy numbers (e.g., total copy number >= 6);
   * ...
   * ...

It is also worth noting that **additional steps are further neede to generate curated CNA segments**. For example, chr17q is split into three segments in the primary tumor, which should be just one large duplication instead (via comparison against the relapse). Furthermore, the breakpoints of the same CNA events between the primrary tumor and the relapse genome are sometimes different and they need to be merged. They should NOT be treated as separate events.

<p align="center">
Primary tumor Sequenza segmentation genome-wide view
  <img src='https://github.com/yuchaojiang/Canopy/blob/master/instruction/primary.jpg' >
</p>
<p align="center">
Relapse genome Sequenza segementation genome-wide view
  <img src='https://github.com/yuchaojiang/Canopy/blob/master/instruction/relapse.jpg' >
</p>

Canopy does not require interger allele-specific copy number as input. While Sequenza infers tumor purity and ploidy and outputs interger-valued allle-specifc copy numbers, Canopy directly takes as input the fractional allele-specific copy numbers, which can be obtained from the segmentation output (see above). The B-allele frequency is Bf = Wm / (WM + Wm) and the depth ratio is depth.ratio = (WM + Wm)/2. From here the input matrix WM and Wm can be calculated. The standard errors of the estimated copy numbers, epsilonM and epsilonm, can be set as the default value by Canopy or be taken as the sd.ratio from Sequenza's output, assuming they are the same. NOTE: epsilonM and epsilonm should NOT be sd(WM) and sd(Wm). The former are both matrix of dimension number of CNA events x number of samples, unless if they are set all the same by default as a scalar; the latter are the standard deviations of the copy numbers across all events in all samples.


## **How do I generate SNA input for Canopy?**

 * We use UnifiedGenotyper by **[GATK](https://software.broadinstitute.org/gatk/)** to call somatic SNAs and follow its [Best Practices](https://software.broadinstitute.org/gatk/best-practices/). A demo code can be found [here](https://github.com/yuchaojiang/Canopy/blob/master/instruction/UnifiedGenotyper.sh). **[MuTect](http://archive.broadinstitute.org/cancer/cga/mutect)** and **[VarScan2](http://massgenomics.org/varscan)** can also be used when paired normal samples are available.

 * *Stringent* QC procedures are strongly recommended. Just to list a few QCs that we have adopted:
    * Pass variant recalibration (VQSR) from GATK;
    * Have only one alternative allele (one locus being double hit by two different SNAs in one patient is very unlikely);
    * Are highly deleterious (i.e., focuse on driver mutations) from functional annotations (**[ANNOVAR](http://annovar.openbioinformatics.org/en/latest/)**);
    * Have low population variant frequency from the 1000 Genomes Project (if no normal samples are available);
    * Don't reside in segmental duplication regions;
    * Have high depth of coverage (total as well as mutated read depth);
    * Reside in target baits (e.g., exonic regions for exome sequencing);
    * ...
    * ...
      

 * After mutation calling and QC procedures, the number of mutated reads will be stored in matrix R while the total number of reads will be stored in matrix X across all mutational loci in all samples. An example data input of a leukemia patient AML43 from [Ding et al. (Nature 2012)](http://www.nature.com/nature/journal/v481/n7382/full/nature10738.html) can be found [here](https://github.com/yuchaojiang/Canopy/blob/master/instruction/AML43_DingEtAl.txt). Note: This study focused on SNAs in copy-neutral regions of the genome.
 
 * A good way for sanity check is to plot the variant allele frequencies (VAFs) across samples.
   * If there are only two samples, a 2-D scatterplot will suffice. In the figure below, the left panel contains the VAFs at two timepoints for the leukemia patient from [Ding et al. (Nature 2012)](http://www.nature.com/nature/journal/v481/n7382/full/nature10738.html). We see that there are clear clusters of mutations indicating they fall on the same branch of the tree, which serves as a nice sanity check of our mutation calling and filtering.
   * If there are more than two samples, heatmap can be used for visualization. The right panel in the figure below is the heatmap of VAFs across 63 somatic mutations in 10 spatially separated tumor slices of a ovarian cancer patient from [Bashashati et al. (The Journal of pathology 2013)](http://onlinelibrary.wiley.com/doi/10.1002/path.4230/abstract). The heatmap below is plotted using the [pheatmap R package](https://CRAN.R-project.org/package=pheatmap).

<p align="center">
  <img src='https://github.com/yuchaojiang/Canopy/blob/master/instruction/demo-page-001.jpg' width='500' height='500' >
</p>

