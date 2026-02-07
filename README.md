# salmonella-enterica-genome-assembly
Background and workflow detailing the steps on assembling the whole genome of the bacteria Salmonella enterica.

## Introduction
Whole-genome sequencing (WGS) has contributed much of our understanding of microbial pathogens. A primary goal of WGS is to computationally reconstruct smaller and fragmented segments of DNA into, ideally, a closed circular genome. Subsequently, this allows analysis goals such as alignment to the reference genome and variant calling to identify genetic differences such as single nucleotide polymorphisms (SNPs) and insertions or deletions (indels). Consequently, scientists can investigate bacterial evolution and clonal/genetic lineage distributions via Multi Locus Sequence Typing (MLST) (dos Santos et al., 2024; Paranthaman et al., 2021). Recent advancements in Oxford Nanopore Technologies (ONT) long-read sequencing have expanded the WGS and assembly toolbox. However, challenges remain due to several bacterial species containing many repetitive elements that make accurate assembly difficult (R. R. Wick et al., 2023). Additionally, long-read sequencing has historically exhibited lower per-base quality and is more expensive compared to short-read sequencing technologies, but the gap is closing recently (Goodwin et al., 2016; Iyer et al., 2024). The main goals of this analysis are to assemble raw reads from a *Salmonella enterica* (*S. enterica*) isolate into a consensus genome, align raw reads to a reference genome, perform variant calling, and visualize. *S. enterica* was chosen due to its role in foodborne diseases and public health relevance.

## Comparing Bioinformatics Tools
Many innovative computational tools have been developed, and new ones continue to emerge. It is important to understand advantages and limitations and to pick appropriate methodologies that produce high-quality and reproducible genome assemblies. SeqKit (Shen et al., 2016a) and Filtlong (R. R. Wick, 2021) are commonly used packages for quality control (QC). SeqKit performs general FASTQ/A manipulations and filtration such as length size and excels at file handling while Filtlong is more optimized for long-read sequencing data (R. R. Wick et al., 2023). For QC visualization, NanoPlot’s key advantages include summary statistics and plots allowing to visually assess quality of long-read sequences (De Coster et al., 2018). Flye (Kolmogorov et al., 2019) and Canu (Koren et al., 2017) are some widely used genome assemblers. Canu generates more accurate assemblies with Pacific Bioscience (PacBio) sequencing but has longer computational times while Flye deals with repetitive genomic repeats more effectively but requires more RAM usage (R. R. Wick & Holt, 2019). Medaka (nanoporetech, 2019) and Nanopolish (Loman et al., 2015) are several programs used for polishing genomes which help reduce assembly errors. Medaka was reported to have higher accuracy than Nanopolish and benefits from picking the appropriate ONT and basecaller model  (R. R. Wick et al., 2023). For variant calling, BCFtools (Li, 2011), Medaka (nanoporetech, 2019) and Clair3 (Zheng et al., 2022) are commonly used. When comparing methods, BCFtools is more suited for short-reads (Hall et al., 2024), while Medaka and Clair3 use deep-learning for variant calling, resulting in higher accuracy especially for bacterial genomes although Clair3 has been shown to have the highest accuracy and the highest indel F1-score(Hall et al., 2024). For visualizations, many programs exists such as Integrative Genomics Viewer (IGV) which stands  out due to its versatility on different forms such as desktop and its handling of genomic data (Robinson et al., 2011), BCFtools’s innate SNPs and indels plotting function (Li, 2011) and Proksee's advantage of viewing bacterial genomes (Grant et al., 2023).

## Methodology
Based on previously evaluated advantages and disadvantages, Nanoplot, Filtlong, Flye, Minimap2, Clair3, BCFTools, IGV and Proksee will be selected for this analysis. Raw FASTQ files were generated using ONT sequencer employing R10 chemistry, with expected accuracy scores of ≥Q20. FASTQ files of *S. enterica* isolate (SRR32410565) were downloaded from NCBI’s Sequence Read Archives (https://trace.ncbi.nlm.nih.gov/Traces/?run=SRR32410565). All bioinformatics tools, except for Clair3, Proksee, and IGV, were installed via Bioconda on Bash shell environment because of its easy installation process and management of bioinformatics tools. IGV was installed directly on desktop (https://igv.org/doc/desktop/#DownloadPage/) (Robinson et al., 2011). Proksee’s web interface (https://proksee.ca/) was used to visualize the entire assembled genome (Grant et al., 2023).

## Data preprocessing, filtering, and assembly
The quality of raw reads and filtered reads were assessed using Nanoplot v1.46.2 (De Coster et al., 2018) from Nanopack collection via fastq input parameter “--fastq” to generate summary statistics and quality visualizations. Filtlong v0.3.1 (Wick, 2021) with parameters “--min_length 1000”, “--keep_percent 90”, and “-q 20” were applied to remove short reads, poor quality reads, and mean quality below 20. While a quality score of Q20 represents 99% base-calling accuracy, there is a 1 in 100 chance of an incorrect base call. In addition, a higher quality score may impact plasmid recovery since short reads may be removed (R. R. Wick et al., 2023). Genome assembly was conducted with Flye v2.9.6 (Kolmogorov et al., 2019) with five threads “-t 5”, parameter “--genome-size” will be set to 5m to reflect the known *S. enterica* genome of around 4.8 Mb, “--asm-coverage 100” for disjointing assembly, and “--nano-hq” as recommended by Flye’s usage manual for R10 chemistry. Medaka v2.1.1 (nanoporetech, 2019) was employed for assembly polishing and model parameter set to “r1041_e82_400bps_sup_v5.0.0”. Quality Assessment Tool for Genome Assemblies (QUAST) v5.3.0 (Gurevich et al., 2013) was employed to determine alignment quality for both unpolished and polished genomes.

## Alignment, Variant Calling, and Visualization
Minimap2 v2.30 (Li, 2018) was employed with parameters “-ax map-ont” to align both consensus genome and raw reads to the *S. enterica* reference genome (RefSeq GCF_000006945.2) and output to Sequence Alignment/Map (SAM) file. Subsequently, SAMtools v1.22.1 (Li et al., 2009) was applied to convert the resulting SAM file to sorted Binary Alignment/Map (BAM) files then indexed for variant calling. Clair3 (Zheng et al., 2022) was employed for variant calling with the model_path set to “r1041_e82_400bps_sup_v500”, “–include --include_all_ctgs”, and “--no_phasing_for_fa” parameters. Proksee (Grant et al., 2023) online web interface was used to visualize the entire assembled genome and the sequence similarity to the reference genome. The genome browser, IGV (Robinson et al., 2011, 2017), was deployed to visualize variants with inputs from the reference genome, aligned and indexed BAM files, and variant call format (vcf) files. Bcftools v1.22 (Li, 2011) plot-vcfstats was used to visualize SNPS and indels.

## Results
<img width="2209" height="1287" alt="Figure_1_Nanoplot_pre_post_filtering" src="https://github.com/user-attachments/assets/51476cbb-cfe7-4f3d-8422-6478874279c0" />
Figure 1. Quality control plots of *S. enterica* isolate SRR32410565 were generated using NanoPlot. Raw sequencing reads were filtered using Filtlong. Plot show read lengths versus the average read quality of A. pre-filtered and B. post-filtered. Post-filtering plot indicates a more compact distribution where low-quality reads and short reads were filtered out, rendering the raw read data suitable for genome assembly and variant calling.  
<br>
<br>
<img width="946" height="447" alt="Table_1_QUAST_summary_statistics" src="https://github.com/user-attachments/assets/f10e7741-28c2-4a59-a528-f7b483f478f3" />  <br>
Table 1. QUAST quality control summary statistics of assembled *S. enterica* genome pre and post polishing by Medaka. Polishing by Medaka of the genome showed either no or minimal changes for each metric.

<img width="2300" height="1306" alt="Figure_2_Assembled_genome" src="https://github.com/user-attachments/assets/97489991-a62f-4b76-8a11-5a376f6cd95d" />
Figure 2. Genome visualization of the assembled *S. enterica* SRR32410565 isolate by Proksee. Briefly, the fasta file of the assembled genome generated by Minimap2, then polished by Medaka, were uploaded to Proksee’s online web interface. A. Assembled genome. Each colour denotes a contig, for a total of 3 contigs. The length of the expected genome is 4.8 Mbp. The orange contig likely denotes a plasmid. B. The assembled genome compared to the reference genome (in green) showing sequence similarity (NCBI Ref Seq assembly GCF 000006945.2 ASM694v2) 
<br>
<br>
<img width="2516" height="1883" alt="Figure_3_IGV" src="https://github.com/user-attachments/assets/ac2ad65d-3203-42a7-a718-2b639c2caac8" />
Figure 3. IGV screenshot visualizations of *S. enterica* alignment. Raw reads were aligned to the *S. enterica* reference genome GCF_000006945.2. A. Visualization of raw reads aligned to the RefSeq NC_003197.2 showcasing the entire genome length of 4.86Mbp. B. Zoomed in segment of the first screenshot. C. Raw reads aligned to the entire length of the reference sequence RefSeq NC_003277.2, likely indicating a plasmid. Each track on the left denotes variants identified by Clair3, aligned reads, and annotated genes.

<img width="2273" height="1058" alt="Figure_4_SNP_Indels" src="https://github.com/user-attachments/assets/196f8118-0731-4e58-b5e6-8fc406ecfde0" />
Figure 4. Visualization of variants identified by Clair3 Variant calling in *S. enterica* isolate SRR32410565. A. Summary statistics were generated by BCFtools stats and visualized by VCFstats. Filtered raw reads aligned to the reference genome show varying SNP substitutions, with the A to G, C to T, G to A, and T to C having the most counts. B. Plot showing frequency of indel lengths, with both deletions and substitutions equally identified. A total of 10,155 variants were identified.
<br>
<br>
<img width="2145" height="2690" alt="Figure_5_finO_gene_IVG" src="https://github.com/user-attachments/assets/6d771743-8f1c-4aff-8e2a-9602cbe8eee4" />

Figure 5. IGV Screenshot visualization of SNPs identified in the *finO* gene of the assembled *S. enterica* genome to the NC_003277.2 reference sequence. Several high-quality SNPs (Phred score > 20), highlighted in the red boxes, were identified in the *finO* gene. Insertions of nucleotide G and A were observed when compared to the reference genome. 
<br>
<br>
<img width="2145" height="2676" alt="Figure_6_housekeeping_genes" src="https://github.com/user-attachments/assets/744bc18e-d449-4c3f-b992-fb5c38a6c515" />
Figure 6. IGV Screenshot visualization of SNPs identified in the *dnaN* and *thrA* gene of the assembled *S. enterica* genome to the NC_003197.2 reference sequence. Several high-quality SNPs (Phred score > 20) were identified in the *dnaN* and *thrA* genes. Insertions of nucleotide T and C were observed when compared to the reference genome. 
<br>

## Discussion


## References

De Coster, W., D’Hert, S., Schultz, D. T., Cruts, M., & Van Broeckhoven, C. (2018). NanoPack: visualizing and processing long-read sequencing data. Bioinformatics, 34(15), 2666–2669. https://doi.org/10.1093/bioinformatics/bty149

Frost, L., Lee, S., Yanchar, N., & Paranchych, W. (1989). finP and fisO mutations in FinP anti-sense RNA suggest a model for FinOP action in the repression of bacterial conjugation by the Flac plasmid JCFLO. Molecular and General Genetics MGG, 218(1), 152–160. https://doi.org/10.1007/BF00330578

Goodwin, S., McPherson, J. D., & McCombie, W. R. (2016). Coming of age: ten years of next-generation sequencing technologies. Nature Reviews Genetics, 17(6), 333–351. https://doi.org/10.1038/nrg.2016.49

Grant, J. R., Enns, E., Marinier, E., Mandal, A., Herman, E. K., Chen, C., Graham, M., Van Domselaar, G., & Stothard, P. (2023). Proksee: in-depth characterization and visualization of bacterial genomes. Nucleic Acids Research, 51(W1), W484–W492. https://doi.org/10.1093/nar/gkad326

Gurevich, A., Saveliev, V., Vyahhi, N., & Tesler, G. (2013). QUAST: quality assessment tool for genome assemblies. Bioinformatics, 29(8), 1072–1075. https://doi.org/10.1093/bioinformatics/btt086

Hall, M. B., Wick, R. R., Judd, L. M., Nguyen, A. N., Steinig, E. J., Xie, O., Davies, M., Seemann, T., Stinear, T. P., & Coin, L. (2024). Benchmarking reveals superiority of deep learning variant callers on bacterial nanopore sequence data. ELife, 13. https://doi.org/10.7554/eLife.98300

Hiley, L., Graham, R. M. A., & Jennison, A. V. (2019). Genetic characterisation of variants of the virulence plasmid, pSLT, in Salmonella enterica serovar Typhimurium provides evidence of a variety of evolutionary directions consistent with vertical rather than horizontal transmission. PLOS ONE, 14(4), e0215207-. https://doi.org/10.1371/journal.pone.0215207

Iyer, S. V, Goodwin, S., & McCombie, W. R. (2024). Leveraging the power of long reads for targeted sequencing. Genome Research, 34(11), 1701–1718. https://doi.org/10.1101/gr.279168.124

Kolmogorov, M., Yuan, J., Lin, Y., & Pevzner, P. A. (2019). Assembly of long, error-prone reads using repeat graphs. Nature Biotechnology, 37(5), 540–546. https://doi.org/10.1038/s41587-019-0072-8 <br />

Koren, S., Walenz, B. P., Berlin, K., Miller, J. R., Bergman, N. H., & Phillippy, A. M. (2017). Canu: scalable and accurate long-read assembly via adaptive k-mer weighting and repeat separation. Genome Research, 27(5), 722–736. https://doi.org/10.1101/gr.215087.116

Li, H. (2011). A statistical framework for SNP calling, mutation discovery, association mapping and population genetical parameter estimation from sequencing data. Bioinformatics (Oxford, England), 27(21), 2987–2993. https://doi.org/10.1093/bioinformatics/btr509

Li, H. (2018). Minimap2: pairwise alignment for nucleotide sequences. Bioinformatics, 34(18), 3094–3100. https://doi.org/10.1093/bioinformatics/bty191

Li, H., Handsaker, B., Wysoker, A., Fennell, T., Ruan, J., Homer, N., Marth, G., Abecasis, G., Durbin, R., & Subgroup, 1000 Genome Project Data Processing. (2009). The Sequence Alignment/Map format and SAMtools. Bioinformatics, 25(16), 2078–2079. https://doi.org/10.1093/bioinformatics/btp352

Liao, Z., & Smirnov, A. (2023). FinO/ProQ-family proteins: an evolutionary perspective. Bioscience Reports, 43(3), BSR20220313. https://doi.org/10.1042/BSR20220313

Loman, N. J., Quick, J., & Simpson, J. T. (2015). A complete bacterial genome assembled de novo using only nanopore sequencing data. Nature Methods, 12(8), 733–735. https://doi.org/10.1038/nmeth.3444

Mengfei, P., Serajus, S., L, B. R., & Debabrata, B. (2018). Alterations of Salmonella enterica Serovar Typhimurium Antibiotic Resistance under Environmental Pressure. Applied and Environmental Microbiology, 84(19), e01173-18. https://doi.org/10.1128/AEM.01173-18

P, dos S. A. M., Pedro, P., G, F. R., Beatriz, P. A., Carolina, de J. A., Alan, O., Dália, R., Magaly, T., Jianghong, M., Marc, A., & A, C.-J. C. (2024). Genomic characterization of a clonal emergent Salmonella Minnesota lineage in Brazil reveals the presence of a novel megaplasmid of resistance and virulence. Applied and Environmental Microbiology, 90(11), e01579-24. https://doi.org/10.1128/aem.01579-24

Paranthaman, K., Mook, P., Curtis, D., Evans, E.-W., Crawley-Boevey, E., Dabke, G., Carroll, K., McCormick, J., Dallman, T. J., & Crook, P. (2021). Development and evaluation of an outbreak surveillance system integrating whole genome sequencing data for non-typhoidal Salmonella in London and South East of England, 2016–17. Epidemiology and Infection, 149, e164. https://doi.org/10.1017/S0950268821001400

Robinson, J. T., Thorvaldsdóttir, H., Wenger, A. M., Zehir, A., & Mesirov, J. P. (2017). Variant Review with the Integrative Genomics Viewer. Cancer Research, 77(21), e31–e34. https://doi.org/10.1158/0008-5472.CAN-17-0337

Robinson, J. T., Thorvaldsdóttir, H., Winckler, W., Guttman, M., Lander, E. S., Getz, G., & Mesirov, J. P. (2011). Integrative genomics viewer. Nature Biotechnology, 29(1), 24–26. https://doi.org/10.1038/nbt.1754

Shen, W., Le, S., Li, Y., & Hu, F. (2016a). SeqKit: A Cross-Platform and Ultrafast Toolkit for FASTA/Q File Manipulation. PLOS ONE, 11(10), e0163962-. https://doi.org/10.1371/journal.pone.0163962

Wick, R. R. (2021). Filtlong. https://github.com/rrwick/Filtlong

Wick, R. R., & Holt, K. E. (2019). Benchmarking of long-read assemblers for prokaryote whole genome      sequencing. F1000Research, 8, 2138. https://doi.org/10.12688/f1000research.21782.3

Wick, R. R., Judd, L. M., & Holt, K. E. (2023). Assembling the perfect bacterial genome using Oxford Nanopore and Illumina sequencing. PLOS Computational Biology, 19(3), e1010905-. https://doi.org/10.1371/journal.pcbi.1010905

Zheng, Z., Li, S., Su, J., Leung, A. W.-S., Lam, T.-W., & Luo, R. (2022). Symphonizing pileup and full-alignment for deep learning-based long-read variant calling. Nature Computational Science, 2(12), 797–803. https://doi.org/10.1038/s43588-022-00387-x
